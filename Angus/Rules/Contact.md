# Angus — Contact Rules

**Product:** Angus
**Target Table:** `Contact`
**Schema File:** `/Angus/Schema/Contact.json`
**Source Workbook:** MRI Angus Data Collection Sheet
**Source Worksheet:** `Contact`

These rules are taken from the MRI Angus Data Collection Sheet. They describe how the
intake (collection sheet) columns must be populated before the file can be integrated
into the Angus `Contact` table.

They are **additional to** the structural rules in `/Angus/Schema/Contact.json`.
Where the two disagree on data type or length, the JSON schema remains the source of
truth for the physical database, and the rules below define the business/intake expectation.

---

## Field Rules

| Intake Column (Database Table Name) | MRI Angus Field Name | Length | Type | Mandatory | Rule |
| ----------------------------------- | -------------------- | -----: | ---- | --------- | ---- |
| Tenant Company Name | Tenant Name | 50 | Text (A) | **YES** | Must match Tenant Company Name on the Tenant tab |
| Property Name | Property Name | 50 | Text (A) | **YES** | Must match MRI Angus |
| Building Name | Building Name | 50 | Text (A) | **YES** | Must match MRI Angus |
| Floor Name | Space | 50 | Text (A) | No (Optional) | Floor where the contact is located |
| Suite/Location Name | Space | 50 | Text (A) | No (Optional) | Suite name where the contact is located |
| Contact First Name | Contact Name | 50 | Text (A) | **YES** | — |
| Contact Last Name | Contact Name | 50 | Text (A) | **YES** | — |
| Contact Title | Title | 50 | Text (A) | No (Optional) | — |
| Contact Email | Email | 128 | Text (A) | **YES** | Only 1 email and cannot be duplicated |
| Contact CC Email | CC Email | 128 | Text (A) | No (Optional) | Only 1 email |
| Contact Phone | Phone | 123 | Text (A) | No (Optional) | Only one phone number and cannot be duplicated |
| Contact Fax | Fax | 128 | Text (A) | No (Optional) | Only 1 fax number |
| Contact Emergency SMS | SMS | 128 | Text (A) | No (Optional) | Only one phone number |
| Contact Emergency Email | Email | 128 | Text (A) | No (Optional) | Only one phone number *(as stated on the sheet; treat as a single email value)* |
| Contact Emergency Phone1 | Phone 1 | 128 | Text (A) | No (Optional) | Only one phone number |
| Contact Emergency Phone2 | Phone 2 | 128 | Text (A) | No (Optional) | Only one phone number |
| Contact Notes | Notes | 2048 | Text (A) | No (Optional) | — |
| Is Administrator | Is Administrator | 4 | Text (A) | No (Optional) | `X` or `x` or leave blank |
| Can Make Requests | Submit Requests | 4 | Text (A) | No (Optional) | `X` or `x` or leave blank |
| Can Host Visitors | Host Visitors | 4 | Text (A) | No (Optional) | `X` or `x` or leave blank |
| Can Make Reservations | Submit Reservations | 4 | Text (A) | No (Optional) | `X` or `x` or leave blank |
| Is COI Subscribed | Tenant COI Subscription | 4 | Text (A) | No (Optional) | `X` or `x` or leave blank |

### Column Groupings (as presented on the worksheet)

* **Mandatory for Contacts:** Tenant Company Name, Property Name, Building Name
* **Optional for Contact:** Floor Name, Suite/Location Name
* **Mandatory:** Contact First Name, Contact Last Name
* **Optional for Contacts:** Contact Title
* **Mandatory:** Contact Email
* **Optional for Contacts:** Contact CC Email, Contact Phone, Contact Fax,
  Contact Emergency SMS, Contact Emergency Email, Contact Emergency Phone1,
  Contact Emergency Phone2, Contact Notes, Is Administrator, Can Make Requests,
  Can Host Visitors, Can Make Reservations, Is COI Subscribed

---

## Validation Rules

### Required Values

Report an **Error** for any row where the following are blank:

* Tenant Company Name
* Property Name
* Building Name
* Contact First Name
* Contact Last Name
* Contact Email

### Maximum Length

Report an **Error** where the supplied value exceeds the stated length:

