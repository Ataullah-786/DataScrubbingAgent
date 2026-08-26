# EVO — F_TASKS Rules

| Item | Value |
| --- | --- |
| Product | EVO (Concept Evolution) |
| Target table | `F_TASKS` |
| Schema file | `EVO/Schema/F_TASKS.json` |
| Source workbook | MRI Software - Data Capture Sheet v2 AI (1).xlsx |
| Source worksheet | `PPM_Tasks` |
| Worksheet title | PPMs |
| Worksheet description | PPMs are frequency-based work-instances that are carried out to prevent the deterioration and breakdown of an asset during its lifetime. More than one PPM schedule may be created against a single asset for tasks that require different frequencies. |
| Intake columns documented | 28 |

> The worksheet supplies business-facing intake labels, not `F_TASKS` column
> names. Map intake columns to physical columns using `EVO/Schema/F_TASKS.json`
> before validating. Do not assume a one-to-one name match.

## Field Rules

| # | Sheet Col | Intake Field Name | Type | Character Limit | Requirement |
| --- | --- | --- | --- | --- | --- |
| 1 | A | Asset Code | (Select from Drop Down) | — | Compulsory |
| 2 | B | Instruction Description | (Select from Drop Down) | — | Compulsory |
| 3 | C | (unnamed column C) | — | — | Auto-generated |
| 4 | D | Class of PPM | (Select from Drop Down) | — | Compulsory |
| 5 | E | Service Frequency | (Select from Drop Down) | — | Compulsory |
| 6 | F | Period Units | (Select from Drop Down) | — | Compulsory |
| 7 | G | Service Frequency (Text) | (Auto Generated) | — | Auto-generated |
| 8 | H | Last Service Date | Date(dd/mm/yyyyy) | — | Compulsory |
| 9 | I | Number of staff required | Decimal | — | Compulsory |
| 10 | J | Priority Description | (Select from Drop Down) | — | Compulsory |
| 11 | K | Generate Task Action? | (Select from Drop Down) | — | Client discretion |
| 12 | L | Must be Actioned before Completion / Sign Off to History | (Select from Drop Down) | — | Client discretion |
| 13 | M | Labour Cost | Estimated labour cost for task | — | Client discretion |
| 14 | N | Stock Cost | Estimated cost of Stock for a task | — | Client discretion |
| 15 | O | Day to Carry out | (Select from Drop Down) | — | Client discretion |
| 16 | P | H & S Task? | (Select from Drop Down) | — | Client discretion |
| 17 | Q | PPM Family | Family For Grouping PPMS Together | 5 | Recommended |
| 18 | R | Cost Centre Name | (Select from Drop Down) | — | Client discretion |
| 19 | S | Cost Code Name | (Select from Drop Down) | — | Client discretion |
| 20 | T | Contract Title | (Select from Drop Down) | — | Compulsory |
| 21 | U | Shift Code | (Select from Drop Down) | — | Client discretion |
| 22 | V | Resource Name 1 | (Select from Drop Down) | — | Client discretion |
| 23 | W | Resource ID | (AUTO-GENERATED) | 10 | Auto-generated |
| 24 | X | Resource Name 2 | (Select from Drop Down) | — | Client discretion |
| 25 | Y | Resource Name 3 | (Select from Drop Down) | — | Client discretion |
| 26 | Z | Resource Name 4 | (Select from Drop Down) | — | Client discretion |
| 27 | AA | Permit to Work? | (Select from Drop Down) | — | Client discretion |
| 28 | AB | Estimated Time for task | Decimal Hours | — | Client discretion |

**Requirement counts:** Auto-generated = 3, Client discretion = 15, Compulsory = 9, Recommended = 1.

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
  `Asset Code`, `Instruction Description`, `Class of PPM`, `Service Frequency`, `Period Units`, `Last Service Date`, `Number of staff required`, `Priority Description`, `Contract Title`
- **Recommended** — should be supplied; absence is a warning, not a failure:
  `PPM Family`
- **Client discretion** — optional; validate format only when a value is present:
  `Generate Task Action?`, `Must be Actioned before Completion / Sign Off to History`, `Labour Cost`, `Stock Cost`, `Day to Carry out`, `H & S Task?`, `Cost Centre Name`, `Cost Code Name`, `Shift Code`, `Resource Name 1`, `Resource Name 2`, `Resource Name 3`, `Resource Name 4`, `Permit to Work?`, `Estimated Time for task`
- **Auto-generated** — derived by the workbook or by EVO — must not be hand-populated:
  `(unnamed column C)`, `Service Frequency (Text)`, `Resource ID`

## Maximum Length Rules

The `Menu` sheet states verbatim:

> The numbers that appear in row 5 on the sheets indicate the character limit.
> These can be exceeded by pasting values into them BUT they WILL be rejected by
> the import procedure.

