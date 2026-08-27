# Harness de IA — API .NET em Clean Architecture

Camada de IA (`.claude/`) preparada para construir APIs .NET em Clean Architecture com agentes,
puxando features direto do Jira. Estrutura de referência: `DTASkills.Api` (.NET 10, PostgreSQL,
EF Core, xUnit).

## O que tem aqui

```
.claude/
  CLAUDE.md                # regras globais: camadas, convenções, contrato de erro, esteira
  PRESENTATION.md          # roteiro da apresentação e da demo ao vivo
  features/                # esteira de features (Jira → plano → código)
  skills/                  # os comandos da esteira (/feature-from-jira, /plan-feature, ...)
  agents/                  # subagentes especializados (analista, implementador, revisor)
  references/              # contexto sob demanda (templates por camada, testes, API, workflow)
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

Depois ajuste em `CLAUDE.md` o prefixo dos projetos (`<Sln>`) e o caminho da solução.
Sem CLAUDE.md ainda? Rode `/create-rules` para derivá-lo do código existente.

## A esteira

```
/feature-from-jira PROJ-123   Jira → features/PROJ-123-slug/feature.md
/plan-feature PROJ-123        feature.md → plan.md (tarefas por camada)
/execute PROJ-123             plan.md → código nas 5 camadas + testes
/code-review                  revisão técnica com relatório
/validate                     dotnet build + test + format
/commit                       feat(PROJ-123): ...
```

Tudo encadeado: `/end-to-end-feature PROJ-123`
Onde parei: `/feature-status`

## Skills principais

| Skill | Para quê |
|---|---|
| `/prime-backend` | carrega a solução, as camadas e uma fatia vertical completa |
| `/feature-from-jira` | importa e destila a issue do Jira |
| `/plan-feature` | plano camada a camada, com padrões e validações |
| `/execute` | implementa o plano, compilando entre as camadas |
| `/code-review` | revisão contra os padrões de Clean Architecture |
| `/code-review-fix` | corrige o que a revisão apontou, com teste |
| `/validate` | portão de qualidade .NET |
| `/commit` | commit convencional com a chave do Jira |
| `/feature-status` | estado da esteira |
| `/spec` | fatia um épico em tickets do tamanho da esteira |
| `/system-review` | revisa o processo (não o código) e evolui esta camada de IA |

## Agentes

| Agente | Papel |
|---|---|
| `jira-feature-analyst` | issue do Jira → requisito destilado por camada |
| `clean-arch-implementer` | plano → código, camada por camada |
| `code-reviewer` | revisão de camada, erro, DI, migration, policy, testes |
| `research-agent` | exploração paralela do código |
| `system-reviewer` | meta-revisão: bugs de processo, não de código |

## Referências

| Arquivo | Conteúdo |
|---|---|
| `clean-architecture-dotnet.md` | template de código de cada camada |
| `dotnet-testing-standards.md` | xUnit + Moq, nomes e cobertura mínima |
| `backend-api-best-practices.md` | rotas, status, validação, paginação, segurança |
| `jira-feature-workflow.md` | mapeamento Jira → camadas, estados da feature |
| `architecture-patterns.md` | material de fundo: padrões arquiteturais e IA |
| `vertical-slice-architecture.md` | material de fundo: fatias verticais |
