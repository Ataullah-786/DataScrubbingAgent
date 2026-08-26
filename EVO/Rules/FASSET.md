# EVO — FASSET Rules

| Item | Value |
| --- | --- |
| Product | EVO (Concept Evolution) |
| Target table | `FASSET` |
| Schema file | `EVO/Schema/FASSET.json` |
| Source workbook | MRI Software - Data Capture Sheet v2 AI (1).xlsx |
| Source worksheet | `Assets` |
| Worksheet title | Assets |
| Worksheet description | Every asset that will be tracked and maintained through Concept Evolution must be defined in the database. |
| Intake columns documented | 27 |

> The worksheet supplies business-facing intake labels, not `FASSET` column
> names. Map intake columns to physical columns using `EVO/Schema/FASSET.json`
> before validating. Do not assume a one-to-one name match.

## Field Rules

| # | Sheet Col | Intake Field Name | Type | Character Limit | Requirement |
| --- | --- | --- | --- | --- | --- |
| 1 | A | AR_SEQ | — | — | Compulsory |
| 2 | B | Asset Code | Text (Unique Value) | 32 | Compulsory |
| 3 | C | Asset Description | Text | 256 | Compulsory |
| 4 | D | Quantity | Numeric | — | Compulsory |
| 5 | E | Asset System | (Select from Drop Down) | — | Recommended |
| 6 | F | Asset Tag | (Select from Drop Down) | — | Compulsory |
| 7 | G | Asset Type | (Select from Drop Down) | — | Client discretion |
| 8 | H | Asset Name | (Select from Drop Down) | — | Client discretion |
| 9 | I | Group | Text | 64 | Client discretion |
| 10 | J | Classification | (Select from Drop Down) | — | Client discretion |
| 11 | K | Condition | (Select from Drop Down) | — | Client discretion |
| 12 | L | Site - Building - Floor - Location - Room Number | (Select from Drop Down) | — | Compulsory |
| 13 | M | Serial Number | Text | 64 | Client discretion |
| 14 | N | Product Code | Text | 64 | Client discretion |
| 15 | O | Drawing Reference Number | Text | 64 | Client discretion |
| 16 | P | Barcode | Text | 32 | Client discretion |
| 17 | Q | Model Type | Text | 64 | Client discretion |
| 18 | R | Manufacturer Name | Text | 64 | Client discretion |
| 19 | S | AR_PARENT | (Select from Drop Down) | — | Client discretion |
| 20 | T | Parent Asset Code | (Select from Drop Down) | — | Client discretion |
| 21 | U | Purchase Date | Date (dd/mm/yyyy) | — | Client discretion |
| 22 | V | Purchase Cost | Numeric | — | Client discretion |
| 23 | W | Warranty Expiry Date | Date (dd/mm/yyyy) | — | Client discretion |
| 24 | X | Asset Tested Date | Date (dd/mm/yyyy) | — | Client discretion |
| 25 | Y | Supplier Name | (Select from Drop Down) | — | Recommended |
| 26 | Z | Cost Centre Name | (Select from Drop Down) | — | Recommended |
| 27 | AA | Comments | Text | 2000 | Client discretion |

**Requirement counts:** Client discretion = 18, Compulsory = 6, Recommended = 3.

## Requirement Classification

The workbook encodes requirement level as **font colour on the field-name row**, not
as a text value. The `Menu` sheet states the key verbatim:

- `Tabs in RED are compulsory`
- `Tabs in BLUE are recommended`
- `Tabs in BLACK are at the clients discretion`
- `Columns in RED are compulsory if the Tab is being used`
- `Auto Generated DO NOT TOUCH` (green)

Applied to this worksheet:

- **Compulsory** — must be present and non-blank on every submitted row:
  `AR_SEQ`, `Asset Code`, `Asset Description`, `Quantity`, `Asset Tag`, `Site - Building - Floor - Location - Room Number`
- **Recommended** — should be supplied; absence is a warning, not a failure:
  `Asset System`, `Supplier Name`, `Cost Centre Name`
- **Client discretion** — optional; validate format only when a value is present:
  `Asset Type`, `Asset Name`, `Group`, `Classification`, `Condition`, `Serial Number`, `Product Code`, `Drawing Reference Number`, `Barcode`, `Model Type`, `Manufacturer Name`, `AR_PARENT`, `Parent Asset Code`, `Purchase Date`, `Purchase Cost`, `Warranty Expiry Date`, `Asset Tested Date`, `Comments`

## Maximum Length Rules

The `Menu` sheet states verbatim:

