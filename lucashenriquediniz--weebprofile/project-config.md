---
trigger: always_on
description: Gerador de SVGs para perfis GitHub. O usuário configura uma GitHub Action (ou o dashboard web) e o sistema gera cards visuais com stats de código, anime e música, salvos como Gists públicos.
---

# WeebProfile — Guia para Claude

Gerador de SVGs para perfis GitHub. O usuário configura uma GitHub Action (ou o dashboard web) e o sistema gera cards visuais com stats de código, anime e música, salvos como Gists públicos.

---

## Estrutura do monorepo

```
WeebProfile/
├── weeb-plugins/        # Biblioteca core de plugins (React + TypeScript)
├── svg-generator/       # Servidor HTTP que converte React → SVG (Node.js + Playwright)
├── weeb-dashboard/      # Frontend de configuração (Next.js 15, App Router)
└── weeb-debug-tool/     # Ferramenta de desenvolvimento local para inspecionar plugins
```

Gerenciador de pacotes: **pnpm workspaces**. Nunca usar `npm` ou `yarn`.

---

## Comandos essenciais

```bash
# Desenvolvimento completo (dashboard + svg-generator em paralelo)
pnpm dev

# Pacotes individuais
pnpm dev:dashboard       # Next.js na porta padrão
pnpm dev:generator       # SVG Generator na porta 3001
pnpm dev:debug-tool      # Debug tool (frontend :5000, backend :5001)

# Build
pnpm build               # Apenas dashboard (Vercel)
pnpm build:railway       # plugins + svg-generator (Railway)
pnpm build:railway:full  # build:railway + instala Playwright + Chromium

# Qualidade de código (rodar antes de commitar)
pnpm typecheck
pnpm lint
pnpm format:check
pnpm check               # os três acima juntos

# Geração de assets (rodar após alterar plugins)
pnpm --filter @weeb/weeb-plugins run generate:metadata   # Regera metadata.ts centralizado
pnpm generate-previews                                    # Regera SVGs de preview
```

---

## Segurança — regras obrigatórias

**NUNCA** expor ou logar as seguintes informações:

- `DATABASE_URL` ou qualquer connection string com senha
- `CRON_SECRET` (autentica o endpoint de cron)
- `WEEB_GH_TOKEN` / `GH_TOKEN` (token GitHub do usuário)
- API keys de plugins (Steam `apiKey`, LastFM key, etc.)
- Qualquer valor de `essentialConfig` passado aos plugins

**Como os segredos funcionam:** dados sensíveis dos usuários (API keys, tokens) são armazenados no Supabase (`essentialConfigs`) e nunca viajam junto com a configuração pública. O `svg-generator` busca esses dados via `userId` em tempo de geração e os passa diretamente para `fetchData`. Nunca serializar ou retornar `essentialConfig` em respostas HTTP.

**Arquivos que NÃO devem ser commitados:**
- `.env` (qualquer variório de ambiente real)
- Qualquer arquivo com credentials reais

O `.env.example` do `svg-generator` contém apenas placeholders — é esse o padrão.

---

## Como adicionar um novo plugin

### 1. Criar a estrutura de arquivos

```
weeb-plugins/src/plugins/{nome}/
├── index.tsx              # Implementação da interface Plugin
├── plugin.metadata.ts     # Metadados (seções, campos, opções de config)
├── types.ts               # Tipos TypeScript específicos
├── components/
│   ├── Render{Nome}.tsx   # Componente raiz que recebe { config, data, style, size }
│   └── {Secao}.tsx        # Um componente por seção
├── services/
│   ├── fetchData.ts       # Busca dados reais da API
│   └── mock-data.ts       # Dados fake para modo dev/preview
└── previews/              # SVGs de preview (gerados automaticamente)
```

### 2. Implementar `index.tsx`

```typescript
import type { Plugin } from '../shared/types/plugin'
import type { PluginConfig, PluginData } from '../../types/index'
import type { EssentialPluginConfig } from '../shared/types/base'
import type { MinhaConfig, MeusDados } from './types'
import { RenderMeuPlugin } from './components/RenderMeuPlugin'
import { fetchMeuPlugin } from './services/fetchData'

export const meuPlugin: Plugin<PluginConfig & MinhaConfig, PluginData & MeusDados> = {
  name: 'meuplugin',                         // snake_case, único
  essentialConfigKeys: ['apiKey'],           // chaves que ficam no Supabase
  config: { enabled: false, sections: [] } as PluginConfig & MinhaConfig,

  fetchData: async (config, dev = false, essentialConfig?: EssentialPluginConfig, previewMode = false) => {
    return await fetchMeuPlugin(config, dev, essentialConfig?.apiKey, previewMode)
  },

  render: (config, data) => {
    const style = ((config as any).style || 'default') as 'default' | 'terminal'
    const size  = ((config as any).size  || 'half')    as 'half'    | 'full'
    return <RenderMeuPlugin config={config} data={data} style={style} size={size} />
  },
}
```

### 3. Implementar `plugin.metadata.ts`

Ver `steam/plugin.metadata.ts` como referência canônica. Campos obrigatórios:
- `displayName`, `description`, `category` (`"coding" | "music" | "anime" | "gaming"`)
- `icon` (nome de ícone do `lucide-react`)
- `requiredFields` — campos que vão para a config pública (ex: `["username"]`)
- `essentialConfigKeys` — segredos armazenados no Supabase (ex: `["apiKey"]`)
- `essentialConfigKeysMetadata` — descrição de cada segredo para o dashboard
- `sections[]` — cada seção com `id`, `name`, `description`, `configOptions[]`

### 4. Registrar no `PluginManager`

Arquivo: `weeb-plugins/src/plugins/manager.ts`

```typescript
import { meuPlugin } from './meuplugin/index'

// No construtor:
this.register(meuPlugin)
```

### 5. Regenerar o metadata centralizado

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LucasHenriqueDiniz/WeebProfile](https://github.com/LucasHenriqueDiniz/WeebProfile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
