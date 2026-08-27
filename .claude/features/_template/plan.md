# Plano — {CHAVE-JIRA} {Título da Feature}

> Gerado por `/plan-feature {CHAVE-JIRA}`. Este arquivo é o contrato de execução: o agente de
> `/execute` só implementa o que estiver aqui.

| | |
|---|---|
| **Feature** | `features/{CHAVE-JIRA}-{slug}/feature.md` |
| **Complexidade** | Baixa / Média / Alta |
| **Camadas afetadas** | Domain, Application, Infrastructure, Api, Testes |
| **Confiança de acerto em uma passada** | {n}/10 |

---

## Resumo da Solução

{2-4 linhas: como a feature será implementada e por quê desta forma.}

---

## CONTEXTO OBRIGATÓRIO

### Ler antes de implementar

- `.claude/references/clean-architecture-dotnet.md` — templates de cada camada
- `.claude/references/dotnet-testing-standards.md` — padrão dos testes
- `{Sln}.Application/Services/{FeatureSimilar}/{X}Service.cs` (linhas {a}-{b}) — padrão a espelhar
- `{Sln}.Api/Controllers/{X}Controller.cs` — padrão de rota/policy
- `{Sln}.Domain/Entities/{X}/{X}Entity.cs` — padrão de mapeamento

### Arquivos a criar

| Arquivo | Camada |
|---|---|
| `{Sln}.Domain/Entities/{Feature}/{Nome}Entity.cs` | Domain |
| `{Sln}.Application/Dtos/{Feature}/{Nome}Dto.cs` | Application |
| `{Sln}.Application/Extensions/{Feature}/{Nome}Extensions.cs` | Application |
| `{Sln}.Application/Services/{Feature}/I{Nome}Service.cs` | Application |
| `{Sln}.Application/Services/{Feature}/{Nome}Service.cs` | Application |
| `{Sln}.Api/Controllers/{Nome}Controller.cs` | Api |
| `{Sln}.TesteUnitario/{Feature}/{Nome}ServiceTests.cs` | Testes |

### Arquivos a modificar

| Arquivo | Alteração |
|---|---|
| `{Sln}.Application/DependencyInjection.cs` | `AddScoped<I{Nome}Service, {Nome}Service>()` |
| `{Sln}.Infrastructure/Data/ApplicationDbContext.cs` | `DbSet` + índices |

---

## TAREFAS PASSO A PASSO

Execute em ordem, de cima para baixo. Cada tarefa é atômica e validável isoladamente.

### T1 — CREATE `{Sln}.Domain/Entities/{Feature}/{Nome}Entity.cs`

- **IMPLEMENTAR**: {campos, tipos, anotações}
- **PADRÃO**: `{Sln}.Domain/Entities/ModelNames/ModelNameEntity.cs`
- **ATENÇÃO**: {snake_case, MaxLength, TypeName de data, auditoria}
- **VALIDAR**: `dotnet build {Sln}.Domain/{Sln}.Domain.csproj`

### T2 — CREATE `{Sln}.Application/Dtos/{Feature}/{Nome}Dto.cs`

- **IMPLEMENTAR**: {campos expostos pela API}
- **PADRÃO**: `{Sln}.Application/Dtos/ModelNames/ModelNameDto.cs`
- **VALIDAR**: `dotnet build {Sln}.Application/{Sln}.Application.csproj`

### T3 — CREATE `{Sln}.Application/Extensions/{Feature}/{Nome}Extensions.cs`

- **IMPLEMENTAR**: `ToDto()` e `ToEntity()` mapeando campo a campo
- **ATENÇÃO**: não mapear campos de auditoria (o service preenche)
- **VALIDAR**: `dotnet build {Sln}.Application/{Sln}.Application.csproj`

### T4 — CREATE `{Sln}.Application/Services/{Feature}/I{Nome}Service.cs`

- **IMPLEMENTAR**: {assinaturas dos métodos}
- **VALIDAR**: `dotnet build {Sln}.Application/{Sln}.Application.csproj`