> The numbers that appear in row 5 on the sheets indicate the character limit.
> These can be exceeded by pasting values into them BUT they WILL be rejected by
> the import procedure.

Reject any value exceeding the stated limit.

| Intake Field Name | Max Characters |
| --- | --- |
| Asset Code | 32 |
| Asset Description | 256 |
| Group | 64 |
| Serial Number | 64 |
| Product Code | 64 |
| Drawing Reference Number | 64 |
| Barcode | 32 |
| Model Type | 64 |
| Manufacturer Name | 64 |
| Comments | 2000 |

Fields with no stated limit in the worksheet must be length-checked against
`EVO/Schema/FASSET.json` instead.

## Allowed Values

Values enforced by an in-workbook dropdown:

| Intake Field Name | Source | Permitted Values |
| --- | --- | --- |
| Parent Asset Code | `$B$7:$B$659` | In-sheet range `$B$7:$B$659` — resolve against the submitted file |

The following fields are marked `(Select from Drop Down)` on the type row but the
workbook does not carry an in-sheet value list for them. Their permitted values are
held in the client's EVO configuration and **cannot be verified from this repository**:

- `Asset System`
- `Asset Tag`
- `Asset Type`
- `Asset Name`
- `Classification`
- `Condition`
- `Site - Building - Floor - Location - Room Number`
- `AR_PARENT`
- `Supplier Name`
- `Cost Centre Name`

Treat unknown values in these fields as **Review**, not **Fail**.

## Data Type Expectations

| Type Marker | Expectation |
| --- | --- |
| `(Select from Drop Down)` | Restricted list value. See **Allowed Values**. |
| `Date (dd/mm/yyyy)` | Date in `dd/mm/yyyy` form. |
| `Numeric` | Whole number. Reject non-numeric characters. |
| `Text` | Free text, subject to the character limit above. |
| `Text (Unique Value)` | Free text; the value must be unique across the whole file. |

## Uniqueness Rules

- `Asset Code` is marked `Text (Unique Value)`. No two rows in the submitted file may
  share the same value. Duplicates are a **Fail**.

## Row Structure Rules

- Field names are on **row 6**. Data starts at **row 7**.
- The type marker is on **row 4**; the character limit is on **row 5**.
- Rows shipped with the workbook are worked examples. Columns marked `END` and
  `EXAMPLE` (and the `char limit exceed` / `blank compulsories` / `Char limit count` /
  `Blank compulsory count` / `Error Check run` helper columns to the right of the data)
  are workbook scaffolding and must be excluded from the import.
- Do not reorder, rename, insert or delete columns. Column position is significant.
- A row is in scope only when at least one non-helper column carries a value.

## External References — Availability

Several fields on this worksheet resolve against lookups held outside this
repository. State the availability of each reference before reporting a result.

| Intake Field Name | Reference Target | Held in this repo? | Notes |
| --- | --- | --- | --- |
| Parent Asset Code | `FASSET` | Yes — `EVO/Schema/FASSET.json` | Self-reference. Parent must exist as an `Asset Code` earlier in the same file or already in EVO. |
| Asset System | EVO configuration list | No | Asset system list. |
| Asset Tag | EVO configuration list | No | Asset tag list. |
| Asset Type | EVO configuration list | No | Asset type list. |
| Asset Name | EVO configuration list | No | Asset name list. |
| Classification | EVO configuration list | No | Asset classification list. |
| Condition | EVO configuration list | No | Asset condition list. |
| Site - Building - Floor - Location - Room Number | EVO configuration list | No | Location hierarchy. |
| Supplier Name | EVO configuration list | No | Supplier / contractor list. |
| Cost Centre Name | EVO configuration list | No | Cost centre list. |

**9 of 10 references cannot be resolved from this repository.**
For those fields:

- Do **not** report the value as valid or invalid.
- Report it as **Review — reference not available**.
- State explicitly that confirming it requires either the corresponding schema and
  rules files being added to this repository, or a live database check.

## Sample Data

Worked examples shipped with the worksheet, reproduced verbatim.

