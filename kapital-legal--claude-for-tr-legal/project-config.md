---
trigger: always_on
description: Bu dosya, repo üzerinde çalışan geliştiriciler (insan veya AI) için yönlendirme verir. Plugin kullanıcıları için ana doküman [README.md](README.md)'dir.
---

# Claude for TR Legal — Geliştirici Notları

Bu dosya, repo üzerinde çalışan geliştiriciler (insan veya AI) için yönlendirme verir. Plugin kullanıcıları için ana doküman [README.md](README.md)'dir.

> **Türev Eser Notu:** Bu dosyanın yapı/format kalıbı (geliştirici dokümanı + repo felsefesi formatı), [anthropics/claude-for-legal](https://github.com/anthropics/claude-for-legal) projesinden uyarlanmıştır. İçerik Türk hukukuna ve Kapital Legal'in sponsorluk pozisyonuna göre yeniden yazılmıştır. Apache 2.0 lisans yükümlülükleri için `LICENSE` ve `NOTICE` dosyalarına bakınız.

## Repo Felsefesi

1. **Anthropic'in iskeletini koru, içeriği Türkçeleştir.** Plugin yapısı (`.claude-plugin/plugin.json` + `skills/*/SKILL.md` + cold-start interview deseni) Anthropic ile birebir aynı kalır. Sadece hukuki içerik, terminoloji ve veri kaynakları Türkçeleşir.

2. **"Draft for attorney review" prensibi.** Hiçbir plugin nihai hukuki tavsiye üretmez. Her çıktı taslak halinde, kaynak gösterilerek, avukat onayına hazır şekilde sunulur. Belirsiz konularda konservatif varsayılan kullanılır.

3. **Veri kaynakları açık ve doğrulanabilir.** Her karar/madde atıfı kaynağıyla birlikte verilir. Kullanıcının resmi kaynaklardan (kvkk.gov.tr, Resmi Gazete vb.) sağladığı içerik öncelendir. Model eğitim verisinden gelen atıflar `[doğrulanmalı]` etiketi taşır. Karar metinlerinin URL'i her zaman dahil edilmelidir.

4. **Paylaşılan firma profili — kendi mekanizmamız.** Bu repo, Anthropic'in plugin başına ayrı CLAUDE.md tutmasının üstüne **kendi paylaşım mekanizmasını** kurar:

   - **Paylaşılan profil:** `~/.claude/claude-for-tr-legal/firma-profili.md` (kök seviye, plugin-bağımsız)
   - **Plugin-spesifik profil:** `~/.claude/plugins/config/claude-for-tr-legal/<plugin>/CLAUDE.md` (her plugin için ayrı)
   - **Akış:** İlk kurulan plugin'in `cold-start-interview` skill'i önce firma profilini kontrol eder. Yoksa kullanıcıdan büro bilgilerini toplar ve **paylaşılan dosyaya** yazar. Sonra plugin-spesifik soruları sorar ve **plugin-spesifik dosyaya** yazar. Sonraki plugin'lerin `cold-start-interview`'u paylaşılan dosyayı okur, kullanıcıya özet gösterip onaylatır ve şirket sorularını **tekrar sormaz** — yalnızca plugin-spesifik sorulara odaklanır.
   - **Implementation referansı:** `kvkk-uyum-tr/skills/cold-start-interview/SKILL.md` (Adım 0 ve Adım 1)
   - **Template:** `references/firma-profili-template.md` (repo kökünde)

5. **TBB Meslek Kuralları gözetilir.** Repo kök seviyede `references/tbb-meslek-kurallari-ozet.md` var. Plugin geliştirenler PR'larında TBB kontrollerini açıkça not etmelidir.

## Klasör Konvansiyonları

```
claude-for-tr-legal/                     ← repo kök
  .claude-plugin/marketplace.json        ← marketplace metadata (plugin listesi)
  LICENSE                                 ← Apache 2.0
  NOTICE                                  ← Apache 2.0 m.4 atıf yükümlülüğü
  README.md, QUICKSTART.md, KATKI.md     ← kullanıcı dokümanları
  CLAUDE.md                               ← bu dosya (geliştirici notları)

  references/                             ← REPO KÖKÜNDE paylaşılan referans dosyalar
    firma-profili-template.md
    tbb-meslek-kurallari-ozet.md
    karar-atif-kurallari.md
    baro-danisma-notu.md

  <plugin-adı>-tr/                        ← her plugin kendi klasöründe
    .claude-plugin/plugin.json            ← {name, version, description, author}
    CLAUDE.md                             ← plugin'e özel practice profile template
    README.md                             ← plugin tanıtımı + skill listesi
    skills/
      cold-start-interview/SKILL.md
      <skill-adı>/SKILL.md
    references/                           ← plugin runtime için statik veri (paketleme sırasında plugin'le birlikte yüklenir)
      firma-profili-template.md           ← (repo kökündeki ile senkron tutulur)
    hooks/                                ← (opsiyonel) lifecycle hooks
    agents/                               ← (opsiyonel) sub-agent tanımları
```

**`references/` klasörü kararı (2026-05-31 turn-6'da revize):**

Türk plugin'ler için **iki yerde** `references/` bulunur:

1. **Repo kökünde `references/`** — geliştirici dokümantasyonu ve repo-wide paylaşılan referanslar:
   - `tbb-meslek-kurallari-ozet.md` — TBB MK + Av.K. madde özetleri
   - `karar-atif-kurallari.md` — halüsinasyon disiplini
   - `baro-danisma-notu.md` — TBB reklam/tabela değerlendirmesi
   - `veri-isleme-bildirimi.md` — Anthropic API iletim çerçevesi
   - `firma-profili-template.md` — geliştirici görüntüsü için kopyası

2. **Plugin içinde `<plugin>/references/`** — plugin runtime'ında gerekli olan statik veri:
   - Şu an yalnızca `firma-profili-template.md` (cold-start template)
   - Plugin başına gerektikçe başka template/şablonlar eklenir

**Neden iki yerde?** `claude plugin install` paketleme sırasında **yalnızca plugin klasörünü** kopyalar; repo kökündeki `references/` plugin paketine dahil edilmez. Cold-start skill'inin runtime'da template'e erişebilmesi için plugin **içinde** bir kopya zorunludur. Aynı dosya geliştirici görüntüsü için kökte de bulunur.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kapital-legal/claude-for-tr-legal](https://github.com/kapital-legal/claude-for-tr-legal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-06 -->
