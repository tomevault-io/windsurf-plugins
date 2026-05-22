---
trigger: always_on
description: Bu proje, şirket içi kıdemli full stack yazılım geliştiricilere yönelik kapsamlı bir Elasticsearch eğitimi için materyaller geliştirmeyi amaçlamaktadır. Temel hedef, katılımcıların Elasticsearch'ü etkin bir şekilde anlamalarını ve kullanmalarını sağlamaktır.
---

# Proje Talimatları: Elasticsearch Eğitimi Materyalleri

## 1. Projeye Genel Bakış

Bu proje, şirket içi kıdemli full stack yazılım geliştiricilere yönelik kapsamlı bir Elasticsearch eğitimi için materyaller geliştirmeyi amaçlamaktadır. Temel hedef, katılımcıların Elasticsearch'ü etkin bir şekilde anlamalarını ve kullanmalarını sağlamaktır.

## 2. Temel Hedef

Katılımcıların Elasticsearch'ün temel mimarisini, anahtar kavramlarını anlamalarını, veri indeksleme (indexing), sorgulama (searching), analiz (aggregations) yeteneklerini kavramalarını ve bu bilgileri pratik senaryolarda (özellikle log yönetimi odaklı) uygulayabilir hale gelmelerini sağlamaktır.

## 3. Hedef Kitle

Şirket içinde çalışan, orta ve kıdemli seviye Full Stack Yazılım Geliştiriciler. Bu kitlenin REST API'ler, JSON, temel veritabanı ve dağıtık sistem kavramlarına aşina olduğu varsayılmaktadır.

## 4. Eğitim Formatı

* **Süre:** 1 günlük, toplam 6 saatlik aktif eğitim.
* **Tür:** Şirket içi iç eğitim.
* **Yaklaşım:** Teorik bilgilerin pratik uygulamalarla pekiştirildiği, interaktif bir eğitim.

## 5. Bugüne Kadar Oluşturulan Temel Çıktılar (Artifacts)

Bu projede şu ana kadar aşağıdaki iki ana doküman oluşturulmuştur:

1. **`elasticsearch_egitim_icerigi_v2_md_kaynak` (Elasticsearch Eğitimi: Kapsamlı Ders İçeriği - Markdown Kaynak):**
    * **Açıklama:** Eğitmenin kullanacağı, 6 saatlik ders akışını, modül detaylarını, konu başlıklarını, ayrılan süreleri (eğitim planlaması için) ve pratik uygulama adımlarını içeren detaylı bir ders planıdır.
    * **Dil:** Türkçe.
    * **Format:** Markdown.
    * **Amacı:** Eğitmenin dersi yapılandırması, slayt ve diğer materyalleri hazırlaması için bir yol haritası sunmak.

2. **`elasticsearch_ders_kitabi_v1` (Elasticsearch Macerası: Bir Geliştiricinin Kılavuzu - Markdown):**
    * **Açıklama:** Yukarıdaki ders içeriğini temel alan, öğrencilere yönelik hazırlanmış bir ders kitabıdır.
    * **Dil:** Anlatım dili Türkçe, teknik terimler, kod örnekleri, index/alan adları İngilizce'dir.
    * **Ton:** Öğrenci dostu, samimi, yer yer esprili ve ilgi çekici bir dil kullanılmıştır.
    * **Format:** Markdown.
    * **Amacı:** Öğrencilere ders öncesinde ön okuma materyali olarak sunmak, ders sırasında ve sonrasında başvuru kaynağı olarak kullanmalarını sağlamak. Log yönetimi senaryolarına özel vurgu yapılmıştır.

## 6. İçerik ve Stil Kuralları (Özellikle Ders Kitabı İçin)

* **Anlatım Dili:** Türkçe. Akıcı, anlaşılır ve samimi bir üslup tercih edilmelidir.
* **Teknik Terimler ve Kod Örnekleri:** Tüm Elasticsearch komutları, API endpoint'leri, index adları, alan adları (fields), JSON yapıları ve kod örnekleri **İngilizce** olmalıdır. Bu, uluslararası standartlara ve resmi dokümantasyona uyumu sağlar.
* **Ton:** Eğlenceli, esprili ve motive edici bir dil kullanılmalı, ancak teknik doğruluktan ödün verilmemelidir. "Cesur geliştirici", "veri okyanusu" gibi ifadeler bu tonu yansıtır.
* **Pratik Odaklılık:** Teorik bilgiler pratik örneklerle desteklenmelidir. Özellikle şirket içi yaygın kullanım alanı olan **log yönetimi ve analizi** senaryolarına özel örnekler ve vurgular içermelidir.
* **Resmi Dokümantasyona Atıflar:** Öğrencilerin daha fazla bilgi edinebilmeleri için ilgili ve önemli noktalarda Elasticsearch resmi dokümantasyonuna yönlendirmeler (linkler) eklenmelidir.
* **Öğrenci Dostu Yaklaşım:** Karmaşık konular basitleştirilerek anlatılmalı, "Neden?" sorularına cevap verilmelidir.
* **Markdown Formatı:** Tüm metin tabanlı çıktılar (ders kitabı, ders içeriği) Markdown formatında olmalıdır.

## 7. Projenin Mevcut Durumu

* Detaylı ders içeriği (`elasticsearch_egitim_icerigi_v2_md_kaynak`) oluşturulmuştur.
* Bu ders içeriğini temel alan, loglama örneklerini de içeren kapsamlı bir öğrenci ders kitabı (`elasticsearch_ders_kitabi_v1`) tamamlanmıştır.

## 8. LLM (Yapay Zeka Modeli) İçin Bağlam Notları

* Bu talimat dosyası, bu projeyle ilgili gelecekteki tüm etkileşimlerde LLM'e bağlam sağlamak için kullanılacaktır.
* LLM'den istenen güncellemeler veya yeni içerikler, yukarıda belirtilen stil, ton, dil ve içerik kurallarına uygun olmalıdır.
* Özellikle "log yönetimi" senaryolarının şirket içi eğitim için önemli olduğu unutulmamalıdır.
* Yeni içerik veya değişiklik taleplerinde, mevcut dokümanların (`elasticsearch_ders_kitabi_v1` ve `elasticsearch_egitim_icerigi_v2_md_kaynak`) en güncel versiyonları referans alınmalıdır.
* Projenin genel amacı, geliştiricilerin Elasticsearch'ü etkili bir şekilde öğrenmelerini ve kullanmalarını sağlamaktır.

## 9. Ders İçin Pratik Uygulama: ".NET Log Simülatörü"

Ders sırasında teorik bilgileri pekiştirmek ve canlı verilerle çalışabilmek amacıyla, log yazımını simüle eden bir .NET uygulaması geliştirilecektir. Bu uygulama, katılımcıların Elasticsearch'ü dinamik bir ortamda test etmelerini sağlayacaktır.

* **Teknoloji:** Uygulama, **.NET 9.0** ve **ASP.NET Core 9.0 Razor Pages** kullanılarak geliştirilecektir.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DogusTeknoloji/ElasticsearchTraining](https://github.com/DogusTeknoloji/ElasticsearchTraining) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