| Intake Field Name | Sample 1 | Sample 2 | Sample 3 | Sample 4 |
| --- | --- | --- | --- | --- |
| AR_SEQ | *(blank)* | *(blank)* | *(blank)* | *(blank)* |
| Asset Code | NWTC/TC/TEL/CC/001 | NWTC/TC/TEL/CC/002 | NWTC/HVAC/BOI/001 | ENWLB002/ELEC/DISB/0002 |
| Asset Description | Meeting Room 1 Chair Person Phone | Meeting Room 1 Secondary Phone | Boiler 1 | 48 WAY DISTRIBUTION BOARD - NO RCD FITTED |
| Quantity | 1 | 2 | 1 | 1 |
| Asset System | TELECOMS | TELECOMS | HVAC | ELEC |
| Asset Tag | TELEPHONE | TELEPHONE | BOILER | ELEC |
| Asset Type | Conference Calling Phone | Conference Calling Phone | *(blank)* | *(blank)* |
| Asset Name | MR 1 PHN | MR 1 PHN | *(blank)* | *(blank)* |
| Group | *(blank)* | *(blank)* | *(blank)* | *(blank)* |
| Classification | Classification | *(blank)* | *(blank)* | *(blank)* |
| Condition | GOOD | *(blank)* | *(blank)* | GOOD |
| Site - Building - Floor - Location - Room Number | Upminster - TEST MRI Building - Basement - Boiler Room () | Upminster - TEST MRI Building - Ground - Foyer () | Upminster - TEST MRI Building - Ground - Foyer () | MRI UK - 9 King Street - G - Reception (A0.01) |
| Serial Number | 1234 | *(blank)* | *(blank)* | *(blank)* |
| Product Code | 5678 | *(blank)* | *(blank)* | *(blank)* |
| Drawing Reference Number | DWG001 | *(blank)* | *(blank)* | *(blank)* |
| Barcode | 111111 | *(blank)* | *(blank)* | *(blank)* |
| Model Type | Test Model | *(blank)* | *(blank)* | *(blank)* |
| Manufacturer Name | Test Manufacturer | *(blank)* | *(blank)* | *(blank)* |
| AR_PARENT | Parent | Child | *(blank)* | *(blank)* |
| Parent Asset Code | *(blank)* | NWTC/TC/TEL/CC/001 | *(blank)* | *(blank)* |
| Purchase Date | *(blank)* | *(blank)* | *(blank)* | *(blank)* |
| Purchase Cost | 73.99 | *(blank)* | *(blank)* | 10000 |
| Warranty Expiry Date | *(blank)* | *(blank)* | *(blank)* | 31/12/2030 |
| Asset Tested Date | *(blank)* | *(blank)* | *(blank)* | *(blank)* |
| Supplier Name | In-House | In-House | In-House | *(blank)* |
| Cost Centre Name | Essex House | Essex House | Essex House | *(blank)* |
| Comments | Speaker phone button faulty | *(blank)* | Example Boiler | *(blank)* |

A blank sample cell is not evidence that the field is optional — check the
**Requirement Classification** section instead.

## Worksheet Guidance Notes

The following notes are reproduced verbatim from the `Menu` sheet of the source workbook:

> Tabs in RED are compulsory
> Tabs in BLUE are recommended
> Tabs in BLACK are at the clients discretion
> Maximum field Type and length
> Auto Generated DO NOT TOUCH
> Select from dropdown ONLY - do not copy / paste, type or amend the values
> Columns in RED are compulsory if the Tab is being used
> The numbers that appears in row 5 on the sheets indicate the character limit. These
> can be exceeded by pasting values into them BUT they WILL be rejected by the import
> procedure.

## Source Notes

- The worksheet is named `Assets`. It maps to `FASSET` because column A is literally headed `AR_SEQ` and column S `AR_PARENT`, matching the `AR_` column prefix used throughout `EVO/Schema/FASSET.json`.
- `AR_SEQ` (column A) is the physical primary key and is coloured red on the worksheet, but it carries no sample values. Treat it as system-assigned: do not require the client to populate it, and do not scrub it.
- `AR_PARENT` (column S) holds the literal words `Parent` / `Child` in the samples rather than a key value; the actual parent key is carried in `Parent Asset Code` (column T).
- Column `Asset Description` states a character limit of `256` on row 5, but a stale data-validation rule on row 660 of the same column specifies `100`. The `Menu` sheet designates row 5 as the authoritative limit, so `256` is used above. Flag the discrepancy rather than resolving it silently.
- The type row marks `Classification`, `Condition`, `Asset System`, `Asset Tag`, `Asset Type`, `Asset Name`, `Group`, `Supplier Name` and `Cost Centre Name` as dropdowns, but the workbook carries no value lists for them.
- Sample row 1 contains the literal placeholder text `Classification` in the `Classification` column. This is workbook filler, not a real value.

## Validation Reporting

- **Fail** — a compulsory field is blank, a length limit is exceeded, a unique
  value is duplicated, or a value is outside a dropdown list that is fully
  defined in this file.
- **Warning** — a recommended field is blank, or a derived column has been
  hand-populated.
- **Review** — a value depends on a lookup that is not held in this repository,
  or a field appears in the schema but is not documented on the worksheet.
