# features/ — Esteira de Features

Cada feature da API vive aqui como uma pasta, do requisito no Jira até o commit.
Esta pasta é o **estado compartilhado entre as skills e os agentes**: uma etapa escreve, a
seguinte lê.

## Estrutura

```
features/
  INDEX.md                          # registro de todas as features + estado atual
  _template/                        # modelos copiados a cada nova feature
    feature.md                      #   requisito destilado do Jira
    plan.md                         #   plano de implementação por camada
    checklist.md                    #   progresso camada a camada
  PROJ-123-cadastro-produtos/       # uma pasta por feature
    feature.md
    plan.md
    checklist.md
    review.md                       # opcional — saída do /code-review
    execution-report.md             # opcional — saída do /execution-report
```

## Como usar

```bash
/feature-from-jira PROJ-123   # Jira  → features/PROJ-123-slug/feature.md
/plan-feature PROJ-123        # feature.md → plan.md
/execute PROJ-123             # plan.md → código nas 5 camadas + testes
/code-review                  # revisão do que foi escrito
/validate                     # dotnet build + test + format
/commit                       # commit "feat(PROJ-123): ..."
```

Tudo de uma vez: `/end-to-end-feature PROJ-123`

Sem acesso ao Jira? `/feature-from-jira` aceita o texto da issue colado no prompt, ou copie
`_template/feature.md` à mão. A esteira segue igual daí em diante.

## Regras

1. **Uma feature = uma pasta = uma branch.** Nada de duas features na mesma pasta.
2. **O `feature.md` é a fonte da verdade do requisito.** Se o Jira mudar, reimporte — não edite
   o plano por fora.
3. **Nenhum código antes do `plan.md`.** O plano é o contrato de execução.
4. **O `checklist.md` é atualizado durante a execução**, não no fim — é ele que permite retomar
   uma feature interrompida.
5. **Feature só é `✅ concluída` com `dotnet build` e `dotnet test` verdes.**

## Referências

- `.claude/references/jira-feature-workflow.md` — mapeamento de campos do Jira e estados
- `.claude/references/clean-architecture-dotnet.md` — templates de código por camada
- `.claude/references/dotnet-testing-standards.md` — padrões de teste