Reject any value exceeding the stated limit.

| Intake Field Name | Max Characters |
| --- | --- |
| PPM Family | 5 |
| Resource ID | 10 |

Fields with no stated limit in the worksheet must be length-checked against
`EVO/Schema/F_TASKS.json` instead.

## Allowed Values

Values enforced by an in-workbook dropdown:

| Intake Field Name | Source | Permitted Values |
| --- | --- | --- |
| Asset Code | `$B$7:$B$659` | In-sheet range `$B$7:$B$659` — resolve against the submitted file |
| Class of PPM | `PPMClass` | `CalendarPPM`, `ShiftableCalendarPPM`, `ShiftableFamilyPPM` |
| Period Units | `PeriodUnits` | `1` … `52` (52 values) |
| Day to Carry out | `Weekdays` | `Monday`, `Tuesday`, `Wednesday`, `Thursday`, `Friday`, `Saturday`, `Sunday` |
| H & S Task? | `YesNo` | `Yes`, `No` |
| Permit to Work? | `YesNo` | `Yes`, `No` |

The following fields are marked `(Select from Drop Down)` on the type row but the
workbook does not carry an in-sheet value list for them. Their permitted values are
held in the client's EVO configuration and **cannot be verified from this repository**:

- `Instruction Description`
- `Service Frequency`
- `Priority Description`
- `Generate Task Action?`
- `Must be Actioned before Completion / Sign Off to History`
- `Cost Centre Name`
- `Cost Code Name`
- `Contract Title`
- `Shift Code`
- `Resource Name 1`
- `Resource Name 2`
- `Resource Name 3`
- `Resource Name 4`

Treat unknown values in these fields as **Review**, not **Fail**.

## Data Type Expectations

| Type Marker | Expectation |
| --- | --- |
| `(AUTO-GENERATED)` | Derived by the workbook. Must not be hand-populated. |
| `(Auto Generated)` | Derived by the workbook. Must not be hand-populated. |
| `(Select from Drop Down)` | Restricted list value. See **Allowed Values**. |
| `Date(dd/mm/yyyyy)` | Date in `dd/mm/yyyy` form. The worksheet spells this `dd/mm/yyyyy` (five `y`s) — treat as a source typo. |
| `Decimal` | Decimal number. Reject non-numeric characters. |
| `Decimal Hours` | Decimal number expressing hours (e.g. `0.3` = 18 minutes). |
| `Estimated cost of Stock for a task` | Free text as described on the worksheet type row. |
| `Estimated labour cost for task` | Free text as described on the worksheet type row. |
| `Family For Grouping PPMS Together` | Free text as described on the worksheet type row. |

## Derived Columns — Do Not Populate

These columns are calculated inside the workbook. They are reproduced here so that
their contents can be recognised, but they must never be hand-typed or scrubbed as
if they were source data:

| Sheet Col | Intake Field Name | Derivation |
| --- | --- | --- |
| C | (unnamed column C) | `=CONCATENATE(A6," - ",B6)` |
| G | Service Frequency (Text) | `=F6 & " " & E6` |
| W | Resource ID | `=VLOOKUP(V6,Resources!A:B,2,FALSE)` |

## Row Structure Rules

- Field names are on **row 4**. Data starts at **row 6**.
- The type marker is on **row 3**; the character limit is on **row 5**.
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
| Asset Code | `FASSET` | Yes — `EVO/Schema/FASSET.json` | Must match an existing `Asset Code`. Verifiable against `EVO/Schema/FASSET.json` plus `EVO/Rules/FASSET.md`. |
| Priority Description | EVO configuration list | No | Priority list. |
| Cost Centre Name | EVO configuration list | No | Cost centre list. |
| Cost Code Name | EVO configuration list | No | Cost code list. |
| Contract Title | EVO configuration list | No | Contract register. |
| Shift Code | EVO configuration list | No | Shift pattern list. |
| Resource Name 1 | EVO configuration list | No | Resource register. `Resource ID` is looked up from the `Resources` worksheet of the source workbook, which is not held here. |
| Resource Name 2 | EVO configuration list | No | Resource register. |
| Resource Name 3 | EVO configuration list | No | Resource register. |
| Resource Name 4 | EVO configuration list | No | Resource register. |
| PPM Family | EVO configuration list | No | PPM family grouping code. |

**10 of 11 references cannot be resolved from this repository.**
For those fields:

- Do **not** report the value as valid or invalid.
- Report it as **Review — reference not available**.
- State explicitly that confirming it requires either the corresponding schema and
  rules files being added to this repository, or a live database check.

## Sample Data

Worked examples shipped with the worksheet, reproduced verbatim.

