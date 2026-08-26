# EVO — F_EVENTS Rules

| Item | Value |
| --- | --- |
| Product | EVO (Concept Evolution) |
| Target table | `F_EVENTS` |
| Schema file | `EVO/Schema/F_EVENTS.json` |
| Source workbook | MRI Software - Data Capture Sheet v2 AI (1).xlsx |
| Source worksheet | `Events` |
| Worksheet title | Call Events |
| Worksheet description | Call events describe events which happen during the life cycle of a task |
| Intake columns documented | 4 |

> The worksheet supplies business-facing intake labels, not `F_EVENTS` column
> names. Map intake columns to physical columns using `EVO/Schema/F_EVENTS.json`
> before validating. Do not assume a one-to-one name match.

## Field Rules

| # | Sheet Col | Intake Field Name | Type | Character Limit | Requirement |
| --- | --- | --- | --- | --- | --- |
| 1 | A | Name | Text | 100 | Compulsory |
| 2 | B | Mitigating | (Select from Drop Down) | Yes / No | Recommended |
| 3 | C | Available in Connect | (Select from Drop Down) | Yes / No | Recommended |
| 4 | D | Available in Reach | (Select from Drop Down) | Yes / No | Recommended |

**Requirement counts:** Compulsory = 1, Recommended = 3.

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
  `Name`
- **Recommended** — should be supplied; absence is a warning, not a failure:
  `Mitigating`, `Available in Connect`, `Available in Reach`

## Maximum Length Rules

The `Menu` sheet states verbatim:

> The numbers that appear in row 5 on the sheets indicate the character limit.
> These can be exceeded by pasting values into them BUT they WILL be rejected by
> the import procedure.

Reject any value exceeding the stated limit.

| Intake Field Name | Max Characters |
| --- | --- |
| Name | 100 |

Fields with no stated limit in the worksheet must be length-checked against
`EVO/Schema/F_EVENTS.json` instead.

## Allowed Values

Values enforced by an in-workbook dropdown:

| Intake Field Name | Source | Permitted Values |
| --- | --- | --- |
| Mitigating | `YesNo` | `Yes`, `No` |
| Available in Connect | `YesNo` | `Yes`, `No` |
| Available in Reach | `YesNo` | `Yes`, `No` |

## Data Type Expectations

| Type Marker | Expectation |
| --- | --- |
| `(Select from Drop Down)` | Restricted list value. See **Allowed Values**. |
| `Text` | Free text, subject to the character limit above. |

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

This worksheet contains no cross-table references. Every value on it is either
free text or resolved from an in-workbook dropdown, so it can be validated in
full from this repository.

## Sample Data

Worked examples shipped with the worksheet, reproduced verbatim.

| Intake Field Name | Sample 1 | Sample 2 | Sample 3 | Sample 4 |
| --- | --- | --- | --- | --- |
| Name | Quote Issued | Job Accepted | Parts Chased | Customer Call |
| Mitigating | No | No | Yes | No |
| Available in Connect | No | No | No | Yes |
| Available in Reach | No | No | No | Yes |

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

- The worksheet is named `Events` and titled `Call Events`. It has been mapped to `F_EVENTS` on the basis of the name and description; the workbook does not state a database table name, so **this mapping is inferred and should be confirmed before use**.
- The worksheet has 4 columns while `EVO/Schema/F_EVENTS.json` has 42. The worksheet therefore describes a *narrow event-type lookup* — the set of call-event types that may be raised — rather than the full transactional event record. Do not treat the absence of a column here as evidence that the underlying column is unused.
- Because the worksheet covers only a subset of the table, always read `EVO/Schema/F_EVENTS.json` for any field not listed above, and report unlisted fields as **Review — no documented rule**.
- Rows 11–19 of the worksheet contain only the `END` marker and no data. They are scaffolding, not blank records.

## Validation Reporting

- **Fail** — a compulsory field is blank, a length limit is exceeded, a unique
  value is duplicated, or a value is outside a dropdown list that is fully
  defined in this file.
- **Warning** — a recommended field is blank, or a derived column has been
  hand-populated.
- **Review** — a value depends on a lookup that is not held in this repository,
  or a field appears in the schema but is not documented on the worksheet.
