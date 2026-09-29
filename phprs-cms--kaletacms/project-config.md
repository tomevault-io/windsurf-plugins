---
trigger: always_on
description: Open-source CMS pro **firemní weby** (stránky, novinky/blog, později builder stránek, kolekce a formuláře) s napojením na
---

# Kaleta

Open-source CMS pro **firemní weby** (stránky, novinky/blog, později builder stránek, kolekce a formuláře) s napojením na
jazykové modely. Návrh, rozhodnutí a fáze: `../kaleta-interni/NAVRH.md`. Čisté PHP 8.4+ bez frameworku a bez Composeru
(vlastní PSR-4 autoloader v `system/bootstrap.php`), MySQL přes PDO, serverové HTML + trocha vanilla JS.

**Veřejně (README, texty, commity) se na projekt, ze kterého jádro vzniklo, neodkazuje.**

## Zásady

- **Jednoduchost nad abstrakcí.** Kód má přečíst i poučený laik. Žádné DI kontejnery, ORM, build kroky ani npm. Nová závislost = silný důvod.
- **Firemní web, ne magazín.** Žádné redakční workflow (korektura, zámky, předávka), rubriky, komentáře, čtenáři, předplatné, reklama,
  ani push – fáze 0 je odstranila, nevracej je (newsletter se vrátil v 1.5 záměrně jen jako jedna šablona podle design systému). Co firmy potřebují navíc (formuláře a poptávky, údaje o firmě, kolekce,
  builder), přibývá podle `NAVRH.md`.
- **Standardy webu 2026/2027 bez ohledu na staré prohlížeče:** CSS vrstvy, `clamp()`, container queries, `color-mix()`/OKLCH, `:has()`,
  Popover API, `<dialog>`, `<details>`, View Transitions. Interaktivita přednostně bez JavaScriptu. Žádné polyfilly, CDN ani cizí písma.
- **Co se nevypisuje, nemá styl ani skript.** Do `image/web.css` ani `style.css` šablony nepatří selektor, který nikde nevzniká; skript nesmí
  hledat `[data-…]` prvek, který nikde nevzniká (hlídá `tools/unit-tests.php`).
- **Tabulky** mají významové názvy (`ka_novinky`, `ka_kategorie`, `ka_uzivatele`, `ka_media`, `ka_nastaveni`…); v kódu vždy přes `{novinky}`.
  Starší názvy sloupců zůstaly: `idc` = novinka, `tema`/`idt` = kategorie, `ido` = médium, `idu` = uživatel.
- Identifikátory v kódu anglicky (od 1.4); komentáře v kódu anglicky, texty rozhraní anglicky se slovníky (`t()`). Česky zůstává
  datový model: sloupce databáze, klíče staveb (JSON) a design systému – na hranici MCP je překládá `Mcp\Translator` a `Mcp\Vocabulary`.
- **Změna databáze = dva zápisy:** úplné schéma `system/sql/schema.sql` a migrace `system/sql/migrace/NNNN-popis.sql` + zvýšit
  `KALETA_DB_VERSION` v `system/bootstrap.php` (hlídá `tools/test.sh`). Výchozí stav je migrace 0001.
- **Rozšíření jsou uzavřený systém** (`Core\Extensions::CATALOG`): žádné cizí plug-iny ani nahrávání kódu z administrace.
- **Role:** správce (2), editor (1 – veškerý obsah, vydává), autor novinek (0 – jen své novinky, nevydává). `Auth::canPublish()`,
  `Auth::managedAuthors()`, `Auth::articleScope()`; práva k sekcím navíc `ka_uzivatele_prava` (výchozí podle role, `Users::defaultModules()`).
  Vlastní role (`ka_role`, modul `Roles`): úroveň 0/1 + sada sekcí; uložení role přepíše `ka_uzivatele_prava` a úroveň
  všem členům (`ka_uzivatele.role`), `Auth` se tak nemění.
- **Rozšíření modulu a prvku:** `Module::EXTENSION` a `Element::EXTENSION` (novinky, poptavky, newsletter…). Vypnuté
  rozšíření: modul zmizí, prvek se nenabízí a na webu nevykreslí, sekce knihovny s ním se nenabízejí, trasy webu vrací 404.
  Nové rozšíření zapnuté ve výchozím stavu potřebuje migraci, která ho doplní webům s uloženým výběrem (viz 0016).
- **Nastavení:** nová volba = klíč v `Settings::DEFAULTS` + typ v `Settings::FIELDS` + řádek `$pole(...)` ve `views/admin/settings/<zalozka>.php`.
- **Nikdy `window.confirm()`** – v administraci atribut `data-potvrdit="text"`.
- **Administrace má CSP `script-src 'self'`:** žádné inline skripty ani `on*=` atributy; chování do `image/admin.js` přes `data-` atributy.
- **Prázdný výpis** v administraci přes `views/admin/empty.php`. Vzhled administrace je jediný (`image/admin.css`); změny kontroluj ve světlém
  i tmavém režimu a v šířce telefonu. Písmo administrace je Bricolage Grotesque (písmo značky, SIL OFL), hostované u sebe (`image/pisma/`).
- **Značka Kaleta** podle manuálu (logo manual v1.0): slovní značka „kaleta.“ malými písmeny se signální tečkou, ikona „k.“.
  Barvy: Ink `#121212`, Paper `#F6F4EE`, Signal `#FF4F2E` (jen tečka a drobné akcenty – nikdy malý text, na Paper má nízký kontrast).
  Logo se nikdy nepřepisuje písmem – vždy `views/admin/logo.php` nebo `image/kaleta-*`. Na weby uživatelů se nedává.

## Web (front)

- `Front\Kernel`: `/` = úvodní stránka (nastavení `home_page`, v jazykové verzi její protějšek `preklad_z`), bez ní výpis novinek;
  úvodní stránka na své vlastní adrese přesměruje 301 na `/`. Novinky na `/novinky`, `/novinky/<seo>` (+ `.md`), `/novinky/kategorie/<seo>`,
  `/novinky/stitek/<seo>`. Stránky na `/<seo>` – vyhrazené adresy `Modules\Pages::RESERVED_SLUGS`.
- **Themeless (od 1.6):** rámec stránky je `system/views/front/base.php` + `image/sablona.css`, pohledy `system/views/front/` nejdou přepsat
  a vlastní PHP layouty (`layout/`) se nepoužívají – Stav systému na zbylou složku upozorní. Vzhled = design system, sdílené třídy, komponenty
  a části webu. `base.php` vypisuje `<?= $hlava ?>` před `</head>` a `<?= $pata ?>` před `</body>` (SEO, strukturovaná data, měření, cookie lišta – `Front\Seo`).
  Barvy, písma, škálu a rozměry ber z tokenů design systému (`--ka-barva-*`, `--ka-krok-*`, `--ka-mezera-*`, `--ka-sirka`…) s vlastní výchozí hodnotou.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phprs-cms/kaletacms](https://github.com/phprs-cms/kaletacms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
