# Roteiro — Construção de API em Clean Architecture com Agentes

Duração-alvo: **45–60 min** (30 min de conteúdo + demo ao vivo + perguntas).
Repositório de exemplo: estrutura de `DTASkills.Api` — .NET 10, PostgreSQL, EF Core, xUnit.

---

## Antes de subir no palco — checklist

- [ ] `dotnet --info` responde e o SDK cobre `net10.0`
- [ ] `dotnet build` da solução verde **antes** de começar
- [ ] `.claude/settings.json` criado a partir do `.example` (hooks e permissões ativos)
- [ ] MCP do Atlassian conectado — ou plano B: usar a feature `DEMO-101` já importada
- [ ] `/feature-status` roda e mostra a esteira
- [ ] Terminal com fonte grande; janela do editor lado a lado para ver os arquivos surgindo
- [ ] Feature de reserva: `features/DEMO-101-cadastro-produtos/feature.md` (não depende de rede)

---

## Bloco 1 — O problema (5 min)

Duas queixas que sempre aparecem juntas:

1. *"A IA escreve código que não parece com o nosso."*
2. *"Clean Architecture é lenta demais para o agente — ele se perde entre as camadas."*

A tese da apresentação: **as duas se resolvem com a mesma coisa** — não com prompt melhor, mas com
uma camada de contexto que torna o padrão do time inevitável.

Mostre o custo real: implementar uma feature nesta arquitetura toca **9 pontos** em 5 projetos.
Esquecer um (`AddScoped`, por exemplo) compila e quebra em runtime.

## Bloco 2 — A arquitetura (8 min)

Abra `CLAUDE.md` §1 e §2 no telão.

- As 5 camadas e a regra de dependência apontando para dentro
- O truque que torna isso AI-friendly: **camadas horizontais, pastas verticais**.
  `Services/Produtos/`, `Dtos/Produtos/`, `Entities/Produtos/` — o agente encontra a feature
  inteira buscando um nome só.
- Abra `references/clean-architecture-dotnet.md` e mostre a tabela dos 9 pontos de toque.

Frase-âncora: *"a arquitetura não é o obstáculo do agente — a arquitetura **não documentada** é."*

## Bloco 3 — As cinco primitivas da camada de IA (7 min)

| Primitiva | Arquivo | Quando age |
|---|---|---|
| Regras | `CLAUDE.md` | sempre |
| Contexto | `references/` | sob demanda |
| Skills | `skills/` | quando invocada |
| Subagentes | `agents/` | em contexto isolado |
| Hooks | `hooks/` | automaticamente, sem escolha do agente |

**Momento forte:** rode o hook ao vivo e mostre que ele bloqueia de verdade —

```bash
echo '{"tool_name":"Write","tool_input":{"file_path":"x/Migrations/20260101_Foo.cs"}}' \
  | python .claude/hooks/pre_tool_use.py; echo "exit=$?"
```

`exit=2` → o agente é impedido de escrever migration à mão e recebe o motivo.
*"Regra pede. Hook garante."*

## Bloco 4 — A esteira Jira → código (5 min)

Abra `references/jira-feature-workflow.md` e mostre o diagrama.

O ponto central: **cada seta é uma skill, cada arquivo é um contrato**.
O que não está no `feature.md` não chega ao `plan.md`; o que não está no `plan.md` não vira código.
É isso que impede o agente de "inventar de passagem".

Mostre `features/DEMO-101-cadastro-produtos/feature.md`: regras de negócio **com status HTTP**,
critérios de aceite **verificáveis**, modelo de dados com colunas em snake_case.

## Bloco 5 — DEMO AO VIVO (20 min)

### Passo 1 — Onde estamos (1 min)

```
/feature-status
```

### Passo 2 — Contexto (2 min)

```
/prime-backend
```

Enquanto roda, narre: ele lê a solução, os `.csproj` (para provar a direção das dependências) e
**uma fatia vertical completa** — que vira o molde do código que ele vai escrever.

### Passo 3 — A feature vem do Jira (3 min)

```
/feature-from-jira DEMO-101
```

> Sem Atlassian conectado? Pule este passo — a `DEMO-101` já está importada. Diga isso e siga;
> é inclusive um bom momento para mostrar o fallback manual descrito na skill.

Mostre a tradução de negócio para camadas: *"SKU único"* virou índice `uq_produtos_sku` **e** erro
409 **e** um teste.

### Passo 4 — Planejar (4 min)

```
/plan-feature DEMO-101
```

Abra o `plan.md` gerado. Destaque:
- a ordem das tarefas seguindo a regra de dependência
- cada tarefa com **PADRÃO** apontando para um arquivo real
- cada tarefa com um comando de validação executável
- a nota de confiança

*"Aqui nenhuma linha de produção foi escrita ainda. É de propósito."*

### Passo 5 — Executar (8 min)

```
/execute DEMO-101
```

Deixe rodar e narre as camadas conforme aparecem. Aponte:
- o `dotnet build` entre camadas (erro barato agora, caro depois)
- o `AddScoped` no `DependencyInjection.cs` — o passo mais esquecido
- a migration gerada pelo `dotnet ef`, não escrita à mão (o hook garante)
- o `checklist.md` sendo marcado em tempo real

### Passo 6 — Revisar e fechar (2 min)

```
/code-review
/validate
```

Mostre o relatório e o portão verde. Se a revisão achar algo, melhor ainda: `/code-review-fix`.

## Bloco 6 — O loop que fecha (5 min)

```
/execution-report DEMO-101
/system-review DEMO-101
```

O `system-reviewer` não revisa código — revisa o **processo**. Divergência boa vira melhoria do
plano; divergência ruim vira regra no `CLAUDE.md`; passo manual repetido 3 vezes vira skill.

*"A camada de IA não nasce pronta. Ela é depurada como código."*

## Bloco 7 — Fechamento (3 min)

Três coisas para levar para casa:

1. **Documentar o padrão vale mais que escolher o padrão.** Clean Architecture funciona muito bem
   com agentes — desde que os 9 pontos de toque estejam escritos.
2. **O contrato entre etapas é o que segura o escopo.** Requisito → plano → código, cada um por
   escrito.
3. **Onde não pode falhar, use hook, não prompt.**

---

## Perguntas prováveis

**"Clean Architecture não gasta mais token que vertical slice?"**
Gasta mais para *navegar*, sim — e é exatamente por isso que existe o `references/` com o template
de cada camada e o `prime-backend` lendo **uma** fatia em vez de tudo. O agente não precisa
descobrir o padrão a cada feature; ele recebe o padrão pronto.

**"E se o agente errar a camada?"**
Três redes: o `plan.md` fixa a ordem, o `code-reviewer` trata violação de camada como crítica, e o
`dotnet build` entre camadas falha cedo.

**"Preciso do Jira para usar isso?"**
Não. `/feature-from-jira` aceita o texto colado, e dá para criar o `feature.md` do `_template/`.
Perde-se a rastreabilidade automática, não a esteira.

**"Isso substitui o desenvolvedor?"**
Não. Ele decide o requisito, aprova o plano e revisa a revisão — os três pontos onde errar sai
caro. O agente faz as 9 tarefas mecânicas com fidelidade, que é onde a atenção humana escorre.

---

## Plano B

| Se falhar | Faça |
|---|---|
| MCP do Atlassian fora | Use `DEMO-101`, já importada |
| Sem banco / `dotnet ef` falha | Pule a migration; mostre o comando e o hook que a exige |
| `/execute` travar ou demorar | Corte para o `checklist.md` e mostre o resultado de uma execução anterior |
| Sem rede | Tudo roda local, exceto o passo do Jira |
