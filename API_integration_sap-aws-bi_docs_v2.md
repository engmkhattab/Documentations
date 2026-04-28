# SAP ERP ECC6 EHP8 & AWS BI Analytics — Sales & Contract Data API Documentation

**Document Version:** 2.0
**Last Updated:** April 2026
**System:** SAP ERP ECC6 EHP8 & AWS BI Analytics — ZAWSSALESDATA/CSTDATA
**Modules:** RE-FX · FI · CO · MM · PS

---

## Revision History

| Version | Date | Description |
| --- | --- | --- |
| 1.0 | April 2026 | Initial release |
| 2.0 | April 2026 | Added full example endpoint with all parameters; expanded and restructured field declarations with data types, constraints, nullability, and transformation notes for AWS BI ingestion |

---

## System Overview

### Infrastructure

**Environment:** AWS Cloud

| System | Service | Description |
| --- | --- | --- |
| SAP ERP Server | ECC6 EHP8 | SAP Core ERP — RE-FX, FI, CO, MM, PS Modules |
| AWS BI Analytics | Analytics Platform | Amer Group Business Intelligence & Reporting |

### SAP Report Details

| Property | Value |
| --- | --- |
| **Transaction Code (TCode)** | `ZCUST2` |
| **Report Type** | Customized ABAP Report |
| **Service Name** | `ZAWSSALESDATA/CSTDATA` |
| **SAP Client** | `500` |

---

## API Parameters Reference

### Common Parameters (Query Parameters)

| Parameter | Arabic | Description | Type | Required | Example |
| --- | --- | --- | --- | --- | --- |
| `sap-client` | — | SAP Client / Mandant Number | String | **Yes** | `500` |
| `COMPCODE` | كود الشركة | Company Code | String | **Yes** | `1001`, `0218` |
| `PROJ` | المشروع | Project Code | String | No | `PHLIOPLS` |
| `UNIT` | رقم الوحدة | Unit Code | String | No | `A1003` |

> **Note:** `sap-client` and `COMPCODE` are mandatory on every request. `PROJ` and `UNIT` are optional filters that narrow the response scope and reduce payload size.

---

## API Endpoints

### 1. Sales & Contract Data Query (ZAWSSALESDATA)

**Retrieve real estate contract, unit, customer, and payment data for AWS BI ingestion**

**Base URL:**

```
https://xccre.amer-group.com/sap/ZAWSSALESDATA/CSTDATA
```

**Method:** `GET`

**Content-Type:** `application/json`

---

### Example Requests

#### Filter by Company Code Only

```
GET https://xccre.amer-group.com/sap/ZAWSSALESDATA/CSTDATA?sap-client=500&COMPCODE=0218
```

#### Filter by Company Code + Project

```
GET https://xccre.amer-group.com/sap/ZAWSSALESDATA/CSTDATA?sap-client=500&COMPCODE=1001&PROJ=PHLIOPLS
```

#### Filter by Company Code + Project + Unit *(Full Precision — Recommended for Single Unit Lookup)*

```
GET https://xccre.amer-group.com/sap/ZAWSSALESDATA/CSTDATA?sap-client=500&COMPCODE=1001&PROJ=PHLIOPLS&UNIT=A1003
```

> **This is the fully qualified endpoint for unit-level data retrieval.** It pins all three business dimensions — company, project, and unit — and returns the complete contract header plus full payment history for unit `A1003` under project `PHLIOPLS` in company `1001`.

---

## Response Structure

```json
[
  {
    "cocode": "1001",
    "unitPrice": "Unit price : 6298000",
    "unitId": "1001/PHLIOPLS/A1003",
    "contractNo1": "0500000000379",
    "customerNo": "0100207213",
    "customerName": "أياد محمد المصطفى محمد سليمان",
    "unitarea": 170,
    "totalunitamount": 6298000,
    "contractdate": "2022-08-15",
    "dmbtr": 7053760,
    "items": [
      {
        "belnr": "1400000422",
        "gjahr": 2022,
        "type": "نقدى",
        "budat": "2022-08-15",
        "paidamount": 10000,
        "paymentmethod": "Cash",
        "status": "",
        "bank": "",
        "sgtxt": "عربون حجز"
      }
    ]
  }
]
```

