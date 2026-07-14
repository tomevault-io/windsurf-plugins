---
trigger: always_on
description: > Bu dosya gelecek katkılar (insan veya AI) için **yazım sesinin
---

# CLAUDE.md — Yazım Stili & Editorial Rehber

> Bu dosya gelecek katkılar (insan veya AI) için **yazım sesinin
> tutarlı kalmasını** garanti eder. Repo'nun benzersiz "tonu" — yargılı,
> eylemsel, Türkçe, placeholder-güvenli — bu kurallarla korunur.

---

## 🎯 Hedef Okuyucu

- **Junior'dan staff/principal'a** Türk DevOps/SRE/Platform mühendisi
- **Production'da** sorunla karşılaşan, "şimdi ne yapayım?" sorusunun cevabını arayan
- **Aksanlı İngilizce'den Türkçe'ye** geçiş yapan, "buzzword listesi" değil **eylem** isteyen
- **Kıdemli mühendis** kendi ekibine paylaşmak için "tek link" gönderebilmek isteyen

---

## ✍️ Yazım Sesi (Voice & Tone)

### Yargılı, Net, Türkçe
| ✅ Yap | ❌ Yapma |
|---|---|
| "Trunk-based artık standart. Git Flow'u kullanma." | "Bazı ekipler Git Flow kullanırken, diğerleri trunk-based tercih edebilir." |
| "Production'da `:latest` yasak." | "Mümkünse `:latest` kullanmamak iyi olur." |
| "PSP kalktı (k8s 1.25+); PSS kullan." | "PSP'nin yerini PSS aldı, opsiyonel olarak değerlendirilebilir." |

### Eylemsel, Adımlı
- "Şunu yap, şu komutu çalıştır, şu çıktıyı bekle"
- "Niye yapıldığı" 1-2 cümle, tarihsel anlatım yok

### Kısa, Damıtılmış
- Akademik tonda 5 paragraf yerine, 1 tablo + 3 bullet
- "Aşağıda tartışacağımız konu..." yok
- Direkt ana fikir

---

## 📐 Doküman Anatomi

Yeni bir deep-dive yazıyorsan **şu sırayla**:

```markdown
# Başlık — Kısa Alt Başlık

> *"Tek satırlık iddia veya alıntı."* — kaynak (varsa)

Giriş paragrafı (3-4 satır):
- Bu doküman ne anlatır
- Kim için yararlı
- Bitirdiğinde okuyucu ne yapabilecek

---

## 🎯 Temel Kavramlar / Tanımlar (varsa)

| Tablo |
|---|

## 🔧 Ana Bölümler (numaralı)

### 1️⃣ veya ### 1. Şunun adı

Kod / yaml örneği (gerçek, çalışır)
Anti-pattern karşılaştırması

## 🚫 Anti-Pattern Tablosu

| Anti-pattern | Niye kötü | Doğru |
|---|---|---|

## 📋 Checklist

```
[ ] Kontrol 1
[ ] Kontrol 2
```

## 📚 Referanslar
- Liste

---

> *"Kapanış sözü — bir cümlelik özet/insight."*
```

> 🔑 **Her yeni dosya bu iskeleti izlesin.** Tutarlılık okuyucuya
> "her doküman aynı şekilde okunuyor" rahatlığı verir.

---

## 🧱 Stil Kuralları

