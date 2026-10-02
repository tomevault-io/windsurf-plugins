---
trigger: always_on
description: Every file under `assets/tools/` is a vendored third-party binary that runs in
---

# Client binaries — rules for changing this tree

Every file under `assets/tools/` is a vendored third-party binary that runs in
production. ADR-0005 explains why they are committed instead of installed at
image build time. Two rules follow from that and apply to every change here:
the bytes in the repository are the bytes that ship, so a binary must be
verified in the runtime environment rather than on a developer machine, and a
directory name is a label chosen by us, which the startup check holds you to
but which nothing enforces while you are assembling the bundle.

The backend resolves a binary as
`assets/tools/<arch>/<engine>/<engine>-<bundle>/bin/<command>`, where `<arch>`
is `x64` for `linux/amd64` and `arm` for `linux/arm64`. The mapping lives in
`backend/internal/util/tools/paths.go`.

## Before adding a version

Adding an engine version is mostly about the binary, not about the code that
reads the version. Work through this list before committing anything; the
first two items often end the job early.

- [ ] **A new upstream release line needs a client here before it is
      supported.** Until the bundle lands, a server on that line is refused at
      the connection test, by design: we never dump a server with a client from
      an older line. Both engines open a line roughly once a quarter, so this
      is the recurring task that keeps that promise.
- [ ] **Is this a new line, or a new release inside a line we already serve?**
      Only a new line needs a bundle. A newer release inside a served line —
      MariaDB 13.2 against our 13.0 client, MySQL 26.10 against our 26.7 one —
      keeps working, and the bundle is refreshed when there is a reason to.
      MariaDB's modern client additionally serves every line from 10.2 up by
      designation, which is recorded in `README.md` in this directory.
- [ ] **Is the release line one upstream still patches?** Short-term and
      innovation releases live about a quarter. Prefer the long-term release of
      that era; bundle a short-term release only when users run it. When a line
      we bundle later gets a long-term release, refresh the bundle to it rather
      than adding an identity.
- [ ] **Do binaries exist for both architectures?** Check, in order: the
      vendor's architecture-independent tarball, the vendor's Debian
      repository, the vendor's Ubuntu repository, then RPMs for an older
      enterprise distribution. Do not conclude a build is unavailable after
      checking one of them.
- [ ] **Will the binary start in the runtime image?** The image is
      `debian:bookworm-slim` with glibc 2.36. Compare with
      `objdump -T <binary> | grep -oE 'GLIBC_2\.[0-9]+' | sort -uV | tail -1`
      and list the shared libraries with `objdump -p <binary> | grep NEEDED`.
      Every library must already be in the image; see the table in
      `README.md` in this directory. A matching soname is not enough: check
      the OpenSSL symbol versions the same way with `OPENSSL_3\.[0-9]+`,
      because bookworm ships OpenSSL 3.0 and a newer requirement fails only at
      run time. A development host with a newer OpenSSL hides the problem, so
      run the binary in the runtime image as the refresh checklist shows.
- [ ] **Know which of the two identity shapes the engine uses.** MySQL's
      identity doubles as the name of its bundle directory, because each MySQL
      line gets a client of its own. MariaDB keeps two values, a server
      identity and a client version, because one client serves many server
      lines. Follow the engine's existing shape rather than introducing the
      other one, and never record a version the server did not report.
- [ ] **Add the bundle and the identity together.** The new directory goes into
      the bundle set the startup check walks, in
      `backend/internal/util/tools/mysql.go` or `mariadb.go`, and the identity
      it serves is mapped to it. Detection refuses any version with no bundle,
      so these two land in one change or the version stays unsupported.
- [ ] **Check both architectures before claiming the version is supported.**
      A version whose client exists for only one architecture is supported only
      there, and is refused on the other. `assets/tools/arm/mysql/` has no 5.7
      bundle for that reason.
- [ ] **Do not look for version tables to extend; there are none left.** The
      restore downgrade order is computed from the numbers in the identity, the
      MySQL compression choice is derived from the identity rather than matched
      against a list of known ones, and the interface renders the identity
      string itself. All three used to be lookup tables whose missing entry
      changed behavior quietly. If you find yourself adding a version to a
      table, the table is the bug.
- [ ] **An identity names a line, so its digits are not a promise about the
      binary.** Identity `26` covers every MySQL 26.x and `13.0` every MariaDB
      13.x, while the bundle holds one patch. That is intended. What is not
      intended is a bundle holding a version from a *different* line, which the
      startup check now rejects.
- [ ] **Add the identity constant on both sides.** The backend constant and the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [databasus/databasus](https://github.com/databasus/databasus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
