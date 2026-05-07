---
trigger: always_on
description: cd /Users/bardi/Projetos/hapai
---

# hapai — Prompt Orquestrador (Cole no Claude Code CLI)

## Como usar

```bash
cd /Users/bardi/Projetos/hapai
claude
```

Então cole o bloco abaixo inteiro no chat. O Claude Code vai lançar os subagentes
em paralelo automaticamente usando a ferramenta Task interna.

---

## PROMPT ORQUESTRADOR — cole inteiro no Claude Code

```
Você é o orquestrador de um plano de fixes para o repositório hapai.
Seu trabalho é lançar subagentes em paralelo usando a ferramenta Task,
aguardar cada wave completar, e depois lançar a próxima wave.

Contexto do projeto:
- hapai é um sistema de guardrails determinísticos para AI coding assistants
- Stack: Bash puro nos hooks + Svelte 5 dashboard + GCP
- Repo: /Users/bardi/Projetos/hapai
- Branch protegida: main — nunca commitar direto nela
- Commits: conventional commits, sem Co-Authored-By, sem mencionar AI/Claude
- Testes: bash tests/run-tests.sh

Diagnóstico já feito — problemas identificados:

P0-A: bin/hapai linha 19 define HAPAI_ROOT como pai do binário. Quando instalado em
/usr/local/bin/hapai, HAPAI_ROOT=/usr/local — hooks não existem lá. Fix: após linha 19,
adicionar fallback para HAPAI_HOME quando HAPAI_ROOT/hooks não existir.
Também: ensure_jq() hardcoded para "brew install jq" mesmo em Linux.
Também: cmd_sync faz source de _lib.sh sem guard de existência.

P0-B: install.sh — quando instala em ~/.local/bin (fallback sem sudo), PATH não é
atualizado. Usuário fica sem acesso ao binário. Fix: detectar shell e appendar ao rc file.
Também: macOS tem Bash 3.2 por padrão — o warn atual não bloqueia, mas deveria dar
exit 1 com instrução clara de "brew install bash".
Também: post_install não verifica se hooks foram realmente copiados.

P0-C: templates/settings.hooks.json — guard-branch.sh está registrado 3 vezes para
"Bash(gh api*)" (linhas 33-48). Causa triple-execution por operação. Fix: manter apenas 1.

P1-A: infra/gcp/functions/main.py — CORS origins hardcoded para renatobardi.github.io.
Quem faz self-host do dashboard tem CORS bloqueado. Fix: ler de env var CORS_ORIGINS.

P1-B: Documentação de comunidade open-source faltando: CONTRIBUTING.md, issue templates,
PR template, e badges no README.

---

WAVE 1 — Lance estes 3 subagentes em PARALELO (use Task para todos de uma vez):

Task A — "fix/hapai-root-standalone":
  Crie a branch fix/hapai-root-standalone a partir de main.
  
  Arquivo: bin/hapai
  
  Fix 1 — Após linha 19 (HAPAI_ROOT="..."), inserir:
    # Fallback: if running as installed binary (not inside repo), use HAPAI_HOME
    if [[ ! -d "$HAPAI_ROOT/hooks" ]]; then
      if [[ -d "$HAPAI_HOME/hooks" ]]; then
        HAPAI_ROOT="$HAPAI_HOME"
      else
        echo "hapai: hooks not found at $HAPAI_ROOT or $HAPAI_HOME" >&2
        echo "Run: curl -fsSL https://raw.githubusercontent.com/renatobardi/hapai/main/install.sh | bash" >&2
        exit 1
      fi
    fi
  
  Fix 2 — Função ensure_jq() (~linha 53), substituir a mensagem de erro fixa por:
    local os_hint
    case "$(uname -s | tr '[:upper:]' '[:lower:]')" in
      darwin*) os_hint="brew install jq" ;;
      linux*)  os_hint="apt-get install -y jq  # ou: dnf install jq / pacman -S jq" ;;
      *)       os_hint="see https://jqlang.github.io/jq/download/" ;;
    esac
    log_error "jq is required but not installed. Install: $os_hint"
    exit 1
  
  Fix 3 — Função cmd_sync (~linha 823), antes do "source", adicionar:
    if [[ ! -f "$HAPAI_ROOT/hooks/_lib.sh" ]]; then
      log_error "hooks/_lib.sh not found. Run: hapai install --global"
      exit 1
    fi
  
  Após editar: rodar bash tests/run-tests.sh e confirmar que passa.
  Commitar os 3 fixes juntos: "fix(bin): HAPAI_ROOT fallback for standalone install, OS-aware jq hint, _lib guard"
  NÃO abrir PR — apenas commitar na branch.

Task B — "fix/installer-robustness":
  Crie a branch fix/installer-robustness a partir de main.
  
  Arquivo: install.sh
  
  Fix 1 — Função install_from_github, no bloco que instala em ~/.local/bin (quando cp
  para /usr/local/bin falha e sudo não está disponível), após o cp e o log_warn de PATH,
  adicionar o auto-fix de PATH:
    local shell_rc=""
    case "$(basename "${SHELL:-bash}")" in
      zsh)  shell_rc="$HOME/.zshrc" ;;
      bash) shell_rc="$HOME/.bashrc" ;;
      fish) shell_rc="$HOME/.config/fish/config.fish" ;;
    esac
    if [[ -n "$shell_rc" ]] && [[ -f "$shell_rc" ]]; then
      if ! grep -q 'HOME/.local/bin' "$shell_rc" 2>/dev/null; then
        printf '\n# hapai — added by installer\nexport PATH="$HOME/.local/bin:$PATH"\n' >> "$shell_rc"
        log_ok "Added ~/.local/bin to PATH in $shell_rc"
        log_warn "Restart your terminal or run: source $shell_rc"
      fi
    fi
  
  Fix 2 — Função check_deps, substituir o bloco do bash version check por:
    local bash_major="${BASH_VERSINFO[0]:-3}"
    if [[ "$bash_major" -lt 4 ]]; then
      local os
      os="$(detect_os)"
      if [[ "$os" == "darwin" ]]; then
        log_error "Bash ${BASH_VERSION} detected. hapai requires Bash 4+."
        log_info "macOS ships with Bash 3.2 (GPL license constraint). Fix:"
        log_info "  brew install bash"
        log_info "Then re-run:"
        log_info '  /opt/homebrew/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/renatobardi/hapai/main/install.sh)"'
        exit 1
      else

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [renatobardi/hapai](https://github.com/renatobardi/hapai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
