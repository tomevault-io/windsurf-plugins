---
trigger: always_on
description: Este arquivo existe para alinhar o trabalho entre o projeto, eu e você.
---

# AGENTS.md

## Objetivo deste arquivo

Este arquivo existe para alinhar o trabalho entre o projeto, eu e você.
Ele resume o sistema, aponta as fontes de verdade e registra o jeito mais seguro de executar tarefas sem perder tempo tentando caminhos que hoje nao funcionam bem.

## Resumo do projeto

- Nome do projeto: `nexo-imoveis`
- Stack principal: `Next.js 16`, `React 19`, `TypeScript`, `Tailwind CSS`, `Supabase`
- Existe codigo legado com `better-sqlite3`, mas a modelagem atual importante do sistema esta no `Supabase`
- O projeto e focado em imoveis/leiloes, com area publica, area admin, autenticacao, arquivos, imagens e dados detalhados dos imoveis

## Estrutura principal

- `app/`: rotas e paginas do Next.js
- `components/`: componentes visuais e formularios
- `lib/`: regras de negocio, auth, integracoes e acesso a dados
- `database/schema.sql`: referencia atual da estrutura do banco no Supabase
- `database/seeds/`: local correto para criar arquivos de alteracao de banco
- `docs/`: documentacao complementar
- `public/`: arquivos estaticos

## Fonte de verdade do banco

### Regra mais importante

O arquivo [database/schema.sql](/C:/Projetos/nexo/nexo-imoveis/database/schema.sql) **nao pode ser editado diretamente**.

Alteracoes no banco devem ser executadas por voce.
Eu posso consultar livremente a estrutura, ler o `schema.sql`, analisar queries, criar o script SQL dentro de `database/seeds` e orientar a mudanca, mas a aplicacao da alteracao no Supabase fica com voce.

Se houver qualquer necessidade de alteracao no banco:

1. Criar um novo arquivo SQL em [database/seeds](/C:/Projetos/nexo/nexo-imoveis/database/seeds)
2. Eu preparo esse arquivo SQL em `database/seeds`
3. Voce aplica esse script no Supabase
4. So depois atualizar o [database/schema.sql](/C:/Projetos/nexo/nexo-imoveis/database/schema.sql) para refletir o estado real do banco

### Fluxo correto para mudancas de banco

- Nunca sair editando `schema.sql` como se fosse migration
- Sempre criar um arquivo novo em `database/seeds`
- Eu devo escrever e preparar esse arquivo SQL para voce em `database/seeds`
- A execucao da alteracao no banco fica com voce
- O `schema.sql` deve espelhar o banco real, nao antecipar mudancas
- Se existir divergencia entre codigo e banco, validar primeiro no Supabase

### Observacao importante

O arquivo [docs/database.md](/C:/Projetos/nexo/nexo-imoveis/docs/database.md) ajuda como documentacao conceitual, mas hoje ele parece estar parcialmente desatualizado em relacao ao [database/schema.sql](/C:/Projetos/nexo/nexo-imoveis/database/schema.sql).

Quando houver conflito:

- confiar primeiro em `database/schema.sql`
- depois conferir o uso real nas queries do `lib/`
- usar `docs/database.md` como apoio, nao como fonte final

## O que funciona hoje

- O acesso principal a dados do sistema atual passa pelo `Supabase`
- Arquivos como [lib/supabase.ts](/C:/Projetos/nexo/nexo-imoveis/lib/supabase.ts), [lib/supabase/admin.ts](/C:/Projetos/nexo/nexo-imoveis/lib/supabase/admin.ts) e [lib/admin/imoveis.ts](/C:/Projetos/nexo/nexo-imoveis/lib/admin/imoveis.ts) mostram esse caminho
- O admin usa fortemente tabelas como `imoveis`, `imovel_imagens`, `imovel_arquivos`, `imovel_detalhes`, `chat_conversas` e `chat_mensagens`
- O projeto roda localmente com `npm run dev`
- O ambiente depende de variaveis em `.env.local`, especialmente as do Supabase e segredos da aplicacao
- Para explorar, editar e validar tarefas, o caminho com mais sucesso costuma ser usando `PowerShell`

## O que nao deve ser assumido como principal

- [lib/db.ts](/C:/Projetos/nexo/nexo-imoveis/lib/db.ts) usa `better-sqlite3` e parece legado ou paralelo ao fluxo principal
- O arquivo `database.sqlite` existe, mas nao deve ser tratado como fonte de verdade do banco atual
- O `README.md` atual esta generico e nao descreve bem o estado real do projeto
- A pasta [database/seeds](/C:/Projetos/nexo/nexo-imoveis/database/seeds) esta vazia neste momento, entao novas alteracoes precisam comecar por ela

## Regra operacional para evitar retrabalho

Se uma tarefa envolver banco, auth, admin, upload, imagens ou dados dos imoveis:

- assumir primeiro que a resposta esta no `Supabase`
- eu tenho liberdade para consultar a estrutura e o codigo relacionado ao banco
- conferir `schema.sql`
- conferir o uso real em `lib/admin/*`, `lib/auth/*` e `lib/supabase/*`
- evitar tomar decisoes com base apenas em `SQLite`, `README` antigo ou documentacao parcial

## Melhor forma de eu executar tarefas neste projeto

### Abordagem preferida

1. Ler a estrutura do repositorio
2. Confirmar a fonte de verdade envolvida
3. Fazer alteracoes pequenas e objetivas
4. Validar com comandos no `PowerShell`
5. Quando houver mudanca de banco, criar SQL em `database/seeds`

### Ferramentas e comandos que costumam funcionar melhor

- listar arquivos:
  - `Get-ChildItem -Force`
  - `Get-ChildItem -Recurse`
- buscar texto no projeto:
  - `rg "texto" app components lib database docs`
- abrir conteudo de arquivo:
  - `Get-Content caminho\\arquivo`
- rodar projeto:
  - `npm run dev`
- build:
  - `npm run build`
- checar scripts disponiveis:
  - `Get-Content package.json`

### Quando priorizar PowerShell


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pitter775/nexo-imoveis](https://github.com/pitter775/nexo-imoveis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
