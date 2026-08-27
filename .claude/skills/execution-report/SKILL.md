---
name: execution-report
description: Gera o relatório estruturado de uma feature recém-implementada — o que foi feito, divergências do plano e dificuldades — como entrada para a revisão de sistema. Use logo após concluir a execução.
argument-hint: <CHAVE-JIRA>
---

# Execution Report

Review and deeply analyze the implementation you just completed.

## Context

You have just finished implementing a feature. Before moving on, reflect on:

- What you implemented
- How it aligns with the plan
- What challenges you encountered
- What diverged and why

## Generate Report

Salve em: `features/<CHAVE>-<slug>/execution-report.md`

### Meta Information

- Feature: `features/<CHAVE>-<slug>/feature.md`
- Plan file: `features/<CHAVE>-<slug>/plan.md`
- Files added: [list with paths]
- Files modified: [list with paths]
- Lines changed: +X -Y

### Validation Results

- `dotnet build`: ✓/✗ [erros, se houver]
- `dotnet test`: ✓/✗ [X passaram, Y falharam]
- `dotnet format --verify-no-changes`: ✓/✗
- Migration gerada: ✓/✗/n-a [nome]
- Critérios de aceite atendidos: X de Y

### Layer Coverage

| Camada | Tocada | Observação |
|---|---|---|
| Domain | sim/não | |
| Application (DTO/mapper/service/DI) | sim/não | |
| Infrastructure (DbSet/índice/migration) | sim/não | |
| Api (controller/policies) | sim/não | |
| Testes | sim/não | |

### What Went Well

List specific things that worked smoothly:

- [concrete examples]

### Challenges Encountered

List specific difficulties:

- [what was difficult and why]

### Divergences from Plan

For each divergence, document:

**[Divergence Title]**

- Planned: [what the plan specified]
- Actual: [what was implemented instead]
- Reason: [why this divergence occurred]
- Type: [Better approach found | Plan assumption wrong | Security concern | Performance issue | Other]

### Skipped Items

List anything from the plan that was not implemented:

- [what was skipped]
- Reason: [why it was skipped]

### Recommendations

Based on this implementation, what should change for next time?

- Plan skill improvements: [suggestions]
- Execute skill improvements: [suggestions]
- CLAUDE.md additions: [suggestions]
- `.claude/references/clean-architecture-dotnet.md`: [template ausente ou incorreto]
- `features/_template/`: [campo que o requisito sempre precisa e o template não pede]

Copie as divergências para a tabela "Divergências do Plano" do `checklist.md` da feature — é ela
que o `/system-review` lê.
