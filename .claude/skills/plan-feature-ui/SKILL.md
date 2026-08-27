---
name: plan-feature-ui
description: Transforma o escopo de tela de uma feature em um plano de implementação Angular + PO-UI (model → service → rotas → listagem → formulário → testes), com os componentes PO escolhidos, os endpoints consumidos e os comandos de validação. Use depois do /plan-feature (API) ou isolado, quando a feature for só de tela.
argument-hint: <CHAVE-JIRA> | <descrição da tela>
---

# Plan Feature UI — do requisito ao plano da tela

## Entrada: $ARGUMENTS

- **Chave do Jira** → localize `features/<CHAVE>-*/feature.md` e use a seção
  **Escopo Frontend** como fonte. Sem a pasta, rode `/feature-from-jira <CHAVE>` antes.
- **Descrição livre** → crie a pasta a partir do `_template/`, preenchendo o que der.

## Princípio

**Nesta fase não se escreve componente.** O objetivo é um plano que permita ao agente de execução
criar a tela na primeira passada, com os componentes PO já escolhidos e os contratos de API já
confirmados.

---

## Fase 1 — Ler o requisito de tela

Do `feature.md`:
- Telas e rotas pedidas
- Campos exibidos e editáveis; quais são obrigatórios
- Ações do usuário (criar, editar, excluir, filtrar, exportar)
- Regras de visibilidade e permissão
- Mensagens de sucesso e de erro esperadas

Se houver mockup ou protótipo anexado no Jira/Confluence, ele manda no layout.

Confira contra o **Bloco C** de `.claude/references/feature-discovery-questions.md`: telas e
navegação, colunas e formato da listagem, campos e componente PO de cada um, ações destrutivas,
mensagens de sucesso e estado vazio. O que não estiver respondido ali e não tiver default entra
como premissa registrada, não como invenção silenciosa.

**Pergunta aberta sobre navegação ou sobre qual campo é editável não bloqueia — assuma o padrão
do app e registre.** Pergunta aberta sobre **qual endpoint existe** bloqueia: confirme antes.

## Fase 2 — Confirmar o contrato da API

Este é o passo que mais evita retrabalho. Para cada endpoint que a tela vai consumir:

| Verificar | Onde |
|---|---|
| Rota e verbo reais | `<Sln>.Api/Controllers/<X>Controller.cs` |
| Campos e tipos do payload | `<Sln>.Application/Dtos/<Feature>/<X>Dto.cs` |
| Formato da resposta | array puro? `ResponseContract`? `PagedResponseContract`? |
| Policy exigida | atributo `[Authorize(...)]` da action |
| Erros possíveis e status | regras do service (sufixo ` 400`/` 404`/` 409`) |

O model TypeScript **espelha o DTO em camelCase**. Divergiu do DTO? O plano está errado, não o
backend.

Se a API ainda não existe (feature fullstack sendo construída junto), o plano da tela depende do
`plan.md` da API — cite-o e mantenha os contratos idênticos.

## Fase 3 — Ancorar no código do app

Sempre carregue:
- `CLAUDE.md`
- `.claude/references/angular-po-ui.md`
- `.claude/references/angular-testing-standards.md`

E investigue:
1. **Feature espelho** — a tela existente mais parecida; anote os caminhos exatos. É o molde.
2. **Módulos PO em uso** — granulares (`PoTableModule`) ou `PoModule`? Siga o que já existe.
3. **Conflitos** — a rota já existe? já há um service para essa entidade?
4. **Shared** — algum componente/pipe reutilizável já resolve parte da tela?
5. **Menu** — a feature precisa de item novo em `PoMenuItem[]`?

## Fase 4 — Escolher os componentes PO

Decida e **justifique** no plano:

- Listagem: `po-page-list` + `po-table` (padrão) ou `po-page-dynamic-table`?
  → Dinâmico exige o contrato `{ items, hasNext }` do PO. Se a API não responde assim, **não use**
  sem uma adaptação explícita e planejada.
- Colunas: `property`, `label`, `type` (`string`/`number`/`currency`/`date`/`boolean`/`label`),
  `format`, `width`.
- Formulário: `po-page-edit` + reactive forms; qual componente para cada campo
  (`po-input`, `po-decimal`, `po-select`, `po-combo`, `po-lookup`, `po-switch`, `po-datepicker`).
- Ação destrutiva → `PoDialogService.confirm`.
- Feedback de sucesso → `PoNotificationService.success`.
- Erro → **nada no componente**: o interceptor já notifica.

## Fase 5 — Escrever o plano

Escreva em **`features/<CHAVE>-<slug>/plan-ui.md`**, usando `features/_template/plan-ui.md`.
Ordem canônica das tarefas:

```
model → service → rotas → componente de listagem → componente de formulário
      → item de menu → testes
```

Cada tarefa com: verbo (`CREATE`/`UPDATE`), caminho exato, **IMPLEMENTAR**, **PADRÃO**
(arquivo a espelhar), **ATENÇÃO** (armadilha) e **VALIDAR** (comando executável).

Inclua obrigatoriamente:
- Tabela **Campo → componente PO → validação** (espelhando as validações da API)
- Tabela **Endpoint → método do service → tela que usa**
- Mapa **erro da API → o que o usuário vê**
- Estratégia de teste (service, listagem, formulário)
- Comandos de validação (`npm run build`, `npx tsc --noEmit`, `npm test`, `npx ng lint`)
- Critérios de aceite de tela, cada um com **como será verificado**

## Fase 6 — Atualizar o estado

- `features/INDEX.md`: marque que a feature tem plano de UI (`📋 planejada (API+UI)`).
- `feature.md`: atualize a linha de estado.

---

## Critérios de qualidade

- [ ] Todo endpoint consumido foi **confirmado no controller**, não presumido
- [ ] Todo campo da tela tem componente PO escolhido e validação definida
- [ ] Toda ação do usuário tem destino (rota, service, diálogo)
- [ ] Todo erro previsto da API tem comportamento de tela definido
- [ ] Toda tarefa tem comando de validação executável
- [ ] Nenhum componente PO usado fora do que a versão instalada oferece

## Saída

- Resumo da tela e da abordagem
- Caminho do `plan-ui.md`
- Componentes PO escolhidos e por quê
- Endpoints consumidos, com o formato de resposta confirmado
- Riscos (contrato do PO dinâmico, campo sem endpoint, permissão indefinida)
- **Nota de confiança /10**
- Próximo passo: `/execute-ui <CHAVE>`