The response is a **JSON Array** where each element represents one contract-unit record. Each record contains a **header object** (contract & unit level) and a nested **`items` array** (payment documents).

---

## Field Declarations — Version 2.0

### Section A — Header Fields (Contract & Unit Level)

These fields describe the real estate unit and its associated sales contract. They map to the **contracts/units dimension table** in AWS BI.

---

#### `cocode`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | كود الشركة |
| **Description** | Company code identifying the legal entity in SAP |
| **Data Type** | `String` |
| **Max Length** | 4 characters |
| **Nullable** | No — always present |
| **SAP Source** | Table: `T001` · Field: `BUKRS` · Module: FI |
| **Example** | `"1001"`, `"0218"` |
| **AWS BI Notes** | Use as a foreign key to the company dimension table. Store as `VARCHAR(4)`. Do not cast to integer — leading zeros are significant (e.g., `"0218"`). |

---

#### `unitPrice`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | سعر الوحدة بدون الصيانة |
| **Description** | Unit base price excluding maintenance, returned as a prefixed string |
| **Data Type** | `String` (raw) → `Decimal` (parsed) |
| **Nullable** | Yes — may be absent or null for some contracts |
| **SAP Source** | Table: `VICDCON` · Module: RE-FX |
| **Example** | `"Unit price : 6298000"` |
| **AWS BI Notes** | **Must be parsed before ingestion.** Strip the prefix `"Unit price : "` and cast the numeric portion to `DECIMAL(18,2)`. Treat null or unparseable values as `NULL` in the target column. Cross-reference against `totalunitamount` — both should reflect the same base price. |
| **Parse Pattern** | `value.replace("Unit price : ", "").trim()` → cast to Decimal |

---

#### `unitId`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | رقم الوحدة |
| **Description** | Composite unit identifier composed of three business keys: `{COMPCODE}/{PROJ}/{UNIT}` |
| **Data Type** | `String` |
| **Format** | `{CompanyCode}/{ProjectCode}/{UnitCode}` |
| **Nullable** | No — always present |
| **SAP Source** | Table: `VIBDBE` · Field: `OBJNR` · Module: RE-FX |
| **Example** | `"1001/PHLIOPLS/A1003"` |
| **AWS BI Notes** | Acts as the **natural primary key** for the unit dimension. Store as `VARCHAR(50)`. Decompose into three derived columns (`dim_company`, `dim_project`, `dim_unit`) by splitting on `/` for dimensional modeling. Matches the three query parameters exactly. |

---

#### `contractNo1`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | رقم العقد |
| **Description** | SAP lease contract number — uniquely identifies the sales contract in RE-FX |
| **Data Type** | `String` |
| **Max Length** | 13 characters (zero-padded) |
| **Nullable** | No — always present |
| **SAP Source** | Table: `VIMI01` · Field: `MIETVERTRAG` · Module: RE-FX |
| **Example** | `"0500000000379"` |
| **AWS BI Notes** | Store as `VARCHAR(13)`. Preserve leading zeros — do not cast to integer. Use as a foreign key linking the unit header to the payments fact table via `items[].belnr`. |

---

#### `customerNo`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | رقم العميل |
| **Description** | SAP customer master number — uniquely identifies the buyer in the FI/AR subledger |
| **Data Type** | `String` |
| **Max Length** | 10 characters (zero-padded) |
| **Nullable** | No — always present |
| **SAP Source** | Table: `KNA1` · Field: `KUNNR` · Module: FI |
| **Example** | `"0100207213"` |
| **AWS BI Notes** | Store as `VARCHAR(10)`. Preserve leading zeros. Use as a foreign key to the customer dimension table. Pair with `customerName` for display; use `customerNo` for joins. |

---

