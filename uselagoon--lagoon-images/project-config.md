---
trigger: always_on
description: > **Architecture, naming, dependency model, build system, CI flow:**
---

# Lagoon Base Images — AI Agent Instructions

> **Architecture, naming, dependency model, build system, CI flow:**
> see [architecture.md](../architecture.md). This file is intentionally
> task-focused — it tells you **what to do**, not **why the repo is
> shaped the way it is**.

---

## When reviewing a PR

### Instruction source for CI Copilot reviews

For guidance that must be enforced on every automated PR review, place it in
`.github/instructions/*.instructions.md`.

Keep this file as task-focused guidance and workflow recipes. Linked docs
such as `architecture.md` and `docs/reviewing/PR_REVIEW_RUBRIC.md` are
reference material. Use `.github/instructions/pr-review.instructions.md` as
the canonical machine-consumed PR review policy.

Use [`.github/instructions/pr-review.instructions.md`](instructions/pr-review.instructions.md)
for enforceable review checks, and
[docs/reviewing/PR_REVIEW_RUBRIC.md](../docs/reviewing/PR_REVIEW_RUBRIC.md)
for explanatory context.
Focus on:

1. **Correctness** — reproducible builds, explicit version pins.
2. **Security** — no unverified external downloads, minimal attack surface.
3. **Testing** — every new image / variant has compose + TESTING markdown coverage.
4. **Clarity** — labels, comments, intent.
5. **Performance** — minimal final image size.
6. **Consistency** — naming, Dockerfile conventions, group/target wiring match the rest of the repo.
7. **Backward compatibility** — no silent breaking changes.
8. **Maintenance** — upstream version is current; deprecation labels present where applicable.

Group findings by category. Skip nits already covered by linters.

---

## Recipe 1 — Add a new version of an existing service

Example: adding `python-3.15`.

### 1. Create the Dockerfile

`images/python/3.15.Dockerfile`, following the **first-tier pattern**
documented in [architecture.md §7.1](../architecture.md#71-first-tier-images-from-upstream).

### 2. Update `docker-bake.hcl` — three edits, all required

```hcl
# (a) New target
target "python-3-15" {
  inherits   = ["default"]
  context    = "images/python"
  dockerfile = "3.15.Dockerfile"
  tags       = tags("python-3.15")
  contexts   = {
    "${LOCAL_REPO}/commons": "target:commons"
  }
}

# (b) Add to the service-specific group
group "python" {
  targets = [
    "commons",
    # … existing versions …
    "python-3-15",
  ]
}

# (c) Add to the default group
group "default" {
  targets = [
    # … existing targets …
    "python-3-15",
    # … more targets …
  ]
}
```

> **Forgetting (b) or (c) means the image silently won't be built by CI.**

Target names use dashes (`python-3-15`). Tags keep dots (`python-3.15`).

### 3. Add test coverage

- For a runtime/HTTP service: add a service block to
  [helpers/images-docker-compose.yml](../helpers/images-docker-compose.yml)
  and test commands to
  [helpers/TESTING_base_images_dockercompose.md](../helpers/TESTING_base_images_dockercompose.md).
- For a stateful service: same pattern in
  [helpers/services-docker-compose.yml](../helpers/services-docker-compose.yml)
  and [helpers/TESTING_service_images_dockercompose.md](../helpers/TESTING_service_images_dockercompose.md).

### 4. Validate locally — non-negotiable

```bash
make build/python-3.15

cp helpers/images-docker-compose.yml docker-compose.yml
sed -i '' 's/uselagoon/lagoon/g' docker-compose.yml   # macOS sed
docker compose up -d python-3-15 commons

# Run EVERY new TESTING_*.md command for this image and confirm output
docker compose exec -T python-3-15 sh -c "python --version" | grep "3.15"
# … etc …

docker ps --filter label=com.docker.compose.project=lagoon-images | grep Up | grep python-3-15

docker compose down && rm docker-compose.yml
```

**Never copy version strings or grep patterns from another service
without verifying against the actually-built image** — Redis, MariaDB
and PHP have all caused CI failures this way.

---

## Recipe 2 — Add a new variant of an existing service

Example: adding `service-X-persistent-drupal` for a service that already
has `service-X` and `service-X-drupal`.

### 1. Create the Dockerfile

`images/service-persistent-drupal/X.Dockerfile`, using the **variant
pattern** from [architecture.md §7.2](../architecture.md#72-variants--specializations-from-a-lagoon-parent).
A combined specialization (`-persistent-drupal`) layers on the *first*
specialization (`-drupal`), not on `-persistent`.

### 2. Wire it in `docker-bake.hcl`

```hcl
target "service-X-persistent-drupal" {
  inherits   = ["default"]
  context    = "images/service-persistent-drupal"
  dockerfile = "X.Dockerfile"
  tags       = tags("service-X-persistent-drupal")
  contexts   = {
    "${LOCAL_REPO}/service-drupal": "target:service-X-drupal"
  }
}
```

Variant targets **only reference their direct parent**, not commons.
**Exception:** if the Dockerfile has its own `FROM ${LOCAL_REPO}/commons AS commons`
build stage (e.g. `solr-9-drupal` does this to use git/curl from
commons), it must also list commons in `contexts`. See
[architecture.md §7.3](../architecture.md#73-the-commons-as-build-stage-exception).

Then add the target to the service group **and** the `default` group.

### 3. Test + validate as in Recipe 1, step 4.

For variants, the normal expectation is **targeted validation** rather

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uselagoon/lagoon-images](https://github.com/uselagoon/lagoon-images) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
