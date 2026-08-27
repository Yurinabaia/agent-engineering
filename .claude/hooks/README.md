# Hooks — a camada determinística da camada de IA

Hooks são a quinta primitiva. Regras, subagentes, ferramentas e skills são coisas que o agente
*escolhe usar*. O hook é a única que o agente nunca escolhe: ele dispara sozinho, num evento do
ciclo de vida.

> **Uma regra pede ao agente que se comporte. Um hook garante.**
>
> Use regra quando "na maioria das vezes" resolve. Use hook quando aquilo **precisa** acontecer
> todas as vezes.

## O que vem aqui

| Arquivo | Evento | O que faz | Bloqueia? |
|---|---|---|---|
| `pre_tool_use.py` | **PreToolUse** | Bloqueia ler/escrever/buscar um arquivo de env real (templates `.env.example` versionados são permitidos), bloqueia `rm -rf` e bloqueia escrever à mão uma migration **nova** do EF Core (artefato gerado tem de vir do `dotnet ef migrations add`; editar migration existente continua permitido). Imprime o motivo no stderr e `exit(2)` → a ferramenta é interrompida e o agente recebe a razão, então ele se adapta. | **Sim** — é aqui que mora a garantia |
| `post_tool_use.py` | **PostToolUse** | Registra cada chamada de ferramenta em `logs/post_tool_use.json` — uma trilha de auditoria completa do que o agente fez. | Não — a ferramenta já rodou; só observa |

Essa divisão *é* o modelo mental: **pre = portão, post = registro.** Esta é, deliberadamente, toda
a camada de hooks que o pacote entrega — segurança sempre ligada e genérica para qualquer
codebase. Qualquer coisa mais específica do *seu* fluxo (um verificador que trava a conclusão, a
passagem de um artefato entre duas skills) está a uma conversa de distância com o seu agente, não
a um arquivo para copiar — veja abaixo.

### Códigos de saída — a única coisa que você precisa acertar

**Só `exit 2` bloqueia.** Não é o exit 1, que é o que todo linter, type checker e runner de teste
devolve ao falhar. Converter um no outro é boa parte do que um hook de portão faz.

| Exit | Significado |
|---|---|
| `0` | sucesso. O stdout vai para o log de debug (exceto em `UserPromptSubmit` / `SessionStart`, onde vira contexto) |
| **`2`** | **erro bloqueante.** O stderr é entregue ao agente como motivo |
| qualquer outro | erro não bloqueante: ruído, sem efeito |

O que o `exit 2` bloqueia depende do evento:

| Evento | O `exit 2` bloqueia? |
|---|---|
| `PreToolUse` | **sim** — a chamada da ferramenta é interrompida (é disso que o `pre_tool_use.py` depende) |
| `Stop` | **sim** — a sessão é impedida de encerrar |
| `PostToolUse` | **não** — a ferramenta já rodou; o stderr é apenas exibido, por isso o `post_tool_use.py` sempre sai com 0 |

## Como ligar

Hooks são a única primitiva que passa a agir no instante em que existe, então o pacote entrega a
configuração como **template**, não como arquivo ativo:

```bash
cp .claude/settings.json.example .claude/settings.json
```

Aqui no pacote o `.claude/settings.json` está no gitignore, para os hooks não dispararem enquanto
você lê o material. **No seu projeto, versione ele** — é assim que o time inteiro herda as mesmas
garantias. Se você já tem um `settings.json`, faça o merge do bloco `hooks` em vez de sobrescrever.

## Experimente

```
peça ao agente para ler seu arquivo de env       -> bloqueado (exit 2, com o motivo devolvido)
peça para ler um .env.example versionado         -> permitido
peça para rodar `rm -rf ...`                     -> bloqueado
peça para criar uma migration à mão              -> bloqueado (com o comando dotnet ef na mensagem)
qualquer comando normal                          -> permitido, e registrado em logs/post_tool_use.json
```

Você também pode testar um hook direto, sem o agente:

```bash
echo '{"tool_name":"Read","tool_input":{"file_path":".env"}}' | uv run .claude/hooks/pre_tool_use.py; echo "exit=$?"
```

`exit=2` significa que a guarda disparou.

## Escrevendo os seus — você não precisa escrever Python à mão, e este pacote não traz exemplos de propósito

Os dois hooks acima são deliberadamente genéricos — defaults seguros para qualquer codebase. No
momento em que você quer algo específico do *seu* fluxo (travar a conclusão nas suas verificações,
passar um artefato de uma skill para a próxima, formatar automaticamente ao editar), isso não é
algo que um pacote genérico consiga entregar pronto — depende dos seus comandos, dos seus
caminhos, das suas verificações. Descreva para o seu agente, em português mesmo:

> "Não me deixe encerrar enquanto meus testes não passarem — rode `dotnet test` quando eu tentar
> parar e, se falhar, bloqueie e me diga por quê."

O Claude Code (e a maioria dos agentes de código atuais) sabe escolher o evento do ciclo de vida
certo, escrever o script e conectá-lo no `settings.json` a partir de uma descrição dessas — sem
você escrever JSON de hook na mão. Formatos comuns que vale conhecer pelo nome antes de pedir:

- **reagir** (`PostToolUse`) — algo dispara porque um arquivo mudou. Formatar ao editar é o caso
  clássico.
- **portão** (`Stop`) — o agente não pode se declarar pronto enquanto suas verificações estiverem
  vermelhas. **Coloque um limite você mesmo** — um hook não tem memória, então, se ele puder
  tentar de novo, dê a ele um contador de tentativas em disco. Não confie em uma flag não
  documentada para limitar o laço; verifique o que acontece de fato antes de depender disso.
- **bastão** — o artefato de uma skill é gravado e o hook inicia a skill seguinte em um contexto
  **novo**. Condicione ao estado dos arquivos (entrada presente, saída ausente), nunca à memória —
  é essa condição que torna seguro disparar cem vezes e agir exatamente uma.

## Duas coisas para saber

- **Hooks executam código real, automaticamente, com as suas credenciais, sem sandbox.** Revise um
  hook como você revisaria um script de CI. Só rode hooks que você leu e em que confia. É o mesmo
  cuidado que se tem com servidores MCP.
- **A cobertura é sua.** O hook tem garantia de *rodar*; o que ele *pega* é só tão bom quanto a
  verificação que você escreveu. Ele é o ponto de aplicação, não onisciência.

## Portabilidade

Isto não é truque de festa do Claude Code. Codex e Cursor usam o mesmo formato (um script, JSON no
stdin, `exit 2` para bloquear); o Gemini CLI faz o mesmo trabalho lendo uma decisão em JSON
estruturado em vez do código de saída; Pi e opencode rodam hooks em processo, como plugins.
Aprendeu uma vez, vale para os outros — do mesmo jeito que o `AGENTS.md` virou o arquivo de regras
compartilhado.
