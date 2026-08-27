# Checklist — {CHAVE-JIRA} {Título}

Atualize **durante** a execução, ao concluir cada camada. É este arquivo que permite retomar a
feature de onde parou.

## Camada 1 — Domain

- [ ] `Entities/{Feature}/{Nome}Entity.cs` criada
- [ ] `[Table]` e `[Column]` em snake_case
- [ ] Campos de auditoria (se aplicável)
- [ ] `dotnet build` ✅

## Camada 2 — Application: contratos de dados

- [ ] `Dtos/{Feature}/{Nome}Dto.cs`
- [ ] DTOs auxiliares (Update/Summary/Request) se necessários
- [ ] `Extensions/{Feature}/{Nome}Extensions.cs` com `ToDto()` / `ToEntity()`
- [ ] `dotnet build` ✅

## Camada 3 — Application: caso de uso

- [ ] `Services/{Feature}/I{Nome}Service.cs`
- [ ] `Services/{Feature}/{Nome}Service.cs`
- [ ] Todas as regras de negócio do `feature.md` implementadas
- [ ] Erros com sufixo de status correto
- [ ] Registrado em `DependencyInjection.cs` (`AddScoped`)
- [ ] `dotnet build` ✅

## Camada 4 — Infrastructure

- [ ] `DbSet<{Nome}Entity>` no `ApplicationDbContext`
- [ ] Índices/relacionamentos no `OnModelCreating`
- [ ] Migration gerada: `{NomeDaMigration}`
- [ ] `dotnet build` ✅

## Camada 5 — Api

- [ ] `Controllers/{Nome}Controller.cs`
- [ ] Rota kebab-case + constraints `{id:int}`
- [ ] Policy declarada em cada action
- [ ] Status de retorno corretos (200/201/204)
- [ ] `dotnet build` ✅

## Camada 6 — Testes

- [ ] `TesteUnitario/{Feature}/{Nome}ServiceTests.cs`
- [ ] Happy path de cada método público
- [ ] Caminho "não encontrado" (404)
- [ ] Cada regra de negócio do `feature.md` com teste
- [ ] `Verify` de `CommitAsync` nos métodos de escrita
- [ ] `dotnet test` ✅

## Fechamento

- [ ] Todos os critérios de aceite do `feature.md` marcados
- [ ] `/validate` verde (build + test + format)
- [ ] `/code-review` sem issue crítica ou alta pendente
- [ ] `INDEX.md` atualizado para ✅ concluída
- [ ] Commit `feat({CHAVE-JIRA}): ...`

## Divergências do Plano

| Item do plano | O que foi feito | Motivo |
|---|---|---|
| | | |
