# Harness de IA — API .NET (Clean Architecture) + Angular (PO-UI)

Camada de IA (`.claude/`) preparada para construir features fullstack com agentes, puxando o
requisito direto do Jira:

- **API** em Clean Architecture — referência `DTASkills.Api` (.NET 10, PostgreSQL, EF Core, xUnit)
- **Frontend** Angular 20 standalone + **PO-UI**, consumindo essa API

As duas trilhas compartilham a mesma pasta de feature e o mesmo contrato de erro.

## O que tem aqui

```
.claude/
  CLAUDE.md                # regras globais: camadas, convenções, contrato de erro, esteira
  PRESENTATION.md          # roteiro da apresentação e da demo ao vivo
  features/                # esteira de features (Jira → plano → código), API e tela
  skills/                  # os comandos da esteira (/feature-from-jira, /plan-feature, ...)
  agents/                  # subagentes (analista, implementadores .NET e Angular, revisores)
  references/              # contexto sob demanda (Clean Arch, PO-UI, testes, workflow)
  hooks/                   # garantias determinísticas (segredos, rm -rf, migration à mão)
  settings.json.example    # permissões + hooks
  .mcp.json.example        # MCP do Atlassian (Jira/Confluence)
```

## As cinco primitivas

| Primitiva | Arquivo | O agente... |
|---|---|---|
| **Regras** | `CLAUDE.md` | sempre carrega |
| **Contexto** | `references/*.md` | carrega quando a tarefa pede |
| **Skills** | `skills/*/SKILL.md` | invoca como comando (`/nome`) |
| **Subagentes** | `agents/*.md` | delega em contexto isolado |
| **Hooks** | `hooks/*.py` | não escolhe — dispara sozinho |

> Regra pede. Hook garante.

## Instalação em um projeto

```bash
cp -r .claude/ <seu-projeto>/.claude/
cp <seu-projeto>/.claude/settings.json.example <seu-projeto>/.claude/settings.json
cp <seu-projeto>/.claude/.mcp.json.example <seu-projeto>/.mcp.json
```

Depois ajuste em `CLAUDE.md` o prefixo dos projetos (`<Sln>`), o caminho da solução e, se o
frontend viver em outro repositório, o caminho do app Angular.
Sem CLAUDE.md ainda? Rode `/create-rules` para derivá-lo do código existente.

## A esteira

```
/feature-from-jira PROJ-123   Jira → features/PROJ-123-slug/feature.md   (API + tela)

  trilha API                        trilha FRONTEND
  /plan-feature PROJ-123            /plan-feature-ui PROJ-123
     → plan.md                         → plan-ui.md
  /execute PROJ-123                 /execute-ui PROJ-123
     → 5 camadas + testes              → tela PO-UI + testes

/code-review                  revisão técnica (API e/ou tela)
/validate                     dotnet + npm
/commit                       feat(PROJ-123): ...
```

Tudo encadeado: `/end-to-end-feature PROJ-123`
Onde parei: `/feature-status`

## Skills principais

| Skill | Para quê |
|---|---|
| `/prime-backend` | carrega a solução, as camadas e uma fatia vertical completa |
| `/prime-frontend` | carrega o app Angular, o PO-UI e uma tela completa |
| `/feature-from-jira` | importa e destila a issue do Jira (API + tela) |
| `/plan-feature` | plano da API, camada a camada |
| `/plan-feature-ui` | plano da tela, com os componentes PO escolhidos |
| `/execute` | implementa a API, compilando entre as camadas |
| `/execute-ui` | implementa a tela Angular + PO-UI |
| `/code-review` | revisão contra os padrões das duas trilhas |
| `/code-review-fix` | corrige o que a revisão apontou, com teste |
| `/validate` | portão de qualidade (.NET e Angular) |
| `/commit` | commit convencional com a chave do Jira |
| `/feature-status` | estado da esteira |
| `/spec` | fatia um épico em tickets do tamanho da esteira |
| `/system-review` | revisa o processo (não o código) e evolui esta camada de IA |

## Agentes

| Agente | Papel |
|---|---|
| `jira-feature-analyst` | issue do Jira → requisito destilado (API + tela) |
| `clean-arch-implementer` | plano → código .NET, camada por camada |
| `angular-po-ui-implementer` | plano de tela → código Angular + PO-UI |
| `code-reviewer` | revisão da API: camada, erro, DI, migration, policy, testes |
| `angular-code-reviewer` | revisão da tela: estrutura, tipagem, PO-UI, forms, RxJS |
| `research-agent` | exploração paralela do código |
| `system-reviewer` | meta-revisão: bugs de processo, não de código |

Numa feature fullstack, `clean-arch-implementer` e `angular-po-ui-implementer` são sequenciais
(a tela depende do contrato da API), mas `code-reviewer` e `angular-code-reviewer` rodam em
paralelo.

## Referências

| Arquivo | Conteúdo |
|---|---|
| `clean-architecture-dotnet.md` | template de código de cada camada da API |
| `dotnet-testing-standards.md` | xUnit + Moq, nomes e cobertura mínima |
| `backend-api-best-practices.md` | rotas, status, validação, paginação, segurança |
| `angular-po-ui.md` | estrutura, service, listagem, formulário e catálogo PO |
| `angular-testing-standards.md` | Jasmine/Karma, TestBed, mocks do PO |
| `frontend-component-best-practices.md` | princípios gerais de componente |
| `jira-feature-workflow.md` | mapeamento Jira → camadas, estados da feature |
| `architecture-patterns.md` | material de fundo: padrões arquiteturais e IA |
| `vertical-slice-architecture.md` | material de fundo: fatias verticais |
