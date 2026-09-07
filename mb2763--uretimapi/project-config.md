---
trigger: always_on
description: `UretimV4` (CepPatronERP masaustu uygulamasi) ve `WebUretim TabletV2` (tablet/web istemcisi) icin **ortak REST API**'sini saglar. Mikro ERP veritabani ile sirket ic Uretim veritabani arasinda dogrudan SQL erisimiyle (Dapper) calisir; siparis aktarim, stok takibi, uretim hareketleri ve kullanici/login islemleri sunar.
---

# CLAUDE.md - WebUretim ApiFeza V1 (Backend API)

## Proje Ozeti

`UretimV4` (CepPatronERP masaustu uygulamasi) ve `WebUretim TabletV2` (tablet/web istemcisi) icin **ortak REST API**'sini saglar. Mikro ERP veritabani ile sirket ic Uretim veritabani arasinda dogrudan SQL erisimiyle (Dapper) calisir; siparis aktarim, stok takibi, uretim hareketleri ve kullanici/login islemleri sunar.

ASP.NET Core 6.0 Web API + JWT authentication + Swagger UI ile sunulur. Windows Service olarak da host edilebilir (`UseWindowsService`).

## Iliskili Projeler

Bu API, su iki projeyle birlikte calisir (ayni veritabanlarini paylasir):

| Proje | Rol | Kullanim |
|-------|-----|----------|
| **UretimV4** (`D:\Projelerimmm\UretimV4`) | WinForms masaustu ERP (CepPatronERP.exe) | Ofis kullanicilari icin tam ozellikli ERP arayuzu — bu API'yi **kullanmaz**, DB'ye dogrudan baglanir |
| **WebUretim ApiFeza V1** *(bu proje)* | Backend API (MyApi.dll) | Tablet ve diger web istemcilerinin DB'ye erismek icin kullandigi REST API |
| **WebUretim TabletV2** (`D:\Projelerimmm\WebUretim TabletV2`) | Tablet/web istemcisi | Bu API uzerinden HTTP istekleriyle uretim takibi yapan saha uygulamasi |

Ortak veritabanlari (paylasilan):
- **Uretim DB** (ProConn): `UretimV3_FEZA` - uretim, recete, istasyon, kullanici
- **Mikro DB** (MikroConn): `MikroDB_V16_FEZA24` - Mikro ERP stok, siparis, cari

## Teknoloji Yigini

- **Framework**: .NET 6.0 (ASP.NET Core)
- **API**: ASP.NET Core MVC + Controllers
- **Auth**: JWT Bearer (`Microsoft.AspNetCore.Authentication.JwtBearer 6.0.14`)
- **ORM**: Dapper 2.0.123 (manuel SQL + entity mapping)
- **SQL Client**: Microsoft.Data.SqlClient 5.1.1 + System.Data.SqlClient 4.8.5
- **Dokuman**: Swashbuckle.AspNetCore 6.2.3 (Swagger UI)
- **JSON**: Newtonsoft.Json 13.0.3 (MVC serileştirici)
- **Logging**: Serilog.Sinks.File 5.0.0
- **Hosting**: Microsoft.Extensions.Hosting.WindowsServices 7.0.0 (Windows Service host)

## Cozum Yapisi

```
UretimApi.sln
├── MyApi/                       # Web API (ASP.NET Core 6) → bin\Debug\net6.0\MyApi.dll, MyApi.exe
│   ├── Controllers/             # GenelController, StokTakipController
│   ├── Extentions/              # MyAuthorizationFilter (JWT yetki filtresi)
│   ├── FisKayitLog/             # Fis kayit loglari
│   ├── Properties/              # launchSettings.json (port: 7098/5094, IIS Express: 17378/44383)
│   ├── Program.cs               # Service registration, JWT config, Swagger
│   ├── ZipManager.cs
│   └── appsettings.json         # ConnectionStrings, TokenOptions, LoginUser, Kestrel
└── My/                          # Class Library (.NET 6)
    ├── Business/
    │   ├── Auth/                # MyAuthenticationService (kullanici login, token uretimi)
    │   ├── Geneller/            # GenelService (BaglantiTest)
    │   ├── Managers/            # MikroAktarimManager, MikroConvertManager (siparis aktarim, Mikro ↔ Uretim donusum)
    │   ├── Mikro/               # MikroService (Mikro ERP'den stok/siparis sorgulari)
    │   ├── StokTakipler/        # UretimStokTakipService, SiparisService
    │   ├── Uretimler/           # Uretim emir/hareket service'leri
    │   ├── Users/               # UserService
    │   └── Cariler/             # (csproj'da derlemeden cikarilmis - eski)
    ├── Core/                    # MyGuid, MyLogger, MyResult (sonuc tipleri)
    ├── DataAccess/
    │   ├── Data/                # MyDbContext (ProConn), MyDbContextMikro (MikroConn) - Dapper baglantilar
    │   └── Security/            # TokenHandler, TokenOptions, AccessToken, SingHandler, LoginModel
    └── Entities/                # POCO modeller (Ayarlar, Kullanicilar, Mikro, Siparisler, StokTakipler, UretimEmirleri, Templer)
```

## Derleme ve Calistirma

### Derleme
```powershell
cd "D:\Projelerimmm\WebUretim ApiFeza V1"
dotnet build UretimApi.sln
```

Cikti: `MyApi\bin\Debug\net6.0\MyApi.dll` ve `MyApi.exe` (Windows Service icin).
**Bilinen durum**: Build 0 hata + ~306 uyari (nullable reference warnings). Calismaya engel degil.

### Calistirma (Development)
```powershell
cd "D:\Projelerimmm\WebUretim ApiFeza V1\MyApi"
dotnet run
```
veya
```powershell
& "D:\Projelerimmm\WebUretim ApiFeza V1\MyApi\bin\Debug\net6.0\MyApi.exe"
```

Varsayilan portlar (launchSettings.json):
- HTTPS: `https://localhost:7098`
- HTTP: `http://localhost:5094`
- Swagger UI: kok adreste (RoutePrefix boş) ve `/swagger`

`appsettings.json` icindeki Kestrel ayari `http://*:8199` portunu dinler (Production icin).

### Windows Service Olarak
`Program.cs:51` icinde `builder.Host.UseWindowsService()` aktif. Service kurulumu:
```powershell
sc.exe create MyApiUretim binPath="D:\Projelerimmm\WebUretim ApiFeza V1\MyApi\bin\Release\net6.0\MyApi.exe"
sc.exe start MyApiUretim
```

### Publish
```powershell
dotnet publish UretimApi.sln -c Release -o D:\WebUretimApi_Publish
```

## Konfigurasyon (appsettings.json)

```json
{
  "ConnectionStrings": {
    "ProConn":   "Server=192.168.3.201;Database=UretimV3_FEZA;User Id=ceppatron;Password=1122334416;TrustServerCertificate=True;",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mb2763/uretimApi](https://github.com/mb2763/uretimApi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
