# PMX — BMAP Rules

**Product:** PMX  
**Target Table:** `BMAP`  
**Schema File:** `/PMX/Schema/BMAP.json`  
**Source Workbook:** Import Tables - All Modules  
**Source Worksheet:** `BMAP`

These rules are taken from the MRI PMX import specification workbook. They describe
how the import file columns must be populated before the file can be integrated into
the PMX `BMAP` table.

They are **additional to** the structural rules in `/PMX/Schema/BMAP.json`.
Where the two disagree on data type or length, the JSON schema remains the source of
truth for the physical database, and the rules below define the business/import
expectation.

> The `BMAP` worksheet defines the bank-account mapping used to link an entity, cash
> type and bank to the relevant GL accounts. Almost every field on this worksheet is a
> reference onto another MRI table.

---

## Field Rules

| # | MRI Table Field Name | PMX Field Name | Type | Length | Required | Client Specific Notes |
| -: | -------------------- | -------------- | ---- | -----: | -------- | --------------------- |
| 1 | `ENTITYID` | !{Entity} Id | A | 6 | **Y** | Must match ENTITY.ENTITYID |
| 2 | `CASHTYPE` | !{Cash Type} Id | A | 2 | **Y** | Must match CTYP.CASHTYPE |
| 3 | `BANKID` | !{Bank} Id | A | 6 | **Y** | Must match BANK.BANKID |
| 4 | `ACCTNUM` | Cash Account Number | A | 9 | **Y** | Must match GACC.ACCTNUM |
| 5 | `DISCACCT` | Discount Account | A | 9 | N | Must match GACC.ACCTNUM |
| 6 | `REMIT` | Include in !{Remittance} Cash | A | 1 | **Y** | Y/N |
| 7 | `APACCTNUM` | AP Account Number | A | 9 | N | If populated on entity table, only need here if separate AP account for each bank. Must match GACC.ACCTNUM |
| 8 | `CMNBANKID` | Common !{Bank} Id | A | 6 | N | Must match BANK.BANKID |
| 9 | `RETAINACCT` | Retainage Account Number | A | 9 | N | Must match GACC.ACCTNUM |
| 10 | `VENDORWITHHOLDINGACCT` | Account Number | A | 9 | N | Must match GACC.ACCTNUM |
| 11 | `ACCREXPACCT` | Accrued Expense Account Number | A | 9 | N | Must match GACC.ACCTNUM |

**Required fields:** 5  
**Optional fields:** 6  
**Total fields:** 11

---

## Validation Rules

### Required Values

Report an **Error** for any row where the following fields are blank:

* `ENTITYID` — !{Entity} Id
* `CASHTYPE` — !{Cash Type} Id
* `BANKID` — !{Bank} Id
* `ACCTNUM` — Cash Account Number
* `REMIT` — Include in !{Remittance} Cash

All other fields on this worksheet are optional and may be left blank.

### Maximum Length

Report an **Error** where a supplied value exceeds the stated length. A length of `0`
on the worksheet means the field is not a fixed-width character field (date or
boolean) and no character limit applies.

| MRI Table Field Name | Type | Max Length |
| -------------------- | ---- | ---------: |
| `ENTITYID` | A | 6 |
| `CASHTYPE` | A | 2 |
| `BANKID` | A | 6 |
| `ACCTNUM` | A | 9 |
| `DISCACCT` | A | 9 |
| `REMIT` | A | 1 |
| `APACCTNUM` | A | 9 |
| `CMNBANKID` | A | 6 |
| `RETAINACCT` | A | 9 |
| `VENDORWITHHOLDINGACCT` | A | 9 |
| `ACCREXPACCT` | A | 9 |

### Data Type Expectations

* **A — Alpha-numeric:** `ENTITYID`, `CASHTYPE`, `BANKID`, `ACCTNUM`, `DISCACCT`, `REMIT`, `APACCTNUM`, `CMNBANKID`, `RETAINACCT`, `VENDORWITHHOLDINGACCT`, `ACCREXPACCT`

