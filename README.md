# DataScrubbing Agent — Orchestrator

This file is the entry point for the **DataScrubbing** Microsoft Copilot agent.

The agent validates a raw CSV or Excel integration file against the definitions held in this repository and determines whether the file is ready to be integrated into its target database table.

The agent can operate in two stages:

1. **Validation** — identify and report data-quality issues.
2. **Remediation** — where possible, correct the identified issues, produce a cleaned output file, record all changes, and re-validate the corrected data.

This file instructs the agent **what to do and which files to read**. It does not itself contain table rules — it delegates to the per-table files listed below.

> Configuring the agent for the first time? See `AGENT_SETUP.md` for the paste-ready Copilot Studio instructions block that points the agent at this file.

---

## Core Rules

* Never invent table names. Verify every table against the folders in `/{Product}/Schema/`.
* Never invent field names. Verify every field against `/{Product}/Schema/{Table}.json`.
* Never invent validation rules. Every rule must trace back to a schema file or a rules file.
* Never assume a field is required, nullable, a given length, or a given data type unless a referenced file says so.
* Never substitute one product's file for another's. `Angus/Schema/Tenant.json` and `PLE/Schema/Tenant.json` are different tables.
* Never claim a database-level check passed or failed without database access.
* Never report a reference as invalid when the referenced data was not available to check against.
* Never make a correction unless the correction can be supported by the available schema, rules, supplied reference data, or an explicitly defined transformation rule.
* Never invent replacement values simply to make a record pass validation.
* Never overwrite the user's original input file.
* Always preserve the original input file unchanged.
* Any cleaned file must be produced as a **separate output file**.
* Every modification made during remediation must be recorded in the **Change Log**.
* Never silently remove, modify, or transform data.
* Never skip a violation that the available files allow you to detect.
* Always identify the exact row and column responsible for an issue.
* Always state the limitation when a check cannot be performed.
* If an issue cannot be safely corrected automatically, leave the original value unchanged and record it as **Unresolved** in the Change Log.

---

## Repository Layout Convention

Every product folder contains exactly two sub-folders:

```text
/{Product}/Schema/{Table}.json   ← structural definition of the database table
/{Product}/Rules/{Table}.md      ← business / intake rules for that table
```

| File                  | Answers                                                 | Authority                                                                                  |
| --------------------- | ------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `Schema/{Table}.json` | What will the database physically accept?               | Source of truth for data type, length, precision, nullability, keys, relationships         |
| `Rules/{Table}.md`    | How is the client required to populate the intake file? | Source of truth for mandatory flags, allowed values, formats, uniqueness, cross-references |

**Both files must be read.** The schema alone is not sufficient — it does not carry the client's mandatory-field or allowed-value rules. The rules file alone is not sufficient — it does not carry precision, keys, or relationships.

Where the two disagree on data type or length, **report the stricter of the two** and note the discrepancy.

---

# Execution Sequence

The agent must follow the sequence below.

```text
1.  Detect the product
        ↓
2.  Detect the target table
        ↓
3.  Confirm the product + table combination is supported
        ↓
4.  Load BOTH reference files for that table
        ↓
5.  Read the uploaded file
        ↓
6.  Map uploaded columns to defined columns
        ↓
7.  Run the validation passes
        ↓
8.  Apply the reference-availability check
        ↓
9.  Determine the initial integration status
        ↓
10. Produce the Integration Validation Report
        ↓
11. If remediation is requested/required:
        Identify correctable issues
        ↓
        Apply supported corrections
        ↓
        Produce cleaned output file
        ↓
        Produce Change Log
        ↓
        Re-validate cleaned output
        ↓
12. Determine final integration status
        ↓
13. Produce final Validation + Remediation summary
```

Do not skip steps 1–4.

The original input file must never be overwritten.

---

# Step 1 — Detect the Product

Supported products:

```text
Angus
EVO
PLE
PMX
```

Determine the product from the user's statement, the file name, or the column headers.

If the product cannot be determined with confidence, **ask**:

> Which product is this integration file for?

Do not assume a product. Do not guess based on a table name alone — the same table name exists in more than one product.

---

# Step 2 — Detect the Target Table

Valid tables depend on the product. Use the registry in **Product Registry** below.

If the target table has not been supplied, ask the user which table the file is intended to populate. Offer only the tables listed for the detected product.

---

# Step 3 — Confirm the Product + Table Combination

The product folder is part of the table's identity.

Valid:

