---
name: code-review-fix
description: Fixes bugs surfaced by a manual or AI code review, one at a time, with tests, then runs full validation. Use after a code review has produced a list of issues or a review file.
argument-hint: [code-review-file-or-issues] [scope]
---

# Code Review Fix

I ran/performed a code review and found these issues:

Code-review (file or description of issues): $1

Please fix these issues one by one. If the Code-review is a file, read the entire file first to understand all of the issue(s) presented there.

Scope: $2

## Process

Corrija por severidade: críticas primeiro, depois altas, depois o resto.

Para cada correção:
1. Explique o que estava errado e qual o cenário de falha
2. Aplique a correção **no padrão do projeto** (`.claude/references/clean-architecture-dotnet.md`) —
   não introduza padrão novo para consertar um bug
3. Escreva ou ajuste o teste que **falha antes** e passa depois
4. `dotnet build` e o teste específico:
   `dotnet test --filter "FullyQualifiedName~<Nome>ServiceTests"`

## Limites

- Corrija apenas o que a revisão apontou. Achou outro problema? Reporte, não conserte de carona.
- Violação de camada não se corrige com atalho: mover o `DbContext` de lugar não resolve —
  o service volta a usar `IUnitOfWork`.
- Issue média/baixa que o usuário decidir não corrigir vira linha na tabela de divergências do
  `checklist.md`, como dívida consciente.

## Output

Ao fim, rode a skill `validate` e reporte:
- Cada issue: corrigida ✅ / não aplicável ⚪ / deixada como dívida ⏸ (com o motivo)
- Testes adicionados
- Resultado de `dotnet build`, `dotnet test` e `dotnet format`
