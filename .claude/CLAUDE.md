# CLAUDE.md — Regras Globais do Projeto

> API de exemplo em **Clean Architecture com .NET 10**, usada como base da apresentação
> "Construção de API em Clean Architecture com agentes de IA".
> A estrutura espelha o projeto real `DTASkills.Api`.

---

## 1. Estrutura da Solução

```
<Sln>.Api/                  # Camada de apresentação (ASP.NET Core Web API)
  Controllers/              #   Controllers por feature (pasta por feature quando >1 controller)
  Filter/                   #   ErrorHandlingFilterAttribute (RFC 7807)
  Middleware/               #   Extração de dados de autenticação
  OpenApi/                  #   Transformers do OpenAPI/Swagger
  Common/                   #   Constantes de header etc.
  Program.cs                #   Composition root

<Sln>.Application/          # Casos de uso / orquestração
  Services/<Feature>/       #   IXService + XService (regra de negócio)
  Dtos/<Feature>/           #   DTOs de entrada e saída dos services
  Extensions/<Feature>/     #   Mappers estáticos ToDto()/ToEntity()
  Common/Interfaces/        #   Contratos de persistência e de providers
  Erros/                    #   ErrosException + IServiceException
  Utils/                    #   Helpers puros

<Sln>.Domain/               # Núcleo — não depende de NADA
  Entities/<Feature>/       #   Entidades persistidas (mapeadas via Data Annotations)
  Enums/ Models/ ValueObjects/ Interfaces/

<Sln>.Infrastructure/       # Detalhes técnicos
  Data/                     #   ApplicationDbContext, UnitOfWork, DbConnector
  Repository/Persistence/   #   Repository<TEntity> genérico
  Context/                  #   Contextos de autenticação por request
  Authentication/           #   Settings de JWT
  Services/                 #   Integrações HTTP externas
  Migrations/               #   Migrations do EF Core

<Sln>.Contracts/            # Contratos públicos da API (request/response, paginação)
<Sln>.TesteUnitario/        # xUnit + Moq
<Sln>.Api.AppHost/          # .NET Aspire (orquestração local)
<Sln>.Api.ServiceDefaults/  # Telemetria, health checks, resiliência
```

## 2. Regra de Dependência (inviolável)

```
Api ──────► Application ──────► Domain
 │              ▲                 ▲
 │              │                 │
 └──► Infrastructure ─────────────┘
 └──► Contracts ──────────────────┘
```

- **Domain** não referencia nenhum outro projeto.
- **Application** referencia apenas Domain.
- **Infrastructure** referencia Application (implementa as interfaces dela).
- **Api** referencia Application, Infrastructure e Contracts — e só monta a composição.
- ❌ Nunca injetar `ApplicationDbContext` em um Service. Persistência **sempre** via `IUnitOfWork`.
- ❌ Nunca retornar `Entity` de um Controller. A fronteira externa é DTO/Contract.
- ❌ Nunca colocar regra de negócio em Controller. Controller é fino: chama o service e devolve.

## 3. Anatomia de uma Feature (8 pontos de toque)

Toda feature nova percorre **sempre** esta ordem:

| # | Camada | Arquivo | Padrão |
|---|--------|---------|--------|
| 1 | Domain | `Entities/<Feature>/<Nome>Entity.cs` | `[Table("snake_case")]` + `[Column("snake_case")]` |
| 2 | Application | `Dtos/<Feature>/<Nome>Dto.cs` | POCO simples, sem lógica |
| 3 | Application | `Extensions/<Feature>/<Nome>Extensions.cs` | `static ToDto()` / `ToEntity()` |
| 4 | Application | `Services/<Feature>/I<Nome>Service.cs` | Só assinaturas `Async` |
| 5 | Application | `Services/<Feature>/<Nome>Service.cs` | Primary constructor + `IUnitOfWork` |
| 6 | Application | `DependencyInjection.cs` | `services.AddScoped<I<Nome>Service, <Nome>Service>();` |
| 7 | Infrastructure | `Data/ApplicationDbContext.cs` | `DbSet<...>` + índices no `OnModelCreating` + migration |
| 8 | Api | `Controllers/<Nome>Controller.cs` | `[ApiController] [ErrorHandlingFilter] [Route("api/kebab-case")]` |
| 9 | Testes | `TesteUnitario/<Feature>/<Nome>ServiceTests.cs` | xUnit + Moq, um `[Fact]` por caminho |