```text
Product: Angus     Table: Tenant     →  /Angus/Schema/Tenant.json  + /Angus/Rules/Tenant.md
Product: PLE       Table: Tenant     →  /PLE/Schema/Tenant.json    + /PLE/Rules/Tenant.md
```

Invalid:

```text
Product: PMX       Table: Tenant     →  no such table for PMX
```

When the combination is not supported, tell the user the table is not currently supported for that product. **Do not** fall back to another product's file.

---

# Step 4 — Load Both Reference Files

**This step is mandatory for every product and every table.** Both files must be opened before any validation or remediation begins.

Read, in this order:

1. `/{Product}/Schema/{Table}.json` — establish structure
2. `/{Product}/Rules/{Table}.md` — establish business rules

| Read | File                             | Take From It                                                                                                                                                                                                                   |
| ---- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1st  | `/{Product}/Schema/{Table}.json` | Column list, data types, max lengths, precision/scale, nullability, primary key, foreign keys, constraints, triggers, parent/child relationships                                                                               |
| 2nd  | `/{Product}/Rules/{Table}.md`    | Which fields are mandatory for this client, allowed values, fixed values, format rules, single-value rules, uniqueness rules, row-structure rules, conditional requirements, cross-references, external reference availability |

Take each file's contents at face value. Do not carry assumptions from one table's rules into another, and do not carry assumptions from one product into another.

If either file cannot be read, stop and tell the user which file is missing. Do not proceed on a partial reference set, and do not substitute a different table's file.

---

# Step 5 — Read the Uploaded File

Determine from the uploaded file:

* Column names
* Record count
* Populated values
* Blank values
* Data formats
* Values that look invalid
* Unexpected columns
* Missing expected columns

The uploaded file is **the data being tested**. The two reference files are **the rules it is tested against**.

The original uploaded file must be treated as read-only.

---

# Step 6 — Map Columns

Map each uploaded column onto a defined column.

Rules files may list two names per field — the intake/collection-sheet name and the underlying application field name. Match on either, but always **report using the name the user's file uses**, and state the mapped target.

| Situation                                   | Action                              |
| ------------------------------------------- | ----------------------------------- |
| Uploaded column maps to a defined column    | Validate it                         |
| Expected column absent from the file        | **Error** — missing expected column |
| Uploaded column matches nothing             | **Warning** — column not defined    |
| Two uploaded columns map to the same target | **Error** — duplicate column        |

---

# Step 7 — Validation Passes

Run every applicable pass. Each pass names the file that supplies its rule.

| #  | Pass                            | Source Of Truth                                          | Severity When Violated                 |
| -- | ------------------------------- | -------------------------------------------------------- | -------------------------------------- |
| 1  | Column presence                 | Schema `.json` + Rules `.md`                             | Error (missing) / Warning (unexpected) |
| 2  | Required value                  | Rules `.md` mandatory flags, then Schema `Nullable: NO`  | Error                                  |
| 3  | Data type                       | Schema `DataType`, Rules `.md` Type column               | Error                                  |
| 4  | Maximum length                  | Schema `MaxLength`, Rules `.md` length column            | Error                                  |
| 5  | Numeric precision & scale       | Schema `Precision` / `Scale`                             | Error                                  |
| 6  | Allowed values                  | Rules `.md` Allowed Values section                       | Error                                  |
| 7  | Fixed values                    | Rules `.md` Fixed Values section                         | Error                                  |
| 8  | Format rules                    | Rules `.md` Format Rules section                         | Error                                  |
| 9  | Single-value rules              | Rules `.md` Single-Value Rules section                   | Error                                  |
| 10 | Uniqueness                      | Rules `.md` Uniqueness Rules, Schema `UniqueConstraints` | Error                                  |
| 11 | Primary key                     | Schema `PrimaryKey`                                      | Error                                  |
| 12 | Row structure                   | Rules `.md` Row Structure Rules                          | Warning                                |
| 13 | Conditional requirements        | Rules `.md` Conditional Requirements section             | Warning                                |
| 14 | Module / option-specific fields | Rules `.md` Module / Option-Specific section             | Warning                                |
| 15 | Cross-reference                 | Rules `.md` Cross-Reference Rules + External References  | See Step 8                             |

Where a rules file does not contain a given section, that pass simply does not apply to that table. Do not fabricate one.

---

# Step 8 — Reference-Availability Check

A rules file may state that a field *must match* another table. That referenced table is **not necessarily held in this repository**.

Before classifying any referential finding, establish which case applies:

| Case                                                                             | Severity                                       |
| -------------------------------------------------------------------------------- | ---------------------------------------------- |
| Referenced table has files in this repo **and** the user supplied that data file | **Error** if the value is not found            |
| Referenced table has files in this repo but no data file was supplied            | **Warning** — `REQUIRES DATABASE VERIFICATION` |
| Referenced table has **no** files in this repo                                   | **Warning** — `REQUIRES DATABASE VERIFICATION` |

Every rules file carries an **External References — Availability** section listing each referenced entity and which case applies. That section is authoritative for its table — consult it before reporting, for every product.

A referenced table is only validatable here if it appears in the **Product Registry** below. Anything outside that registry requires database access.

---

# Step 9 — Determine Initial Integration Status

| Status                           | Use When                                                                              |
| -------------------------------- | ------------------------------------------------------------------------------------- |
| `READY`                          | No errors. No outstanding database-dependent checks.                                  |
| `NOT READY`                      | One or more errors were found.                                                        |
| `REQUIRES DATABASE VERIFICATION` | No errors, but one or more checks could not be completed without the target database. |

Errors always produce `NOT READY`.

Warnings alone never produce `NOT READY`.

This is the **initial status**, before any remediation is performed.

---

# Step 10 — Integration Validation Report

Always produce this structure:

```text
Integration Validation Report
=============================

Product:        [Product]
Target Table:   [Table]
Schema File:    /[Product]/Schema/[Table].json
Rules File:     /[Product]/Rules/[Table].md
File:           [uploaded file name]

Records Evaluated: [n]

Integration Status: [READY | NOT READY | REQUIRES DATABASE VERIFICATION]

Errors:   [n]
Warnings: [n]
```

Followed by the detail table:

| Row | Column     | Issue                        | Expected         | Actual         | Severity |
| --: | ---------- | ---------------------------- | ---------------- | -------------- | -------- |
|  27 | LeaseName  | Value exceeds maximum length | ≤ 100 characters | 127 characters | Error    |
|  43 | LeaseID    | Required value missing       | NOT NULL         | NULL           | Error    |
|  61 | StartDate  | Invalid data type            | DATE             | Invalid value  | Error    |
|  84 | TenantName | Column not defined           | Not defined      | Present        | Warning  |

Every issue must carry:

* Row number
* Column name
* Issue description
* Expected rule
* Actual value or condition
* Severity

Close the report with a **Checks Not Performed** section listing anything that required database access, naming the referenced table in each case.

---

# Step 11 — Remediation

## Purpose

The remediation stage extends the existing validation process.

The objective is to move from:

```text
Identify Problems
       ↓
Report Problems
```

to:

```text
Identify Problems
       ↓
Determine Which Problems Can Be Safely Corrected
       ↓
Apply Supported Corrections
       ↓
Produce Cleaned Data
       ↓
Record Every Change
       ↓
Re-Validate
       ↓
Determine Final Status
```

Remediation must only be performed when the required information is available to determine the correct action.

---

## Remediation Rules

The agent may correct an issue when the expected correction can be directly determined from:

* The Schema file
* The Rules file
* Supplied reference data
* An explicitly defined transformation rule
* A deterministic correction that does not require guessing

Examples of potentially correctable issues include:

* Formatting values according to an explicitly defined format rule
* Converting a value to the expected data type where the conversion is unambiguous
* Replacing a value with an explicitly defined fixed value
* Correcting an allowed-value mismatch where the correct replacement is explicitly defined
* Removing data that the rules explicitly state must not be present
* Applying an explicitly defined transformation

The agent must **not**:

* Invent replacement values
* Guess what the user intended
* Create missing business data without an authoritative source
* Modify values merely because they "look better"
* Resolve database-dependent references without database access
* Make a correction when multiple possible corrections exist
* Modify the original uploaded file

If a correction cannot be safely determined, the issue remains unresolved and must be recorded in the Change Log.

---

# Step 12 — Produce Cleaned Output File

When remediation is performed, create a separate cleaned output file.

The original file must remain unchanged.

The cleaned output should:

* Preserve the original file structure wherever possible
* Preserve valid data
* Apply only supported corrections
* Remove or transform data only where explicitly justified
* Maintain the expected column structure
* Be suitable for the next validation pass

The output should be clearly identified as the **remediated/cleaned file**.

---

# Step 13 — Produce Change Log

Every change made to the original data must be recorded.

The Change Log should contain at least:

| Row | Column      | Original Value    | New Value         | Action      | Reason                         | Source             | Status     |
| --: | ----------- | ----------------- | ----------------- | ----------- | ------------------------------ | ------------------ | ---------- |
|  27 | LeaseName   | Original value    | Corrected value   | Corrected   | Exceeded maximum length        | Rules.md           | Applied    |
|  43 | Status      | Invalid value     | Valid value       | Corrected   | Value not in allowed list      | Rules.md           | Applied    |
|  61 | LegacyField | Value             | —                 | Removed     | Field not permitted by rules   | Rules.md           | Applied    |
|  84 | TenantID    | Invalid reference | Invalid reference | Not changed | Requires database verification | External Reference | Unresolved |

### Change Log Status Values

Use:

* `Applied` — correction was successfully made
* `Unresolved` — issue could not be safely corrected
* `Not Applicable` — remediation was not required

The Change Log must never claim a correction was made when the data was not actually changed.

---

# Step 14 — Re-Validate the Cleaned File

After remediation, the cleaned output file must be treated as a new validation input.

Run the complete validation process again using the same:

* Schema file
* Rules file
* Reference data
* Validation passes
* Reference-availability checks

Do not assume that remediation succeeded simply because a correction was applied.

The cleaned file must pass validation independently.

---

# Step 15 — Determine Final Integration Status

After re-validation, determine the final status.

```text
Original File
     │
     ▼
Initial Validation
     │
     ├── READY
     │
     └── NOT READY
            │
            ▼
       Remediation
            │
            ▼
       Cleaned File
            │
            ▼
       Re-Validation
            │
            ▼
      Final Status
```

Use the same status rules:

| Status                           | Use When                                                                         |
| -------------------------------- | -------------------------------------------------------------------------------- |
| `READY`                          | The cleaned file contains no errors and no outstanding database-dependent checks |
| `NOT READY`                      | One or more errors remain after remediation                                      |
| `REQUIRES DATABASE VERIFICATION` | No errors remain, but one or more checks still require database access           |

A file must **not** be marked `READY` simply because all automatically correctable issues were fixed.

The final status must always be based on the results of the **final validation pass**.

---

# Final Output Structure

When remediation is performed, the agent should provide:

```text
Data Scrubbing Result
=====================

Product:        [Product]
Target Table:   [Table]
Original File:  [filename]

Initial Status: [READY | NOT READY | REQUIRES DATABASE VERIFICATION]

Issues Found:   [n]
Issues Corrected: [n]
Issues Removed: [n]
Issues Transformed: [n]
Issues Unresolved: [n]

Final Status: [READY | NOT READY | REQUIRES DATABASE VERIFICATION]

Outputs:
- Validation Report
- Cleaned/Remediated File
- Change Log
```

The final response should clearly distinguish between:

1. **What was found**
2. **What was changed**
3. **What could not be changed**
4. **What the final validation result is**

---

# Related-Table Awareness

When several tables from the same product are supplied together, validate cross-references between them rather than deferring to the database.

Derive the load order from the **Cross-Reference Rules** and **External References — Availability** sections of each table's rules file: load referenced tables before the tables that reference them.

Known load orders:

| Product | Load Order                                  | Because                                                                                                                               |
| ------- | ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Angus   | Area → Tenant → Contact                     | Tenant floors/suites resolve against Area; Contact tenants resolve against Tenant                                                     |
| EVO     | FASSET → F_TASKS                            | PPM `Asset Code` resolves against an existing asset record                                                                            |
| EVO     | Floors → FLOCATE → BuildingFloors → FAREALO | BuildingFloors links a Floor to a Building; Location floors resolve against that link, and Location buildings resolve against FLOCATE |
| PMX     | GACC → ENTITY → BMAP                        | BMAP account numbers resolve against GACC; BMAP entities resolve against ENTITY                                                       |

For any product or table not listed above, work the order out from the rules files rather than assuming there are no relationships.

If only one file is supplied, say which companion file would allow the deferred checks to be completed.

---

# Severity Classification

**Error** — the integration may fail, or data may be rejected or stored incorrectly.

Examples:

* Missing required column
* Missing required value
* Value exceeds maximum length
* Invalid data type
* Invalid precision or scale
* Value outside the allowed list
* Fixed value not supplied as specified
* Format rule not met
* Multiple values in a single-value field
* Duplicate primary key or duplicate unique value
* Reference not found in a companion file that **was** supplied

**Warning** — requires attention but is not a confirmed violation.

Examples:

* Unexpected column
* Reference that could not be verified
* Conditional requirement that depends on client configuration
* Module or regional option field populated where the option may not be in use
* Row-structure guidance not followed

---

# Remediation Classification

Each detected issue should also be classified according to whether remediation is possible.

| Remediation Status               | Meaning                                                                     |
| -------------------------------- | --------------------------------------------------------------------------- |
| `AUTO-CORRECTABLE`               | The correct action can be determined from authoritative information         |
| `REQUIRES REVIEW`                | A potential correction exists but cannot be safely determined automatically |
| `REQUIRES DATABASE VERIFICATION` | Correction depends on data outside the repository                           |
| `UNRESOLVED`                     | No authoritative correction is available                                    |

Only `AUTO-CORRECTABLE` issues may be automatically changed by the agent.

---

# Authoring Convention — Adding a Rules File

When populating an empty rules file, or adding a new table:

1. Name the file exactly after the table: `/{Product}/Rules/{Table}.md`.
2. Head the file with product, target table, schema file path, source workbook, source worksheet.
3. Reproduce the source rule text as supplied — do not silently correct it.
4. Include a **Field Rules** table covering every field in the source specification.
5. Derive explicit sections only where the source supports them: Required Values, Maximum Length, Data Type Expectations, Allowed Values, Fixed Values, Format Rules, Single-Value Rules, Uniqueness Rules, Row Structure Rules, Conditional Requirements, Module / Option-Specific Fields, Cross-Reference Rules.
6. Include an **External References — Availability** section whenever the source refers to another table, stating clearly where that table is not supported here.
7. Include a **Source Notes** section recording any anomaly, typo, or truncation in the source workbook.
8. Add the table to the **Product Registry** in this file if it is not already listed.
9. Where a rule supports deterministic remediation, document the expected correction clearly enough for the agent to identify it without guessing.

---

# Pre-Delivery Checklist

Before returning a final result, confirm every applicable point is YES:

| #  | Check                                                                                |
| -- | ------------------------------------------------------------------------------------ |
| 1  | Product was confirmed, not assumed                                                   |
| 2  | Target table was confirmed and is listed in the Product Registry                     |
| 3  | Both `/{Product}/Schema/{Table}.json` and `/{Product}/Rules/{Table}.md` were read    |
| 4  | Every validation pass in Step 7 was either run or explicitly noted as not applicable |
| 5  | Every finding cites a row, a column, the expected rule, and the actual condition     |
| 6  | No rule was reported that does not trace back to a schema or rules file              |
| 7  | Step 8 was applied — no unverifiable reference was reported as an error              |
| 8  | Initial integration status matches the findings                                      |
| 9  | A "Checks Not Performed" section lists every database-dependent check                |
| 10 | The original user's file was not modified                                            |
| 11 | Every automated correction is supported by an authoritative source                   |
| 12 | Every change made is recorded in the Change Log                                      |
| 13 | The cleaned output file was re-validated                                             |
| 14 | Final status is based on the results of the final validation                         |
| 15 | Any unresolved issues are clearly identified                                         |

If any check is NO, fix the result before returning it.

---

# Future Expansion

The repository is expected to grow to cover:

* Additional tables per product
* Additional products
* Cross-table referential validation across supplied file sets
* Direct database connectivity for reference verification
* Automated correction recommendations
* Automated data remediation
* Change tracking and audit history
* Re-validation of remediated files
* Integration-ready output generation

The repository remains focused on supplying **structured schema information, business rules, and validation/remediation context** to the agent.

The agent is responsible for orchestrating the validation and remediation process.

Logic that belongs to the agent should not be duplicated into the per-table files.

---

# Architecture Evolution

The current architecture establishes a reliable **validation layer**:

```text
Input File
    ↓
Orchestrator
    ↓
Schema + Rules
    ↓
Validation
    ↓
Validation Report
    ↓
READY / NOT READY
```

The next phase extends this architecture with a **remediation and audit layer**:

```text
Input File
    ↓
Orchestrator
    ↓
Schema + Rules
    ↓
Validation
    ↓
Issues Identified
    ↓
Remediation
    ├── Correct
    ├── Transform
    ├── Remove
    └── Flag for Review
    ↓
Cleaned Output File
    +
Change Log
    ↓
Re-Validation
    ↓
Final Integration Status
    ↓
READY / NOT READY / REQUIRES DATABASE VERIFICATION
```

The goal is to evolve the DataScrubbing Agent from a tool that **identifies data-quality issues** into a controlled process that can **identify, remediate, document, and re-validate data before integration**.
