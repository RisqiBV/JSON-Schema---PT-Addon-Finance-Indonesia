# Addon Finance Indonesia — Data Contracts (JSON)

**Version:** 1.0 • **Last updated:** 2025-09-19 (Asia/Jakarta)

This README consolidates the JSON data contracts you specified across **Master Data**, **Orders & Production**, **Warehouse**, **Logistics**, and **Sales**. It captures naming conventions, timestamp formats, numeric precision, required fields, and representative examples.

> All JSON keys use **camelCase**. All schemas use `"additionalProperties": false` unless noted.

---

## 1) Conventions

### 1.1 Keys & Naming

- **camelCase** for all keys (e.g., `itemCode`, `unitCode`, `poNo`).
- Columns containing spaces or slashes are normalized (e.g., `State/Province` → `stateOrProvince`).

### 1.2 Datetime

- SQL Server–style pattern: **`yyyy-MM-dd HH:mm:ss.SSS`**
  - JSON Schema: `pattern: ^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}\.\d{3}$`
- If you prefer ISO 8601 (`format: "date-time"`), you can switch per schema.

### 1.3 Numeric Precision

- SQL `decimal(p, s)` is encoded with **`multipleOf`**:
  - `decimal(18,2)` → `multipleOf: 0.01`
  - `decimal(18,4)` → `multipleOf: 0.0001`
  - `decimal(20,5)` → `multipleOf: 0.00001`

### 1.4 Enums

- `orderType`: `FOB` or `CMT`
- `fc`: `PO` or `FOC`
- `category` (Sales): `Export` or `Local`

### 1.5 Cross-References

- `categoryCodeBC` → **ListOfCategoryBC.CategoryCode**
- `aliasUnitCode` → **UnitINSW.code**
- `poNo` ties `Production Order`, `BOM`, `Dispatch`, `Delivery Note`, `Sales`, and receiving transactions
- `batchNo` references batches from **Goods Received**

---

## 2) Master Data

### 2.1 INSW Item Master

**Purpose:** Core item catalog including INSW classification and internal category.

**Required:** `itemCode`, `itemName`, `categoryCodeBC`, `categoryInternal`, `createdAt`, `updatedAt`, `deletedAt`

| Field            | Type              | Notes                                   |
| ---------------- | ----------------- | --------------------------------------- |
| itemCode         | string(50)        | Primary key                             |
| itemName         | string(250)       |                                         |
| categoryCodeBC   | integer           | Refers to ListOfCategoryBC.CategoryCode |
| hsCode           | string(10)\|null  | Optional                                |
| categoryInternal | string(50)        | Internal classification                 |
| createdAt        | string (DateTime) | `yyyy-MM-dd HH:mm:ss.SSS`               |
| updatedAt        | string (DateTime) | `yyyy-MM-dd HH:mm:ss.SSS`               |
| deletedAt        | string\|null      | `yyyy-MM-dd HH:mm:ss.SSS` or null       |

**Example**

```json
{
  "itemCode": "ITM-0001",
  "itemName": "Plastic Pellets",
  "categoryCodeBC": 2,
  "hsCode": "39011090",
  "categoryInternal": "Raw Material",
  "createdAt": "2025-04-10 11:46:07.333",
  "updatedAt": "2025-04-11 09:15:22.127",
  "deletedAt": null
}
```

---

### 2.2 Unit Master

**Required:** `internalUnitCode`, `unitDescription`, `aliasUnitCode`

| Field            | Type       | Notes                   |
| ---------------- | ---------- | ----------------------- |
| internalUnitCode | string(10) | Primary key             |
| unitDescription  | string(50) |                         |
| aliasUnitCode    | string(10) | Refers to UnitINSW.code |

**Example**

```json
{
  "internalUnitCode": "PCS",
  "unitDescription": "Pieces",
  "aliasUnitCode": "PCE"
}
```

---

### 2.3 Supplier Master