Detalhes e código de referência: `.claude/references/clean-architecture-dotnet.md`.

## 4. Convenções de Código

- **C# 13 / .NET 10**, `Nullable` e `ImplicitUsings` habilitados em todos os projetos.
- **Indentação de 2 espaços**, chaves em linha própria (Allman).
- **Namespace em bloco** (`namespace X { }`) no código de produção; file-scoped é aceito nos testes.
- **Primary constructors** para injeção de dependência:
  ```csharp
  public class ModelNameService(IUnitOfWork unitOfWork) : IModelNameService
  {
    private readonly IUnitOfWork _unitOfWork = unitOfWork;
  }
  ```
- Campos privados com `_camelCase`. Interfaces com prefixo `I`. Entidades com sufixo `Entity`,
  DTOs com `Dto`, contratos com `Contract`.
- Tudo que toca I/O é `async`/`await` e termina em `Async`.
- Rotas em **kebab-case** (`api/model-names`), tabelas e colunas em **snake_case**.

## 5. Tratamento de Erro (padrão do projeto)

O `ErrorHandlingFilterAttribute` converte exceções em `ProblemDetails` (RFC 7807). O status HTTP
é derivado do **sufixo numérico na mensagem** da exceção:

```csharp
throw new Exception($"Produto {id} não encontrado 404");
throw new Exception("Nome já cadastrado 409");
throw new Exception("Preço deve ser maior que zero 400");
// sem sufixo => 500
```

- Mensagens de erro em **português**, voltadas ao usuário da API.
- Services envolvem a operação em `try/catch` e re-lançam com `throw new Exception(ex.Message)`
  para preservar o sufixo de status.
- ❌ Não retornar `BadRequest(...)` manualmente no controller — deixe o filtro agir.

## 6. Persistência

- Acesso a dados **somente** por `IUnitOfWork.Repository<TEntity>()`.
- `GetFirstAsync(predicate, asNoTracking: true)` para leitura; sem `asNoTracking` quando for
  alterar/remover.
- Escrita sempre finaliza com `await _unitOfWork.CommitAsync();`.
- Toda mudança de schema exige migration:
  `dotnet ef migrations add <NomeDescritivo> -p <Sln>.Infrastructure -s <Sln>.Api`.
- Entidades com auditoria preenchem `RecCreatedBy/On` e `RecModifiedBy/On` a partir de
  `IPortalAuthDataContext.Email` e `DateTime.UtcNow`.

## 7. Autorização

- `[Authorize("PortalLogPolicy")]` — usuário autenticado do portal (leitura).
- `[Authorize("AdminPolicy")]` — exige role `Admin` (escrita/administração).
- Toda action nova declara explicitamente sua policy.

## 8. Testes

- xUnit + Moq, no projeto `<Sln>.TesteUnitario`, espelhando a pasta da feature.
- Nomes em português no formato `Metodo_QuandoCondicao_ResultadoEsperado`:
  `GetByIdAsync_QuandoNaoEncontrado_LancaException`.
- Mock de `IUnitOfWork` devolvendo `Mock<IRepository<TEntity>>`; nunca banco real em teste unitário.
- Cobertura mínima por feature: happy path + não encontrado + validação inválida de cada método público.

## 9. Comandos

```bash
dotnet build <Sln>.Api/<Sln>.Api.sln          # compilar tudo
dotnet test  <Sln>.Api/<Sln>.Api.sln          # rodar testes
dotnet format <Sln>.Api/<Sln>.Api.sln --verify-no-changes   # checar formatação
dotnet run --project <Sln>.Api.AppHost        # subir via Aspire
```

## 10. Fluxo de Trabalho com Agentes

Toda feature entra pelo **Jira** e desce pela esteira:

```
/feature-from-jira PROJ-123   → features/PROJ-123-slug/feature.md
/plan-feature PROJ-123        → features/PROJ-123-slug/plan.md
/execute PROJ-123             → código, camada por camada + testes
/code-review                  → relatório de revisão
/commit                       → commit convencional com a chave do Jira
```

Atalho de ponta a ponta: `/end-to-end-feature PROJ-123`.

Regras da esteira:
- Nenhum código é escrito antes do `plan.md` aprovado.
- Nenhuma feature é "concluída" com `dotnet build` ou `dotnet test` falhando.
- O `checklist.md` da feature é atualizado a cada camada concluída.
