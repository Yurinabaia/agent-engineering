---
name: validate
description: Roda o portão de qualidade do projeto — build, testes, formatação e smoke da API (.NET) e build, tipos, testes e lint da tela (Angular) — e reporta a saúde geral com ✅/❌. Use antes de abrir PR ou fechar uma feature.
argument-hint: [CHAVE-JIRA]
---

# Validate — portão de qualidade

Detecte primeiro **o que existe neste repositório** e rode só as trilhas aplicáveis:

| Encontrou | Rode |
|---|---|
| `*.sln` | a trilha **API** (níveis 1 a 5) |
| `angular.json` | a trilha **Frontend** (níveis 6 a 9) |
| os dois | as duas, e reporte as duas no relatório |

---

# Trilha API (.NET)

Descubra a solução real antes de rodar: `ls *.sln` ou `find . -name "*.sln" -maxdepth 3`.
No projeto de referência ela fica em `<Sln>.Api/<Sln>.Api.sln`.

Se receber uma chave de feature em `$ARGUMENTS`, use-a para rodar também o filtro de testes da
feature e reportar os critérios de aceite.

## Nível 1 — Compilação

```bash
dotnet build <caminho>.sln --nologo
```

Zero erro. **Warning novo também conta** — compare com o estado anterior do branch se houver
dúvida.

## Nível 2 — Testes

```bash
dotnet test <caminho>.sln --nologo
```

Feature específica:

```bash
dotnet test <caminho>.sln --filter "FullyQualifiedName~<Nome>ServiceTests"
```

Com cobertura:

```bash
dotnet test <caminho>.sln --collect:"XPlat Code Coverage"
```

## Nível 3 — Formatação

```bash
dotnet format <caminho>.sln --verify-no-changes
```

Falhou? `dotnet format <caminho>.sln` e recompile.

## Nível 4 — Migrations pendentes

Se a feature alterou entidade, confirme que a migration existe e está consistente:

```bash
dotnet ef migrations list -p <Sln>.Infrastructure -s <Sln>.Api
```

Entidade alterada sem migration correspondente é ❌.

## Nível 5 — Smoke da API (opcional)

```bash
dotnet run --project <Sln>.Api.AppHost
```

Confira o health endpoint e as rotas da feature no Swagger (`/swagger`, só em Development).

---

# Trilha Frontend (Angular + PO-UI)

## Nível 6 — Tipos

```bash
npx tsc --noEmit
```

Rápido e pega quase tudo. `any` novo não falha aqui — é achado da revisão, não do portão.

## Nível 7 — Build

```bash
npm run build
```

Erro de template (input de componente PO que não existe, `@for` sem `track`) só aparece aqui.
Warning de budget estourado conta: bundle inchado normalmente é `PoModule` inteiro importado em
componente pequeno.

## Nível 8 — Testes

```bash
npm test -- --watch=false --browsers=ChromeHeadless
```

Sem `--watch=false --browsers=ChromeHeadless` o processo trava esperando browser.
Teste específico: `npm test -- --watch=false --browsers=ChromeHeadless --include='**/produto*.spec.ts'`

## Nível 9 — Lint

```bash
npx ng lint
```

Sem script de lint configurado, reporte `n/a` — não invente configuração.

## Nível 10 — Smoke da tela (opcional)

```bash
npm start
```

Abra a rota da feature, confirme que a listagem carrega e que um erro forçado da API vira toast.
A skill `agent-browser` automatiza isso e salva screenshots.

---

## Relatório

### API (se aplicável)

| Nível | Comando | Resultado |
|---|---|---|
| Build | `dotnet build` | ✅/❌ |
| Testes | `dotnet test` | ✅/❌ (n passaram, n falharam) |
| Formatação | `dotnet format --verify-no-changes` | ✅/❌ |
| Migrations | `dotnet ef migrations list` | ✅/❌/n/a |
| Smoke | AppHost + Swagger | ✅/❌/n/a |

### Frontend (se aplicável)

| Nível | Comando | Resultado |
|---|---|---|
| Tipos | `npx tsc --noEmit` | ✅/❌ |
| Build | `npm run build` | ✅/❌ |
| Testes | `npm test` | ✅/❌ (n passaram, n falharam) |
| Lint | `npx ng lint` | ✅/❌/n/a |
| Smoke | `npm start` + rota da feature | ✅/❌/n/a |

**Saúde geral: PASS / FAIL** (basta uma trilha vermelha para o veredito ser FAIL)

Em caso de FAIL, liste cada falha com o arquivo e a mensagem real do compilador ou do teste —
sem resumir a ponto de perder a causa.

## Notas

- Regra de negócio que depende de data, fuso, arredondamento de preço ou usuário autenticado
  precisa de teste com valor não trivial. É o tipo de regressão que passa batido na revisão.
- `dotnet test` que não roda nenhum teste é ❌ disfarçado de ✅ — confira a contagem. Vale igual
  para `npm test`.
- Numa feature fullstack, um lado verde não fecha a feature. Se a API passa e a tela não compila,
  o veredito é FAIL.
- `npm install` desatualizado é causa comum de falha falsa no build da tela — se o `package.json`
  mudou no branch, rode `npm ci` antes de acusar o código.
