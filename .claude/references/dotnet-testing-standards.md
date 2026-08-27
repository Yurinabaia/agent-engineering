# Padrões de Teste — xUnit + Moq

Contexto sob demanda. Carregue antes de escrever ou revisar testes.

---

## Onde os testes moram

`<Sln>.TesteUnitario/<Feature>/<Nome>ServiceTests.cs` — a pasta espelha a pasta da feature em
`Application/Services/`.

Stack: **xUnit 2.9** + **Moq 4.20**, `net10.0`, `<Using Include="Xunit" />` global no csproj.

## Nomenclatura

`Metodo_QuandoCondicao_ResultadoEsperado` — em português, como o resto do domínio:

```
GetAllAsync_QuandoExistemVersoes_RetornaListaOrdenada
GetByIdAsync_QuandoNaoEncontrado_LancaException
CreateAsync_QuandoNomeDuplicado_LancaExceptionComStatus409
UpdateAsync_QuandoValido_PreencheCamposDeAuditoria
```

## Esqueleto padrão

```csharp
using DTASkills.Application.Common.Interfaces.Persistence;
using DTASkills.Application.Dtos.ModelNames;
using DTASkills.Application.Services.ModelNames;
using DTASkills.Domain.Entities.ModelNames;
using Moq;
using System.Linq.Expressions;

namespace DTASkills.TesteUnitario.ModelNames;

public class ModelNameServiceTests
{
  private readonly Mock<IUnitOfWork> _unitOfWorkMock;
  private readonly Mock<IRepository<ModelNameEntity>> _repoMock;
  private readonly ModelNameService _service;

  public ModelNameServiceTests()
  {
    _unitOfWorkMock = new Mock<IUnitOfWork>();
    _repoMock = new Mock<IRepository<ModelNameEntity>>();
    _unitOfWorkMock.Setup(u => u.Repository<ModelNameEntity>()).Returns(_repoMock.Object);
    _service = new ModelNameService(_unitOfWorkMock.Object);
  }

  // --- GetByIdAsync ---

  [Fact]
  public async Task GetByIdAsync_QuandoEncontrado_RetornaDto()
  {
    var entity = new ModelNameEntity { Id = 1, ModelName = "gpt-4o" };

    _repoMock.Setup(r => r.GetFirstAsync(
        It.IsAny<Expression<Func<ModelNameEntity, bool>>>(),
        It.IsAny<bool>(),
        It.IsAny<Expression<Func<ModelNameEntity, object>>[]>()))
      .ReturnsAsync(entity);

    var result = await _service.GetByIdAsync(1);

    Assert.NotNull(result);
    Assert.Equal(1, result.Id);
    Assert.Equal("gpt-4o", result.ModelName);
  }

  [Fact]
  public async Task GetByIdAsync_QuandoNaoEncontrado_LancaException()
  {
    _repoMock.Setup(r => r.GetFirstAsync(
        It.IsAny<Expression<Func<ModelNameEntity, bool>>>(),
        It.IsAny<bool>(),
        It.IsAny<Expression<Func<ModelNameEntity, object>>[]>()))
      .ReturnsAsync((ModelNameEntity?)null);

    await Assert.ThrowsAsync<Exception>(() => _service.GetByIdAsync(99));
  }
}
```

## Regras

- Organizar por método com comentário separador: `// --- GetByIdAsync ---`.
- `GetFirstAsync` tem 3 parâmetros no mock (predicate, `asNoTracking`, `includes`) — o `It.IsAny`
  dos três é obrigatório, senão o setup não casa e o mock devolve `null` silenciosamente.
- Serviço com auditoria: mockar `IPortalAuthDataContext` retornando um e-mail fixo
  (`"user@example.com"`) no construtor da classe de teste.
- Verificar efeito colateral de escrita com `Verify`:
  ```csharp
  _repoMock.Verify(r => r.AddAsync(It.IsAny<ModelNameEntity>()), Times.Once);
  _unitOfWorkMock.Verify(u => u.CommitAsync(), Times.Once);
  ```
- Nunca usar banco real, `InMemory` ou `DbContext` em teste unitário — só mocks.
- Sem `Thread.Sleep`, sem dependência de ordem entre testes.

## Cobertura mínima por feature

Para **cada** método público do service:

| Cenário | Obrigatório |
|---|---|
| Happy path | sim |
| Recurso não encontrado (404) | sim, quando houver busca por id |
| Validação de negócio violada (400/409) | sim, quando houver regra |
| Commit chamado exatamente uma vez | sim, em métodos de escrita |
| Campos de auditoria preenchidos | sim, quando a entidade tiver auditoria |

## Comandos

```bash
dotnet test <Sln>.Api/<Sln>.Api.sln
dotnet test <Sln>.Api/<Sln>.Api.sln --filter "FullyQualifiedName~ModelNameServiceTests"
dotnet test <Sln>.Api/<Sln>.Api.sln --collect:"XPlat Code Coverage"
```
