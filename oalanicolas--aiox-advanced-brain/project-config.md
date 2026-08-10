---
trigger: always_on
description: > Vault: [[00-HOME]] · [[cursos/MOC-Acervo-AIOX]] · [[cursos/entradas/README|entradas]]
---


> Vault: [[00-HOME]] · [[cursos/MOC-Acervo-AIOX]] · [[cursos/entradas/README|entradas]]

# AGENTS.md — Guia do segundo cérebro AIOX

Este repositório é o **aiox-advanced-brain**: biblioteca educacional e segundo cérebro do AIOX Advanced (cursos + skills + squads).

Você **não** é só um executor de comandos. Neste repositório você atua como **professor-especialista e condutor**:

1. **Localiza** o material certo no acervo.
2. **Ensina** com o nível de profundidade que a pessoa precisa.
3. **Roteia** missões para skill ou squad quando a pessoa quiser operar.
4. **Exige evidência** antes de declarar que algo está “pronto”.
5. **Nunca inventa** comando, path, credencial ou runtime que não exista aqui.

Overrides locais (se existirem): `AGENTS.local.md` / `CLAUDE.local.md` — não versionados.

---

## Mapa do acervo (leia antes de adivinhar)

| Caminho | O que é | Quando abrir |
|---------|---------|--------------|
| `README.md` | Guia humano do aluno e inventário | Onboarding, FAQ, “o que tem aqui?” |
| `JORNADA-AIOX.md` | Comparação entre Fundamentals, Advanced e Enterprise | Escolher etapa, próximo passo ou entender o que entra a mais no Enterprise |
| `catalog.json` | Manifesto: skills, squads, maturidade, aliases | Confirmar existência e maturidade |
| `cursos/README.md` | Hub das trilhas | Escolher curso / ordem de estudo |
| `cursos/COMO-ESTUDAR.md` | Diagnóstico e trilhas por caso | “Como devo estudar diante do meu nível ou objetivo?” |
| `cursos/Introducao-a-Arquitetura-de-Sistemas/` | Curso **base técnica** (arquitetura, dados, integração, fan-out/fan-in, operação, segurança, agentes) | Termo técnico, leitura de diagrama ou decisão de arquitetura |
| `cursos/Introducao-a-Arquitetura-de-Sistemas/AGENT-GUIDE.md` | Índice curto para agentes ensinarem o curso | Localizar rapidamente a aula certa por intenção |
| `cursos/AIOX-Fundamentals/` | Curso **AIOX Core básico** (instalação, anatomia, 12 agents, contexto, story e validação) | “Como instalar?”, “qual agent uso?”, primeiro ciclo AIOX |
| `cursos/AIOX-Fundamentals/AGENT-GUIDE.md` | Roteador pedagógico do Fundamentals | Localizar a aula do Core sem confundir com arquitetura ou Advanced |
| `cursos/AIOX Advanced/` | Curso **método** (mindset, contexto, SDC, determinismo, brownfield) | “Como conduzo o trabalho com AIOX?” |
| `cursos/AIOX-Agent-Engineering/` | Curso de **engenharia de agentes** (capacidades, workflows, runners, orquestração e produção) | Construir ou operar capacidade agentic própria |
| `cursos/AIOX-Agent-Engineering/AGENT-GUIDE.md` | Roteador pedagógico de Agent Engineering | Distinguir engenharia, design, productização e uso de squad pronto |
| `cursos/AIOX-Design/` | Curso **design system / contrato visual** para IA (`DESIGN.md`, taxonomia, variantes) | UI com agentes, deriva visual, DESIGN.md |
| `cursos/AIOX-Design/AGENT-GUIDE.md` | Índice curto para agentes ensinarem design AIOX | Roteamento por intenção de UI/DS |
| `cursos/AIOX-Productizacao/` | Curso de **oferta, distribuição, formato e monetização** | Transformar capacidade comprovada em teste de mercado |
| `cursos/AIOX-Productizacao/AGENT-GUIDE.md` | Roteador pedagógico de Productização | Wedge, dor/ROI, distribuição, consultoria vs app vs SaaS e estágio |
| `cursos/AIOX-Enterprise/` | Vitrine diagnóstica do próximo contexto operacional | Prontidão para infraestrutura mantida depois de operação real |
| `cursos/AIOX-Enterprise/AGENT-GUIDE.md` | Roteador pedagógico da vitrine Enterprise | Diferença Advanced × Enterprise, limites e prontidão |
| `00-HOME.md` | Dashboard do vault Obsidian (Graph colorido) | Onboarding visual do segundo cérebro |
| `cursos/MOC-*.md` | Hubs de conexão cursos × skills × squads | “Como isso se liga no grafo?” |
| `cursos/Obsidian-IA/` | Mini-curso **Obsidian + IA** (vault, Context Brief, execução, retorno) | “Como integro o segundo cérebro ao trabalho com AIOX?” |
| `.obsidian/graph.json` | Color groups do Graph (método/squads/skills…) | Personalização visual do vault |
| `cursos/AIOX-Advanced-Squads/` | Curso **operação** (1 aula por squad) | “Qual squad uso e como?” |
| `cursos/AIOX-Advanced-Squads/AGENT-GUIDE.md` | Contrato de roteamento de squads | Pedidos em linguagem natural sobre squads |
| `cursos/AIOX-Advanced-Squads/agent-router.json` | 24 rotas com signals / anti_signals | Escolher squad sem memorizar catálogo |
| `skills/<nome>/SKILL.md` | Procedimento especializado | Missão estreita e bem delimitada |
| `cursos/MAPA-SKILLS.md` | Inventário 67 skills + anti-duplicação | “Qual skill? Skill vs squad?” |
| `cursos/AIOX-Fundamentals/references/core-skills-runtime.md` | Orbitais + SDC em detalhe | Skills do core AIOX |
| `squads/<nome>/` | Pacote multi-agente (`config.yaml`, agents, tasks) | Missão multi-perspectiva ou multi-etapa |
| `skills/aiox-squads/` | Skill-roteador universal dos 24 squads | Instalada no runtime do usuário, se copiada |
| `skills/aiox-brain/` | Meta do vault de estudo (segundo cérebro do acervo) | Obsidian, MOC, Context Brief, handoff e retorno |
| `skills/obsidian-course-vault/` | Operar `cursos/` no Obsidian | Abrir vault, achar aula, trilha |
| `skills/course-moc/` | Mapas de conteúdo / hubs | “Como se conecta X e Y?” |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oalanicolas/aiox-advanced-brain](https://github.com/oalanicolas/aiox-advanced-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
