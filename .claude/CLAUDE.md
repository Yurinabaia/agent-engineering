# CLAUDE.md — Regras Globais do Projeto

> Aplicação de exemplo da apresentação "Construção de API em Clean Architecture com agentes de
> IA": **API em Clean Architecture com .NET 10** (estrutura espelhando o projeto real
> `DTASkills.Api`) e **frontend Angular 20 + PO-UI** consumindo essa API.

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

### Frontend (Angular 20 + PO-UI)

```
src/app/
  core/                     # infra transversal — criada uma vez, quase nunca muda
    interceptors/           #   auth (Bearer) e error (ProblemDetails -> PoNotificationService)
    guards/ services/ models/
  shared/                   # reutilizável por 3+ features
    components/ pipes/ utils/
  features/                 # UMA PASTA POR FEATURE (fatia vertical)
    <feature>/
      models/               #   interface espelhando o DTO da API (camelCase)
      services/             #   HttpClient tipado, um método por endpoint
      pages/                #   <x>-list/ e <x>-form/ (standalone components)
      <feature>.routes.ts   #   rotas lazy da feature
  app.config.ts             # providers (router, http + interceptors, animations)
  app.routes.ts             # rotas raiz, lazy por feature
environments/               # apiUrl por ambiente
```

**Regra de dependência do frontend:** `features/*` -> `shared/` -> `core/`.
Uma feature **nunca** importa de outra feature; o que for comum sobe para `shared/`.

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

### Anatomia de uma tela (7 pontos de toque)

| # | Alvo | Arquivo | Padrão |
|---|---|---|---|
| 1 | Model | `features/<f>/models/<x>.model.ts` | `interface` espelhando o DTO em camelCase |
| 2 | Service | `features/<f>/services/<x>.service.ts` | `inject(HttpClient)`, sem `subscribe` |
| 3 | Rotas | `features/<f>/<f>.routes.ts` | `loadComponent`, kebab-case |
| 4 | Rota raiz | `app.routes.ts` | `loadChildren` lazy |
| 5 | Listagem | `pages/<x>-list/` | `po-page-list` + `po-table` |
| 6 | Formulário | `pages/<x>-form/` | `po-page-edit` + reactive forms |
| 7 | Testes | `*.spec.ts` ao lado | Jasmine/Karma, `provideNoopAnimations()` |

Detalhes e código de referência: `.claude/references/angular-po-ui.md`.

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

## 4.1 Convenções do Frontend

- **Angular 20 standalone** — sem NgModules em código novo. `inject()` no lugar de construtor.
- `ChangeDetectionStrategy.OnPush` sempre; estado de tela em `signal()`.
- Novo control flow: `@if` / `@for` **com `track`**. `*ngIf`/`*ngFor` em código novo é desvio.
- Reactive forms (`fb.nonNullable.group`); nada de `ngModel`.
- Componentes do PO importados por **módulo granular** (`PoTableModule`, `PoFieldModule`,
  `PoPageModule`), não `PoModule` inteiro.
- Layout de formulário com o grid do PO (`po-row`, `po-md-*`). CSS próprio só para o que o design
  system não cobre.
- `provideAnimations()` é obrigatório no `app.config.ts` — sem ele, modal, notification e loading
  do PO não aparecem, e **sem erro no console**.
- Sem `any`. Sem `subscribe` aninhado. Sem `subscribe` dentro de service.
- Toda feature é **lazy**.

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

**O mesmo contrato chega à tela.** No frontend, o `errorInterceptor` lê o `ProblemDetails`
(`title`, `status`, extensão `log.id`) e exibe via `PoNotificationService.error`. Consequência
prática: **nenhum componente Angular trata erro de API**. Um `notification.error(...)` dentro de
componente quase sempre é duplicação do interceptor — o usuário vê dois toasts.

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

### API

- xUnit + Moq, no projeto `<Sln>.TesteUnitario`, espelhando a pasta da feature.
- Nomes em português no formato `Metodo_QuandoCondicao_ResultadoEsperado`:
  `GetByIdAsync_QuandoNaoEncontrado_LancaException`.
- Mock de `IUnitOfWork` devolvendo `Mock<IRepository<TEntity>>`; nunca banco real em teste unitário.
- Cobertura mínima por feature: happy path + não encontrado + validação inválida de cada método público.

### Frontend

- Jasmine + Karma, spec ao lado do arquivo testado.
- Componente standalone entra em `imports` do `TestBed`; `provideNoopAnimations()` é obrigatório
  (PO-UI usa animações); `http.verify()` no `afterEach`.
- Cobertura mínima: um caso por método de service, carga e erro na listagem, confirmação antes de
  excluir, e formulário inválido que não chama o service.
- Detalhes: `.claude/references/angular-testing-standards.md`.

## 9. Comandos

### API

```bash
dotnet build <Sln>.Api/<Sln>.Api.sln          # compilar tudo
dotnet test  <Sln>.Api/<Sln>.Api.sln          # rodar testes
dotnet format <Sln>.Api/<Sln>.Api.sln --verify-no-changes   # checar formatação
dotnet run --project <Sln>.Api.AppHost        # subir via Aspire
```

### Frontend

```bash
npm install
npx tsc --noEmit                                      # checagem de tipos
npm run build                                         # build de produção
npm test -- --watch=false --browsers=ChromeHeadless   # testes
npx ng lint                                           # lint
npm start                                             # http://localhost:4200
```

## 10. Fluxo de Trabalho com Agentes

Toda feature entra pelo **Jira** e desce pela esteira:

```
/feature-from-jira PROJ-123   -> features/PROJ-123-slug/feature.md   (requisito: API + tela)

  trilha API                          trilha FRONTEND
  /plan-feature PROJ-123              /plan-feature-ui PROJ-123
    -> plan.md                          -> plan-ui.md
  /execute PROJ-123                   /execute-ui PROJ-123
    -> 5 camadas + testes               -> tela Angular + PO-UI + testes

/code-review                  -> relatório de revisão (API e/ou tela)
/validate                     -> dotnet + npm
/commit                       -> commit convencional com a chave do Jira
```

Atalho de ponta a ponta: `/end-to-end-feature PROJ-123` (API e depois tela).

Regras da esteira:
- **Requisito incompleto não vira plano.** Toda feature passa pelo questionário de descoberta
  (`.claude/references/feature-discovery-questions.md`) na importação. Pergunta que muda schema,
  migration ou contrato da API é **bloqueante**; o resto assume o default do projeto e fica
  **registrado como premissa** no `feature.md`.
- Nenhum código é escrito antes do plano aprovado (`plan.md` / `plan-ui.md`).
- **O contrato da API é a fonte da verdade da tela.** O model TypeScript espelha o DTO; se
  divergirem, quem está errado é o frontend.
- Numa feature fullstack, a **API vem primeiro** — ou, no mínimo, o contrato dela precisa estar
  definido no `plan.md` antes de planejar a tela.
- Nenhuma feature é "concluída" com `dotnet build`, `dotnet test`, `npm run build` ou `npm test`
  falhando.
- Os checklists (`checklist.md` e `checklist-ui.md`) são atualizados a cada etapa concluída.
