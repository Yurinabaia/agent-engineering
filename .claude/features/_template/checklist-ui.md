# Checklist de Tela — {CHAVE-JIRA} {Título}

Atualize **durante** a execução, ao concluir cada etapa.

## Etapa 1 — Model

- [ ] `features/{feature}/models/{x}.model.ts` criado
- [ ] Campos em camelCase, idênticos ao DTO da API
- [ ] Sem `any`
- [ ] `npx tsc --noEmit` ✅

## Etapa 2 — Service

- [ ] `features/{feature}/services/{x}.service.ts` criado
- [ ] Um método por endpoint do contrato
- [ ] `inject(HttpClient)`, URL de `environment`
- [ ] Sem `subscribe` e sem `catchError` engolindo erro
- [ ] `npx tsc --noEmit` ✅

## Etapa 3 — Rotas

- [ ] `{feature}.routes.ts` com `''`, `'novo'`, `':id'`
- [ ] Registro lazy em `app.routes.ts`
- [ ] Guard aplicado, se a feature exigir
- [ ] `npm run build` ✅

## Etapa 4 — Listagem

- [ ] `pages/{x}-list/` com `.ts`, `.html`, `.css`
- [ ] `po-page-list` + `po-table`
- [ ] `PoTableColumn[]` tipado, com `type`/`format` corretos
- [ ] `PoPageAction[]` e `PoTableAction[]`
- [ ] Filtro (`PoPageFilter`), se pedido
- [ ] Exclusão com `PoDialogService.confirm`
- [ ] `OnPush` + `signal()` + `p-loading`
- [ ] `npm run build` ✅

## Etapa 5 — Formulário

- [ ] `pages/{x}-form/` com `.ts`, `.html`, `.css`
- [ ] `po-page-edit` + reactive forms (`fb.nonNullable.group`)
- [ ] Componente PO correto para cada campo
- [ ] Validações espelhando as regras da API
- [ ] Modo novo e modo edição pela rota
- [ ] `p-disable-submit` durante o salvamento
- [ ] Sucesso com `PoNotificationService.success`
- [ ] `npm run build` ✅

## Etapa 6 — Navegação

- [ ] `PoMenuItem` adicionado (se aplicável)
- [ ] Breadcrumb configurado (se aplicável)

## Etapa 7 — Testes

- [ ] `{x}.service.spec.ts` — um caso por método
- [ ] `{x}-list.component.spec.ts` — carga, erro, confirmação de exclusão
- [ ] `{x}-form.component.spec.ts` — inválido não salva; edição faz update
- [ ] `provideNoopAnimations()` e `http.verify()` presentes
- [ ] `npm test -- --watch=false --browsers=ChromeHeadless` ✅

## Fechamento

- [ ] Critérios de aceite de tela do `feature.md` marcados
- [ ] `npx tsc --noEmit`, `npm run build`, `npm test`, `npx ng lint` verdes
- [ ] Validação visual feita (screenshots em `screenshots/{CHAVE}-*.png`)
- [ ] `/code-review` sem issue crítica ou alta pendente
- [ ] `INDEX.md` atualizado
- [ ] Commit `feat({CHAVE-JIRA}): ...`

## Divergências do Plano

| Item do plano | O que foi feito | Motivo |
|---|---|---|
| | | |
