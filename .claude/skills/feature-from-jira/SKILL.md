---
name: feature-from-jira
description: Importa uma issue do Jira e materializa a pasta da feature em features/<CHAVE>-<slug>/ com o requisito destilado em feature.md — escopo de API e de tela — pronto para o /plan-feature e o /plan-feature-ui. Use ao iniciar qualquer trabalho novo que tenha um ticket. Aceita fallback manual quando o MCP do Atlassian está indisponível.
argument-hint: <CHAVE-JIRA> [| texto da issue colado]
---

# Feature from Jira — porta de entrada da esteira

## Entrada

`$ARGUMENTS` — a chave da issue (`PROJ-123`). Múltiplas chaves separadas por vírgula criam uma
pasta por issue.

Se nenhuma chave for informada, pergunte qual é. Se o usuário colou o texto da issue no lugar da
chave, siga direto para o Passo 2 usando esse texto e uma chave provisória (`SEM-JIRA-<slug>`).

## Passo 1 — Buscar a issue

1. `mcp__atlassian__getAccessibleAtlassianResources` → obtém o `cloudId`.
2. `mcp__atlassian__getJiraIssue` com `{ cloudId, issueIdOrKey: "<CHAVE>", responseContentFormat: "markdown" }`.
3. Se a issue tiver `parent` (épico), busque o épico também — o contexto arquitetural costuma
   estar lá, não na story.
4. Se houver `issuelinks` do tipo *blocks/is blocked by*, registre como dependência.

**Se o MCP do Atlassian falhar** (timeout, não configurado, sem permissão): não invente conteúdo.
Avise em uma linha que o Atlassian não respondeu, e peça ao usuário para colar o conteúdo da
issue ou confirmar a criação a partir do `_template/`. A esteira continua normalmente.

## Passo 2 — Destilar o requisito

Leia a issue inteira e **destile** — não copie. O critério: um parágrafo só entra no `feature.md`
se ele muda uma decisão de código.

Extraia, nesta ordem:

1. **Objetivo** — uma frase de negócio.
2. **História de usuário** — reescreva no formato Como/Quero/Para que se a issue não tiver.
3. **Regras de negócio** — numeradas RN1..RNn, cada uma com o **status HTTP de erro esperado**
   quando ela for violada. Regra sem status é regra incompleta.
4. **Critérios de aceite** — verificáveis. "Deve funcionar bem" não é critério; reescreva ou
   registre como pergunta em aberto.
5. **Escopo técnico (API)** — traduza o requisito para as camadas respondendo às 8 perguntas de
   `.claude/references/jira-feature-workflow.md` §"Traduzindo requisito de negócio para camadas":
   endpoints (rota, verbo, policy), modelo de dados (campos, colunas snake_case, tipos,
   obrigatoriedade, índices, auditoria) e impacto por camada.
5b. **Escopo frontend** — se a feature tiver tela, preencha também: telas e rotas, campos com o
   componente PO correspondente, ações do usuário (e quais pedem confirmação), endpoints
   consumidos e critérios de aceite de tela. Anexo com mockup ou protótipo manda no layout —
   registre o link. **Sem tela? Escreva "Sem escopo de tela"** em vez de deixar a seção em branco,
   para não deixar dúvida se foi esquecimento.
6. **Fora de escopo** — o que a issue explicitamente não cobre.
7. **Dependências** e **Perguntas em aberto**.

Antes de escrever, dê uma olhada rápida no código para ancorar o escopo técnico:
- `Services/` para ver se já existe feature parecida a espelhar
- `ApplicationDbContext.cs` para conferir se a tabela/entidade já existe
- `Controllers/` para conferir se a rota já está tomada
- `src/app/features/` e `app.routes.ts` para conferir se a tela ou a rota já existem

Isso evita propor uma entidade que já existe, uma rota de API duplicada ou uma tela que o app já
tem.

## Passo 3 — Criar a pasta da feature

1. Slug: `<CHAVE>-<título em kebab-case, máx. 5 palavras>` → `PROJ-123-cadastro-produtos`.
2. Se a pasta já existir: **não sobrescreva**. Mostre o diff do que mudou no Jira e pergunte se
   é para atualizar o `feature.md` (mantendo `plan.md` e `checklist.md` intactos).
3. Copie `features/_template/feature.md` para `features/<slug>/feature.md` e preencha com a
   destilação do Passo 2.
4. Copie `features/_template/checklist.md` para `features/<slug>/checklist.md`, substituindo
   `{CHAVE-JIRA}`, `{Título}`, `{Feature}` e `{Nome}` pelos valores reais.
   Havendo escopo de tela, copie também `features/_template/checklist-ui.md` para
   `features/<slug>/checklist-ui.md`.
5. Não crie os planos agora — eles são gerados por `/plan-feature` e `/plan-feature-ui`.

## Passo 4 — Registrar no índice

Adicione (ou atualize) a linha da feature em `features/INDEX.md`:

```
| PROJ-123 | Cadastro de Produtos | 📥 importada | [PROJ-123-cadastro-produtos](PROJ-123-cadastro-produtos/) | AAAA-MM-DD |
```

Estado inicial: `📥 importada` — ou `⛔ bloqueada` se houver pergunta em aberto que afete
modelo de dados ou contrato da API.

## Saída

Relate de forma escaneável:

### Feature Importada
- Chave, título, tipo, épico, prioridade
- Caminho da pasta criada

### Requisito em 3 linhas
O que a feature faz, para quem, e qual é a regra mais crítica.

### Escopo Técnico Detectado
- Endpoints (tabela)
- Entidade e tabela nova/alterada
- Camadas afetadas
- Telas e rotas do frontend, com os componentes PO previstos (ou "sem escopo de tela")

### Pendências
- Perguntas em aberto (se houver) e se elas bloqueiam o planejamento
- Dependências de outras features

### Próximo Passo
`/plan-feature <CHAVE>` e, havendo tela, `/plan-feature-ui <CHAVE>` depois — ou, se bloqueada, o
que precisa ser respondido antes.

## Regras

- **Nunca invente requisito.** O que não estiver na issue vira pergunta em aberto, não suposição.
- **Nunca escreva código nesta skill.** Aqui só se produz entendimento.
- Regra de negócio sem status HTTP e critério de aceite não verificável são defeitos de entrada —
  aponte-os explicitamente na saída.