| Intake Field Name | Sample 1 | Sample 2 | Sample 3 | Sample 4 |
| --- | --- | --- | --- | --- |
| Asset Code | NWTC/HVAC/BOI/001 | NWTC/TC/TEL/CC/002 | NWTC/TC/TEL/CC/001 | ENWLB002/AIRC/COND/0008 |
| Instruction Description | Roof Access Check 3M | Roof Access Check 6M | Boardroom Deep Clean 1M | 03-01 AIR HANDLING UNITS - GENERAL 12M |
| (unnamed column C) | NWTC/HVAC/BOI/001 - Roof Access Check 3M | NWTC/TC/TEL/CC/002 - Roof Access Check 6M | NWTC/TC/TEL/CC/001 - Boardroom Deep Clean 1M | ENWLB002/AIRC/COND/0008 - 03-01 AIR HANDLING UNITS - GENERAL 12M |
| Class of PPM | CalendarPPM | CalendarPPM | CalendarPPM | CalendarPPM |
| Service Frequency | Months | Months | Months | Months |
| Period Units | 3 | 6 | 1 | 12 |
| Service Frequency (Text) | 3 Months | 6 Months | 1 Months | 12 Months |
| Last Service Date | 01/05/2015 | 01/02/2015 | 01/05/2015 | 01/01/2025 |
| Number of staff required | 1 | 2 | 1 | 1 |
| Priority Description | PPM | PPM | PPM | PPM |
| Generate Task Action? | *(blank)* | All Child Assets | One Level Child Assets | All Child Assets |
| Must be Actioned before Completion / Sign Off to History | N/A | N/A | Completion | History |
| Labour Cost | 26 | 52 | 12 | *(blank)* |
| Stock Cost | 4.82 | 8.41 | 1.34 | *(blank)* |
| Day to Carry out | Monday | Monday | Sunday | *(blank)* |
| H & S Task? | Yes | Yes | No | *(blank)* |
| PPM Family | R001 | R001 | *(blank)* | *(blank)* |
| Cost Centre Name | Essex House | Essex House | Essex House | *(blank)* |
| Cost Code Name | Planned Maintenance | Planned Maintenance | Planned Maintenance | *(blank)* |
| Contract Title | Security | Security | Cleaning | Hard Services |
| Shift Code | DS | ES | DS | *(blank)* |
| Resource Name 1 | KK Security | KK Security | Jerry Doolittle | Siysbonga Gulube |
| Resource ID | KK9291 | KK9291 | MRI00311 | *(blank)* |
| Resource Name 2 | Park Mechanical Solutions | Jerry Doolittle | JD Hardy Cleaning | *(blank)* |
| Resource Name 3 | Jerry Doolittle | Park Mechanical Solutions | Park Mechanical Solutions | *(blank)* |
| Resource Name 4 | JD Hardy Cleaning | JD Hardy Cleaning | KK Security | *(blank)* |
| Permit to Work? | No | No | No | *(blank)* |
| Estimated Time for task | 0.3 | 0.3 | 0.35 | *(blank)* |

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

- The worksheet is named `PPM_Tasks` and titled `PPMs`. It has been mapped to `F_TASKS` on the basis of the description (frequency-based work instances against an asset); the workbook does not state a database table name, so **this mapping is inferred and should be confirmed before use**.
- This worksheet uses a different row layout to the other sheets in the same workbook: the type row is row 3, the field-name row is row 4 and the character-limit row is row 5. Do not assume the `Assets` layout applies.
- The columns `Service Frequency` (E) and `Period Units` (F) appear to be transposed relative to their names. `Service Frequency` holds the unit word (`Months`) and `Period Units` holds the count (`3`, `6`, `12`), and the `Period Units` dropdown is a list of `1`–`52`. The derived column G concatenates them as `F & " " & E`, producing `3 Months`. Validate by content, not by label, and flag the naming as a source anomaly.
- Column C is unlabelled on the field-name row but carries the formula `=CONCATENATE(A6," - ",B6)`. It is a workbook helper, not an intake field.
- `Resource ID` (column W) is a `VLOOKUP` against a `Resources` worksheet in the source workbook. That worksheet is not reproduced here, so `Resource ID` cannot be validated from this repository.
- `Must be Actioned before Completion / Sign Off to History` carries the literal value `N/A` in several samples. `N/A` is a permitted value in the samples, not a missing value.
- The type row describes `Labour Cost` as `Estimated labour cost for task`, `Stock Cost` as `Estimated cost of Stock for a task` and `PPM Family` as `Family For Grouping PPMS Together`. These are descriptions occupying the type row, not type markers; treat all three as numeric / text respectively per the samples.

## Validation Reporting

- **Fail** — a compulsory field is blank, a length limit is exceeded, a unique
  value is duplicated, or a value is outside a dropdown list that is fully
  defined in this file.
- **Warning** — a recommended field is blank, or a derived column has been
  hand-populated.
- **Review** — a value depends on a lookup that is not held in this repository,
  or a field appears in the schema but is not documented on the worksheet.
