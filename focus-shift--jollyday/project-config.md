---
trigger: always_on
description: A Java library for querying public holidays supporting 230+ countries with XML-based holiday calendar configuration
---


# Jollyday - Public Holiday Library

## Project Overview

**Jollyday** is a Java library to query public holidays. It currently supports **over 230 countries**. The holiday data is stored in XML files (one for each country) and will be read from the classpath.

### Key Features

- Supports 230+ countries with public holiday data
- ISO 3166-1 alpha-2 country code support
- ISO 3166-2 subdivision support (states, provinces, regions)
- Multiple holiday types: Fixed dates, moving holidays (Easter-based, Islamic calendar, etc.)
- XML-based configuration with validation via XSD
- Supports JAXB and Jackson for XML unmarshalling
- Java 17+ required

### Modules

| Module             | Description                                    |
|--------------------|------------------------------------------------|
| `jollyday-core`    | Core API, holiday data, parser implementations |
| `jollyday-jackson` | Jackson XML binding implementation             |
| `jollyday-jaxb`    | Jakarta XML Binding implementation             |
| `jollyday-tests`   | Integration and country-specific tests         |

## Architecture

```
jollyday/
├── jollyday-core/
│   ├── src/main/java/de/focus_shift/jollyday/core/
│   │   ├── Holiday.java                 # Holiday value object
│   │   ├── HolidayManager.java          # Main API entry point
│   │   ├── HolidayCalendar.java         # Enum of all supported countries
│   │   ├── ManagerParameters.java       # Configuration builder
│   │   ├── impl/                        # Manager implementations
│   │   ├── parser/                      # Holiday parsing API
│   │   │   └── impl/                    # Parser implementations
│   │   └── spi/                         # Holiday configuration interfaces
│   └── src/main/resources/
│       ├── holidays/                    # Country holiday XML files
│       ├── descriptions/                # Localized holiday descriptions
│       ├── focus_shift.de/jollyday/     # XSD schema definition
│       └── jollyday.properties          # Default configuration
├── jollyday-jackson/                    # Jackson XML unmarshalling
├── jollyday-jaxb/                       # JAXB XML unmarshalling
└── jollyday-tests/                      # Test suite
```

## Key Concepts

### Holiday Types

| Type                         | Description                          | Examples                                        |
|------------------------------|--------------------------------------|-------------------------------------------------|
| **Fixed**                    | Same calendar date every year        | New Year (Jan 1), Christmas (Dec 25)            |
| **FixedWeekday**             | Specific weekday occurrence in month | Labor Day (first Monday in Sept)                |
| **ChristianHoliday**         | Easter-based moving holidays         | Good Friday, Easter Monday, Ascension           |
| **IslamicHoliday**           | Islamic calendar holidays            | Eid al-Fitr, Eid al-Adha                        |
| **EthiopianOrthodoxHoliday** | Ethiopian Orthodox holidays          | Timkat, Meskel                                  |
| **RelativeToFixed**          | Relative to a fixed date             | German Repentance Day (Wednesday before Nov 23) |
| **RelativeToWeekdayInMonth** | Relative to a moving holiday         | Various country-specific holidays               |

### Holiday Categories

| Category           | Description                                 |
|--------------------|---------------------------------------------|
| **Public Holiday** | Official non-working days (paid time off)   |
| **Bank Holiday**   | Days when financial institutions are closed |
| **Observance**     | Celebrations without official time off      |

### ISO Standards

- **ISO 3166-1 alpha-2**: Country codes (e.g., `DE`, `US`, `FR`)
- **ISO 3166-2**: Subdivision codes (e.g., `DE-BY` for Bavaria, Germany)

## XML Holiday Configuration

### Root Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>

<Configuration hierarchy="[country_code]" description="[Country Name]"
               xmlns="https://focus_shift.de/jollyday/schema/holiday"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="https://focus_shift.de/jollyday/schema/holiday https://focus_shift.de/jollyday/schema/holiday/holiday.xsd">
  <Holidays>
    <!-- National holidays -->
  </Holidays>

  <Sources>
    <Source>https://en.wikipedia.org/wiki/Public_holidays_in_[Country]</Source>
    <Source of="ISO 3166">https://www.iso.org/obp/ui/#iso:code:3166:[CODE]</Source>
  </Sources>
</Configuration>
```

### Holiday Elements

#### Fixed Date
```xml
<Fixed month="JANUARY" day="1" descriptionPropertiesKey="NEW_YEAR"/>
<Fixed month="DECEMBER" day="25" validFrom="1990" descriptionPropertiesKey="CHRISTMAS"/>
```

#### Fixed Weekday in Month
```xml
<FixedWeekday which="FIRST" weekday="MONDAY" month="SEPTEMBER" descriptionPropertiesKey="LABOUR_DAY"/>
<FixedWeekday which="LAST" weekday="MONDAY" month="MAY" descriptionPropertiesKey="MEMORIAL_DAY"/>
```
- `which`: FIRST, SECOND, THIRD, FOURTH, LAST

#### Christian (Easter-based)
```xml
<ChristianHoliday type="GOOD_FRIDAY"/>
<ChristianHoliday type="EASTER_MONDAY"/>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [focus-shift/jollyday](https://github.com/focus-shift/jollyday) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
