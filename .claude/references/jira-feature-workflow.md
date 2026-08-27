# Workflow Jira → Feature → Código

Como uma issue do Jira vira uma pasta de feature e desce pela esteira de agentes.

---

## O ciclo completo

```
   Jira (PROJ-123)
        │  /feature-from-jira PROJ-123
        ▼
   feature.md                               ← o QUE fazer (API + tela)
        │
        ├──────────── trilha API ───────────┬──────── trilha FRONTEND ────────
        │  /plan-feature                    │  /plan-feature-ui
        ▼                                   ▼
   plan.md                             plan-ui.md
        │  /execute                         │  /execute-ui
        ▼                                   ▼
   5 camadas + testes                  tela PO-UI + testes
        │                                   │
        └──────────────┬────────────────────┘
                       │  /code-review  →  /validate
                       ▼
   checklist.md + checklist-ui.md           ← evidência de conclusão
                       │  /commit
                       ▼
   commit "feat(PROJ-123): ..."
```

A trilha da tela depende do **contrato** da API, não do código dela pronto — mas na prática é mais
barato terminar a API primeiro, porque o model TypeScript espelha o DTO real.

Cada seta é uma skill. Cada arquivo é o contrato entre uma etapa e a próxima — o que não estiver
escrito no `feature.md` não chega ao `plan.md`, e o que não estiver no `plan.md` não vira código.

---

## Anatomia da pasta da feature

```
features/
  INDEX.md                       # registro de todas as features e seus estados
  _template/                     # modelos copiados a cada nova feature
    feature.md  plan.md  checklist.md  plan-ui.md  checklist-ui.md
  PROJ-123-cadastro-produtos/
    feature.md                   # requisito destilado do Jira (fonte da verdade)
    plan.md                      # plano da API, camada a camada
    checklist.md                 # progresso das camadas da API
    plan-ui.md                   # plano da tela Angular + PO-UI
    checklist-ui.md              # progresso das etapas da tela
    review.md                    # (opcional) saída do code review
    execution-report.md          # (opcional) relatório pós-execução
```

Nome da pasta: `<CHAVE-JIRA>-<slug-kebab-do-titulo>`, slug com no máximo 5 palavras.

---

## Mapeamento de campos do Jira

O que a skill `feature-from-jira` extrai de `mcp__atlassian__getJiraIssue`:

| Campo Jira | Vai para | Uso |
|---|---|---|
| `key` | cabeçalho + nome da pasta | rastreabilidade, mensagem de commit |
| `summary` | título da feature | slug da pasta |
| `issuetype` | Tipo | Story → feature; Bug → considerar `/rca` antes |
| `status` | Estado | não iniciar se estiver `Done` |
| `priority` | Prioridade | ordem de execução quando há várias |
| `description` | Contexto / Regras de negócio | corpo do `feature.md` |
| Acceptance Criteria / DoD | Critérios de Aceite | vira checklist verificável |
| `parent` (epic) | Épico | contexto arquitetural |
| `issuelinks` | Dependências | features que precisam vir antes |
| `attachment` | Anexos | contratos, exemplos de payload; **mockups mandam no layout da tela** |
| `comment` | Decisões | só as que mudam o requisito |

**Regra de ouro:** o `feature.md` é uma **destilação**, não um dump. Se um parágrafo do Jira não
muda uma decisão de código, ele não entra.

---

## Traduzindo requisito de negócio para camadas

Ao ler o `feature.md`, o agente responde estas perguntas na ordem — é o que gera as tarefas
do plano:

1. **Que dado novo existe?** → entidade(s) em Domain + colunas + migration
2. **O que entra e sai da API?** → DTOs em Application
3. **Como converto um no outro?** → Extensions (mapper)
4. **Que operações o negócio precisa?** → assinaturas na interface do service
5. **Quais são as regras e validações?** → corpo do service + erros com sufixo de status
6. **Quem pode chamar?** → policy do controller (`PortalLogPolicy` / `AdminPolicy`)
7. **Qual a rota e o verbo?** → controller (kebab-case, `{id:int}`)
8. **O que pode dar errado?** → testes de cada caminho de erro

Um critério de aceite que não caiu em nenhuma dessas oito perguntas é um critério **não
implementável** — volte ao Jira e pergunte antes de planejar.

### E se a feature tiver tela

Mais cinco perguntas, respondidas na seção **Escopo Frontend** do `feature.md`:

9. **Quais telas e rotas?** → `features/<f>/pages/` + `<f>.routes.ts` (kebab-case, lazy)
10. **Que campos aparecem e quais são editáveis?** → componente PO de cada campo
    (`po-input`, `po-decimal`, `po-select`, `po-switch`, `po-datepicker`…)
11. **Que ações o usuário tem?** → `PoPageAction[]` / `PoTableAction[]`; quais pedem
    `PoDialogService.confirm`
12. **Quais endpoints a tela consome?** → métodos do service Angular, com o formato de resposta
    **confirmado no controller**
13. **O que o usuário vê quando dá erro?** → nada a implementar por tela: o `errorInterceptor`
    traduz o `ProblemDetails` da API em `PoNotificationService.error`

A pergunta 12 é a que mais causa retrabalho quando é pulada: um model TypeScript que não espelha
o DTO quebra em runtime, não na compilação.

---

## Estados da feature (mantidos no `INDEX.md`)

| Estado | Significado | Próximo comando (API / tela) |
|---|---|---|
| `📥 importada` | `feature.md` criado a partir do Jira | `/plan-feature` / `/plan-feature-ui` |
| `📋 planejada` | plano aprovado | `/execute` / `/execute-ui` |
| `⚙️ em execução` | codificação em andamento | continuar a execução |
| `🔍 em revisão` | código pronto, revisão pendente | `/code-review` |
| `✅ concluída` | build + testes verdes nas duas trilhas, commitada | — |
| `⛔ bloqueada` | falta informação ou dependência | resolver a pendência descrita |

Cada trilha tem seu estado; o estado da feature é o **menos avançado** dos dois.

---

## MCP do Atlassian

As skills usam:

- `mcp__atlassian__getAccessibleAtlassianResources` → obtém o `cloudId` (sempre primeiro)
- `mcp__atlassian__getJiraIssue` → `{ cloudId, issueIdOrKey, responseContentFormat: "markdown" }`
- `mcp__atlassian__searchJiraIssuesUsingJql` → filhos de um épico: `parent = PROJ-42`
- `mcp__atlassian__createJiraIssue` → usada por `/spec` ao fatiar um épico
- `mcp__atlassian__getConfluencePage` → especificação de apoio

**Se o MCP do Atlassian não estiver conectado**, as skills aceitam fallback manual: cole o
conteúdo da issue no prompt ou crie o `feature.md` a partir do `_template/`. A esteira continua
funcionando — só perde a rastreabilidade automática.
