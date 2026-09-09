---
trigger: always_on
description: Guidance for AI coding agents (Claude Code, etc.) working in this repository.
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, etc.) working in this repository.
`CLAUDE.md` points here; this file is the source of truth.

## About

NCANode — сервер (Spring Boot 3.5, Java 25, Gradle 9) для работы с ЭЦП Республики Казахстан: подпись и проверка XML (xmldsig), CMS, PDF, WSSE (SmartBridge), JWT, проверка сертификатов через OCSP/CRL, TSP-метки. REST API на JSON, порт по умолчанию 14579.

## Build & Run

Требуются проприетарные библиотеки KalkanCrypt (`knca_provider_jce_kalkan-*.jar`, `kalkancrypt-xmldsig-*.jar`) в директории `lib/` — Gradle подключает их через flatDir-репозиторий. Без них проект не соберётся; их можно запросить на https://pki.gov.kz/developers/.

```bash
./gradlew bootRun            # запуск без сборки
./gradlew bootJar            # сборка jar -> build/libs/NCANode.jar
./gradlew bootWar            # сборка war
```

Проверка после запуска: http://localhost:14579/actuator/health, Swagger UI: `/swagger-ui/`.

Вся конфигурация — через переменные окружения `NCANODE_*` (см. `src/main/resources/application.yml`): порт, URL-ы CRL/OCSP/TSP/CA, прокси, каталог кэша (`NCANODE_CACHE_DIR`, по умолчанию `./cache`), `NCANODE_DEBUG` для детальных ошибок.

## Tests

Тесты написаны на Spock (Groovy), лежат в `src/test/groovy`:

```bash
./gradlew test                                                    # все тесты
./gradlew test --tests 'kz.ncanode.unit.service.CmsServiceTest'   # один класс
./gradlew test --tests '*CmsServiceTest' --tests '*XmlServiceTest'
./gradlew check                                                   # тесты + jacoco-отчёт
```

- `unit/` — юнит-тесты сервисов и wrapper'ов.
- `integration/` — тесты контроллеров через standalone MockMvc; базовый класс `common/IntegrationSpecification.groovy` (метод `doPostQuery`), тестовые ключи/данные — в `common/WithTestData.groovy` и `src/test/resources` (ca, certs, crl, ocsp, tsp).

`check` зависит от `jacocoTestReport`; из покрытия исключены dto, configuration, constants, exception, util.

### Тесты не должны зависеть от текущего времени

Тестовые ключи и сертификаты имеют срок действия, а CRL/OCSP-ответы — окно валидности. Тест, который проверяет подпись/цепочку «на `new Date()`», начнёт падать, как только сертификат протухнет или CRL устареет.

- Валидацию по времени прогонять на **фиксированную дату внутри срока действия** тестового сертификата, а не на «сейчас». Пробрасывать время параметром / через `Clock`, не звать `new Date()` / `Instant.now()` в проверяемом коде без возможности подмены.
- Для проверки существующих подписей-эталонов (`xades-test-signed-*.xml` и т.п.) фиксировать validation time на момент подписания из самого файла (`SigningTime`) либо на заведомо валидную дату.
- Ассерты вида «подпись валидна сегодня» — запрещены; только «подпись валидна на дату T».

### Тестовые ключи

GOST-2015 (действующий на 2026–2027):
`/Users/malikzh/Downloads/SDK/SDK/SDK/SDK 2.0/Keys and Certs/Gost2015/2026.05.08-2027.05.07/Физическое лицо/valid/GOST512_ec425659bd2fc6dc587b871aede1857727cf8451.p12`
пароль `Qwerty12`. Это тот ключ, которым созданы эталоны `xades-test-signed-*.xml` (субъект `ТЕСТОВ ТЕСТ`, IIN123456789011, УЦ `... (GOST) TEST 2022`, CDP `http://test.pki.gov.kz/crl/nca_gost2022_test.crl`).

Прочие тестовые ключи (Base64 PKCS12) и их пароли/алиасы — в `WithTestData.groovy` (`KEY_INDIVIDUAL_VALID_2015` и др.).

## XAdES / CAdES / PAdES (LT / LTA) — work in progress

Задача: выдавать не голый xmldsig/CMS-with-TSP, а профили ETSI **B / T / LT / LTA** (клиентский запрос про «профили A, B, LT, LTA и XAdES»).

**Статус реализации.** Общий enum уровня — `dto/ades/AdesLevel { B, T, LT, LTA }`.
Сбор LT-материала — `CertificateService.collectAdesValidationData(signer, extraCerts)`
→ `dto/ades/AdesValidationData` (цепочка через `CaService.buildChain`, OCSP-в-DER через
`OcspService.getRawResponses`, CRL-в-DER через `CrlService.getEncodedCrlsFor`, сертификаты TSA
через `TspService.extractCertificates`). Формат-нейтрально. Нет OCSP/CRL для цепочки → `ClientException`.

- **XAdES B/T/LT/LTA** — `/xml/sign`, `XmlSignRequest.xadesLevel` (`null` = обычный XMLDSIG) + `tsaPolicy`.
  Оркестрация `XmlService.signXades`, формат — `wrapper/XadesSignatureWrapper` (Santuario + Kalkan).
- **CAdES B/T/LT/LTA** — `/cms/sign`, `CmsCreateRequest.cadesLevel` (`null` = обычный CMS + флаг `withTsp`).
  На `/cms/sign/add` (co-sign) — не поддержано, `ClientException`.
  Оркестрация `CmsService.applyCadesLevel`, формат — `wrapper/CadesSignatureWrapper`:
  - B — signed-атрибут `id-aa-signingCertificateV2` (ESSCertIDv2, SHA-256) + `signingTime`
    (в `CmsService.cadesSignedAttributes`, через 5-арг `addSigner`).
  - T — `TspService.addTspToSigner` (`id-aa-signatureTimeStampToken`).
  - LT — цепочка в `SignedData.certificates` (`replaceCertificatesAndCRLs`), отзыв в `SignedData.crls`
    (`CertificateList` / `[1] OtherRevocationInfoFormat` c `id-ri-ocsp-response`).
  - LTA — `id-aa-ets-archiveTimestampV3` (`0.4.0.1733.2.4`) + `id-aa-ATSHashIndex-v3` (`0.4.0.19122.1.5`)
    внутри токена; имприт по ETSI EN 319 122-1 §5.5.3 (порядок 1:1 с движком NCALayer).
    ponytail: для detached-CMS архивный имприт хэширует пустое содержимое (как и движок NCALayer).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ncanode-kz/NCANode](https://github.com/ncanode-kz/NCANode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
