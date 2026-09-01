# EVO — FAREALO Rules

**Product:** EVO
**Target Table:** `FAREALO`
**Schema File:** `/EVO/Schema/FAREALO.json`
**Source Workbook:** MRI Evolution Data Collection Sheet (`4/492 Iss 7`, v5.3.0)
**Source Worksheet:** `Locations`

These rules are taken from the MRI Evolution Data Collection Sheet. They describe how the
intake (collection sheet) columns must be populated before the file can be integrated into
the EVO `FAREALO` table.

They are **additional to** the structural rules in `/EVO/Schema/FAREALO.json`.
Where the two disagree on data type or length, the JSON schema remains the source of
truth for the physical database, and the rules below define the business/intake expectation.

> `FAREALO` is the **Locations** table in Concept Evolution. The worksheet note reads:
> *"Locations within Concept Evolution are grouped together using logical grouping.
> Buildings are grouped on Sites, Locations are grouped within Buildings."*
>
> The worksheet tab is **RED — compulsory**.

---

## Field Rules

| # | Intake Column (Worksheet) | Target Column (`FAREALO`) | Type | Length | Mandatory | Rule |
| -: | ------------------------- | ------------------------- | ---- | -----: | --------- | ---- |
| 1 | Location Description | `LO_DESCRIPTION` | Text (A) | 50 | **YES (RED)** | The location's name (e.g. `Boiler Room`, `Reception`). |
| 2 | Floor | `LO_FLOOR` / `FloorId` / `BuildingFloorId` | Select from Drop Down | — | No (Client's discretion) | Select from the drop-down only. Must match a `Floor` on the `Floors` worksheet, **and** that floor must be linked to this row's building on the `BuildingFloors` worksheet. |
| 3 | Room Number | `LO_ROOM_NUMBER` | Text (A) | 20 | No (Client's discretion) | The room / space reference (e.g. `A0.01`). |
| 4 | Cost Code | `LO_CCENTRE` | Select from Drop Down | — | No (Client's discretion) | Select from the drop-down only. Must match a `Key` on the `Cost Codes` worksheet. See **Source Notes** for the length conflict. |
| 5 | Max Seats | `LO_SEATING_MAX` | Number (N) | — | No (Client's discretion) | Whole number. Only meaningful for bookable locations. |
| 6 | Telephone | `LO_TEL` | Text (A) | 25 | No (Client's discretion) | Free text — extensions such as `Ext 3461` are acceptable. |
| 7 | Area | `AreaId` / `AreaName` | Select from Drop Down | — | No (Client's discretion) | Select from the drop-down only. Must match a `Description` on the `Areas` worksheet (e.g. `Plant Rooms`, `Toilets`). |
| 8 | Location Type | `LO_FKEY_LOT_SEQ` | Select from Drop Down | — | No (Client's discretion) | Select from the drop-down only. Must match a `Decription` on the `Location Type` worksheet (e.g. `Boardroom`, `Meeting`, `Conference`, `Training`). |
| 9 | Full Location Description | `HierarchyPath` *(inferred)* | Auto Generated — **DO NOT TOUCH** | — | Auto | Worksheet formula producing `{Site - Building Name} - {Floor} - {Location Description} ({Room Number})`. |
| 10 | Latiitude | `Latitude` | Number (Dec) | 15 | No (Client's discretion) | Decimal degrees. Worksheet heading is misspelled — see **Source Notes**. |
| 11 | Longitude | `Longitude` | Number (Dec) | 15 | No (Client's discretion) | Decimal degrees. |
| 12 | Elevation | `Elevation` | Number (Dec) | 15 | No (Client's discretion) | — |
| 13 | GIS Reference | `GISReference` | Text (A) | 255 | No (Client's discretion) | Worksheet row 4 reads `Test` — a typo for `Text`. |
| 14 | External System | `ExtSystem` | Text (A) | 100 | No (Client's discretion) | Name of the third-party system this location originates from. |
| 15 | External Object | `ExtObject` | Text (A) | 100 | No (Client's discretion) | Object type in that system. |
| 16 | External Identifier | `ExtIdentifier` | Text (A) | 100 | No (Client's discretion) | Identifier in that system. |
| 17 | Net Usable Space | `LO_NET_SPACE` | Number (N) — **Square Metres** | — | No (Client's discretion) | Must be supplied in square metres. Decimal permitted. |
| 18 | Location URL | `LocationUrl` | Text (A) | 255 | No (Client's discretion) | — |
| 19 | Unit Weighting | `SchedulingWeight` | Decimal (2dp) | — | No (Client's discretion) | Numeric, to 2 decimal places. |
| 20 | Site - Building Name | `LO_FKEY_BG_SEQ` / `LO_BCODE` | Select from Drop Down | — | **YES (RED)** | Select from the drop-down only. Must match a `SIte - Building` value generated on the `Buildings` worksheet, in the form `{Complex / Site} - {Building Name}`. |

### Column Groupings (as presented on the worksheet)

* **Compulsory (RED):** Location Description, Site - Building Name
* **Client's discretion (BLACK):** Floor, Room Number, Cost Code, Max Seats, Telephone, Area, Location Type, Net Usable Space, Location URL, Unit Weighting
* **Client's discretion (unmarked):** Latiitude, Longitude, Elevation, GIS Reference, External System, External Object, External Identifier
* **Auto generated (DO NOT TOUCH):** Full Location Description

---

## Validation Rules

### Required Values

Report an **Error** for any row where the following are blank:

* Location Description
* Site - Building Name

All other columns are at the client's discretion and may be left blank.

Report a **Warning** where `Floor` is blank but the row's building has floors linked on the
`BuildingFloors` worksheet — an unfloored location in a multi-floor building is usually a
data-capture omission.

### Maximum Length

Report an **Error** where the supplied value exceeds the stated length. Per `README.md`,
report the **stricter** of the worksheet and schema limits and note any discrepancy.

| Column | Worksheet Max Length | Schema Column | Schema `MaxLength` (bytes) | Effective Characters | Stricter |
| ------ | -------------------: | ------------- | -------------------------: | -------------------: | -------- |
| Location Description | 50 | `LO_DESCRIPTION` | 510 | 255 | Worksheet (50) |
| Floor | — | `LO_FLOOR` | 128 | 64 | Schema (64) |
| Room Number | 20 | `LO_ROOM_NUMBER` | 128 | 64 | Worksheet (20) |
| Cost Code | — | `LO_CCENTRE` | 24 | 12 | Schema (12) |
| Max Seats | — | `LO_SEATING_MAX` (`int`) | 4 | n/a | Numeric |
| Telephone | 25 | `LO_TEL` | 50 | 25 | Equal (25) |
| Area | — | `AreaName` | 128 | 64 | Schema (64) |
| Full Location Description | — | `HierarchyPath` | 510 | 255 | Schema (255) |
| Latiitude | 15 | `Latitude` (`float`) | 8 | n/a | Numeric |
| Longitude | 15 | `Longitude` (`float`) | 8 | n/a | Numeric |
| Elevation | 15 | `Elevation` (`float`) | 8 | n/a | Numeric |
| GIS Reference | 255 | `GISReference` | 510 | 255 | Equal (255) |
| External System | 100 | `ExtSystem` | 200 | 100 | Equal (100) |
| External Object | 100 | `ExtObject` | 200 | 100 | Equal (100) |
| External Identifier | 100 | `ExtIdentifier` | 200 | 100 | Equal (100) |
| Net Usable Space | — | `LO_NET_SPACE` (`float`) | 8 | n/a | Numeric |
| Location URL | 255 | `LocationUrl` | 510 | 255 | Equal (255) |
| Unit Weighting | — | `SchedulingWeight` (`float`) | 8 | n/a | Numeric |

`nvarchar` columns in `/EVO/Schema/FAREALO.json` record `MaxLength` in **bytes**. The
character limit is `MaxLength / 2`.

### Data Type Expectations

| Column | Expected |
| ------ | -------- |
| Location Description, Room Number, Telephone, GIS Reference, External System, External Object, External Identifier, Location URL | Alpha-numeric text (A) |
| Max Seats | Whole number (N). A decimal or non-numeric value is an **Error**. |
| Net Usable Space | Numeric, decimal permitted (N), expressed in **square metres**. |
| Unit Weighting | Numeric, decimal to **2 places** (N). More than 2 decimal places is a **Warning**. |
| Latiitude, Longitude, Elevation | Numeric, decimal permitted (N). |
| Floor, Cost Code, Area, Location Type, Site - Building Name | Drop-down text, matched exactly against the source list |

### Allowed Values

The following columns are drop-down restricted. A value not present in the corresponding
source list is an **Error** when that source worksheet was supplied, and a
**Warning / REQUIRES DATABASE VERIFICATION** when it was not.

| Column | Source Worksheet | Source Column | Example Values |
| ------ | ---------------- | ------------- | -------------- |
| Floor | `Floors` | `Floor` | `B`, `G`, `1`, `2` … |
| Cost Code | `Cost Codes` | `Key` | `CCPPM`, `CCRCT`, `6030`, `7306` … |
| Area | `Areas` | `Description` | `Tea Point`, `Garden`, `Plant Rooms`, `Toilets` |
| Location Type | `Location Type` | `Decription` | `Boardroom`, `Meeting`, `Conference`, `Training` |
| Site - Building Name | `Buildings` | `SIte - Building` | `MRI UK - 9 King Street` … |

### Format Rules

* `Site - Building Name` must be in the form `{Complex / Site} - {Building Name}` — the site
  name, a space, a hyphen, a space, then the building name. A value that does not contain
  ` - ` is an **Error**.
* `Full Location Description` is generated as
  `{Site - Building Name} - {Floor} - {Location Description} ({Room Number})`. Where
  `Room Number` is blank the parentheses are still emitted, e.g.
  `Upminster - TEST MRI Building - Basement - Boiler Room ()`. If supplied and it does not
  match that concatenation of the row's own values, report an **Error**.
* Leading and trailing whitespace on drop-down columns is an **Error** — values are matched
  exactly. See **Source Notes** regarding leading spaces on `Location Description`.

### Single-Value Rules

* Each row describes **one** location. `Location Description`, `Room Number` and `Floor` must
  each carry a single value — a comma- or slash-separated list is an **Error**.

### Uniqueness Rules

1. **Full Location Description** must be unique. Two rows generating the same value are an
   **Error** — the same location is being loaded twice.
2. `Location Description` must be unique within the combination of
   `Site - Building Name` + `Floor`. Duplicates are an **Error**.
3. `Room Number`, where supplied, must be unique within `Site - Building Name`. Duplicates
   are an **Error**.

### Row Structure Rules

* The worksheet is terminated by a row containing `END` in every column. Do not treat the
  `END` row as data.
* Column U (and beyond) contains the internal `END` / `EXAMPLE` markers used by the
  workbook. These are **not** intake columns.
* Rows marked `EXAMPLE` in the far-right column are worked examples supplied by MRI and are
  **not** client data.
* Rows where only the auto-generated `Full Location Description` column contains a value
  (` -  -  ()`) are empty template rows — ignore them.

### Conditional Requirements

* A `Floor` value is only valid for a row if the pair
  `{Site - Building Name}` + `{Floor}` exists on the `BuildingFloors` worksheet. Where that
  worksheet was supplied, a missing link is an **Error**; otherwise report
  **Warning / REQUIRES DATABASE VERIFICATION**.
* `Max Seats` is only meaningful where `Location Type` allows bookings (`Allow Bookings = Yes`
  on the `Location Type` worksheet). `Max Seats` populated on a non-bookable location type is
  a **Warning**.
* Load order: **Floors → FLOCATE (Buildings) → BuildingFloors → FAREALO (Locations)**.

### Cross-Reference Rules

* `Site - Building Name` resolves against `/EVO/Rules/FLOCATE.md`.
* `Floor` resolves against `/EVO/Rules/Floors.md`, and the building/floor pairing resolves
  against `/EVO/Rules/BuildingFloors.md`.

### External References — Availability

Not every entity referenced by this worksheet is held in this repository. Before reporting
a referential issue, check whether the reference can actually be validated. Never state
that a reference is valid or invalid when the referenced data is not available.

| Referenced | Used By | Held In This Repo? | How To Validate |
| ---------- | ------- | ------------------ | --------------- |
| Buildings (`FLOCATE`) | `Site - Building Name` | **Yes** — `/EVO/Schema/FLOCATE.json` + `/EVO/Rules/FLOCATE.md` | **Error** if the `Buildings` worksheet was supplied and the value is not found. **Warning / REQUIRES DATABASE VERIFICATION** if it was not supplied. |
| Floors | `Floor` | **Yes** — `/EVO/Schema/Floors.json` + `/EVO/Rules/Floors.md` | **Error** if the `Floors` worksheet was supplied and the value is not found. **Warning / REQUIRES DATABASE VERIFICATION** if it was not supplied. |
| BuildingFloors | `Floor` + `Site - Building Name` pairing | **Yes** — `/EVO/Schema/BuildingFloors.json` + `/EVO/Rules/BuildingFloors.md` | **Error** if the `BuildingFloors` worksheet was supplied and the pair is not found. **Warning / REQUIRES DATABASE VERIFICATION** if it was not supplied. |
| Cost Codes | `Cost Code` | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION** — do not report as an error. |
| Areas | `Area` | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION**. |
| Location Type | `Location Type` | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION**. |
| Sites | `Site - Building Name` (prefix) | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION**. |

> **Note:** `FAREALO` has 78 columns in `/EVO/Schema/FAREALO.json`, of which only the 19
> intake columns above are collected on this worksheet. `LO_SEQ` is the identity primary
> key; the remaining columns (booking, exam and scheduling settings, stock flags, condition,
> notes, user-defined fields, parent/scope columns and audit columns) are either nullable or
> carry database defaults. Do **not** report them as missing expected columns.

---

## Sample Data (from the collection sheet)

| # | Location Description | Floor | Room Number | Max Seats | Telephone | Area | Location Type | Net Usable Space | Site - Building Name | Marker |
| - | -------------------- | ----- | ----------- | --------: | --------- | ---- | ------------- | ---------------: | -------------------- | ------ |
| 1 | Boiler Room | Basement | | | | Plant Rooms | | | Upminster - TEST MRI Building | EXAMPLE |
| 2 | Men's WC | Ground | | | | Toilets | | | Upminster - TEST MRI BUILDING 2 | EXAMPLE |
| 3 | Foyer | Ground | | | | | | | Upminster - TEST MRI Building | EXAMPLE |
| 4 | Boardroom | Ground | | 12 | Ext 3461 | | Boardroom | 12.5 | Upminster - TEST MRI Building | EXAMPLE |
| 5 | Reception | G | A0.01 | | | | | | MRI UK - 9 King Street | |
| 6 | Foyer | G | A0.02 | | | | | | MRI UK - 9 King Street | |
| 7 | Main Corridor | G | A0.03 | | | | | | MRI UK - 9 King Street | |
| 8 | Main Kitchen | 1 | A0.04 | | | | | | MRI UK - 9 King Street | |
| 9 | Archive Room | 2 | A0.05 | | | | | | MRI UK - 9 King Street | |
| 10 | Comms Room | 4 | FR151 | | | | Training | | MRI South Africa - FredRd Main Building | |

---

## Worksheet Guidance Notes

* Worksheet heading: *"Locations"*.
* Worksheet note: *"Locations within Concept Evolution are grouped together using logical
  grouping. Buildings are grouped on Sites, Locations are grouped within Buildings."*
* Menu key: *"Columns in RED are compulsory if the Tab is being used"*.
* Menu key: *"Select from dropdown ONLY - do not copy / paste, type over etc.."*
* Menu key: *"The numbers that appear in row 5 on the sheets indicate the character limit."*

---

## Source Notes

1. The column heading in `J6` is spelled **`Latiitude`** on the worksheet. It maps to
   `FAREALO.Latitude`. Match on either spelling but report using the name the supplied file
   uses.
2. Row 4 of column `M` (GIS Reference) reads **`Test`**. This is a typo for `Text` — the same
   typo appears on the `Buildings` worksheet. Treat GIS Reference as a text field.
3. The `Location Type` worksheet spells its first column heading **`Decription`** (missing
   `s`). It is the description column and is the drop-down source for `Location Type` here.
4. Almost every client row on the worksheet has a **leading space** in
   `Location Description` (` Reception`, ` Foyer`, ` Main Corridor` …), and that leading
   space is carried into the auto-generated `Full Location Description`. Report leading and
   trailing whitespace in `Location Description` as a **Warning** so it can be trimmed before
   load — it will otherwise be stored verbatim.
5. The example rows use `Basement` and `Ground` in the `Floor` column, but the `Floors`
   worksheet holds those values in its `Floor Description` column — the corresponding `Floor`
   codes are `B` and `G`. Treat the `EXAMPLE`-marked rows as demonstration data.
6. The `Cost Codes` worksheet permits a 16-character `Key`, but `FAREALO.LO_CCENTRE` is
   `nvarchar` 24 bytes (12 characters). A 13–16 character cost code will not fit. Report the
   discrepancy rather than silently resolving it.
7. Row 8 and row 11 of the worksheet are an identical `EXAMPLE` pair
   (`Men's WC` / `Ground` / `Upminster - TEST MRI BUILDING 2`). Taken literally this breaches
   the uniqueness rules; treat `EXAMPLE`-marked rows as demonstration data only.
8. `Net Usable Space` is captured in **square metres** (worksheet row 4). Confirm the client
   has not supplied square feet — the worksheet performs no conversion.

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