| Column | Max Length |
| ------ | ---------: |
| Tenant Company Name | 50 |
| Property Name | 50 |
| Building Name | 50 |
| Floor Name | 50 |
| Suite/Location Name | 50 |
| Contact First Name | 50 |
| Contact Last Name | 50 |
| Contact Title | 50 |
| Contact Email | 128 |
| Contact CC Email | 128 |
| Contact Phone | 123 |
| Contact Fax | 128 |
| Contact Emergency SMS | 128 |
| Contact Emergency Email | 128 |
| Contact Emergency Phone1 | 128 |
| Contact Emergency Phone2 | 128 |
| Contact Notes | 2048 |
| Is Administrator | 4 |
| Can Make Requests | 4 |
| Can Host Visitors | 4 |
| Can Make Reservations | 4 |
| Is COI Subscribed | 4 |

### Single-Value Rules

Each of the following must contain exactly **one** value — no lists, no delimiters, no
additional values in free text. Multiple values are an **Error**.

* Contact Email — one email address
* Contact CC Email — one email address
* Contact Phone — one phone number
* Contact Fax — one fax number
* Contact Emergency SMS — one number
* Contact Emergency Email — one value
* Contact Emergency Phone1 — one phone number
* Contact Emergency Phone2 — one phone number

### Uniqueness Rules

* `Contact Email` **cannot be duplicated** across the file. Duplicate email values are an
  **Error** — report every affected row.
* `Contact Phone` **cannot be duplicated** across the file. Duplicate phone values are an
  **Error** — report every affected row.

### Allowed Values (Permission / Flag Columns)

`Is Administrator`, `Can Make Requests`, `Can Host Visitors`, `Can Make Reservations`
and `Is COI Subscribed` must contain either `X` / `x` or be blank. Any other value is an
**Error**.

### Row Structure Rules

* Only put **one floor per tenant** on a row. If a tenant is on multiple floors, enter
  the tenant on multiple rows. Report a **Warning** where multiple floors appear to be
  combined in one Floor Name value.

### Cross-Reference Rules

* `Tenant Company Name` must match a Tenant Company Name on the **Tenant** worksheet
  (`/Angus/Rules/Tenant.md`). A value not present there is an **Error** when the Tenant
  file is available for comparison; otherwise report a **Warning**.
* `Floor Name` and `Suite/Location Name` should correspond to values defined on the
  **Area** worksheet (`/Angus/Rules/Area.md`) for the same Property Name + Building Name.
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
| Tenant worksheet | `Tenant Company Name` | **Yes** — `/Angus/Rules/Tenant.md` | Validate against the supplied Tenant file when one is provided (**Error** if the value is not present). Otherwise **Warning / REQUIRES DATABASE VERIFICATION**. |
| Area worksheet | `Floor Name`, `Suite/Location Name` | **Yes** — `/Angus/Rules/Area.md` | Validate against the supplied Area file when one is provided (**Error** if the value is not present). Otherwise **Warning / REQUIRES DATABASE VERIFICATION**. |
| Property (MRI Angus) | `Property Name` | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION** — do not report as an error. |
| Building (MRI Angus) | `Building Name` | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION** — do not report as an error. |

> **Note:** Property and Building are referenced by this worksheet but are **not**
> supported tables in this repository — there are no schema or rules files for them.
> Any check against them requires access to the target database and must be reported
> as `REQUIRES DATABASE VERIFICATION`.

---

## Sample Data (from the collection sheet)

| # | Tenant Company Name | Property Name | Building Name | Contact First Name | Contact Last Name |
| - | ------------------- | ------------- | ------------- | ------------------ | ----------------- |
| 1 | Tenant A | 123 Main Street | 123 Main Street | Contact First Name | Contact Last Name |
| 2 | Tenant A | 123 Main Street | 123 Main Street | Contact First Name | Contact Last Name |
| 3 | Tenant B | 123 Main Street | 123 Main Street | Contact First Name | Contact Last Name |
| 4 | Tenant B | 123 Main Street | 123 Main Street | Contact First Name | Contact Last Name |
| 5 | Tenant B | 123 Main Street | 123 Main Street | Contact First Name | Contact Last Name |

---

## Worksheet Guidance Notes

* Only put one floor per tenant. If a tenant is on multiple floors, then enter the tenant
  on multiple rows.

## Source Notes

* The worksheet states a character limitation of `123` for `Contact Phone`. This is
  recorded verbatim; it is likely intended to be `128` in line with the surrounding
  contact fields. Confirm with the source system before treating a 124–128 character
  value as an error.
* `Contact Emergency Email` carries the rule text "Only one phone number" on the
  worksheet; this is a copy/paste artefact and is applied as "only one email value".

## Type Legend

| Code | Meaning |
| ---- | ------- |
| A | Alpha-numeric field |
| D | Date field — format `dd/mm/yyyy` is acceptable |
| N | Numeric field |
