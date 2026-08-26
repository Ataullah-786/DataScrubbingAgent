# PMX — ENTITY Rules

**Product:** PMX  
**Target Table:** `ENTITY`  
**Schema File:** `/PMX/Schema/ENTITY.json`  
**Source Workbook:** Import Tables - All Modules  
**Source Worksheet:** `ENTITY`

These rules are taken from the MRI PMX import specification workbook. They describe
how the import file columns must be populated before the file can be integrated into
the PMX `ENTITY` table.

They are **additional to** the structural rules in `/PMX/Schema/ENTITY.json`.
Where the two disagree on data type or length, the JSON schema remains the source of
truth for the physical database, and the rules below define the business/import
expectation.

> The `ENTITY` worksheet defines the GL entity and its accounting, AP, address and
> reporting defaults. It is the widest of the PMX worksheets and is referenced by
> `BMAP.ENTITYID`.

---

## Field Rules

| # | MRI Table Field Name | PMX Field Name | Type | Length | Required | Client Specific Notes |
| -: | -------------------- | -------------- | ---- | -----: | -------- | --------------------- |
| 1 | `PROJID` | !{Project} Id | A | 6 | **Y** | Must match PROJ.PROJID |
| 2 | `ENTITYID` | GL !{Entity} ID | A | 6 | **Y** | \*\*no special characters |
| 3 | `NAME` | GL !{Entity} Name | A | 30 | **Y** | — |
| 4 | `LEDGCODE` | Ledger Code | A | 2 | **Y** | — |
| 5 | `CLOSEDAY` | Close Day | A | 3 | **Y** | Day of Month or EOM |
| 6 | `BASIS` | Cash, Accrual or Both | A | 1 | **Y** | (C)ash (A)ccrual (B)oth |
| 7 | `SUMDET` | Summary or Detail (S/D) | A | 1 | **Y** | S - Summary creates one journal entry for all transactions posted to an expense account; D - Detail creates a separate journal entry for each transaction posted to an expense account |
| 8 | `SUMCASH` | Summarize Cash Account | A | 1 | **Y** | Y - Summarize creates one journal entry for all transactions posted to the cast account; N - Creates separate journal entry for each transaction posted to the cash account |
| 9 | `ACTIVE` | Active Flag | A | 1 | **Y** | — |
| 10 | `SOFTCLOSEGL` | Soft GL Close Flag | A | 1 | **Y** | — |
| 11 | `ActiveSegment` | Actively used for GL segmentation | B | — | **Y** | — |
| 12 | `SegmentAccountFilter` | Has Account Filters for GL segmentation | B | — | **Y** | — |
| 13 | `YEAREND` | Year End Period | A | 6 | **Y** | YYYYMM |
| 14 | `CURPED` | Current Period | A | 6 | **Y** | — |
| 15 | `MAXOPEN` | Open GL Periods Alllowed | N | 2 | **Y** | How many GL Periods are allowed to be open simultaneously |
| 16 | `APACCTNO` | AP Account | A | 9 | **Y** | — |
| 17 | `INVCSTAT` | Default Invoice Status | A | 1 | **Y** | Default status when entering invoices.  Valid values are: <br>H - Hold Payment; <br>I - Information Only; <br>P - Pay in next check selection; <br>R - Release for Payment |
| 18 | `CASHTYPE` | Default AP !{Cash Type} | A | 2 | **Y** | Enter the default cash type for use in entering invoices.  Can be changed during invoice entry |
| 19 | `MAXAPOPEN` | Open AP Periods Allowed | N | 2 | **Y** | — |
| 20 | `ADDR1` | Address Line 1 | A | 35 | N | — |
| 21 | `ADDR2` | Address Line 2 | A | 35 | N | — |
| 22 | `ADDR3` | Address Line 3 | A | 35 | N | — |
| 23 | `STATE` | State | A | 3 | N | — |
| 24 | `CITY` | City | A | 17 | N | — |
| 25 | `ZIPCODE` | Zip Code | A | 9 | N | No - |
| 26 | `PHONE` | Phone Number | A | 15 | N | No () - or . |
| 27 | `FEET` | !{Square Footage} | N | 19 | N | — |
| 28 | `UNITS` | Number of Units | N | 10 | N | — |
| 29 | `LASTDATE` | Last Update | D | — | **Y** | Always SYSDATE |
| 30 | `USERID` | User Id | A | 20 | **Y** | Always CONV |
| 31 | `INTRACCT` | Inter!{entity} Account | A | 9 | N | Do not populate unless using MRI interentity accounting |
| 32 | `GLPURGE` | Last GL Purge | A | 6 | N | — |
| 33 | `OPENBAL` | Opening A/P Balance | N | 19 | N | — |
| 34 | `APPURGE` | Last AP Purge | A | 6 | N | — |
| 35 | `M_1099DATE` | Date 1099's Printed | D | 8 | N | — |
| 36 | `VATREG` | VAT Registration Number | A | 9 | N | International Option |
| 37 | `ATAXACCT` | !{Sales Tax} GL Account | A | 9 | N | — |
| 38 | `ATAXCASH` | !{Sales Tax} !{Cash Type} | A | 2 | N | — |
| 39 | `VENDPPED` | Last !{Vendor} History Purge | A | 6 | N | — |
| 40 | `PAYENTRY` | Type of Payable Entry | A | 1 | N | — |
| 41 | `TAXEXEMP` | Tax Exemption # | A | 12 | N | Purchase Order |
| 42 | `SHIPADR1` | Ship to Address 1 | A | 35 | N | Purchase Order |
| 43 | `SHIPADR2` | Ship to Address 2 | A | 35 | N | Purchase Order |
| 44 | `SHIPADR3` | Ship To Address 3 | A | 35 | N | Purchase Order |
| 45 | `SHIPCITY` | Ship To City | A | 17 | N | Purchase Order |
| 46 | `SHIPST` | Ship To State | A | 3 | N | Purchase Order |
| 47 | `SHIPZIP` | Ship To Zip Code | A | 9 | N | Purchase Order |
| 48 | `BILLADR1` | Bill To Address 1 | A | 35 | N | Purchase Order |
| 49 | `BILLADR2` | Bill To Address 2 | A | 35 | N | Purchase Order |
| 50 | `BILLADR3` | Bill To Address 3 | A | 35 | N | Purchase Order |
| 51 | `BILLCITY` | Bill To City | A | 17 | N | Purchase Order |
| 52 | `BILLST` | Bill To State | A | 3 | N | Purchase Order |
| 53 | `BILLZIP` | Bill To Zip Code | A | 9 | N | Purchase Order |
| 54 | `ACCTNUM` | Retainage Control Account | A | 9 | N | Only if using Job Cost |
| 55 | `APPLIMIT` | P.O. Approval Limit | N | 10 | N | Purchase Order |
| 56 | `STATEID` | State Id | A | 3 | N | — |
| 57 | `PROPTYPE` | Property Type | A | 3 | N | — |
| 58 | `PROPSUBTYPE` | Property Sub Type | A | 3 | N | — |
| 59 | `ACQUIRED` | Date Acquired | D | — | N | — |
| 60 | `DISPOSED` | Disposition Date | D | — | N | — |
| 61 | `INVESTFLAG` | Investment Only Flag | A | 1 | N | — |
| 62 | `CLASSID` | Property Class | A | 1 | N | — |
| 63 | `INVTYPE` | Investment Type | A | 6 | N | — |
| 64 | `LIFECODE` | Life Cycle Code | A | 6 | N | — |
| 65 | `LOCAID` | Location Id | A | 3 | N | — |
| 66 | `GROSSVALUE` | Gross Asset Value | N | 19 | N | — |
| 67 | `NETVALUE` | Net Asset Value | N | 19 | N | — |
| 68 | `INCRET` | Income Return | N | 19 | N | — |
| 69 | `CAPRET` | Capital Return | N | 19 | N | — |
| 70 | `TOTRET` | Total Return | N | 19 | N | — |
| 71 | `JEDESCID` | Journal Entry Description Id | A | 9 | N | — |
| 72 | `OPTED` | VAT Opted | A | 1 | **Y** | Always N |
| 73 | `CURRCODE` | Base Currency Code | A | 3 | N | Multi-currency Option |
| 74 | `APREALEXCHG` | AP !{Realized} Gain Account Number | A | 9 | N | Multi-currency Option |
| 75 | `APUNREEXCHG` | AP !{Unrealized} Gain Account Number | A | 9 | N | Multi-currency Option |
| 76 | `CROSSCURYN` | Accept Non-Invoiced Currency Payments | A | 1 | N | Multi-currency Option |
| 77 | `CMREALACCT` | CM !{Realized} Gain Account Number | A | 9 | N | Multi-currency Option |
| 78 | `CMUNREACCT` | CM !{Unrealized} Gain Account Number | A | 9 | N | Multi-currency Option |
| 79 | `CMDEPGLACCT` | CM Security Deposit Gain Account Number | A | 9 | N | Multi-currency Option |
| 80 | `RMTACCTNO` | !{Remittance} Account Number | A | 9 | N | Australian Option |
| 81 | `RMTCASHTYPE` | !{Remittance} !{Cash Type} | A | 2 | N | Australian Option |
| 82 | `RMTOTHRFEE` | Other !{Remittance} Fees | N | 19 | N | Australian Option |
| 83 | `TOGFEES` | !{Remittance} TOG Fees | N | 19 | N | Australian Option |
| 84 | `MINBAL` | !{Remittance} Minimum Balance | N | 19 | N | Australian Option |
| 85 | `HELDFUNDS` | !{Remittance} Held Funds | N | 19 | N | Australian Option |
| 86 | `BADTAXACCT` | BAD Tax Account Number | A | 9 | N | Australian Option |
| 87 | `BADTAXCSHTYPE` | BAD Tax !{Cash Type} | A | 2 | N | Australian Option |
| 88 | `FIDTAXACCT` | FID Tax Account Number | A | 9 | N | Australian Option |
| 89 | `FIDTAXCSHTYPE` | FID Tax !{Cash Type} | A | 2 | N | Australian Option |
| 90 | `TOGACCTNUM` | TOG fees Account Number | A | 9 | N | Australian Option |
| 91 | `TOGCASHTYPE` | TOG Fees !{Cash Type} | A | 2 | N | Australian Option |
| 92 | `TOGVENDID` | TOG Fees !{Vendor} ID | A | 6 | N | Australian Option |
| 93 | `OTRACCTNUM` | Other Fees Account Number | A | 9 | N | Australian Option |
| 94 | `OTRCASHTYPE` | Other Fees !{Cash Type} | A | 2 | N | Australian Option |
| 95 | `OTRVENDID` | Other Fees !{Vendor} Id | A | 6 | N | Australian Option |
| 96 | `ENTTYPE` | !{Entity} Type | A | 3 | N | — |
| 97 | `PERIODCNT` | Period Count | N | 5 | **Y** | Always 12 |
| 98 | `RECLAIMPCT` | Reclaimable !{Tax} Percentage | N | 53 | N | — |
| 99 | `ALTTAXID` | Alternate !{Tax} Id | A | 15 | N | — |
| 100 | `TAXSUSPENSE` | Enable Tax Suspense Processing | A | 1 | **Y** | Always N |
| 101 | `CMNBANKING` | Use Common Banking | A | 1 | **Y** | UK Option = Y<br>North America = N |
| 102 | `ARREALACCT` | AR !{Realized} Gain Account | A | 9 | N | International Option |
| 103 | `ARUNREACCT` | AR !{Unrealized} Gain Account | A | 9 | N | International Option |
| 104 | `RETAINACCT` | Retainage Account Number | A | 9 | N | International Option |
| 105 | `BILLFEES` | Bill Fees Only If There Is  Activity | A | 1 | **Y** | Always N |
| 106 | `FUNDBALACCT` | !{Account Funding} Balancing Account Num | A | 9 | N | International Option |
| 107 | `TAXRPTFREQ` | !{Tax} Report Frequency | A | 1 | N | — |
| 108 | `OWNERID` | !{Owner} Id | A | 6 | N | — |
| 109 | `COUNTRY` | Country | A | 2 | N | — |
| 110 | `APTAXSUSPENSEACCOUNTNUMBER` | AP !{Tax} Suspense Account Number | A | 9 | N | — |
| 111 | `STLINEBASIS` | Straight Line Basis | A | 1 | N | Basis type if FASB 13 entries to separate basis |
| 112 | `MATRIXID` | Matrix Group ID | N | 12 | N | — |
| 113 | `ENTMATRIXOPTION` | Date Period Matrix Option | A | 1 | N | — |
| 114 | `NCREIFACQUIREDFIRSTDAYQTR` | Acquired First Dayof Quarter | A | 1 | N | — |
| 115 | `NCREIFDISPOSEDLASTDAYQTR` | DisposedLast Day of Quarter | A | 1 | N | — |
| 116 | `CHECKLOCATIONID` | !{Check} Printing Location Id | A | 6 | N | — |
| 117 | `APREALLOSS` | AP !{Realized} Loss Account | A | 9 | N | If using Accural, this field is required |
| 118 | `APUNREALLOSS` | AP !{Unrealized} Loss Account | A | 9 | N | If using Accural, this field is required |
| 119 | `ARREALLOSS` | AR !{Realized} Loss Account | A | 9 | N | If using Accural, this field is required |
| 120 | `ARUNREALLOSS` | AR !{Unrealized} Loss Account | A | 9 | N | If using Accural, this field is required |
| 121 | `CMREALLOSS` | CM !{Realized} Loss Account | A | 9 | N | If using Accural, this field is required |
| 122 | `CMUNREALLOSS` | CM !{Unrealized} Loss Account | A | 9 | N | If using Accural, this field is required |
| 123 | `CMDEPLOSS` | CM Security Deposit Loss Account | A | 9 | N | If using Accural, this field is required |
| 124 | `VENDORWITHHOLDINGACCT` | Account Number | A | 9 | N | — |
| 125 | `SRVCDEBT` | CM Service Fee Debit Account | A | 9 | N | — |
| 126 | `SRVCCRED` | CM Service Fee Credit Account | A | 9 | N | — |
| 127 | `RMSRVCDEBT` | RM Service Fee Debit Account | A | 9 | N | — |
| 128 | `RMSRVCCRED` | RM Service Fee Credit Account | A | 9 | N | — |
| 129 | `IAENTTYPE` | Investment Entity Type | A | 20 | N | — |
| 130 | `IAVENDORID` | !{Vendor} Id | A | 6 | N | — |
| 131 | `IACORPORATEACCOUNTID` | Corporate !{Account}s Receivable ID | A | 12 | N | — |
| 132 | `CONNECTCURRCODE` | Connect Currency Code | A | 3 | N | — |
| 133 | `CISREGID` | CIS Registration Id | N | 10 | N | — |
| 134 | `COUNTY` | County | A | 30 | N | — |
| 135 | `ACCREXPACCT` | Accrued Expense Account Number | A | 9 | N | — |
| 136 | `INTERENTITYREF` | Interentity Reference | N | 10 | N | — |
| 137 | `SegmentCustomFilter` | Has Custom Filters for GL segmentation | B | — | N | — |

