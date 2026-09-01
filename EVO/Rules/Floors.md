# EVO — Floors Rules

**Product:** EVO
**Target Table:** `Floors`
**Schema File:** `/EVO/Schema/Floors.json`
**Source Workbook:** MRI Evolution Data Collection Sheet (`4/492 Iss 7`, v5.3.0)
**Source Worksheet:** `Floors`

These rules are taken from the MRI Evolution Data Collection Sheet. They describe how the
intake (collection sheet) columns must be populated before the file can be integrated into
the EVO `Floors` table.

They are **additional to** the structural rules in `/EVO/Schema/Floors.json`.
Where the two disagree on data type or length, the JSON schema remains the source of
truth for the physical database, and the rules below define the business/intake expectation.

> The `Floors` worksheet defines the **floor library** — the master list of floor names used
> across every Site and Building in Concept Evolution. Floors are defined once here and then
> attached to individual Buildings on the `BuildingFloors` worksheet.
>
> The worksheet tab is **RED — compulsory**.

---

## Field Rules

| Intake Column (Worksheet) | Target Column | Type | Length | Mandatory | Rule |
| ------------------------- | ------------- | ---- | -----: | --------- | ---- |
| Floor | `Name` | Text (A) — Unique Value | 64 | **YES** | The floor code / short name (e.g. `B`, `G`, `1`, `2`). Must be unique across the whole worksheet.<br>See **Source Notes** — the worksheet limit of 64 exceeds the `Floors.Name` limit in the schema. |
| Floor Description | *(no column on `Floors`)* | Text (A) | 255 | **YES** | The readable description of the floor (e.g. `Basement`, `Ground`).<br>Carried on `BuildingFloors.Description` — see `/EVO/Rules/BuildingFloors.md`. |

### Column Groupings (as presented on the worksheet)

* **Compulsory (RED):** Floor, Floor Description

Row 4 of the worksheet states the field type, row 5 states the character limit, and row 6
carries the column heading. Rows 7 onward are data. Rows marked `EXAMPLE` in the far-right
column are worked examples supplied by MRI and are **not** client data.

---

## Validation Rules

### Required Values

Report an **Error** for any row where the following are blank:

* Floor
* Floor Description

### Maximum Length

Report an **Error** where the supplied value exceeds the stated length.

| Column | Worksheet Max Length | Schema Column | Schema `MaxLength` (bytes) | Effective Characters |
| ------ | -------------------: | ------------- | -------------------------: | -------------------: |
| Floor | 64 | `Floors.Name` (`nvarchar`) | 32 | 16 |
| Floor Description | 255 | *(not on `Floors`)* | — | — |

`nvarchar` columns in `/EVO/Schema/Floors.json` record `MaxLength` in **bytes**. The
character limit is `MaxLength / 2`. Per `README.md`, report the **stricter** of the two
limits and note the discrepancy — for `Floor` the schema (16 characters) is stricter than
the worksheet (64 characters).

### Data Type Expectations

| Column | Expected |
| ------ | -------- |
| Floor | Alpha-numeric text. Values such as `1`, `2`, `3` are text, not numbers — a leading zero or trailing space must be preserved exactly. |
| Floor Description | Alpha-numeric text. |

### Uniqueness Rules

1. **Floor** is declared `Text (Unique Value)` on the worksheet. Duplicate `Floor` values
   anywhere on the sheet are an **Error**.
2. `Floor Description` is not required to be unique, but two different `Floor` values sharing
   the same `Floor Description` should be raised as a **Warning** — it usually indicates a
   duplicated row.

### Row Structure Rules

* The worksheet is terminated by a row containing `END` in every column. Do not treat the
  `END` row as data.
* Column C (and beyond) contains the internal `END` / `EXAMPLE` markers used by the
  workbook. These are **not** intake columns — report them as **Warning: column not
  defined** only if they appear in a supplied file.

### Cross-Reference Rules

