---
trigger: always_on
description: Leitor de livros (EPUB) e artigos web em Flutter com RSVP (Rapid Serial Visual Presentation), para Android/iOS/tablet/Linux desktop.
---

# Ledor

Leitor de livros (EPUB) e artigos web em Flutter com RSVP (Rapid Serial Visual Presentation), para Android/iOS/tablet/Linux desktop.

## Comandos

```bash
flutter pub get                                    # instalar deps
dart run build_runner build --delete-conflicting-outputs  # gerar codigo (drift)
flutter gen-l10n                                   # gerar strings i18n
flutter analyze                                    # verificar erros
flutter test test/                                 # rodar testes (requer lld instalado)
flutter test --coverage test/ && python3 tool/check_coverage.py  # cobertura (gate do CI: --min 72)
flutter run                                        # rodar no device/emulador
git tag vX.Y.Z && git push --tags                  # cortar release: CI builda APK assinado + tar.gz Linux
                                                   # (bumpar version: no pubspec com +buildNumber ANTES da tag)
```

## Arquitetura

Feature-based Clean Architecture com Riverpod. Ver [docs/architecture.md](docs/architecture.md).

**Stack:** Flutter 3.x | Riverpod 2 (sem codegen) | Drift/SQLite | SharedPreferences | epub_pro | go_router | http | receive_sharing_intent (mobile-only) | google_sign_in (mobile) + googleapis_auth loopback (desktop) + googleapis (Drive v3) | flutter_secure_storage + url_launcher (desktop OAuth) | google_fonts (Lora + Inter) | fl_chart (stats) | share_plus (export PNG) | desktop_drop (Linux) | intl (DateFormat) | flutter_tts (TTS mobile/desktop) + `SpeechdSocketBackend` (TTS Linux, socket SSIP com autospawn do daemon) | audio_service (TTS background + lockscreen controls)

## Estrutura de pastas

```
lib/
  core/
    theme/        # design system editorial
      app_colors    — AppPalette dual (light/dark)
      app_theme     — AppTheme.build(brightness:) com 14+ component themes
      app_typography — Lora (serif headlines) + Inter (sans body) + tabular figures
      app_spacing   — escala 4/8/12/16/24/32/48
      app_radius    — sm(6)/md(10)/lg(16)/xl(24) + BorderRadius helpers
      app_motion    — duracoes (fast/base/slow) + curvas (standard/emphasized)
      responsive    — Breakpoints (compact/medium), DeviceType enum,
                      extensions context.isTablet/isLandscape/deviceType
    routing/      # app_router (go_router), selected_book_provider (master-detail)
    constants/    # app_constants, responsive_defaults (font scale + margins por device)
    widgets/      # section_card, skeleton_loader (shimmer com AnimationController compartilhado)
    utils/        # orp_calculator, word_timing, html_stripper, text_tokenizer,
                  # token_codec (serialização compacta do tokensJson),
                  # readability_extractor, url_utils, sync_file_name, font_mapper,
                  # image_export_service (RepaintBoundary -> PNG -> share_plus),
                  # platform_capabilities (supportsShareIntent/supportsDriveSync/isDesktop),
                  # voice_label_formatter (TtsVoice -> "Inglês (Reino Unido) · Feminina 1")
    di/           # provider overrides (appDatabaseProvider etc.)
    share/        # share_intent_handler (Android share target),
                  # desktop_drop_handler (drag-drop de EPUB/URL no Linux)
  database/       # Drift: app_database, tables/ (books, reading_progress,
                  # reading_session, cached_tokens, sync_import_failures,
                  # book_source constants), daos/
  features/
    book_library/
      presentation/
        screens/    library_screen (master-detail host, tabs, listeners)
        widgets/    book_card, library_list, library_fab, library_appbar_bottom,
                    library_skeleton, library_empty_state, library_section_header,
                    reading_progress_bar, reader_placeholder
        providers/  book_library_provider (categorized stream)
      data/         book_persistence (persistParsedBook)
    epub_import/     # parsing EPUB -> WordToken, cache de tokens no DB
    article_import/  # fetch URL -> readability -> WordToken, cache de tokens no DB
    library_sync/    # sync de biblioteca (EPUB) + progresso + settings via Google Drive
                     # (drive.file scope, pasta "Ledor" no Drive do usuario)
                     # pipeline paraleliza read/list, pula write quando nada mudou,
                     # compacta tombstones zumbis, cache de fileId no gateway
    rsvp_reader/
      domain/entities/  rsvp_state (inclui finishTicket), display_settings, word_token, chapter, bookmark
      presentation/
        screens/    rsvp_reader_screen (modes, top bar, side panel host,
                    ref.listen em finishTicket -> /books/:id/completion)
        widgets/    rsvp_word_display, context_scroll_view,
                    rsvp_paragraph_view (extraido pra testar sem engine provider;
                      aceita onWordLongPress -> converte tokens em WidgetSpan
                      pra ter onTap + onLongPress independentes),
                    rsvp_controls (dock compositor),
                    controls_shell, controls_progress_row,
                    controls_transport_row, seek_slider,
                    reader_mode_fab (seletor de modos flutuante),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daniel-pimenta-cc/ledor](https://github.com/daniel-pimenta-cc/ledor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
