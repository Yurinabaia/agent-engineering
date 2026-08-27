# Banco de Perguntas — Descoberta de Feature

Contexto sob demanda. **Carregue ao importar ou planejar qualquer feature.**

Este é o questionário que transforma um pedido em requisito implementável. Cada pergunta existe
porque **decide uma linha de código** — a coluna "o que decide" é o que separa este banco de um
formulário burocrático. Pergunta que não decide nada não entra aqui, e não deve ser feita ao PO.

---

## Como usar

1. **Responda primeiro sozinho.** Antes de perguntar ao humano, procure a resposta na issue do
   Jira, nos anexos, no código existente e nas features vizinhas. Perguntar o que já está escrito
   queima a paciência de quem responde.
2. **Classifique o que sobrou.** Toda pergunta sem resposta cai em uma de três caixas:

   | | Significado | O que fazer |
   |---|---|---|
   | 🔴 **Bloqueante** | Errar aqui obriga a refazer schema, contrato de API ou migration | **Pare e pergunte.** A feature vai para `⛔ bloqueada` |
   | 🟡 **Assumível** | Existe um default de projeto razoável | Assuma, **registre a premissa** no `feature.md` e siga |
   | 🟢 **Opcional** | Só refina; não muda estrutura | Anote como pendência e siga |

3. **Registre tudo.** Resposta e premissa vão para a seção **Perguntas e Premissas** do
   `feature.md`. Premissa não registrada vira retrabalho silencioso três semanas depois.

> Regra prática: **o que muda o banco de dados ou o contrato da API é bloqueante.** O resto quase
> nunca é.

---

## BLOCO A — Perguntas de Negócio

### A1. Problema e valor

| Pergunta | O que decide |
|---|---|
| Que problema isso resolve, em uma frase? | Objetivo do `feature.md`; serve de critério para cortar escopo |
| Como as pessoas resolvem isso hoje? | Revela regras não escritas (a planilha atual costuma ser a especificação real) |
| O que acontece se não fizermos nada? | Prioridade e tamanho do MVP |
| Quantas pessoas usam por dia/semana? | Decide se precisa de paginação, cache e índice |

🟢 Opcionais — mas as respostas mudam o **tamanho** da entrega, então valem os dois minutos.

### A2. Atores e permissões 🔴

| Pergunta | O que decide |
|---|---|
| Quem usa essa funcionalidade? Quantos perfis diferentes? | Policies do controller |
| Quem pode **ver**? Quem pode **criar/alterar/excluir**? | `[Authorize("PortalLogPolicy")]` vs `[Authorize("AdminPolicy")]` por action |
| Existe dado que um perfil não pode enxergar? | Filtro no service + decisão 404 vs 403 |
| A permissão depende do dado (só vê o que é dele)? | Filtro por usuário/tenant na consulta — **nunca só na tela** |

Bloqueante: policy errada é falha de segurança, e descobrir depois costuma exigir mudar assinatura
de método e testes.

### A3. Ciclo de vida do dado 🔴

| Pergunta | O que decide |
|---|---|
| Como o registro nasce? Manual, importado, ou por integração? | Endpoints necessários (POST simples vs carga em lote) |
| Ele passa por estados? Quais e em que ordem? | Campo de status + enum + validação de transição |
| Pode ser alterado depois de criado? Todos os campos ou só alguns? | Campos travados em edição, no DTO e na tela |
| Excluir apaga mesmo ou só inativa? | `DELETE` físico vs campo `Ativo` — **muda a migration** |
| Precisa de aprovação de alguém? | Estado extra + endpoint de aprovação + policy |

### A4. Regras e invariantes 🔴

| Pergunta | O que decide |
|---|---|
| O que **nunca** pode acontecer nesse cadastro? | Validações no service, cada uma com seu status HTTP |
| Que combinação de campos é única? | Índice único `uq_<tabela>_<coluna>` + erro 409 |
| Há limites de valor (mínimo, máximo, faixa)? | `Validators` na tela + validação no service (400) |
| Quais campos são obrigatórios de verdade? | `[Required]` + `MaxLength` na entidade, `p-required` na tela |
| Qual mensagem o usuário deve ver quando violar cada regra? | Texto da exceção — que chega na tela pelo interceptor |

**Toda regra precisa sair daqui com um status HTTP.** Regra sem status é regra incompleta:
ela vira 500 e mente para quem consome a API.

### A5. Volume e crescimento 🟡

| Pergunta | O que decide |
|---|---|
| Quantos registros existem hoje? Quantos por mês? | Paginação sim/não; `PagedResponseContract` |
| Por quais campos as pessoas vão buscar? | Índices e o filtro do `po-page-list` |
| Qual a ordenação natural da lista? | `OrderBy` determinístico (sem ele, paginação repete registros) |

