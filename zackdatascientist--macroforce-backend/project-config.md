---
trigger: always_on
description: O **Macro Core Service** é o cérebro quantitativo do ecossistema MacroForce. Ele opera como uma API "Headless" (sem interface própria) que consome dados de mercado, processa algoritmos matemáticos (Scoring Inteligente, Volatilidade) e fornece um **Dashboard Unificado (JSON)** para o Front-end React.
---

# Product Requirements Document (PRD) - Macro Core Service v4.6

## 1. Visão do Produto
O **Macro Core Service** é o cérebro quantitativo do ecossistema MacroForce. Ele opera como uma API "Headless" (sem interface própria) que consome dados de mercado, processa algoritmos matemáticos (Scoring Inteligente, Volatilidade) e fornece um **Dashboard Unificado (JSON)** para o Front-end React.

**Arquitetura de Motores Independentes:**
1.  **Macro-D Engine (Smart Scoring):** Consome dados em tempo real do *Ingester* (33 ativos brutos) para medir o fluxo com ponderação inteligente.
2.  **Volatility Engine (Grid):** Consome dados históricos da *AwesomeAPI* (USD-BRL) para calcular níveis de preço.
3.  **TrendEngine (Context):** Calcula o desvio estatístico (Z-Score) para validar a qualidade do fluxo.

---

## 2. Requisitos Funcionais (RFs)

### [RF-01] Classificação e Scoring (Motor Macro-D)
O sistema deve processar **todos os 33 ativos** recebidos do Ingester. A variação destes ativos compõe o cálculo de fluxo (Risco vs Segurança).

#### **1. Ativos de Risco (Risk ON)**
*(Regra de Correlação: Alta destes ativos = Pressão de BAIXA no Dólar)*

* **Commodities (Indicadores de Atividade Econômica):**
    * `CL` (Petróleo WTI)
    * `GC` (Ouro)
    * `HG` (Cobre)
    * `ZS` (Soja)
    * `I0.DCE` (Minério de Ferro - Dalian)
* **Índices e Futuros Globais:**
    * `.GDOW` (Global Dow)
    * `YMH26` (Dow Jones Futuro - Contrato Março/26)
    * `.OSEAX` (Oslo Benchmark - Proxy de Energia)
    * `.BSESN` (SENSEX Índia)
* **ETFs e Setores (S&P Spiders):**
    * `XLF` (Financeiro)
    * `XLP` (Consumo Básico)
    * `XLE` (Energia)
    * `XME` (Metais e Mineração)
    * `SOXX.O` (Semiconductores - Tech Risk)
    * `EEM` (Mercados Emergentes)
* **Ativos Brasileiros (ADRs/ETFs):**
    * `VALE.K` (ADR Vale)
    * `PBR` (ADR Petrobras)
    * `EWZ` (ETF MSCI Brazil)
* **Bloco China (Agregador Asiático):**
    * *Componentes:* `HSIQF6`, `CHINA50`, `.DJSH`, `.SZI`, `.SSEC`.
    * *Tratamento:* Média aritmética dos 5 ativos conta como **1 único ativo sintético**.

#### **2. Ativos de Segurança (Risk OFF)**
*(Regra de Correlação: Alta destes ativos = Pressão de ALTA no Dólar)*

* **Volatilidade e Dólar Global:**
    * `VX` (VIX - Índice do Medo)
    * `DX` (DXY - Dólar Index)
* **Pares de Moedas (Forex - USD vs Mundo):**
    * `USD/MXN` (Peso Mexicano)
    * `USD/NOK` (Coroa Norueguesa)
    * `USD/NZD` (Dólar Neozelandês)
    * `USD/AUD` (Dólar Australiano)
    * `USD/KRW` (Won Sul-Coreano)
    * `USD/CNY` (Yuan Chinês Offshore)
    * `EUR/BRL` (Euro vs Real)

---

### [RF-02] Regras de Pontuação (Thresholds Imutáveis)
Se a variação do ativo romper o limite, soma 1 ponto base para a direção correspondente. Caso contrário, é considerado Neutro (0). **Esta regra define a entrada do ativo no cálculo, mas não seu peso final.**

1.  **Regra Específica - Minério de Ferro (`I0.DCE`):**
    * Threshold: **3.00%**
    * *Pontuação Base:* ±1 ponto.

2.  **Regra Específica - VIX (`VX`):**
    * Threshold: **5.00%**
    * *Pontuação Base:* ±1 ponto.

3.  **Regra Padrão (Demais 31 ativos):**
    * Threshold: **0.30%**
    * *Pontuação Base:* ±1 ponto.

---

### [RF-03] Sanitização de Tickers
Os dados brutos do Ingester podem conter descrições (ex: `USD/MXN - Dólar Americano...`).
* **Regra:** O sistema deve cortar a string no primeiro separador " - ", utilizando apenas o código do ativo para o match com as regras.

---

### [RF-04] Pré-Mercado: Cálculo de Variação Estendida
**Descrição:** Para ativos com pré-mercado (ADRs e ETFs), calcula a variação % usando preço estendido se disponível.
**Ativos Elegíveis:** `VALE.K`, `PBR`, `EWZ`, `XLF`, `XLP`, `XLE`, `XME`, `EEM`, `SOXX.O`.

**Fórmula:**
Se `precoEstendido` != null: `((PrecoEstendido - PrecoAnterior) / PrecoAnterior) * 100`.
Caso contrário: considera neutro ou variação padrão.

---

### [RF-05] Agregador China
Tratamento de bloco para os índices asiáticos para evitar peso excessivo da região.
* **Input:** `HSIQF6`, `CHINA50`, `.DJSH`, `.SZI`, `.SSEC`.
* **Lógica:** Média aritmética das variações percentuais.
* **Peso:** O resultado consolidado conta como a variação de **1 único ativo de Risco**.
* **Integridade:** Se um componente for nulo, o bloco retorna neutro (0).

**Fórmula:** `varChina % = (varHangSeng + varChinaA50 + varDJShanghai + varSZSE + varShanghaiComp) / 5`

---

### [RF-06] Indicadores de Fluxo Ponderados (Smart Momentum)
O sistema deve implementar uma arquitetura de "Separation of Concerns" (Separação de Responsabilidades), distinguindo claramente a **Contabilidade de Tendência** (Saldo) da **Física de Movimento** (Oscilador), utilizando uma frequência de atualização híbrida.

* **Frequência de Execução (Cron):** A cada **1 minuto** (Cron Trigger: `0 */1 * * * *`).

#### **Estratégia Híbrida de Atualização:**
Para garantir fluidez visual no gráfico sem gerar ruído na tendência acumulada, o sistema deve operar com duas frequências lógicas:
1.  **Oscilador (Física):** Calculado e emitido em **todo ciclo (1 min)**.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZackDataScientist/macroforce-backend](https://github.com/ZackDataScientist/macroforce-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
