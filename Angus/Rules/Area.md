# Angus — Area Rules

**Product:** Angus
**Target Table:** `Area`
**Schema File:** `/Angus/Schema/Area.json`
**Source Workbook:** MRI Angus Data Collection Sheet
**Source Worksheet:** `Areas`

These rules are taken from the MRI Angus Data Collection Sheet. They describe how the
intake (collection sheet) columns must be populated before the file can be integrated
into the Angus `Area` table.

They are **additional to** the structural rules in `/Angus/Schema/Area.json`.
Where the two disagree on data type or length, the JSON schema remains the source of
truth for the physical database, and the rules below define the business/intake expectation.

---

## Field Rules

| Intake Column (Database Table Name) | MRI Angus Field Name | Length | Type | Mandatory | Rule |
| ----------------------------------- | -------------------- | -----: | ---- | --------- | ---- |
| Property Name | Property | 50 | Text (A) | **YES** | Must match MRI Angus |
| Building Name | Building | 50 | Text (A) | **YES** | Must match MRI Angus |
| Floor Name | Floor | 50 | Text (A) | **YES** | Must be unique to the Building Name |
| Suite Name | Location / Suite | 50 | Text (A) | No (Optional) | Must be unique to the floor for the Property Name + Building Name + Floor Name combination.<br>Only enter Suites for unoccupied spaces or common areas. |
| Is TR Area | Show in TR | 4 | Text (A) | **YES** | `X` or blank.<br>A combination of Is TR Area or Is PM Area is required if Is Common Area is blank. |
| Is PM Area | Show in PM | 4 | Text (A) | **YES** | `X` or blank.<br>A combination of Is TR Area or Is PM Area is required if Is Common Area is blank. |
| Is Common Area | Common Area | 4 | Text (A) | **YES** | `X` or blank.<br>The combination of Is TR Area, Is PM Area and Is Common Area must be unique.<br>Only use for shared space (lobby, conference room, gym, etc.).<br>Only applicable to a TR area — do not mark a Tenant Suite as a Common Area. |

### Column Groupings (as presented on the worksheet)

* **Mandatory:** Property Name, Building Name, Floor Name
* **Optional:** Suite Name
* **Mandatory:** Is TR Area, Is PM Area, Is Common Area

---

## Validation Rules

### Required Values

Report an **Error** for any row where the following are blank:

* Property Name
* Building Name
* Floor Name
* Is TR Area / Is PM Area / Is Common Area (see flag rules below — the flag group must be satisfied)

### Maximum Length

Report an **Error** where the supplied value exceeds the stated length:

| Column | Max Length |
| ------ | ---------: |
| Property Name | 50 |
| Building Name | 50 |
| Floor Name | 50 |
| Suite Name | 50 |
| Is TR Area | 4 |
| Is PM Area | 4 |
| Is Common Area | 4 |

### Allowed Values

`Is TR Area`, `Is PM Area` and `Is Common Area` must contain either `X` (case-insensitive)
or be blank. Any other value is an **Error**.

### Flag Combination Rules

1. If `Is Common Area` is blank, then at least one of `Is TR Area` or `Is PM Area` must
   be marked with `X`. Otherwise report an **Error**.
2. The combination of `Is TR Area`, `Is PM Area` and `Is Common Area` must be unique for
   the area. Report an **Error** on conflicting combinations.
3. `Is Common Area` should only be used for shared space (lobby, conference room, gym,
   restroom, etc.). A Tenant Suite must **not** be marked as a Common Area — report a
   **Warning** where a suite that looks tenant-occupied is flagged as a Common Area.

### Uniqueness Rules

1. **Floor Name** must be unique within a Building Name. Duplicate Floor Name values for
   the same Property Name + Building Name are an **Error**.
2. **Suite Name** must be unique to the floor, i.e. unique for the
   Property Name + Building Name + Floor Name combination. Duplicates are an **Error**.
3. Suites should only be entered for unoccupied spaces or common areas. A populated
   Suite Name on an occupied tenant row should be raised as a **Warning**.

### Cross-Reference Rules

* `Property Name` and `Building Name` must match existing values in MRI Angus.
  Without database access this cannot be confirmed — report as
  **Warning / REQUIRES DATABASE VERIFICATION**.
* `Floor Name` values on this sheet are the authoritative list referenced by the
  `Tenant` and `Contact` sheets.

---

## Sample Data (from the collection sheet)

| # | Property Name | Building Name | Floor Name | Suite Name | Is TR Area | Is PM Area | Is Common Area |
| - | ------------- | ------------- | ---------- | ---------- | ---------- | ---------- | -------------- |
| 1 | 123 Main Street | 123 Main Street | 01 | Lobby | | | X |
| 2 | 123 Main Street | 123 Main Street | 02 | Conference Room | | | X |
| 3 | 123 Main Street | 123 Main Street | 03 | Restroom | | | X |
| 4 | 123 Main Street | 123 Main Street | 04 | Janitors Room | X | | |
| 5 | 123 Main Street | 123 Main Street | 05 | Electrical Room | X | | |
| 6 | 123 Main Street | 123 Main Street | 06 | 601 | | X | |
| 7 | 123 Main Street | 123 Main Street | 07 | 701 | X | | |

---

## Worksheet Guidance Notes

* Suites should be unique to the floor.

## Type Legend

| Code | Meaning |
| ---- | ------- |
| A | Alpha-numeric field |
| D | Date field — format `dd/mm/yyyy` is acceptable |
| N | Numeric field |
