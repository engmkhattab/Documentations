# SAP ERP ECC6 EHP8 & Amer Group (PVC) Reservation System API Documentation

## System Overview

### Infrastructure
**Environment:** AWS Cloud

| System | Internal IP | External IP | Description |
|--------|-------------|-------------|-------------|
| SAP ERP Server | 172.20.1.5 | - | SAP ECC6 EHP8 System |
| PVC Reservation System | 172.20.1.80 | 34.255.31.82 | Amer Group Reservation Platform |

---

## API Parameters Reference

### Common Parameters (Query Parameters)

| Parameter | Arabic | Description | Type | Example |
|-----------|--------|-------------|------|---------|
| `Bukrs` / `BURKS` | كود الشركة | Company Code | String | `4103` |
| `Swenr` / `PROJ` | المشروع | Project Code | String | `PSHAROW`, `PCV` |
| `Recnnr` | رقم العقد | Contract Number | String | `500000018797` |
| `Kunnr` | كود العميل | Customer Code | String | `0100214489` |
| `Smenr` / `UNIT` | رقم الوحدة | Unit Number | String | `145004` |
| `Ofid` | رقم الاوفر | Offer Number | String | - |

---

## API Endpoints

### 1. Financial Transactions Query
**Query units with financial transactions within a date range**

**Endpoint:**
```
GET http://xccre.amer-group.com/sap/ZCHPAYMENT/paymob
```

**Query Parameters:**
- `sap-client`: SAP client number (required) - `500`
- `BURKS`: Company code (required) - e.g., `4103`
- `DATEFROM`: Start date (required) - Format: `DDMMYYYY`
- `DATETO`: End date (required) - Format: `DDMMYYYY`

**Example Request:**
```
GET http://xccre.amer-group.com/sap/ZCHPAYMENT/paymob?sap-client=500&BURKS=4103&DATEFROM=01072025&DATETO=30072025
```

**Response:** Returns units with financial movements during the specified period

---

### 2. Unit Information Query (ZROSTATUS Report)
**Get unit information based on ZROSTATUS report**

**Endpoint:**
```
GET http://xccre.amer-group.com/sap/zcrmunitdata/crmdata
```

**Query Parameters:**
- `sap-client`: SAP client number (required) - `500`
- `PROJECTMASTER`: Master project code (required) - e.g., `PCV`
- `PROJECTCODE`: Project code (required) - e.g., `4101`
- `PROJECTSATUS`: Project status (required) - e.g., `M001`

**Example Request:**
```
GET http://xccre.amer-group.com/sap/zcrmunitdata/crmdata?sap-client=500&PROJECTMASTER=PCV&PROJECTCODE=4101&PROJECTSATUS=M001
```

---

### 3. Payment Posting (CRM Payment)
**Post payment transactions to SAP**

**Endpoint:**
```
POST http://52.31.28.112:8010/sap/zcrmpayment/crmpay?sap-client=500
```

**Request Body (JSON):**
```json
{
    "DOCNO": "1",
    "BLDAT": "17.02.2024",
    "BLART": "zc",
    "BUKRS": "4103",
    "MONAT": "02",
    "WAERS": "egp",
    "XBLNR": "500000012357",
    "BKTXT": "4103/PSOKNOWV/10718J",
    "PORTF": "C0307",
    "NEWBS": "09",
    "NEWKO": "100170668",
    "NEWUM": "w",
    "WRBTR": "10000",
    "ZUONR": "500000012357",
    "SGTXT": "Paymob (6,000.00) - Other - Dec-2024",
    "ZFBDT": "16.03.2024",
    "WNAME": "",
    "WORT1": "",
    "WBZOG": "C0307",
    "BOENO": "11223344",
    "BANK": "3",
    "ACCOU": "512345xxxxxx2346",
    "XREF1": "500000012357",
    "XREF3": "4103/PSOKNOWV/10718J",
    "BELNR": ""
}
```

