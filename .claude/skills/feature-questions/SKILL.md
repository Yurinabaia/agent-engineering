---
name: feature-questions
description: Roda o questionário de descoberta sobre uma feature — perguntas de negócio e técnicas — respondendo o que já está na issue e no código, e devolvendo só as lacunas reais, separadas entre bloqueantes, assumíveis e opcionais. Use antes de planejar, quando o requisito parecer vago, ou para preparar uma conversa com o PO.
argument-hint: <CHAVE-JIRA> | <descrição da feature>
---

# Feature Questions — o que ainda não sabemos

## Objetivo

Descobrir **o que falta saber** antes de planejar, sem transformar isso num interrogatório.
O valor desta skill não é fazer perguntas — é **não fazer as que já têm resposta**.

Fonte das perguntas: `.claude/references/feature-discovery-questions.md`. Leia antes de começar.

## Entrada: $ARGUMENTS

- **Chave do Jira** → leia `features/<CHAVE>-*/feature.md` se existir; senão, busque a issue.
- **Descrição livre** → trabalhe com o texto informado.
- **Sem argumento** → pergunte sobre qual feature é.

## Passo 1 — Responder sozinho primeiro

**Esta é a etapa que mais economiza tempo do time.** Para cada pergunta do banco, procure a
resposta, nesta ordem:

1. **A issue do Jira** — descrição, critérios de aceite, comentários que mudam requisito, anexos
2. **O épico pai** — contexto arquitetural costuma morar lá
3. **O código existente** — e é aqui que a maior parte aparece:
   - `Entities/` e `ApplicationDbContext.cs` → a entidade já existe? há índice único? há auditoria?
   - `Controllers/` → a rota já está tomada? qual policy o time usa em caso parecido?
   - `Services/` de uma feature vizinha → como regras semelhantes já foram validadas
   - `src/app/features/` → já existe tela parecida? qual padrão de navegação?
4. **Os defaults do projeto** (tabela no banco de perguntas) — para as 🟡

Marque cada pergunta como **respondida** (com a fonte), **assumida** (com o default) ou
**em aberto**.

## Passo 2 — Classificar as lacunas

Use a classificação do banco:

| | Critério | Consequência |
|---|---|---|
| 🔴 Bloqueante | Muda schema, migration ou contrato da API | Feature vai para `⛔ bloqueada` |
| 🟡 Assumível | Existe default de projeto | Assuma e registre a premissa |
| 🟢 Opcional | Só refina | Anota e segue |

Na dúvida entre 🔴 e 🟡, pergunte-se: **"errar isso me obriga a escrever outra migration ou a
mudar a assinatura de um endpoint?"** Se sim, é 🔴.

## Passo 3 — Perguntar ao humano

Leve **só as 🔴 e as 🟡 relevantes**, no máximo 8 por rodada. Para cada uma:

- Escreva a pergunta em linguagem de negócio, não de banco de dados.
  ❌ "Qual a cardinalidade entre Produto e Categoria?"
  ✅ "Um produto pode estar em mais de uma categoria ao mesmo tempo?"
- **Ofereça a opção que você assumiria**, para a pessoa poder só confirmar:
  "Vou assumir que excluir apaga o registro de vez. Confirma, ou o certo é inativar?"
- Explique em meia linha o que muda: "isso define se a tabela ganha um campo `ativo`".

Perguntas 🟢 vão numa lista separada, marcada como "não bloqueia, responda quando puder".

## Passo 4 — Registrar

Escreva na seção **Perguntas e Premissas** do `feature.md` as três listas — Respondidas,
Premissas assumidas, Em aberto. Sempre as três, mesmo que uma esteja vazia ("nenhuma").

Se houver 🔴 em aberto, marque a feature como `⛔ bloqueada` em `features/INDEX.md`.

## Saída

### Cobertura do Questionário

| Bloco | Respondidas | Assumidas | Em aberto |
|---|---|---|---|
| A — Negócio | n | n | n |
| B — API | n | n | n |
| C — Frontend | n | n | n |
| D — Operação | n | n | n |

### Respondidas sem precisar perguntar
Lista curta com a **fonte** de cada uma (issue, código, default). É isto que mostra que o
questionário não é burocracia.

### 🔴 Bloqueantes — precisam de resposta antes de planejar
Numeradas, com a opção sugerida e o que cada uma muda no código.

### 🟡 Premissas que estou assumindo
O que foi assumido, com base em quê, e **qual o custo se estiver errado**.

### 🟢 Podem esperar
Lista enxuta.

### Veredito
**Pronta para planejar** (`/plan-feature <CHAVE>`) ou **bloqueada**, com o que falta.

## Regras

- **Nunca invente resposta.** Não achou e não há default? É pergunta em aberto.
- **Nunca faça pergunta que o código responde.** Procure antes.
- **Nunca despeje o banco inteiro no usuário.** 60 perguntas de uma vez não são levantamento de
  requisito, são desistência. Priorize.
- Feature simples e bem descrita pode sair daqui com **zero** perguntas — e esse é o melhor
  resultado possível, não um sinal de que a skill não fez nada.
