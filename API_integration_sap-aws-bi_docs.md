# SAP ERP ECC6 EHP8 & AWS BI Analytics — Sales & Contract Data API Documentation

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

| Parameter | Arabic | Description | Type | Example |
| --- | --- | --- | --- | --- |
| `sap-client` | — | SAP Client / Mandant Number | String | `500` |
| `COMPCODE` | كود الشركة | Company Code | String | `1001`, `0218` |
| `PROJ` | المشروع | Project Code | String | `PHLIOPLS` |
| `UNIT` | رقم الوحدة | Unit Code | String | `A1003` |

---

## API Endpoints

### 1. Sales & Contract Data Query (ZAWSSALESDATA)

**Retrieve real estate contract, unit, customer, and payment data for AWS BI ingestion**

**Endpoint:**

```
GET https://xccre.amer-group.com/sap/ZAWSSALESDATA/CSTDATA
```

**Query Parameters:**

- `sap-client`: SAP client number (required) — `500`
- `COMPCODE`: Company code (required) — e.g., `1001`
- `PROJ`: Project code (optional) — e.g., `PHLIOPLS`
- `UNIT`: Unit code (optional) — e.g., `A1003`

**Example Requests:**

```
# Filter by company only
GET https://xccre.amer-group.com/sap/ZAWSSALESDATA/CSTDATA?sap-client=500&COMPCODE=0218

# Filter by company + project
GET https://xccre.amer-group.com/sap/ZAWSSALESDATA/CSTDATA?sap-client=500&COMPCODE=1001&PROJ=PHLIOPLS

# Filter by company + project + unit
GET https://xccre.amer-group.com/sap/ZAWSSALESDATA/CSTDATA?sap-client=500&COMPCODE=1001&PROJ=PHLIOPLS&UNIT=A1003
```

**Response (JSON Array):**

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

---

## Response Field Reference

### Header Fields — Contract & Unit Level

| Field | Arabic | Description | Type | Required | SAP Source | Example |
| --- | --- | --- | --- | --- | --- | --- |
| `cocode` | كود الشركة | Company code in SAP | String | Yes | T001 (BUKRS) | `1001` |
| `unitPrice` | سعر الوحدة بدون الصيانة | Unit price excluding maintenance (string format) | String | No | VICDCON | `Unit price : 6298000` |
| `unitId` | رقم الوحدة | Unit identifier: `CompCode/Project/Unit` | String | Yes | VIBDBE (OBJNR) | `1001/PHLIOPLS/A1003` |
| `contractNo1` | رقم العقد | SAP lease contract number | String | Yes | VIMI01 (MIETVERTRAG) | `0500000000379` |
| `customerNo` | رقم العميل | SAP customer number | String | Yes | KNA1 (KUNNR) | `0100207213` |
| `customerName` | اسم العميل | Customer full name in Arabic | String | Yes | KNA1 (NAME1) | `أياد محمد المصطفى` |
| `unitarea` | مساحة الوحدة | Unit area in square meters | Number | Yes | VIBDBE (FLAECHE) | `170` |
| `totalunitamount` | إجمالي سعر الوحدة | Total unit price excluding maintenance | Number | Yes | VICDCON | `6298000` |
| `contractdate` | تاريخ العقد | Contract signing date | Date | Yes | VIMI01 (MIETBEGINN) | `2022-08-15` |
| `dmbtr` | سعر الوحدة بالصيانة | Unit price including maintenance (local currency) | Number | Yes | BSEG (DMBTR) | `7053760` |

### Items Array — Payment Document Fields (FI)

Each object in the `items` array represents one Financial Accounting (FI) document entry.

| Field | Arabic | Description | Type | Required | SAP Source | Example |
| --- | --- | --- | --- | --- | --- | --- |
| `belnr` | رقم قيد السداد | FI payment document number | String | Yes | BKPF (BELNR) | `1400000422` |
| `gjahr` | سنة القيد | Fiscal year of the document | Number | Yes | BKPF (GJAHR) | `2022` |
| `type` | نوع القيد | Document type in Arabic | String | Yes | BKPF (BLART) | `نقدى` |
| `budat` | تاريخ القيد | Document posting date | Date | Yes | BKPF (BUDAT) | `2022-08-15` |
| `paidamount` | قيمة السداد بالقيد | Amount paid in this document | Number | Yes | BSEG (DMBTR) | `10000` |
| `paymentmethod` | طريقة السداد | Payment method in English | String | Yes | BSEG | `Cash` |
| `status` | الحالة | Payment status | String | No | — | `""` |
| `bank` | بنك التحصيل | Collecting bank name | String | No | — | `""` |
| `sgtxt` | وصف القيد | Document description in Arabic | String | No | BSEG (SGTXT) | `عربون حجز` |

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

- **Cash** / نقدى
- **Bank Transfer** / تحويل بنكي
- **Cheque** / شيك
- **Maintenance Cheque** / شيك صيانة

### Common Document Descriptions (sgtxt)

- **عربون حجز** — Reservation deposit
- **دفعة مقدمة** — Down payment
- **قسط** — Instalment
- **وديعة صيانة** — Maintenance deposit

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

### Key Business Rules

- `unitId` is composed of `{COMPCODE}/{PROJ}/{UNIT}` — matches the three query parameters exactly
- `totalunitamount` is the base price **excluding** maintenance; `dmbtr` **includes** maintenance charges (RE-FX + MM)
- `unitPrice` returns a prefixed string (e.g., `"Unit price : 6298000"`) — parse the numeric portion for AWS BI calculations
- The `items` array may contain zero or more entries; always handle an empty array gracefully
- All monetary values are in **Saudi Riyals (SAR)** unless otherwise indicated

---

## Sample Integration Workflow

1. **Query by Company** → Fetch all units for a given `COMPCODE` to populate the master unit list
2. **Drill Down by Project** → Add `PROJ` filter to scope data to a specific real estate project
3. **Fetch Single Unit** → Add `UNIT` filter to retrieve full payment history for one unit
4. **Load Header Data** → Map header fields to the AWS BI contracts/units dimension table
5. **Load Payment Items** → Map `items` array entries to the AWS BI payments fact table
6. **Reconcile Amounts** → Cross-check `totalunitamount` vs sum of `paidamount` across items for payment progress

---

## Support & Maintenance

For technical support or API issues:

- Check SAP system logs on the ECC6 EHP8 application server
- Verify that the ZAWSSALESDATA service is active in SAP transaction SICF
- Confirm `sap-client=500` is correct for the target environment
- Review ABAP short dumps in SAP transaction ST22 for `500` errors
- Contact the SAP BASIS team for RFC connectivity or service availability issues

---

**Document Version:** 1.0  
**Last Updated:** April 2026  
**System:** SAP ERP ECC6 EHP8 & AWS BI Analytics — ZAWSSALESDATA/CSTDATA  
**Modules:** RE-FX · FI · CO · MM · PS
