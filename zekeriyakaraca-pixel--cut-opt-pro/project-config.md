---
trigger: always_on
description: Bu dosya, CutOpt Pro v2.x projesinin genel mimari yapısını, klasör hiyerarşisini, kod bağımlılıklarını ve geliştirme prensiplerini açıklamaktadır.
---

# CutOpt Pro - Proje Mimarisi ve Geliştirici Rehberi

Bu dosya, CutOpt Pro v2.x projesinin genel mimari yapısını, klasör hiyerarşisini, kod bağımlılıklarını ve geliştirme prensiplerini açıklamaktadır.
Yeni özellik geliştirirken veya hata ayıklarken bu rehberdeki mimari kurallara uyulması esastır.

---

## Genel Bakış

**CutOpt Pro**, profil ve boru kesim işlemleri için geliştirilmiş **1-Boyutlu Kesim Stok Problemi (1D Cutting Stock Problem)** optimizasyon yazılımıdır.
Proje, eski monolitik yapısından (v1) çıkarak **Modüler Mimari (v2)** yapısına geçiş yapmıştır.

- **Frontend** (PySide6 / Qt) ve **Backend** (İş Mantığı, Algoritmalar, Raporlama) birbirinden tamamen izole edilmiştir.
- Sistem çok dilli destek (i18n / TR + EN), CNC entegrasyonu (DSTV/NC1), lisanslama ve MCP tabanlı AI entegrasyonu içermektedir.
- Ticari bir üründür: Trial / Basic / Professional lisans katmanları mevcuttur.

---

## Klasör Hiyerarşisi

```text
kesim_qt_build/
│
├── main.py                         # Uygulama giriş noktası; QApplication, MainApp(QMainWindow),
│                                   # QUndoStack, sekme yönetimi, lisans kontrolü, autosave timer
├── app_config.json                 # Uygulama ayarları: tema, dil, aktif proje yolu, otomatik kayıt aralığı
├── profil_database.json            # Profil geometri veritabanı (IPE, HEA, SHS vb. h/b/tw/tf/kg_m)
├── requirements.txt                # Python paket bağımlılıkları (sürüm kısıtlamalı)
├── CutOptPro.spec                  # PyInstaller konfigürasyonu (hidden imports, data files, PySide6)
├── RELEASE_BUILD.bat               # Yayın sürümü derleme betiği
├── setup_script.iss                # Inno Setup Windows yükleyici scripti
├── LICENSE.txt                     # Ticari yazılım lisans metni
├── E-378.nc1                       # DSTV/NC1 format örnek dosyası (test için)
├── COP-*.lic                       # Test lisans dosyaları (dev, trial, basic)
│
├── (Yardımcı Scriptler - Kök)
│   ├── add_international_profiles.py  # Profil veritabanına TR/EN çeviri ekler
│   ├── enrich_db.py                   # Profil veritabanını kg/m değerleriyle zenginleştirir
│   ├── gen_ses.py                     # Oturum üretici yardımcısı
│   ├── fix_script.py                  # Veri onarım aracı
│   └── create_icon.py                 # Uygulama ikonu üretici
│
├── backend/                        # [İş Mantığı, Veri, Raporlar, Algoritmalar — Qt BAĞIMSIZ]
│   │
│   ├── core/                       # v2 Modüler Mimari Çekirdeği
│   │   │
│   │   ├── algorithms/             # Optimizasyon Motorları
│   │   │   ├── optimizer.py        # Ana koordinatör; problem boyutuna göre algoritma otomatik seçer
│   │   │   ├── greedy.py           # FFD / BFD / WFD + 2-opt post-processing
│   │   │   ├── column_generation.py# Column Generation + Linear Programming (scipy)
│   │   │   ├── genetic.py          # Genetik Algoritma (OX-crossover, swap+reverse mutasyon)
│   │   │   ├── branch_bound.py     # Branch & Bound (küçük örnekler için)
│   │   │   └── common.py           # Algoritmalar arası paylaşılan yardımcılar
│   │   │
│   │   ├── business/               # Servis Katmanı (Qt-free, Unit Test edilebilir)
│   │   │   ├── optimization_service.py  # Optimizasyon iş akışı orkestrasyonu
│   │   │   ├── stock_service.py         # Stok/envanter yönetimi
│   │   │   ├── project_service.py       # Çoklu proje CRUD işlemleri
│   │   │   ├── stats_service.py         # Dashboard istatistikleri (verimlilik, fire, maliyet)
│   │   │   ├── cost_service.py          # Malzeme maliyet hesaplamaları
│   │   │   ├── batch_service.py         # Toplu (batch) operasyonlar
│   │   │   └── command_service.py       # Undo/redo komutları için iş mantığı
│   │   │
│   │   └── data/                   # Veri Katmanı
│   │       ├── models.py           # Dataclass'lar: StokItem, ParcaItem, ProjeData, KesimSonucu
│   │       ├── repository.py       # Repository Pattern; JSON + Excel dosya I/O, atomik yazma
│   │       └── validators.py       # Giriş doğrulama kuralları
│   │
│   ├── i18n/                       # Çok Dilli Destek
│   │   ├── __init__.py             # i18n motoru; tr/en JSON tabanlı, Türkçe fallback
│   │   ├── tr.json                 # Türkçe çeviriler (~19KB)
│   │   └── en.json                 # İngilizce çeviriler (~17KB)
│   │
│   ├── reports/                    # Raporlama Modülleri
│   │   ├── base.py                 # PDF/Excel raporlayıcıları için abstract taban sınıf
│   │   ├── excel/
│   │   │   ├── exporter.py         # Koşullu formatlı, grafikli Excel export
│   │   │   ├── parser.py           # Excel veri okuma
│   │   │   └── styles.py           # Hücre stilleri ve formatlama
│   │   ├── pdf/
│   │   │   ├── cutting_card.py     # Operatör kesim kartı (ReportLab)
│   │   │   ├── material_list.py    # Malzeme listesi PDF
│   │   │   └── visual_plan.py      # Görsel kesim planı PDF
│   │   └── txt/
│   │       ├── cutting_report.py   # Metin tabanlı kesim raporu
│   │       ├── material_report.py  # Metin tabanlı malzeme raporu
│   │       └── operator_card.py    # Metin tabanlı operatör kartı
│   │
│   ├── utils/                      # Yardımcı Araçlar

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zekeriyakaraca-pixel/Cut_opt_pro](https://github.com/zekeriyakaraca-pixel/Cut_opt_pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
