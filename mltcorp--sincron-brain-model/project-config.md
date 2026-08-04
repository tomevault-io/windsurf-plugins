---
trigger: always_on
description: Modelo de memória plug-and-play para agentes de IA. Distribuído como MCP server.
---

# Sincron Brain Model

Modelo de memória plug-and-play para agentes de IA. Distribuído como MCP server.
Qualquer projeto com IA pluga e ganha memória estruturada de longo prazo, inspirada em Obsidian + estrutura cognitiva humana.

---

## Identidade e propósito

- **O que é**: uma camada de memória organizada, indexada e recuperável por tags hierárquicas + pontuação cognitiva. Plug-in para projetos de IA existentes.
- **Como se instala**: MCP server. Desenvolvedor pluga no cliente dele (Claude Code, Claude Desktop, Cursor, app próprio via SDK MCP) e expõe as tools de memória pro agente.
- **Inspiração**: Obsidian (vault de .md com links entre notas) + ponderação de relevância do cérebro humano (memórias decaem, são reativadas pelo uso e ganham piso durável quando o usuário reage à IA).

---

## Princípio arquitetural central

**O único eixo de recuperação é Major Tag → Tag → sinopse → conteúdo.**

- **Major Tag** = tema amplo (ex: "Pessoas", "Trabalho", "Família").
- **Tag** = tópico dentro do tema (ex: "Mateus Massari - Cofundador").
- **Sinopse** = descrição curta (~300-400 chars) no topo do .md, no estilo de description de skill. Permite ao agente decidir se vale aprofundar sem ler o arquivo todo.
- **Conteúdo** = corpo do .md.
- **Go Deeper** = referências cruzadas entre memórias (estilo wikilink Obsidian).

Uma memória pode pertencer a múltiplas Major Tags.

A IA do agente usa o próprio raciocínio dela pra navegar essa estrutura. Ela sabe que "sócio" é parente de "cofundador" sem precisar de embedding pra calcular isso.

---

## O que está DENTRO do escopo

- Receber conteúdo **textualizado** + metadados via tool `remember(...)`.
- Indexar em estrutura .md + SQLite (FTS5 para fallback de busca textual).
- Sistema de pontuação 1-100 com decaimento temporal, reativação e piso emocional não-decaente.
- Sono noturno (cron customizável, default 03:00) que processa rascunhos via LLM-as-judge.
- Sugerir/manter Go Deeper entre memórias relacionadas.
- Tools de leitura/navegação pelo agente do host.
- CLI auxiliar (`init`, `sleep-now`, `stats`).

## O que está FORA do escopo (não engordar o projeto)

- ❌ Transcrição de áudio (responsabilidade do app host).
- ❌ Processamento de imagens / visão (responsabilidade do app host).
- ❌ Fetch/scraping de páginas web (responsabilidade do app host).
- ❌ Gerenciamento da janela de contexto do host.
- ❌ Funcionar como provider de LLM ou de chat.
- ❌ Embeddings vetoriais (rejeitado por decisão de design, ver "Decisões travadas").

O app host pode lidar com qualquer modalidade — quando converter para texto, chama `remember(...)` passando o texto + um `asset_ref` opaco apontando pro binário dele.

---

## Decisões travadas (não relitigar sem motivo forte)

1. **Distribuição**: MCP Server. Não plugin Claude Code (limitaria portabilidade), não lib pura (mais fricção de integração).
2. **Storage**: `.md` no disco como fonte da verdade + SQLite como índice (scores, timestamps, FTS5). Vault legível no Obsidian.
3. **Sem embeddings**. A IA do agente faz similaridade semântica raciocinando sobre sinopses. Major Tag → Tag é o eixo, FTS é a rede de segurança textual. Rejeitado porque:
   - Major Tag → Tag resolve ~70% dos casos sozinho.
   - FTS cobre mais ~20%.
   - LLM raciocinando sobre sinopses cobre o restante com qualidade superior a similaridade vetorial.
   - Elimina infra de embedding, dimensão fixa, reindex em troca de provider, custo de indexação.
4. **Sono cron** (default 03:00, customizável). Não eager indexing durante conversa — economiza token e mantém conversa fluida.
5. **Cascata de custo no sono**: heurística barata (palavras-chave, regex) marca candidatos; LLM-as-judge revisa só os marcados. Otimização de custo sem perder precisão.
6. **Provider configurável** (OpenAI, Anthropic, Voyage, Cohere, Gemini, Mistral, Jina, Azure, Bedrock, Ollama local, custom OpenAI-compatible). Camada de abstração via `litellm` ou equivalente. Mesma regra de provider serve pra qualquer uso de LLM no sistema (atualmente só o judge — sem embedding).
7. **Uma chave de API só** pra rodar tudo (a do judge). Detecta chaves no ambiente (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, etc.) e usa o que tiver.
8. **Reindex incremental por score descendente**: se trocar provider/modelo do judge, re-processa memórias de score alto primeiro. Sistema fica usável durante migração.
9. **Decaimento de score**: piso global = 1, nunca 0. Nenhuma memória é apagada sozinha — só perde superficialidade.
10. **Emoção como feedback sobre a IA**: emoção narrada no fato vira conteúdo; feedback positivo ou negativo sobre resposta, lembrança, esquecimento ou correção da IA aumenta o `emotion_floor`.
11. **Audit local por padrão**: o vault mantém `_audit.jsonl` com chamadas de tools e decisões do sono, sem conteúdo completo de memória e com campos sensíveis redigidos. Retenção default: 90 dias ou 25 MB.

---

## Modelo De Pontuação

A escala principal continua limitada a `1-100`.

- **Memória nova** nasce com `score = 100`.
- **Decaimento temporal** reduz o score em `1.5` ponto por dia desde `last_scored`.
- **Piso global** é `1`: memória não é apagada sozinha.
- **Piso emocional** (`emotion_floor`) impede que uma memória reforçada por feedback afunde abaixo de um limite durável.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MLTCorp/sincron-brain-model](https://github.com/MLTCorp/sincron-brain-model) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