Default quando não respondem: **até ~500 registros, lista inteira; acima disso, paginação.**

### A6. Auditoria e conformidade 🔴 (quando a resposta é "sim")

| Pergunta | O que decide |
|---|---|
| Precisa saber quem alterou e quando? | Campos `rec_created_by/on`, `rec_modified_by/on` — **está na migration** |
| Precisa do histórico de versões, não só do último? | Tabela de histórico — muda bastante o desenho |
| Tem dado pessoal (LGPD)? | Log, mascaramento, retenção, quem pode consultar |
| Há prazo de retenção ou expurgo? | Job/rotina — normalmente fora do escopo, mas precisa ficar registrado |

### A7. Escopo e conclusão 🟡

| Pergunta | O que decide |
|---|---|
| O que é MVP e o que fica para depois? | Seção "Fora de Escopo" |
| Depende de alguma outra feature ficar pronta antes? | Ordem na esteira |
| Como saberemos que está funcionando? Quem valida? | Critérios de aceite — se não der para responder, o critério não é verificável |

---

## BLOCO B — Perguntas Técnicas: API (.NET / Clean Architecture)

### B1. Modelo de dados 🔴

- Quais entidades novas? Alguma entidade existente muda?
- Campo a campo: nome, tipo, tamanho, obrigatório?, default, valor inicial
- Qual o nome da tabela e das colunas em `snake_case`?
- Existe chave natural além do `Id` (SKU, código, CPF)?
- Datas: precisa de fuso? (`timestamp with time zone`)
- Decimais: quantas casas? (preço = `decimal(18,2)`, nunca `double`)

→ Decide: `Entities/<Feature>/<X>Entity.cs`, anotações e **a migration**.

### B2. Relacionamentos 🔴

- Se relaciona com que entidades? 1:N ou N:N?
- Excluir o pai faz o quê com o filho? (`Cascade`, `Restrict`, órfão)
- A listagem precisa trazer o relacionado junto? (`includes` — senão vira N+1)

### B3. Endpoints 🔴

- Quais operações a API precisa expor? (CRUD completo? só leitura? ação específica?)
- Rota e verbo de cada uma (`api/<recurso>` em kebab-case)
- O que entra e o que sai de cada uma? Um DTO só ou DTOs distintos por operação?
- Existe operação que não é CRUD? (importar, aprovar, duplicar → sub-rota explícita)
- Algum consumidor já usa uma rota parecida? Isso quebra contrato existente?

### B4. Erros 🔴

- Mapa completo: **regra violada → status → mensagem em português**
- Recurso não encontrado: 404 sempre? (se revelar a existência já é vazamento, 404 também no
  caso sem permissão)
- Erro de integração externa: o que a API devolve?

### B5. Consulta 🟡

- Precisa de filtro? Por quais campos? Combinados?
- Precisa de ordenação configurável?
- Precisa de paginação? (ver A5)
- Leitura pura → `asNoTracking: true`

### B6. Escrita e transação 🟡

- A operação grava em mais de uma tabela? → **um** `CommitAsync` no fim
- Existe operação em lote? Quantos itens por vez? Falha parcial: aborta tudo ou registra e segue?
- Pode haver duas pessoas editando o mesmo registro? Precisa de controle de concorrência?

### B7. Integrações 🟡

- Chama sistema externo? Qual, com que autenticação?
- Timeout aceitável? Tem retry? O que fazer se estiver fora do ar?
- É síncrono no request ou pode ser assíncrono?

### B8. Migration e dados existentes 🔴

- Tabela nova ou alteração em tabela com dados em produção?
- Campo novo obrigatório em tabela populada: qual o valor dos registros antigos? (**backfill**)
- Precisa de carga inicial (seed)?
- A migration roda em janela de manutenção ou a quente?

### B9. Configuração 🟡

- Precisa de chave nova em `appsettings` ou de segredo novo?
- O valor muda por ambiente?

### B10. Testes 🟡

- Que cenário de erro é mais provável na vida real? (esse ganha teste primeiro)
- Existe regra dependente de data, fuso ou arredondamento? (teste com valor não trivial)

---

## BLOCO C — Perguntas Técnicas: Frontend (Angular + PO-UI)

### C1. Telas e navegação 🟡

- Quantas telas? (listagem + formulário é o padrão)
- De onde o usuário chega? Precisa de item novo no menu?
- Depois de salvar, volta para a listagem ou continua no formulário?
- Precisa de tela de detalhe (só leitura) além do formulário?

