---
name: end-to-end-feature
description: Desenvolve uma feature de ponta a ponta encadeando a esteira completa — Jira, contexto, plano, execução, revisão e commit. Use quando quiser a construção autônoma a partir de uma chave do Jira ou de uma descrição.
argument-hint: <CHAVE-JIRA> | <descrição da feature>
---

# End-to-End Feature — a esteira completa

**Entrada**: $ARGUMENTS

Encadeia as skills da esteira. Cada etapa só começa quando a anterior termina bem — o resultado
de uma é a entrada da próxima.

```
/feature-from-jira → /prime-backend → /plan-feature → /execute → /code-review → /validate → /commit
```

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

## Etapa 5 — Revisar

Rode a skill `code-review` (`.claude/skills/code-review/SKILL.md`).

Issues **críticas ou altas** voltam para correção antes do commit — corrija e revalide.
Issues médias/baixas podem ser registradas no `checklist.md` como dívida consciente.

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
2. ✅ Contexto carregado
3. ✅ Planejada — `plan.md` (confiança n/10)
4. ✅ Executada — n tarefas, n arquivos
5. ✅ Revisada — n issues (n críticas)
6. ✅ Validada — build ✅ testes ✅ format ✅
7. ✅ Commitada — `<hash>`

**Entregue:**
- Endpoints publicados (método + rota + policy)
- Entidade/tabela e migration criadas
- Testes adicionados e resultado
- Critérios de aceite atendidos: n de n

**Próximos passos:**
- `git push` e abertura do PR
- Atualizar o status da issue no Jira
- Pendências conscientes registradas no `checklist.md`

---

## Regras da esteira

- Uma etapa vermelha **para a esteira**. Nada de seguir para o commit com teste falhando.
- Nenhuma etapa é pulada silenciosamente: se algo for pulado, diga qual e por quê.
- O escopo é o do `feature.md`. Melhoria fora do escopo vira sugestão no relatório, não código.
