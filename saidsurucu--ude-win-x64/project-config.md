---
trigger: always_on
description: Bu depo, resmî **UYAP Doküman Editörü**'nü (UDE) Windows x64'te modern + keskin çalıştırmak
---

# CLAUDE.md — ude-win-x64 mühendislik notları

Bu depo, resmî **UYAP Doküman Editörü**'nü (UDE) Windows x64'te modern + keskin çalıştırmak
için **build-zamanı bytecode yaması** uygular. Kaynak kod içermez; resmî `editor-app.jar`
build sırasında uyap.gov.tr'den indirilir, yamalanır ve gömülü Java 11 ile `.exe`'ye paketlenir.

Bu, [`ude-mac-arm64`](https://github.com/saidsurucu/ude-mac-arm64) portunun Windows'a aktarımıdır.
Hedef: **Mac yaması ile birebir feature parity** (Mac-spesifik girdi/donanım/pencere-kromu hariç).

## Build hattı

```powershell
.\build.ps1                 # tam yapı (TÜM özellikler varsayılan ACIK) -> dist\*.exe
.\build.ps1 -Only patch     # sadece jar'ı yamala (build\input\editor-app.jar)
.\build.ps1 -Only download  # sadece resmî paketi indir
.\build.ps1 -NoSkin -NoIcons  # bir özelliği kapat
```

Fazlar (`scripts/build.ps1` orkestrasyonu): **deps** (JDK 11+17 + WiX) → **download**
(resmî zip → editor-app.jar) → **patch** (`scripts/patch.ps1`) → **package**
(`jlink` minimal runtime + `jpackage --type exe --input build\input`).

`scripts/common.ps1` ortak yardımcıları/yolları tanımlar (`$InputDir`, `$MainJar`,
`$BuildDir`, `$VendorDir`, `Get-Jdk11Home`, `Write-Ok`, `New-Dir`).

## Apply deseni (her özellik)

Obfuscated jar'a yama şu sırayla yapılır (apply-*.ps1 içinde):
1. Yardımcı sınıfları (`macosX/*.java` veya `com/udewin/X/*.java`) **jar'a karşı** derle.
2. `jar uf $jar -C <helper> .` ile yardımcıları jar'a enjekte et (patcher onları classpath'ten çözer).
3. `*Patch.java` (Javassist) patcher'ı derle (cp: javassist + helper).
4. Patcher'ı çalıştır: `java ... XPatch <jar> <out-dir>` → yamalı `.class`'ları out-dir'e yazar.
5. `jar uf $jar -C <out-dir> .` ile yamalı sınıfları jar'a geri yaz.

Javassist `3.30.2-GA` `$VendorDir\lib`'e indirilir. Bayraklar `scripts/patch.ps1` içinde
bir `foreach` döngüsü (default-on, `$env:X -ne '0'`) veya ayrı bloklarla yönetilir.

## Özellikler (hepsi varsayılan ACIK; `$env:X=0` / `-NoX` kapatır)

