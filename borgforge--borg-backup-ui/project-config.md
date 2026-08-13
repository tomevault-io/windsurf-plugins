---
trigger: always_on
description: Dieses Repository ist vollstaendig eigenstaendig.
---

# AGENTS.md - borg-backup-ui (Unraid Plugin)

## Repository Scope

Dieses Repository ist vollstaendig eigenstaendig.

Alle fuer dieses Repository notwendigen Informationen befinden sich innerhalb dieses Repositories.

Treffe keine Annahmen ueber weitere Repositories, Services oder zukuenftige Architekturvarianten.

Arbeite ausschliesslich mit dem vorhandenen Code, der vorhandenen Dokumentation und den Anweisungen in diesem Repository.

---

## Zweck dieses Repositories

Dieses Repository ist die eigenstaendige Entwicklungs- und Produktionslinie fuer das Unraid-Plugin `borg-backup-ui`.

Das Plugin wird aktiv fuer Unraid weiterentwickelt und gepflegt.

---

## Rolle in der Architektur

Dieses Repository verantwortet:

* Unraid-Plugin-Funktionalitaet
* Unraid-spezifische Integration
* Benutzeroberflaeche
* Packaging
* Release-Erstellung
* Upgrade-Pfade fuer bestehende Nutzer

Neue Funktionen duerfen implementiert werden, sofern sie mit der bestehenden Architektur vereinbar sind.

---

## Erlaubte Aenderungen

* Bugfixes
* Security-Fixes
* Performance-Verbesserungen
* Verbesserte Fehlermeldungen
* Logging-Verbesserungen
* Wartungsarbeiten
* UI-Anpassungen
* Unraid-Integrationen
* Plugin-Manifest-Anpassungen
* Build- und Packaging-Anpassungen
* Neue Funktionen fuer das Unraid-Plugin

---

## Architekturregeln

Bewahre die bestehende Architektur.

Keine groesseren Refactorings ohne ausdrueckliche Freigabe.

Insbesondere keine:

* Framework-Wechsel
* FastAPI-Migration
* Datenbank-Migration
* API-Breaking-Changes
* Plugin-Layout-Aenderungen
* Grossflaechigen Verzeichnisumbauten
* Automatischen Modernisierungen

Wenn eine Architekturentscheidung unklar ist:

* Bestehendes Verhalten bevorzugen
* Rueckwaertskompatibilitaet bevorzugen (ggf. Nachfragen weil nicht jede Funktion Rueckwaertskompatibel sein muss)
* Nachfragen statt spekulieren

## Migrations- und Architekturwechsel-Regeln

Solange das Plugin noch nicht ueber Unraid Community Apps oeffentlich
veroeffentlicht ist, muessen freigegebene Architekturwechsel nicht dauerhaft
alte interne Implementierungen parallel mitschleppen.

Wenn der Repository-Maintainer einen Architekturwechsel ausdruecklich freigibt:

* Eine saubere neue Implementierung ist gegenueber dauerhaften Legacy- und
  Fallback-Pfaden zu bevorzugen.
* Bestehende Nutzerdaten, Jobs, Profile, Reports, Secrets und Statusdateien
  duerfen nicht stillschweigend verworfen werden.
* Statt permanenter Legacy-Kompatibilitaet ist eine einmalige, idempotente
  Migration vom aktuellen internen Stand auf das neue Modell vorzusehen.
* Jede Migration muss nachvollziehbar sein:
  * eindeutige Migrations-ID
  * Status `pending`, `applied`, `skipped`, `failed`, `blocked` oder `not_applicable`
  * Zeitstempel
  * betroffene Dateien/Objekte
  * konkrete Aktionen
  * Fehlerdetails ohne Secrets
* Migrationen sollen einen strukturierten Audit-Log schreiben, bevorzugt als
  JSONL oder als bestehendes Migrationslog-Format.
* Migrationen muessen getestet werden:
  * Ausgangszustand vor der Migration
  * erfolgreiche Anwendung
  * erneuter Lauf ohne doppelte Aenderungen
  * Fehlerfall, soweit sinnvoll simulierbar
  * ungueltige oder unvollstaendige Rueckgaben des Migrationscodes
  * keine Ausfuehrung nachfolgender Migrationen nach dem ersten Fehler
* Der zentrale Runner erwartet einen festen Vertrag:
  * `detect(config)` liefert ein Mapping mit dem booleschen Feld `required`.
  * `apply(config)` liefert ein Mapping mit einem unterstuetzten Status.
  * Fehler nennen Migrations-ID, Phase, maskierten Fehlertyp und maskierte
    Ursache.
* Nach dem ersten Fehler werden spaetere Migrationen als `blocked` erfasst und
  nicht mehr erkannt oder angewendet. Normalbetrieb, Scheduler und schreibende
  API-Aufrufe bleiben bis zu einem fehlerfreien Neustart gesperrt.
* Migrations-Snapshots sichern betroffene Nutzerdaten und Konfigurationen. Sie
  sind kein automatisches Downgrade des installierten Plugins. Unraid erlaubt
  keinen durch die Anwendung gesteuerten Rollback auf ein altes Plugin-Paket.
  Reparaturen erfolgen mit der installierten bzw. einer korrigierten Version;
  eine manuelle Wiederherstellung aus dem Snapshot muss nachvollziehbar
  dokumentiert werden.
* Wenn praktikabel, sollen Migrationsstatus und Logdetails in der UI sichtbar
  sein, bevorzugt unter `Einstellungen > Systemzustand & Migration`.
* Dauerhafte Fallbacks sind nur einzubauen, wenn sie fuer Datenerhalt,
  sichere Wiederherstellung oder eine explizit gewuenschte
  Rueckwaertskompatibilitaet notwendig sind.

---

## Sicherheitsregeln

Niemals:

* Zugangsdaten committen
* Secrets in Logs ausgeben
* Secrets in Changelogs ausgeben
* Secrets in Releases ausgeben
* Nutzerdaten ohne Freigabe loeschen
* Borg-Repositories ohne Freigabe loeschen
* Konfigurationsdateien automatisch ueberschreiben

Sicherheitsrelevante Aenderungen muessen minimal-invasiv erfolgen.

---

## Arbeitsverzeichnis

Arbeite ausschliesslich innerhalb:

git/borg-backup-ui

Verwende keine temporaeren Arbeitsverzeichnisse ausserhalb des Repositories.

Arbeite insbesondere nicht in:

* /tmp
* zufaelligen Clone-Verzeichnissen
* separaten Test-Repositories

---

## Git-Regeln

Keine destruktiven Git-Kommandos ohne ausdrueckliche Freigabe.

Insbesondere nicht:

* git reset --hard
* git checkout --

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [borgforge/borg-backup-ui](https://github.com/borgforge/borg-backup-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
