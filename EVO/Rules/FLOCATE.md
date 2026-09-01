# EVO — FLOCATE Rules

**Product:** EVO
**Target Table:** `FLOCATE`
**Schema File:** `/EVO/Schema/FLOCATE.json`
**Source Workbook:** MRI Evolution Data Collection Sheet (`4/492 Iss 7`, v5.3.0)
**Source Worksheet:** `Buildings`

These rules are taken from the MRI Evolution Data Collection Sheet. They describe how the
intake (collection sheet) columns must be populated before the file can be integrated into
the EVO `FLOCATE` table.

They are **additional to** the structural rules in `/EVO/Schema/FLOCATE.json`.
Where the two disagree on data type or length, the JSON schema remains the source of
truth for the physical database, and the rules below define the business/intake expectation.

> `FLOCATE` is the **Buildings** table in Concept Evolution. The worksheet note reads:
> *"Locations within Concept Evolution are grouped together using logical grouping.
> Buildings are grouped on Sites, Locations are grouped within Buildings."*
>
> The worksheet tab is **RED — compulsory**.

---

## Field Rules

| # | Intake Column (Worksheet) | Target Column (`FLOCATE`) | Type | Length | Mandatory | Rule |
| -: | ------------------------- | ------------------------- | ---- | -----: | --------- | ---- |
| 1 | Complex / Site | `BG_FKEY_BGP_SEQ` *(inferred — site/building-group FK)* | Select from Drop Down | — | Recommended (BLUE) | Select from the drop-down only. Must match a `Name` on the `Sites` worksheet. Groups the building under a site. |
| 2 | Code | `BG_CODE` | Text (A) | 20 | **YES (RED)** | The building code. Must be unique across the worksheet. |
| 3 | Building Name | `BG_SITE` | Text (A) | 40 | **YES (RED)** | The building's display name. |
| 4 | Country | `BG_COUNTRY` / `CountryLibraryId` | Select from Drop Down | — | **YES (RED)** | Select from the drop-down only. Must match a value on the `Countries` worksheet. |
| 5 | Building Type | `BG_BUILDING_TYPE` / `BuildingTypeId` | Select from Drop Down | — | No (Client's discretion) | Select from the drop-down only. Must match a `Name` on the `Building Type` worksheet. |
| 6 | Building Address | `BG_ADDRESS` | Text (A) | 300 | No (Client's discretion) | Free-text address. May contain embedded line breaks. |
| 7 | Building Postcode | `BG_POSTCODE` | Text (A) | 15 | No (Client's discretion) | — |
| 8 | Time Zone | `TimeZoneId` | Select from Drop Down | — | **YES (RED)** | Select from the drop-down only. Must match a `Code` on the `Time Zone` worksheet (e.g. `GMT`, `UTC`). |
| 9 | Latiitude | `Latitude` | Number (Dec) | 15 | No (Client's discretion) | Decimal degrees. Worksheet heading is misspelled — see **Source Notes**. |
| 10 | Longitude | `Longitude` | Number (Dec) | 15 | No (Client's discretion) | Decimal degrees. |
| 11 | Elevation | `Elevation` | Number (Dec) | 15 | No (Client's discretion) | — |
| 12 | GIS Reference | `GISReference` | Text (A) | 255 | No (Client's discretion) | Worksheet row 4 reads `Test` — a typo for `Text`. See **Source Notes**. |
| 13 | External System | `ExtSystem` | Text (A) | 100 | No (Client's discretion) | Name of the third-party system this building originates from. |
| 14 | External Object | `ExtObject` | Text (A) | 100 | No (Client's discretion) | Object type in that system. |
| 15 | External Identifier | `ExtIdentifier` | Text (A) | 100 | No (Client's discretion) | Identifier in that system. |
| 16 | SIte - Building | *(none — not imported)* | Auto Generated — **DO NOT TOUCH** | 100 | Auto | Worksheet formula producing `{Complex / Site} - {Building Name}`. Consumed as the drop-down source by the `BuildingFloors` and `Locations` worksheets. |

### Column Groupings (as presented on the worksheet)

* **Compulsory (RED):** Code, Building Name, Country, Time Zone
* **Recommended (BLUE):** Complex / Site
* **Client's discretion (BLACK):** Building Type, Building Address, Building Postcode, Latiitude, Longitude, Elevation, GIS Reference, External System, External Object, External Identifier
* **Auto generated (DO NOT TOUCH):** SIte - Building

---

## Validation Rules

### Required Values

Report an **Error** for any row where the following are blank:

* Code
* Building Name
* Country
* Time Zone

Report a **Warning** where `Complex / Site` is blank — it is recommended, not compulsory, but
a blank value produces an unusable `SIte - Building` key (` - {Building Name}`) which the
`BuildingFloors` and `Locations` worksheets depend on.

All other columns are at the client's discretion and may be left blank.

### Maximum Length

Report an **Error** where the supplied value exceeds the stated length. Per `README.md`,
report the **stricter** of the worksheet and schema limits and note any discrepancy.

| Column | Worksheet Max Length | Schema Column | Schema `MaxLength` (bytes) | Effective Characters | Stricter |
| ------ | -------------------: | ------------- | -------------------------: | -------------------: | -------- |
| Code | 20 | `BG_CODE` | 64 | 32 | Worksheet (20) |
| Building Name | 40 | `BG_SITE` | 510 | 255 | Worksheet (40) |
| Country | — | `BG_COUNTRY` | 128 | 64 | Schema (64) |
| Building Type | — | `BG_BUILDING_TYPE` | 60 | 30 | Schema (30) |
| Building Address | 300 | `BG_ADDRESS` | 600 | 300 | Equal (300) |
| Building Postcode | 15 | `BG_POSTCODE` | 30 | 15 | Equal (15) |
| Latiitude | 15 | `Latitude` (`float`) | 8 | n/a | Numeric |
| Longitude | 15 | `Longitude` (`float`) | 8 | n/a | Numeric |
| Elevation | 15 | `Elevation` (`float`) | 8 | n/a | Numeric |
| GIS Reference | 255 | `GISReference` | 510 | 255 | Equal (255) |
| External System | 100 | `ExtSystem` | 200 | 100 | Equal (100) |
| External Object | 100 | `ExtObject` | 200 | 100 | Equal (100) |
| External Identifier | 100 | `ExtIdentifier` | 200 | 100 | Equal (100) |
| SIte - Building | 100 | *(not imported)* | — | — | Worksheet (100) |

`nvarchar` columns in `/EVO/Schema/FLOCATE.json` record `MaxLength` in **bytes**. The
character limit is `MaxLength / 2`.

### Data Type Expectations

| Column | Expected |
| ------ | -------- |
| Code, Building Name, Building Address, Building Postcode, GIS Reference, External System, External Object, External Identifier | Alpha-numeric text (A) |
| Latiitude, Longitude, Elevation | Numeric, decimal permitted (N). A non-numeric value is an **Error**. |
| Complex / Site, Country, Building Type, Time Zone | Drop-down text, matched exactly against the source list |

### Allowed Values

The following columns are drop-down restricted. A value not present in the corresponding
source list is an **Error** when that source worksheet was supplied, and a
**Warning / REQUIRES DATABASE VERIFICATION** when it was not.

| Column | Source Worksheet | Source Column | Example Values |
| ------ | ---------------- | ------------- | -------------- |
| Complex / Site | `Sites` | `Name` | `Upminster`, `MRI UK`, `MRI South Africa` |
| Country | `Countries` | Country name | `England`, `United Kingdom`, `South Africa` |
| Building Type | `Building Type` | `Name` | `Store`, `Sports` |
| Time Zone | `Time Zone` | `Code` | `GMT`, `UTC` |

### Format Rules

* `SIte - Building` is generated as `{Complex / Site} - {Building Name}` — site name, space,
  hyphen, space, building name. If supplied and it does not match that concatenation of the
  row's own `Complex / Site` and `Building Name`, report an **Error**.
* `Building Address` may contain embedded newline characters (the worksheet examples do).
  Do not report a newline as an invalid character; do include it in the length count.
* Leading and trailing whitespace on drop-down columns is an **Error** — values are matched
  exactly.

### Uniqueness Rules

1. **Code** must be unique across the worksheet. Duplicates are an **Error**.
2. **SIte - Building** must be unique — it is the key consumed by `BuildingFloors` and
   `Locations`. Two rows producing the same `{Complex / Site} - {Building Name}` are an
   **Error**, even where the `Code` values differ.
3. `Building Name` need not be globally unique, but must be unique within a
   `Complex / Site` (this is what makes rule 2 hold).

### Row Structure Rules

* The worksheet is terminated by a row containing `END` in every column. Do not treat the
  `END` row as data.
* Column Q (and beyond) contains the internal `END` / `EXAMPLE` markers used by the
  workbook. These are **not** intake columns.
* Rows marked `EXAMPLE` in the far-right column are worked examples supplied by MRI and are
  **not** client data.
* Rows where only the auto-generated `SIte - Building` column contains a value (` - `) are
  empty template rows — ignore them.

### Cross-Reference Rules

* The `SIte - Building` values produced here are the authoritative drop-down source for:
  * `/EVO/Rules/BuildingFloors.md` — the `Site - Building` column
  * `/EVO/Rules/FAREALO.md` — the `Site - Building Name` column
* Any value used on those worksheets that does not appear here is an **Error** when the
  `Buildings` sheet was supplied, and a **Warning / REQUIRES DATABASE VERIFICATION**
  when it was not.

### External References — Availability

Not every entity referenced by this worksheet is held in this repository. Before reporting
a referential issue, check whether the reference can actually be validated. Never state
that a reference is valid or invalid when the referenced data is not available.

| Referenced | Used By | Held In This Repo? | How To Validate |
| ---------- | ------- | ------------------ | --------------- |
| Sites | `Complex / Site` | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION** — do not report as an error. |
| Countries | `Country` | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION**. |
| Building Type | `Building Type` | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION**. |
| Time Zone | `Time Zone` | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION**. Note the `Time Zone` worksheet marks `GMT` and `UTC` as *"Default value already on the database"*. |
| Cost Centre (`CostCentreId`) | *(not collected here)* | **No** | Not an intake column on this worksheet. |
| Client (`BG_FKEY_CLNT_SEQ`, `ClientId`) | *(not collected here)* | **No** | Not an intake column on this worksheet. |

> **Note:** `FLOCATE` has 145 columns in `/EVO/Schema/FLOCATE.json`, of which only the 15
> intake columns above are collected on this worksheet. `BG_SEQ` is the identity primary
> key; the remaining columns (property valuations, lease details, floor and parking counts,
> opening days, authorities, user-defined fields, audit columns and the `…Utc` / `…TZ`
> time-zone pairs) are either nullable or carry database defaults. Do **not** report them
> as missing expected columns.

---

## Sample Data (from the collection sheet)

| # | Complex / Site | Code | Building Name | Country | Building Type | Building Postcode | Time Zone | SIte - Building (auto) | Marker |
| - | -------------- | ---- | ------------- | ------- | ------------- | ----------------- | --------- | ---------------------- | ------ |
| 1 | Upminster | EXCHANGE | TEST MRI Building | England | Store | RM14 3BT | GMT | Upminster - TEST MRI Building | EXAMPLE |
| 2 | Upminster | ESSEX | TEST MRI BUILDING 2 | England | Sports | RM14 2SJ | GMT | Upminster - TEST MRI BUILDING 2 | EXAMPLE |
| 3 | MRI UK | 201 | 9 King Street | United Kingdom | | EC2V 8EA | GMT | MRI UK - 9 King Street | |
| 4 | MRI UK | 202 | MRI Sleaford | United Kingdom | | RM14 2SJ | GMT | MRI UK - MRI Sleaford | |
| 5 | MRI South Africa | 301 | FredRd Main Building | South Africa | Store | 7740 | GMT | MRI South Africa - FredRd Main Building | |

---

## Worksheet Guidance Notes

* Worksheet heading: *"Buildings"*.
* Worksheet note: *"Locations within Concept Evolution are grouped together using logical
  grouping. Buildings are grouped on Sites, Locations are grouped within Buildings."*
* Menu key: *"Columns in RED are compulsory if the Tab is being used"*.
* Menu key: *"Select from dropdown ONLY - do not copy / paste, type over etc.."*
* Menu key: *"The numbers that appear in row 5 on the sheets indicate the character limit."*

---

## Source Notes

1. The column heading in `I6` is spelled **`Latiitude`** on the worksheet. It maps to
   `FLOCATE.Latitude`. Match on either spelling but report using the name the supplied file
   uses.
2. The column heading in `P6` is spelled **`SIte - Building`** (capital `I` in `SIte`),
   while the `BuildingFloors` worksheet refers to the same value as `Site - Building`. They
   are the same field.
3. Row 4 of column `L` (GIS Reference) reads **`Test`**. This is a typo for `Text` — the same
   typo appears on the `Locations` worksheet. Treat GIS Reference as a text field.
4. Client rows 3 and 4 leave `Building Type` blank while the `EXAMPLE` rows populate it.
   `Building Type` is at the client's discretion, so a blank is not an error.
5. The `Sites` worksheet lists `Upminster` with country `England` and `MRI UK` with country
   `United Kingdom`. Both appear in the `Country` drop-down; the worksheet does not
   normalise them.

---

## Type Legend

| Code | Meaning |
| ---- | ------- |
| A | Alpha-numeric field |
| D | Date field — format `dd/mm/yyyy` is acceptable |
| N | Numeric field |

## Worksheet Colour Legend (from the `Menu` tab)

| Colour | Meaning |
| ------ | ------- |
| Red | Compulsory |
| Blue | Recommended |
| Black | Client's discretion |
| Green | Auto generated — DO NOT TOUCH |
