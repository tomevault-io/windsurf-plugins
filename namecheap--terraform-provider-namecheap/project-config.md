---
trigger: always_on
description: Run these after making changes:
---

# CLAUDE.md

## Verification Steps

Run these after making changes:

```bash
make build          # Build binary
make format         # go fmt ./...
make check          # go vet ./...
make lint           # golangci-lint run
make test           # Unit tests with coverage
make testacc-mock   # Mock-backed acceptance tests (no credentials; needs a terraform/tofu binary)
```

Run a single test: `go test -v ./namecheap/... -run TestFunctionName -count=1`

`make testacc-mock` runs the `testacc`-tagged mock acceptance suite; the mock
lives in `namecheap/mock_server_test.go` and is exercised on every PR (see
`CONTRIBUTING.md`). `make testacc` remains the live-API sandbox suite (needs
`NAMECHEAP_*` credentials + a whitelisted IP).

## DCO Sign-off

All git commits must include a `Signed-off-by` line for the [Developer Certificate of Origin](https://developercertificate.org/) (DCO) check to pass. The DCO bot on GitHub will block PRs that contain unsigned commits.

- Use `git commit --signoff` (or `-s`) to add the sign-off automatically.
- To sign off an entire branch retroactively: `git rebase HEAD~N --signoff` (replace N with the number of commits).
- The `Signed-off-by` identity must match the commit's author or committer name and email.

## Git privacy

Before creating git commits, check that `git config user.email` is set. If it is not configured, suggest the contributor set one. Do not override an already-configured email.

## Commit Messages

- Do **not** add `Co-Authored-By` trailers to commits. Keep authorship clean. (The DCO `Signed-off-by` line is still required — see DCO Sign-off above.)
- Use [Conventional Commits](https://www.conventionalcommits.org/) for PR titles and commit subjects: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`, `ci:`, `perf:`, `build:`, `revert:`. The release automation ([release-please](https://github.com/googleapis/release-please)) parses these to compute version bumps and the changelog: `fix:` → patch, `feat:` → minor, `feat!:`/`BREAKING CHANGE:` → major. A non-conventional title leaves a change out of the release notes. `pr-title.yml` enforces this on PRs.

### Releasing vs non-releasing changes

release-please bumps the version only on `fix:` → patch, `feat:` → minor, `feat!:`/`BREAKING CHANGE:` → major. `chore:`/`ci:`/`refactor:`/`test:`/`docs:` are **non-releasing** — at most they appear in the changelog when some *other* commit cuts a release. Two **published artifacts** make this easy to get wrong:

- **Runtime dependencies** (anything in `go.mod` — the `gomod` ecosystem) ship inside the provider binary, so they **must** land as `fix(deps):` (or `feat(deps):` if the bump exposes new provider behavior). `chore(deps):` here is a bug: the update ships to nobody until an unrelated `fix:`/`feat:` sweeps it into the next Release PR.
- **Registry-published docs** — the `docs/` tree (tfplugindocs output) is ingested by the Terraform Registry *only at a release tag*, so a correction there must ride a release: commit it as `fix(docs):`. Repo-internal markdown (README, `RELEASE.md`, `CLAUDE.md`, code comments) publishes nowhere and stays `docs:` — non-releasing.
- **CI/tooling only** (GitHub Actions, scanners, runners — the `github-actions` ecosystem) never touch either published artifact, so they use a non-releasing type: `ci(deps):` (Dependabot) or `ci:` (manual).

Rule of thumb: **if a change lands in something published (the binary or the `docs/` tree), it needs a releasing type (`fix:`/`feat:`); if it's repo-internal, it doesn't.** For Dependabot this is enforced via `commit-message.prefix` in `.github/dependabot.yml` (gomod → `fix`, github-actions → `ci`).

## Releases

Semi-automatic, maintainer-gated via release-please. Merging conventional-commit PRs to `master` updates a long-lived "Release PR"; merging that Release PR tags `vX.Y.Z` and triggers GoReleaser. Full flow and required secrets are documented in `RELEASE.md`. Don't hand-cut tags except for the emergency path described there.

## Documentation Layout

The `docs/` directory (`index.md`, `guides/`, `resources/`) is reserved for Terraform Registry provider documentation (tfplugindocs output) — it is published to the registry. Do **not** put internal design specs or process docs there; those belong in root-level markdown files (e.g. `RELEASE.md`), mirroring the sibling `terraform-provider-spaceship` repo.

## Pull Requests

- All CI checks must pass before merge (unit tests, acceptance tests, CodeQL, DCO).
- PRs should include both unit tests and Terraform acceptance tests where applicable.
- Acceptance tests use `resource.Test()` with `TestStep` — see `namecheap/provider_test.go` for examples.
- `SECURITY_COMPLIANCE.md` is the authoritative reference for the compliance gates a PR is judged against (dependency drift, vulnerability/license scans, SBOM, supply-chain pinning). Check it before proposing anything that touches `go.mod`, CI workflows, or action pins.

### Dependabot PRs

Dependabot commit/PR-title prefixes are pinned in `.github/dependabot.yml` so `go.mod` bumps land as releasing `fix(deps):` and action bumps as non-releasing `ci(deps):` — see *Commit Messages > Dependency bumps must use a releasing type* above for the rationale.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [namecheap/terraform-provider-namecheap](https://github.com/namecheap/terraform-provider-namecheap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
