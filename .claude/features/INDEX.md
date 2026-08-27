# Índice de Features

Registro de todas as features da esteira. Atualizado pelas skills `/feature-from-jira`,
`/execute` e `/commit` — e consultável com `/feature-status`.

| Chave | Título | Escopo | API | Tela | Estado | Pasta | Atualizado |
|---|---|---|---|---|---|---|---|
| DEMO-101 | Cadastro de Produtos | API + tela | 📥 | 📥 | 📥 importada | [DEMO-101-cadastro-produtos](DEMO-101-cadastro-produtos/) | 2026-08-27 |

## Legenda de estados

| Estado | Significado | Próximo comando |
|---|---|---|
| 📥 importada | `feature.md` criado a partir do Jira | `/plan-feature <CHAVE>` |
| 📋 planejada | `plan.md` gerado e aprovado | `/execute <CHAVE>` |
| ⚙️ em execução | codificação em andamento | continuar `/execute <CHAVE>` |
| 🔍 em revisão | código pronto, revisão pendente | `/code-review` |
| ✅ concluída | build + testes verdes, commitada | — |
| ⛔ bloqueada | falta informação ou dependência | resolver a pendência do `feature.md` |

As colunas **API** e **Tela** mostram o estado de cada trilha; o estado da feature é o **menos
avançado** dos dois. Feature sem escopo de tela usa `—` na coluna Tela.

Comandos por trilha:

| Trilha | Planejar | Executar |
|---|---|---|
| API | `/plan-feature <CHAVE>` | `/execute <CHAVE>` |
| Tela | `/plan-feature-ui <CHAVE>` | `/execute-ui <CHAVE>` |
