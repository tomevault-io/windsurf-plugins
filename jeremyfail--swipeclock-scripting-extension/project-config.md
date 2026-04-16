---
trigger: always_on
description: Swipeclock scripting is a custom scripting language used in Swipeclock's TimeWorksPlus and WorkforceHub software products. This language is case-insensitive and has unique syntax rules that differ significantly from standard programming languages.
---

# Cursor Rules for Swipeclock Scripting Language

## Language Overview
Swipeclock scripting is a custom scripting language used in Swipeclock's TimeWorksPlus and WorkforceHub software products. This language is case-insensitive and has unique syntax rules that differ significantly from standard programming languages.

## Critical Language Rules

### Case Sensitivity
- **ALL keywords, operators, functions, and object names are CASE-INSENSITIVE**
- `employee.firstname` is the same as `EMPLOYEE.FIRSTNAME` or `Employee.FirstName`
- `if`, `IF`, and `If` are all valid
- `and`, `AND`, and `And` are all valid
- `contains`, `CONTAINS`, and `Contains` are all valid

### Object-Based Properties
- Properties MUST be accessed through objects: `employee.propertyName` or `reportingdate.propertyName`
- **NEVER use properties without the object prefix** (e.g., `firstname` is INVALID, must use `employee.firstname`)
- `date` is INVALID - must use `reportingdate.date`
- `firstname` is INVALID - must use `employee.firstname`

### Employee Object Properties
Valid properties for `employee.*`:
- Basic info: `yearsofservice`, `daysofservice`, `monthsofservice`, `anniversary`, `monthiversary`, `title`, `firstname`, `lastname`, `code` (or `employeecode`), `designation`, `startdate`, `enddate`
- Organization: `department` (default), `department1`, `department2`, etc. (if enabled), `location`, `location1`, `location2`, etc. (if enabled), `supervisor`
- Contact: `home1`, `home2`, `home3`, `home4`, etc. (if enabled)
- Time settings: `autolunchhours`, `lunchminutes`
- Accrual: `accrualfactor`, `employeetype`, `accrualvalidator` (when BasicAccruals rule is activated)
- Pay: `payrate0` (default), `payrate1`, `payrate2`, `payrate3`, etc. (if enabled)

### ReportingDate Object Properties
Valid properties for `reportingdate.*`:
- Date info: `tomorrow`, `yesterday`, `date`, `year`, `month`, `weekday` (returns MTWRFSU), `todaysdate`, `isfirstdayofmonth`, `islastdayofmonth`, `payperiodstart`, `payperiodend`, `isholiday`
- Time calculations: `spread` (timespan), `totalhours`, `totalhoursot`, `hourstodate`, `hourstodateot`, `weekhours`, `pphours`, `islastpunchpp`, `islastpunchweek`
- Functions with arguments: `totalweek("<promptFieldName>")`, `totalpp("<promptFieldName>")`, `totalday("<promptFieldName>")`, `depthours("<FieldName>", "<Value>")`
- Schedule: `punchsets`, `schedhours(<punchset>)`, `schedlines`, `schedin(<punchset>)`, `schedout(<punchset>)`, `schedplace(<punchset>)`, `workweekend`
- Other: `average` (undocumented but valid)

### Global Functions (NOT object-based)
These functions are global and do NOT use object notation:
- **Date functions**: `dateadd("mm", 6, employee.startdate)`, `dateserial(year, month, day)`, `weekday(date)`, `cdate(string)`, `cdatetime(string)`, `ctime(datetime)`, `day(date)`, `month(date)`, `year(date)`
- **Number functions**: `val(string)` (returns 0 for null), `cint(value)` (throws error for null), `cstr(value)`, `abs(number)`
- **String functions**: `translate(field, list1, list2)`, `within(field, list)`, `left(string, count)`, `right(string, count)`, `mid(string, start, count)`
- **Time rounding**: `round(timeString)` or `round(number)`, `roundin(timeString)`, `roundout(timeString)`, `roundends(timeString)`, `roundtoschedule(a1, a2, b1, b2)`
- **Math rounding**: `roundup(number)`, `rounddown(number)` (NOT for time rounding)
- **Additional**: `addalert(message)`, `unpay(hours)`, `touches(startTime, endTime)`, `isedited(property)`, `tomorrow(0)`, `yesterday(0)`, `overlaps(startTime, endTime)`, `overlap(start1, end1, start2, end2)`, `addentry("type", amount, "category")`, `otrules("OT Rule Name")`
- **Accrual**: `accrueup("Bucket", amount, max, vestDate, expDate)`, `accruedown("Bucket", amount, min)`, `getbalance("Bucket")`, `setbalance("Bucket", value)`

### Global Timecard Properties (NOT object-based)
These properties are global and do NOT use object notation:
- `payrate`, `isfirsttoday`, `islasttoday`, `hours`, `minutes`, `seconds`, `breakseconds`, `minutesout` or `minutesout(false)`, `minutestil`, `punchset`, `category`, `otcategory`, `punchdate`, `intime`, `outtime`, `indt`, `outdt`, `inismissing`, `outismissing`, `inispresent`, `outispresent`, `istimes`, `ishours`, `ispayonly`, `inisedited`, `outisedited`, `isedited`, `hourstopunch`, `hourstopunchot`, `linetonow`, `inip`, `outip`

### Variables
- **Local variables**: Start with `$` (e.g., `$myvar = 10`)
- **Global variables**: No prefix, just name it (e.g., `myvar = 10`)
- Variables are case-insensitive

### Operators (NOT functions - no parentheses)
- Logical: `and`, `or`, `&&`, `||`
- Comparison: `==`, `!=`, `<>`, `>`, `>=`, `<`, `<=`
- Arithmetic: `+`, `-`, `*`, `/`, `\` (integer division), `%`, `mod`
- String: `contains`, `startswith`, `endswith` (these are operators, NOT functions)
- Assignment: `=`, `==` (both work for equality, `==` preferred)

### Syntax Rules
- **If statements**: Use curly braces `{}` for code blocks
- **Case insensitive**: Everything is case-insensitive
- **Spacing**: Braces and parentheses can have spaces or not

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JeremyFail) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
