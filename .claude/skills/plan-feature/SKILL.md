---
name: plan-feature
description: Transforma o feature.md de uma feature em um plano de implementação camada a camada (Domain → Application → Infrastructure → Api → Testes), com padrões do código real, comandos de validação e critérios de aceite rastreáveis. Use depois do /feature-from-jira e antes de escrever qualquer linha de código.
argument-hint: <CHAVE-JIRA> | <descrição da feature>
---

# Plan Feature — do requisito ao plano executável

## Entrada: $ARGUMENTS

- **Chave do Jira** (`PROJ-123`) → localize `features/PROJ-123-*/feature.md` e use como fonte.
  Se a pasta não existir, rode `/feature-from-jira PROJ-123` primeiro.
- **Descrição livre** → crie a pasta `features/SEM-JIRA-<slug>/` a partir do `_template/`,
  preenchendo o `feature.md` com o que der para extrair, e registre o que ficou em aberto.

## Princípio

**Nesta fase não se escreve código de produção.** O objetivo é um plano tão completo que o agente
de execução acerte na primeira passada, sem precisar pesquisar nem adivinhar.

Contexto é rei: o plano carrega os padrões, os caminhos de arquivo, as anotações, os nomes de
coluna e os comandos de validação — tudo mastigado.

---

## Fase 1 — Ler o requisito

1. Leia o `feature.md` **inteiro**.
2. Se houver pergunta em aberto que afete **modelo de dados ou contrato da API**, pare e pergunte
   ao usuário. Não planeje schema em cima de suposição.
3. Liste as regras de negócio (RN1..RNn) e os critérios de aceite (CA1..CAn) — cada um precisará
   de um destino no plano.

## Fase 2 — Ancorar no código real

Sempre carregue:
- `CLAUDE.md`
- `.claude/references/clean-architecture-dotnet.md`
- `.claude/references/dotnet-testing-standards.md`
- `.claude/references/backend-api-best-practices.md`

Depois investigue o código (use o agente `research-agent` em paralelo quando a feature tocar
várias áreas):

1. **Feature espelho** — ache a feature existente mais parecida e anote os caminhos exatos
   (`Services/X/XService.cs`, `Dtos/X/XDto.cs`, `Controllers/XController.cs`,
   `TesteUnitario/X/XServiceTests.cs`) com linhas de referência. Ela é o molde.
2. **Conflitos** — a entidade já existe? a rota já está tomada? o nome de service colide?
3. **Persistência** — o que o `IRepository` já oferece cobre as consultas necessárias? Se não,
   planeje a extensão do repositório como tarefa explícita (nunca `DbContext` no service).
4. **Auditoria e autorização** — a entidade precisa de `Rec*`? Quais policies em cada action?
5. **Migrations** — qual o nome da migration e há risco em tabela existente?

## Fase 3 — Decidir

Responda antes de escrever o plano:

- Uma entidade ou mais? Relacionamentos? Índices únicos?
- Um DTO só, ou DTOs distintos para entrada/saída/resumo?
- Onde cada RN é validada (service, sempre) e com qual status de erro?
- A listagem precisa de paginação (`PagedResponseContract`)?
- Existe operação em lote? Se sim, um único `CommitAsync` no fim.
- Alguma decisão contraria o padrão do projeto? Se sim, justifique no plano ou abandone-a.

## Fase 4 — Escrever o plano

Escreva em **`features/<CHAVE>-<slug>/plan.md`**, usando `features/_template/plan.md` como
estrutura. Preencha todas as seções:

1. **Resumo da solução** — 2-4 linhas.
2. **Contexto obrigatório** — arquivos a ler, com caminho e por quê; arquivos a criar; arquivos
   a modificar.
3. **Tarefas passo a passo** — na ordem canônica das camadas:

   ```
   Domain (entidade)
     → Application (DTO → mapper → interface → service)
       → DependencyInjection
         → Infrastructure (DbSet + índices + migration)
           → Api (controller)
             → Testes
   ```

   Cada tarefa tem: verbo (`CREATE`/`UPDATE`/`ADD`/`REFACTOR`), caminho exato do arquivo,
   **IMPLEMENTAR** (o que exatamente), **PADRÃO** (arquivo:linha a espelhar), **ATENÇÃO**
   (armadilha conhecida) e **VALIDAR** (comando executável e não interativo).

   Toda RN do `feature.md` precisa aparecer no **IMPLEMENTAR** de alguma tarefa, com o status de
   erro. Se uma RN não couber em nenhuma tarefa, o plano está incompleto.

4. **Estratégia de teste** — tabela método × cenário × esperado, cobrindo happy path, 404, cada
   RN e o `Verify` do `CommitAsync`.
5. **Comandos de validação** — os 5 níveis (build, teste da feature, suíte completa, format,
   manual via Swagger), com o caminho real da solução.
6. **Critérios de aceite** — copiados do `feature.md`, cada um com **como será verificado**
   (nome do teste ou chamada HTTP).
7. **Riscos e decisões** — inclusive as alternativas descartadas e o porquê.

## Fase 5 — Atualizar o estado

- Marque a feature como `📋 planejada` em `features/INDEX.md`.
- Atualize a linha **Estado na esteira** no `feature.md`.

---

## Critérios de qualidade do plano

- [ ] Passa no teste "sem conhecimento prévio": alguém que nunca viu o repositório consegue
      executar só com o plano
- [ ] Toda tarefa tem um comando de validação executável
- [ ] Toda referência de padrão aponta para arquivo (e linha quando útil) que existe de fato
- [ ] Toda RN e todo CA do `feature.md` têm destino rastreável
- [ ] Nenhuma tarefa inventa padrão novo sem justificativa escrita
- [ ] A ordem das tarefas respeita a regra de dependência das camadas

## Saída

Ao terminar, reporte:

- Resumo da feature e da abordagem
- Caminho do `plan.md` criado
- Complexidade (Baixa/Média/Alta) e número de tarefas
- Riscos principais
- **Nota de confiança /10** de acerto em uma passada
- Próximo passo: `/execute <CHAVE>`
