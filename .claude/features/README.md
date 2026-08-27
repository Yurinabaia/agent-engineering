# features/ — Esteira de Features

Cada feature da API vive aqui como uma pasta, do requisito no Jira até o commit.
Esta pasta é o **estado compartilhado entre as skills e os agentes**: uma etapa escreve, a
seguinte lê.

## Estrutura

```
features/
  INDEX.md                          # registro de todas as features + estado atual
  _template/                        # modelos copiados a cada nova feature
    feature.md                      #   requisito destilado do Jira (API + tela)
    plan.md                         #   plano da API, camada a camada
    checklist.md                    #   progresso das camadas da API
    plan-ui.md                      #   plano da tela Angular + PO-UI
    checklist-ui.md                 #   progresso das etapas da tela
  PROJ-123-cadastro-produtos/       # uma pasta por feature
    feature.md                      #   requisito (as duas trilhas)
    plan.md        checklist.md     #   trilha API
    plan-ui.md     checklist-ui.md  #   trilha frontend
    review.md                       # opcional — saída do /code-review
    execution-report.md             # opcional — saída do /execution-report
```

Uma feature só de backend não tem `plan-ui.md`/`checklist-ui.md` — e o `feature.md` diz
"Sem escopo de tela" para deixar claro que foi decisão, não esquecimento.

## Como usar

```bash
/feature-from-jira PROJ-123   # Jira → feature.md (escopo de API e de tela)

# trilha API
/plan-feature PROJ-123        # feature.md → plan.md
/execute PROJ-123             # plan.md → código nas 5 camadas + testes

# trilha frontend
/plan-feature-ui PROJ-123     # feature.md + contrato da API → plan-ui.md
/execute-ui PROJ-123          # plan-ui.md → tela Angular + PO-UI + testes

/code-review                  # revisão do que foi escrito (as duas trilhas)
/validate                     # dotnet + npm
/commit                       # commit "feat(PROJ-123): ..."
```

Tudo de uma vez: `/end-to-end-feature PROJ-123`

Sem acesso ao Jira? `/feature-from-jira` aceita o texto da issue colado no prompt, ou copie
`_template/feature.md` à mão. A esteira segue igual daí em diante.

## Regras

1. **Uma feature = uma pasta = uma branch.** Nada de duas features na mesma pasta.
2. **O `feature.md` é a fonte da verdade do requisito.** Se o Jira mudar, reimporte — não edite
   o plano por fora.
3. **Nenhum código antes do plano.** `plan.md` para a API, `plan-ui.md` para a tela.
4. **A API vem antes da tela.** O model TypeScript espelha o DTO real; se divergirem, quem está
   errado é o frontend.
5. **Os checklists são atualizados durante a execução**, não no fim — são eles que permitem
   retomar uma feature interrompida.
6. **Feature só é `✅ concluída` com as duas trilhas verdes** (`dotnet build`/`dotnet test` e
   `npm run build`/`npm test`).

## Referências

- `.claude/references/jira-feature-workflow.md` — mapeamento de campos do Jira e estados
- `.claude/references/clean-architecture-dotnet.md` — templates de código por camada (API)
- `.claude/references/dotnet-testing-standards.md` — padrões de teste da API
- `.claude/references/angular-po-ui.md` — templates de tela com PO-UI
- `.claude/references/angular-testing-standards.md` — padrões de teste da tela
