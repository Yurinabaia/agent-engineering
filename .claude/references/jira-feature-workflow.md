# Workflow Jira → Feature → Código

Como uma issue do Jira vira uma pasta de feature e desce pela esteira de agentes.

---

## O ciclo completo

```
   Jira (PROJ-123)
        │  /feature-from-jira PROJ-123
        ▼
   features/PROJ-123-<slug>/feature.md      ← o QUE fazer (requisito destilado)
        │  /plan-feature PROJ-123
        ▼
   features/PROJ-123-<slug>/plan.md         ← COMO fazer (tarefas por camada)
        │  /execute PROJ-123
        ▼
   código nas 5 camadas + testes            ← o agente codifica
        │  /code-review  →  /validate
        ▼
   features/PROJ-123-<slug>/checklist.md    ← evidência de conclusão
        │  /commit
        ▼
   commit "feat(PROJ-123): ..."
```

Cada seta é uma skill. Cada arquivo é o contrato entre uma etapa e a próxima — o que não estiver
escrito no `feature.md` não chega ao `plan.md`, e o que não estiver no `plan.md` não vira código.

---

## Anatomia da pasta da feature

```
features/
  INDEX.md                       # registro de todas as features e seus estados
  _template/                     # modelos copiados a cada nova feature
    feature.md
    plan.md
    checklist.md
  PROJ-123-cadastro-produtos/
    feature.md                   # requisito destilado do Jira (fonte da verdade)
    plan.md                      # plano de implementação por camada
    checklist.md                 # progresso camada a camada
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
| `attachment` | Anexos | contratos, mockups, exemplos de payload |
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

---

## Estados da feature (mantidos no `INDEX.md`)

| Estado | Significado | Próximo comando |
|---|---|---|
| `📥 importada` | `feature.md` criado a partir do Jira | `/plan-feature <CHAVE>` |
| `📋 planejada` | `plan.md` aprovado | `/execute <CHAVE>` |
| `⚙️ em execução` | codificação em andamento | continuar `/execute` |
| `🔍 em revisão` | código pronto, revisão pendente | `/code-review` |
| `✅ concluída` | build + testes verdes, commitada | — |
| `⛔ bloqueada` | falta informação ou dependência | resolver a pendência descrita |

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
