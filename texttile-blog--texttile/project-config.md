---
trigger: always_on
description: Texttile is a blog CMS for two people who write together. Phoenix, LiveView and
---

# Texttile

Texttile is a blog CMS for two people who write together. Phoenix, LiveView and
SQLite in one Docker image. One volume holds the whole installation: the
database and every uploaded file. Nothing else is needed to run it.

You can think of Texttile as a self-hosted alternative to WordPress, Ghost and
Substack, in the spirit of "one container, your data".

## What makes Texttile special?

These are the promises the product is built on. Keep them intact.

### 1. Nothing is loaded from outside

A reader's browser talks to this server and to nothing else. No CDN, no
third-party script, no tracker, no captcha, no hosted font, no external video
player. The statistics are counted here, the spam filter runs here, the mail
leaves from here. A change that adds an outside call breaks the product, not
just a rule.

### 2. Light enough for a slow line

Texttile is written for a person reading on a phone far from a data center.
Pages stay small, JavaScript stays little, and a picture is only ever as large
as the screen asks for. Weigh every kilobyte you add. Continuously repainting
animations are out; they cost the reader battery and the writer frames.

### 3. Written together

Two people work on the same entry at the same time. Only the body text is
locked, and only softly: the holder writes, the other one watches the text
arrive live and can take over in one click. Tiles, tags, settings and publish
controls stay free for both, last write wins per field. The admin area shows
who is on which screen. This is the hero feature, and it is why the lock is a
GenServer and not a database column.

### 4. The writer's Markdown, byte for byte

The body is plain text, not a document tree. What the writer typed is what is
stored, character for character, so a version diff shows real edits and
nothing else. Never normalize, reflow or clean Markdown, and never introduce an
editor that serializes it from a tree (ProseMirror, TipTap, Milkdown).

### 5. Minimal, and everybody is an admin

A part is right when nothing is left to take away. There are no roles and no
permission matrix: the accounts are the whole access model, every one of them
an admin, and it is built for people who trust each other. `ADMIN_USERS` is not
that model any more. It says which addresses get an account at the start of the
server, so the first admin comes in without anybody to invite them, and
Settings does the same without a deploy. Both ways end in a mailed link, so who
you are is who reads that inbox.

Taking access away means deleting the account, and the account keeps its row:
what a person wrote carries their name, so a reader never sees an entry lose its
byline and the admin area still says who was there, marked as gone. The address
is free again at once, the variable included: an address that stands in
`ADMIN_USERS` is invited again at the next start, so revoking somebody means
deleting the account **and** taking the address out of the variable.

Settings have no Save button. The one exception is a field that owns the
account: moving your address asks for your password first. When a feature needs
a configuration switch to be bearable, the feature is wrong.

### 6. Mobile first

Reading and writing are laid out for a phone. A wide screen gets the same
screens with more room, never a different product. Judge a UI change on the
phone screenshot first.

## A note from Klaus

I like ambitious ideas, simple systems, and software that feels obvious. Do not
keep complexity because it is already there. Do not add machinery because it
looks architecturally impressive. Understand the real constraint, then fight for
the smallest model that makes the correct behavior unsurprising.

Channel both "measure twice, cut once" and YAGNI. Fight scope creep. Propose a
bold idea when it truly helps, and say so plainly instead of building it behind
my back.

A question is read-only. If I ask how hard something is, why something happens,
or whether something should be done, answer it and offer the change. Do not
start editing.

Match the ceremony to the task. One agent in one pass beats a panel for
ordinary work.

The rest of this file helps you find your way and make changes well. Treat it as
good defaults, not as scripture. My preferences in the moment beat anything
written here. If a rule fights the task in front of you, say so and get my
sign-off before breaking it.

## A small glossary

Use this language in code, in the UI and when talking to me.

- **you** means the agent reading this file and changing Texttile.
- **we, us, maintainers** mean Klaus and the people building Texttile.
- **admin** means a person with an account here who signs in and writes. The
  account is an email address and a password; there is no username.
- **invitation** means the mailed link that gives an account its first
  password. `ADMIN_USERS` sends one at the start of the server, Settings sends
  one on a click, and the link is the same one a forgotten password gets.
- **deleted account** means a row with a `deleted_at`: out of every list of
  accounts, out of every browser, its address free, and still the name under
  everything it wrote. `Accounts.here/0` is the query that leaves it out.
- **reader** means anybody who is not signed in.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [texttile-blog/texttile](https://github.com/texttile-blog/texttile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