### Markdown
- **H1** dosya başına 1 tane
- **H2** ana bölüm; emoji opsiyonel ama tutarlı (🎯 için "Hedef", 📋 için "Checklist", 🚫 için "Anti-pattern")
- **H3** alt bölüm
- Code block: dil belirt (` ```yaml `, ` ```bash `, ` ```python `)
- Liste: `-` ile başla
- Numaralı: `1.` ya da `1️⃣` (görsel emfaz için)

### Türkçe-İngilizce
- Akronim / protokol / tool adı: **Bırak.** (`Kubernetes`, `TLS`, `OIDC`)
- Kavram: **Türkçeleştir** (`vulnerability` → `açık`, `audit log` → `denetim kaydı`)
- İlk kullanımda: `Service Level Indicator (SLI)` formu, sonra sadece `SLI`
- Belirsizse: [`Glossary.md`](Glossary.md)'a bak

### Placeholder güvenliği — **KIRMIZI ÇİZGİ**
- ❌ Gerçek IP, gerçek domain, gerçek e-mail, gerçek credential **ASLA**
- ✅ `<TARGET_IP>`, `<DOMAIN>`, `<NAMESPACE>`, `<REGISTRY>`, `<APP>`, `<VERSION>`, `<KMS_KEY_ID>`
- Sürüm numarası: `<VERSION>` veya `<CHART_VERSION>` (rakam değil)
- GitHub Actions: `aquasecurity/trivy-action@<VERSION>` (SHA pin gerçek versiyondan iyi olsa da örnekte placeholder)

### Komut blokları
- Multi-line uzun komutlar: `\` ile satır kır
- Çıktı varsa: yorum (`#`) olarak göster
- Komutu önce yaz, sonra ne yaptığını açıkla

```bash
# ✅ İyi
kubectl get pods -n <NAMESPACE> -l app=<APP> \
  --field-selector=status.phase=Running

# ❌ Kötü (gerçek değer)
kubectl get pods -n production -l app=payment-svc
```

---

## 🛡️ İçerik Kuralları

### "Niye" mutlaka
Bir kural / öneri / kontrol verirken **niye** olduğunu yaz. 1 cümle yeter.

```markdown
### `automountServiceAccountToken: false`
Pod ServiceAccount token'a ihtiyaç duymuyorsa kapat — compromised
container içinden K8s API'ye erişimi engeller.
```

### Anti-pattern tablosu **her doküman**
Her deep-dive sonunda **mutlaka** anti-pattern tablosu olmalı. 8-15 satır.
"Niye kötü" + "doğru" sütunları her zaman.

### Checklist her doküman
Sonda bir `[ ]` checklist — "production-ready'e ulaşmak için neler".

### Türkiye spesifik notlar
KVKK, BDDK, yerli vendor (örn: Iyzico), Türkçe iş kültürü dinamikleri
gerektiğinde 🇹🇷 emoji ile vurgulu kutu içinde.

```markdown
> 🇹🇷 **KVKK notu:** ...
```

### Referans bölümü
- Açık kaynak: GitHub URL veya proje sayfası
- Kitap: yazar + başlık (URL opsiyonel)
- Paywall'lı kaynaklar: belirt
- Repo'nun kendi dosyaları: relative link

---

## 🚧 Yapılması Yasak

### Pazarlama / Tedarikçi Tonu
- ❌ "Vendor X 'pazardaki en iyi çözüm' sunar"
- ❌ "Kurumsal-grade synergy" gibi boş laflar
- ✅ Sade tarif, tradeoff, "şu durumda iyi, şu durumda kötü"

### Klişeler
- ❌ "Modern dünyada artık geleneksel yaklaşımlar yetersiz..."
- ❌ "DevOps yolculuğunuzun başında..."
- ❌ "Topluluğun gücüyle..."

### Aşırı emoji
- ✅ Section header'da 1 emoji OK
- ❌ Cümle ortasında, "Çok güzel 🎉🚀✨" tarzı yasak

### LLM-vibe yapısal kalıplar
- ❌ "Aşağıda tartışacağımız konular şunlardır:" + bullet
- ❌ "Sonuç olarak görmüş olduk ki..."
- ❌ Aşırı paragraf girişi/çıkışı simetrisi
- ✅ Direkt başla, direkt bitir

### Çeviri zorlamaları
- ❌ "Sürekli entegrasyon (Continuous Integration)" — sürekli tekrar
- ✅ "CI" — ilk kullanımda tanım, sonra akronim

---

## 🧪 Yeni Doküman Yazmadan Önce

### 1. Repo'da var mı?
```bash
grep -ri "<TOPIC>" --include="*.md"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [halilibrahimd27/DevOps](https://github.com/halilibrahimd27/DevOps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