**Field Descriptions:**
- `DOCNO`: Document number
- `BLDAT`: Document date (DD.MM.YYYY)
- `BLART`: Document type
- `BUKRS`: Company code
- `MONAT`: Posting month (MM)
- `WAERS`: Currency (e.g., `egp`)
- `XBLNR`: Reference document number
- `BKTXT`: Document header text
- `PORTF`: Portfolio
- `NEWBS`: Posting key
- `NEWKO`: Account number
- `WRBTR`: Amount
- `ZUONR`: Assignment number
- `SGTXT`: Text description
- `ZFBDT`: Baseline date
- `BOENO`: Business entity number
- `BANK`: Bank number
- `ACCOU`: Account number (masked)
- `XREF1`, `XREF3`: Reference fields
- `BELNR`: Document number (output)

---

### 4. Customer Data Query
**Retrieve customer information and membership details**

**Endpoint:**
```
GET https://xccre.amer-group.com:44314/sap/zcrmcustdata/crmdata
```

**Query Parameters:**
- `sap-client`: SAP client number (required) - `500`
- `pr`: Parameter flag - `1`
- `DATEFROM`: Start date - Format: `DDMMYYYY`
- `DATETO`: End date - Format: `DDMMYYYY`
- `PROJECTMASTER`: Master project code - e.g., `PCV`
- `PROJECTCODE`: Project code - e.g., `4101`

**Example Request:**
```
GET https://xccre.amer-group.com:44314/sap/zcrmcustdata/crmdata?sap-client=500&pr=1&DATEFROM=01012023&DATETO=31122023&PROJECTMASTER=PCV&PROJECTCODE=4101
```

**Response (JSON Array):**
```json
[
  {
    "empNameE": "ايمان احمد على احمد القللى",
    "natinalid": "28109250101306",
    "empPasspor": "",
    "mobile": "01221514537",
    "email": "",
    "membershipno": "00000744",
    "iimembershipno": "",
    "bpnumber": "0100214489",
    "personalbalance": "28 day",
    "project": "PCV",
    "zone": "GENERAL",
    "floor": "000",
    "companycode": "4101",
    "masterdataproject": "",
    "cotractstatus": "",
    "contractno": "0500000005081",
    "adress": "ع8 ش قنطره البكريه الظاهر د3ش8",
    "owner": "X"
  }
]
```

**Response Fields:**
- `empNameE`: Customer name (Arabic)
- `natinalid`: National ID
- `empPasspor`: Passport number
- `mobile`: Mobile phone
- `email`: Email address
- `membershipno`: Membership number
- `bpnumber`: Business partner number
- `personalbalance`: Personal balance
- `project`: Project code
- `zone`: Zone
- `floor`: Floor number
- `companycode`: Company code
- `contractno`: Contract number
- `adress`: Address
- `owner`: Owner flag

---

### 5. Customer Payment Details (Customer Statement)
**Retrieve detailed payment information for customers**

**Endpoint:**
```
GET https://xccre.amer-group.com/sap/opu/odata/sap/ZBAPI_CUSTOMERSTATMENT_INT_SRV_01/SDetailsSet
```

**Query Parameters (OData $filter):**
- `Bukrs`: Company code - e.g., `'4103'`
- `Swenr`: Project code - e.g., `'PSHAROW'`
- `Recnnr`: Contract number - e.g., `'500000018797'`
- `Kunnr`: Customer code - Optional, use `''` if not filtering
- `Smenr`: Unit number - Optional, use `''` if not filtering
- `Ofid`: Offer ID - Optional, use `''` if not filtering

**Example Request:**
```
GET https://xccre.amer-group.com/sap/opu/odata/sap/ZBAPI_CUSTOMERSTATMENT_INT_SRV_01/SDetailsSet?$filter=Bukrs eq '4103' and Swenr eq 'PSHAROW' and Recnnr eq '500000018797' and Kunnr eq '' and Smenr eq '' and Ofid eq ''
```

**Response Format:** OData XML feed with Atom format

**Key Response Elements:**
```xml
<d:UnitId1>145004</d:UnitId1>
<d:Kidno>Z001-20240130</d:Kidno>
<d:Condition>تحويل بنكي</d:Condition>
<d:ConditionE>Bank Transfer</d:ConditionE>
<d:PaymentMethod>تحويل بنكي</d:PaymentMethod>
<d:PaymentMethodE>Bank Transfer</d:PaymentMethodE>
<d:DMBTR>0.000</d:DMBTR>
<d:Zfbdt>2024-01-31</d:Zfbdt>
<d:Waers>EGP</d:Waers>
<d:ChequeStatus>تحويل بنكي</d:ChequeStatus>
<d:ChequeStatusE>Bank Transfer</d:ChequeStatusE>
<d:PaidAmount>29000.000</d:PaidAmount>
<d:Buzei>002</d:Buzei>
<d:Sgtxt_code>3202</d:Sgtxt_code>
<d:Sgtxt>20% دفعة مقدمة</d:Sgtxt>
```