**Required fields:** 26  
**Optional fields:** 111  
**Total fields:** 137

---

## Validation Rules

### Required Values

Report an **Error** for any row where the following fields are blank:

* `PROJID` — !{Project} Id
* `ENTITYID` — GL !{Entity} ID
* `NAME` — GL !{Entity} Name
* `LEDGCODE` — Ledger Code
* `CLOSEDAY` — Close Day
* `BASIS` — Cash, Accrual or Both
* `SUMDET` — Summary or Detail (S/D)
* `SUMCASH` — Summarize Cash Account
* `ACTIVE` — Active Flag
* `SOFTCLOSEGL` — Soft GL Close Flag
* `ActiveSegment` — Actively used for GL segmentation
* `SegmentAccountFilter` — Has Account Filters for GL segmentation
* `YEAREND` — Year End Period
* `CURPED` — Current Period
* `MAXOPEN` — Open GL Periods Alllowed
* `APACCTNO` — AP Account
* `INVCSTAT` — Default Invoice Status
* `CASHTYPE` — Default AP !{Cash Type}
* `MAXAPOPEN` — Open AP Periods Allowed
* `LASTDATE` — Last Update
* `USERID` — User Id
* `OPTED` — VAT Opted
* `PERIODCNT` — Period Count
* `TAXSUSPENSE` — Enable Tax Suspense Processing
* `CMNBANKING` — Use Common Banking
* `BILLFEES` — Bill Fees Only If There Is  Activity

