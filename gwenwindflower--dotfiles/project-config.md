---
trigger: always_on
description: Cross-platform dotfiles managed with chezmoi. Fish shell and Neovim running on Kitty terminal, with Catppuccin Frappe theming throughout.
---

# chezmoi Dotfiles Agent Guidance

Cross-platform dotfiles managed with chezmoi. Fish shell and Neovim running on Kitty terminal, with Catppuccin Frappe theming throughout.

OSes supported:

- **macOS:** Full interactive workstation — GUI apps, fonts, terminal emulators, all tools
- **Linux:** Dev-focused CLI toolkit for VMs (exe.dev, Fly.io Sprites) and containers

> [!IMPORTANT]
> You will need to run `chezmoi apply` on new changes for them to propagate to the system. Because of the potentially destructive nature of this command, `chezmoi apply` is under `ask` permissions, but is allowed with the `-n` dry-run flag for testing changes.

## Repo Structure

```text
.chezmoidata/packages.yaml        # Packages: darwin (homebrew + uv), linux (apt). JS/TS globals live in symsource_mise/config.toml.
.chezmoiscripts/                  # Lifecycle scripts (bootstrap, taps, packages, global tools, mise install, shell, yazi plugins, bat cache, ephemeral symlink materialization)
.chezmoiignore                    # Excludes dev files + OS-conditional dirs
.chezmoitemplates/fish/           # Fish config fragment templates (assembled into config.fish)
.chezmoitemplates/agents/         # Shared agent prompt/rule fragments (assembled into platform guidance files)

private_dot_config/               # → ~/.config/
  fish/                           #   config.fish.tmpl + exact_functions/ + exact_completions/ + exact_conf.d/
  nvim/                           #   LazyVim: lua/, snippets/; lockfile + spellfile symlinks back to symsource_nvim/
  kitty/                          #   Darwin-only (excluded on linux via .chezmoiignore)
  karabiner/                      #   Darwin-only
  tmux/                           #   tmux.conf + statusline + pane-icon script
  starship.toml                   #   Cross-platform prompt
  bat/, fzf/, ripgrep/, tlrc/     #   CLI tool configs
  yazi/                           #   File manager (package.toml symlinked, rest copied)
  mise/, uv/                      #   Language version managers (mise config.toml symlinked)
  delta/, gh/, gh-dash/, meteor/  #   Git ecosystem
  git/                            #   Gitconfig fragments included by ~/.gitconfig (os.gitconfig.tmpl, aliases.gitconfig, pretty.gitconfig)
  opencode/                       #   OpenCode config (opencode.jsonc + tui.jsonc + exact_agents/)
  bottom/, cmus/, freeze/, glow/  #   System monitor, music player, code snapshots, markdown viewer
  k9s/, lazydocker/, lazygit/     #   Container/cluster/git TUI tools
  lsd/, macchina/                 #   ls replacement, system info
  marimo/, spotify-player/        #   Python notebooks, Spotify TUI
  worktrunk/                      #   Worktrunk config

dot_claude/                       # → ~/.claude/
  keybindings.json, statusline.toml  # Copied normally
  exact_hooks/, exact_agents/     # Pruned-on-apply collections
  symlink_settings.json.tmpl      #   → symsource_claude/settings.json
  symlink_skills.tmpl             #   → ~/.agents/skills (post-apply target)

dot_agents/                       # → ~/.agents/ (shared agent hub)
  AGENTS.md.tmpl                  #   → ~/.agents/AGENTS.md, assembled from .chezmoitemplates/agents/
  exact_rules/                    #   → ~/.agents/rules/ generated from .chezmoitemplates/agents/rules/
  exact_skills/                   # Pruned-on-apply skill collection

symlink_dot_gitconfig.tmpl        # → symsource_git/gitconfig (externally writable; native git [include]s pull fragments from ~/.config/git/)
dot_gitignore_global              # → ~/.gitignore_global
dot_bashrc, dot_zshrc             # Minimal configs (worktrunk init, starship, zoxide)
dot_profile.tmpl, dot_zprofile.tmpl  # Login shells (SHELL export, darwin SSH agent)
private_dot_ssh/                  # → ~/.ssh/ (allowed_signers)

# Symlink source dirs (in .chezmoiignore as `symsource_*/`, not deployed as ~/*)
symsource_nvim/                   # lazy-lock.json, lazyvim.json, spell/en.utf-8.add{,.spl}
symsource_claude/                 # settings.json
symsource_yazi/                   # package.toml
symsource_mise/                   # config.toml
symsource_uv/                     # .python-version
symsource_aube/                   # config.toml
symsource_amoxide/                # config.toml, profiles.toml
symsource_worktrunk/              # config.toml
symsource_git/                    # gitconfig (root config; [include]s ~/.config/git/*.gitconfig fragments)
```

## Key Patterns

### OS conditionals

Two mechanisms, use whichever fits:

- **In `.tmpl` files:** `{{ if eq .chezmoi.os "darwin" }}...{{ end }}`
- **In `.chezmoiignore`:** Exclude entire dirs on non-darwin (kitty, karabiner)

> [!TIP]
> When a `.tmpl` file's entire body is OS-gated, the wrong-OS render evaluates to empty — and chezmoi removes empty files by default (only `empty_`-prefixed sources stay). `private_dot_config/git/os.gitconfig.tmpl` uses this: darwin gets a real `~/.config/git/os.gitconfig`, linux gets nothing on disk. Combined with git's tolerance for missing `[include]` paths, that's a clean OS split with no `if`-wrapped consumer.

### Symlinks: only for externally-modified files


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gwenwindflower/dotfiles](https://github.com/gwenwindflower/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
