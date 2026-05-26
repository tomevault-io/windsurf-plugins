---
trigger: always_on
description: [![MVP Status](https://img.shields.io/badge/Status-MVP%20Ready-green)]()
---

# 🏛️ nLic - Sistema de Consulta de Dispensas de Licitação

[![MVP Status](https://img.shields.io/badge/Status-MVP%20Ready-green)]()
[![Tech Stack](https://img.shields.io/badge/Stack-React%20%2B%20FastAPI-blue)]()
[![License](https://img.shields.io/badge/License-MIT-yellow)]()

> **Aplicação web moderna para consulta eficiente de Dispensas de Licitação através da API do Portal Nacional de Contratações Públicas (PNCP)**

## 🎯 Sobre o Projeto

O **nLic** é um MVP (Produto Mínimo Viável) desenvolvido para simplificar e otimizar consultas de dispensas de licitação, oferecendo:


## Stack Recomendada: Next.js (React + API Routes)

### Visão Geral

Para acelerar o desenvolvimento do MVP e simplificar a arquitetura, recomenda-se o uso do **Next.js** como stack única. O Next.js integra:

- **Frontend:** React, com suporte opcional a SSR (Server-Side Rendering) e SSG (Static Site Generation).
- **Backend:** API Routes (Node.js), permitindo criar endpoints backend no mesmo projeto.

### Vantagens do Next.js para o nMVP

- **Projeto Unificado:** Frontend e backend no mesmo repositório, facilitando manutenção, deploy e versionamento.
- **API Proxy:** As API Routes permitem criar endpoints internos que fazem proxy para a API do PNCP, resolvendo problemas de CORS e protegendo chaves ou lógica sensível.
- **SSR/SSG Opcional:** Possibilidade de renderizar páginas no servidor para melhor SEO e performance, ou gerar páginas estáticas.
- **Exportação para Excel:** Fácil integração com a biblioteca `xlsx` (SheetJS) para exportação dos dados da tabela diretamente do frontend.
- **Comunidade Forte:** Grande ecossistema, documentação robusta e muitos exemplos disponíveis.
- **Hospedagem Simplificada:** Deploy facilitado em plataformas como **Vercel** (oficial), **Netlify** e **Railway**.
- **Desenvolvimento Local e Docker:** Next.js roda facilmente em ambientes locais e pode ser containerizado, mas atenção à versão do Node.js suportada pela imagem Docker escolhida (recomenda-se Node.js 18+ para Next.js 13/14).

### Estrutura de Projeto Sugerida

```
nMVP/
└── app/ ou pages/         # Páginas React (SSR/SSG ou SPA)
└── pages/api/             # API Routes (endpoints backend)
└── components/            # Componentes reutilizáveis React
└── package.json
└── next.config.js
└── Dockerfile (opcional)
```

### Exemplo de Integração com API do PNCP

**API Route para Proxy (pages/api/dispensas.js):**
```javascript
// filepath: /pages/api/dispensas.js
import axios from 'axios';

export default async function handler(req, res) {
  const { dataInicial, dataFinal, uf, pagina } = req.query;
  try {
    const response = await axios.get('https://pncp.gov.br/api/consulta/v1/contratacoes/publicacao', {
      params: {
        dataInicial,
        dataFinal,
        codigoModalidadeContratacao: 8,
        uf,
        pagina
      }
    });
    res.status(200).json(response.data);
  } catch (error) {
    res.status(error.response?.status || 500).json({ error: error.message });
  }
}
```

**Frontend (React) para Consulta e Exportação:**
- Use `fetch` ou `axios` para consumir `/api/dispensas`.
- Utilize a biblioteca `xlsx` para exportação dos dados para Excel.

### Hospedagem e Deploy

- **Vercel:** Deploy automático e integração nativa com Next.js.
- **Netlify:** Suporte a Next.js com funções serverless.
- **Railway:** Deploy de aplicações Node.js com facilidade.
- **Docker:** Para ambientes customizados, crie um `Dockerfile` com Node.js 18+ e configure as variáveis de ambiente conforme necessário.

> **Atenção:** Certifique-se de que a versão do Node.js utilizada no ambiente Docker ou na plataforma de hospedagem é compatível com a versão do Next.js adotada (Node.js 18+ recomendado para Next.js 13 ou superior).

- ⚡ **Consultas 10x mais rápidas** que métodos manuais
- 📊 **Dashboard intuitivo** com métricas em tempo real  
- 📤 **Exportação inteligente** para Excel/CSV
- 🔍 **Filtros avançados** para análises precisas
- 📱 **Interface responsiva** para qualquer dispositivo

## 1. Endpoint de Consulta

Para buscar as dispensas de licitação, utilize o seguinte endpoint:

- **Método HTTP:** `GET`
- **URL Base:** `https://pncp.gov.br/api/consulta`
- **Endpoint:** `/v1/contratacoes/publicacao`

## 2. Parâmetro Chave: `codigoModalidadeContratacao`

Para filtrar exclusivamente as **Dispensas de Licitação**, é obrigatório o uso do seguinte parâmetro na sua requisição:

- `codigoModalidadeContratacao=8`

Este código é definido na Tabela de Domínio **5.2. Modalidade de Contratação** do manual do PNCP.

## 3. Parâmetros de Requisição (Filtros)

Os seguintes parâmetros podem ser combinados para refinar a busca.

| Campo | Tipo | Obrigatório | Descrição |
| --- | --- | --- | --- |
| `dataInicial` | Data | Sim | Data inicial do período a ser consultado no formato `AAAAMMDD`. |
| `dataFinal` | Data | Sim | Data final do período a ser consultado no formato `AAAAMMDD`. |
| `codigoModalidadeContratacao` | Inteiro | Sim | **Deve ser `8` para Dispensa de Licitação.** |
| `uf` | String | Não | Sigla da Unidade Federativa (ex: `DF`, `SP`). |
| `codigoMunicipioIbge` | String | Não | Código IBGE do Município (ex: `5300108` para Brasília). |
| `cnpj` | String | Não | CNPJ do órgão ou entidade que publicou a contratação. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felipedsvit/nMVP](https://github.com/felipedsvit/nMVP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
