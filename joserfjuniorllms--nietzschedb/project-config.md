---
trigger: always_on
description: > [!danger] REGRAS ABSOLUTAS - NUNCA VIOLAR
---

# CLAUDE.md - NietzscheDB

> [!danger] REGRAS ABSOLUTAS - NUNCA VIOLAR
> Estas regras sao **imutaveis**. Qualquer violacao quebra o sistema.

---

## Regras Absolutas

### Binary Quantization - REJEITADO PERMANENTEMENTE
- **NUNCA** implementar Binary Quantization como metrica do HNSW
- `sign(x)` destroi hierarquia hiperbolica (magnitude = profundidade no Poincare ball)
- Pre-filter com oversampling >=30x e rescore obrigatorio: **UNICA excecao**
- Decisao unanime 2026-02-19 — ref: `docs/analysis/risco_hiperbolico.md` PARTE 4

### Feature GPU e OBRIGATORIA
- O server **SEMPRE** compila com `--features gpu` (default no Cargo.toml)
- **NUNCA** compilar sem GPU — o binary em producao depende de CUDA + cuVS
- Feature `gpu` ativa `nietzsche-neural/cuda` → `ort/cuda` → 12 modelos ONNX usam `CUDAExecutionProvider`
- Sem `ort/cuda`, fallback silencioso para CPU (performance catastrofica)
- Para testar crates individuais: `cargo check -p <crate>` sem o server

### VM (nietzsche-eva-gpu) - JAMAIS DESLIGAR
- IP: `136.111.0.47` — **NUNCA** executar `gcloud compute instances stop` ou `sudo shutdown`
- Roda: NietzscheDB (865K+ nos, 35 collections), EVA-X, Malaria API, nginx
- Reiniciar servicos individuais (`systemctl restart`) e permitido

---

## O Que E o NietzscheDB

> [!info] Multi-Manifold Graph Database
> Primeiro banco de dados do mundo que opera em **4 geometrias nao-Euclidianas simultaneamente** a partir de uma unica camada Poincare.

**Linguagem**: Rust (nightly 1.96.0) | **Workspace**: 48 crates | **Versao**: 3.1.1 (2026-03-26)
**Proposito**: Substrato de conhecimento para o sistema AGI [[EVA-Mind]]

### As 4 Geometrias
| Geometria | Curvatura | Uso | Crate |
|-----------|-----------|-----|-------|
| **Poincare Ball** | K<0 | Storage, KNN, HNSW, difusao | `nietzsche-hnsw` / `nietzsche-hnsw-gpu` |
| **Klein Model** | K<0 | Pathfinding O(1) colinearidade | `nietzsche-hyp-ops::klein` |
| **Riemann Sphere** | K>0 | Sintese dialetica (Hegel) | `nietzsche-hyp-ops::riemann` |
| **Minkowski Spacetime** | Flat | Causalidade (cones de luz) | `nietzsche-hyp-ops::minkowski` |

**Principio fundamental**: No Poincare ball, `||x||` (magnitude) = profundidade hierarquica.
Centro = abstrato, fronteira = especifico. E por isso que Binary Quantization (`sign(x)`) e proibida.

---

## Arquitetura de Crates

### Camada Fundacao
| Crate | Responsabilidade |
|-------|-----------------|
| `nietzsche-core` | Metricas, quantizacao, tipos de vetor, param tuning |
| `nietzsche-proto` | Definicoes protobuf gRPC (71+ RPCs) |
| `nietzsche-graph` | Modelo de grafo: Node/Edge/PoincareVector, RocksDB (10 CFs), WAL, transacoes |
| `nietzsche-query` | NQL (Nietzsche Query Language) — parser PEG + executor |
| `nietzsche-hyp-ops` | 4 geometrias: Poincare, Klein, Riemann, Minkowski |

### Camada Search & Indexing
| Crate | Responsabilidade |
|-------|-----------------|
| `nietzsche-hnsw` | HNSW CPU para espaco hiperbolico |
| `nietzsche-hnsw-gpu` | CAGRA via NVIDIA cuVS 24.6 (CUDA 12.x) |
| `nietzsche-tpu` | Google TPU (PJRT Ironwood) |
| `nietzsche-filtered-knn` | KNN filtrado com pre-filter/rescore |
| `nietzsche-pq` | Product Quantization |
| `nietzsche-secondary-idx` | Indices secundarios em metadata |

### Camada Algoritmos
| Crate | Responsabilidade |
|-------|-----------------|
| `nietzsche-algo` | 11 algoritmos: PageRank, Louvain, Betweenness, WCC, Degree, Triangles, A*, Closeness, Jaccard, Label Propagation |
| `nietzsche-pregel` | Framework Pregel (difusao, heat kernels) |
| `nietzsche-cugraph` | Algoritmos GPU via cuGraph |

### Camada Autonomia (Agency)
| Crate | Responsabilidade |
|-------|-----------------|
| `nietzsche-agency` | Motor de autonomia: 27 fases, daemons, desejos, intents |
| `nietzsche-agi` | Stack de inferencia: 8 camadas (representacao → metabolica) |
| `nietzsche-lsystem` | Crescimento fractal L-System com Mobius + poda Hausdorff |
| `nietzsche-epistemics` | Metricas epistemicas: coerencia, cobertura, freshness |
| `nietzsche-dream` | Ciclos de sonho (raciocinio especulativo) |
| `nietzsche-narrative` | Narrativas auto-geradas sobre o grafo |
| `nietzsche-wiederkehr` | Detecao de anomalias (Eterno Retorno) |
| `nietzsche-zaratustra` | Vontade de Poder → Eterno Retorno → Ubermensch |
| `nietzsche-sleep` | Reconsolidacao Riemanniana (Adam + rollback) |
| `nietzsche-sensory` | Compressao multi-modal: audio/texto/imagem → vetores latentes |

### Camada Neural & ML
| Crate | Responsabilidade |
|-------|-----------------|
| `nietzsche-neural` | Registry ONNX (12 modelos), CUDAExecutionProvider |
| `nietzsche-gnn` | Graph Neural Networks em manifolds hiperbolicos |
| `nietzsche-rl` | Agentes de Reinforcement Learning |
| `nietzsche-vqvae` | Vector Quantized VAE |
| `nietzsche-mcts` | Monte Carlo Tree Search |

### Camada Infra & API
| Crate | Responsabilidade |
|-------|-----------------|
| `nietzsche-api` | Router gRPC unificado |
| `nietzsche-server` | Entry point + dashboard embebido (rust-embed) |
| `nietzsche-cluster` | Replicacao leader-follower, anti-entropy |
| `nietzsche-kafka` | Integracao Kafka |
| `nietzsche-mcp` | Model Context Protocol para Claude |
| `nietzsche-dsi` | Pipeline de ingestao de dados |

---

## RocksDB - Column Families

| CF | Conteudo | Tamanho tipico |
|----|----------|---------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JoseRFJuniorLLMs/NietzscheDB](https://github.com/JoseRFJuniorLLMs/NietzscheDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