All other fields on this worksheet are optional and may be left blank.

### Maximum Length

Report an **Error** where a supplied value exceeds the stated length. A length of `0`
on the worksheet means the field is not a fixed-width character field (date or
boolean) and no character limit applies.

| MRI Table Field Name | Type | Max Length |
| -------------------- | ---- | ---------: |
| `PROJID` | A | 6 |
| `ENTITYID` | A | 6 |
| `NAME` | A | 30 |
| `LEDGCODE` | A | 2 |
| `CLOSEDAY` | A | 3 |
| `BASIS` | A | 1 |
| `SUMDET` | A | 1 |
| `SUMCASH` | A | 1 |
| `ACTIVE` | A | 1 |
| `SOFTCLOSEGL` | A | 1 |
| `ActiveSegment` | B | n/a |
| `SegmentAccountFilter` | B | n/a |
| `YEAREND` | A | 6 |
| `CURPED` | A | 6 |
| `MAXOPEN` | N | 2 |
| `APACCTNO` | A | 9 |
| `INVCSTAT` | A | 1 |
| `CASHTYPE` | A | 2 |
| `MAXAPOPEN` | N | 2 |
| `ADDR1` | A | 35 |
| `ADDR2` | A | 35 |
| `ADDR3` | A | 35 |
| `STATE` | A | 3 |
| `CITY` | A | 17 |
| `ZIPCODE` | A | 9 |
| `PHONE` | A | 15 |
| `FEET` | N | 19 |
| `UNITS` | N | 10 |
| `LASTDATE` | D | n/a |
| `USERID` | A | 20 |
| `INTRACCT` | A | 9 |
| `GLPURGE` | A | 6 |
| `OPENBAL` | N | 19 |
| `APPURGE` | A | 6 |
| `M_1099DATE` | D | 8 |
| `VATREG` | A | 9 |
| `ATAXACCT` | A | 9 |
| `ATAXCASH` | A | 2 |
| `VENDPPED` | A | 6 |
| `PAYENTRY` | A | 1 |
| `TAXEXEMP` | A | 12 |
| `SHIPADR1` | A | 35 |
| `SHIPADR2` | A | 35 |
| `SHIPADR3` | A | 35 |
| `SHIPCITY` | A | 17 |
| `SHIPST` | A | 3 |
| `SHIPZIP` | A | 9 |
| `BILLADR1` | A | 35 |
| `BILLADR2` | A | 35 |
| `BILLADR3` | A | 35 |
| `BILLCITY` | A | 17 |
| `BILLST` | A | 3 |
| `BILLZIP` | A | 9 |
| `ACCTNUM` | A | 9 |
| `APPLIMIT` | N | 10 |
| `STATEID` | A | 3 |
| `PROPTYPE` | A | 3 |
| `PROPSUBTYPE` | A | 3 |
| `ACQUIRED` | D | n/a |
| `DISPOSED` | D | n/a |
| `INVESTFLAG` | A | 1 |
| `CLASSID` | A | 1 |
| `INVTYPE` | A | 6 |
| `LIFECODE` | A | 6 |
| `LOCAID` | A | 3 |
| `GROSSVALUE` | N | 19 |
| `NETVALUE` | N | 19 |
| `INCRET` | N | 19 |
| `CAPRET` | N | 19 |
| `TOTRET` | N | 19 |
| `JEDESCID` | A | 9 |
| `OPTED` | A | 1 |
| `CURRCODE` | A | 3 |
| `APREALEXCHG` | A | 9 |
| `APUNREEXCHG` | A | 9 |
| `CROSSCURYN` | A | 1 |
| `CMREALACCT` | A | 9 |
| `CMUNREACCT` | A | 9 |
| `CMDEPGLACCT` | A | 9 |
| `RMTACCTNO` | A | 9 |
| `RMTCASHTYPE` | A | 2 |
| `RMTOTHRFEE` | N | 19 |
| `TOGFEES` | N | 19 |
| `MINBAL` | N | 19 |
| `HELDFUNDS` | N | 19 |
| `BADTAXACCT` | A | 9 |
| `BADTAXCSHTYPE` | A | 2 |
| `FIDTAXACCT` | A | 9 |
| `FIDTAXCSHTYPE` | A | 2 |
| `TOGACCTNUM` | A | 9 |
| `TOGCASHTYPE` | A | 2 |
| `TOGVENDID` | A | 6 |
| `OTRACCTNUM` | A | 9 |
| `OTRCASHTYPE` | A | 2 |
| `OTRVENDID` | A | 6 |
| `ENTTYPE` | A | 3 |
| `PERIODCNT` | N | 5 |
| `RECLAIMPCT` | N | 53 |
| `ALTTAXID` | A | 15 |
| `TAXSUSPENSE` | A | 1 |
| `CMNBANKING` | A | 1 |
| `ARREALACCT` | A | 9 |
| `ARUNREACCT` | A | 9 |
| `RETAINACCT` | A | 9 |
| `BILLFEES` | A | 1 |
| `FUNDBALACCT` | A | 9 |
| `TAXRPTFREQ` | A | 1 |
| `OWNERID` | A | 6 |
| `COUNTRY` | A | 2 |
| `APTAXSUSPENSEACCOUNTNUMBER` | A | 9 |
| `STLINEBASIS` | A | 1 |
| `MATRIXID` | N | 12 |
| `ENTMATRIXOPTION` | A | 1 |
| `NCREIFACQUIREDFIRSTDAYQTR` | A | 1 |
| `NCREIFDISPOSEDLASTDAYQTR` | A | 1 |
| `CHECKLOCATIONID` | A | 6 |
| `APREALLOSS` | A | 9 |
| `APUNREALLOSS` | A | 9 |
| `ARREALLOSS` | A | 9 |
| `ARUNREALLOSS` | A | 9 |
| `CMREALLOSS` | A | 9 |
| `CMUNREALLOSS` | A | 9 |
| `CMDEPLOSS` | A | 9 |
| `VENDORWITHHOLDINGACCT` | A | 9 |
| `SRVCDEBT` | A | 9 |
| `SRVCCRED` | A | 9 |
| `RMSRVCDEBT` | A | 9 |
| `RMSRVCCRED` | A | 9 |
| `IAENTTYPE` | A | 20 |
| `IAVENDORID` | A | 6 |
| `IACORPORATEACCOUNTID` | A | 12 |
| `CONNECTCURRCODE` | A | 3 |
| `CISREGID` | N | 10 |
| `COUNTY` | A | 30 |
| `ACCREXPACCT` | A | 9 |
| `INTERENTITYREF` | N | 10 |
| `SegmentCustomFilter` | B | n/a |

