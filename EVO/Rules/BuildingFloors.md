# EVO — BuildingFloors Rules

**Product:** EVO
**Target Table:** `BuildingFloors`
**Schema File:** `/EVO/Schema/BuildingFloors.json`
**Source Workbook:** MRI Evolution Data Collection Sheet (`4/492 Iss 7`, v5.3.0)
**Source Worksheet:** `BuildingFloors`

These rules are taken from the MRI Evolution Data Collection Sheet. They describe how the
intake (collection sheet) columns must be populated before the file can be integrated into
the EVO `BuildingFloors` table.

They are **additional to** the structural rules in `/EVO/Schema/BuildingFloors.json`.
Where the two disagree on data type or length, the JSON schema remains the source of
truth for the physical database, and the rules below define the business/intake expectation.

> The `BuildingFloors` worksheet is a **link table**. It records which floors from the
> global floor library (`Floors` worksheet) exist in which building (`Buildings`
> worksheet). It carries no attributes of its own on the collection sheet.
>
> The worksheet tab is **RED — compulsory**.

---

## Sheet-Level Control Question

Row 4 of the worksheet carries a control question that governs whether the grid below is
populated at all:

| Cell | Question | Answer | Effect |
| ---- | -------- | ------ | ------ |
| `A4` / `B4` | *"Should ALL Floors be assigned to ALL Buildings?"* | `Yes` / `No` (drop-down, `YesNo` list) | *"If No please complete the table below. If Yes do not complete the table below."* |

Validation consequences:

1. If the answer is **`Yes`**, the grid from row 7 onward must be **empty**. Populated rows
   are a **Warning** — they will be ignored, because every floor is assigned to every
   building.
2. If the answer is **`No`**, the grid from row 7 onward must contain at least one row.
   An empty grid is an **Error**.
3. Any value other than `Yes` or `No` in `B4` is an **Error**.
4. If `B4` is blank, treat the sheet as **`No`** (the workbook ships with `No` selected) and
   report a **Warning** that the control answer was not supplied.

---

## Field Rules

| Intake Column (Worksheet) | Target Column | Type | Length | Mandatory | Rule |
| ------------------------- | ------------- | ---- | -----: | --------- | ---- |
| Site - Building | `BuildingId` | Select from Drop Down | — | **YES** | Select from the drop-down only — do not copy / paste or type over.<br>Must exactly match a `SIte - Building` value generated on the `Buildings` worksheet, in the form `{Complex / Site} - {Building Name}`. |
| Floor | `FloorLibraryId` / `Name` | Select from Drop Down | 64 | **YES** | Select from the drop-down only.<br>Must exactly match a `Floor` value on the `Floors` worksheet. |
| Confirmation | *(none — not imported)* | Auto Generated — **DO NOT TOUCH** | — | Auto | Worksheet formula that echoes `A link between {Site - Building} and {Floor} has been established`. Not an intake column and not loaded into the database. |

### Column Groupings (as presented on the worksheet)

* **Drop-down only:** Site - Building, Floor
* **Auto generated (green — DO NOT TOUCH):** Confirmation

---

## Validation Rules

### Required Values

When the control question is answered `No`, report an **Error** for any populated row where
the following are blank:

* Site - Building
* Floor

A row where **both** are blank is not an error — it is an unused row and must be ignored.
A row where **one** is blank is an **Error** (incomplete link).

### Maximum Length

Report an **Error** where the supplied value exceeds the stated length.

| Column | Schema Column | Schema `MaxLength` (bytes) | Effective Characters |
| ------ | ------------- | -------------------------: | -------------------: |
| Floor | `BuildingFloors.Name` (`nvarchar`) | 128 | 64 |
| *(Floor Description — from the `Floors` worksheet)* | `BuildingFloors.Description` (`nvarchar`) | 510 | 255 |

`nvarchar` columns in `/EVO/Schema/BuildingFloors.json` record `MaxLength` in **bytes**.
The character limit is `MaxLength / 2`.

### Allowed Values

`Site - Building` and `Floor` are both drop-down-restricted. Any value not present in the
corresponding source list is an **Error** when that source worksheet was supplied, and a
**Warning / REQUIRES DATABASE VERIFICATION** when it was not.

| Column | Source List |
| ------ | ----------- |
| Site - Building | `Buildings` worksheet, auto-generated `SIte - Building` column → `/EVO/Rules/FLOCATE.md` |
| Floor | `Floors` worksheet, `Floor` column → `/EVO/Rules/Floors.md` |

### Format Rules

* `Site - Building` must be in the form `{Complex / Site} - {Building Name}` — the site name,
  a space, a hyphen, a space, then the building name. A value that does not contain
  ` - ` is an **Error**.
* Leading and trailing whitespace on either column is an **Error** — drop-down values are
  matched exactly.

### Uniqueness Rules

1. The combination of `Site - Building` + `Floor` must be **unique**. A repeated pair means
   the same floor is being attached to the same building twice — report as an **Error**.
2. The same `Floor` value may appear against many different buildings, and the same building
   may appear against many floors. Neither on its own is an issue.

### Row Structure Rules