#### `customerName`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | اسم العميل |
| **Description** | Customer full legal name in Arabic as registered in SAP customer master |
| **Data Type** | `String` |
| **Encoding** | UTF-8 / Arabic RTL text |
| **Max Length** | 35 characters (SAP NAME1 field limit) |
| **Nullable** | No — always present |
| **SAP Source** | Table: `KNA1` · Field: `NAME1` · Module: FI |
| **Example** | `"أياد محمد المصطفى محمد سليمان"` |
| **AWS BI Notes** | Store as `NVARCHAR(35)` to support Arabic Unicode characters. Ensure the AWS BI target database collation supports Arabic (`Arabic_CI_AS` or `UTF8`). Use for display only; always join on `customerNo`. |

---

#### `unitarea`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | مساحة الوحدة |
| **Description** | Unit gross area measured in square meters |
| **Data Type** | `Number` → `Decimal` |
| **Unit of Measure** | Square Meters (m²) |
| **Nullable** | No — always present |
| **SAP Source** | Table: `VIBDBE` · Field: `FLAECHE` · Module: RE-FX / MM |
| **Example** | `170` |
| **AWS BI Notes** | Store as `DECIMAL(10,2)`. Used in per-sqm price calculations: `totalunitamount / unitarea`. Verify against MM measurement documents if discrepancies arise. |

---

#### `totalunitamount`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | إجمالي سعر الوحدة |
| **Description** | Total unit sale price **excluding** maintenance charges, in local currency (SAR) |
| **Data Type** | `Number` → `Decimal` |
| **Currency** | Saudi Riyals (SAR) |
| **Nullable** | No — always present |
| **SAP Source** | Table: `VICDCON` · Module: RE-FX |
| **Example** | `6298000` |
| **AWS BI Notes** | Store as `DECIMAL(18,2)`. This is the **contract base amount** used for payment progress reconciliation. Formula: `SUM(items[].paidamount) / totalunitamount × 100` = payment completion %. Compare with `dmbtr` to derive the maintenance premium: `dmbtr - totalunitamount`. |

---

#### `contractdate`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | تاريخ العقد |
| **Description** | Date the sales contract was signed / lease commencement date |
| **Data Type** | `Date` |
| **Format** | `YYYY-MM-DD` |
| **Nullable** | No — always present |
| **SAP Source** | Table: `VIMI01` · Field: `MIETBEGINN` · Module: RE-FX |
| **Example** | `"2022-08-15"` |
| **AWS BI Notes** | Store as `DATE`. Convert to ISO 8601 for AWS BI ingestion: `2022-08-15T00:00:00Z`. Use as the anchor date for instalment schedule calculations and aging analysis. |

---

#### `dmbtr`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | سعر الوحدة بالصيانة |
| **Description** | Total unit price **including** maintenance charges, in local currency (SAR) |
| **Data Type** | `Number` → `Decimal` |
| **Currency** | Saudi Riyals (SAR) |
| **Nullable** | No — always present |
| **SAP Source** | Table: `BSEG` · Field: `DMBTR` · Module: FI |
| **Example** | `7053760` |
| **AWS BI Notes** | Store as `DECIMAL(18,2)`. Represents the **full liability amount** on the FI accounting document. Maintenance premium = `dmbtr - totalunitamount`. This is the figure used for financial reporting and AR reconciliation. |

---

### Section B — Items Array Fields (Payment Document Level)

Each element in the `items` array represents one Financial Accounting (FI) document entry linked to this contract. Items map to the **payments fact table** in AWS BI.

---

#### `belnr`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | رقم قيد السداد |
| **Description** | FI accounting document number — uniquely identifies the payment posting in SAP |
| **Data Type** | `String` |
| **Max Length** | 10 characters (zero-padded) |
| **Nullable** | No — always present within an item |
| **SAP Source** | Table: `BKPF` · Field: `BELNR` · Module: FI |
| **Example** | `"1400000422"` |
| **AWS BI Notes** | Store as `VARCHAR(10)`. Composite key for the payments fact table: `belnr + gjahr + cocode`. Preserve leading zeros. Links to the contract header via `contractNo1`. |

---

#### `gjahr`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | سنة القيد |
| **Description** | SAP fiscal year in which the payment document was posted |
| **Data Type** | `Number` → `Integer` |
| **Format** | 4-digit year (Gregorian) |
| **Nullable** | No — always present within an item |
| **SAP Source** | Table: `BKPF` · Field: `GJAHR` · Module: FI |
| **Example** | `2022` |
| **AWS BI Notes** | Store as `SMALLINT` or `INT`. Required as part of the composite primary key with `belnr` and `cocode`. Use for fiscal year partitioning in AWS BI. |