Report an **Error** where a value cannot be represented as its declared type — for
example non-numeric text in an `N` field, or an unparseable date in a `D` field.
Dates in `dd/mm/yyyy` format are acceptable.

### Allowed Values

The following fields accept only `Y` or `N`. Any other value is an **Error**.

* `REMIT` — Include in !{Remittance} Cash

### Conditional Requirements

These fields are not unconditionally required, but become required — or must be
left blank — depending on how the client system is configured. Report a **Warning**
where the condition cannot be confirmed from the file alone.

* `APACCTNUM` — If populated on entity table, only need here if separate AP account for each bank. Must match GACC.ACCTNUM

### Cross-Reference Rules

The following fields are foreign keys onto other MRI tables. Where the referenced
file is supplied alongside this one, validate the value exists there and report an
**Error** if it does not. Where the referenced data is only available in the target
database, report a **Warning / REQUIRES DATABASE VERIFICATION**.

| MRI Table Field Name | Must Match |
| -------------------- | ---------- |
| `ENTITYID` | `ENTITY.ENTITYID` |
| `CASHTYPE` | `CTYP.CASHTYPE` |
| `BANKID` | `BANK.BANKID` |
| `ACCTNUM` | `GACC.ACCTNUM` |
| `DISCACCT` | `GACC.ACCTNUM` |
| `APACCTNUM` | `GACC.ACCTNUM` |
| `CMNBANKID` | `BANK.BANKID` |
| `RETAINACCT` | `GACC.ACCTNUM` |
| `VENDORWITHHOLDINGACCT` | `GACC.ACCTNUM` |
| `ACCREXPACCT` | `GACC.ACCTNUM` |

### External References — Availability

Not every table referenced by this worksheet is held in this repository. Before
reporting a referential issue, check whether the referenced table can actually be
validated. Never state that a reference is valid or invalid when the referenced data
is not available.

| Referenced | Used By | Held In This Repo? | How To Validate |
| ---------- | ------- | ------------------ | --------------- |
| `BANK.BANKID` | `BANKID`, `CMNBANKID` | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION** — do not report as an error. |
| `CTYP.CASHTYPE` | `CASHTYPE` | **No** | Cannot be validated from this repository. Report as **Warning / REQUIRES DATABASE VERIFICATION** — do not report as an error. |
| `ENTITY.ENTITYID` | `ENTITYID` | **Yes** — `/PMX/Rules/ENTITY.md` | Validate against the supplied `ENTITY` file when one is provided (**Error** if the value is not present). Otherwise **Warning / REQUIRES DATABASE VERIFICATION**. |
| `GACC.ACCTNUM` | `ACCREXPACCT`, `ACCTNUM`, `APACCTNUM`, `DISCACCT`, `RETAINACCT`, `VENDORWITHHOLDINGACCT` | **Yes** — `/PMX/Rules/GACC.md` | Validate against the supplied `GACC` file when one is provided (**Error** if the value is not present). Otherwise **Warning / REQUIRES DATABASE VERIFICATION**. |

> **Note:** `BANK`, `CTYP` are referenced by this worksheet but are **not** supported
> tables in this repository — there are no schema or rules files for them. Any
> check against them requires access to the target database and must be reported
> as `REQUIRES DATABASE VERIFICATION`.

---

## Source Notes

* PMX Field Name values containing `!{...}` (for example `!{Entity} Id`) are MRI
  terminology placeholders. The literal wording shown in the application depends on the
  client's configured terminology set, so match on the MRI Table Field Name rather than
  the displayed label.
* Rule text in the Client Specific Notes column is reproduced from the workbook as
  supplied, including any spelling inconsistencies.
* Only `ENTITY`, `BMAP` and `GACC` are supported PMX tables in this repository. Rules
  that reference other MRI tables cannot be validated from files held here and should
  be reported as requiring database verification.

## Type Legend

| Code | Meaning |
| ---- | ------- |
| A | Alpha-numeric field |
| N | Numeric field |
| D | Date field — format `dd/mm/yyyy` is acceptable |
| B | Boolean / bit field |

A `Length` of `0` indicates no fixed character length applies to the field.