* `Floor` values on this worksheet are the authoritative list consumed by:
  * `/EVO/Rules/BuildingFloors.md` — the `Floor` drop-down
  * `/EVO/Rules/FAREALO.md` — the `Floor` drop-down on the `Locations` worksheet
* Any `Floor` value used on those worksheets that does not appear here is an **Error** when
  the `Floors` sheet was supplied, and a **Warning / REQUIRES DATABASE VERIFICATION**
  when it was not.

### External References — Availability

Not every entity referenced by this worksheet is held in this repository. Before reporting
a referential issue, check whether the reference can actually be validated. Never state
that a reference is valid or invalid when the referenced data is not available.

| Referenced | Used By | Held In This Repo? | How To Validate |
| ---------- | ------- | ------------------ | --------------- |
| Floor Library (`FloorLibraryId` target table) | `BuildingFloors.FloorLibraryId` | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION** — do not report as an error. |
| Building (`Floors.BuildingId`) | `Floors.BuildingId` | **Yes**, as `/EVO/Schema/FLOCATE.json` | Validate against the supplied `Buildings` worksheet where one is provided; otherwise **Warning / REQUIRES DATABASE VERIFICATION**. |

> **Note:** `Floors.FloorId`, `Floors.BuildingId`, `Floors.FloorOrder` and the audit columns
> (`Version`, `Hash`, `Status`, `Deleted`, `CreatedBy`, `CreatedDate`, `ModifiedBy`,
> `ModifiedDate`) are **not** collected on the worksheet. They are system-populated —
> `FloorId` is an identity column and the remainder carry database defaults. Do **not**
> report them as missing expected columns.

Values that **can** be validated in this repository are the internal consistency rules
above (required values, lengths, data types and uniqueness), plus the `Floor` values
consumed by `/EVO/Rules/BuildingFloors.md` and `/EVO/Rules/FAREALO.md`.

---

## Sample Data (from the collection sheet)

| # | Floor | Floor Description | Marker |
| - | ----- | ----------------- | ------ |
| 1 | B | Basement | EXAMPLE |
| 2 | G | Ground | EXAMPLE |
| 3 | G | G | |
| 4 | 1 | 1 | |
| 5 | 2 | 2 | |
| 6 | 3 | 3 | |
| 7 | 4 | 4 | |
| 8 | 5 | 5 | |
| 9 | 6 | 6 | |
| 10 | 7 | 7 | |
| 11 | 8 | 8 | |
| 12 | 9 | 9 | |

---

## Worksheet Guidance Notes

* Worksheet heading: *"Floors"*.
* Worksheet note: *"Building Floors across all Sites"* — the floor library is global, not
  per-building. Floors are attached to individual Buildings on the `BuildingFloors`
  worksheet.
* Menu key: *"Columns in RED are compulsory if the Tab is being used"*.
* Menu key: *"The numbers that appear in row 5 on the sheets indicate the character limit."*

---

## Source Notes

1. The example rows supplied by MRI (`B` / `Basement`, `G` / `Ground`) and the client rows
   (`G` / `G`, `1` / `1` …) both contain a `G` value. Taken literally this breaches the
   `Text (Unique Value)` rule on `Floor`. Treat the two `EXAMPLE`-marked rows as
   demonstration data, not client data, and only apply the uniqueness rule across client
   rows.
2. The worksheet permits 64 characters for `Floor`, but `Floors.Name` in
   `/EVO/Schema/Floors.json` is `nvarchar` with `MaxLength` 32 bytes (16 characters), and
   `BuildingFloors.Name` is `nvarchar` 128 bytes (64 characters). The 64-character
   worksheet limit aligns with `BuildingFloors.Name`, not `Floors.Name`. Report the
   discrepancy rather than silently resolving it.
3. `Floor Description` has no counterpart column on the `Floors` table. The only 255-character
   description column in the EVO schema files supplied is `BuildingFloors.Description`
   (`nvarchar` 510 bytes = 255 characters), which matches the worksheet limit exactly.

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