**Field Descriptions:**
- `UnitId1`: Unit ID
- `Kidno`: Payment schedule ID
- `Condition`: Payment condition (Arabic)
- `ConditionE`: Payment condition (English)
- `PaymentMethod`: Payment method (Arabic)
- `PaymentMethodE`: Payment method (English)
- `DMBTR`: Document amount
- `Zfbdt`: Payment due date (YYYY-MM-DD)
- `Waers`: Currency
- `ChequeStatus`: Cheque status (Arabic)
- `ChequeStatusE`: Cheque status (English)
- `PaidAmount`: Paid amount
- `Buzei`: Line item number
- `Sgtxt_code`: Text code
- `Sgtxt`: Payment description (Arabic)

---

## Payment Methods & Conditions

### Payment Methods (Arabic / English)
- **تحويل بنكي** / Bank Transfer
- **شيك** / Cheque
- **تسوية** / Reconciliation
- **شيك صيانة** / Maintenance Cheque

### Cheque Status Values
- **حصل بالبنك** / Cleared at bank
- **تحت التحصيل** / Presented to a bank
- **تسوية** / Reconciliation
- **تحويل بنكي** / Bank Transfer

### Condition Types
- **Z001**: دفعه مقدمه / Down Payment
- **Z002**: دفعه ربع سنويه / Quarterly instalment
- **Z006**: وديعه صيانه / Maintenance Advances

### Payment Description Codes (Sgtxt_code)
- **3201**: اقساط / Instalments
- **3202**: Paid instalments
- **3203**: Scheduled payments
- **3204**: Maintenance deposit scheduled
- **3205**: Maintenance deposit paid

---

## Security & Authentication

- All APIs require `sap-client` parameter set to `500`
- HTTPS endpoints use port `44314` for secure communication
- HTTP endpoints use port `8010` for internal communication
- Ensure network connectivity between AWS instances (172.20.1.5 and 172.20.1.80)

---

## Error Handling

### Common HTTP Status Codes
- `200 OK`: Successful request
- `400 Bad Request`: Invalid parameters
- `401 Unauthorized`: Authentication failed
- `404 Not Found`: Resource not found
- `500 Internal Server Error`: Server error

### Best Practices
1. Always validate date formats before sending requests
2. Use proper URL encoding for Arabic text in query parameters
3. Check for empty result sets in responses
4. Implement retry logic for timeout scenarios
5. Log all API calls for audit purposes

---

## Integration Notes

### Date Format Guidelines
- **Financial Query**: `DDMMYYYY` (e.g., `01072025`)
- **Payment Posting**: `DD.MM.YYYY` (e.g., `17.02.2024`)
- **Customer Data**: `DDMMYYYY` (e.g., `01012023`)
- **OData Response**: `YYYY-MM-DD` (e.g., `2024-01-31`)

### Network Configuration
- SAP ERP internal traffic flows through `172.20.1.5`
- PVC Reservation System accessible externally via `34.255.31.82`
- Ensure firewall rules allow communication between both systems
- Use internal IPs for inter-service communication within AWS VPC

---

## Sample Integration Workflow

1. **Query Customer Data** → Get customer and contract information
2. **Check Payment Status** → Retrieve payment history and pending amounts
3. **Post Payment** → Submit new payment transaction
4. **Verify Transaction** → Query financial transactions to confirm posting
5. **Update Unit Status** → Query unit information to reflect changes

---

## Support & Maintenance

For technical support or API issues:
- Check SAP system logs on `172.20.1.5`
- Review PVC system logs on `172.20.1.80`
- Verify network connectivity between systems
- Ensure SAP services are running on correct ports

---

**Document Version:** 1.0  
**Last Updated:** November 2025  
**System:** SAP ERP ECC6 EHP8 & Amer Group PVC Reservation System