* The worksheet is terminated by a row containing `END` in every column. Do not treat the
  `END` row as data.
* Column D (and beyond) contains the internal `END` / `EXAMPLE` markers used by the
  workbook. These are **not** intake columns.
* Rows marked `EXAMPLE` in the far-right column are worked examples supplied by MRI and are
  **not** client data.

### Conditional Requirements

* This worksheet is only meaningful once both `/EVO/Rules/Floors.md` and
  `/EVO/Rules/FLOCATE.md` have been loaded. If either is supplied alongside this file,
  validate the cross-references against it rather than deferring to the database.
* Load order: **Floors → FLOCATE (Buildings) → BuildingFloors → FAREALO (Locations)**.

### External References — Availability

Not every entity referenced by this worksheet is held in this repository. Before reporting
a referential issue, check whether the reference can actually be validated. Never state
that a reference is valid or invalid when the referenced data is not available.

| Referenced | Used By | Held In This Repo? | How To Validate |
| ---------- | ------- | ------------------ | --------------- |
| Buildings (`FLOCATE`) | `Site - Building` | **Yes** — `/EVO/Schema/FLOCATE.json` + `/EVO/Rules/FLOCATE.md` | **Error** if the `Buildings` worksheet was supplied and the value is not found. **Warning / REQUIRES DATABASE VERIFICATION** if it was not supplied. |
| Floors | `Floor` | **Yes** — `/EVO/Schema/Floors.json` + `/EVO/Rules/Floors.md` | **Error** if the `Floors` worksheet was supplied and the value is not found. **Warning / REQUIRES DATABASE VERIFICATION** if it was not supplied. |
| Sites | `Site - Building` (prefix) | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION** — do not report as an error. |
| Floor Library (`FloorLibraryId` target table) | `Floor` | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION**. |

> **Note:** `BuildingFloorId`, `Height`, `Elevation`, `Notes`, `ExtSystem`, `ExtObject`,
> `ExtIdentifier`, `CreatedBy`, `CreatedDate`, `ModifiedBy`, `ModifiedDate`, `Version` and
> `Deleted` are **not** collected on this worksheet. `BuildingFloorId` is an identity
> column and the remainder are nullable or carry database defaults. Do **not** report them
> as missing expected columns.

---

## Sample Data (from the collection sheet)

| # | Site - Building | Floor | Confirmation (auto) | Marker |
| - | --------------- | ----- | ------------------- | ------ |
| 1 | Upminster - TEST MRI Building | Basement | A link between Upminster - TEST MRI Building and Basement has been established | EXAMPLE |
| 2 | Upminster - TEST MRI Building | Ground | A link between Upminster - TEST MRI Building and Ground has been established | EXAMPLE |
| 3 | Upminster - TEST MRI BUILDING 2 | Basement | A link between Upminster - TEST MRI BUILDING 2 and Basement has been established | EXAMPLE |
| 4 | Upminster - TEST MRI BUILDING 2 | Ground | A link between Upminster - TEST MRI BUILDING 2 and Ground has been established | EXAMPLE |
| 5 | MRI UK - 9 King Street | G | A link between MRI UK - 9 King Street and G has been established | |
| 6 | MRI UK - 9 King Street | 1 | … | |
| 7 | MRI UK - 9 King Street | 2 | … | |
| 8 | MRI UK - 9 King Street | 3 | … | |
| 9 | MRI UK - 9 King Street | 4 | … | |
| 10 | MRI UK - 9 King Street | 5 | … | |
| 11 | MRI UK - MRI Sleaford | G | … | |
| 12 | MRI UK - MRI Sleaford | 1 | … | |
| 13 | MRI South Africa - FredRd Main Building | G | … | |
| 14 | MRI South Africa - FredRd Main Building | 1 | … | |
| 15 | MRI South Africa - FredRd Main Building | 2 | … | |
| 16 | MRI South Africa - FredRd Main Building | 3 | … | |
| 17 | MRI South Africa - FredRd Main Building | 4 | … | |

---

## Worksheet Guidance Notes

* Worksheet heading: *"Building - Floor Links"*.
* Worksheet note: *"Links between Buildings and Floors need to be specified"*.
* Menu key: *"Select from dropdown ONLY - do not copy / paste, type over etc.."*
* Menu key: *"Auto Generated DO NOT TOUCH"* — applies to the `Confirmation` column.

---

## Source Notes

1. The example rows reference floors `Basement` and `Ground`, but the `Floors` worksheet
   holds those values in its **`Floor Description`** column — the corresponding `Floor`
   codes are `B` and `G`. The drop-down on this sheet is sourced from the `Floor` column.
   Treat the `EXAMPLE`-marked rows as demonstration data and do not use them to infer that
   descriptions are acceptable in the `Floor` column.
2. The `Buildings` worksheet spells its auto-generated column `SIte - Building` (capital
   `I`), while this worksheet spells the same concept `Site - Building`. They are the same
   field — match on either spelling.
3. Rows 25–34 of the worksheet contain a single space in the `Confirmation` column with no
   `Site - Building` or `Floor` value. These are empty template rows, not data.

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
