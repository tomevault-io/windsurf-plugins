---
trigger: always_on
description: **Projeto:** Painel de Controle de Acordos de Cooperação Técnica e Convênios — DEPI / FVS-RCP
---

# CLAUDE.md — Registro Histórico do Projeto

**Projeto:** Painel de Controle de Acordos de Cooperação Técnica e Convênios — DEPI / FVS-RCP
**Responsável:** Walter Oliva Pinto Filho Segundo
**Diretoria:** DEPI — Diretoria de Ensino, Pesquisa e Inovação / FVS-RCP
**Plataforma:** GitHub Pages (site estático) + GitHub Actions (automação de email mensal)

---

## Visão Geral

Painel web estático hospedado no GitHub Pages para monitoramento de vigência e renovação de instrumentos jurídicos (ACTs, Convênios e Termos de Colaboração) da DEPI/FVS-RCP. O sistema envia automaticamente, todo dia 1º de cada mês, um resumo por email para os gerentes responsáveis.

---

## Estrutura de Arquivos

```
/
├── index.html                        # Painel web (SPA estática, sem backend)
├── CLAUDE.md                         # Este arquivo — registro histórico
├── requirements.txt                  # Dependências Python (nenhuma externa)
├── .gitignore
├── assets/
│   ├── fvs-logo.png                  # Logo FVS-RCP
│   ├── fvs-logo_page-0001.jpg        # Logo FVS-RCP (alternativo)
│   └── logo_depi.jpeg                # Logo DEPI
├── data/
│   └── tbl_instrumentos.csv          # Base de dados dos instrumentos (editada manualmente)
├── scripts/
│   ├── monitor_act.py                # Lê o CSV e gera resumo_execucao.json
│   └── send_email.py                 # Lê o JSON e envia email via SMTP
└── .github/
    └── workflows/
        └── relatorio_mensal.yml      # GitHub Actions: roda dia 1º de cada mês
```

---

## Como o Sistema Funciona

### Painel Web (`index.html`)
- Página HTML única com JavaScript puro (sem framework)
- Lê `data/tbl_instrumentos.csv` via PapaParse (CDN)
- Gráficos via Plotly.js (CDN)
- Funcionalidades: KPIs, gráfico de status (donut), gráfico de vencimentos por mês (barra), mapa do Brasil por UF, gráfico de diretorias, tabela com filtros e busca, detalhamento expandível por instrumento
- Classificação de vigência:
  - **VIGENTE**: termina em mais de 180 dias
  - **ALERTA 180 DIAS**: termina em até 180 dias
  - **SEM DATA**: sem data de vencimento cadastrada
  - **ARQUIVADO**: oculto do painel (campo `arquivado = SIM`)

### Automação Mensal (GitHub Actions)
Executa em dois passos sequenciais:
1. `monitor_act.py` — lê o CSV, classifica os instrumentos em faixas de 60 e 180 dias, salva `data/resumo_execucao.json`
2. `send_email.py` — lê o JSON e envia o resumo por email via SMTP

**Classificação do email (diferente do painel):**
- Confortável: > 180 dias
- Atenção: 61–180 dias
- Crítico: ≤ 60 dias
- Vencido: data já passou
- Sem data: sem registro de vigência

---

## Configuração de Secrets no GitHub

Todos os parâmetros sensíveis são gerenciados via **GitHub Settings → Secrets and variables → Actions**:

| Secret       | Descrição                                              | Obrigatório |
|--------------|--------------------------------------------------------|-------------|
| `SMTP_USER`  | Email remetente (login no servidor SMTP)               | Sim         |
| `SMTP_PASS`  | Senha de app (não a senha normal da conta)             | Sim         |
| `SMTP_TO`    | Emails destinatários, separados por vírgula            | Sim         |
| `SITE_URL`   | URL completa do painel (ex: https://usuario.github.io/...) | Sim     |
| `SMTP_HOST`  | Servidor SMTP (padrão: smtp.gmail.com)                 | Não         |
| `SMTP_PORT`  | Porta SMTP (padrão: 587)                               | Não         |

### Como gerar senha de app no Gmail
1. Acesse sua conta Google → Segurança → Verificação em duas etapas (deve estar ativa)
2. Em "Senhas de app", crie uma nova para "Email" / "Outro"
3. Use a senha gerada como `SMTP_PASS`

---

## Como Atualizar a Base de Dados

1. Edite `data/tbl_instrumentos.csv` diretamente no GitHub (aba "Edit this file") ou localmente e faça push
2. O painel atualiza automaticamente (sem rebuild — é lido em tempo real)
3. O email mensal usará os dados na próxima execução do dia 1º

### Colunas do CSV

| Coluna                  | Descrição                                              |
|-------------------------|--------------------------------------------------------|
| `id_instrumento`        | Número sequencial interno                              |
| `tipo_instrumento`      | ACT / Convênio / TC                                    |
| `identificacao`         | Número do instrumento (ex: 20/2023)                    |
| `instituicao_parceira`  | Nome da instituição parceira                           |
| `diretoria_responsavel` | Diretoria da FVS-RCP responsável                       |
| `gerencia_responsavel`  | Gerência responsável                                   |
| `numero_processo`       | Número do processo SEI                                 |
| `vigencia_inicio`       | Data de início (formato DD/MM/AAAA)                    |
| `vigencia_termino`      | Data de término (formato DD/MM/AAAA) — campo crítico   |
| `arquivado`             | SIM ou NÃO — se SIM, oculta do painel                  |
| `possui_aditivo`        | SIM ou NÃO                                             |
| `quantos_aditivos`      | Quantidade de aditivos (número)                        |
| `data_assinatura`       | Data de assinatura do instrumento                      |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DEPI-FVS-RCP/Controle_de_Acordos_de_Cooperacao_Tecnica_e_Convenio_DEPI_FVS_RCP](https://github.com/DEPI-FVS-RCP/Controle_de_Acordos_de_Cooperacao_Tecnica_e_Convenio_DEPI_FVS_RCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
