# Clean Architecture em .NET — Referência de Implementação

Contexto sob demanda. **Carregue este arquivo antes de criar ou alterar qualquer feature.**
Todo trecho abaixo é código real do projeto de referência (`DTASkills.Api`) — copie o padrão,
não invente um novo.

---

## 1. As camadas e o que mora em cada uma

| Camada | Responsabilidade | Pode depender de | Nunca contém |
|--------|------------------|------------------|--------------|
| **Domain** | Entidades, enums, value objects, contratos de contexto | nada | EF Core fluente, HTTP, DI |
| **Application** | Casos de uso, DTOs, mapeamento, orquestração | Domain | `DbContext`, `HttpContext`, atributos de MVC |
| **Infrastructure** | EF Core, repositórios, JWT, clients HTTP | Application, Domain | regra de negócio |
| **Api** | Controllers, filtros, middlewares, composição | Application, Infrastructure, Contracts | regra de negócio, acesso a dados |
| **Contracts** | Contratos públicos (request/response, paginação) | Domain | lógica |

### Por que Clean e não "vertical slice puro"?

As camadas são horizontais, mas **as pastas dentro delas são verticais**: `Services/ChatAgents/`,
`Dtos/ChatAgents/`, `Entities/ChatAgents/`. Uma feature é uma fatia vertical *atravessando*
camadas estáveis. O agente acha tudo da feature buscando o nome dela em cada projeto, e a regra
de dependência continua protegendo o núcleo.

> Na apresentação: este é o ponto de equilíbrio entre "AI-friendly" (tudo da feature com o mesmo
> nome de pasta) e "arquitetura defensável" (dependências apontando para dentro).

---

## 2. Domain — Entidade

`<Sln>.Domain/Entities/<Feature>/<Nome>Entity.cs`

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace DTASkills.Domain.Entities.ModelNames
{
  [Table("model_names")]
  public class ModelNameEntity
  {
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    [Column("id")]
    public int Id { get; set; }

    [Required]
    [MaxLength(150)]
    [Column("model_name")]
    public string ModelName { get; set; } = string.Empty;
  }
}
```

Regras:
- Tabela e colunas em `snake_case` via Data Annotations (o projeto **não** usa configuração
  fluente por entidade; relacionamentos e índices ficam no `OnModelCreating`).
- `string` sempre inicializada com `= string.Empty` (nullable habilitado).
- Datas: `[Column("...", TypeName = "timestamp with time zone")]` (PostgreSQL).
- Auditoria (quando a feature exigir): `RecCreatedBy`, `RecCreatedOn`, `RecModifiedBy`,
  `RecModifiedOn` — `MaxLength(50)` nos campos de usuário.

---

## 3. Application — DTO

`<Sln>.Application/Dtos/<Feature>/<Nome>Dto.cs`

```csharp
namespace DTASkills.Application.Dtos.ModelNames
{
  public class ModelNameDto
  {
    public int Id { get; set; }
    public string ModelName { get; set; } = string.Empty;
  }
}
```

- Um DTO por finalidade quando entrada e saída divergem (`...Dto`, `...UpdateDto`, `...SummaryDto`).
- DTO não tem método, não tem validação de negócio, não conhece entidade.

---

## 4. Application — Mapper (extension estático)

`<Sln>.Application/Extensions/<Feature>/<Nome>Extensions.cs`

```csharp
using DTASkills.Application.Dtos.ModelNames;
using DTASkills.Domain.Entities.ModelNames;

namespace DTASkills.Application.Extensions.ModelNames
{
  public static class ModelNameExtensions
  {
    public static ModelNameDto ToDto(this ModelNameEntity entity)
    {
      return new ModelNameDto
      {
        Id = entity.Id,
        ModelName = entity.ModelName
      };
    }

    public static ModelNameEntity ToEntity(this ModelNameDto dto)
    {
      return new ModelNameEntity
      {
        Id = dto.Id,
        ModelName = dto.ModelName
      };
    }
  }
}
```

- Não usar AutoMapper. O projeto mapeia à mão, explicitamente.
- Campos de auditoria **não** são preenchidos no mapper — o service os preenche.

---

## 5. Application — Interface do Service

`<Sln>.Application/Services/<Feature>/I<Nome>Service.cs`

```csharp
using DTASkills.Application.Dtos.ModelNames;

