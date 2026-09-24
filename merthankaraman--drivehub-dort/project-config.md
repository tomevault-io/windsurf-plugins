---
trigger: always_on
description: > **Proje:** MG4 elektrikli araç için Android Automotive controller uygulaması (DriveHub Dort)
---

# DriveHub Dort (MG4 EH32)

> **Proje:** MG4 elektrikli araç için Android Automotive controller uygulaması (DriveHub Dort)  
> **Platform:** Android Automotive OS (AAOS) - EH32 donanım  
> **Paket:** `com.drivehub.dort`  
> **Dil:** Java  
> **Min SDK:** 28 (Android 9)  
> **Target SDK:** 33 (Android 13)

---

## 📋 Proje Amacı

MG4 EH32'de çalışan bir Android uygulaması ile araç kontrolcüsüne (VHAL - Vehicle HAL) direkt Binder üzerinden erişim sağlayarak:
- Sürüş modunu değiştirme (Eco, Normal, Sport, Kar)
- Regen seviyesini ayarlama (Kapalı, Düşük, Orta, Yüksek, Adaptif)
- Tek pedal modunu açma/kapama
- Direksiyon ısıtmasını açma

**Hardkey entegrasyonu:** Direksiyondaki favori tuşu (keyCode 66) ile sürüş modunu döngüsel değiştirme.

---

## 🏗️ Teknik Altyapı

### Binder IPC
Android Automotive'de araç servisleri `ServiceManager` üzerinden Binder ile erişilir:
```java
IBinder binder = ServiceManager.getService("vehiclesetting");
```

### Kullanılan Servisler
| Servis Adı | Descriptor | Görev |
|---|---|---|
| `vehiclesetting` | `com.saicmotor.sdk.vehiclesettings.IVehicleSettingService` | Sürüş modu, regen, tek pedal |
| `aircondition` | `com.saicmotor.sdk.vehiclesettings.IAirConditionService` | Direksiyon ısıtma |

### Binder Protokolü (vehiclesetting)
```java
Parcel data = Parcel.obtain();
data.writeInterfaceToken("com.saicmotor.sdk.vehiclesettings.IVehicleSettingService");
data.writeInt(16777216);           // Area ID (0x01000000) - ZORUNLU
data.writeInt(1);                  // Count - ZORUNLU (yoksa araç reddeder)
data.writeInt(value);              // Ayarlanacak değer
data.writeFloatArray(new float[0]);
data.writeByteArray(new byte[0]);
binder.transact(TX_ID, data, reply, 0);
```

### Transaction ID'ler
| Fonksiyon | TX ID | Değer Aralığı |
|---|---|---|
| `setDriveMode` | 151 | 0=Kar, 1=Eco, 3=Normal, 4=Sport |
| `setRegenerativeLevel` | 180 | 0-4 (Kapalı, Düşük, Orta, Yüksek, Adaptif) |
| `setOnePedalConfigCode` | 181 | 0=Kapalı, 1=Açık |
| `setRegenerativeBrakeSwitch` | 182 | 0=Kapalı, 1=Açık (regen ana switch) |
| `setSteeringWheelHeat` | 52 | 0=Kapalı, 1=Açık (aircondition servisi) |

### Property ID'ler (referans)
- Sürüş Modu: `557883772`
- Regen Seviyesi: `557883793`
- Tek Pedal: `557883795`

---

## 📁 Proje Yapısı

```
DriveHubDort/
├── app/src/main/
│   ├── java/com/example/DriveHubDort/
│   │   ├── ui/
│   │   │   └── MainActivity.java          # Ana UI, buton yönetimi
│   │   ├── service/
│   │   │   ├── MG4ControlService.java     # Foreground service, hardkey receiver
│   │   │   └── BootReceiver.java          # Araç açılışında otomatik başlatma
│   │   ├── hardware/
│   │   │   └── MG4Hardware.java           # Binder haberleşme katmanı
│   │   └── model/
│   │       ├── DriveMode.java             # Enum: Eco, Normal, Sport, Kar
│   │       └── RegenLevel.java            # Enum: OFF, Low, Medium, High, Adaptive
│   ├── res/layout/
│   │   └── activity_main.xml              # 1920x720 optimize layout
│   └── AndroidManifest.xml
└── build.gradle
```

### Dosya Görevleri

#### `MainActivity.java`
- UI yönetimi (butonlar, durum gösterimi)
- Intent ile servise komut gönderme
- WRITE_SETTINGS izin kontrolü

#### `MG4ControlService.java`
- Foreground service (sistem tarafından öldürülmez)
- BroadcastReceiver ile hardkey yakalama (`com.saic.keyevent.hardkey.report`)
- Komut yönetimi (DRIVE_CYCLE, REGEN_CYCLE, vb.)
- Notification güncellemesi

#### `MG4Hardware.java`
- Binder haberleşme katmanı
- ServiceManager ile servis alma
- Parcel paketleme ve transaction gönderimi
- Hata yönetimi ve loglama

#### `BootReceiver.java`
- `BOOT_COMPLETED` broadcast'i yakalama
- Araç açılışında servisi otomatik başlatma

#### `DriveMode.java` / `RegenLevel.java`
- Enum sınıfları
- `next()` metodu ile döngüsel geçiş
- `fromValue()` metodu ile değer çevirme

---

## 🔧 Şu Ana Kadar Yapılanlar

### ✅ Tamamlananlar
1. **Gradle yapılandırması** — AGP 8.7.3, Gradle 8.9, Java 17
2. **Binder protokolü** — Area ID ve count parametreleri eklendi
3. **UI tasarımı** — 1920x720 landscape için 3 kolonlu layout
4. **Hardkey receiver** — `com.saic.keyevent.hardkey.report` için broadcast kayıt
5. **Regen OFF modu** — `setRegenerativeBrakeSwitch()` ile tamamen kapatma
6. **APK derleme** — Build başarılı, araca yüklendi
7. **Git kurulumu** — Nested git hatası çözüldü, ilk commit yapıldı
8. **APK analizi** — SAIC orijinal APK'ları decompile edildi, transaction ID'ler doğrulandı
9. **SELinux/servis erişim sorunu ÇÖZÜLDÜ** — `sharedUserId` + AOSP platform key imzası eklendi
10. **Platform key imzalı APK hazır** — `app-debug-platform-signed.apk` araca yüklenmeyi bekliyor

### 📝 README.md
Proje kök dizininde kapsamlı dokümantasyon mevcut:
- Binder protokol detayları
- Property ID tablosu
- Debug komutları
- ADB test prosedürü

---

## ✅ Çözülen Sorun: SELinux / Servis Erişimi

### Kök Neden (Tespit Edildi)
`vehiclesetting` servisi, SELinux politikaları gereği yalnızca `android.uid.system` grubundaki uygulamalara görünür. Normal user app olarak yüklenince `ServiceManager.getService("vehiclesetting")` → `null` dönüyordu.

### Çözüm
**2 değişiklik yeterliydi — root veya `/system/priv-app` GEREKMİYOR:**

1. `AndroidManifest.xml`'e `android:sharedUserId="android.uid.system"` eklendi

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [merthankaraman/DriveHub_Dort](https://github.com/merthankaraman/DriveHub_Dort) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
