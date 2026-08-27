---
name: init-project
description: Prepara e roda ESTE projeto .NET localmente a partir de um clone limpo — SDK, restore, banco, migrations e execução via Aspire ou pela API. Lê a configuração real do repositório em vez de assumir.
---

# Inicializar o projeto localmente

Leia primeiro o `README`, os `appsettings*.json`, o `launchSettings.json` e o `AppHost.cs`
(quando houver Aspire) — e use os valores reais do projeto, não defaults inventados.

## Passos

### 1. SDK

```bash
dotnet --info
```

Confirme que a versão do SDK cobre o `<TargetFramework>` dos `.csproj` (o projeto de referência
usa `net10.0`). Faltando, instale o SDK correspondente antes de seguir.

### 2. Restore e build

```bash
dotnet restore <Sln>.Api/<Sln>.Api.sln
dotnet build   <Sln>.Api/<Sln>.Api.sln
```

### 3. Banco de dados

O projeto de referência usa **PostgreSQL** com a connection string
`ConnectionStrings:DefaultConnectionPostgres` (ou a variável de ambiente
`POSTGRESQLCONNSTR_DefaultConnectionPostgres`).

- Suba o banco (container, instância local ou o recurso declarado no AppHost).
- Configure a connection string em `appsettings.Development.json` ou em user-secrets:
  ```bash
  dotnet user-secrets set "ConnectionStrings:DefaultConnectionPostgres" "<conn>" -p <Sln>.Api
  ```
  Prefira user-secrets a editar o `appsettings` versionado.

### 4. Migrations

```bash
dotnet tool install --global dotnet-ef        # se ainda não tiver
dotnet ef database update -p <Sln>.Infrastructure -s <Sln>.Api
```

Em build não-DEBUG o projeto aplica `Database.Migrate()` no startup; em desenvolvimento,
aplique manualmente.

### 5. Configurações obrigatórias

Confira as seções que o startup exige e que quebram a aplicação se faltarem:
- `JwtSettings` e o esquema do portal (`JwtPortalLogSettings`)
- Integrações externas configuradas via `AddHttpClient` (ex.: `McpSql:BaseUrl`,
  `McpSql:BearerToken` no projeto de referência)

Falta configurada vira `InvalidOperationException` no startup — leia a mensagem, ela nomeia a chave.

### 6. Rodar

```bash
dotnet run --project <Sln>.Api.AppHost     # com Aspire (dashboard + dependências)
dotnet run --project <Sln>.Api             # só a API
```

## Validar

- Dashboard do Aspire abre e mostra o recurso da API saudável
- Health endpoint responde
- Swagger em `/swagger` (só em Development) lista os controllers
- Um `GET` autenticado em uma rota conhecida retorna 200

## Notas

- Portas, nomes de recurso e chaves de configuração mudam por projeto — sempre confira
  `launchSettings.json` e `appsettings*.json` antes de assumir.
- 401 em todas as rotas normalmente é configuração de JWT ausente, não bug de código.
