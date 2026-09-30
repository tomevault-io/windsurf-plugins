---
trigger: always_on
description: Descreve um agente uma única vez, com as suas instruções, o seu modelo e as suas ferramentas, e o Pepe trata do resto, chamando o modelo e a executar ferramentas até ter uma resposta a sério.
---


## O que é um agente

Um agente resume-se a uma descrição curta que escreves uma vez: o nome, as
instruções (o prompt de sistema que lhe dá uma persona), o modelo com que pensa e a
lista de ferramentas que tem permissão para chamar. Um punhado de opções adicionais,
um limite de iterações, uma temperatura, quem pode contactar, quem pode administrar,
fecham o conjunto. É só isto. O próprio agente não guarda nenhuma lógica: quem faz o
trabalho é o Pepe, chamando o modelo, executando as ferramentas que este pede,
devolvendo-lhe os resultados e repetindo o processo até surgir uma resposta final.

Cada agente existe como uma entrada dentro de um único ficheiro JSON,
`~/.pepe/config.json`. Não há nenhuma base de dados por trás disto. Há três formas de
criar e editar agentes, e todas acabam por escrever no mesmo ficheiro:

1. A ferramenta de linha de comandos `pepe`.
2. O painel web.
3. Uma conversa normal, falando com um agente que já tenha a ferramenta de gestão
   correspondente.

Assim fica um agente completo, tal como é guardado em disco:

```json
{
  "agents": {
    "assistant": {
      "description": "General-purpose helper",
      "model": "openrouter",
      "system_prompt": "És um assistente prestável e direto.",
      "tools": ["bash", "read_file", "write_file", "web_search"],
      "auto_approve": [],
      "can_message": [],
      "can_manage": null,
      "hooks": [],
      "max_iterations": 12,
      "temperature": null
    }
  }
}
```

## O teu primeiro agente

Antes de conseguir pensar, um agente precisa de uma ligação de modelo. Se ainda não
tens nenhuma criada, a configuração guiada leva-te pela mão a escolher um
fornecedor, a iniciar sessão e a selecionar um modelo:

```bash
pepe setup
```

A seguir, define um agente com um prompt e algumas ferramentas:

```bash
pepe agent add assistant \
  --model openrouter \
  --prompt "És um assistente prestável e direto." \
  --tools bash,read_file,write_file,web_search
```

Corre um prompt avulso contra ele. A resposta vai sendo transmitida para o teu
terminal à medida que é produzida:

```bash
pepe run assistant "Que ficheiros existem no diretório atual?"
```

Só este comando já dispara o ciclo completo: o agente percebe que precisa de
espreitar o sistema de ficheiros, chama a ferramenta `list_dir` ou `bash`, lê o
resultado e devolve-te a resposta em linguagem corrente.

<div class="note"><strong>A partir do painel.</strong> A secção de Agentes do painel
web faz exatamente o mesmo através de um formulário: nome, persona, modelo, uma
lista de ferramentas para marcar e o âmbito de administração. O que escreve em
<code>~/.pepe/config.json</code> é a mesma entrada, por isso podes alternar
livremente entre a CLI, o painel e a edição manual do ficheiro.</div>

### Fazer isto por chat

Qualquer agente com a ferramenta `manage_agent` consegue criar e configurar outros
agentes através de conversa. É assim que o primeiro agente de todos (ver "O agente
proprietário" mais abaixo) te deixa construir o resto da tua frota sem nunca tocar
na CLI. Uma mensagem como esta:

```text
Cria um agente chamado researcher. Dá-lhe uma persona focada em pesquisa
cuidadosa na web, aponta-o ao modelo openrouter e ativa web_search e
fetch_url.
```

faz o agente chamar `manage_agent` com `action: "create"` e, a seguir,
`set_persona`, `set_model` e `add_tool` para cada capacidade pedida. O `manage_agent`
é uma ferramenta de risco, por isso passa sempre pela barreira de permissão: numa
superfície onde há alguém a quem perguntar (a consola, um canal de chat), o runtime
pede-te para autorizares a alteração antes de a escrever, e a própria ferramenta
está instruída a confirmar contigo o plano primeiro. Um agente só consegue gerir os
agentes dentro do seu âmbito `can_manage` (explicado mais abaixo, em
[Administrar agentes](#administrar-agentes)); pedir-lhe para mexer nalgum que esteja
fora desse âmbito é simplesmente recusado, com educação.

## Os campos, um a um

| Campo | O que faz | Predefinição |
|-------|--------------|---------|
| `name` | O rótulo pelo qual o agente é endereçado. Dentro de um projeto passa a um identificador como `acme/assistant` (ver abaixo). O agente também guarda um id interno estável, por isso mudar este nome nunca quebra nenhuma ligação já existente. | obrigatório |
| `description` | Uma nota curta, só para humanos lerem. Nunca é enviada ao modelo. | nenhuma |
| `model` | O nome de uma ligação de modelo. Deixa por preencher para usar o modelo predefinido do projeto. | predefinição do projeto |
| `system_prompt` | A persona e as instruções com que o agente corre. | `És o Pepe, um agente de IA prestável.` (um prompt semente) |
| `langfuse_prompt` | Vai buscar a persona ao prompt deste nome no [Langfuse](#gerir-uma-persona-a-partir-do-langfuse), em vez de usar `system_prompt`. | `null` (desligado) |
| `tools` | A lista de ferramentas que este agente pode chamar. Só estas chegam a ser oferecidas ao modelo. | todas as ferramentas: um agente novo nasce com tudo ligado, e és tu quem retira o que não quer |
| `auto_approve` | Ferramentas que este agente pode correr sem pedir autorização. `["*"]` significa todas. | `[]` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pepe-agent/pepe](https://github.com/pepe-agent/pepe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
