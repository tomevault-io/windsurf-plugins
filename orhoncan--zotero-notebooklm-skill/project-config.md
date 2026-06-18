---
trigger: always_on
description: Zotero koleksiyonları ile NotebookLM defterleri arasında iki yönlü senkronizasyon — defter oluştur, kaynak yükle, podcast/slayt/harita üret, deep research ile genişlet
---


Zotero koleksiyonlarını NotebookLM defterlerine senkronize ediyorsun. Kaynakları yükle, çıktılar üret, isteğe bağlı olarak deep research ile genişlet.

## Setup

**Bu adımları ilk mesajda paralel olarak yap:**

1. **Zotero MCP kontrolü:** `ToolSearch` ile `"zotero add_by_doi"` ara. `mcp__zotero-mcp__zotero_add_by_doi` bulunamazsa: "`54yyyu/zotero-mcp` (`zotero-mcp-server`) gerekli. Kurulum: `uv tool install zotero-mcp-server`" yaz ve dur.

2. **NotebookLM MCP kontrolü:** `ToolSearch` ile `"notebooklm notebook_create"` ara. `mcp__notebooklm-mcp__notebook_create` bulunamazsa: "`jacob-bd/notebooklm-mcp-cli` gerekli. Kurulum: `uv tool install notebooklm-mcp-cli`" yaz ve dur.

3. **Auth kontrolü:** `mcp__notebooklm-mcp__server_info` çağır. Auth hatası gelirse kullanıcıyı `! nlm login` çalıştırmaya yönlendir.

Her iki MCP de hazırsa devam et.

**Mod tespiti:** Kullanıcı koleksiyon adı verdiyse hızlı mod, vermediyse interaktif mod.

---

## Adım 1 — Koleksiyon Seçimi

### Hızlı mod (koleksiyon adı verildi)
- `mcp__zotero-mcp__zotero_search_collections` ile verilen adı ara.
- Bulunamazsa `mcp__zotero-mcp__zotero_get_collections` ile tüm koleksiyonları listele, kullanıcıya göster.

### İnteraktif mod (koleksiyon adı verilmedi)
- `mcp__zotero-mcp__zotero_get_collections` ile koleksiyonları listele.
- Kullanıcıdan numara veya isimle seçim iste.

### Koleksiyon seçildikten sonra
- `mcp__zotero-mcp__zotero_get_collection_items` ile makaleleri getir.
- Eğer >15 makale varsa uyar: "Bu koleksiyonda {N} makale var. NotebookLM'de optimum 10-15 kaynak. Hepsini yükleyelim mi, yoksa alt seçim yapmak ister misin?"

---

## Adım 2 — Defter Oluşturma

- `mcp__notebooklm-mcp__notebook_list` ile mevcut defterleri kontrol et.
- Aynı isimde defter varsa sor: "'{name}' adında bir defter zaten var. Mevcut deftere ekle mi, yenisini oluştur mu?"
- Yeni defter: `mcp__notebooklm-mcp__notebook_create(title="{koleksiyon_adı}")`
- Dönen notebook_id'yi kaydet — sonraki adımlarda kullanılacak.

---

## Adım 3 — Kaynak Yükleme

Her makale için sırayla:

1. `mcp__zotero-mcp__zotero_get_item_fulltext` ile tam metni al.

2. **NotebookLM'e metin kaynağı olarak ekle:**
   `mcp__notebooklm-mcp__source_add(notebook_id="{nb_id}", source_type="text", text="{fulltext}", title="{makale_başlığı}")`

3. `mcp__zotero-mcp__zotero_get_annotations` ile anotasyonları kontrol et. Varsa:
   - Anotasyonları derle ve ayrı bir metin kaynağı olarak ekle:
   `mcp__notebooklm-mcp__source_add(notebook_id="{nb_id}", source_type="text", text="{derlenmiş_anotasyonlar}", title="Araştırmacı Notları: {makale_adı}")`

**İlerleme:** Her yükleme sonrası "{loaded}/{total} kaynak yüklendi..." yaz.

**Hata yönetimi:**
- Yüklenemeyen makaleleri atla, raporda not et.
- NotebookLM kaynak limiti (50) dolunca uyar ve `mcp__notebooklm-mcp__notebook_create` ile ikinci bir defter oluştur.

---

## Adım 4 — Readwise Entegrasyonu (opsiyonel)

**Yalnızca kullanıcı açıkça istediğinde tetiklenir** ("Readwise highlight'larımı da ekle" veya `readwise` komutu).

1. `ToolSearch` ile readwise araçlarını kontrol et. Yoksa "Readwise MCP mevcut değil, bu adım atlanıyor." de ve geç.
2. Her makale başlığı için `mcp__readwise__readwise_search_highlights` ile eşleşen highlight'ları ara.
3. Eşleşen highlight'ları makale bazında grupla, tek bir metin kaynağı olarak deftere ekle:
   `mcp__notebooklm-mcp__source_add(notebook_id="{nb_id}", source_type="text", text="{derlenmiş_highlights}", title="Readwise Highlight'ları")`
4. Eşleşme yoksa: "Readwise'da eşleşen highlight bulunamadı."

---

## Adım 5 — Çıktı Seçimi (interaktif modda)

Hızlı modda bu adımı atla.

Kullanıcıya seçenekleri sun (birden fazla seçilebilir):

| Komut | Çıktı | MCP Aracı |
|-------|-------|-----------|
| `podcast` | Audio Overview | `mcp__notebooklm-mcp__studio_create(notebook_id, artifact_type="audio")` |
| `slayt` | Slayt sunumu | `mcp__notebooklm-mcp__studio_create(notebook_id, artifact_type="slides")` |
| `video` | Video | `mcp__notebooklm-mcp__studio_create(notebook_id, artifact_type="video")` |
| `infografik` | İnfografik | `mcp__notebooklm-mcp__studio_create(notebook_id, artifact_type="infographic")` |
| `harita` | Literatür haritası | `mcp__notebooklm-mcp__notebook_query` ile metin sorgusu |
| `özet` | Karşılaştırmalı özet | `mcp__notebooklm-mcp__notebook_query` ile metin sorgusu |
| `sorgu` | Serbest soru | `mcp__notebooklm-mcp__notebook_query` |
| `yok` | Çıktı yok, defter hazır | — |

### podcast / slayt / video / infografik
- `mcp__notebooklm-mcp__studio_create(notebook_id="{nb_id}", artifact_type="{tip}", confirm=true)` ile oluştur.
- `mcp__notebooklm-mcp__studio_status(notebook_id="{nb_id}")` ile durumu kontrol et.
- Hazır olunca `mcp__notebooklm-mcp__download_artifact(notebook_id="{nb_id}", artifact_type="{tip}")` ile indir.
- Uzun sürerse: "Oluşturuluyor, 3-5 dk sürebilir. Kontrol edeyim mi?"

### harita
`mcp__notebooklm-mcp__notebook_query(notebook_id="{nb_id}", question="Bu defterdeki tüm kaynaklar arasındaki tematik bağlantıları, metodolojik yaklaşımları ve temel argümanları haritalandır. Hangi makale hangi makaleyle ne konuda örtüşüyor, nerede ayrışıyor?")` ile sor.

### özet

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orhoncan/zotero-notebooklm-skill](https://github.com/orhoncan/zotero-notebooklm-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