### Data Type Expectations

* **A — Alpha-numeric:** `PROJID`, `ENTITYID`, `NAME`, `LEDGCODE`, `CLOSEDAY`, `BASIS`, `SUMDET`, `SUMCASH`, `ACTIVE`, `SOFTCLOSEGL`, `YEAREND`, `CURPED`, `APACCTNO`, `INVCSTAT`, `CASHTYPE`, `ADDR1`, `ADDR2`, `ADDR3`, `STATE`, `CITY`, `ZIPCODE`, `PHONE`, `USERID`, `INTRACCT`, `GLPURGE`, `APPURGE`, `VATREG`, `ATAXACCT`, `ATAXCASH`, `VENDPPED`, `PAYENTRY`, `TAXEXEMP`, `SHIPADR1`, `SHIPADR2`, `SHIPADR3`, `SHIPCITY`, `SHIPST`, `SHIPZIP`, `BILLADR1`, `BILLADR2`, `BILLADR3`, `BILLCITY`, `BILLST`, `BILLZIP`, `ACCTNUM`, `STATEID`, `PROPTYPE`, `PROPSUBTYPE`, `INVESTFLAG`, `CLASSID`, `INVTYPE`, `LIFECODE`, `LOCAID`, `JEDESCID`, `OPTED`, `CURRCODE`, `APREALEXCHG`, `APUNREEXCHG`, `CROSSCURYN`, `CMREALACCT`, `CMUNREACCT`, `CMDEPGLACCT`, `RMTACCTNO`, `RMTCASHTYPE`, `BADTAXACCT`, `BADTAXCSHTYPE`, `FIDTAXACCT`, `FIDTAXCSHTYPE`, `TOGACCTNUM`, `TOGCASHTYPE`, `TOGVENDID`, `OTRACCTNUM`, `OTRCASHTYPE`, `OTRVENDID`, `ENTTYPE`, `ALTTAXID`, `TAXSUSPENSE`, `CMNBANKING`, `ARREALACCT`, `ARUNREACCT`, `RETAINACCT`, `BILLFEES`, `FUNDBALACCT`, `TAXRPTFREQ`, `OWNERID`, `COUNTRY`, `APTAXSUSPENSEACCOUNTNUMBER`, `STLINEBASIS`, `ENTMATRIXOPTION`, `NCREIFACQUIREDFIRSTDAYQTR`, `NCREIFDISPOSEDLASTDAYQTR`, `CHECKLOCATIONID`, `APREALLOSS`, `APUNREALLOSS`, `ARREALLOSS`, `ARUNREALLOSS`, `CMREALLOSS`, `CMUNREALLOSS`, `CMDEPLOSS`, `VENDORWITHHOLDINGACCT`, `SRVCDEBT`, `SRVCCRED`, `RMSRVCDEBT`, `RMSRVCCRED`, `IAENTTYPE`, `IAVENDORID`, `IACORPORATEACCOUNTID`, `CONNECTCURRCODE`, `COUNTY`, `ACCREXPACCT`
* **B — Boolean / bit:** `ActiveSegment`, `SegmentAccountFilter`, `SegmentCustomFilter`
* **D — Date:** `LASTDATE`, `M_1099DATE`, `ACQUIRED`, `DISPOSED`
* **N — Numeric:** `MAXOPEN`, `MAXAPOPEN`, `FEET`, `UNITS`, `OPENBAL`, `APPLIMIT`, `GROSSVALUE`, `NETVALUE`, `INCRET`, `CAPRET`, `TOTRET`, `RMTOTHRFEE`, `TOGFEES`, `MINBAL`, `HELDFUNDS`, `PERIODCNT`, `RECLAIMPCT`, `MATRIXID`, `CISREGID`, `INTERENTITYREF`

