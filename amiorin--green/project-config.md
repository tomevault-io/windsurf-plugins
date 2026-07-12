---
trigger: always_on
description: validates. See its `README.md` and `PLAN.md`.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`green` is a babashka-compatible Clojure library for building idempotent
devops CLIs: desired state in EDN, workflows as step graphs threaded by a
map, Selmer-scaffolded config files, OpenTofu as the muscle. The full
specification lives in `index.html` (open it in a browser).

## Commands

```sh
bb test             # run the test suite under babashka
clojure -X:test     # run the test suite under the JVM

clojure -T:build jar       # build target/green-<version>.jar + pom
clojure -T:build install   # install into the local ~/.m2
clojure -T:build deploy    # deploy to Clojars (needs CLOJARS_USERNAME/CLOJARS_PASSWORD)
```

Run a single test namespace under the JVM: `clojure -X:test :nses '[green.advice-test]'`.
Under babashka, `bb.edn`'s `test` task explicitly requires and runs the
namespaces named there — add new test namespaces to both `:requires` and the
`run-tests` call. Under the JVM, `clojure -X:test` uses `deps.edn`'s `:test`
paths with `cognitect.test-runner` discovery; there is no namespace list to
maintain in `deps.edn`.

Dependency rule: `green` has not been published to Clojars yet. External
consumer docs and launcher snippets must use a git dependency with an explicit
`:git/sha`; do not recommend `:mvn/version` until a Clojars release exists.
In-repo examples may keep `:local/root "../.."` for development.

Try the examples end-to-end:

```sh
cd examples/zookeeper
./green create --dry-run   # print what would run, touch nothing
./green create             # fake 3-node ZooKeeper cluster in ./work
./green delete

cd ../multi-zookeeper
./green create --dry-run   # inherited dry-run advice across wf/step
./green create             # two clusters, one composed workflow, in parallel
./green delete

cd ../once
./green create --dry-run   # print what would run, touch nothing
./green create             # fake ONCE-style VPS, DNS/SMTP, smtp-post, and Ansible scaffolds
./green delete

cd ../multi-once
./green create --dry-run   # two ONCE boxes from one once-wf; dry-run touches nothing
./green create             # NOTE: S3 backend is demonstration-only — needs a real bucket
./green delete

cd ../floci-zookeeper
./green create --dry-run   # print the run; touches nothing, needs nothing
./green create             # REAL 3-node ZooKeeper on floci (local AWS emulator)
./green delete             # ansible delete.yml first, then tofu destroys
```

`examples/multi-once` composes the single-VPS `once` workflow the way
`multi-zookeeper` composes the cluster workflow: `green.edn` holds a seq of
`:once/deployments`, and a parent workflow forks one `once-wf` run per
deployment via `wf/step`. It swaps two inherited advices on the embedded
steps under their original ids — `::provider` becomes data-driven
(`:eu` → digitalocean, `:us` → oci, read from a per-deployment
`:once/provider`) and `::backend` becomes S3, isolated by a per-deployment
+ per-step `:key` (`once/<deployment>/<step>/terraform.tfstate`) so no two of
the `2×4` tofu states collide. The parent declares `dry-run/advise` **last**,
so `::skip` outranks the re-added `:before` provider/backend advice and
`--dry-run` skips them too (touches nothing). The S3 backend is
demonstration-only: `create` needs a real bucket, so the offline path is
`--dry-run`.

Each example's `./green` script is a self-contained babashka script that
pulls in `green` via `:local/root "../.."` — no separate build step needed
to try changes made to `src/`. `examples/once/SPEC.md` documents the ONCE-style
example: provider-swap advice for compute, a
`compute ∥ smtp → dns → smtp-post → (ansible-local ∥ ansible-remote)`
fork/join, threaded opts, per-step tofu state, and scaffold-only Ansible config.

The end-to-end ZooKeeper suite (`test/green/zookeeper_test.clj`) drives real
`tofu` over HCL containing only `locals`/`output` blocks — full
render/apply/destroy cycles, zero real infrastructure — and skips when `tofu`
is not on `PATH`. `test/green/tofu_test.clj` covers backend advice without
invoking `tofu`; `test/green/ansible_test.clj` covers playbook selection,
PLAY RECAP parsing, inventory rendering, and inventory advice without invoking
`ansible-playbook`.

`examples/floci-zookeeper` is the one example that builds something real:
OpenTofu's AWS provider pointed at floci (a local AWS emulator on
`localhost:4566`, Docker-backed EC2) creates three instances, and
`green.ansible` provisions an actual ZooKeeper ensemble over SSH
(`create.yml`/`delete-node.yml`, no user-data except a 3-line sshd bootstrap
compensating for a floci bug). Create uses two fan-out/join cycles: per-node
tofu apply (3 parallel) → provision join (collects IPs) → per-node ansible
(3 parallel, each with a single-host inventory and the full ensemble as
extra-vars for `zoo.cfg`) → health join (quorum check). Delete fans out 3
`wf/step` sub-workflows, each running ansible-stop → tofu-destroy for one
node independently — `wf/step` is needed because bare `:zk/ansible →
:zk/node` branches would join at `:zk/node` (different parents). The
parent's advice on `:zk/ansible` and `:zk/node` (backends, inventory, SSH
wait, dry-run) is inherited into each sub-workflow by step name. It also

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amiorin/green](https://github.com/amiorin/green) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
