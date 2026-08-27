---
name: prime-frontend
description: Carrega o contexto do frontend Angular com PO-UI — estrutura de features, services HTTP, interceptors, rotas lazy e uma tela completa de ponta a ponta — sem puxar o backend junto. Use no início de qualquer sessão de trabalho em tela. Aceita chaves do Jira e páginas do Confluence para ancorar o contexto na tarefa.
argument-hint: [chaves-jira] [ids-confluence]
---

# Prime Frontend — contexto do Angular + PO-UI

## Objetivo

Entender o app o suficiente para criar uma tela nova no padrão do time.
**Disciplina de escopo:** leia **uma** feature completa, não todas as telas do app.

## Passo 0 — Contexto externo (opcional)

Argumentos: `[chaves-jira] [ids-confluence]`.

**Jira** (`PROJ-12` ou `PROJ-12,PROJ-13`):
1. `mcp__atlassian__getAccessibleAtlassianResources` → `cloudId`
2. `mcp__atlassian__getJiraIssue` com `{ cloudId, issueIdOrKey, responseContentFormat: "markdown" }`

**Confluence** (ids numéricos): `mcp__atlassian__getConfluencePage` com `contentFormat: "markdown"`.
Protótipos e mockups anexados são contexto de primeira classe aqui — leia antes de olhar código.

Sem argumentos, pule para o Passo 1. MCP fora do ar? Diga em uma linha e siga.

## Passo 1 — Versões e dependências

```bash
cat package.json
cat angular.json | head -60
```

Registre:
- Versão do **Angular** (17+ → control flow novo; 19+ → standalone por padrão)
- Versão do **PO-UI** (`@po-ui/ng-components`, `@po-ui/ng-templates`, `@po-ui/style`)
- Tema do PO carregado em `angular.json` → `styles`
- Runner de teste (Karma/Jasmine ou Jest) e scripts de `build`, `test`, `lint`
- Gerenciador de pacote pelo lockfile (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`)

> Nunca assuma a API de um componente PO pela memória. Na dúvida, confira
> `node_modules/@po-ui/ng-components` ou <https://po-ui.io> para a versão instalada.

## Passo 2 — Ler as regras

- `CLAUDE.md`
- `.claude/references/angular-po-ui.md`
- `.claude/references/angular-testing-standards.md`
- `.claude/references/frontend-component-best-practices.md`

## Passo 3 — Bootstrap e infra transversal

- `src/app/app.config.ts` — providers: `provideRouter`, `provideHttpClient(withInterceptors(...))`,
  **`provideAnimations()`** (exigido pelo PO-UI)
- `src/app/app.routes.ts` — rotas raiz e lazy loading
- `src/app/app.ts` / `app.html` — shell (`po-toolbar`, `po-menu`)
- `src/app/core/interceptors/` — auth e erro. **Leia o de erro com atenção**: é ele que traduz o
  `ProblemDetails` da API em `PoNotificationService`
- `src/environments/` — `apiUrl` de cada ambiente

## Passo 4 — Ler uma feature completa

Escolha a feature mais simples em `src/app/features/` e leia **tudo** dela:

1. `models/<x>.model.ts`
2. `services/<x>.service.ts`
3. `pages/<x>-list/<x>-list.component.ts` + `.html`
4. `pages/<x>-form/<x>-form.component.ts` + `.html`
5. `<x>.routes.ts`
6. um `.spec.ts`

Esta fatia é o molde de tudo que você vai escrever depois. Anote quais **módulos PO** o projeto
importa (granulares como `PoTableModule`, ou `PoModule` inteiro) e siga a mesma escolha.

## Passo 5 — Estado atual

```bash
git log -10 --oneline
git status
```

Confira também `features/INDEX.md` para ver quais features têm escopo de tela em aberto.

## Relatório

### Contexto Externo (se houver)
Issue(s), página(s) e mockups: o que a tela precisa fazer.

### Stack
- Angular (versão), PO-UI (versão), tema aplicado
- Standalone ou NgModules; signals em uso?
- Runner de teste, gerenciador de pacote, scripts disponíveis

### Estrutura
- `core/`, `shared/`, `features/` — o que existe em cada
- Features já implementadas

### Fatia de Referência
Nome da feature lida e os caminhos, para servir de molde.

### Padrões Internalizados
- Nomenclatura de arquivo (`.component.ts` explícito ou padrão novo do CLI)
- Como os services chamam a API e de onde vem a URL
- Como o erro da API chega ao usuário (interceptor → `PoNotificationService`)
- Quais componentes PO o projeto já usa e como (granular vs `PoModule`)
- Padrão de formulário (reactive forms, grid `po-row`/`po-md-*`)
- Padrão de teste

### Integração com a API
Endpoints já consumidos e o formato de resposta esperado (array puro, `data`, ou
`items`/`hasNext` dos templates dinâmicos do PO).

### Estado Atual
Branch, mudanças recentes, pendências visíveis.

Use bullets e títulos — o relatório precisa ser escaneável.