---

#### `type`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | نوع القيد |
| **Description** | SAP FI document type describing the nature of the accounting entry, in Arabic |
| **Data Type** | `String` |
| **Nullable** | No — always present within an item |
| **SAP Source** | Table: `BKPF` · Field: `BLART` · Module: FI |
| **Example** | `"نقدى"` (Cash), `"تحويل بنكي"` (Bank Transfer), `"شيك"` (Cheque) |
| **AWS BI Notes** | Store as `NVARCHAR(20)`. For English-language BI reports, maintain a lookup/translation table mapping Arabic document type values to English equivalents. Cross-reference with `paymentmethod` for validation. |

---

#### `budat`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | تاريخ القيد |
| **Description** | Posting date of the FI payment document — the date the payment was recorded in SAP |
| **Data Type** | `Date` |
| **Format** | `YYYY-MM-DD` |
| **Nullable** | No — always present within an item |
| **SAP Source** | Table: `BKPF` · Field: `BUDAT` · Module: FI |
| **Example** | `"2022-08-15"` |
| **AWS BI Notes** | Store as `DATE`. Convert to ISO 8601 for AWS BI: `2022-08-15T00:00:00Z`. Use for payment timeline analysis, overdue tracking, and instalment schedule comparison. |

---

#### `paidamount`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | قيمة السداد بالقيد |
| **Description** | Monetary amount paid and recorded in this specific FI document, in local currency (SAR) |
| **Data Type** | `Number` → `Decimal` |
| **Currency** | Saudi Riyals (SAR) |
| **Nullable** | No — always present within an item |
| **SAP Source** | Table: `BSEG` · Field: `DMBTR` · Module: FI |
| **Example** | `10000` |
| **AWS BI Notes** | Store as `DECIMAL(18,2)`. Core measure in the payments fact table. Aggregate: `SUM(paidamount)` per `contractNo1` for total collected amount. Payment completion %: `SUM(paidamount) / totalunitamount × 100`. |

---

#### `paymentmethod`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | طريقة السداد |
| **Description** | Payment method used for this document, in English |
| **Data Type** | `String` |
| **Nullable** | No — always present within an item |
| **SAP Source** | Table: `BSEG` · Module: FI |
| **Allowed Values** | `Cash`, `Bank Transfer`, `Cheque`, `Maintenance Cheque` |
| **Example** | `"Cash"` |
| **AWS BI Notes** | Store as `VARCHAR(30)`. Use for payment method distribution analysis. Cross-reference with `type` (Arabic) to validate consistency. Index this column in AWS BI for high-frequency filtering. |

---

#### `status`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | الحالة |
| **Description** | Payment status indicator for this document entry |
| **Data Type** | `String` |
| **Nullable** | Yes — frequently empty |
| **SAP Source** | Derived / Custom field |
| **Example** | `""` (empty), or a status code when populated |
| **AWS BI Notes** | Store as `VARCHAR(50)`. **Treat empty string `""` as `NULL`** in the AWS BI target — do not store blank strings. When populated, use as a filter for cleared vs. open payment items. |

---

#### `bank`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | بنك التحصيل |
| **Description** | Name of the collecting bank through which the payment was processed |
| **Data Type** | `String` |
| **Nullable** | Yes — only populated for bank transfer and cheque payments |
| **SAP Source** | Derived / Custom field |
| **Example** | `""` (for cash), `"البنك الأهلي"` (for bank transfers) |
| **AWS BI Notes** | Store as `NVARCHAR(100)`. **Treat empty string `""` as `NULL`**. Only relevant when `paymentmethod` is `Bank Transfer` or `Cheque`. Use for bank collection analysis and reconciliation reports. |

---

#### `sgtxt`