Report an **Error** where a value cannot be represented as its declared type — for
example non-numeric text in an `N` field, or an unparseable date in a `D` field.
Dates in `dd/mm/yyyy` format are acceptable.

### Allowed Values

**`BASIS` — Cash, Accrual or Both**

| Value | Meaning |
| ----- | ------- |
| `C` | Cash |
| `A` | Accrual |
| `B` | Both |

Any value outside this list is an **Error**.

**`SUMDET` — Summary or Detail (S/D)**

| Value | Meaning |
| ----- | ------- |
| `S` | Summary creates one journal entry for all transactions posted to an expense account |
| `D` | Detail creates a separate journal entry for each transaction posted to an expense account |

Any value outside this list is an **Error**.

**`SUMCASH` — Summarize Cash Account**

| Value | Meaning |
| ----- | ------- |
| `Y` | Summarize creates one journal entry for all transactions posted to the cast account |
| `N` | Creates separate journal entry for each transaction posted to the cash account |

Any value outside this list is an **Error**.

**`INVCSTAT` — Default Invoice Status**

| Value | Meaning |
| ----- | ------- |
| `H` | Hold Payment |
| `I` | Information Only |
| `P` | Pay in next check selection |
| `R` | Release for Payment |

Any value outside this list is an **Error**.

