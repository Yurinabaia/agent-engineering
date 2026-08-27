---
name: prime-backend
description: Carrega o contexto da API .NET em Clean Architecture — solução, camadas, uma feature completa de ponta a ponta, persistência e autenticação — sem puxar o repositório inteiro. Use no início de qualquer sessão de trabalho na API. Aceita chaves do Jira e páginas do Confluence para ancorar o contexto na tarefa.
argument-hint: [chaves-jira] [ids-confluence]
---

# Prime Backend — contexto da API .NET

## Objetivo

Entender a solução o suficiente para planejar e implementar sem reler o repositório inteiro.
**Disciplina de escopo:** leia uma fatia vertical completa, não todos os arquivos de cada camada.

## Passo 0 — Contexto externo (opcional)

Argumentos: `[chaves-jira] [ids-confluence]`.

**Chaves do Jira** (`PROJ-12` ou `PROJ-12,PROJ-13`):
1. `mcp__atlassian__getAccessibleAtlassianResources` → `cloudId`.
2. `mcp__atlassian__getJiraIssue` com `{ cloudId, issueIdOrKey, responseContentFormat: "markdown" }`.

**Ids do Confluence** (numéricos): `mcp__atlassian__getConfluencePage` com
`contentFormat: "markdown"`.

Sem argumentos, pule para o Passo 1. Se o MCP falhar, diga em uma linha e siga — não invente
conteúdo de ticket.

Resuma o contexto externo em 3 linhas antes de continuar: é ele que enquadra o resto do priming.

## Passo 1 — Mapear a solução

```bash
find . -name "*.sln" -maxdepth 3
find . -name "*.csproj" -not -path "*/bin/*" -not -path "*/obj/*"
git ls-files | head -100
```

Identifique os projetos e confirme a divisão em camadas: `.Api`, `.Application`, `.Domain`,
`.Infrastructure`, `.Contracts`, `.TesteUnitario` (+ `.AppHost` / `.ServiceDefaults` se houver
Aspire).

Leia os `.csproj` de Api, Application, Infrastructure e Domain — as `ProjectReference` são a
**prova** da direção das dependências. Se a realidade divergir do `CLAUDE.md`, reporte.

## Passo 2 — Ler as regras

- `CLAUDE.md`
- `.claude/references/clean-architecture-dotnet.md`
- `.claude/references/backend-api-best-practices.md`
- `.claude/references/dotnet-testing-standards.md`

## Passo 3 — Ler uma fatia vertical completa

Escolha a feature mais simples e leia **todos** os arquivos dela, em ordem:

1. `<Sln>.Domain/Entities/<Feature>/<X>Entity.cs`
2. `<Sln>.Application/Dtos/<Feature>/<X>Dto.cs`
3. `<Sln>.Application/Extensions/<Feature>/<X>Extensions.cs`
4. `<Sln>.Application/Services/<Feature>/I<X>Service.cs` e `<X>Service.cs`
5. `<Sln>.Api/Controllers/<X>Controller.cs`
6. `<Sln>.TesteUnitario/<Feature>/<X>ServiceTests.cs`

Esta fatia é o molde de tudo que você vai escrever depois.

## Passo 4 — Ler a infraestrutura transversal

- `<Sln>.Api/Program.cs` — composition root, middlewares, ordem do pipeline
- `<Sln>.Application/DependencyInjection.cs` e `<Sln>.Infrastructure/DependencyInjection.cs`
- `<Sln>.Infrastructure/Data/ApplicationDbContext.cs` — DbSets, índices, relacionamentos
- `<Sln>.Application/Common/Interfaces/Persistence/IRepository.cs` e `IUnitOfWork.cs`
- `<Sln>.Api/Filter/ErrorHandlingFilterAttribute.cs` e `<Sln>.Application/Erros/ErrosException.cs`
- Policies de autorização (em `AddAuth`, na Infrastructure)

## Passo 5 — Estado atual

```bash
git log -10 --oneline
git status
ls <Sln>.Infrastructure/Migrations | tail -5
```

Note migrations recentes, mudanças de schema em andamento e features abertas em
`features/INDEX.md`.

## Relatório

### Contexto Externo (se houver)
Issue(s) e página(s): chave, título, objetivo em uma linha, critérios de aceite.

### Solução
- Nome, versão do .NET, banco e ORM
- Projetos e o papel de cada um
- Aspire presente? health checks?

### Regra de Dependência (verificada)
Grafo real extraído dos `.csproj`, com ⚠️ em qualquer referência que viole Clean Architecture.

### Fatia de Referência
Nome da feature lida e os 6 caminhos, para servir de molde.

### Padrões Internalizados
- Nomenclatura (Entity/Dto/Contract, `_camelCase`, kebab-case nas rotas, snake_case no banco)
- Persistência (`IUnitOfWork.Repository<T>()`, `asNoTracking`, `CommitAsync`)
- Erros (sufixo de status → `ProblemDetails`)
- Autorização (`PortalLogPolicy` / `AdminPolicy`)
- Testes (xUnit + Moq, nomes em português)

### Estado Atual
Branch, commits recentes, migrations pendentes, features em andamento e observações
(validação faltando, endpoint sem policy, entidade sem migration).

Use bullets e títulos — o relatório precisa ser escaneável.
