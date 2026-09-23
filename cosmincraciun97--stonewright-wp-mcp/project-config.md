---
trigger: always_on
description: These rules apply to every coding agent operating in this repository. They
---

# Stonewright agent rules

These rules apply to every coding agent operating in this repository. They
override default behavior.

## Identity

- Product name: **Stonewright**.
- PHP namespace: `Stonewright\WpMcp`.
- Ability prefix: `stonewright/`.
- MCP server id: `stonewright`.
- Composer package: `stonewright/wp-mcp`.
- NPM package: `@stonewright/companion`.
- Plugin license: `AGPL-3.0-or-later`.
- Companion license: `MIT`.

## Hard rules

1. **Full PHP runtime access is first-class.** Use
   `stonewright/php-execute` for direct PHP snippets inside the loaded
   WordPress runtime. Do not replace it with another MCP adapter, direct REST
   runner calls, shell scripts, or private client-config workarounds.
2. **No `__return_true` for writes.** Every ability that writes, updates, or
   deletes state must use a real permission callback that calls into
   `Stonewright\WpMcp\Security\Permissions`. Read-only abilities may use simple
   callbacks but must still pass through the Permissions helpers.
3. **Backup before write.** Before mutating an Elementor post, a global styles
   record, a template, or any theme.json-backed content, call
   `Stonewright\WpMcp\Security\Backup::snapshot_post( $post_id )`.
4. **Validator before render.** Before handing a design spec to any renderer,
   call `Stonewright\WpMcp\DesignSpec\Validator::validate( $spec )`. Reject
   invalid specs with a structured `WP_Error` whose code is
   `stonewright_spec_invalid`.
5. **Confirmation tokens for destructive operations.** When
   `get_option( 'stonewright_mode', 'development' ) === 'production-safe'`,
   every destructive ability must verify a token via
   `ConfirmationToken::verify( $token, $ability_name, $args )`. Tokens are
   issued by `stonewright/security-issue-confirmation-token`.
6. **Mode support.** The plugin must always honor the three modes
   `development`, `staging`, and `production-safe`. The admin UI exposes the
   toggle. Permissions and ability gates read the option.
7. **Companion WP-CLI stays tokenized.** The Node companion handles WP-CLI,
   health checks, and an optional MCP HTTP proxy. It must not call WordPress
   REST write endpoints, must run WP-CLI with `execFile` argv tokens only, and
   PHP snippets must go through `stonewright/php-execute` rather than WP-CLI
   eval, shell, package, `--exec`, or `--require` entry points.
8. **Custom code stops at human approval.** For theme files, Customizer CSS,
   WPCode, Code Snippets, or any equivalent PHP/CSS/JS/HTML surface, run the
   typed dry-run first. Return `approval_url`, exact target path, byte counts,
   and a short change summary, then stop. Never open the approval page, issue
   or retrieve a grant, or apply with `custom_code_grant` unless the user
   explicitly asks the agent to perform that approval step. Pluginless Direct
   mode may inspect custom CSS but must not write it because it has no
   authenticated wp-admin grant boundary.
9. **Updater contract is part of every user-consumed release.** Do not ship a
   version, tag, or GitHub release that operators should install unless all of
   the following are true in the published artifacts (not only in source):
   plugin `Version` / `STONEWRIGHT_VERSION`, companion `package.json` and
   `companion/src/version.ts`, and the GitHub tag equal the same SemVer;
   the GitHub release body contains exactly one line ``Release channel: `supported` ``
   (or `preview` / `stable` as chosen in the release decision record);
   GitHub `prerelease` is false for `supported` and `stable`, true for `preview`;
   assets are exactly `stonewright-VERSION.zip`, `stonewright-companion-VERSION.tgz`,
   and `SHA256SUMS.txt`; `GitHubUpdater` would select that release for a site
   still on the previous same-channel version. After publish, a WordPress
   Dashboard → Updates → Check again (or `wp_update_plugins`) must be able to
   see `new_version` equal to that tag. Never treat changelog-only or
   "the ZIP exists" as enough. Never skip this checklist to save a step.
- After the plugin updates itself, release metadata caches must be
  invalidated; Plugins → View details must never render a release older
  than the installed version.
- The plugin release ZIP must bundle the built-in skill pack
  (`skills/` inside the plugin directory) and release packaging must fail
  closed when it is missing, so ZIP installs seed the same built-in
  skills as repository checkouts.
10. **GitHub release notes are untrusted Markdown.** Every updater or release
   implementation must:
   1. treat GitHub release bodies as untrusted Markdown input;
   2. keep release-channel parsing on the original raw body;
   3. render only the supported Markdown subset for WordPress View details;
   4. sanitize that HTML with an explicit `wp_kses` allowlist and an
      HTTPS-only link policy;
   5. add security and formatting regression coverage whenever the
      release-note format changes.

## Third-party source reuse

- Third-party source may be inspected, copied, adapted, or ported when its
  license permits it and the resulting Stonewright component uses compatible
  licensing.
- Preserve upstream copyright and SPDX notices in copied or derived files.
- Record source repository, source path, source version or hash, destination,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cosmincraciun97/stonewright-wp-mcp](https://github.com/cosmincraciun97/stonewright-wp-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
