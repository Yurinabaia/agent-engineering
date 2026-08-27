# DEMO-101 — Cadastro de Produtos

| | |
|---|---|
| **Jira** | DEMO-101 *(exemplo da apresentação — issue fictícia)* |
| **Tipo** | Story |
| **Épico** | DEMO-100 — Catálogo |
| **Prioridade** | Alta |
| **Status Jira** | To Do |
| **Estado na esteira** | 📥 importada |
| **Importada em** | 2026-08-27 |

---

## Objetivo

Permitir que o time de catálogo cadastre, consulte e mantenha os produtos vendidos, hoje
controlados em planilha.

## História de Usuário

Como analista de catálogo
Quero cadastrar e manter produtos pela API
Para que o catálogo tenha uma fonte única e auditável

## Contexto de Negócio

Os produtos são identificados por um **SKU** único (texto, até 30 caracteres, maiúsculas).
Cada produto tem nome, preço e situação ativo/inativo. Produto nunca é apagado fisicamente do
catálogo quando já foi vendido — mas nesta primeira entrega o `DELETE` remove o registro, e a
regra de "não excluir vendido" fica para a integração com Vendas (fora de escopo).

Toda alteração precisa registrar quem fez e quando, porque o catálogo é auditado
trimestralmente.

## Regras de Negócio

- **RN1**: SKU é único no catálogo. Tentar criar SKU existente → erro **409**.
- **RN2**: SKU é armazenado sempre em maiúsculas, sem espaços nas pontas.
- **RN3**: Preço deve ser maior que zero → caso contrário, erro **400**.
- **RN4**: Nome é obrigatório e tem no máximo 200 caracteres → erro **400**.
- **RN5**: Produto novo nasce **ativo**.
- **RN6**: Consultar/alterar/excluir id inexistente → erro **404**.
- **RN7**: `GET` de listagem retorna apenas produtos ativos por padrão; `?incluirInativos=true`
  traz todos.

## Critérios de Aceite

- [ ] CA1: `POST api/produtos` com payload válido cria o produto e retorna **201** com o id.
- [ ] CA2: `POST api/produtos` com SKU já existente retorna **409** com mensagem em português.
- [ ] CA3: `POST api/produtos` com preço zero ou negativo retorna **400**.
- [ ] CA4: `GET api/produtos` retorna apenas ativos; com `?incluirInativos=true` retorna todos.
- [ ] CA5: `GET api/produtos/{id}` inexistente retorna **404**.
- [ ] CA6: `PUT api/produtos/{id}` atualiza nome, preço e situação e retorna **200**.
- [ ] CA7: `DELETE api/produtos/{id}` retorna **204** e o produto some da listagem.
- [ ] CA8: Criação e alteração gravam usuário e data nos campos de auditoria.
- [ ] CA9: Leitura exige `PortalLogPolicy`; escrita exige `AdminPolicy`.

## Escopo Técnico

### Endpoints

| Método | Rota | Policy | Descrição |
|---|---|---|---|
| GET | `api/produtos` | PortalLogPolicy | Lista produtos (`?incluirInativos=true` opcional) |
| GET | `api/produtos/{id:int}` | PortalLogPolicy | Obtém produto por id |
| POST | `api/produtos` | AdminPolicy | Cria produto |
| PUT | `api/produtos/{id:int}` | AdminPolicy | Atualiza produto |
| DELETE | `api/produtos/{id:int}` | AdminPolicy | Remove produto |

### Modelo de Dados

Tabela: `produtos` — com auditoria.

| Campo | Tipo | Coluna | Obrigatório | Observação |
|---|---|---|---|---|
| Id | int | `id` | sim | identity |
| Sku | string(30) | `sku` | sim | índice único `uq_produtos_sku` |
| Nome | string(200) | `nome` | sim | |
| Preco | decimal(18,2) | `preco` | sim | > 0 |
| Ativo | bool | `ativo` | sim | default `true` |
| RecCreatedBy | string(50) | `rec_created_by` | sim | e-mail do portal |
| RecCreatedOn | DateTime | `rec_created_on` | sim | `timestamp with time zone` |
| RecModifiedBy | string(50) | `rec_modified_by` | sim | |
| RecModifiedOn | DateTime | `rec_modified_on` | sim | |

### Escopo Frontend (Angular + PO-UI)

#### Telas e Rotas