### T5 — CREATE `{Sln}.Application/Services/{Feature}/{Nome}Service.cs`

- **IMPLEMENTAR**: cada método + as regras RN1..RNn do `feature.md`
- **PADRÃO**: `{Sln}.Application/Services/ModelNames/ModelNameService.cs`
- **ATENÇÃO**: `IUnitOfWork` (nunca `DbContext`); erros com sufixo de status; `CommitAsync` na escrita
- **VALIDAR**: `dotnet build {Sln}.Application/{Sln}.Application.csproj`

### T6 — UPDATE `{Sln}.Application/DependencyInjection.cs`

- **IMPLEMENTAR**: `services.AddScoped<I{Nome}Service, {Nome}Service>();`
- **VALIDAR**: `dotnet build`

### T7 — UPDATE `{Sln}.Infrastructure/Data/ApplicationDbContext.cs`

- **IMPLEMENTAR**: `DbSet` + índice único `uq_{tabela}_{coluna}` no `OnModelCreating`
- **VALIDAR**: `dotnet build`

### T8 — CREATE migration

- **COMANDO**: `dotnet ef migrations add Add{Nome} -p {Sln}.Infrastructure -s {Sln}.Api`
- **VALIDAR**: arquivo gerado em `Migrations/` e `dotnet build`

### T9 — CREATE `{Sln}.Api/Controllers/{Nome}Controller.cs`

- **IMPLEMENTAR**: actions da tabela de endpoints do `feature.md`
- **PADRÃO**: `{Sln}.Api/Controllers/ModelNamesController.cs`
- **ATENÇÃO**: `[ApiController] [ErrorHandlingFilter] [Route("api/{kebab}")]`, policy por action
- **VALIDAR**: `dotnet build`

### T10 — CREATE `{Sln}.TesteUnitario/{Feature}/{Nome}ServiceTests.cs`

- **IMPLEMENTAR**: happy path, não encontrado, cada RN, `Verify` de commit
- **PADRÃO**: `{Sln}.TesteUnitario/AgentVersions/PlatformVersionsServiceTests.cs`
- **VALIDAR**: `dotnet test --filter "FullyQualifiedName~{Nome}ServiceTests"`

---

## ESTRATÉGIA DE TESTE

### Casos obrigatórios

| Método | Cenário | Esperado |
|---|---|---|
| `GetByIdAsync` | id inexistente | `Exception` com sufixo 404 |
| `CreateAsync` | payload válido | DTO com Id + `CommitAsync` uma vez |
| `CreateAsync` | {regra violada} | `Exception` com sufixo {400/409} |

### Casos de borda

- {lista, extraída das regras de negócio e dos limites de campo}

---

## COMANDOS DE VALIDAÇÃO

```bash
# Nível 1 — compilação
dotnet build {Sln}.Api/{Sln}.Api.sln

# Nível 2 — testes da feature
dotnet test {Sln}.Api/{Sln}.Api.sln --filter "FullyQualifiedName~{Nome}ServiceTests"

# Nível 3 — suíte completa (sem regressão)
dotnet test {Sln}.Api/{Sln}.Api.sln

# Nível 4 — formatação
dotnet format {Sln}.Api/{Sln}.Api.sln --verify-no-changes

# Nível 5 — manual (Swagger em Development)
dotnet run --project {Sln}.Api.AppHost
# GET/POST em https://localhost:{porta}/swagger nas rotas da feature
```

---

## CRITÉRIOS DE ACEITE

Copiados do `feature.md` — todos precisam ser verificáveis por um dos comandos acima.

- [ ] CA1: {critério} → verificado por: {teste ou chamada}
- [ ] CA2: {critério} → verificado por: {teste ou chamada}

---

## RISCOS E DECISÕES

| Risco / decisão | Impacto | Como tratar |
|---|---|---|
| {ex.: migration em tabela grande} | {alto} | {rodar fora do pico} |
