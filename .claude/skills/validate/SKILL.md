---
name: validate
description: Roda o portão de qualidade do projeto .NET — build, testes, formatação e (opcional) smoke da API — e reporta a saúde geral com ✅/❌. Use antes de abrir PR ou fechar uma feature.
argument-hint: [CHAVE-JIRA]
---

# Validate — portão de qualidade

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

## Relatório

| Nível | Comando | Resultado |
|---|---|---|
| Build | `dotnet build` | ✅/❌ |
| Testes | `dotnet test` | ✅/❌ (n passaram, n falharam) |
| Formatação | `dotnet format --verify-no-changes` | ✅/❌ |
| Migrations | `dotnet ef migrations list` | ✅/❌/n/a |
| Smoke | AppHost + Swagger | ✅/❌/n/a |

**Saúde geral: PASS / FAIL**

Em caso de FAIL, liste cada falha com o arquivo e a mensagem real do compilador ou do teste —
sem resumir a ponto de perder a causa.

## Notas

- Regra de negócio que depende de data, fuso, arredondamento de preço ou usuário autenticado
  precisa de teste com valor não trivial. É o tipo de regressão que passa batido na revisão.
- `dotnet test` que não roda nenhum teste é ❌ disfarçado de ✅ — confira a contagem.
