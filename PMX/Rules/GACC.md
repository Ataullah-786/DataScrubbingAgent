# PMX — GACC Rules

**Product:** PMX  
**Target Table:** `GACC`  
**Schema File:** `/PMX/Schema/GACC.json`  
**Source Workbook:** Import Tables - All Modules  
**Source Worksheet:** `GACC`

These rules are taken from the MRI PMX import specification workbook. They describe
how the import file columns must be populated before the file can be integrated into
the PMX `GACC` table.

They are **additional to** the structural rules in `/PMX/Schema/GACC.json`.
Where the two disagree on data type or length, the JSON schema remains the source of
truth for the physical database, and the rules below define the business/import
expectation.

> The `GACC` worksheet defines the General Ledger chart of accounts. Account numbers
> defined here are referenced by the `BMAP` and `ENTITY` worksheets.

---

## Field Rules

| # | MRI Table Field Name | PMX Field Name | Type | Length | Required | Client Specific Notes |
| -: | -------------------- | -------------- | ---- | -----: | -------- | --------------------- |
| 1 | `ACCTNUM` | Account Number | A | 9 | **Y** | First two characters of the ledger code |
| 2 | `ACCTNAME` | Account Name | A | 25 | **Y** | — |
| 3 | `TYPE` | Account Type | A | 1 | **Y** | (B)alance Sheet <br>(C)ash Account <br>(I)ncome Statement <br>(L) Memo Balance Sheet (M)emo Income Statament |
| 4 | `M_1099ACCT` | 1099 Account | A | 1 | **Y** | Y/N |
| 5 | `ACTIVE` | Active Flag | A | 1 | **Y** | Y/N |
| 6 | `LASTDATE` | Last Update | D | — | N | Always SYSDATE |
| 7 | `USERID` | User Id | A | 20 | N | Always CONV |
| 8 | `DPRSTR` | DP Restrictions | A | 1 | N | — |
| 9 | `PEXCHTYPE` | Period Exchange Type | A | 1 | N | — |
| 10 | `OWNERTAX` | !{Ownership} !{Tax} Type | A | 1 | N | Ownership tax types are used to calculate tax on the revenue and expenses of an owner who lives abroad. |
| 11 | `SUBWITH` | Subject to Sub-Contractor Withholding | A | 1 | N | Y/N |
| 12 | `ACCTBASIS` | Accounting Basis | A | 1 | N | (A)ccrual <br>(B)oth <br>(C)ash <br>(P) Client Preference |
| 13 | `LEGALACCT` | Legal Account? | A | 1 | N | Y or N - This setting is only valid if your system is set up to use alternate journal entry numbering schemes and you are required to report fiscal activity to auditors using the government-provided account numbers (also known as national account numbers). Used in |

**Required fields:** 5  
**Optional fields:** 8  
**Total fields:** 13

---

## Validation Rules

### Required Values

Report an **Error** for any row where the following fields are blank:

* `ACCTNUM` — Account Number
* `ACCTNAME` — Account Name
* `TYPE` — Account Type
* `M_1099ACCT` — 1099 Account
* `ACTIVE` — Active Flag

All other fields on this worksheet are optional and may be left blank.

### Maximum Length

Report an **Error** where a supplied value exceeds the stated length. A length of `0`
on the worksheet means the field is not a fixed-width character field (date or
boolean) and no character limit applies.

| MRI Table Field Name | Type | Max Length |
| -------------------- | ---- | ---------: |
| `ACCTNUM` | A | 9 |
| `ACCTNAME` | A | 25 |
| `TYPE` | A | 1 |
| `M_1099ACCT` | A | 1 |
| `ACTIVE` | A | 1 |
| `LASTDATE` | D | n/a |
| `USERID` | A | 20 |
| `DPRSTR` | A | 1 |
| `PEXCHTYPE` | A | 1 |
| `OWNERTAX` | A | 1 |
| `SUBWITH` | A | 1 |
| `ACCTBASIS` | A | 1 |
| `LEGALACCT` | A | 1 |

### Data Type Expectations

* **A — Alpha-numeric:** `ACCTNUM`, `ACCTNAME`, `TYPE`, `M_1099ACCT`, `ACTIVE`, `USERID`, `DPRSTR`, `PEXCHTYPE`, `OWNERTAX`, `SUBWITH`, `ACCTBASIS`, `LEGALACCT`
* **D — Date:** `LASTDATE`

Report an **Error** where a value cannot be represented as its declared type — for
example non-numeric text in an `N` field, or an unparseable date in a `D` field.
Dates in `dd/mm/yyyy` format are acceptable.

### Allowed Values

The following fields accept only `Y` or `N`. Any other value is an **Error**.

* `M_1099ACCT` — 1099 Account
* `ACTIVE` — Active Flag
* `SUBWITH` — Subject to Sub-Contractor Withholding
* `LEGALACCT` — Legal Account?

**`TYPE` — Account Type**

| Value | Meaning |
| ----- | ------- |
| `B` | Balance Sheet |
| `C` | Cash Account |
| `I` | Income Statement |
| `L` | Memo Balance Sheet |
| `M` | Memo Income Statament |

Any value outside this list is an **Error**.

**`ACCTBASIS` — Accounting Basis**

| Value | Meaning |
| ----- | ------- |
| `A` | Accrual |
| `B` | Both |
| `C` | Cash |
| `P` | Client Preference |

Any value outside this list is an **Error**.

### Fixed Values

These fields are populated by the conversion process and must carry a fixed value.

* `LASTDATE` — must always be `SYSDATE`. Any other value is an **Error**.
* `USERID` — must always be `CONV`. Any other value is an **Error**.

### Format Rules

* `ACCTNUM` — First two characters of the ledger code

Report an **Error** where a value does not conform to the stated format.

### Other Field Notes

Remaining guidance carried over from the worksheet.

* `OWNERTAX` — Ownership tax types are used to calculate tax on the revenue and expenses of an owner who lives abroad.

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