| Tela | Rota | Descrição | Permissão |
|---|---|---|---|
| Listagem de produtos | `/produtos` | Lista, filtra por SKU/nome, ações de editar e excluir | Portal |
| Cadastro de produto | `/produtos/novo` | Formulário de criação | Admin |
| Edição de produto | `/produtos/:id` | Formulário de edição | Admin |

#### Campos da Tela

| Campo | Componente PO | Obrigatório | Editável | Observação |
|---|---|---|---|---|
| SKU | `po-input` | sim | só na criação | `p-maxlength="30"`, exibido em maiúsculas |
| Nome | `po-input` | sim | sim | `p-maxlength="200"` |
| Preço | `po-decimal` | sim | sim | `p-decimals-length="2"`, mínimo 0,01 |
| Situação | `po-switch` | não | sim | rótulos "Ativo" / "Inativo" |

Colunas da listagem: SKU (`string`), Nome (`string`), Preço (`type: 'currency'`, `format: 'BRL'`),
Situação (`type: 'boolean'` com `trueLabel: 'Ativo'` / `falseLabel: 'Inativo'`).

#### Ações do Usuário

- **Novo produto** (`PoPageAction`) → navega para `/produtos/novo`
- **Editar** (`PoTableAction`) → navega para `/produtos/:id`
- **Excluir** (`PoTableAction`, `type: 'danger'`) → `PoDialogService.confirm` antes de chamar a API;
  ao concluir, toast de sucesso e recarga da lista
- **Filtrar** (`PoPageFilter`) → busca por SKU ou nome
- **Salvar** → toast "Produto salvo com sucesso." e volta para a listagem

#### Endpoints Consumidos

| Método | Rota | Usado em |
|---|---|---|
| GET | `api/produtos` | listagem (`?incluirInativos=true` no filtro avançado) |
| GET | `api/produtos/{id}` | formulário em modo edição |
| POST | `api/produtos` | formulário em modo criação |
| PUT | `api/produtos/{id}` | formulário em modo edição |
| DELETE | `api/produtos/{id}` | ação de excluir da listagem |

#### Critérios de Aceite de Tela

- [ ] CAT1: A listagem carrega ao abrir `/produtos` e exibe preço formatado em Real e situação
      como "Ativo"/"Inativo".
- [ ] CAT2: O botão "Novo produto" leva ao formulário vazio; salvar cria e volta para a listagem
      com toast de sucesso.
- [ ] CAT3: Salvar com SKU duplicado exibe o toast de erro com a mensagem vinda da API (409),
      sem sair do formulário.
- [ ] CAT4: Salvar com preço zero é bloqueado pela validação do formulário (botão desabilitado).
- [ ] CAT5: Excluir pede confirmação; ao confirmar, o produto some da lista com toast de sucesso.
- [ ] CAT6: Com a API indisponível, o usuário vê "Não foi possível conectar ao servidor." e a tela
      não fica travada em loading.
- [ ] CAT7: O filtro busca por SKU ou nome.

### Impacto nas Camadas

- **Domain**: `Entities/Produtos/ProdutoEntity.cs`
- **Application**: `Dtos/Produtos/ProdutoDto.cs`, `Extensions/Produtos/ProdutoExtensions.cs`,
  `Services/Produtos/IProdutoService.cs` + `ProdutoService.cs`, registro no `DependencyInjection`
- **Infrastructure**: `DbSet<ProdutoEntity>`, índice único no `OnModelCreating`, migration `AddProdutos`
- **Api**: `Controllers/ProdutosController.cs`
- **Contracts**: nenhum contrato novo nesta entrega (sem paginação — catálogo pequeno)
- **Testes (API)**: `TesteUnitario/Produtos/ProdutoServiceTests.cs`
- **Frontend**: `src/app/features/produtos/` — `models/produto.model.ts`,
  `services/produto.service.ts`, `produtos.routes.ts`, `pages/produto-list/`,
  `pages/produto-form/`, item de menu "Produtos"
- **Testes (UI)**: `produto.service.spec.ts`, `produto-list.component.spec.ts`,
  `produto-form.component.spec.ts`

## Fora de Escopo

- Integração com Vendas (bloqueio de exclusão de produto vendido)
- Importação em lote via planilha
- Imagens do produto
- Paginação no backend (a listagem é carregada inteira; o filtro é client-side)
- Exportação para Excel/CSV
- Tela responsiva para mobile (o PO já entrega o básico; ajuste fino fica fora)

## Dependências

Nenhuma.

## Perguntas em Aberto

Nenhuma — feature de demonstração, requisitos fechados.
