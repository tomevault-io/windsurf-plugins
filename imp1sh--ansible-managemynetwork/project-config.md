---
trigger: always_on
description: This file is the canonical contributor guide for agents (human or AI) working on this
---

# AGENTS.md — ground rules for `imp1sh.ansible_managemynetwork`

This file is the canonical contributor guide for agents (human or AI) working on this
collection. It consolidates the conventions already stated in `README.md` with the
best practices proven by the most polished roles. **Not every existing role satisfies
every rule today.** Treat the rules below as the target state; when you touch a role,
leave it closer to compliance than you found it.

Don't blindly commit AI-generated content. Read it, understand it, verify it. If you
cannot stand behind a change, do not commit it.

## Scope and identity

- Namespace `imp1sh`, collection `ansible_managemynetwork`, abbreviated **MMN**.
- Primary targets: **Debian GNU/Linux** and **OpenWrt**. Other families (Alpine,
  FreeBSD, Arch, EL, Fedora) remain supported where a role already handles them, but
  new work should not broaden the matrix without reason.
- Requires Ansible **>= 2.11** (`meta/runtime.yml`). Per-role `meta/main.yml` must
  declare `min_ansible_version: "2.11"` to match — several roles still say `2.9`,
  fix them when touched.
- Release version stays below `1.x` until every shipped role meets this standard
  (see `README.md` "Maintenance Status"). Production-ready roles are enumerated in
  the README table; unlisted roles are candidates for uplift or removal.

## Reference roles

When in doubt, imitate these three roles — they are the de facto style guide:

- `roles/ansible_packages/`
- `roles/ansible_users/`
- `roles/ansible_groups/`

For OpenWrt roles, `roles/ansible_openwrtsystem/` is the clearest exemplar of the
imagebuilder-aware delegation pattern described below. Not for all roles it makes
sense to support imagebuilder. ansible_podman e.g. is a no no for imagebuilder it
is not possible to deploy container images onto a firmware image. Don't blindly add
imagebuilder support. First evaluate if imagebuilder support makes sense in the first
place.

## Paradigms (carry forward from README.md)

- **Separation of duty.** A role owns exactly one concern. Package installation is
  the job of `ansible_packages` / `ansible_openwrtpackages`; other roles feed it via
  `*_additional_packages` lists and `include_role`, they do not call `apt`/`opkg`
  themselves. Diverge only when there is genuinely no alternative.
- **Logic lives in roles, not playbooks.** Playbooks should set variables and invoke
  roles; nearly all programmatic behaviour belongs inside the role so callers only
  configure data.
- **Design for portability.** Differences between OSes belong in OS-specific
  variable/task files loaded dynamically — never in sprawling inline `when` chains.
- **Host- and group-scoped variables.** Where a role accepts per-host and per-group
  inputs, expose dedicated `<prefix>_*host` and `<prefix>_*group` keys (soft
  requirement; follow what the reference roles do).
- **Be opinionated only when the opinion is strong and validated**, never out of
  laziness.

## Required role structure

Every role must have, at minimum:

```
roles/<role>/
├── defaults/main.yml      # user-tunable knobs (lowest precedence)
├── meta/main.yml          # galaxy_info + accurate platforms + galaxy_tags
├── README.md              # role purpose, variables, usage, imagebuilder support
└── tasks/
    ├── main.yml           # dispatcher only — guards + include_tasks
    └── NN<name>.yml        # ordered, focused task files (01…, 02…)
```

Add `vars/`, `handlers/`, `templates/`, `files/` as needed.

Rules:

- `tasks/main.yml` is a **dispatcher**: guards, mode selection, and
  `include_tasks`/`include_role` calls. Put substantive work in numbered task files
  (`01live.yml`, `01imagebuilder.yml`, `02…`, `checks.yml`, …). Numbers sort the
  run; names describe the phase.
- Split **live mode** and **imagebuilder mode** into separate task files (see
  "OpenWrt imagebuilder contract").
- Use `vars/` for role-internal constants and OS-specific mappings. Load them
  dynamically with the `first_found` lookup — do not hardcode distribution branches
  in tasks:

  ```yaml
  - name: "MMN <role> - Setting OS variables"
    ansible.builtin.include_vars: "{{ lookup('ansible.builtin.first_found', params) }}"
    vars:
      params:
        files:
          - "{{ ansible_distribution }}.yml"
          - "{{ ansible_os_family }}.yml"
          - default.yml
        paths:
          - "{{ role_path }}/vars"
  ```

  Provide a `vars/default.yml` fallback so the lookup never fails silently.

- Handlers go in `handlers/main.yml`. Guard any handler that touches a live service
  with `when: not <prefix>_runimagebuilder | default(false) | bool` so image builds
  do not attempt to restart daemons that do not exist on the buildhost.

## Naming conventions

- **Variables** are namespaced with a stable domain prefix tied to the role, and
  that prefix is used consistently throughout the role. Existing schemes:
  | Role | Prefix |
  | ---- | ------ |
  | `ansible_packages` | `packages_` |
  | `ansible_users` | `system_users_` |
  | `ansible_groups` | `system_groups_` |
  | `ansible_openwrt*` | `openwrt_<section>_` (e.g. `openwrt_system_`, `openwrt_firewall_`) |
  | `ansible_postgresql` (illustrative) | `psql_` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imp1sh/ansible_managemynetwork](https://github.com/imp1sh/ansible_managemynetwork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
