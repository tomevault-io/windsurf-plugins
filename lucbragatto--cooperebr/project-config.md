---
trigger: always_on
description: **Inegociável** — toda sessão Code (Claude Code CLI) **abre** apresentando
---

# Instruções permanentes — Claude Code no CoopereBR

## Ritual de abertura e fechamento de sessão

**Inegociável** — toda sessão Code (Claude Code CLI) **abre** apresentando
"Onde paramos + Pendências" e **fecha** atualizando o mesmo registro vivo.

Formato fixo em `~/.claude/projects/C--Users-Luciano-cooperebr/memory/ritual_abertura_fechamento.md`.

Estado vivo fica em `docs/CONTROLE-EXECUCAO.md` (seção **"ONDE PARAMOS"**).

Aplica:
- Toda sessão Code, mesmo se for "continuação" ou "mesmo dia"
- claude.ai web por reflexo (Luciano cola `CONTROLE-EXECUCAO.md` ao abrir)

**Origem:** Luciano em 2026-05-02. Necessidade de não perder contexto entre
sessões e ter pendências sempre visíveis.

## Fechamento de sessão — REGRA INEGOCIÁVEL BILATERAL (13/05/2026)

Reforço da regra de ritual de fechamento. Aplicação bilateral
claude.ai + Code.

### Ao final de TODA sessão (claude.ai OU Code)

Executar 3 itens obrigatórios antes de standby:

1. **Doc-sessão em `docs/sessoes/YYYY-MM-DD-[escopo].md`** contendo:
   - TL;DR 3-5 linhas legível pra leigo
   - Lista de entregas + SHAs dos commits
   - Lista de débitos novos catalogados
   - Lista de débitos resolvidos
   - Lista de bugs descobertos durante validação
   - Pendências abertas pra próxima sessão
   - Decisões catalogadas
   - Próximo passo único e claro

2. **`docs/CONTROLE-EXECUCAO.md` atualizado:**
   - Seção "Última sessão" reescrita com resumo
   - Frase de retomada COMANDANTE atualizada (formato copy-paste pro
     Code, NÃO descritiva tipo "próxima sessão fará X")

3. **`docs/diagramas/jornada-membro.html` atualizado (Sugestão #6 da
   memória persistente claude.ai — ritual_atualizacao_jornada_diaria):**
   - Se sessão modificou estado do projeto (novo cooperado, débito,
     engine, sprint), regenerar HTML versão nova (bump v1.X)
   - Refletir mudanças: cores do diagrama, tabela de status, pontos
     críticos, próximos passos, changelog

4. **Commit + push obrigatórios**
   - Mensagem: `docs(sessao): fechamento DD/MM - [resumo curto]`
   - Push obrigatório — não deixar commits locais

### Bilateral — lembranças mútuas

**Se claude.ai esquecer (só empacotar commits de código sem fechamento
documental):**

Code, no ÚLTIMO turno antes de entrar em standby de qualquer prompt,
DEVE verificar:
- Existe `docs/sessoes/YYYY-MM-DD-*.md` correspondente ao dia atual
  refletindo o que rodou nesta sessão?
- `CONTROLE-EXECUCAO.md` seção "Última sessão" reflete realidade
  recente?
- Frase de retomada atualizada?
- Há commits locais sem push (`git log origin/main..HEAD`)?

Se algum NÃO: ANTES de standby, perguntar ao Luciano:

> "Sessão sem fechamento documentado detectado. Posso criar
> docs/sessoes/YYYY-MM-DD-[escopo].md + atualizar CONTROLE-EXECUCAO +
> push? (regra inegociável 13/05/2026)"

**Se Code esquecer (próxima sessão):**

Abertura de próxima sessão claude.ai deve checar:
- Última doc em `docs/sessoes/` é do dia anterior?
- Há gap entre data da última doc-sessão e commits recentes (`git log
  --since=...`)?

Se gap detectado: claude.ai puxa contexto via git log + cria
fechamento retroativo ANTES de qualquer trabalho novo.

### Quando dispensar

**Nunca.** Mesmo sessões curtas (15min) ou só leitura merecem
registro. Sessão de leitura gera doc curta tipo:
"Sessão leitura DD/MM 15min — consultei docs/X, decidi Y, próximo
passo Z."

### Por que essa regra existe

13/05/2026 — sessão maratona Sub-Fase A canário + D-48 + D-50..D-55
acumulou 11+ commits sem doc-sessão consolidada. Luciano apontou.
Memória persistente claude.ai estava atualizada (lado claude.ai),
mas repo do projeto estava sem rastreabilidade narrativa do que
aconteceu. Sem regra forte, débitos se acumulam invisíveis.

Detalhamento completo em memória persistente claude.ai
`~/.claude/projects/C--Users-Luciano-cooperebr/memory/regra_fechamento_sessao_inegociavel.md`.

## Contatos de teste — REGRA INEGOCIÁVEL (14/05/2026)

Para QUALQUER teste que dispare comunicação real (SMTP, WhatsApp,
Asaas boleto/PIX, notificações) OU movimentação financeira real:

**Substituir IMPRETERIVELMENTE contatos do cooperado por:**
- Telefone/WhatsApp: `27981341348`
- Email: `lucbragatto@gmail.com`

**CPF/CNPJ mantém** (Asaas valida CPF da pessoa física/jurídica real).

**Aplicação obrigatória:**
- Testes de envio Asaas (boleto/PIX em produção)
- Testes WhatsApp (notificações, lembretes, confirmações)
- Testes email (faturas, comunicações, lembretes)
- Sub-canários (CAROLINA, AMAGES, qualquer outro)
- Desativação de `ambienteTeste=true` em piloto
- Testes E2E que envolvam SMTP/IMAP/WA/Asaas

**Quando NÃO aplicar:**
- Cadastros via wizard normal (mantém dados reais + `ambienteTeste=true`)
- Testes sintéticos (whitelist LGPD já bloqueia)
- Operações read-only no banco

**Origem:** Luciano em 14/05/2026 — "todos os dados são extraídos do
sistema que temos, embora sejam reais, não podemos contatá-los, use
sempre e impreterivelmente meu celular e meu email para testes".

**Bilateral:** Code verifica antes de qualquer disparo real e pergunta
se contatos devem ser substituídos. Claude.ai catalogou em memória
persistente `regra_contato_teste_impreterivel.md`.

**Refinamento email com `+suffix` (14/05/2026):**

Luciano é cooperado real CoopereBR com email `lucbragatto@gmail.com`. Por

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucbragatto/cooperebr](https://github.com/lucbragatto/cooperebr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
