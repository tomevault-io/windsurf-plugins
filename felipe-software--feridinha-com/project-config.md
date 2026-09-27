---
trigger: always_on
description: Guia de contribuição para o Feridinha.com. Estas regras orientam novas
---

# AGENTS.md

Guia de contribuição para o Feridinha.com. Estas regras orientam novas
alterações e não exigem uma refatoração completa do código legado existente.

## Visão geral do repositório

Este é um monorepo com duas aplicações principais:

- `client/`: frontend Next.js com React, TypeScript, `styled-components`, CSS
  Modules, Tailwind utilities e `next-intl`.
- `api/`: backend TypeScript executado com Bun, incluindo controllers,
  services, validações, testes e integrações externas.

Alterações de interface devem ficar em `client/`. Regras de negócio, acesso a
dados, autenticação e integrações devem ficar em `api/`. Não mover lógica entre
as duas aplicações sem necessidade clara de contrato compartilhado.

## Convenções de código

- Use TypeScript estrito e preserve os tipos existentes.
- Prefira imports pelo alias `@/` no client.
- Use componentes funcionais React e mantenha a lógica de apresentação próxima
  do componente que a utiliza.
- Evite `any`, casts desnecessários e supressões de erro.
- Preserve os padrões de internacionalização existentes. Textos visíveis não
  devem ser adicionados diretamente ao JSX quando a tela já usa mensagens
  localizadas.
- Use nomes de componentes, funções e constantes em inglês, seguindo o padrão
  atual do código.
- Não introduza uma nova biblioteca para resolver algo que já é coberto pelas
  dependências atuais.

## Organização de estilos

### Styled-components

`styled-components` é a opção preferencial para estilos específicos de
componentes React. Evite criar novos arquivos `.module.css`; use
`styled-components` e um `styles.ts` local como padrão. Arquivos CSS Modules
existentes devem ser tratados como legado e só devem ser ampliados quando não
houver uma alternativa razoável.

Exporte os elementos estilizados de um arquivo `styles.ts`
quando o componente possuir vários blocos, regras aninhadas, responsividade ou
animações.

Estrutura recomendada para componentes maiores:

```text
Component/
├── index.tsx
├── styles.ts
└── types.ts
```

Um componente pequeno pode manter um único styled component no próprio arquivo.
Separe para `styles.ts` quando houver mais de um bloco relevante, quando o
arquivo ultrapassar aproximadamente 80 linhas por causa de estilos ou quando
os estilos dificultarem a leitura da lógica React.

Use como referências de organização:

- `client/src/components/dashboard/styles.ts`
- `client/src/components/Navbar/styles.ts`
- `client/src/components/landing/UploadBox/styles.ts`

Evite misturar `styled-components`, CSS Modules e `style={{ ... }}` no mesmo
componente. A mistura só é aceitável quando houver uma razão específica, como
um valor realmente dinâmico, uma API de terceiro ou uma regra global/legada que
não possa ser encapsulada.

### Layout e alinhamento

Não use `margin` como mecanismo principal para alinhar ou distribuir elementos.
Prefira:

- `display: flex` como padrão para layouts lineares;
- `display: grid` somente quando houver uma relação real de linhas e colunas
  bidimensionais;
- `gap` para espaçamento entre elementos;
- `align-items`, `justify-content` e `place-items` para alinhamento;
- `padding` para o espaço interno de um container;
- `margin-inline: auto` somente quando o objetivo for centralizar um bloco com
  largura definida ou limitada.

Não use `grid` automaticamente em todo container. Para uma linha, coluna,
toolbar, lista, navegação ou grupo de ações, prefira `flex`. Reserve `grid` para
composições realmente bidimensionais, como galerias, tabelas visuais ou áreas
com linhas e colunas independentes.

Evite compensar desalinhamentos com margens negativas, combinações de margens
entre irmãos ou valores arbitrários. Se o layout precisar de margem para
funcionar, revise primeiro a estrutura do container e a distribuição com flex;
use grid somente quando a relação bidimensional justificar.

### Unidades

Use `rem` para espaçamentos, dimensões, tipografia, raios, offsets, sombras e
breakpoints. Isso mantém a interface consistente quando o tamanho base da
fonte muda.

```ts
const Card = styled.div`
    padding: 1rem;
    gap: 0.75rem;
    border-radius: var(--border-radius-m);
    font-size: 1rem;
`
```

Evite `px`, `em` e números mágicos em novos estilos. Exceções aceitáveis:

- linhas extremamente finas, quando `0.0625rem` não produzir resultado
  consistente;
- valores exigidos por uma API externa ou por um SVG;
- propriedades técnicas em que a unidade é definida pelo navegador, como
  certos `outline` ou `device-pixel-ratio` hacks.

Mesmo nas exceções, prefira uma variável ou um comentário curto explicando a
necessidade.

### Cores e tokens

Use variáveis CSS para todas as cores novas. Os tokens principais ficam em
`client/src/global.css`, incluindo `--base`, `--base-dark`, `--foreground`,
`--nav-highlight`, os tokens `--dracula-*` e os raios compartilhados.

Não adicione hexadecimais, `rgb()`, `rgba()` ou nomes de cores diretamente em
novos styled components. Se uma cor for reutilizável, adicione primeiro um
token semântico em `global.css`, por exemplo `--surface-muted` ou
`--focus-ring`, e use esse token nos componentes.

Prefira tokens semânticos para novas necessidades. Não crie variações quase
idênticas sem confirmar que um token existente não atende ao caso.

### Bordas e superfícies


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felipe-software/feridinha.com](https://github.com/felipe-software/feridinha.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
