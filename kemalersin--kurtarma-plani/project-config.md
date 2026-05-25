---
trigger: always_on
description: Ant Design Vue UI ve UX kalıpları
---


# UI Kalıpları (Ant Design Vue)

## Kit ve stil

- `ant-design-vue` v4 + `@ant-design/icons-vue` (on-demand import)
- **Tailwind yok.** Stil: AntDV token + `<style scoped>` + minimal global `app.css`
- **Sayfa genişliği (iki mod):**
  - **Dar (varsayılan):** `.kp-page` → `--kp-page-max-width: 800px` (`app.css`). Panel, ayarlar, formlar, kurulum/seçim (`.kp-card` 520px).
  - **Geniş (liste):** route `meta.pageLayout: 'wide'` → AppShell `.kp-page--wide` (`max-width: none`). Birincil içerik `<EntityListPage>` veya `<a-table>` olan sayfalarda **zorunlu**; yeni liste route'larında meta'yı unutma.
  - Sayfa bazında ekstra `max-width` koyma; genişlik yalnızca route meta ile
- Tema: `theme.defaultAlgorithm` / `theme.darkAlgorithm`; sistem + manuel toggle
- Lokalizasyon: `<a-config-provider :locale="trTR">`

## İkonlar (emoji yasak)

- AntDV ikonları (`@ant-design/icons-vue`) birinci tercih
- AntDV'de yoksa `src/components/icons/<Name>Icon.vue` SVG bileşeni
- `currentColor` + `1em` boyut; aria-label ile erişilebilirlik
- **Tab başlıklarında ikon kullanma.** Sadece düz metin (`tab="..."`)

## AppShell

- `<a-layout-sider :collapsible="true">` + manuel pin (localStorage)
- Üst navbar `<a-input-search>` global arama (banka/hesap/borç/gelir/gider)
- İçerik alanı `<a-layout-content>`; mobile-first responsive

## Liste

- **Genişlik:** route `meta.pageLayout: 'wide'`; paylaşılan liste bileşeni `EntityListPage`
- **Sekmeli liste araç çubuğu:** sekme başlığı yeterli; `EntityListPage` içinde **h2/liste başlığı yok**. Masaüstü tek satır: **arama + filtre ikonu (sol)** · **arşiv segmenti araç çubuğunun tam ortasında** (`position: absolute; left: 50%`) · **yeni kayıt (sağ)**. Mobil (`≤640px`): **arşiv segmenti en üstte ortada** → arama + filtre ikonu → yeni kayıt
- **Filtreler:** arama kutusunun **hemen sağında** `FilterOutlined` ikonlu trigger düğmesi; `<a-popover trigger="click">` ile açılır (`kp-list-filter-popover`). Aktif filtre sayısı `<a-badge>` rozeti ile gösterilir; aktifken `type="primary" ghost`. Popover içinde her alan `kp-list-filter__field`; alt footer'da **「Filtreyi temizle」** butonu — tüm built-in + declarative filtreleri sıfırlar. **Banka filtresi built-in** (`bank-filter` + `:banks`); diğer filtreler **declarative** `:filters="..."` prop'u ile (`ListFilter<T>` tipi). Mobilde popover viewport'a sığar (`max-width: calc(100vw - 16px)`)
- **Filtre tipleri (`ListFilter<T>`):**
  - **`select`** — `options[]` + `getValue(item)`; AntDV `<Select>` arama destekli (`textIncludesSearch`). Statüs, tür, hedef gibi sınıflandırmalar
  - **`numberRange`** — `getValue(item)` + opsiyonel `numberKind` (`currency`/`integer`/`percent`); iki `LocaleInputNumber` (Min – Maks). URL: `<key>From` + `<key>To`
  - **`dateRange`** — `getValue(item)` (ISO string); `<DatePicker.RangePicker>` (dayjs). Kıyaslama tarih kısmı (`slice(0, 10)`); URL: `<key>From=YYYY-MM-DD` + `<key>To=YYYY-MM-DD`
- **Filtre seçimi (sayfa türüne göre):**
  - **Borçlar → Krediler / Taksitli avans:** banka + durum (`active`/`overdue`/`closed`) + anapara + **aylık taksit** + **kalan** + vade (ay) + başlangıç tarihi
  - **Borçlar → Kredi kartları:** banka + limit + borç + **kullanılabilir** + **asgari ödeme**
  - **Borçlar → Nakit avans:** banka + limit + anapara + **işleyen faiz** + **toplam borç** + **kullanılabilir**
  - **Yönetim → Hesaplar:** banka + tür + açılış bakiyesi + açılış tarihi
  - **Yönetim → Kasalar:** açılış bakiyesi + açılış tarihi
  - **Yönetim → Bankalar / Gelir-Gider türleri:** **Durum** = kullanım durumu (`used` / `unused`); kayıt herhangi bir başka entity tarafından referans alınıyorsa `used`. Arşiv segmenti ile çakışmaz (arşiv = soft-delete; kullanım = etkin referans)
  - **Para birimi filtresi kullanma:** profil para birimi sabit; kayıtlar arası currency dağılımı anlamlı değil
  - **Nakit akışı → Gelir/Gider:** tür + hedef/kaynak (`account:<id>` / `cash:<id>`) + durum (`realized`/`overdue`/`due`/`upcoming`) + tutar + plan/gerçek tarih aralığı
  - **Nakit akışı → Transfer:** kaynak + hedef + tutar + tarih aralığı
- **Mobil liste:** `EntityListPage` tablo yerine **kart listesi** (`kp-list-card`); sayfalama korunur. Masaüstünde `<a-table>`
- **Dikey:** liste sayfası + sekme içeriği flex ile kalan viewport yüksekliğini doldurur; tablo gövdesi `scroll.y` (ResizeObserver) + `--kp-table-body-min-h` ile **kalan alanın tamamını** kaplar (az satırda da gövde yüksekliği korunur; satır sayısı az olsa da liste alanı boş kalmaz)
- **Yatay:** `EntityListPage` `scroll.x` = `max(sütun min toplamı, konteyner genişliği)` — tablo **her durumda en az container kadar** geniş; sütunlar dar ekranda min genişliklerini korur, geniş ekranda fazla alan sütunlara dağılır. `ant-table-cell-scrollbar` gizli (boş sağ şerit yok)
- **Sütun genişliği:** `prepareListTableColumns` — açık `width` korunur; yoksa `minWidth` (varsayılan 112px). **İstisna:** birincil ad `adminPrimaryNameColumn(title)` — **280px** (`ADMIN_PRIMARY_NAME_COLUMN_WIDTH`). İşlem sütunu `__actions` — 88px
- `<a-table>` client-side sort/page/filter (Dexie query)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kemalersin/kurtarma-plani](https://github.com/kemalersin/kurtarma-plani) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