| Attribute | Value |
| --- | --- |
| **Arabic Label** | وصف القيد |
| **Description** | Free-text description of the payment document as entered in SAP, in Arabic |
| **Data Type** | `String` |
| **Encoding** | UTF-8 / Arabic RTL text |
| **Max Length** | 50 characters (SAP SGTXT field limit) |
| **Nullable** | Yes — optional description field |
| **SAP Source** | Table: `BSEG` · Field: `SGTXT` · Module: FI |
| **Example** | `"عربون حجز"` (Reservation deposit), `"قسط"` (Instalment) |
| **AWS BI Notes** | Store as `NVARCHAR(50)`. For categorization in AWS BI, maintain a lookup table mapping common Arabic `sgtxt` values to English labels and payment stage codes. See common values table below. |

---

### Common `sgtxt` Values — Reference Table

| Arabic Value | English Translation | Payment Stage |
| --- | --- | --- |
| `عربون حجز` | Reservation deposit | Stage 1 — Booking |
| `دفعة مقدمة` | Down payment | Stage 2 — Down Payment |
| `قسط` | Instalment | Stage 3–N — Instalment |
| `وديعة صيانة` | Maintenance deposit | Final — Maintenance |

---

## SAP Module Reference

### Modules Providing Data to This API

| Module | Full Name | Data Provided |
| --- | --- | --- |
| **RE-FX** | Flexible Real Estate Management | Unit master data, contracts, lease terms, rental objects |
| **FI** | Financial Accounting | Payment documents (belnr, budat, dmbtr), company code, fiscal year |
| **CO** | Controlling | Cost center allocations, internal orders linked to contracts |
| **MM** | Materials Management | Unit area measurements, maintenance service orders |
| **PS** | Project System | Project codes (PROJ), WBS elements for real estate projects |

### Field-to-Module Mapping

| Field | SAP Module | SAP Table / Object | Notes |
| --- | --- | --- | --- |
| `cocode` | FI | T001 (BUKRS) | Company code master record |
| `unitId` | RE-FX | VIBDBE (OBJNR) | Rental object identifier |
| `contractNo1` | RE-FX | VIMI01 (MIETVERTRAG) | Lease contract number |
| `customerNo` / `customerName` | FI | KNA1 (KUNNR / NAME1) | Customer master data |
| `unitarea` | MM | VIBDBE (FLAECHE) | Area in sqm from RE-FX / MM |
| `totalunitamount` / `unitPrice` | RE-FX | VICDCON | Base price excluding maintenance |
| `dmbtr` | FI | BSEG (DMBTR) | Local currency amount including maintenance |
| `contractdate` | RE-FX | VIMI01 (MIETBEGINN) | Contract start / signing date |
| `belnr` | FI | BKPF (BELNR) | FI accounting document number |
| `gjahr` | FI | BKPF (GJAHR) | Fiscal year of document |
| `budat` | FI | BKPF (BUDAT) | Posting date in FI |
| `paidamount` | FI | BSEG (DMBTR) | Amount on payment line item |
| `PROJ` (query param) | PS | PROJ (PSPNR) | Project system WBS code |

---

## Payment Methods & Document Types

### Payment Methods

| English | Arabic | Notes |
| --- | --- | --- |
| Cash | نقدى | No bank field populated |
| Bank Transfer | تحويل بنكي | `bank` field populated |
| Cheque | شيك | `bank` field populated |
| Maintenance Cheque | شيك صيانة | Linked to maintenance deposit |

---

## Security & Authentication

- All requests require the `sap-client` parameter set to `500`
- SAP RFC/HTTP basic authentication is required on every request
- HTTPS is used for all production endpoints
- Requests without valid credentials return `401 Unauthorized`

---

## Error Handling

### HTTP Status Codes

| Code | SAP Status | Description |
| --- | --- | --- |
| `200 OK` | S / Success | Request succeeded — JSON array returned |
| `200 OK (empty)` | W / Warning | No records match the filter — returns `[]` |
| `400 Bad Request` | E / Error | Missing or invalid query parameter |
| `401 Unauthorized` | — | Invalid SAP credentials or expired session |
| `404 Not Found` | — | Service not found — verify `sap-client` value |
| `500 Internal Server Error` | A / Abort | SAP ABAP runtime error — contact BASIS team |

### Best Practices

