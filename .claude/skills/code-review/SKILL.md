---
name: code-review
description: Revisão técnica dos arquivos alterados — API .NET em Clean Architecture e tela Angular + PO-UI — cobrindo violações de camada, bugs, segurança e aderência aos padrões do projeto, com relatório salvo. Use antes de commitar, como portão de qualidade.
argument-hint: [CHAVE-JIRA]
---

# Code Review — revisão técnica

Revisa o que mudou no branch. Se receber uma chave de feature, use o `feature.md` e o `plan.md`
dela como referência do que **deveria** ter sido feito.

## Filosofia

- Simplicidade é o objetivo — cada linha precisa se justificar.
- Código é lido muito mais do que escrito.
- O melhor código costuma ser o que não precisou existir.
- Consistência com o padrão do projeto vale mais que elegância pessoal.

## Passo 0 — Delimitar a revisão

Veja o que mudou e decida quais trilhas revisar:

- Arquivos `.cs` → **trilha API** (seções 1 a 9 abaixo)
- Arquivos `.ts` / `.html` / `.css` sob `src/app/` → **trilha Frontend** (seção 10)
- Os dois → revise as duas. Numa feature fullstack, delegue em paralelo aos agentes
  `code-reviewer` (API) e `angular-code-reviewer` (tela), cada um no seu conjunto de arquivos, e
  consolide os dois relatórios.

## Passo 1 — Contexto

Leia antes de julgar qualquer linha:
- `CLAUDE.md`
- `.claude/references/clean-architecture-dotnet.md`
- `.claude/references/backend-api-best-practices.md`
- `.claude/references/dotnet-testing-standards.md`
- `.claude/references/angular-po-ui.md` e `angular-testing-standards.md` (se houver mudança de tela)
- `feature.md`, `plan.md` e `plan-ui.md` da feature (quando houver chave)

## Passo 2 — Levantar as mudanças

```bash
git status
git diff HEAD
git diff --stat HEAD
git ls-files --others --exclude-standard
```

Leia **cada arquivo novo ou alterado por inteiro**, não só o diff — contexto parcial gera
falso positivo.

## Passo 3 — Analisar

### 1. Violação de camada (CRÍTICO)

- `DbContext` injetado ou usado fora de Infrastructure
- Domain referenciando Application/Infrastructure/EF Core
- Regra de negócio em Controller
- Entidade vazando para fora do service (retorno de Controller ou DTO)
- `HttpContext` dentro de service (o correto é `IPortalAuthDataContext`/`IRequestAuthDataContext`)
- `using` de `Microsoft.AspNetCore.*` dentro de Application ou Domain

### 2. Bugs de lógica

- `GetFirstAsync` sem `asNoTracking: true` em leitura pura (ou **com** ele antes de um update
  que depende de tracking)
- Escrita sem `CommitAsync` — silenciosamente não persiste
- `Update` sem `setState: true` quando a entidade veio destacada
- Condições invertidas, off-by-one, `null` não tratado apesar do nullable habilitado
- `async` sem `await`; `.Result`/`.Wait()` (risco de deadlock)
- Coleção iterada e modificada; `Single` onde cabe `First`

### 3. Contrato de erro

- Exceção sem sufixo de status quando deveria virar 4xx (vira 500 e mente para o cliente)
- Sufixo errado (`404` para conflito, `400` para não encontrado)
- Mensagem em inglês, ou vazando nome de coluna, caminho de arquivo ou dado sensível
- `try/catch` no controller (o `[ErrorHandlingFilter]` deixa de agir)

### 4. Persistência e migrations

- Entidade nova/alterada **sem migration**
- `DbSet` não declarado no `ApplicationDbContext`
- Falta de índice único onde a regra de negócio exige unicidade
- N+1: relacionamento acessado em laço sem `includes`
- Colunas fora de snake_case; `MaxLength` ausente em `string`
- Vários `CommitAsync` dentro de um laço

### 5. DI

- Service criado e **não registrado** em `DependencyInjection.cs` (quebra em runtime)
- Tempo de vida errado (`Singleton` guardando estado de request)

### 6. Segurança

- Action sem `[Authorize]` e sem `[AllowAnonymous]` explícito
- Policy mais permissiva que o requisito (escrita liberada para `PortalLogPolicy`)
- Segredo/connection string em código
- Entrada do usuário concatenada em SQL cru
- Log de token, senha ou payload autenticado completo