### C2. Listagem 🟡

- Quais colunas aparecem, em que ordem?
- Formato de cada uma: texto, número, **moeda**, data, booleano, tag de status?
- Ordenação inicial da lista?
- Precisa de filtro? Busca por qual campo? Filtro avançado?
- Quantos registros em tela? (liga com A5)

### C3. Formulário 🔴 (na parte que espelha regra de negócio)

- Quais campos aparecem? Quais são editáveis na criação e quais na edição?
- Qual componente PO para cada um? (`po-input`, `po-decimal`, `po-select`, `po-combo`,
  `po-lookup`, `po-switch`, `po-datepicker`)
- Combos: as opções são fixas ou vêm de endpoint? Qual?
- Máscara ou formatação especial? (CPF/CNPJ, telefone, percentual)
- As validações da tela batem exatamente com as da API?

Bloqueante quando a resposta contradiz a API — validação frouxa na tela é irritação; validação
mais rígida que a API é bug funcional.

### C4. Ações 🟡

- Quais ações na tela e na linha da tabela?
- Quais são destrutivas? (essas **sempre** passam por `PoDialogService.confirm`)
- Alguma ação só aparece para certo perfil? Esconder ou desabilitar?

### C5. Feedback 🟡

- Qual a mensagem de sucesso de cada operação?
- Lista vazia: o que o usuário vê?
- Erro da API: já está resolvido pelo `errorInterceptor` — **nada a implementar por tela**

### C6. Não-funcionais da tela 🟢

- Precisa funcionar bem em telas pequenas?
- Tem requisito de acessibilidade explícito?
- Precisa de exportação (Excel/CSV)? Impressão?

---

## BLOCO D — Operação 🟢

- Como saberemos que quebrou em produção? (log, alerta, chamado)
- Alguém precisa de acesso a um relatório ou consulta de suporte?
- Precisa de feature flag para ligar/desligar?

---

## Defaults do projeto

Quando a pergunta é 🟡 e ninguém responde, assuma isto — e **escreva a premissa**:

| Tema | Default assumido |
|---|---|
| Paginação | Sem paginação até ~500 registros; acima, `PagedResponseContract` |
| Ordenação | Pelo campo de nome/código, ascendente |
| Exclusão | Físico (`DELETE`), a não ser que exista histórico ou referência de outra entidade |
| Auditoria | Sem campos `Rec*`, a não ser que a resposta de A6 seja "sim" |
| Permissão | Leitura `PortalLogPolicy`, escrita `AdminPolicy` |
| Registro novo | Nasce ativo |
| Texto sem tamanho informado | `MaxLength(200)` |
| Erro sem status definido | Validação de entrada 400; duplicidade 409; inexistente 404 |
| Depois de salvar | Toast de sucesso e volta para a listagem |
| Filtro da listagem | Busca por código e nome |
| Concorrência | Sem controle otimista |

Default é ponto de partida, não desculpa. Se o default estiver errado, o custo de corrigir é uma
migration — por isso ele nunca se aplica às perguntas 🔴.

---

## O mínimo, quando não há tempo

Doze perguntas que sozinhas permitem escrever um `feature.md` implementável:

1. Que problema resolve?
2. Quem pode ler e quem pode escrever?
3. Que campos o registro tem — nome, tipo, tamanho, obrigatório?
4. Que combinação é única?
5. O que nunca pode acontecer? (cada regra com seu status HTTP)
6. Pode ser alterado? Pode ser excluído — apaga ou inativa?
7. Precisa saber quem alterou e quando?
8. Quantos registros esperamos?
9. Quais operações a API precisa expor?
10. Quais telas, com quais campos e quais ações?
11. O que **não** entra nesta entrega?
12. Como saberemos que está funcionando?

---

## Como registrar no `feature.md`

Na seção **Perguntas e Premissas**:

```markdown
### Respondidas
- A2 (permissões): leitura por qualquer usuário do portal; escrita só Admin. — fonte: descrição do Jira
- A4 (unicidade): SKU único no catálogo → 409. — fonte: comentário do PO em 2026-08-20

### Premissas assumidas (🟡 sem resposta)
- A5 (volume): assumido < 500 produtos → sem paginação, filtro client-side.
  **Se passar disso, vira dívida:** paginar no backend.
- C1 (navegação): após salvar, volta para a listagem.

### Em aberto (🔴 bloqueantes)
- A3: excluir produto já vendido apaga ou inativa? — **bloqueia a migration**
```

Três seções, sempre — inclusive quando alguma estiver vazia ("nenhuma"). A ausência da seção é
ambígua; "nenhuma" é informação.
