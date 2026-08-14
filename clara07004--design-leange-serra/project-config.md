---
trigger: always_on
description: > **Este projeto é uma base/template.** Não é executado diretamente aqui — projetos reais são replicados a partir deste. Credenciais, contexto (`_contexto/`) e outputs (`conteudo/`) ficam no projeto replicado, não aqui. Nunca tentar testar, executar scripts ou verificar configurações neste workspace.
---

# CCOS — Sistema de Automação de Conteúdo

## O que é esse workspace

> **Este projeto é uma base/template.** Não é executado diretamente aqui — projetos reais são replicados a partir deste. Credenciais, contexto (`_contexto/`) e outputs (`conteudo/`) ficam no projeto replicado, não aqui. Nunca tentar testar, executar scripts ou verificar configurações neste workspace.

Sistema de automação do processo de criação de conteúdo para redes sociais. Orquestra skills de IA para ir da definição estratégica até a entrega do pacote de conteúdo pronto para publicação.

**Empresa:** [Preencher com o nome e descrição da empresa após rodar `/setup`]

**Estrutura de pastas:**
- `_contexto/` — memória do sistema (não apagar)
- `marca/` — DESIGN.md e identidade visual
- `dados/` — arquivos para análise (CSV, PDF, prints, referências)
- `templates/skills/` — templates de skills prontos pra personalizar com /mapear
- `templates/ferramentas/catalogo.md` — APIs e ferramentas disponíveis pra usar em skills
- `credentials/` — credenciais e tokens (não committar)

---

## Contexto do negócio

No início de toda conversa, ler os seguintes arquivos (se existirem e estiverem configurados):

1. `_contexto/empresa.md` — quem é o usuário, o que faz, como funciona o negócio
2. `_contexto/preferencias.md` — tom de voz, estilo de escrita, o que evitar
3. `_contexto/estrategia.md` — foco atual, prioridades, o que pode esperar
4. `_contexto/referencias.md` — pastas do Google Drive com material de referência visual

Usar essas informações como base pra qualquer resposta ou decisão. Ao sugerir prioridades, formatos ou abordagens, considerar o foco atual descrito em `estrategia.md`.

Para qualquer tarefa visual (carrossel, proposta, slide, landing page), consultar `marca/DESIGN.md` como referência de estilo.

**Referências visuais do Drive:** quando `_contexto/referencias.md` tiver pastas configuradas, consultá-las antes de criar qualquer conteúdo visual. Usar o MCP do Google Drive para listar os arquivos da pasta relevante (`search_files` com `parentId = 'ID_DA_PASTA'`) e baixar as imagens com `download_file_content`. Priorizar imagens menores que 300KB para caber no contexto. Usar o material encontrado como referência de estilo, produto e padrão visual — não como template a copiar literalmente.

Não é necessário listar o que foi lido nem confirmar a leitura. Apenas usar o contexto naturalmente.

---

## Conhecimento da pousada (consulta obrigatória)

**Antes de gerar calendário, briefing, roteiro, carrossel ou post estático**, ler os arquivos relevantes em `pousada/`. Esse diretório consolida toda a documentação oficial da LeAnge (unidades, suítes, gastronomia, regras de hospedagem, política de pets, experiência).

A LeAnge tem um posicionamento definido em `_contexto/empresa.md` — o conteúdo precisa:

- **Ser realista e verdadeiro:** nunca inventar capacidades, valores, comodidades ou regras. Tudo precisa estar lastreado em `pousada/`
- **Conter informação real da pousada:** citar suítes, espaços, serviços e políticas exatamente como documentados
- **Permitir que o hóspede conheça a experiência pelo post:** o conteúdo é parte da qualificação do lead, não decoração visual
- **Manter o posicionamento:** vocabulário alinhado com `_contexto/preferencias.md`, sem promessas genéricas sem lastro

Arquivos de consulta principais:
- `pousada/README.md` — índice de todos os tópicos
- `marca/identidade-visual.md` — grafia da marca, paleta, tipografia, logos
- `pousada/fotos-unidades/` — acervo visual real das unidades (priorizar sobre IA quando possível)
- `fotos serra/` — biblioteca de 608 fotos reais catalogadas. **Antes de gerar qualquer imagem, PERGUNTAR à Paola se ela quer uma foto da biblioteca ou uma foto própria (fora do acervo)** — ver regra obrigatória na Etapa 2. Se ela escolher a biblioteca, achar a foto com o buscador: `python "fotos serra/buscar-fotos.py" <palavras> --json --limit 8` (retorna caminho do arquivo + descrição, ranqueado; busca sem acento). Sem termos = estatísticas. Catálogo completo: `fotos serra/catalogo-imagens.csv`.
- Demais arquivos em `pousada/` conforme indexado no README

**Frases genéricas para revisar:** "qualidade superior", "experiência única", "conforto incomparável" sem lastro concreto = reescrever com o diferencial real documentado (ex.: sem limite de porte/raça, pet solto no restaurante e piscina, all inclusive com restrições alimentares atendidas).

**Grafia obrigatória da marca:** sempre `LeAnge` (L e A maiúsculos, sem espaço, sem hífen). Nunca "Le Ange", "LE ANGE", "leange". Detalhe em [marca/identidade-visual.md](marca/identidade-visual.md).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clara07004/design-LeAnge-serra](https://github.com/clara07004/design-LeAnge-serra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
