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

### Impacto nas Camadas

- **Domain**: {entidades/enums novos ou alterados}
- **Application**: {DTOs, mappers, services}
- **Infrastructure**: {DbSet, índices, migration, integrações}
- **Api**: {controller, policies, rotas}
- **Contracts**: {contratos públicos, paginação}
- **Testes**: {classes de teste}

## Fora de Escopo

- {o que explicitamente NÃO será feito nesta feature}

## Dependências

- {Chave de outra feature que precisa vir antes, ou "nenhuma"}

## Perguntas em Aberto

- [ ] {ambiguidade que precisa de resposta do PO antes de planejar}

> Se houver pergunta em aberto que muda o modelo de dados ou o contrato da API, a feature entra
> como ⛔ bloqueada — não planeje em cima de suposição sobre schema ou contrato.
