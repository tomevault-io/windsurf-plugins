---
trigger: always_on
description: Fredbet is a Spring Boot football betting web application.
---

# Copilot Instructions – Fredbet

Fredbet is a Spring Boot football betting web application.
- **Base package**: `de.fred4jupiter.fredbet`
- **Java version**: 25
- **Spring Boot version**: 4.0.4
- **Build tool**: Maven

---

## Package Structure

The project follows a **feature-based (domain-driven) package layout**:

```
de.fred4jupiter.fredbet
├── admin/              # Administration services
├── betting/            # Betting domain logic + repositories
│   └── repository/     # Repositories specific to betting
├── calendar/           # iCal export
├── config/             # Spring @Configuration classes
├── crests/             # Team crest images
├── data/               # Test data generation helpers
├── domain/             # Shared domain types
│   ├── entity/         # JPA entities
│   └── builder/        # Builder classes for entities
├── event/              # Login/session event handlers
├── excel/              # Excel import/export
├── image/              # Image handling
├── imexport/           # Data import/export
├── info/               # Info pages
├── integration/        # External API integration (football-data.org)
├── match/              # Match domain logic + repository
├── pdf/                # PDF export
├── pointcourse/        # Point-course logic
├── points/             # Points calculation
├── props/              # Configuration properties, constants, profiles
├── ranking/            # Ranking calculation
├── registration/       # Self-registration feature
├── security/           # Spring Security setup, permissions, user groups
├── settings/           # Runtime settings
├── standings/          # League standings
├── statistic/          # Statistics
├── team/               # Team management
├── teambundle/         # Team bundle management
├── user/               # User management
├── util/               # Shared utilities
└── web/                # All @Controller classes, Commands, Mappers
    └── <feature>/      # Web layer per feature (matches, bet, user, …)
```

---

## Naming Conventions

### Classes

| Type | Suffix / Rule | Annotation |
|---|---|---|
| Service | `*Service` | `@Service` |
| Repository (JPA) | `*Repository` | extends `JpaRepository` |
| Custom repo interface | `*RepositoryCustom` | – (package-private interface) |
| Custom repo implementation | `*RepositoryImpl` | – |
| Controller | `*Controller` | `@Controller` |
| Configuration | `*Config` | `@Configuration` |
| Component / Mapper | descriptive name | `@Component` |
| Command (web DTO) | `*Command` | – |
| Command Mapper | `*CommandMapper` | `@Component` |
| Domain Event | `*Event` (record) | – |
| Builder | `*Builder` | – |
| Exception | descriptive + `Exception` | – |
| Utility / Constants | `*Constants`, `*Names`, `*Profile` | `final` class, private constructor |
| Properties record | `*Properties` | `@ConfigurationProperties` |
| Permission interface | `FredBetPermission` | `interface` with `PERM_*` string constants |
| User group enum | `FredBetUserGroup` | – |

> **Interfaces must NOT have names containing the word "Interface"** (enforced by ArchUnit).

### Fields

- Constants: `UPPER_SNAKE_CASE` (`public static final` or `private static final`)
- Instance fields: `camelCase`
- Logger: always `private static final Logger LOG = LoggerFactory.getLogger(ClassName.class);`

### Database

- Table names: `UPPER_CASE` (e.g., `MATCHES`, `APP_USER`)
- Column names: `UPPER_CASE` with underscores (e.g., `MATCH_ID`, `KICK_OFF_DATE`)

---

## Architectural Rules (enforced by ArchUnit)

These rules are **hard constraints** verified by tests in `src/test/java/.../architecture/`:

1. **Controllers** must NOT directly access Repositories.
2. **Services and Components** must NOT access Controllers.
3. **Repositories** must NOT access Services.
4. Only **Repositories** may use `EntityManager`.
5. **No field injection** – always use **constructor injection**.
6. **No `System.out` / `System.err`** – use SLF4J.
7. **No generic exceptions** (`RuntimeException`, `Exception`) – throw specific exceptions.
8. **No JodaTime** – use `java.time`.
9. **No `java.util.logging`** – use SLF4J.
10. Logger fields must be `private static final`.
11. Interface names must NOT end with or contain `Interface`.
12. Classes whose simple name ends with `Config` must be annotated with `@Configuration`.

---

## Dependency Injection

- Always use **constructor injection** (never field injection with `@Autowired`).
- Dependencies are declared as `private final` fields and injected via the constructor.

```java
@Service
public class BettingService {

    private static final Logger LOG = LoggerFactory.getLogger(BettingService.class);

    private final BetRepository betRepository;
    private final MatchService matchService;

    public BettingService(BetRepository betRepository, MatchService matchService) {
        this.betRepository = betRepository;
        this.matchService = matchService;
    }
}
```

---

## Entity Conventions

- JPA entities live in `domain/entity/`.
- **No Lombok** – write explicit getters, setters, `equals`, `hashCode`, `toString`.
- Use `EqualsBuilder`, `HashCodeBuilder`, `ToStringBuilder` from **Apache Commons Lang 3**.
- Use `GenerationType.IDENTITY` for ID generation.
- Enum columns use `@Enumerated(EnumType.STRING)`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fred4jupiter/fredbet](https://github.com/fred4jupiter/fredbet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
