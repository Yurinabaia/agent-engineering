---
name: end-to-end-feature
description: Desenvolve uma feature de ponta a ponta encadeando a esteira completa — Jira, contexto, plano e execução da API, plano e execução da tela Angular + PO-UI, revisão e commit. Use quando quiser a construção autônoma a partir de uma chave do Jira ou de uma descrição.
argument-hint: <CHAVE-JIRA> | <descrição da feature>
---

# End-to-End Feature — a esteira completa

**Entrada**: $ARGUMENTS

Encadeia as skills da esteira. Cada etapa só começa quando a anterior termina bem — o resultado
de uma é a entrada da próxima.

```
/feature-from-jira
   → /prime-backend  → /plan-feature    → /execute       (trilha API)
   → /prime-frontend → /plan-feature-ui → /execute-ui    (trilha tela)
   → /code-review → /validate → /commit
```

**Escopo:** leia o `feature.md` e execute só as trilhas que a feature tem.
Sem "Escopo Frontend" preenchido → só a trilha API. Sem "Escopo Técnico" de backend (tela
consumindo API que já existe) → só a trilha de tela. Diga qual decisão você tomou e por quê.

---

## Etapa 1 — Importar a feature

Rode a skill `feature-from-jira` (`.claude/skills/feature-from-jira/SKILL.md`) com `$ARGUMENTS`.

Anote a **chave** e o **caminho da pasta** criada — todas as etapas seguintes dependem deles.

**Porta de saída:** se a feature ficar `⛔ bloqueada` (pergunta em aberto sobre modelo de dados
ou contrato da API), **pare aqui** e apresente as perguntas ao usuário. Não planeje sobre
suposição.

## Etapa 2 — Carregar contexto da API

Rode a skill `prime-backend` (`.claude/skills/prime-backend/SKILL.md`) com a chave da feature.

Pule esta etapa se a sessão já estiver primada (você já leu a solução e a fatia de referência).

## Etapa 3 — Planejar

Rode a skill `plan-feature` (`.claude/skills/plan-feature/SKILL.md`) com a chave.

Gera `features/<CHAVE>-<slug>/plan.md`. Apresente ao usuário o resumo e a **nota de confiança**.
Se a nota for menor que 7/10, exponha o motivo antes de executar.

## Etapa 4 — Executar

Rode a skill `execute` (`.claude/skills/execute/SKILL.md`) com a chave.

Implementa camada a camada, compilando entre elas, e atualiza o `checklist.md`.

## Etapa 4.1 — Contexto do frontend

*Pule as etapas 4.1 a 4.3 se a feature não tiver escopo de tela.*

Rode a skill `prime-frontend` (`.claude/skills/prime-frontend/SKILL.md`).

Pule se a sessão já estiver primada no frontend.

## Etapa 4.2 — Planejar a tela

Rode a skill `plan-feature-ui` (`.claude/skills/plan-feature-ui/SKILL.md`) com a chave.

Gera `features/<CHAVE>-<slug>/plan-ui.md`.

**Porta de saída:** o plano da tela depende do contrato da API. Se a Etapa 4 mudou algum campo ou
rota em relação ao planejado, o `plan-ui.md` precisa refletir isso — confirme os endpoints no
controller real antes de seguir, não no plano da API.

## Etapa 4.3 — Executar a tela

Rode a skill `execute-ui` (`.claude/skills/execute-ui/SKILL.md`) com a chave.

Implementa model, service, rotas, listagem, formulário e testes.

## Etapa 5 — Revisar

Rode a skill `code-review` (`.claude/skills/code-review/SKILL.md`) — ela cobre as duas trilhas.

Numa feature fullstack, os agentes `code-reviewer` (API) e `angular-code-reviewer` (tela) podem
rodar em paralelo, cada um no seu conjunto de arquivos.

Issues **críticas ou altas** voltam para correção antes do commit — corrija e revalide.
Issues médias/baixas podem ser registradas nos checklists como dívida consciente.

## Etapa 6 — Validar

Rode a skill `validate` (`.claude/skills/validate/SKILL.md`) com a chave.

Só siga com **PASS**. Com FAIL, corrija e repita.

## Etapa 7 — Commitar

Rode a skill `commit` (`.claude/skills/commit/SKILL.md`).

Mensagem no formato `feat(<CHAVE>): <descrição>`.

---

## Resumo Final

### Feature Concluída

**Entrada**: $ARGUMENTS
**Chave / Título**: ...
**Pasta**: `features/<CHAVE>-<slug>/`

**Etapas:**
1. ✅ Importada do Jira — `feature.md`
2. ✅ Contexto carregado (API / tela)
3. ✅ API planejada — `plan.md` (confiança n/10)
4. ✅ API executada — n tarefas, n arquivos
5. ✅ Tela planejada — `plan-ui.md` (confiança n/10) — ou ⚪ sem escopo de tela
6. ✅ Tela executada — n tarefas, n arquivos
7. ✅ Revisada — n issues (n críticas)
8. ✅ Validada — dotnet ✅ / npm ✅
9. ✅ Commitada — `<hash>`

**Entregue:**
- Endpoints publicados (método + rota + policy)
- Entidade/tabela e migration criadas
- Telas publicadas (rota + componentes PO usados)
- Testes adicionados (API e tela) e resultado
- Critérios de aceite atendidos: n de n (incluindo os de tela)

**Próximos passos:**
- `git push` e abertura do PR
- Atualizar o status da issue no Jira
- Pendências conscientes registradas no `checklist.md`

---

## Regras da esteira

- Uma etapa vermelha **para a esteira**. Nada de seguir para o commit com teste falhando.
- Nenhuma etapa é pulada silenciosamente: se algo for pulado, diga qual e por quê.
- O escopo é o do `feature.md`. Melhoria fora do escopo vira sugestão no relatório, não código.
- **A API vem antes da tela.** O model TypeScript espelha o DTO real, não o planejado — se a
  execução da API divergiu do plano, a tela segue o código, não o documento.
- Divergência de contrato descoberta na trilha de tela é **parada obrigatória**: reporte, não
  adapte o model para "fazer funcionar".
