---
name: execute-ui
description: Implementa a tela Angular + PO-UI a partir do plan-ui.md, na ordem model → service → rotas → listagem → formulário → testes, compilando entre as etapas e atualizando o checklist da feature. Use quando o plano de UI estiver pronto.
argument-hint: <CHAVE-JIRA> | <caminho do plan-ui.md>
---

# Execute UI — implementar a tela

## Entrada: $ARGUMENTS

- **Chave do Jira** → o plano é `features/<CHAVE>-*/plan-ui.md`.
- **Caminho de arquivo** → use-o diretamente.

Sem `plan-ui.md`, **pare**: rode `/plan-feature-ui <CHAVE>` antes.

## Passo 1 — Carregar contexto

1. Leia o `plan-ui.md` inteiro, depois a seção **Escopo Frontend** do `feature.md`.
2. Leia `CLAUDE.md` e `.claude/references/angular-po-ui.md`.
3. Leia **todos** os arquivos citados em "Contexto obrigatório" — especialmente a tela-espelho.
4. Confira a versão do PO-UI em `package.json`. Componente ou input que você não tem certeza que
   existe nessa versão: confirme em `node_modules/@po-ui/ng-components` antes de usar.
5. Abra `checklist-ui.md`: havendo itens marcados, retome de onde parou.
6. Marque a feature como `⚙️ em execução` em `features/INDEX.md`.

## Passo 2 — Executar na ordem

```
1. model        interface espelhando o DTO da API (camelCase)
2. service      HttpClient tipado, um método por endpoint
3. rotas        <feature>.routes.ts + registro lazy em app.routes.ts
4. listagem     po-page-list + po-table
5. formulário   po-page-edit + reactive forms
6. menu         PoMenuItem, quando a feature precisar
7. testes       service + listagem + formulário
```

Para **cada** tarefa:
1. Abra o arquivo-padrão citado em **PADRÃO** antes de escrever.
2. Implemente exatamente o que está em **IMPLEMENTAR**.
3. Rode o comando de **VALIDAR**. Falhou? Corrija e rode de novo antes de seguir.
4. Marque o item no `checklist-ui.md`.

### Invariantes (verifique enquanto escreve)

| Camada | Invariante |
|---|---|
| Model | `interface`, campos em camelCase idênticos ao DTO; sem lógica; sem `any` |
| Service | `@Injectable({providedIn:'root'})`, `inject(HttpClient)`, URL de `environment`, **nenhum `subscribe`**, sem `catchError` engolindo erro |
| Rotas | feature lazy via `loadComponent`/`loadChildren`, path em kebab-case |
| Componente | `standalone: true`, `imports` granulares dos módulos PO, `ChangeDetectionStrategy.OnPush`, estado em `signal()` |
| Listagem | `PoTableColumn[]` tipado, `p-loading` ligado ao signal, ação destrutiva via `PoDialogService.confirm` |
| Formulário | reactive forms (`fb.nonNullable.group`), validações espelhando as da API, grid `po-row`/`po-md-*` |
| Feedback | sucesso com `PoNotificationService.success`; **erro nunca no componente** — o interceptor notifica |
| Template | novo control flow `@if`/`@for` com `track` |
| Testes | standalone em `imports`, `provideNoopAnimations()`, `http.verify()` no `afterEach` |

### Regras durante a execução

- **Compile entre as etapas**: `npx tsc --noEmit` é rápido e pega quase tudo.
- **Não invente input de componente PO.** Não existe na versão instalada? Use o que existe e
  registre a divergência — não improvise com CSS por cima.
- **Não estilize o que o PO já resolve.** Layout de formulário é `po-row`/`po-md-*`; espaçamento é
  classe utilitária do PO. CSS próprio só para o que o design system não cobre.
- **Não amplie o escopo**: nada de refatorar tela vizinha ou "melhorar" o shared de passagem.
- **Não duplique tratamento de erro.** Se você escreveu `notification.error(...)` num componente,
  provavelmente está duplicando o interceptor — confira.
- Contrato divergiu da API (campo que não existe, formato de resposta diferente)? **Pare e
  reporte.** Não adapte o model para "fazer funcionar" — isso esconde o bug.

## Passo 3 — Testes

Implemente os testes do plano: service (um por método), listagem (carga, erro, confirmação de
exclusão) e formulário (inválido não salva; edição faz `update`, novo faz `create`).

## Passo 4 — Validação final

```bash
npx tsc --noEmit
npm run build
npm test -- --watch=false --browsers=ChromeHeadless
npx ng lint
```

Falhou? Corrija e rode de novo. **Não relate conclusão com comando vermelho.**

Depois, percorra os critérios de aceite de tela do `feature.md` e marque os atendidos.

## Passo 5 — Validação visual (opcional, recomendado)

Com o app rodando (`npm start`), use a skill `agent-browser` para exercitar a tela:
caminho feliz, validação de campo obrigatório, e o erro vindo da API (ex.: SKU duplicado → toast).
Salve os screenshots em `screenshots/<CHAVE>-*.png` e cite os caminhos.

## Passo 6 — Fechar o estado

- `checklist-ui.md`: itens marcados e divergências registradas
- `feature.md`: critérios de aceite de tela marcados
- `INDEX.md`: estado `🔍 em revisão`

## Saída

### Tarefas Concluídas
T1..Tn com ✅/❌ e o arquivo tocado.

### Arquivos
Criados e modificados, com caminho completo.

### Componentes PO Utilizados
Quais, em quais telas, e qualquer input que precisou ser trocado por indisponibilidade na versão.

### Integração com a API
Endpoints consumidos e confirmação de que o model bate com o DTO.

### Testes
Arquivos, número de casos, resultado.

### Validação
Saída de cada comando com veredito ✅/❌.

### Divergências do Plano
Cada desvio, com motivo. "Nenhuma" se for o caso.

### Pronto para
`/code-review` e depois `/commit` — ou o que ainda falta.
