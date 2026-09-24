---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Multi-User Web-App zur Erfassung von Keuschheitsgürtel-Einschlusszeiten. Benutzer werden in der DB mit bcrypt-Passwort gespeichert. Admins verwalten Benutzer, setzen Trainingsvorgaben und sehen Statistiken. Jeder Benutzer kann Einschlüsse (VERSCHLUSS/OEFFNEN), Prüfungen (PRUEFUNG) und Orgasmen (ORGASMUS) mit Zeitstempel, Foto und Notiz erfassen.

## Commands

```bash
npm run dev       # Dev-Server starten (http://localhost:3000)
npm run build     # Produktions-Build
npm run start     # Produktions-Server starten

# Prisma
DATABASE_URL="file:./dev.db" npx prisma migrate dev --name <name>  # Migration erstellen
DATABASE_URL="file:./dev.db" npx prisma studio                     # DB-Browser öffnen
npx prisma generate                                                  # Client regenerieren

# Tests (Vitest unit)
npm test                                     # alle Tests (vitest run)
npm run test:watch                           # Watch-Mode
npx vitest run <pfad/zur/datei.test.ts>      # einzelne Datei
```

## Git-Absender — VOR dem ersten Commit prüfen

Jeder Commit in diesem Repo muss als `trublue-2 <info@trublue.ch>` entstehen, Autor **und** Committer. Die Konfiguration ist repo-lokal und wird **nicht** mitgeklont:

```bash
git config user.name  # muss: trublue-2
git config user.email # muss: info@trublue.ch
```

Stimmt etwas nicht, zuerst setzen:

```bash
git config user.name "trublue-2"
git config user.email "info@trublue.ch"
```

**Cloud- und Remote-Sessions sind der Regelfall für diesen Fehler.** Sie arbeiten in einem frischen Klon ohne lokale Config und committen dann als `Claude <noreply@anthropic.com>` — ohne Warnung, ohne dass es beim Review auffällt. Genau so entstand `docs(aufgaben): Übergabe-Papier für die Etappen 1-4` (16.08.2026); die Korrektur kostete einen `filter-branch` über 21 Commits und einen Force-Push auf einen bereits veröffentlichten Branch. In `main` stehen aus demselben Grund 26 Commits mit falschem Absender, die dort bleiben müssen — ein Rewrite der Hauptlinie träfe die ausgerollten Instanzen, den `release`-Tag und jeden Fork.

Dasselbe gilt für **Worktrees**: `git worktree add` erbt die Config des Repos, ein frisch geklontes Verzeichnis nicht.

**Warum das kein Formalismus ist:** Dieses Repo ist öffentlich, und der Absender darf niemals auf den System-Benutzer oder eine private Adresse zeigen. Die Prüfung gehört vor den ersten Commit, nicht danach — hinterher ist sie ein Rewrite.

### Pushen

Gepusht wird ausschliesslich über das Konto `trublue-2`. Auf trublues Maschine ist das verkabelt und nicht wählbar: die Remote-URL lautet `git@github-trublue:…`, ein SSH-Alias auf den Key dieses Kontos. Wer eine andere Remote-URL setzt, umgeht das — also keine setzen.

Dort greifen zusätzlich globale Hooks (`core.hooksPath`): `pre-commit` und `pre-push` erzwingen `info@trublue.ch` / `trublue-2` als Absender und weisen ausserdem **Inhalte** ab, die auf die Person hinter trublue zeigen — privater Benutzername und private Domains, in Dateitext wie in Dateipfaden. Die Muster stehen in `identity-rules.sh` und gehören bewusst NICHT in eine eingecheckte Datei. Die Zuordnung hängt an der Remote-URL, damit sie auch in einem Klon ohne lokale Config greift.

**In Cloud- und Remote-Umgebungen existieren diese Hooks NICHT.** Ein Hook wirkt nur dort, wo er installiert ist. Dort ist dieser Abschnitt die einzige Sperre — die Prüfung von Absender und Inhalt ist dann Handarbeit und muss vor dem Push passieren, nicht danach.

---

## Deployment

Drei Workflows, alle `workflow_dispatch` (kein Auto-Deploy bei Push):

- **`.github/workflows/docker.yml`** — baut das Image, pusht es nach GHCR, ruft danach den Deploy auf.
- **`.github/workflows/promote.yml`** — befördert ein **bestehendes** Image in einen Kanal (Retag über die Registry, **kein Rebuild**). Der einzige Weg zu `:latest`.
- **`.github/workflows/deploy.yml`** — das Deploy-Skript selbst (`workflow_call`), von beiden oben genutzt. Nicht direkt dispatchbar.

**Drei Ringe — `:feature` → `:portal` → `:latest`:**

| Tag | Für wen | Wann er wandert |
|-----|---------|-----------------|
| `:feature` | trublues Instanz **und mittestende Fremd-Instanzen**, Tests vor dem Merge | Feature-Branch-Build, oder `main`-Build mit `tagFeature=true` |
| `:portal` | die Portal-Instanzen | jeder `main`-Build |
| `:latest` | alle, inkl. Self-Hoster — der **offizielle Release** | nur durch `promote.yml` |
| `:v<version>`, `:sha-<sha>` | unveränderliche Referenz zum Pinnen, Promoten, Rollback | pro `main`-Build (`v…`) bzw. pro Build (`sha-…`) |

**Daneben: Seitenkanäle (`publishAs`).** Ein Build von einem Nicht-`main`-Zweig taggt normalerweise `:feature` — und trifft damit ALLE, die dort mittesten. Für Arbeit, die über längere Zeit getestet werden soll, ohne den Ring zu belegen, veröffentlicht `-f publishAs=<name>` **statt** `:feature` einen eigenen rollenden Kanal:

```bash
# Erstmalig: Kanal bauen UND die eigene Instanz darauf umpinnen.
# `pinnedTo` nennt den ALTEN Pin — ohne ihn schlägt der Lauf fehl, siehe unten.
gh workflow run docker.yml --ref <branch> -f publishAs=<kanal> \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trublue-2/chastitytracker](https://github.com/trublue-2/chastitytracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
