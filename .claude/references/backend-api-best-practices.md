# Boas Práticas de API — ASP.NET Core

Contexto sob demanda para agentes mexendo em endpoints. Carregue antes de criar ou alterar
controllers. Complementa `clean-architecture-dotnet.md` (que traz os templates de código).

---

## Rotas

- Agrupe por recurso: `api/model-names`, `api/chat-agents`, `api/platform-versions`.
- **Substantivos no plural, kebab-case, sem verbo na URL** — `GET api/documents`, nunca
  `api/getDocuments`.
- Sub-recurso apenas um nível: `api/chat-agents/{id:int}/skills`.
- Constraints tipadas sempre: `{id:int}`, `{key:guid}`.
- Ações que não são CRUD viram sub-rota explícita: `POST api/platform-versions/import`.

## Verbos e status de sucesso

| Operação | Verbo | Retorno |
|---|---|---|
| Listar | `GET` | `Ok(list)` |
| Obter por id | `GET {id:int}` | `Ok(dto)` |
| Criar | `POST` | `CreatedAtAction(nameof(GetById), new { id = result.Id }, result)` |
| Atualizar | `PUT {id:int}` | `Ok(dto)` |
| Atualização parcial | `PATCH {id:int}` | `Ok(dto)` |
| Excluir | `DELETE {id:int}` | `NoContent()` |

Em `PUT`, force o id da rota no DTO antes de chamar o service: `dto.Id = id;`.

## Validação de entrada

- `[ApiController]` já devolve `400` automático para binding inválido.
- Data Annotations no DTO (`[Required]`, `[MaxLength]`, `[Range]`) cobrem o formato.
- **Regra de negócio se valida no service**, não no controller — e lança com sufixo de status:
  ```csharp
  if (dto.Preco <= 0) throw new Exception("Preço deve ser maior que zero 400");
  if (await repo.AnyAsync(e => e.Nome == dto.Nome)) throw new Exception("Nome já cadastrado 409");
  ```
- Nunca interpolar entrada do usuário em SQL cru. Consultas passam por LINQ/EF Core.

## Tratamento de erro

- Centralizado no `[ErrorHandlingFilter]` — resposta em `ProblemDetails` (RFC 7807).
- ❌ Sem `try/catch` no controller. ❌ Sem `return BadRequest(...)` manual.
- Mensagem em português, voltada a quem consome a API; sem stack trace, sem caminho interno,
  sem nome de coluna do banco.
- Erro inesperado vira 500 com a mensagem original — por isso mensagens de exceção nunca devem
  conter dado sensível.

| Situação | Status |
|---|---|
| Criado | 201 |
| Sem conteúdo (delete) | 204 |
| Validação / regra violada | 400 |
| Não autenticado | 401 |
| Autenticado sem permissão | 403 |
| Não encontrado | 404 |
| Conflito / duplicidade | 409 |
| Erro do servidor | 500 |

## Autorização

- Toda action declara sua policy explicitamente — não existe endpoint sem `[Authorize]`
  a menos que o requisito diga que é público (e aí `[AllowAnonymous]` explícito).
- `PortalLogPolicy` para leitura autenticada; `AdminPolicy` para escrita/administração.
- Dados do usuário autenticado vêm de `IPortalAuthDataContext` / `IRequestAuthDataContext`,
  nunca de `HttpContext` dentro do service.

## Paginação

- Qualquer listagem que possa crescer usa `PaginationFilterContract` (default 1/10, teto 1000)
  e devolve `PagedResponseContract<T>`.
- Ordenação determinística sempre — sem `OrderBy` a paginação repete/perde registros.
- Filtro e ordenação vêm como query string, aplicados no service via `WhereAsync(...)`.

## Assíncrono

- Toda action é `async Task<IActionResult>`; todo método de service que toca I/O é `Async`.
- ❌ Nunca `.Result` ou `.Wait()` — deadlock em ASP.NET.
- Propague `CancellationToken` quando adicionar operações longas.

## Documentação (OpenAPI)

- O projeto expõe OpenAPI + Swagger UI apenas em Development (`app.MapOpenApi()`).
- Nomes de action claros (`GetAll`, `GetById`, `Create`) — viram `operationId`.
- Anote respostas não óbvias com `[ProducesResponseType(StatusCodes.Status404NotFound)]`.

## Performance

- Leitura sem alteração usa `asNoTracking: true`.
- Evite N+1: carregue relacionamentos com o parâmetro `includes` do repositório.
- Projete para DTO o mais cedo possível; não traga a entidade inteira quando precisa de 2 campos.
- Operações em lote usam `AddRange`/`RemoveRange` + **um** `CommitAsync` no fim.

## Segurança

- Segredos só em configuração (`appsettings`, variáveis de ambiente, Key Vault) — nunca no código.
- CORS já configurado globalmente; não abra política por controller.
- Não logar token, senha ou payload completo de request autenticado.
- Valide que o recurso pertence ao tenant/usuário antes de devolver — 404 é preferível a 403
  quando revelar a existência do recurso já é vazamento.