namespace DTASkills.Application.Services.ModelNames
{
  public interface IModelNameService
  {
    Task<List<ModelNameDto>> GetAllAsync();
    Task<ModelNameDto> GetByIdAsync(int id);
    Task<ModelNameDto> CreateAsync(ModelNameDto dto);
    Task<ModelNameDto> UpdateAsync(ModelNameDto dto);
    Task DeleteAsync(int id);
  }
}
```

---

## 6. Application — Service (o coração da feature)

`<Sln>.Application/Services/<Feature>/<Nome>Service.cs`

```csharp
using DTASkills.Application.Common.Interfaces.Persistence;
using DTASkills.Application.Dtos.ModelNames;
using DTASkills.Application.Extensions.ModelNames;
using DTASkills.Domain.Entities.ModelNames;

namespace DTASkills.Application.Services.ModelNames
{
  public class ModelNameService(IUnitOfWork unitOfWork) : IModelNameService
  {
    private readonly IUnitOfWork _unitOfWork = unitOfWork;

    public async Task<ModelNameDto> GetByIdAsync(int id)
    {
      try
      {
        var repo = _unitOfWork.Repository<ModelNameEntity>();
        var entity = await repo.GetFirstAsync(e => e.Id == id, asNoTracking: true)
          ?? throw new Exception($"Modelo {id} não encontrado 404");

        return entity.ToDto();
      }
      catch (Exception ex)
      {
        throw new Exception(ex.Message);
      }
    }

