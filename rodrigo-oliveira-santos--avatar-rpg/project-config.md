---
trigger: always_on
description: **Updated:** 2026-06-30
---

# Avatar RPG — GitHub Copilot Instructions

**Updated:** 2026-06-30

## Stack
- Frontend: HTML5 + CSS3 + JS ES6 modules (no framework, no bundler)
- Backend: Netlify Functions + Supabase (PostgreSQL) + Supabase Realtime
- Default: localStorage; opt-in Supabase via `public/config.js` (`useSupabase: true`) or `?supabase=1`
- Hosting: Netlify free tier
- **Cross-Origin Isolation** activado via `public/serve.json` (dev) +
  `netlify.toml` (prod) — necessário para o iframe Godot (mapa do Hub
  Avatar) usar SharedArrayBuffer.

## Status: Multi-game platform (Avatar + D&D + Minecraft) + admin cross-app

Três apps independentes a partilhar landing, login overlay e
infraestrutura (router, registry de utilizadores, persistência).

| App | Status | Tabs |
|-----|--------|------|
| Avatar | Fases 1-6 + GM tooling + combat + cross-browser trades + GM delegated modals + Hub map | Personagem · Skills (×5) · Itens · Loja · Hub · Importar · Admin · GM Control · Monstros |
| D&D 5e | MVP + Admin | Ficha · Perícias · Magias · Inventário · Trade · Hub · Importar · Admin |
| Minecraft | MVP + Admin | Galeria · Painel Pessoal · Minhas Listas · Admin |
| Landing | Game selector + Admin Global | botão admin se sessão admin activa |

### Recente (2026-06-30) — Combat tweaks + chi regen + item modifiers

- **Vocabulário de combate**: "ronda" → **Turno** (loop completo);
  "turno" individual → **Vez** (slot de cada combatente). Apenas
  strings de UI; colunas `current_round` / `current_turn_index`
  ficam como estão. Botões agora: `Próxima vez →`, `Fim da minha vez`,
  badge `Turno N`.
- **Status effects no fim da vez**: DoT (sangrando, queimadura,
  regeneração) movidos de `tick_when:'start'` para `'end'` — efeitos
  aplicam-se depois das ações do alvo. Effects de bloqueio de ação
  (stun/paralisia/medo/congelado) ficam em `'start'`. Default no
  engine (`processEffects`) também passou explicitamente a `'end'`.
- **Chi regen +20 a cada 2 turnos**: novo `combat/regen.js` cobre
  jogadores (via `updateVitals`) e monstros (via `tickPatch` quando
  têm `cp_max` definido). Disparado em rondas 3/5/7… no GM
  "Próxima vez" e no jogador "Fim da minha vez". Toast resume
  quantos foram afetados.
- **Monsters chi pool**: nova migração
  `20260630000000_monsters_chi.sql` adiciona `cp_max` + `cp_current`
  (nullable). Editor de monstros tem helper `_numberNullable` — em
  branco = monstro não usa chi (regen ignora-o).
- **Item modifiers**: `validators.js` aceita `item.modifiers`
  (string, opcional). UI mostra bloco "Modificadores" + aviso
  `⚠ Cálculos automáticos pendentes no backend.` no detalhe do item.
- **Skill chi_cost top-level**: `validators.js` aceita
  `skill.chi_cost` (number ≥ 0). `SkillCard` renderiza chip
  `Chi: N` ao lado do tier. Per-attack `attack.chi_cost` continua
  inalterado.
- **Ferramentas GM colapsáveis**: HubPage renomeou "Ferramentas GM
  (simulado)" → "Ferramentas GM" e ganhou botão
  `▲ Esconder / ▼ Mostrar` (mesma UX do mapa, persistido em
  `localStorage.avatar_rpg_hub_gmtools_collapsed`).

### Recente (anterior 2026-06-30) — Multi-game platform + admin panels

- **Multi-game platform**: landing (#/) + router (`#/<game>/<page>`) +
  per-game session isolation (`avatar_rpg_user` / `dnd_user` /
  `mc_user`). Single-game mode via `?game=<id>` (`npm run
  dev:avatar|dnd|minecraft`).
- **D&D 5e MVP+Admin**: full sheet, multiclass simples, trade com
  pre-validation, pack import 4 domínios (spells / subclasses / magic
  items / races), tab Admin com modo **impersonate** (admin edita
  ficha de outro user com banner roxo).
- **Minecraft MVP+Admin**: galeria pública 2 vistas (grelha + lista),
  tags, links sociais (YT/IG), likes/dislikes, playlists "Minhas
  Listas", tab Admin com role mgmt + transferência de builds
  single/bulk.
- **Admin Global** (landing): painel agregador visível quando alguma
  sessão activa for admin. Stats por jogo + role mgmt cross-app +
  apagar conta cascata.
- **Módulo partilhado `games/lib/users-registry.js`**: registry
  `avatar_rpg_users_registry` + validação + sincronização das **4
  chaves de sessão** (`avatar_rpg_user`, `dnd_user`, `mc_user`,
  `landing_user`) quando uma role muda. Avatar `AdminPanel` delega
  aqui o role-change (bug-fix: antes só sincronizava sessão Avatar).
- **Login overlay partilhado** (`#login-overlay`) com brand+cor por
  jogo via CSS var `--login-accent` + botão "← Início" global.

### Recente (Avatar — pré-multi-game)

- **GM Control delegated modals**: `PlayerShopModal` (comprar por
  jogador), `PlayerInventoryModal` (equip/unequip/usar),
  `PlayerSkillsModal` (skill tree). Nome do jogador + 👤 abre
  `CharacterModal` read-only. `💰 Recompensas em Grupo` + `🎁 Entregar
  Loot` no header (reusa Hub).
- **Admin delete account**: cascata users + cleanup orphan trades +
  guard self / last-admin.
- **Hub Avatar map**: iframe Godot (`html.itch.zone/html/8396265/`)
  cacheado + colapsável. Requer COI; loader supabase-js movido para
  jsdelivr (envia CORP).
- **Lazy non-bender path picker**: só aparece quando user abre tab
  "Sem Dobra"; dismissable; tabs de preview antes de committar.
- **Encounters** + EncounterPanel + status-effect tick engine +
  `endOwnTurn` do lado do jogador.
- **Persistência cirúrgica**: colunas `hp_current` / `cp_current` /

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rodrigo-Oliveira-Santos/avatar-rpg](https://github.com/Rodrigo-Oliveira-Santos/avatar-rpg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
