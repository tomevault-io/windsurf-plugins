---
trigger: always_on
description: **IMPORTANTE:** Sempre que iniciar qualquer sessão, leia TODOS os arquivos abaixo. Eles contêm regras e contexto essenciais para o trabalho.
---

# Dash for CF - Agent Instructions

**IMPORTANTE:** Sempre que iniciar qualquer sessão, leia TODOS os arquivos abaixo. Eles contêm regras e contexto essenciais para o trabalho.

| Arquivo        | Conteúdo                                                        |
| -------------- | --------------------------------------------------------------- |
| @./CODEBASE.md | **Mapa do código** - onde cada funcionalidade está implementada |
| @./ADR.md      | Decisões técnicas e arquiteturais                               |
| @./README.md   | Visão geral do projeto                                          |
| @./ROADMAP.md  | Tarefas e progresso                                             |

As regras deste arquivo (AGENTS.md) são **obrigatórias** e devem ser seguidas rigorosamente.

---

## Referência Rápida

| Campo               | Valor                                    |
| ------------------- | ---------------------------------------- |
| **Nome**            | Dash for CF                      |
| **Package ID**      | `ad.dash.cf`                             |
| **Web URL**         | `cf.dash.ad`                             |
| **Plataformas**     | Android, iOS, Web, Linux, macOS, Windows |
| **Plataformas Dev** | Android, Linux                           |
| **Repositório**     | github.com/verseles/dash-for-cloudflare  |

---

## Comandos Make (OBRIGATÓRIO)

Use SEMPRE comandos make. Eles suprimem logs de sucesso para economizar tokens.

| Comando            | Descrição                                | Tempo |
| ------------------ | ---------------------------------------- | ----- |
| `make check`       | Validação rápida (deps+gen+analyze+test) | ~20s  |
| `make precommit`   | Verificação completa (check+builds)      | ~30s  |
| `make android`     | Build APK (arm64) + upload via hey       | ~30s  |
| `make android-x64` | Build APK (x64 para emulador)            | ~30s  |
| `make linux`       | Build Linux release                      | ~10s  |
| `make web`         | Build Web release                        | ~20s  |
| `make test`        | Executar testes                          | ~10s  |
| `make analyze`     | Análise estática + budget gate (max 50)  | ~3s   |
| `make coverage`    | Testes + threshold gate (min 25%)        | ~15s  |
| `make icons-check` | Validar artefatos de ícones              | ~1s   |
| `make deps`        | Instalar dependências                    | ~2s   |
| `make gen`         | Gerar código (Freezed, Retrofit)         | ~5s   |
| `make clean`       | Limpar artefatos de build                | ~2s   |
| `make release V=`  | Bump versão, commit, tag, push           | ~5s   |
| `make install`     | Instalar no Linux (~/.local)             | -     |
| `make uninstall`   | Desinstalar do Linux                     | -     |

### Fluxo de Trabalho

```bash
# Durante desenvolvimento (várias vezes):
make check

# Antes de commit (uma vez):
make precommit

# Para builds específicas:
make android      # Build + upload para Telegram
make linux        # Build Linux
make web          # Build Web

# Para builds específicas:
make android      # Build + upload para Telegram
make linux        # Build Linux
make web          # Build Web

# Após alterar dependências ou models:
make deps         # Após alterar pubspec.yaml
make gen          # Após alterar models Freezed/Retrofit

# Qualidade:
make coverage     # Testes com cobertura + threshold
make icons-check  # Validar ícones

# Release:
make release V=patch  # Bump patch, commit, tag, push
make release V=minor  # Bump minor, commit, tag, push
```

---

## Regras de Trabalho

### Build & Commit

1. **`make check` durante desenvolvimento.** Validação rápida (~20s) para feedback iterativo.

2. **`make precommit` antes de QUALQUER commit.** Verificação completa incluindo builds.

3. **Commit a cada fase concluída.** Atualize o roadmap e faça commit bem descrito.

4. **Push só se já pedido na sessão.** Não faça push automaticamente, apenas se o usuário já pediu pelo menos uma vez na conversa atual.

5. **`make android` após push bem-sucedido.** Envia APK via hey para o celular do usuário testar, sempre faça esse e avise o usuário que já pode testar no celular.

6. **Se `make analyze` ou `make test` falhar, corrija TODOS os erros.** Não prossiga com erros pendentes.

7. **Incrementar Versão.** Sempre incremente a versão no `pubspec.yaml` antes de cada `make android`. O Android impede a instalação se a versão for menor ou igual ao que já está no dispositivo.

8. **Limpeza de Cache.** Se houver mudanças em modelos (`.freezed`, `.g.dart`) ou se o APK anterior apresentou erro de instalação, execute `make clean` antes de `make android` para garantir uma build íntegra.

### Desenvolvimento

7. **To-do list sempre atualizado.** Monte e mantenha a lista de fases/sub-fases pendentes.

8. **Testes atualizados.** Crie novos testes e atualize existentes conforme necessário.

9. **Refatoração livre.** Projeto não está em produção, não precisa de compatibilidade com versões anteriores.

### Web Search

10. **Use web search com frequência:**
    - Para confirmar métodos eficientes/modernos
    - Para resolver erros quando ficar preso

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [verseles/dash-for-cloudflare](https://github.com/verseles/dash-for-cloudflare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
