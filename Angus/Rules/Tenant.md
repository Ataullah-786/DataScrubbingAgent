# Angus — Tenant Rules

**Product:** Angus
**Target Table:** `Tenant`
**Schema File:** `/Angus/Schema/Tenant.json`
**Source Workbook:** MRI Angus Data Collection Sheet
**Source Worksheet:** `Tenant`

These rules are taken from the MRI Angus Data Collection Sheet. They describe how the
intake (collection sheet) columns must be populated before the file can be integrated
into the Angus `Tenant` table.

They are **additional to** the structural rules in `/Angus/Schema/Tenant.json`.
Where the two disagree on data type or length, the JSON schema remains the source of
truth for the physical database, and the rules below define the business/intake expectation.

---

## Field Rules

| Intake Column (Database Table Name) | MRI Angus Field Name | Length | Type | Mandatory | Rule |
| ----------------------------------- | -------------------- | -----: | ---- | --------- | ---- |
| Property Name | Property | 50 | Text (A) | **YES** | Must match MRI Angus |
| Building Name | Building Name | 50 | Text (A) | **YES** | Must match MRI Angus |
| Tenant Company Name | Tenant Name | 50 | Text (A) | **YES** | — |
| Floor Name | Floor | 50 | Text (A) | **YES** | Must match Floor Name on the Area tab |
| Suite Name | Location / Suite | 50 | Text (A) | **YES** | — |
| Tenant Company Phone | Phone | 50 | Text (A) | No (Optional) | Only one phone number |
| Tenant Fax | Fax | 50 | Text (A) | No (Optional) | Only one fax number |
| Tenant Notes | Notes | 2048 | Text (A) | No (Optional) | — |

### Column Groupings (as presented on the worksheet)

* **Mandatory for Tenants:** Property Name, Building Name, Tenant Company Name, Floor Name, Suite Name
* **Optional for Tenants:** Tenant Company Phone, Tenant Fax, Tenant Notes

---

## Validation Rules

### Required Values

Report an **Error** for any row where the following are blank:

* Property Name
* Building Name
* Tenant Company Name
* Floor Name
* Suite Name

### Maximum Length

Report an **Error** where the supplied value exceeds the stated length:

| Column | Max Length |
| ------ | ---------: |
| Property Name | 50 |
| Building Name | 50 |
| Tenant Company Name | 50 |
| Floor Name | 50 |
| Suite Name | 50 |
| Tenant Company Phone | 50 |
| Tenant Fax | 50 |
| Tenant Notes | 2048 |

### Single-Value Rules

* `Tenant Company Phone` must contain **only one** phone number. Multiple numbers
  (comma, slash, semicolon or "/" separated, or additional numbers in free text) are an
  **Error**.
* `Tenant Fax` must contain **only one** fax number. Multiple values are an **Error**.

### Row Structure Rules

* One tenant occupies one row per floor. If a tenant is on multiple floors, the tenant
  must be entered on a **separate row for each different floor**. Report a **Warning**
  where multiple floors appear to be combined in a single row (e.g. `01, 02` in
  Floor Name).
* A tenant occupying multiple suites on the same floor is entered on one row per suite
  (see Sample 1 and Sample 2 below).

### Cross-Reference Rules

* `Floor Name` must match a Floor Name present on the **Area** worksheet
  (`/Angus/Rules/Area.md`). A value not present there is an **Error** when the Area file
  is available for comparison; otherwise report a **Warning**.
* `Suite Name` should correspond to a suite/location defined for the same
  Property Name + Building Name + Floor Name combination.
* `Property Name` and `Building Name` must match existing values in MRI Angus. Without
  database access this cannot be confirmed — report as
  **Warning / REQUIRES DATABASE VERIFICATION**.

### External References — Availability

Not every entity referenced by this worksheet is held in this repository. Before
reporting a referential issue, check whether the reference can actually be validated.
Never state that a reference is valid or invalid when the referenced data is not
available.

| Referenced | Used By | Held In This Repo? | How To Validate |
| ---------- | ------- | ------------------ | --------------- |
| Area worksheet | `Floor Name`, `Suite Name` | **Yes** — `/Angus/Rules/Area.md` | Validate against the supplied Area file when one is provided (**Error** if the value is not present). Otherwise **Warning / REQUIRES DATABASE VERIFICATION**. |
| Property (MRI Angus) | `Property Name` | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION** — do not report as an error. |
| Building (MRI Angus) | `Building Name` | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION** — do not report as an error. |

> **Note:** Property and Building are referenced by this worksheet but are **not**
> supported tables in this repository — there are no schema or rules files for them.
> Any check against them requires access to the target database and must be reported
> as `REQUIRES DATABASE VERIFICATION`.

---

## Sample Data (from the collection sheet)

| # | Property Name | Building Name | Tenant Company Name | Floor Name | Suite Name | Tenant Company Phone |
| - | ------------- | ------------- | ------------------- | ---------- | ---------- | -------------------- |
| 1 | 123 Main Street | 123 Main Street | Tenant A | 01 | 100 | 555-555-5555 |
| 2 | 123 Main Street | 123 Main Street | Tenant A | 01 | 101 | 555-555-5555 |
| 3 | 123 Main Street | 123 Main Street | Tenant B | 02 | 200 | 444-444-4444 |
| 4 | 123 Main Street | 123 Main Street | Tenant B | 02 | 201 | 444-444-4444 |
| 5 | 123 Main Street | 123 Main Street | Tenant B | 02 | 202 | 444-444-4444 |
| 6 | Property B | Building B1 | Tenant C | 03 | 301 | 333-333-3333 |
| 7 | Property B | Building B2 | Tenant C | 04 | 402 | 333-333-3333 |

---

## Worksheet Guidance Notes

* If a tenant is on multiple floors, enter the tenant on a separate row for each
  different floor.

## Source Notes

* The worksheet lists the Type for `Floor Name` as "Test"; this is a typo on the
  collection sheet and is treated as **Text**.

## Type Legend

| Code | Meaning |
| ---- | ------- |
| A | Alpha-numeric field |
| D | Date field — format `dd/mm/yyyy` is acceptable |
| N | Numeric field |
