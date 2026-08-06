# DataScrubbingAgent

## Purpose

**DataScrubbingAgent** is a schema-driven data validation repository designed to support a Microsoft Copilot Studio agent.

The repository contains database table schema definitions in **JSON format**, organized by product.

The Microsoft Copilot agent uses these schema definitions as the authoritative rules for validating raw **CSV or Excel integration files** before they are used in a database integration process.

The primary objective is to determine whether an uploaded data file is **ready for integration into its intended database table**.

The agent should identify schema violations, data-quality issues, and potential integration problems before the file is passed to the integration process.

---

# Repository Structure

Schema files are organized into separate folders based on the product they belong to.

```text
DataScrubbingAgent/
│
├── README.md
│
├── PLE/
│   ├── Lease.json
│   ├── Tenant.json
│   ├── Property.json
│   ├── Unit.json
│   ├── Demise.json
│   ├── Address.json
│   └── ...
│
├── PMX/
│   ├── ENTITY.json
│   ├── BMAP.json
│   ├── GACC.json
│   └── ...
│
└── Angus/
    ├── Lease.json
    ├── Tenant.json
    ├── Area.json
    ├── TenantAreaLease.json
    ├── Contact.json
    └── ...
```

Each product folder contains **only the schema definitions belonging to that product**.

The same table name can exist in multiple products. For example, both **PLE** and **Angus** contain a `Lease` table.

Therefore, the agent **must identify the product before selecting a schema file**.

---

# Supported Products and Tables

## PLE

Product folder:

```text
/PLE/
```

Supported tables:

| Table    | Schema File          |
| -------- | -------------------- |
| Lease    | `/PLE/Lease.json`    |
| Tenant   | `/PLE/Tenant.json`   |
| Property | `/PLE/Property.json` |
| Unit     | `/PLE/Unit.json`     |
| Demise   | `/PLE/Demise.json`   |
| Address  | `/PLE/Address.json`  |

---

## PMX

Product folder:

```text
/PMX/
```

Supported tables:

| Table  | Schema File        |
| ------ | ------------------ |
| ENTITY | `/PMX/ENTITY.json` |
| BMAP   | `/PMX/BMAP.json`   |
| GACC   | `/PMX/GACC.json`   |

---

## Angus

Product folder:

```text
/Angus/
```

Supported tables:

| Table           | Schema File                   |
| --------------- | ----------------------------- |
| Lease           | `/Angus/Lease.json`           |
| Tenant          | `/Angus/Tenant.json`          |
| Area            | `/Angus/Area.json`            |
| TenantAreaLease | `/Angus/TenantAreaLease.json` |
| Contact         | `/Angus/Contact.json`         |

---

# Copilot Agent Workflow

The Microsoft Copilot Studio agent should follow this workflow when a user provides an integration file.

```text
1. Identify Product
        ↓
2. Identify Target Table
        ↓
3. Confirm Product + Table combination
        ↓
4. Locate the corresponding JSON schema
        ↓
5. Read schema definition
        ↓
6. Read uploaded CSV / Excel file
        ↓
7. Map uploaded columns to schema columns
        ↓
8. Validate data against schema rules
        ↓
9. Identify errors and warnings
        ↓
10. Generate Integration Readiness Report
```

---

# 1. Identify the Product

The agent must determine which product the uploaded file is intended for.

Supported products are:

* `PLE`
* `PMX`
* `Angus`

The product may be provided directly by the user.

Example:

```text
Product: Angus
```

If the user has not specified a product and the product cannot be reliably determined from the available context, the agent should ask:

> Which product is this integration file for?

The agent **must not assume a product**.

---

# 2. Identify the Target Table

The agent must determine which supported table the uploaded file is intended to populate.

The valid tables depend on the selected product.

### PLE

```text
Lease
Tenant
Property
Unit
Demise
Address
```

### PMX

```text
ENTITY
BMAP
GACC
```

### Angus

```text
Lease
Tenant
Area
TenantAreaLease
Contact
```

If the target table has not been provided, the agent should ask the user which table the file is intended for.

---

# 3. Confirm the Product + Table Combination

The agent must verify that the requested table exists for the selected product.

For example:

```text
Product: Angus
Table: Lease
```

is valid because:

```text
/Angus/Lease.json
```

exists within the supported Angus table definitions.

However:

```text
Product: PMX
Table: Lease
```

is **not a valid combination**, because `Lease` is not currently listed as a supported PMX table.

The agent should not attempt to find or use:

```text
/PLE/Lease.json
```

when the user has specified PMX.

Instead, it should inform the user that `Lease` is not currently a supported PMX table.

---

# 4. Locate the Correct JSON Schema

Once the product and table have been established, the agent should retrieve the corresponding JSON schema file.

The schema path follows this pattern:

```text
/{Product}/{Table}.json
```

Examples:

```text
/PLE/Lease.json
/PLE/Property.json

/PMX/ENTITY.json
/PMX/GACC.json

/Angus/Lease.json
/Angus/Area.json
/Angus/TenantAreaLease.json
```

### Important

The product folder is part of the schema identity.

For example:

```text
/PLE/Lease.json
```

and

```text
/Angus/Lease.json
```

must be treated as **two different schema definitions**.

The agent must never substitute one for the other.

---

# 5. Use the JSON Schema as the Source of Truth

The selected JSON file represents the expected structure and rules of the target database table.

Depending on the schema extraction process, the JSON may contain information such as:

* Database name
* Schema name
* Table name
* Column names
* Data types
* Maximum field lengths
* Precision
* Scale
* Nullable status
* Primary keys
* Foreign keys
* Constraints
* Parent relationships
* Child relationships
* Referenced tables
* Referencing tables
* Other relevant database metadata

The agent should use the information contained in the JSON schema when validating the uploaded file.

The agent should **not invent or assume database rules that are not present in the schema information**.

---

# 6. Read the Uploaded Integration File

The user may provide an integration file in a supported tabular format, such as:

* CSV
* Excel (`.xlsx`)

The agent should inspect the uploaded file and determine:

* Column names
* Number of records
* Populated values
* Blank values
* Data formats
* Potentially invalid values
* Unexpected columns
* Missing expected columns

The uploaded file represents the **data being tested**.

The JSON schema represents the **rules against which the data is tested**.

---

# 7. Validate the File Against the Schema

The agent should validate the uploaded file against the selected table schema.

## Column Validation

Verify that the uploaded file contains the expected table columns.

Identify:

* Missing columns
* Unexpected columns
* Incorrect column names
* Duplicate columns where applicable

Example:

```text
Target Table: Angus → Lease

Expected:
LeaseID
LeaseName
StartDate
EndDate

Uploaded:
LeaseID
LeaseName
StartDate
TenantName
```

Result:

```text
❌ Missing expected column: EndDate
⚠️ Unexpected column: TenantName
```

---

## Data Type Validation

Compare uploaded values against the expected database data type.

Examples:

```text
Expected: INTEGER
Actual: ABC

Expected: DATE
Actual: Not a Date

Expected: DECIMAL
Actual: ABC123
```

Where the value does not conform to the expected data type, the agent should identify the affected record and report the issue.

---

## Maximum Length Validation

Where a schema defines a maximum field length, the agent must check that uploaded values do not exceed that limit.

Example:

```text
Column: LeaseName
Maximum Length: 100

Row: 27
Actual Length: 127
```

Result:

```text
❌ Row 27 — LeaseName exceeds the maximum permitted length.

Expected: ≤ 100 characters
Actual: 127 characters
```

This should be treated as an **integration error** because the value may be truncated or rejected by the target database.

---

## Nullable / Required Field Validation

Where the schema indicates that a field does not allow NULL values, identify records where the corresponding value is missing or blank.

Example:

```text
Column: LeaseID
Nullable: NO

Row: 154
Value: NULL
```

Result:

```text
❌ Row 154 — Required field LeaseID is missing.
```

---

## Numeric Precision and Scale Validation

Where numeric fields contain precision and scale definitions, validate the uploaded values against those limits.

Example:

```text
Column: RentalAmount
Type: DECIMAL
Precision: 10
Scale: 2
```

Values that cannot be represented within the defined precision and scale should be reported.

---

## Primary Key Validation

Where primary-key information is available, check for:

* Missing primary-key columns
* Blank primary-key values
* Duplicate primary-key values within the uploaded file

Example:

```text
❌ Duplicate primary key detected.

Column: LeaseID
Value: 10452
Rows: 27, 91, 143
```

---

## Foreign Key Validation

Where foreign-key information exists within the JSON schema, identify potential foreign-key issues that can be determined from the uploaded file.

However, the agent must distinguish between:

**Schema-level validation**

and

**Database-level validation**.

For example, the schema may indicate:

```text
Lease.TenantID → Tenant.TenantID
```

This establishes that a relationship exists.

It does **not** prove that the corresponding `TenantID` actually exists in the target database unless the agent has access to the database.

If database access is unavailable, the agent should state:

```text
⚠️ Foreign-key relationship identified.
Database-level existence could not be verified.
```

The agent must not claim that a foreign key is valid or invalid without sufficient evidence.

---

# 8. Integration Readiness

After completing validation, the agent should determine the overall integration status.

## READY

Use `READY` when no blocking schema or data issues were identified.

```text
Integration Status: READY
```

---

## NOT READY

Use `NOT READY` when one or more blocking issues were identified.

Examples:

* Missing required column
* Required field is blank
* Value exceeds maximum field length
* Invalid data type
* Invalid numeric precision/scale
* Duplicate primary-key value
* Other direct schema violations

```text
Integration Status: NOT READY
```

---

## REQUIRES DATABASE VERIFICATION

Use `REQUIRES DATABASE VERIFICATION` when the file passes the checks that can be performed using the uploaded file and schema, but additional validation requires access to the target database.