### Fixed Values

These fields are populated by the conversion process and must carry a fixed value.

* `LASTDATE` — must always be `SYSDATE`. Any other value is an **Error**.
* `USERID` — must always be `CONV`. Any other value is an **Error**.
* `OPTED` — must always be `N`. Any other value is an **Error**.
* `PERIODCNT` — must always be `12`. Any other value is an **Error**.
* `TAXSUSPENSE` — must always be `N`. Any other value is an **Error**.
* `BILLFEES` — must always be `N`. Any other value is an **Error**.

### Format Rules

* `ENTITYID` — \*\*no special characters
* `CLOSEDAY` — Day of Month or EOM
* `YEAREND` — YYYYMM
* `ZIPCODE` — No -
* `PHONE` — No () - or .

Report an **Error** where a value does not conform to the stated format.

### Conditional Requirements

These fields are not unconditionally required, but become required — or must be
left blank — depending on how the client system is configured. Report a **Warning**
where the condition cannot be confirmed from the file alone.

* `INTRACCT` — Do not populate unless using MRI interentity accounting
* `ACCTNUM` — Only if using Job Cost
* `APREALLOSS` — If using Accural, this field is required
* `APUNREALLOSS` — If using Accural, this field is required
* `ARREALLOSS` — If using Accural, this field is required
* `ARUNREALLOSS` — If using Accural, this field is required
* `CMREALLOSS` — If using Accural, this field is required
* `CMUNREALLOSS` — If using Accural, this field is required
* `CMDEPLOSS` — If using Accural, this field is required

