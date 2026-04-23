---
trigger: always_on
description: Catálogo de técnicas RAG avançadas (Query Decomposition, Adaptive Re-ranking, Router, Self-RAG, CRAG) com complexidade, ROI, quando usar e métricas validadas
---


# 📘 CATÁLOGO DE TÉCNICAS RAG - BSC Project

**Versão:** 1.0
**Última Atualização:** 2025-10-14
**Técnicas Catalogadas:** 3 (Fase 2A completa) + 2 (Fase 2B planejadas)

---

## 🎯 COMO USAR ESTE CATÁLOGO

**3 formas de navegação:**

1. **Índice Navegável** (Seção 1) - Quick Reference Table
2. **Por Categoria** (Seção 2) - Query Enhancement, Re-ranking, Agentic
3. **Por Complexidade** (Seção 3) - ⭐⭐ a ⭐⭐⭐⭐⭐

**Quando consultar:**

- ✅ Planejar próxima técnica RAG a implementar
- ✅ Decidir trade-offs (latência vs qualidade vs custo)
- ✅ Estimar tempo de implementação
- ✅ Identificar pré-requisitos técnicos
- ✅ Validar se técnica é necessária (evitar over-engineering)

---

## 📑 SEÇÃO 1: ÍNDICE NAVEGÁVEL

### Quick Reference Table

| TECH-ID | Nome | Categoria | Complexidade | Tempo Real | ROI | Status |
|---------|------|-----------|--------------|------------|-----|--------|
| TECH-001 | Query Decomposition | Query Enhancement | ⭐⭐ | 4 dias | ⭐⭐⭐⭐⭐ | ✅ Implementado |
| TECH-002 | Adaptive Re-ranking | Re-ranking | ⭐⭐ | 2 dias | ⭐⭐⭐⭐ | ✅ Implementado |
| TECH-003 | Router Inteligente | Agentic RAG | ⭐⭐⭐⭐ | 6h | ⭐⭐⭐⭐⭐ | ✅ Implementado |
| TECH-004 | Self-RAG | Emergente | ⭐⭐⭐⭐ | 3-4 dias | ⭐⭐⭐⭐ | 📋 Planejado (2B.1) |
| TECH-005 | CRAG | Emergente | ⭐⭐⭐⭐⭐ | 4-5 dias | ⭐⭐⭐⭐ | 📋 Planejado (2B.2) |

---

## 📊 SEÇÃO 2: TÉCNICAS POR CATEGORIA

### 🔍 Query Enhancement

Técnicas que transformam/melhoram a query antes do retrieval.

#### TECH-001: Query Decomposition

**Status:** ✅ Implementado (Fase 2A.1)

**O que é:** Quebra queries BSC complexas em 2-4 sub-queries independentes.

**Quando usar:**

- ✅ Queries com 2+ perguntas ("Como X e qual Y?")
- ✅ Palavras de ligação ("e", "também", "considerando")
- ✅ Múltiplas perspectivas BSC mencionadas
- ✅ Queries relacionais ("relação entre X e Y")
- ❌ Queries simples factual (<30 palavras, "O que é BSC?")

**ROI Esperado:** +30-40% recall, +25-35% precision
**ROI Real:** Heurística 100% accuracy (ground truth não validável ainda)

**Complexidade:** ⭐⭐ (Simples)
**Tempo:** 4 dias (implementação + testes + docs)
**Custo:** ~$0.0001 por query (GPT-5 mini)

**Componentes:**

- `src/rag/query_decomposer.py` (270 linhas)
- `src/prompts/query_decomposition_prompt.py` (110 linhas)
- `tests/test_query_decomposer.py` (20 testes, 91% coverage)

**Pré-requisitos:**

- ✅ RRF já implementado (multilíngue)
- ✅ BSCRetriever funcional
- ✅ GPT-5 mini configurado

**Métricas:**

| Métrica | Target | Real | Status |
|---------|--------|------|--------|
| Heurística Accuracy | >80% | 100% | ✅ |
| Latência Adicional | <3s | +4.25s | ⚠️ Aceitável |
| Coverage | >80% | 91% | ✅ |
| Testes | 15+ | 20 | ✅ |

**Código Exemplo:**

```python
from src.rag.query_decomposer import QueryDecomposer
from src.rag.retriever import BSCRetriever

# Setup
decomposer = QueryDecomposer(llm=gpt4o_mini)
retriever = BSCRetriever()

# Uso
query = "Como implementar BSC considerando perspectivas financeira e clientes?"
should, score = decomposer.should_decompose(query)  # (True, 3)

if should:
    docs = await retriever.retrieve_with_decomposition(query, k=10)
    # Sub-queries geradas → Retrieval paralelo → RRF fusion → Top-10
```

**Lições-Chave:**

1. Heurísticas simples >80% accuracy (word boundaries críticos!)
2. GPT-5 mini suficiente (não precisa GPT-4o)
3. RRF reutilização economizou 1 dia
4. Ground truth precisa campo `source` em metadata Qdrant

**Documentação:** `docs/techniques/QUERY_DECOMPOSITION.md` (400+ linhas)

---

### 🎯 Re-ranking

Técnicas que melhoram qualidade e diversidade dos documentos re-ranked.

#### TECH-002: Adaptive Re-ranking

**Status:** ✅ Implementado (Fase 2A.2)

**O que é:** Re-ranking inteligente com 3 componentes: MMR (diversidade), Metadata Boost (múltiplas fontes), Adaptive Top-N (ajuste dinâmico).

**Quando usar:**

- ✅ Documentos repetidos/similares no top-k
- ✅ Queries multi-perspectiva (garantir variedade)
- ✅ Precisar ajustar top_n por complexidade
- ❌ Dataset muito pequeno (<50 docs totais)
- ❌ Latência crítica (<2s requirement)

**ROI Esperado:** Diversity score >0.7, ≥2 fontes no top-5
**ROI Real:** 100% coverage, 38 testes validados

**Complexidade:** ⭐⭐ (Simples)
**Tempo:** 2 dias (implementação + 38 testes + docs)
**Custo:** +1-2s latência (cálculo embeddings + MMR)

**Componentes:**

1. **MMR Algorithm**: λ * relevance - (1-λ) * max_similarity
2. **Metadata Boost**: +20% source diferente, +15% perspective diferente
3. **Adaptive Top-N**: Query simples=5, média=10, complexa=15

**Implementação:**

```python
# src/rag/reranker.py
reranker.rerank_with_diversity(
    query=query,
    documents=docs,
    embeddings=embeddings,
    top_n=None  # Adaptativo: 5/10/15
)
```

**Configuração (.env):**

```bash
ENABLE_DIVERSITY_RERANKING=True
DIVERSITY_LAMBDA=0.5            # Balanceado
DIVERSITY_THRESHOLD=0.8         # Max similaridade
METADATA_BOOST_ENABLED=True
ADAPTIVE_TOPN_ENABLED=True
```

**Métricas:**

| Métrica | Target | Real | Status |
|---------|--------|------|--------|
| Coverage | >80% | 100% | ✅ |
| Testes | 15+ | 38 | ✅ +153% |
| Diversity Score | >0.7 | Validado | ✅ |
| Tempo | 2-3d | 2d | ✅ |

**Lições-Chave:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hpm27) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
