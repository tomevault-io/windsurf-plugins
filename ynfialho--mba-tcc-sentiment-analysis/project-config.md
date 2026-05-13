---
trigger: always_on
description: Projeto de conclusão de curso (TCC) que coleta reportagens do portal G1, identifica menções a bairros de São Paulo via NER (Named Entity Recognition) e classifica o sentimento associado a cada bairro (positivo, neutro, negativo) usando modelos transformer.
---

# Memória do Projeto — MBA TCC Análise de Sentimento

## Visão Geral

Projeto de conclusão de curso (TCC) que coleta reportagens do portal G1, identifica menções a bairros de São Paulo via NER (Named Entity Recognition) e classifica o sentimento associado a cada bairro (positivo, neutro, negativo) usando modelos transformer.

**Stack**: Python 3.11+ · UV (gerenciador de pacotes) · PostgreSQL · SQLModel · spaCy · Hugging Face Transformers · PyTorch

---

## Estrutura de Módulos

| Módulo | Responsabilidade |
|---|---|
| `common/` | Modelos ORM (SQLModel), configuração do banco e utilitários compartilhados |
| `extractor/` | Web crawler que lê o sitemap do G1, respeita robots.txt, extrai artigos e armazena no banco |
| `wrangler/` | Pré-processamento de texto (normalização, remoção de URLs, unicode, etc.) |
| `modeling/ner/` | NER rule-based com spaCy PhraseMatcher para bairros de SP (com variações e validação geográfica) |
| `modeling/models/` | Analyzers de sentimento (DistilBERT, BERTimbau), model registry e factory |
| `modeling/sentiment/` | Processamento em batch, agregação e CLI de sentimento |
| `modeling/tuning/` | Avaliação, grid search de hiperparâmetros, dataset builder e fine-tuning |
| `modeling/constants.py` | Constantes compartilhadas: padrões NER, regras neutras, palavras fortes, falsos positivos |
| `tests/` | Testes unitários e de integração (pytest) |
| `iac/` | Docker Compose + scripts SQL de inicialização do PostgreSQL |
| `scripts/` | Utilitários (backup do banco) |

---

## Como Rodar

### Pré-requisitos

```bash
# Instalar UV (se necessário)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Instalar dependências
uv sync

# Subir o banco PostgreSQL
cd iac && docker-compose up -d
```

Credenciais ficam no arquivo `.env` na raiz do projeto.

### CLIs Disponíveis

#### Extractor (crawler G1)

```bash
# Extrair artigos do sitemap
uv run python -m extractor crawl

# Extrair com limite
uv run python -m extractor crawl --limit 500
```

#### Main (NER)

```bash
# Rodar NER nos artigos extraídos
uv run python main.py ner --limit 1000
```

#### Sentiment Analysis

```bash
# Analisar sentimento (todos os níveis)
uv run python -m modeling.sentiment analyze all

# Analisar por nível específico
uv run python -m modeling.sentiment analyze zone|subprefecture|neighborhood

# Com modelo alternativo (feature toggle)
uv run python -m modeling.sentiment analyze all --model-key bertimbau --fine-tuned-path models/bertimbau-sentiment/best

# Estatísticas
uv run python -m modeling.sentiment stats [--format json]

# Breakdown por zona ou subprefeitura
uv run python -m modeling.sentiment breakdown zone|subprefecture

# Testar com texto livre
uv run python -m modeling.sentiment test --text "Texto aqui"

# Listar modelos disponíveis
uv run python -m modeling.sentiment models

# Popular tabela de agregações
uv run python -m modeling.sentiment aggregate --period-type monthly
```

#### Tuning & Avaliação

```bash
# Avaliar com mock data (8 test cases)
uv run python -m modeling.tuning evaluate [-v|-vv]

# Grid search de hiperparâmetros
uv run python -m modeling.tuning tune --quick|--full

# Analisar test case específico
uv run python -m modeling.tuning analyze <nome>

# Listar test cases
uv run python -m modeling.tuning list

# Construir dataset de avaliação a partir do banco
uv run python -m modeling.tuning build-dataset --limit 100 --strategy random|balanced|recent

# Avaliar contra dataset JSON rotulado manualmente
uv run python -m modeling.tuning evaluate-json <caminho.json>

# Fine-tuning do BERTimbau
uv run python -m modeling.tuning fine-tune --dataset data/eval_dataset.json --epochs 5
```

#### Testes

```bash
# Rodar todos os testes (exceto slow)
uv run python -m pytest tests/ -v -k "not slow"

# Rodar apenas testes do modeling
uv run python -m pytest tests/modeling/ -v

# Rodar apenas testes do extractor
uv run python -m pytest tests/extractor/ -v

# Com cobertura
uv run python -m pytest tests/ --cov=modeling --cov=common
```

---

## Modelos ORM (common/models.py)

| Tabela | Descrição |
|---|---|
| `SitemapEntry` | URLs do sitemap G1 com status de extração |
| `Article` | Artigos extraídos (título, conteúdo, data, autor, URL) |
| `Neighborhood` | Bairros de SP com zona e subprefeitura |
| `EntityMention` | Menções de bairros encontradas nos artigos (NER) |
| `SentimentAnalysisResult` | Resultado de sentimento por menção (label, score, raw_scores) |
| `SentimentAggregation` | Agregações de sentimento por período e nível geográfico |

---

## Pipeline de Sentimento (Hybrid: Rules + Model)

```
Texto do artigo
  │
  ├─ STEP 1: Validação geográfica (rule-based)
  │   └─ Detecta se o bairro está no contexto de outra cidade → DISCARD
  │
  ├─ STEP 2: Validação semântica (rule-based)
  │   └─ Verifica se a menção é referência a local e não outro significado → DISCARD
  │
  ├─ STEP 3: Extrai sentenças relevantes (que mencionam o bairro)
  │
  ├─ STEP 3.5: Regras neutras (pre-model)
  │   └─ Padrões fortes/fracos de conteúdo factual → NEUTRAL
  │
  ├─ STEP 4: Inferência do modelo transformer
  │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ynfialho/mba-tcc-sentiment-analysis](https://github.com/ynfialho/mba-tcc-sentiment-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
