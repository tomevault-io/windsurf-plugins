---
trigger: always_on
description: Extrai texto de documentos jurídicos escaneados em PDF usando OCR otimizado para linguagem jurídica brasileira. Use quando precisar converter PDFs escaneados (sentenças, petições, acórdãos) em texto editável com alta precisão. Suporta documentos de baixa qualidade, multi-colunas, tabelas e termos jurídicos específicos.
---


# Legal OCR - Extração de Texto de Documentos Jurídicos

## Visão Geral

Esta skill extrai texto de documentos jurídicos escaneados em formato PDF usando técnicas avançadas de OCR (Optical Character Recognition) otimizadas especificamente para a linguagem jurídica brasileira.

**Principais características:**
- **Alta precisão**: 95%+ em documentos limpos, 85-90% em documentos de baixa qualidade
- **Otimizado para português jurídico**: Dicionário especializado com 200+ termos jurídicos
- **Pré-processamento inteligente**: Correção de inclinação, remoção de ruído, melhoria de contraste
- **Multi-engine**: PaddleOCR (primário) + EasyOCR (fallback)
- **Estrutura preservada**: Identifica seções (relatório, fundamentação, dispositivo)
- **Validação de qualidade**: Score de confiança e detecção de erros

## Quando Usar

Use esta skill quando precisar:
- Converter PDFs escaneados de processos judiciais em texto editável
- Extrair conteúdo de sentenças, acórdãos, petições digitalizadas
- Processar documentos antigos ou de baixa qualidade de escaneamento
- Alimentar sistema de análise jurídica (RAG) com documentos históricos
- Criar banco de jurisprudência a partir de documentos físicos digitalizados

**NÃO use para:**
- PDFs nativos digitais (use ferramentas de extração de texto direto)
- Documentos manuscritos (acurácia limitada)
- Imagens com resolução < 200 DPI (resultados ruins)

## Como Funciona

### Pipeline de Processamento

```
PDF Scaneado
    ↓
[1] Conversão PDF → Imagem (PyMuPDF, 300 DPI)
    ↓
[2] Pré-processamento
    • Conversão para escala de cinza
    • Correção de inclinação (Hough Transform)
    • Remoção de ruído (Median Blur)
    • Melhoria de contraste (CLAHE)
    • Binarização adaptativa
    ↓
[3] OCR Engine (PaddleOCR pt-BR)
    • Detecção de texto
    • Reconhecimento de caracteres
    • Score de confiança
    ↓
[4] Fallback (EasyOCR) se confiança < 30%
    ↓
[5] Pós-processamento
    • Correção com dicionário jurídico
    • Correção de acentuação
    • Identificação de estrutura do documento
    ↓
[6] Validação de Qualidade
    • Score de confiança
    • Detecção de problemas (O/0, l/1, etc.)
    • Flag para revisão humana
    ↓
Texto Estruturado (JSON)
```

### Tecnologias Utilizadas

| Componente | Biblioteca | Função |
|------------|-----------|---------|
| PDF → Imagem | PyMuPDF (fitz) | Conversão rápida (3.3x mais rápida que pdf2image) |
| Pré-processamento | OpenCV | Melhoria de qualidade da imagem |
| OCR Primário | PaddleOCR | Melhor acurácia para português (95%+) |
| OCR Fallback | EasyOCR | Alternativa quando PaddleOCR falha |
| Pós-processamento | Custom + Transformers | Correção de termos jurídicos |

## Recursos

### 1. Pré-processamento Avançado
- **Correção de inclinação**: Detecta e corrige documentos escaneados tortos
- **Remoção de ruído**: Remove artefatos de escaneamento
- **CLAHE**: Melhoria de contraste adaptativa (crítico para documentos antigos)
- **Binarização**: Converte para preto/branco otimizado para OCR

### 2. Dicionário Jurídico Brasileiro
- 200+ termos jurídicos pré-cadastrados
- Correção automática de erros comuns (ex: "açao" → "ação", "decisao" → "decisão")
- Fuzzy matching para termos similares (85%+ similaridade)
- Cobertura de áreas: civil, penal, trabalhista, tributário

### 3. Identificação de Estrutura
Reconhece automaticamente seções típicas de documentos jurídicos:
- **Cabeçalho**: Tribunal, número do processo
- **Preâmbulo**: Partes, juiz
- **Relatório**: Síntese dos fatos
- **Fundamentação**: Argumentação jurídica
- **Dispositivo**: Decisão final
- **Assinaturas**: Bloco de assinaturas

### 4. Suporte a Layouts Complexos
- Multi-colunas
- Tabelas (extração estruturada)
- Cabeçalhos e rodapés
- Notas de rodapé

### 5. Validação de Qualidade
- Score de confiança (0-100)
- Detecção de confusão O/0, l/1, S/5
- Verificação de elementos obrigatórios (juiz, tribunal, data, etc.)
- Flag automático para revisão humana quando confiança < 70%

## Uso Básico

### Comando Simples
```bash
# Extrair texto de um único PDF
python .claude/skills/legal-ocr/pipeline_ocr.py sentenca_escaneada.pdf
```

### Comando com Opções
```bash
# Alta qualidade + GPU + output customizado
python .claude/skills/legal-ocr/pipeline_ocr.py \
  --input documento.pdf \
  --output resultado.json \
  --quality high \
  --use-gpu \
  --dpi 400
```

### Batch Processing
```bash
# Processar múltiplos PDFs
python .claude/skills/legal-ocr/pipeline_ocr.py \
  --input-dir ./processos_escaneados/ \
  --output-dir ./textos_extraidos/ \
  --batch-size 32
```

## Output Format

O resultado é um arquivo JSON estruturado:

```json
{
  "filename": "sentenca_123.pdf",
  "timestamp": "2025-12-10T21:00:00",
  "metadata": {
    "total_pages": 15,
    "processing_time_seconds": 45.2,
    "gpu_used": true,
    "primary_engine": "paddleocr",
    "fallback_used_pages": [3, 7]
  },
  "pages": [
    {
      "page_num": 1,
      "text": "SENTENÇA\n\nProcesso nº 0001234-56.2024.8.26.0100...",
      "confidence": 0.92,
      "source": "paddleocr",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fbmoulin/legal-ocr-skill](https://github.com/fbmoulin/legal-ocr-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