    public async Task<ModelNameDto> CreateAsync(ModelNameDto dto)
    {
      try
      {
        var repo = _unitOfWork.Repository<ModelNameEntity>();
        var entity = dto.ToEntity();
        await repo.AddAsync(entity);
        await _unitOfWork.CommitAsync();
        return entity.ToDto();
      }
      catch (Exception ex)
      {
        throw new Exception(ex.Message);
      }
    }
  }
}
```

Padrões obrigatórios:
1. Primary constructor + campo privado `_camelCase`.
2. `var repo = _unitOfWork.Repository<XEntity>();` no início de cada método.
3. `?? throw new Exception($"... 404")` para "não encontrado".
4. `try/catch` re-lançando `new Exception(ex.Message)` (preserva o sufixo de status).
5. Escrita termina em `await _unitOfWork.CommitAsync();`.
6. Update: primeiro `GetFirstAsync(..., asNoTracking: true)` para validar existência, depois
   `repo.Update(entity, setState: true)`.
7. Com auditoria, injetar `IPortalAuthDataContext` e preencher antes do commit:
   `entity.RecCreatedBy = _portalAuthDataContext.Email; entity.RecCreatedOn = DateTime.UtcNow;`

### API do repositório disponível

```csharp
Task<TEntity?> GetFirstAsync(predicate, bool asNoTracking = false, params includes);
Task<IEnumerable<TEntity>> GetAllAsync();
Task<IEnumerable<TEntity>> WhereAsync(predicate, orderBy, orderByDescending, skip, take, asNoTracking, includes);
Task<bool> AnyAsync(predicate);
void Add(TEntity); Task AddAsync(TEntity);
void Update(TEntity, bool setState = false);
void Remove(TEntity); void RemoveRange(IEnumerable<TEntity>);
```

Precisa de algo que a interface não oferece? **Estenda `IRepository`/`Repository`** — não injete
`DbContext` no service.

---

## 7. Application — Registro no DI

`<Sln>.Application/DependencyInjection.cs`

```csharp
services.AddScoped<IModelNameService, ModelNameService>();
```

- Services de feature são **sempre** `Scoped`.
- Providers sem estado podem ser `Singleton` (ver `IBillingProvider`).
- Integrações HTTP usam `services.AddHttpClient<IXService, XService>(...)`.

Esquecer este passo é o erro mais comum: compila e quebra em runtime com
`Unable to resolve service for type ...`.

---

## 8. Infrastructure — DbContext e migration

`<Sln>.Infrastructure/Data/ApplicationDbContext.cs`

```csharp
public DbSet<ModelNameEntity> ModelNames { get; set; } = null!;

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
  modelBuilder.Entity<ChatSkillsEntity>()
    .HasIndex(e => e.Name)
    .IsUnique()
    .HasDatabaseName("uq_chat_skills_name");
  // relacionamentos N:N via .UsingEntity<...>()
}
```

Depois:

```bash
dotnet ef migrations add Add<Nome> -p <Sln>.Infrastructure -s <Sln>.Api
```

- Nome do índice único: `uq_<tabela>_<coluna>`.
- Migration é obrigatória em toda mudança de entidade — sem exceção.

---

## 9. Api — Controller

`<Sln>.Api/Controllers/<Nome>Controller.cs`

```csharp
using DTASkills.Api.Filter;
using DTASkills.Application.Dtos.ModelNames;
using DTASkills.Application.Services.ModelNames;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace DTASkills.Api.Controllers
{
  [ApiController]
  [ErrorHandlingFilter]
  [Route("api/model-names")]
  public class ModelNamesController(IModelNameService modelNameService) : ControllerBase
  {
    [Authorize("PortalLogPolicy")]
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
      var result = await modelNameService.GetAllAsync();
      return Ok(result);
    }

    [Authorize("AdminPolicy")]
    [HttpGet("{id:int}")]
    public async Task<IActionResult> GetById(int id)
    {
      var result = await modelNameService.GetByIdAsync(id);
      return Ok(result);
    }

    [Authorize("AdminPolicy")]
    [HttpPost]
    public async Task<IActionResult> Create([FromBody] ModelNameDto dto)
    {
      var result = await modelNameService.CreateAsync(dto);
      return CreatedAtAction(nameof(GetById), new { id = result.Id }, result);
    }

    [Authorize("AdminPolicy")]
    [HttpPut("{id:int}")]
    public async Task<IActionResult> Update(int id, [FromBody] ModelNameDto dto)
    {
      dto.Id = id;
      var result = await modelNameService.UpdateAsync(dto);
      return Ok(result);
    }

    [Authorize("AdminPolicy")]
    [HttpDelete("{id:int}")]
    public async Task<IActionResult> Delete(int id)
    {
      await modelNameService.DeleteAsync(id);
      return NoContent();
    }
  }
}
```

Regras:
- Injeção pelo primary constructor, usada direto (sem campo privado).
- Retorno sempre `Task<IActionResult>`: `Ok`, `CreatedAtAction`, `NoContent`.
- Constraint de rota tipada: `{id:int}`.
- Sem `try/catch` — o `[ErrorHandlingFilter]` cuida.
- Controllers com muitas actions relacionadas ganham subpasta: `Controllers/AgentVersions/`.

---

## 10. Erros — mapeamento de status

`ErrosException` interpreta o sufixo da mensagem:

| Sufixo na mensagem | Status | Quando usar |
|---|---|---|
| ` 400` | 400 | validação de entrada / regra violada |
| ` 401` | 401 | credencial ausente ou inválida |
| ` 403` | 403 | autenticado sem permissão |
| ` 404` | 404 | recurso não encontrado |
| ` 409` | 409 | duplicidade / conflito de estado |
| (nenhum) | 500 | erro inesperado |

`" IdLog=<id>"` no fim da mensagem sai como extensão `log.id` no `ProblemDetails`.

---

## 11. Contracts — respostas padronizadas

```csharp
new ResponseContract<T>(data);                              // { data: ... }
new PagedResponseContract<T>(data, page, pageSize, total);  // + page/pageSize/totalRecords
new PaginationFilterContract(page, pageSize);               // default 1/10, teto 1000
```

Use paginação sempre que o endpoint puder retornar coleção não limitada.

---

## 12. Checklist de revisão de camada

- [ ] Entidade em Domain, com `[Table]`/`[Column]` em snake_case
- [ ] DTO sem lógica, em `Dtos/<Feature>/`
- [ ] Mapper manual em `Extensions/<Feature>/`
- [ ] Interface + implementação em `Services/<Feature>/`
- [ ] Service usa `IUnitOfWork`, nunca `DbContext`
- [ ] Erros com sufixo de status e mensagem em português
- [ ] Registro em `DependencyInjection.cs`
- [ ] `DbSet` no `ApplicationDbContext` + migration gerada
- [ ] Controller fino, com policy declarada e rota kebab-case
- [ ] Testes xUnit/Moq espelhando a pasta da feature
- [ ] `dotnet build` e `dotnet test` verdes
