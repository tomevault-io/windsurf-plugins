---
trigger: always_on
description: Liman, sunucuları, istemcileri ve ağ cihazlarını merkezi olarak yönetmek için açık kaynaklı bir platform. HAVELSAN tarafından geliştirilen bu sistem, genişletilebilir eklenti mimarisi ile güvenli sunucu yönetimi sağlar.
---

# Liman MYS (Merkezi Yönetim Sistemi) - GitHub Copilot Instructions

## Proje Genel Bakış

Liman, sunucuları, istemcileri ve ağ cihazlarını merkezi olarak yönetmek için açık kaynaklı bir platform. HAVELSAN tarafından geliştirilen bu sistem, genişletilebilir eklenti mimarisi ile güvenli sunucu yönetimi sağlar.

## Teknoloji Stack

### Backend
- **Framework**: Laravel 12 (PHP 8.4)
- **Veritabanı**: PostgreSQL
- **Auth**: JWT (php-open-source-saver/jwt-auth)
- **Cache**: Redis
- **Queue**: Laravel Queue
- **WebSocket**: Laravel Reverb

### Frontend
- External bir NextJS uygulaması kullanılıyor. Bu sistem sadece API olarak hizmet vermekte.

### Deployment
- **Container**: Docker (Ubuntu Jammy)
- **Web Server**: Nginx
- **PHP**: PHP-FPM 8.4
- **Process Manager**: Supervisor

## Proje Yapısı

```
/liman/server/
├── app/
│   ├── Classes/              # Özel sınıflar
│   │   ├── Authentication/   # Auth adaptörleri (Liman, Keycloak, LDAP, OIDC)
│   │   ├── Ldap.php         # LDAP bağlantı sınıfı
│   │   └── NotificationBuilder.php
│   ├── Connectors/          # Sunucu bağlantı adaptörleri
│   │   ├── Connector.php
│   │   └── GenericConnector.php
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── API/         # REST API endpoints
│   │   │   ├── Extension/   # Eklenti sandbox
│   │   │   └── HASync/      # Yüksek erişilebilirlik sistemi yardımcı endpointleri
│   │   ├── Middleware/      # Auth, permission, server middlewares
│   │   └── Helpers.php      # Global helper fonksiyonlar
│   ├── Models/              # Eloquent modeller
│   ├── System/              # Sistem seviye sınıflar
│   └── ...
├── config/                  # Laravel config dosyaları
├── database/
│   ├── migrations/          # Veritabanı migrasyonları
│   └── seeds/               # Seed dosyaları
├── routes/
│   ├── api.php             # API rotaları
│   ├── web.php             # Web rotaları
│   └── ...
├── storage/                 # Dosya depolama
└── resources/
    ├── views/              # Blade templates
    └── assets/             # Frontend assets
```

## Önemli Özellikler

### 1. Multi-Authentication System
- **Liman Auth**: Yerli kullanıcı sistemi
- **Keycloak**: OAuth2/OIDC entegrasyonu
- **LDAP**: Active Directory/OpenLDAP desteği
- **OIDC**: Generic OIDC provider desteği

### 2. Sunucu Yönetimi
- SSH/WinRM üzerinden uzak komut çalıştırma
- Dosya transferi ve yönetimi
- Gerçek zamanlı sistem monitörü
- Kubernetes cluster yönetimi

### 3. Eklenti Sistemi
- Sandbox ortamında eklenti çalıştırma
- PHP/JavaScript eklenti desteği
- Rol tabanlı eklenti yetkilendirme

### 4. Güvenlik
- JWT token authentication
- Role-based access control (RBAC)
- Multi-factor authentication (2FA)
- SSL/TLS certificate management
- IP range restrictions

### 5. Monitoring & Logging
- Audit logging
- Authentication logs
- Performance monitoring
- Real-time notifications

## Veri Modelleri

### Ana Modeller
- **User**: Kullanıcı yönetimi (local, ldap, keycloak, oidc auth types)
- **Server**: Sunucu bilgileri ve bağlantı detayları
- **Extension**: Eklenti meta verileri ve lisans bilgileri
- **Role**: Rol tanımları ve yetkilendirme
- **Permission**: Granular izin sistemi
- **Certificate**: SSL sertifika yönetimi
- **Notification**: Bildirim modeli

### Auth Modelleri
- **AccessToken**: API token yönetimi
- **AuthLog**: Giriş logları
- **Oauth2Token**: OAuth2 token storage
- **AuditLog**: Sistem denetim logları

## API Endpoints

### Authentication (`/api/auth/`)
- `POST /login`: Kullanıcı girişi (multi-auth support)
- `POST /logout`: Çıkış
- `POST /refresh`: Token yenileme
- `GET /user`: Kullanıcı profili
- `POST /change_password`: Şifre değiştirme
- `POST /setup_mfa`: 2FA kurulumu

### Server Management (`/api/servers/`)
- `GET /`: Sunucu listesi
- `POST /`: Yeni sunucu ekleme
- `GET /{id}`: Sunucu detayları
- `GET /{id}/specs`: Sunucu özellikleri
- `POST /{id}/users/local`: Yerel kullanıcı ekleme

### Extension Management (`/api/extensions/`)
- `GET /`: Eklenti listesi
- `POST /assign`: Sunucuya eklenti atama
- `POST /unassign`: Eklenti atama kaldırma

### Settings (`/api/settings/`)
- `GET|POST /users/`: Kullanıcı yönetimi
- `GET|POST /roles/`: Rol yönetimi
- `GET|POST /extensions/`: Eklenti yönetimi
- `GET|POST /certificates/`: Sertifika yönetimi

## Önemli Sınıflar ve Fonksiyonlar

### Helper Functions (`app/Http/Helpers.php`)
```php
validate($rules, $messages = [], $fieldNames = []): void
user(): User                    // Authenticated user
server(): Server               // Current server context
extension(): Extension         // Current extension context
sudo(): string                 // Sudo command prefix
```

### Authentication Classes
- `Authenticator`: Token management
- `KeycloakAuthenticator`: Keycloak entegrasyonu
- `LDAPAuthenticator`: LDAP entegrasyonu
- `LimanAuthenticator`: Yerli auth sistem
- `OIDCAuthenticator`: Generic OIDC provider integration

### Server Connection
- `GenericConnector`: SSH/WinRM bağlantı yönetimi
- `Command`: Güvenli komut çalıştırma wrapper

## Güvenlik Considerations

### 1. Command Injection Prevention

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [limanmys/core](https://github.com/limanmys/core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
