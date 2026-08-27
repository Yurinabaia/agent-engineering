# Plano de Tela — {CHAVE-JIRA} {Título da Feature}

> Gerado por `/plan-feature-ui {CHAVE-JIRA}`. Contrato de execução da camada Angular: o agente de
> `/execute-ui` só implementa o que estiver aqui.

| | |
|---|---|
| **Feature** | `features/{CHAVE-JIRA}-{slug}/feature.md` |
| **Plano da API** | `features/{CHAVE-JIRA}-{slug}/plan.md` (ou "API já existente") |
| **Complexidade** | Baixa / Média / Alta |
| **Angular / PO-UI** | {versões lidas do package.json} |
| **Confiança de acerto em uma passada** | {n}/10 |

---

## Resumo da Solução

{2-4 linhas: quais telas, quais componentes PO e por quê.}

---

## CONTEXTO OBRIGATÓRIO

### Ler antes de implementar

- `.claude/references/angular-po-ui.md` — templates de model, service, listagem e formulário
- `.claude/references/angular-testing-standards.md` — padrão dos specs
- `src/app/features/{FeatureEspelho}/` — tela existente a espelhar (caminhos exatos)
- `src/app/core/interceptors/error.interceptor.ts` — como o erro da API vira toast
- `{Sln}.Api/Controllers/{X}Controller.cs` e `{Sln}.Application/Dtos/{Feature}/{X}Dto.cs` —
  contrato real consumido

### Arquivos a criar

| Arquivo | Papel |
|---|---|
| `src/app/features/{feature}/models/{x}.model.ts` | interface espelhando o DTO |
| `src/app/features/{feature}/services/{x}.service.ts` | chamadas HTTP |
| `src/app/features/{feature}/{feature}.routes.ts` | rotas lazy |
| `src/app/features/{feature}/pages/{x}-list/{x}-list.component.ts\|html\|css` | listagem |
| `src/app/features/{feature}/pages/{x}-form/{x}-form.component.ts\|html\|css` | formulário |
| `.../{x}.service.spec.ts`, `.../{x}-list.component.spec.ts`, `.../{x}-form.component.spec.ts` | testes |

### Arquivos a modificar

| Arquivo | Alteração |
|---|---|
| `src/app/app.routes.ts` | rota lazy `{feature}` |
| `src/app/app.ts` (menu) | novo `PoMenuItem`, se aplicável |

---

## CONTRATO DA API (confirmado, não presumido)

| Endpoint | Verbo | Policy | Resposta | Método do service |
|---|---|---|---|---|
| `api/{recurso}` | GET | PortalLogPolicy | `{X}Dto[]` | `getAll()` |
| `api/{recurso}/{id:int}` | GET | PortalLogPolicy | `{X}Dto` | `getById(id)` |
| `api/{recurso}` | POST | AdminPolicy | `{X}Dto` (201) | `create(dto)` |
| `api/{recurso}/{id:int}` | PUT | AdminPolicy | `{X}Dto` | `update(id, dto)` |
| `api/{recurso}/{id:int}` | DELETE | AdminPolicy | 204 | `delete(id)` |

Formato da listagem: {array puro / `ResponseContract` / `PagedResponseContract`} — **confirmado em
{arquivo}**.

---

## MAPA DE CAMPOS

| Campo (DTO) | Model TS | Componente PO | Validação | Coluna na tabela |
|---|---|---|---|---|
| `Sku` | `sku: string` | `po-input` `p-maxlength="30"` | required, maxLength(30) | `{ property:'sku', label:'SKU' }` |
| `Preco` | `preco: number` | `po-decimal` `p-decimals-length="2"` | required, min(0.01) | `type:'currency', format:'BRL'` |
| `Ativo` | `ativo: boolean` | `po-switch` | — | `type:'boolean'` + labels |

---

## MAPA DE ERROS

| Erro da API | Status | O que o usuário vê |
|---|---|---|
| {mensagem} | 409 | toast de erro com o `title` do ProblemDetails (via interceptor) |
| {mensagem} | 404 | toast de erro + volta para a listagem |
| conexão indisponível | 0 | "Não foi possível conectar ao servidor." |

Nenhum `notification.error` no componente — tudo passa pelo interceptor.

---

## TAREFAS PASSO A PASSO

### T1 — CREATE `models/{x}.model.ts`
- **IMPLEMENTAR**: interface com os campos do mapa acima, em camelCase
- **ATENÇÃO**: sem `any`; opcional só onde o DTO permite nulo
- **VALIDAR**: `npx tsc --noEmit`

### T2 — CREATE `services/{x}.service.ts`
- **IMPLEMENTAR**: um método por endpoint da tabela de contrato
- **PADRÃO**: `src/app/features/{FeatureEspelho}/services/{y}.service.ts`
- **ATENÇÃO**: URL de `environment`; sem `subscribe`; sem `catchError`
- **VALIDAR**: `npx tsc --noEmit`

### T3 — CREATE `{feature}.routes.ts` + UPDATE `app.routes.ts`
- **IMPLEMENTAR**: rotas `''`, `'novo'`, `':id'` com `loadComponent`; registro lazy na raiz
- **VALIDAR**: `npm run build`

### T4 — CREATE `pages/{x}-list/`
- **IMPLEMENTAR**: `po-page-list` + `po-table`; colunas, `PoPageAction[]`, `PoTableAction[]`,
  filtro, exclusão com `PoDialogService.confirm`
- **PADRÃO**: {arquivo espelho}
- **ATENÇÃO**: `OnPush` + `signal()`; `p-loading`; sem toast de erro no componente
- **VALIDAR**: `npm run build`

### T5 — CREATE `pages/{x}-form/`
- **IMPLEMENTAR**: `po-page-edit` + reactive forms; modo novo e modo edição pela rota
- **ATENÇÃO**: `p-disable-submit` durante o salvamento; validações espelhando a API
- **VALIDAR**: `npm run build`

### T6 — UPDATE menu (`PoMenuItem`)
- **IMPLEMENTAR**: item apontando para a rota da feature
- **VALIDAR**: `npm run build`

### T7 — CREATE testes
- **IMPLEMENTAR**: service (um por método), listagem (carga, erro, confirmação), formulário
  (inválido não salva; edição faz update)
- **PADRÃO**: `.claude/references/angular-testing-standards.md`
- **VALIDAR**: `npm test -- --watch=false --browsers=ChromeHeadless`

---

## COMANDOS DE VALIDAÇÃO

```bash
npx tsc --noEmit                                        # tipos
npm run build                                           # build de produção
npm test -- --watch=false --browsers=ChromeHeadless     # testes
npx ng lint                                             # lint
npm start                                               # validação manual em http://localhost:4200
```

---

## CRITÉRIOS DE ACEITE DE TELA

- [ ] CA{n}: {critério} → verificado por: {teste ou passo manual}

---

## RISCOS E DECISÕES

| Risco / decisão | Impacto | Como tratar |
|---|---|---|
| {ex.: `po-page-dynamic-table` exige `items`/`hasNext`} | alto | usar `po-page-list` + `po-table` |
