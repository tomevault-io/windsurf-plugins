---
trigger: always_on
description: Written for coding agents, and useful to anyone new to the repository. In
---

# Working on tun-manager

Written for coding agents, and useful to anyone new to the repository. In
English like the rest of it; conversations with the maintainer are in French.

## Testing

**Everything is tested.** A change arrives with the test that would have caught
its absence. Write the test first, watch it fail for the reason you expect, then
make it pass — a test that has never been red has not been shown to test
anything.

**When the call is known, stand in for it.** Everything this program cannot
make fail on demand — `os.Stat` on a file that was there a moment ago, a `chmod`
on a directory it has just created, `crypto/rand`, `os.Exit`, the uid behind a
`FileInfo`, `go list` — is reached through a variable, and a test swaps that
variable for one that fails. Those branches are the ones that keep a permission
check from reporting something false, so they are exactly the ones worth having
red once.

`internal/fsx` holds the filesystem calls, under the names the standard library
gives them, so a reader who knows `os.Stat` knows what `fsx.Stat` does. Anything
else is a variable beside the code that uses it, with a comment saying what a
test does with it:

```go
// generateSeed draws the key init-privileged writes. A variable so a test can
// make the draw fail: crypto/rand does not fail on darwin, and a key drawn from
// a source that ran out would have fewer bits than it claims.
var generateSeed = func() (string, error) { return feed.GenerateSeed(nil) }
```

The rule is not "mock everything". It is: when the thing being called has a
shape you already know — a function, a method, an interface — write that shape
down and let a test supply it. What is left after that is code with no seam
worth adding, and there is currently none of it: **the whole module is at 100%
of statements**, `internal/tools` included.

**Nothing is currently untested.** `docs/coverage-gaps.md` used to argue for two
dozen omissions and has been deleted: every claim of that kind in this
repository turned out to be wrong. The WireGuard control sockets, the host
interface listing, `osascript`, the composition root, a glob pattern "that can
only be a literal", `main` itself, and the twenty-odd filesystem races that were
argued to be out of reach — each was one line that did nothing but call out, and
one line can be stood in for.

If a real one ever turns up, it is explained in two places, both required:

1. A `NOT TESTED:` comment **on the code itself**, next to what is untested,
   saying why in a couple of lines and naming the section of
   `docs/coverage-gaps.md` that argues for it. Whoever reads that code sees the
   reason without going looking for it.
2. That file, brought back, with a `###` heading matching the name the comment
   refers to.

`make markers-check` fails when a marker names a section that does not exist,
and says so when there are no markers at all. It runs in `make all` and in CI.
What guards the reverse — code nobody has run — is `make cover`, whose floor is
100.

**Tests are hermetic.** No test may read anything the machine happens to have.
Configuration paths, binaries and run directories are pointed at `t.TempDir()`.
This is not theoretical: the `doctor` tests once read `/opt/homebrew/bin/wg-quick`,
passed locally and failed in CI.

**Fixtures are invented.** Never copy real configuration into `testdata`.
Addresses come from the ranges reserved for documentation — RFC 5737
(`192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`) and RFC 3849
(`2001:db8::/32`) — so no test can reach a real host. WireGuard keys are valid
base64 generated for the repository. Tunnel names are placeholders: `alpha`,
`bravo`, `charlie`.

**Coverage.** `make cover` prints the total and fails below the floor in the
Makefile, which is 100 and covers every package, `internal/tools` included. A
change that lowers it is a change that shipped a branch nobody has run.

## Before saying it is done

```sh
make all      # vet, lint, tests, notices, markers, build
make cover    # the total against the floor
```

Report what the commands printed. If something fails, say so with the output.

## Secrets

`PrivateKey` and `PresharedKey` are never parsed, stored or printed. The TUI log
pane redacts anything shaped like a WireGuard key. Keep it that way, and never
commit a real endpoint, key or tunnel name.

## The menu bar application

`macos/` holds a Swift client of the status feed. It is built with SwiftPM and
has no Xcode project, so every file in it reviews as text.

The testing rules above apply, with one carve-out and one addition:

- **The Go coverage floor does not apply to Swift.** They are separate
  contracts, and `make all` deliberately does not build or test `macos/` — the
  Go gate must not start requiring Xcode.
- **`Sources/TunManagerMenuBar/` is untested on purpose**, argued for in
  `macos/docs/coverage-gaps.md` the same way a `NOT TESTED:` marker is here.
  That is only defensible because every decision lives in `TunManagerFeed`,
  which does not link AppKit — so a decision cannot drift into an untested file
  without the compiler noticing. **An `if` in the AppKit layer is a bug of
  placement.**

Fixtures follow the same rules: invented keys, RFC 5737 addresses, tunnels
called `alpha`, `bravo`, `charlie`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nledez/tun-manager](https://github.com/nledez/tun-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