**Required:** `supplierCode`, `supplierName`, `address`, `countryOrRegion`

| Field           | Type             | Notes                        |
| --------------- | ---------------- | ---------------------------- |
| supplierCode    | string(50)       | Primary key                  |
| supplierName    | string(200)      |                              |
| address         | string(200)      |                              |
| stateOrProvince | string(50)\|null | Optional                     |
| countryOrRegion | string(50)       |                              |
| city            | string(50)\|null | Optional                     |
| npwp            | string(50)\|null | Optional (Indonesian tax ID) |
| emailAddress    | string(50)\|null | `format: email`              |
| businessPhone   | string(25)\|null |                              |
| faxNo           | string(25)\|null |                              |
| zipOrPostalCode | string(15)\|null |                              |

---

### 2.4 Customer Master

**Required:** `customerCode`, `customerName`, `address`, `country`

| Field        | Type             | Notes       |
| ------------ | ---------------- | ----------- |
| customerCode | string(10)       | Primary key |
| customerName | string(50)       |             |
| address      | string(50)       |             |
| city         | string(50)\|null | Optional    |
| province     | string(50)\|null | Optional    |
| country      | string(50)       |             |
| npwp         | string(50)\|null | Optional    |

---

## 3) Orders & Production

### 3.1 Production Order (Header)

**Required:** `poNo`, `article`, `productGroup`, `qty`, `orderType`, `sku`, `unitCode`

| Field        | Type           | Notes                     |
| ------------ | -------------- | ------------------------- |
| poNo         | string(25)     | PRO No (Primary)          |
| article      | string(50)     |                           |
| startDate    | string\|null   | `yyyy-MM-dd HH:mm:ss.SSS` |
| productGroup | string(50)     |                           |
| qty          | number         |                           |
| orderType    | enum(FOB, CMT) |                           |
| sku          | string(50)     | SKU for finished goods    |
| unitCode     | string(10)     |                           |

**Example**

```json
{
  "poNo": "PO-24-000789",
  "article": "T-Shirt Classic 180gsm",
  "startDate": "2025-09-22 08:00:00.000",
  "productGroup": "Apparel",
  "qty": 5000,
  "orderType": "CMT",
  "sku": "TS-CLS-180-BLK-MIX",
  "unitCode": "PCS"
}
```

---

### 3.2 Bill of Material (BOM) — Line

**Required:** `poNo`, `materialCode`, `consumption`, `unitCode`, `poQty`

| Field        | Type       | Notes                                 |
| ------------ | ---------- | ------------------------------------- |
| poNo         | string(25) | PRO No                                |
| materialCode | string(50) | ItemCode of material                  |
| consumption  | number     | `multipleOf: 0.00001` (numeric(18,5)) |
| unitCode     | string(10) | Material UOM                          |
| poQty        | number     | Production quantity                   |

**Example**

```json
{
  "poNo": "PO-24-000789",
  "materialCode": "FAB-COT-180-BLK",
  "consumption": 0.32,
  "unitCode": "KG",
  "poQty": 5000
}
```

---

### 3.3 Purchase Order (Header + Detail)

**Header Required:** `supplierCode`, `poDate`, `poNo`, `fc`  
**Detail Required:** `orderNo`, `itemCode`, `vendorQty`, `vendorUnitCode`, `unitPrice`, `currency`

**Header**
| Field | Type | Notes |
|--------------|--------------------|-------|
| supplierCode | string(50) | |
| poDate | string (DateTime) | `yyyy-MM-dd HH:mm:ss.SSS` |
| poNo | string(50) | Primary (Header) |
| etdActual | string\|null | DateTime |
| eta | string\|null | DateTime |
| fc | enum(PO, FOC) | |