### Module / Option-Specific Fields

These fields only apply when the corresponding MRI module or regional option is in
use. If the client is not using that option, the field should be left blank —
report populated values as a **Warning** for review rather than an error.

* **International Option:** `VATREG`, `ARREALACCT`, `ARUNREACCT`, `RETAINACCT`, `FUNDBALACCT`
* **Purchase Order:** `TAXEXEMP`, `SHIPADR1`, `SHIPADR2`, `SHIPADR3`, `SHIPCITY`, `SHIPST`, `SHIPZIP`, `BILLADR1`, `BILLADR2`, `BILLADR3`, `BILLCITY`, `BILLST`, `BILLZIP`, `APPLIMIT`
* **Multi-currency Option:** `CURRCODE`, `APREALEXCHG`, `APUNREEXCHG`, `CROSSCURYN`, `CMREALACCT`, `CMUNREACCT`, `CMDEPGLACCT`
* **Australian Option:** `RMTACCTNO`, `RMTCASHTYPE`, `RMTOTHRFEE`, `TOGFEES`, `MINBAL`, `HELDFUNDS`, `BADTAXACCT`, `BADTAXCSHTYPE`, `FIDTAXACCT`, `FIDTAXCSHTYPE`, `TOGACCTNUM`, `TOGCASHTYPE`, `TOGVENDID`, `OTRACCTNUM`, `OTRCASHTYPE`, `OTRVENDID`

### Cross-Reference Rules

The following fields are foreign keys onto other MRI tables. Where the referenced
file is supplied alongside this one, validate the value exists there and report an
**Error** if it does not. Where the referenced data is only available in the target
database, report a **Warning / REQUIRES DATABASE VERIFICATION**.

| MRI Table Field Name | Must Match |
| -------------------- | ---------- |
| `PROJID` | `PROJ.PROJID` |

### Other Field Notes

Remaining guidance carried over from the worksheet.

* `MAXOPEN` — How many GL Periods are allowed to be open simultaneously
* `CASHTYPE` — Enter the default cash type for use in entering invoices.  Can be changed during invoice entry
* `CMNBANKING` — UK Option = Y<br>North America = N
* `STLINEBASIS` — Basis type if FASB 13 entries to separate basis

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
