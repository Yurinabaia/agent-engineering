# {CHAVE-JIRA} — {Título da Feature}

| | |
|---|---|
| **Jira** | [{CHAVE-JIRA}]({url-da-issue}) |
| **Tipo** | Story / Task / Bug |
| **Épico** | {chave e nome do épico, ou "—"} |
| **Prioridade** | {prioridade} |
| **Status Jira** | {status} |
| **Estado na esteira** | 📥 importada |
| **Importada em** | {AAAA-MM-DD} |

---

## Objetivo

{Uma frase: que problema de negócio esta feature resolve.}

## História de Usuário

Como {tipo de usuário}
Quero {ação/objetivo}
Para que {benefício}

## Contexto de Negócio

{Destilação da descrição do Jira. Somente o que muda uma decisão de código.
Anexos, mockups e exemplos de payload relevantes entram aqui.}

## Regras de Negócio

- RN1: {regra} → status de erro esperado: {400/409/...}
- RN2: {regra}

## Critérios de Aceite

- [ ] CA1: {critério verificável}
- [ ] CA2: {critério verificável}

## Escopo Técnico (preenchido na importação, refinado no plano)

### Endpoints

| Método | Rota | Policy | Descrição |
|---|---|---|---|
| GET | `api/{recurso}` | PortalLogPolicy | Lista |
| POST | `api/{recurso}` | AdminPolicy | Cria |

### Modelo de Dados

| Campo | Tipo | Coluna | Obrigatório | Observação |
|---|---|---|---|---|
| Id | int | `id` | sim | identity |
| {campo} | {tipo} | `{snake_case}` | sim/não | {MaxLength, default, índice} |

Tabela: `{snake_case}` — Precisa de auditoria (`rec_created_by/on`, `rec_modified_by/on`)?
{sim/não}

### Escopo Frontend (Angular + PO-UI)

> Preencha apenas se a feature tiver tela. Só backend? Escreva "Sem escopo de tela" e siga.

#### Telas e Rotas

| Tela | Rota | Descrição | Permissão |
|---|---|---|---|
| Listagem de {recurso} | `/{recurso}` | Lista, filtra e dá acesso às ações | {perfil} |
| Cadastro/Edição | `/{recurso}/novo` e `/{recurso}/:id` | Formulário | {perfil} |

#### Campos da Tela

| Campo | Componente PO | Obrigatório | Editável | Observação |
|---|---|---|---|---|
| {campo} | `po-input` / `po-decimal` / `po-select` / `po-switch` | sim/não | sim/não | {máscara, limite, opções} |

#### Ações do Usuário

- {ação} → {resultado esperado, mensagem de sucesso}
- {ação destrutiva} → exige confirmação (`PoDialogService.confirm`)

#### Endpoints Consumidos

| Método | Rota | Usado em |
|---|---|---|
| GET | `api/{recurso}` | listagem |
| POST | `api/{recurso}` | formulário (novo) |

#### Critérios de Aceite de Tela

- [ ] CAT1: {critério verificável na interface}
- [ ] CAT2: {mensagem de erro da API aparece como toast}

### Impacto nas Camadas

- **Domain**: {entidades/enums novos ou alterados}
- **Application**: {DTOs, mappers, services}
- **Infrastructure**: {DbSet, índices, migration, integrações}
- **Api**: {controller, policies, rotas}
- **Contracts**: {contratos públicos, paginação}
- **Testes (API)**: {classes de teste}
- **Frontend**: {feature Angular, telas, service, rota lazy, item de menu}
- **Testes (UI)**: {specs de service, listagem e formulário}

## Fora de Escopo

- {o que explicitamente NÃO será feito nesta feature}

## Dependências

- {Chave de outra feature que precisa vir antes, ou "nenhuma"}

## Perguntas e Premissas

Levantadas com `.claude/references/feature-discovery-questions.md`.
As três listas são obrigatórias — escreva "nenhuma" em vez de omitir a seção.

### Respondidas

| Ref | Pergunta | Resposta | Fonte |
|---|---|---|---|
| A2 | {quem pode ler/escrever} | {resposta} | descrição do Jira |
| B1 | {tipo e tamanho de campo} | {resposta} | comentário do PO / código existente |

### Premissas assumidas (🟡 — sem resposta, default aplicado)

| Ref | Premissa | Base | Custo se estiver errada |
|---|---|---|---|
| A5 | {ex.: menos de 500 registros → sem paginação} | default do projeto | refazer o endpoint com paginação |

### Em aberto

| Ref | Pergunta | Classe | O que muda no código |
|---|---|---|---|
| A3 | {ex.: excluir apaga ou inativa?} | 🔴 | campo `ativo` + migration |
| C6 | {ex.: precisa exportar para Excel?} | 🟢 | ação extra na listagem |

> Enquanto houver 🔴 em aberto, a feature fica ⛔ **bloqueada** — não se planeja schema nem
> contrato de API em cima de suposição. 🟡 e 🟢 não bloqueiam: assuma, registre e siga.
> Para aprofundar o levantamento: `/feature-questions <CHAVE>`.
