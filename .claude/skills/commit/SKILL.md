---
name: commit
description: Cria um commit das alterações pendentes com mensagem convencional e rastreabilidade da issue do Jira. Use quando o trabalho estiver validado e pronto.
argument-hint: [CHAVE-JIRA]
---

# Commit — registrar o trabalho

## Processo

1. Levante o que está pendente:
   ```bash
   git status && git diff HEAD && git status --porcelain
   ```
2. Identifique a feature: use `$ARGUMENTS`, ou o nome do branch, ou a pasta em `features/`
   alterada no diff.
3. Confira que o portão passou. Se `dotnet build` ou `dotnet test` não foram rodados após a
   última alteração, rode-os agora. **Não commite com build ou teste vermelho** — se o usuário
   pedir mesmo assim, deixe isso explícito na resposta.
4. Adicione os arquivos novos e alterados — inclusive os artefatos da feature
   (`feature.md`, `plan.md`, `checklist.md`) e a migration gerada.
5. Escreva a mensagem:

   ```
   <tipo>(<CHAVE-JIRA>): <descrição atômica no imperativo>
   ```

   Tipos: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`.
   Sem chave do Jira, omita os parênteses: `feat: <descrição>`.

   Exemplos:
   ```
   feat(DEMO-101): adiciona CRUD de produtos com SKU único e auditoria
   fix(DEMO-104): corrige status 409 na duplicidade de SKU
   test(DEMO-101): cobre regras de preço e produto inexistente
   ```

6. Um commit por unidade coerente. Se o diff mistura duas features, separe.

## Saída

Depois do commit, imprima:

### O Que Mudou
3 a 6 frases: qual problema a feature resolve, quais camadas foram tocadas e quais os arquivos
principais. Escreva para quem vai ler o `git log` daqui a seis meses.

### Endpoints e Schema
- Rotas publicadas ou alteradas (método + rota + policy)
- Migration incluída no commit (nome), se houver

### Mudanças na Camada de IA
Só inclua se algo em `.claude/` mudou (CLAUDE.md, `references/`, `skills/`, `agents/`,
`features/`). Liste cada arquivo com uma linha sobre o que evoluiu e por quê. Se nada mudou,
omita a seção inteira.

### Estado da Feature
Atualize `features/INDEX.md` para `✅ concluída` quando o commit fecha a feature, e diga isso
aqui. Se ficou pendência, diga qual e qual o próximo comando.