**Detail**
| Field | Type | Notes |
|-------------------|----------------------|-------|
| additionalRemarks | string\|null | |
| orderNo | string(25) | |
| itemCode | string(50) | |
| vendorQty | number | `multipleOf: 0.01` (decimal(18,2)) |
| vendorUnitCode | string(10) | |
| unitPrice | number | `multipleOf: 0.00001` (decimal(20,5)) |
| currency | string(10) | |

---

### 3.4 Material Transaction (Header + Detail)

**Header Required:** `transactionNo`, `transactionDate`, `poNo`, `noAju`  
**Detail Required:** `fc`, `itemCode`, `vendorQty`, `vendorUnitCode`, `internalQty`, `internalUnitCode`, `unitPrice`, `batchNo`

**Header**
| Field | Type | Notes |
|-----------------|--------------------|-------|
| transactionNo | string(50) | Primary |
| transactionDate | string (DateTime) | `yyyy-MM-dd HH:mm:ss.SSS` |
| poNo | string(50) | |
| noAju | string(26) | PPKEK |

**Detail**
| Field | Type | Notes |
|------------------|-------------------|-------|
| fc | enum(PO, FOC) | |
| itemCode | string(50) | |
| vendorQty | number | `multipleOf: 0.01` |
| vendorUnitCode | string(10) | |
| internalQty | number | By Base UOM |
| internalUnitCode | string(10) | |
| unitPrice | number | |
| batchNo | integer | |

---

## 4) Warehouse

### 4.1 Goods Received to Finished Goods (FG) Warehouse

**Required:** `transactionNo`, `transactionDate`, `poNo`, `qty`

| Field           | Type              | Notes                     |
| --------------- | ----------------- | ------------------------- |
| transactionNo   | string(50)        | Primary                   |
| transactionDate | string (DateTime) | `yyyy-MM-dd HH:mm:ss.SSS` |
| poNo            | string(25)        | PRO No                    |
| qty             | integer           | Units of FG               |

---

### 4.2 Goods Received (CMT) — Header + Detail

**Header Required:** `transactionNo`, `transactionDate`, `noAju`  
**Detail Required:** `itemCode`, `qty`, `unitCode`, `unitPrice`, `batchNo`

**Header**
| Field | Type | Notes |
|-----------------|--------------------|-------|
| transactionNo | string(50) | Primary |
| transactionDate | string (DateTime) | `yyyy-MM-dd HH:mm:ss.SSS` |
| noAju | string(26) | PPKEK |

**Detail**
| Field | Type | Notes |
|-----------|---------------------|-------|
| itemCode | string(50) | |
| qty | number | `multipleOf: 0.01` (decimal(18,2)) |
| unitCode | string(10) | |
| unitPrice | number | |
| batchNo | integer | |

---

## 5) Logistics

### 5.1 Dispatch Material (to Production) — Header + Detail

**Header Required:** `transactionNo`, `transactionDate`, `dispatchTo`  
**Detail Required:** `productionNo`, `itemCode`, `qty`, `unitCode`, `batchNo`

**Header**
| Field | Type | Notes |
|-----------------|--------------------|-------|
| transactionNo | string(50) | Primary |
| transactionDate | string (DateTime) | `yyyy-MM-dd HH:mm:ss.SSS` |
| dispatchTo | string(50) | Default: "Production" |

**Detail**
| Field | Type | Notes |
|--------------|---------------------|-------|
| productionNo | string(25) | PRO No |
| itemCode | string(50) | |
| qty | number | `multipleOf: 0.0001` (decimal(18,4)) |
| unitCode | string(10) | |
| batchNo | integer | Ref GR batch |

---

### 5.2 Dispatch (General)

**Required:** `transactionNo`, `transactionDate`, `dispatchTo`, `itemCode`, `qty`, `unitCode`, `batchNo`

