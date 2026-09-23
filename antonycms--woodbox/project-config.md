---
trigger: always_on
description: @/home/antony/.codex/RTK.md
---

@/home/antony/.codex/RTK.md

# Woodbox — regras para agentes

## Escopo

- Aplicação desktop Electron + React para gerenciamento de bancos de dados.
- Stack principal: Electron 43, electron-vite, React 19, TypeScript, CSS Modules, Monaco Editor, Knex, PostgreSQL, MySQL e SQLite.
- `src/main`: processo principal do Electron, IPC, conexões, queries e storage local.
- `src/preload`: ponte segura entre Electron e renderer.
- `src/renderer`: interface React.
- Interface e mensagens de produto devem usar português brasileiro, salvo textos técnicos de SQL/banco.

## Regras globais

- Faça mudanças pequenas e focadas no pedido.
- Não aplique lint/format geral, apenas no que for modificado.
- Não altere linhas, imports, estilos ou arquivos fora do necessário.
- Reutilize padrões existentes antes de criar abstrações novas.
- Evite `any` quando houver tipo viável, mesmo com `strict: false` no projeto.
- Use aliases existentes:
  - `@renderer/*` dentro do renderer.
  - `@shared/*` para tipos e utilitários compartilhados no main, preload e renderer.
  - imports relativos no `src/main` e `src/preload` para os demais módulos, seguindo o padrão atual.
- Em comandos shell, siga o RTK: prefixe com `rtk`.
- Não introduza Tailwind, styled-components ou nova lib visual sem pedido explícito.

## Comandos úteis

Na raiz:

```bash
rtk npm run dev
rtk npm run typecheck
rtk npm run build
```

Typecheck separado:

```bash
rtk npm run typecheck:node
rtk npm run typecheck:web
```

Atenção: `npm run lint` executa Biome com `check --write`. Só rode se for pedido ou combinado.

## Versões e tags

- Ao gerar uma nova tag de versão, atualize antes o `version` do `package.json`.
- Faça commit da mudança de versão antes de criar a tag.
- Antes de criar a tag, analise os commits desde a última tag de versão (`git describe --tags --abbrev=0` e `git log <tag>..HEAD --oneline`).
- Gere release notes a partir desses commits, incluindo apenas mudanças perceptíveis ao usuário, como correções, melhorias de uso, desempenho, compatibilidade e recursos novos.
- Não inclua mudanças internas sem impacto direto no usuário, como documentação do projeto, refactors internos, ajustes de CI/build sem efeito no produto ou tarefas de manutenção.
- Crie tags de versão como tags anotadas, sempre com mensagem em inglês, pois a mensagem da tag é usada como descrição da Release pela GitHub Action: `git tag -a vX.Y.Z -m "..."`.
- Depois faça push do commit e da tag.

## Estrutura do projeto

```txt
src/
├── main/               # Processo principal Electron
│   ├── database/       # Conexões, dialetos, queries e execução SQL
│   ├── files/          # Operações de arquivos
│   ├── storage/        # Persistência local via electron-store
│   └── utils/          # Helpers de IPC/eventos
├── preload/            # API exposta ao renderer
├── shared/             # Contratos e utilitários compartilhados entre camadas
└── renderer/           # React frontend
    └── src/
        ├── components/ # Componentes reutilizáveis
        ├── stores/     # Estado e ações com Zustand
        ├── database/   # Tipos/helpers de banco usados no renderer
        ├── hooks/      # Hooks reutilizáveis
        ├── styles/     # Reset, tema e ícones
        ├── utils/      # Utilidades do renderer
        └── views/      # Telas principais
```

## Renderer (`src/renderer`)

### Base técnica

- Use React com componentes funcionais.
- Use `@renderer/...` para imports do renderer.
- Prefira reutilizar componentes, hooks, stores e utils existentes antes de criar novos.
- Não crie estado manual extenso quando já houver hook ou store no projeto que resolva o caso.
- Ícones devem seguir `unplugin-icons` com imports `~icons/{collection}/{name}`; quando fizer sentido, centralize em `src/renderer/src/styles/icons.tsx`.

### Estrutura e componentes

- Componentes reutilizáveis ficam em `src/renderer/src/components/<Nome>/index.tsx`.
- Estilos locais ficam em `styles.module.css` ao lado do componente.
- Views/telas principais ficam em `src/renderer/src/views/<NomeDaView>/index.tsx`.
- Hooks reutilizáveis ficam em `src/renderer/src/hooks`.
- Utils reutilizáveis ficam em `src/renderer/src/utils`.
- Quando uma view tiver componentes usados apenas nela, isole em `components/` dentro da pasta da própria view.
- Só mova para `src/renderer/src/components` quando o componente for realmente reutilizado por mais de uma view/fluxo.
- Ao criar view ou componente novo, crie uma pasta com o nome da view/componente e coloque os arquivos dentro dela.
- Estrutura padrão: `index.tsx`; quando necessário, adicionar `constants.ts`, `dtos.ts`, `types.ts`, `utils.ts`, `styles.module.css`, `hooks/` e `components/`.
- Mantenha no `index.tsx` da view apenas a orquestração principal: carregamento essencial, estados de abertura e composição da tela.
- Evite acumular componentes grandes, cards, modais, menus e helpers específicos no `index.tsx`.
- Lógicas exclusivas de modal/dropdown/painel devem ficar dentro do próprio componente.
- Operações usadas só por um modal/dropdown/painel devem ser disparadas nele pelos hooks de domínio, não na view pai.
- A view pai deve passar apenas dados mínimos de contexto para filhos, como `active`, `onClose`, ids e registro selecionado.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antonycms/woodbox](https://github.com/antonycms/woodbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
