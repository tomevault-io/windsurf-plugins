---
trigger: always_on
description: Bu dosya, kullanıcının global `AGENTS.md` kurallarını **tamamlayan repository-seviyesi talimatları** içerir.
---

# AgentCheck — Repository Agent Çalışma Sözleşmesi

## 1. Amaç ve Kapsam

Bu dosya, kullanıcının global `AGENTS.md` kurallarını **tamamlayan repository-seviyesi talimatları** içerir.

- Global kuralları gevşetmez.
- Global güvenlik, yetki ve onay sınırlarını kaldırmaz.
- AgentCheck'e özgü ürün, mimari, kapsam ve doğrulama kurallarını tanımlar.
- Bir kural çelişkili görünüyorsa sessizce yorumlama yapma; daha kısıtlayıcı davranışı seç ve önemliyse kullanıcıya bildir.

Bu repository için kalıcı ürün hedefi:

> AgentCheck, AI destekli yazılım geliştirme sonrasında repository'de ne değiştiğini, neyin şüpheli olduğunu ve commit öncesi nelerin incelenmesi gerektiğini bağımsız olarak gösteren bir doğrulama aracıdır.

AgentCheck başka bir coding agent değildir.

---

## 2. v0.1 Ürün İlkeleri

v0.1 şu özellikleri korumalıdır:

- Basit
- Hızlı
- Local-first
- Güvenilir
- Mümkün olduğunca deterministik
- AI olmadan faydalı
- Kullanılan coding agent'tan bağımsız
- Kolay kurulabilir
- Kolay anlaşılır
- Kolay kaldırılabilir
- Açık kaynak

Ana doğrulama döngüsü:

```text
Coding agent çalışır
        ↓
Agent "done" der
        ↓
Developer AgentCheck çalıştırır
        ↓
Değişiklikleri ve bulguları inceler
        ↓
Commit eder
```

Bir özellik şu soruya yardımcı olmuyorsa büyük olasılıkla v0.1 kapsamına ait değildir:

> "Coding agent işi bitirdiğini söylüyor. Ne değişti, ne şüpheli ve commit etmeden önce neyi incelemeliyim?"

---

## 3. Onaylanmış v0.1 Teknik Yönü

Aşağıdaki kararlar bu repository için **önceden onaylanmış teknik yön** kabul edilir. Bunları uygulamak için yeniden kullanıcı onayı isteme.

### Teknoloji

- TypeScript / Node.js
- Minimum hedef Node.js: `>=22`
- Küçük bir monorepo
- Ortak iş mantığı `@agentcheck/core` içinde
- CLI ve VS Code aynı core paketini kullanır
- CLI ve VS Code için ayrı business logic oluşturulmaz

Beklenen üst seviye yapı:

```text
packages/
  core/
  cli/
  vscode/
```

Yapı ihtiyaç halinde küçük ölçüde iyileştirilebilir; yeni katman/pattern eklemek gerekiyorsa global onay kurallarına uy.

### Checkpoint mimarisi

Checkpoint ve repository karşılaştırma semantiği için `SEMANTICS.md` **source of truth** kabul edilir.

Onaylanmış temel yaklaşım:

- Kendi tam dosya snapshot sistemimizi kurma.
- Git'in tree/index mekanizmasını kullan.
- Developer'ın gerçek Git index'ine dokunma.
- Geçici/alternatif Git index kullan.
- AgentCheck'e ait Git objelerini mümkün olduğunca AgentCheck metadata alanında izole et.
- Working tree'yi değiştirme.
- Stash/reset/checkout/commit gibi destructive veya state-changing Git işlemleri kullanma.
- Dirty working tree ve untracked dosyaları doğru ele al.
- Karşılaştırmayı checkpoint anındaki state ile mevcut state arasında yap.

Bu semantiği değiştirmek yeni bir mimari/persistence kararıdır; kullanıcı onayı olmadan değiştirme.

---

## 4. v0.1 Kapsam Dışı

Açık kullanıcı kararı olmadan aşağıdakileri uygulama:

- Backend
- Veritabanı
- Kullanıcı hesabı
- Login/auth
- Cloud storage
- Telemetry
- Payment/subscription
- LLM veya AI entegrasyonu
- OpenAI/Anthropic entegrasyonu
- Agent-specific API entegrasyonu
- Claude Code/Codex/Cursor'a özel entegrasyon
- Embedding/vector database
- Repository-wide knowledge graph
- GitHub App
- PR bot
- GitHub Actions ürünü
- Team/organization özellikleri
- Enterprise dashboard
- Otomatik code review düzeltmeleri
- Test generation
- Kaynak kodu otomatik değiştirme
- Otomatik fix

v0.1 analiz eder; developer'ın kaynak kodunu değiştirmez.

---

## 5. Uygulama İlkeleri

Her görevde:

1. Önce ilgili mevcut kodu ve dokümanları incele.
2. Görevi karşılayan en küçük tam çözümü uygula.
3. Spekülatif abstraction oluşturma.
4. Sadece olası gelecek ihtiyaçlar için interface/factory/layer ekleme.
5. Strong typing tercih et.
6. Core katmanını VS Code API'sinden bağımsız tut.
7. CLI presentation kodunu core davranışına karıştırma.
8. Analyzer'ları bağımsız test edilebilir tut.
9. Production dependency sayısını düşük tut.
10. Deterministik ve açıklanabilir davranışı tercih et.
11. Hata durumlarında anlaşılır ve güvenli biçimde fail et.
12. Cross-platform path/command davranışını dikkate al.
13. Shell string birleştirmek yerine mümkün olduğunda process argument API'lerini kullan.
14. Kullanıcının gerçek Git index'ini, working tree'sini veya history'sini değiştirme.
15. İlgisiz refactor yapma.

Kodun mevcut basitliği bir değerdir. Daha "kurumsal" görünmesi tek başına abstraction ekleme gerekçesi değildir.

---

## 6. Git Güvenlik Kuralları

AgentCheck'in güvenilirliği repository-state karşılaştırmasına bağlıdır.

Agent:

- `.git` yolunun her zaman fiziksel bir klasör olduğunu varsaymamalıdır.
- Git metadata yollarını mümkün olduğunda Git komutlarıyla çözmelidir.
- Gerçek index üzerinde `git add`, `git reset` veya eşdeğer state-changing işlem çalıştırmamalıdır.
- Snapshot oluşturmak için gerçek index yerine alternatif/geçici index kullanmalıdır.
- Kullanıcının staged değişikliklerini korumalıdır.
- Pre-existing dirty state'i checkpoint'in bir parçası olarak kabul etmelidir.
- Non-ignored untracked dosyaları checkpoint karşılaştırmasına dahil etmelidir.
- Ignored untracked dosyaları v0.1'de kapsam dışında tutmalıdır.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emreordu/agentcheck](https://github.com/emreordu/agentcheck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