| Field           | Type              | Notes                                |
| --------------- | ----------------- | ------------------------------------ |
| transactionNo   | string(50)        | Primary                              |
| transactionDate | string (DateTime) | `yyyy-MM-dd HH:mm:ss.SSS`            |
| dispatchTo      | string(50)        | "Production" or "External"           |
| itemCode        | string(50)        |                                      |
| qty             | number            | `multipleOf: 0.0001` (decimal(18,4)) |
| unitCode        | string(10)        |                                      |
| batchNo         | integer           |                                      |
| deliveryNoteNo  | string(50)\|null  | Optional                             |
| noAju           | string(26)\|null  | Optional (PPKEK)                     |
| deliverTo       | string(200)\|null | Optional                             |
| vehicleId       | string(50)\|null  | Optional                             |
| nw              | number\|null      | Net weight (KGM)                     |
| gw              | number\|null      | Gross weight (KGM)                   |

---

### 5.3 Delivery Note

**Required:** `deliveryNoteNo`, `transactionDate`, `noAju`, `deliverTo`, `vehicleId`, `poNo`, `qty`

| Field           | Type              | Notes                     |
| --------------- | ----------------- | ------------------------- |
| deliveryNoteNo  | string(50)        | Primary                   |
| transactionDate | string (DateTime) | `yyyy-MM-dd HH:mm:ss.SSS` |
| noAju           | string(26)        | PPKEK                     |
| deliverTo       | string(200)       | Recipient                 |
| vehicleId       | string(50)        | Plate/Vehicle             |
| poNo            | string(25)        | PRO No                    |
| qty             | number            | In PRO UnitCode           |
| nw              | number\|null      | Optional (KGM)            |
| gw              | number\|null      | Optional (KGM)            |

---

## 6) Sales

### 6.1 Sales Invoice

**Required:** `invoiceNo`, `invoiceDate`, `vesselDate`

| Field          | Type                      | Notes                     |
| -------------- | ------------------------- | ------------------------- |
| invoiceNo      | string(50)                | Primary                   |
| invoiceDate    | string (DateTime)         | `yyyy-MM-dd HH:mm:ss.SSS` |
| vesselDate     | string (DateTime)         | `yyyy-MM-dd HH:mm:ss.SSS` |
| customerCode   | string(10)\|null          | Optional                  |
| deliveryNoteNo | string(200)\|null         | Optional                  |
| category       | enum(Export, Local)\|null | Optional                  |
| poNo           | string(25)\|null          | Optional                  |
| qty            | number\|null              | Optional                  |
| price          | number\|null              | Optional (to bill buyer)  |
| priceFob       | number\|null              | Optional (declared FOB)   |
| currency       | string(10)\|null          | Optional                  |

---

## 7) Validation & Tooling

### 7.1 Node.js (AJV) Quickstart

```bash
npm i ajv ajv-formats
```

```js
const Ajv = require("ajv");
const addFormats = require("ajv-formats");
const schema = require("./your-schema.json"); // e.g., sales-invoice.schema.json

const ajv = new Ajv({ allErrors: true, strict: false });
addFormats(ajv);

const validate = ajv.compile(schema);
const valid = validate(yourPayload);
if (!valid) console.error(validate.errors);
```

### 7.2 .NET (Newtonsoft.JsonSchema or System.Text.Json)

- Define DTOs mirroring these contracts and use fluent validation / data annotations.
- For strict schema validation, consider `Newtonsoft.Json.Schema` (JSchema) or community libraries.

---

## 8) Versioning & Change Control

- **v1.0 (2025-09-19):** Initial consolidation of all data contracts from the working session.
- Suggest keeping each schema in its own `*.schema.json` and tracking changes via Git (Semantic Versioning).

---

## 9) Glossary

- **PRO**: Production Order
- **FOB**: Free On Board (order type)
- **CMT**: Cut, Make, Trim (order type)
- **FOC**: Free Of Charge
- **PPKEK**: Customs doc; `noAju` is the customs submission number
- **FG**: Finished Goods
- **UOM**: Unit of Measure

---

### Need the schemas as files?

Say the word and I’ll export **each** section above as `*.schema.json` plus sample payloads into a ZIP.