1. Always include both `sap-client` and `COMPCODE` — they are required on every request
2. Use `PROJ` and `UNIT` filters to reduce payload size for large company codes
3. Handle empty `items` arrays — a contract may exist with no payment documents yet
4. Treat empty string values (`""`) in `status` and `bank` as `NULL` in AWS BI
5. Log all API calls with timestamp and parameters for audit purposes
6. Implement retry logic for `500` errors with exponential back-off

---

## Integration Notes

### Date Format Guidelines

| Context | Format | Example |
| --- | --- | --- |
| API response — contract fields | `YYYY-MM-DD` | `2022-08-15` |
| API response — items (budat) | `YYYY-MM-DD` | `2022-08-15` |
| AWS BI target format | ISO 8601 | `2022-08-15T00:00:00Z` |

### AWS BI Target Schema Summary

| Field | AWS BI Column Type | Table Target | Nullable |
| --- | --- | --- | --- |
| `cocode` | `VARCHAR(4)` | contracts_dim | No |
| `unitPrice` | `DECIMAL(18,2)` *(parsed)* | contracts_dim | Yes |
| `unitId` | `VARCHAR(50)` | units_dim (PK) | No |
| `contractNo1` | `VARCHAR(13)` | contracts_dim | No |
| `customerNo` | `VARCHAR(10)` | customers_dim (FK) | No |
| `customerName` | `NVARCHAR(35)` | customers_dim | No |
| `unitarea` | `DECIMAL(10,2)` | units_dim | No |
| `totalunitamount` | `DECIMAL(18,2)` | contracts_dim | No |
| `contractdate` | `DATE` | contracts_dim | No |
| `dmbtr` | `DECIMAL(18,2)` | contracts_dim | No |
| `belnr` | `VARCHAR(10)` | payments_fact | No |
| `gjahr` | `SMALLINT` | payments_fact | No |
| `type` | `NVARCHAR(20)` | payments_fact | No |
| `budat` | `DATE` | payments_fact | No |
| `paidamount` | `DECIMAL(18,2)` | payments_fact | No |
| `paymentmethod` | `VARCHAR(30)` | payments_fact | No |
| `status` | `VARCHAR(50)` | payments_fact | Yes |
| `bank` | `NVARCHAR(100)` | payments_fact | Yes |
| `sgtxt` | `NVARCHAR(50)` | payments_fact | Yes |

### Key Business Rules

- `unitId` is composed of `{COMPCODE}/{PROJ}/{UNIT}` — matches the three query parameters exactly
- `totalunitamount` is the base price **excluding** maintenance; `dmbtr` **includes** maintenance charges
- `unitPrice` returns a prefixed string — parse the numeric portion before AWS BI ingestion
- The `items` array may contain zero or more entries; always handle an empty array gracefully
- All monetary values are in **Saudi Riyals (SAR)** unless otherwise indicated
- Composite primary key for payments fact table: `belnr + gjahr + cocode`

---

## Sample Integration Workflow

1. **Query by Company** → Fetch all units for a given `COMPCODE` to populate the master unit list
2. **Drill Down by Project** → Add `PROJ` filter to scope data to a specific real estate project
3. **Fetch Single Unit** → Use the fully qualified endpoint:
   ```
   GET https://xccre.amer-group.com/sap/ZAWSSALESDATA/CSTDATA?sap-client=500&COMPCODE=1001&PROJ=PHLIOPLS&UNIT=A1003
   ```
4. **Load Header Data** → Map header fields to the AWS BI contracts/units dimension table
5. **Load Payment Items** → Map `items` array entries to the AWS BI payments fact table
6. **Reconcile Amounts** → Cross-check `totalunitamount` vs `SUM(paidamount)` across items for payment progress

---

## Support & Maintenance

For technical support or API issues:

- Check SAP system logs on the ECC6 EHP8 application server
- Verify that the ZAWSSALESDATA service is active in SAP transaction `SICF`
- Confirm `sap-client=500` is correct for the target environment
- Review ABAP short dumps in SAP transaction `ST22` for `500` errors
- Contact the SAP BASIS team for RFC connectivity or service availability issues