| Bayrak | Ne | Obfuscated hedefler |
|---|---|---|
| ICONS | Fluent ikonlar + HiDPI (1x/@1.5x/@2x multi-res) | `Utils.b`, `Utils.a`, Flamingo `ImageWrapperIcon`/`FilteredResizableIcon.paintIcon` |
| NATIVE_DIALOGS | Win32 Aç/Kaydet (`java.awt.FileDialog`) | `tr/*` `show*` çağrıları; matcher = **ad+imza+declaring-class JFileChooser alt-tip** (sınıf-bağımsız). UDE diyalogları `gui.dp`→`gui.a.p`→`JFileChooser` üzerinden; tarama "JFileChooser literal" şartı YOK (yoksa `fm/iI/nn/op` Aç çağıranları atlanırdı) |
| **FILEASSOC** | .udf çift-tık açma | `WPAppManager.main`'e `$1=ArgFix.normalize($1)` inject; kontrol-jetonu yoksa (yalnız dosya yolu) başa `getNewWPInstance` ekle (yoksa `a(String[])` erken `return` ediyor) |
| LIVETOGGLE | Otomatik-düzeltme anında etkin | `...pki.b.l` reflection |
| TABLEDELETE | Backspace/Delete tablo sil | `WPAppManager.main` inject; `DocumentEx` **void f(int)** overload |
| IMGFULL | Satır-içi imaj tam-çöz | `editor.utils.h`, `swing.wp.b.at.drawImage` (bicubic) |
| IMGRESIZE | Fare köşe-tutamağıyla boyutlandırma | `text.hj` paint/mouse + `macosimgresize.ImageResizeController` |
| ANTET | "Antetlerim" (`%APPDATA%\UDE\Antetler`) | `gui.gR.c()` → `AntetUI.install` |
| PDFFRESH | "PDF Kaydet" canlı serialize | `text.J.b(File)`: `lo.a(out)` → `lo.a(out,true)` |
| PASTEIMG | Panodan imaj kalitesi | `text.hj.paste`: `aa.a` yıkıcı küçültme atla + `Conv` |
| **FOPFONTS** | PDF'te Türkçe harf (ğĞşŞıİ) | `editor.b.a` (FopFactory.newInstance→FopFonts.apply), `b.c` (awtToPdf→ITextFonts.map), `b.b` (getPageFormat→PageFix.a4) |
| **CARETFIX** | Zoom imleç/tıklama hizası (sadece Faz-2) | `wp.prof.d.O` modelToView/viewToModel + `wp.textUtils.p.c` |
| **ZOOMKEYS** | Ctrl+/Ctrl− klavye zoom | `WPAppManager.main` inject; zoom JSlider sür |
| PASTERICH+PLAINPASTE | Stilli/formatsız yapıştırma | `text.hj` paste yolu; CF_HTML `DataFlavor.allHtmlFlavor` |
| SKIN | Düz Substance + Word widgets + açık/koyu + canlı geçiş | `SkinPatch` (~750 satır); `FlatUdeSkin`/`Dark`, `Word*`, `DarkMode`, `ModeSwitch`; **winlook.jar** -javaagent (renk-modu picker + canlı geçiş) |

Obfuscated adlar platformlar arası **AYNI** (aynı UDE build'i) → Mac patcher'ları çoğunlukla
verbatim çalışır. Hedefleri yamadan önce `javap`/`jar tf` ile DOĞRULA.

## Windows uyarlamaları (Mac'ten farklar)

- **Koyu mod tespiti**: kayıt defteri `HKCU\...\Themes\Personalize\AppsUseLightTheme`
  (Mac `defaults read` yerine). `macosskin/DarkMode.java`.
- **Pano HTML**: `DataFlavor.allHtmlFlavor` (Windows CF_HTML soyutlar; Mac NSPasteboard yerine).
- **Yollar**: `%APPDATA%\UDE\Antetler`, log `%LOCALAPPDATA%` (Mac `~/Library/...` yerine).
- **Font**: arayüz `Segoe UI` (Mac `Helvetica Neue`); FOP/iText için **Liberation** gömülü
  (Mac sistem Arial/Times yerine — Windows'ta da sistem fontuna güvenmiyoruz).
- **FOP file URL**: `file:/C:/...` ileri-bölü, **%-kodlama YOK** (FOP 0.92 %20'yi çözmüyor).
- **winlook.jar agent** (Mac MacLook'un eşi): `WinLook.java`; sadece platform-nötr metotlar
  (fixRulerBackground/boldTaskTabs/removeScopeCombo/addDarkPageToggle/addColorModeCombo).
  Mac chrome metotları (unifyTitleBar/hookTitle/removeMemoryBar) ATLANDI.

## Mac-spesifik ATLANANLAR (port edilmedi; gerekçeli)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saidsurucu/ude-win-x64](https://github.com/saidsurucu/ude-win-x64) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
