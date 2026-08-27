---
name: execute
description: Implementa a feature a partir do plan.md, camada a camada (Domain → Application → Infrastructure → Api → Testes), compilando a cada etapa e atualizando o checklist da feature. Use quando o plano estiver pronto e aprovado.
argument-hint: <CHAVE-JIRA> | <caminho do plan.md>
---

# Execute — implementar o plano, camada a camada

## Entrada: $ARGUMENTS

- **Chave do Jira** → o plano é `features/<CHAVE>-*/plan.md`.
- **Caminho de arquivo** → use-o diretamente.

Se não houver `plan.md`, **pare**: rode `/plan-feature <CHAVE>` antes. Não improvise o plano.

## Passo 1 — Carregar contexto

1. Leia o `plan.md` inteiro, depois o `feature.md` da mesma pasta.
2. Leia `CLAUDE.md` e `.claude/references/clean-architecture-dotnet.md`.
3. Leia **todos** os arquivos listados em "Contexto obrigatório / Ler antes de implementar" —
   é deles que sai o estilo do código que você vai escrever.
4. Abra o `checklist.md` da feature: se já houver itens marcados, retome de onde parou em vez de
   refazer.
5. Marque a feature como `⚙️ em execução` em `features/INDEX.md`.

## Passo 2 — Executar na ordem das camadas

Siga as tarefas T1..Tn **em ordem**. A ordem existe porque a compilação depende dela.

```
Domain → Application (DTO → mapper → interface → service) → DI
       → Infrastructure (DbSet → índices → migration) → Api (controller) → Testes
```

Para **cada** tarefa:

1. **Abra o arquivo-padrão** citado em **PADRÃO** antes de escrever. Espelhe estilo,
   indentação (2 espaços), namespace em bloco, primary constructor, ordem dos usings.
2. **Implemente** exatamente o que está em **IMPLEMENTAR**, nada além.
3. **Rode o comando de VALIDAR.** Se falhar: corrija, rode de novo, e só então siga.
4. **Marque o item no `checklist.md`** da feature.

### Invariantes por camada (verifique enquanto escreve)

| Camada | Invariante |
|---|---|
| Domain | `[Table]`/`[Column]` em snake_case; `string` com `= string.Empty`; sem referência a outro projeto |
| Application/DTO | POCO puro, sem lógica, sem conhecer entidade |
| Application/Mapper | mapeamento manual campo a campo; sem AutoMapper; sem campos de auditoria |
| Application/Service | `IUnitOfWork` (jamais `DbContext`); `try/catch` re-lançando `new Exception(ex.Message)`; erro com sufixo de status; `CommitAsync` em toda escrita |
| DI | `AddScoped<IXService, XService>()` — esquecer isso compila e quebra em runtime |
| Infrastructure | `DbSet` + índice `uq_<tabela>_<coluna>` + migration gerada |
| Api | controller fino, `[ApiController] [ErrorHandlingFilter] [Route("api/kebab-case")]`, policy por action, sem `try/catch` |
| Testes | xUnit + Moq, nome `Metodo_QuandoCondicao_ResultadoEsperado`, três `It.IsAny` no setup de `GetFirstAsync` |

### Regras durante a execução

- **Não pule a compilação entre camadas.** Erro de camada é barato agora e caro no fim.
- **Não invente padrão.** Se o plano não cobre um caso, siga a feature-espelho. Se ainda assim
  não couber, registre em "Divergências do Plano" no `checklist.md` e explique.
- **Não amplie o escopo.** Nada de refatorar código vizinho, renomear coisas ou "melhorar de
  passagem". O que está fora do plano fica fora.
- **Não invente migration à mão.** Use `dotnet ef migrations add`; se falhar, reporte o erro real.
- Se uma tarefa se mostrar impossível como planejada, **pare e explique** antes de improvisar
  uma arquitetura alternativa.

## Passo 3 — Testes

Implemente os testes descritos na "Estratégia de Teste" do plano — todos, incluindo os caminhos
de erro. Um teste por cenário da tabela; sem teste "de fachada" que só verifica o mock.

Cobertura mínima: happy path, não encontrado, cada RN do `feature.md`, e `Verify` do
`CommitAsync` nos métodos de escrita.

## Passo 4 — Validação final

Rode **todos** os comandos de validação do plano, em ordem:

```bash
dotnet build <Sln>.Api/<Sln>.Api.sln
dotnet test  <Sln>.Api/<Sln>.Api.sln --filter "FullyQualifiedName~<Nome>ServiceTests"
dotnet test  <Sln>.Api/<Sln>.Api.sln
dotnet format <Sln>.Api/<Sln>.Api.sln --verify-no-changes
```

Falhou algum? Corrija e rode de novo. **Não relate conclusão com comando vermelho** — se algo
ficou quebrado, diga exatamente o quê e por quê.

Depois, confira os critérios de aceite do `feature.md` um a um e marque os atendidos.

## Passo 5 — Fechar o estado

- `checklist.md`: todos os itens das camadas concluídas marcados; divergências registradas.
- `feature.md`: critérios de aceite atendidos marcados.
- `INDEX.md`: estado `🔍 em revisão`.

## Saída

### Tarefas Concluídas
Lista T1..Tn com ✅/❌ e o arquivo tocado.

### Arquivos
- Criados (caminho completo)
- Modificados (caminho completo + o que mudou)

### Testes
- Arquivo(s) de teste, número de casos, resultado

### Validação
Saída resumida de cada comando, com veredito ✅/❌.

### Divergências do Plano
Cada desvio, com motivo. "Nenhuma" se for o caso.

### Pronto para
`/code-review` e depois `/commit` — ou a lista do que ainda falta.
