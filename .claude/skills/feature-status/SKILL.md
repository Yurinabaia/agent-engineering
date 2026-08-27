---
name: feature-status
description: Mostra o estado da esteira de features — o que está importado, planejado, em execução, em revisão ou bloqueado — e diz qual é o próximo comando de cada uma. Use para retomar o trabalho ou abrir a apresentação.
argument-hint: [CHAVE-JIRA]
---

# Feature Status — onde cada feature está

## Sem argumento: visão geral

1. Leia `features/INDEX.md`.
2. Liste as pastas em `features/` (ignorando `_template/`) e cheque se alguma não está no índice —
   se estiver faltando, acrescente.
3. Para cada feature, confirme o estado pelos artefatos, não pelo que o índice afirma:

   | Existe | Estado real |
   |---|---|
   | só `feature.md` | 📥 importada |
   | `plan.md` sem item marcado no `checklist.md` | 📋 planejada |
   | `checklist.md` com itens marcados e outros não | ⚙️ em execução |
   | checklist completo, sem commit | 🔍 em revisão |
   | checklist completo + commit com a chave | ✅ concluída |
   | pergunta em aberto não resolvida no `feature.md` | ⛔ bloqueada |

4. Corrija o `INDEX.md` onde ele divergir da realidade.

Verifique commits da feature com:

```bash
git log --oneline --grep="<CHAVE>"
```

## Com uma chave: detalhe da feature

Mostre:
- Cabeçalho do `feature.md` (título, tipo, épico, estado)
- Critérios de aceite: quantos marcados de quantos
- Checklist por camada: quais camadas fechadas, qual está aberta
- Divergências registradas no checklist
- Último commit relacionado

## Saída

### Esteira

| Chave | Título | Estado | Próximo comando |
|---|---|---|---|
| ... | ... | ... | `/plan-feature ...` |

### Atenção
- Features bloqueadas e o que falta responder
- Features em execução paradas no meio (qual camada retomar)
- Divergências entre `INDEX.md` e os artefatos, já corrigidas

### Sugestão
Qual feature retomar primeiro e com qual comando.
