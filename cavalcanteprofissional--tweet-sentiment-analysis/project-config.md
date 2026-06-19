---
trigger: always_on
description: >
---


# 🧠 Ensemble de Modelos — Análise de Sentimento em Redes Sociais
### Trabalho Final — Python em Projetos de IA

---

## 📌 Visão Geral do Projeto

**Problema real:** Classificar automaticamente o sentimento de tweets em positivo, negativo ou neutro,
com aplicação prática em monitoramento de marca, análise de crises e pesquisa social.

**Abordagem escolhida:** Ensemble de 3 modelos pré-treinados via Hugging Face Transformers, com
estratégia de votação ponderada por confiança, publicado no HF Hub e versionado no GitHub.

**Por que isso é portfólio nível sênior?**
- Usa modelos state-of-the-art (não treina do zero — escolha inteligente)
- Ensemble é técnica usada em competições e produção real
- Publicação no HF Hub demonstra conhecimento de MLOps
- Código limpo, documentado e reproduzível

---

## 🗂️ Estrutura do Repositório

```
sentiment-ensemble/
├── README.md                    # Card do projeto com badges e resultados
├── requirements.txt
├── .github/
│   └── workflows/
│       └── lint.yml             # CI básico com flake8
├── notebooks/
│   ├── 01_EDA.ipynb             # Análise exploratória do dataset
│   ├── 02_baseline.ipynb        # Modelos individuais + métricas
│   └── 03_ensemble.ipynb        # Ensemble + análise final
├── src/
│   ├── __init__.py
│   ├── data_loader.py           # Carregamento e split do dataset
│   ├── preprocessor.py          # Limpeza específica para tweets
│   ├── models.py                # Wrapper para cada modelo HF
│   ├── ensemble.py              # Lógica de votação ponderada
│   └── evaluate.py              # Métricas + relatório
├── scripts/
│   ├── run_inference.py         # Inferência standalone (demo)
│   └── push_to_hub.py           # Publica modelo no HF Hub
└── tests/
    └── test_preprocessor.py     # Testes unitários básicos
```

---

## 1️⃣ DEFINIÇÃO DO PROBLEMA

### O que está sendo resolvido
Tweets têm linguagem extremamente ruidosa: gírias, emojis, abreviações, ironia, hashtags e menções.
Modelos genéricos de sentimento falham nesse contexto. O projeto resolve isso usando modelos
**pré-treinados especificamente em dados de redes sociais**, combinados em ensemble para
maximizar robustez.

### Impacto prático
- Empresas monitoram percepção de marca em tempo real
- Campanhas políticas analisam reação do público
- Pesquisadores estudam saúde mental coletiva
- Jornalismo de dados usa para cobrir eventos ao vivo

### Solução atual e suas limitações
VADER e TextBlob são usados amplamente mas:
- VADER não lida bem com português ou contextos culturais específicos
- Modelos bag-of-words perdem contexto sequencial (ironia, negação)
- Fine-tuning individual de um único modelo é frágil a distribuições out-of-distribution

### Tipo de problema
Classificação multiclasse (3 classes: positivo, negativo, neutro) com textos curtos e ruidosos.

---

## 2️⃣ DATASET

### Dataset escolhido: `tweet_eval` (Hugging Face Datasets)

**Justificativa de escolha:**
- Benchmark padrão da comunidade NLP para tweets — usado em papers acadêmicos
- Subset `sentiment`: ~45k tweets em inglês já anotados (positivo/negativo/neutro)
- Split pré-definido (train/validation/test) — evita data leakage
- Carregamento direto via `datasets` lib sem necessidade de scraping ou API

**Alternativa PT-BR (opcional para diferencial):**
- `HeiReS/twitter_sentiment_pt` ou `ruanchaves/hatebr` (disponíveis no HF Hub)
- Se optar por PT-BR, os modelos mudam (ver seção 4)

**Carregamento:**
```python
from datasets import load_dataset

ds = load_dataset("tweet_eval", "sentiment")
# train: 45.615 | validation: 2.000 | test: 12.284
```

**Análise de desbalanceamento (EDA obrigatória):**
```python
import pandas as pd
import matplotlib.pyplot as plt

df_train = pd.DataFrame(ds['train'])
print(df_train['label'].value_counts())
# Espera-se: neutro > positivo > negativo (desequilíbrio moderado)

df_train['label'].value_counts().plot(kind='bar', title='Distribuição de Classes')
plt.savefig('assets/class_distribution.png')
```

**Por que a distribuição importa:**
- Se neutro dominar, Accuracy sozinha vai enganar
- Precisamos de F1-Macro como métrica principal (ver seção 6)

---

## 3️⃣ PRÉ-PROCESSAMENTO

### Limpeza específica para tweets (src/preprocessor.py)

```python
import re

def clean_tweet(text: str) -> str:
    """
    Limpeza focada em preservar semântica emocional.
    NÃO remove emojis — eles carregam sentimento.
    """
    # Remove URLs
    text = re.sub(r'http\S+|www\S+', '', text)
    # Remove menções (@usuario) mas mantém o @ para contexto
    text = re.sub(r'@\w+', '@user', text)
    # Normaliza hashtags (remove # mas mantém a palavra)
    text = re.sub(r'#(\w+)', r'\1', text)
    # Remove caracteres repetidos excessivos (looooove -> loove)
    text = re.sub(r'(.)\1{3,}', r'\1\1', text)
    # Remove espaços extras
    text = re.sub(r'\s+', ' ', text).strip()
    return text
```

**Justificativa de cada escolha:**
| Técnica | Por quê |
|---|---|
| Substituir @user | Preserva estrutura sem vazar dados de usuários |
| Manter emojis | 😂 e 😢 são features riquíssimas de sentimento |
| Normalizar repetições | "looool" e "lol" são o mesmo token semanticamente |
| NÃO fazer stemming | Transformers tokenizam no nível de subpalavras — stemming atrapalha |
| NÃO remover stopwords | Negações como "não" e "never" são críticas para sentimento |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cavalcanteprofissional/tweet-sentiment-analysis](https://github.com/cavalcanteprofissional/tweet-sentiment-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