### 7. Padrões do projeto

- Nomenclatura: `Entity`/`Dto`/`Contract`, `I` em interface, `_camelCase`, `Async` no sufixo
- Indentação de 2 espaços, namespace em bloco, primary constructor
- Rota kebab-case, constraint `{id:int}`, status de retorno correto (201/204)
- Mapeamento manual (nada de AutoMapper)

### 8. Testes

- Regra de negócio do `feature.md` sem teste correspondente
- Caminho de erro (404/400/409) sem teste
- `GetFirstAsync` mockado sem os três `It.IsAny` (setup não casa e devolve null)
- Teste que só verifica o mock, sem asserção de comportamento
- Nome fora do padrão `Metodo_QuandoCondicao_ResultadoEsperado`

### 9. Aderência ao plano

Compare com o `plan.md`: tarefa não feita, tarefa feita diferente sem registro de divergência,
critério de aceite sem cobertura.

### 10. Frontend (Angular + PO-UI)

Só quando houver mudança em `src/app/`. Detalhamento completo no agente `angular-code-reviewer`;
o essencial:

- **Estrutura**: feature isolada em `features/<f>/` com `models/`, `services/`, `pages/`,
  `*.routes.ts`; feature importando de outra feature é violação; rota lazy registrada
- **Tipagem**: `any` é achado, sempre; model espelhando o DTO em camelCase — **divergência de
  campo é crítica** (quebra em runtime, não na compilação); `PoTableColumn[]`/`PoPageAction[]`
  tipados
- **Service**: `inject(HttpClient)`, URL de `environment`, **sem `subscribe`**, sem `catchError`
  que engole erro e devolve lista vazia
- **PO-UI**: `provideAnimations()` presente; imports granulares; nenhum input inventado (confirme
  em `node_modules/@po-ui/ng-components`); `div` + CSS reimplementando componente que o PO já
  tem; `po-page-dynamic-*` contra API que não responde `{ items, hasNext }`
- **Formulário**: reactive forms; validações espelhando a API; `p-required` nos obrigatórios;
  submit bloqueado durante a requisição
- **Erro duplicado**: `notification.error(...)` no componente quando o interceptor já notifica —
  o usuário vê dois toasts
- **Ação destrutiva** sem `PoDialogService.confirm`
- **Change detection**: `OnPush` ausente; `subscribe` aninhado; subscription sem
  `takeUntilDestroyed`; função chamada no template
- **Template**: `*ngIf`/`*ngFor` em código novo; `@for` sem `track`
- **Testes**: standalone em `imports`, `provideNoopAnimations()`, `http.verify()`,
  `fixture.detectChanges()` presente

## Passo 4 — Confirmar que o problema é real

Não reporte suspeita como fato:
- Rode o teste específico para confirmar o bug
- Confira o caminho de arquivo e o registro no DI
- Verifique se a migration existe de fato em `Migrations/`
- No frontend: rode `npx tsc --noEmit`, e confirme input de componente PO em
  `node_modules/@po-ui/ng-components` antes de afirmar que não existe

Achado não confirmado vai marcado como **PLAUSÍVEL**, não como confirmado.

## Saída

Salve em `.claude/code-reviews/<CHAVE-ou-nome>.md` (e cite o caminho na resposta). Numa feature
fullstack, separe os achados em **API** e **Frontend**, mas dê **um único veredito**.

**Estatísticas:** arquivos modificados / adicionados / removidos, linhas +/-.

**Para cada issue:**

```
severidade: crítica|alta|média|baixa
arquivo: DTASkills.Application/Services/Produtos/ProdutoService.cs
linha: 42
issue: [uma linha]
detalhe: [por que é um problema — cenário concreto de falha]
correção: [como corrigir, com o padrão do projeto]
```

Ordene por severidade. Violação de camada e falha de segurança são sempre **críticas**.

Sem problemas: "Revisão aprovada. Nenhum problema técnico detectado." — e ainda assim liste o
que foi verificado.

## Importante

- Seja específico: arquivo e linha, nunca reclamação vaga.
- Foque em bug e violação de padrão, não em estilo pessoal.
- Proponha a correção, não só o problema.
- **Não corrija nada nesta skill.** Reporte e aguarde aprovação — a correção é `/code-review-fix`.