```text
Integration Status: REQUIRES DATABASE VERIFICATION
```

---

# 9. Validation Report

The agent should produce a clear integration-readiness report.

Recommended structure:

```text
Integration Validation Report
=============================

Product: Angus
Target Table: Lease
File: Lease.xlsx

Records Evaluated: 5,000

Integration Status: NOT READY

Errors: 37
Warnings: 12
```

The report should then provide details of the identified issues.

Recommended format:

| Row | Column     | Issue                        | Expected         | Actual         | Severity |
| --: | ---------- | ---------------------------- | ---------------- | -------------- | -------- |
|  27 | LeaseName  | Value exceeds maximum length | ≤ 100 characters | 127 characters | Error    |
|  43 | LeaseID    | Required value missing       | NOT NULL         | NULL           | Error    |
|  61 | StartDate  | Invalid data type            | DATE             | Invalid value  | Error    |
|  84 | TenantName | Column not defined in schema | Not defined      | Present        | Warning  |

Where possible, every issue should contain:

* Row number
* Column name
* Issue description
* Expected rule
* Actual value or condition
* Severity

---

# Error and Warning Classification

## Errors

Errors represent conditions that may cause the integration to fail or cause data to be rejected or incorrectly stored.

Examples:

* Missing required column
* Missing required value
* Value exceeds maximum field length
* Invalid data type
* Invalid numeric precision
* Invalid numeric scale
* Duplicate primary-key values
* Other direct violations of the available schema rules

Errors should result in:

```text
Integration Status: NOT READY
```

---

## Warnings

Warnings represent conditions that require attention but cannot necessarily be classified as a direct schema violation.

Examples:

* Unexpected columns
* Foreign-key relationships that require database verification
* Information that cannot be validated without database access
* Other potential issues requiring user review

Warnings should be clearly separated from errors.

---

# Agent Rules

The Copilot agent should follow these rules at all times.

### Rule 1 — Product First

Always establish the product before selecting a schema.

Supported products:

```text
PLE
PMX
Angus
```

### Rule 2 — Validate the Product/Table Combination

Only use tables that are supported for the selected product.

### Rule 3 — Product Determines the Schema

The product folder is part of the schema identity.

For example:

```text
/PLE/Lease.json
```

is not interchangeable with:

```text
/Angus/Lease.json
```

### Rule 4 — JSON Is the Source of Truth

Use the selected JSON schema as the authoritative source for database structure and validation rules.

### Rule 5 — Do Not Invent Rules

Do not assume a field is required, nullable, a particular length, or a particular data type unless the available schema information supports that conclusion.

### Rule 6 — Do Not Ignore Violations

Any identifiable violation of the schema should be reported.

### Rule 7 — Identify Affected Records

Where possible, identify the exact row and column responsible for the issue.

### Rule 8 — Do Not Claim Database Validation Without Database Access

Schema information alone cannot confirm whether referenced records exist in the target database.

### Rule 9 — Do Not Modify the Source File by Default

The default role of the agent is to **validate and report**.

It should not modify, overwrite, or transform the uploaded file unless the user explicitly requests it.

### Rule 10 — Be Transparent About Limitations

If a validation cannot be performed using the uploaded file and available schema information, clearly state the limitation.

---

# Example End-to-End Scenario

A user provides:

```text
Product: Angus
Target Table: Lease
File: Lease.xlsx
```

The agent identifies the correct schema:

```text
/Angus/Lease.json
```

The agent reads the schema and determines, for example, that:

```text
LeaseName
Type: VARCHAR
Maximum Length: 100
Nullable: NO
```

The uploaded file contains:

```text
Row 27
LeaseName = [127-character value]
```

The agent identifies:

```text
❌ Row 27 — LeaseName

Schema Rule:
VARCHAR(100)

Actual:
127 characters

Issue:
The supplied value exceeds the maximum permitted length by 27 characters.

Integration Impact:
The value may be rejected or truncated during database integration.
```

The overall result becomes:

```text
Integration Status: NOT READY
```

---

# Current Schema Coverage

## PLE

```text
/PLE/
├── Lease.json
├── Tenant.json
├── Property.json
├── Unit.json
├── Demise.json
└── Address.json
```

## PMX

```text
/PMX/
├── ENTITY.json
├── BMAP.json
└── GACC.json
```

## Angus

```text
/Angus/
├── Lease.json
├── Tenant.json
├── Area.json
├── TenantAreaLease.json
└── Contact.json
```

---

# Future Expansion

The repository may eventually support additional validation capabilities, including:

* Cross-table validation
* Referential integrity validation
* Database connectivity
* Product-specific business rules
* Integration-specific transformation rules
* Automated correction recommendations
* Generation of corrected integration files
* Integration readiness scoring
* Detailed validation reports
* Additional products
* Additional tables

The repository should remain focused on providing **structured schema information and validation context** to the Copilot agent.
