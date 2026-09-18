# Google Apps Script Web App Integration (Mecatos el Santi)

This document specifies the Google Apps Script (GAS) deployment required to connect the backend API of **Mecatos el Santi** directly with Google Sheets for real-time inventory and operational logging.

---

## 1. Sheet Architecture Setup (Tabs Required)

Create a Google Sheet with the following **3 tabs** matching the exact names:

### Tab 1: `INVENTORY` (Upsert per SKU, one column per branch)

* **Headers (Row 1):** `SKU`, `Product_Name`, `Category`, `Total_Stock`, `Last_Updated`

Branches in this system are fully dynamic — an admin can create, rename, or
add sedes at any time, so there is no fixed set of branch columns. Instead,
`Code.gs` **inserts a new `Stock_<Branch Name>` column automatically** the
first time it sees a branch it hasn't seen before, always placed between
`Category` and `Total_Stock`. So after the first sync involving, say,
"Sede Centro" and "Sede Norte", the sheet ends up with headers like:

```
SKU | Product_Name | Category | Stock_Sede Centro | Stock_Sede Norte | Total_Stock | Last_Updated
```

If a branch is renamed later, a new column is added for the new name (the
old column is left as historical data, not merged) — rename branches
sparingly if you want a clean sheet.

### Tab 2: `OPERATIONAL_LOGS` (Append-Only Transactions)
* **Headers (Row 1):** `Timestamp`, `Branch`, `Movement_Type`, `Category`, `Description_ID`, `Amount`, `Payment_Method`, `Payment_Status`, `Cashier_User`

`Payment_Status` is `COMPLETED` or `PENDING_PAYMENT` — only meaningful for
`SALE` rows with `Payment_Method = DELIVERY_APP` (Rappi/DiDi settle to the
bank days later, see root `CLAUDE.md` point 34); every other row (cash/
Nequi/card sales, purchases, expenses) always logs `COMPLETED`. This tab is
append-only, so confirming a payment later (`PATCH /api/admin/sales/:id/
confirm-payment`) does **not** retroactively update this row — the sheet
only reflects the status at the moment the sale was created.

### Tab 3: `CASH_CLOSURES` (Append-Only Daily Closures)
* **Headers (Row 1):** `Date`, `Branch`, `Total_Sales`, `Cash`, `Nequi`, `Card`, `Delivery_Apps`, `Petty_Cash_Expenses`, `Discrepancy`

---

## 2. Google Apps Script Code (`Code.gs`)

Paste the following Google Apps Script in `Extensions > Apps Script` inside your Google Sheet, then deploy it as a Web App (`Deploy > New deployment > Web app`, execute as yourself, access to "Anyone with the link" — or more restricted, if your Google Workspace supports it). Copy the resulting `/exec` URL into `GOOGLE_SHEETS_WEBHOOK_URL` (see section 3).

```javascript
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const ss = SpreadsheetApp.getActiveSpreadsheet();

    switch (data.action) {
      case 'UPDATE_INVENTORY':
        return handleInventoryUpdate(ss, data.payload);
      case 'LOG_TRANSACTION':
        return handleOperationalLog(ss, data.payload);
      case 'LOG_CASH_CLOSURE':
        return handleCashClosure(ss, data.payload);
      default:
        return responseJSON({ status: 'ERROR', message: 'Invalid action provided' });
    }
  } catch (err) {
    return responseJSON({ status: 'ERROR', message: err.toString() });
  }
}

// 1. UPDATE INVENTORY (upsert by SKU, one dynamic column per branch)
//
// payload shape: { sku, productName, category, totalStock,
//                  stockByBranch: [{ branchName, quantity }, ...] }
//
// Branches are NOT a fixed set of 3 — the backend sends whatever branches
// currently exist. This function creates a "Stock_<branchName>" column the
// first time it sees a branch name, inserting it right before Total_Stock,
// so the sheet grows to match however many sedes the ERP actually has.
function handleInventoryUpdate(ss, payload) {
  const sheet = ss.getSheetByName('INVENTORY');
  ensureInventoryHeaders_(sheet);
  const stockByBranch = payload.stockByBranch || [];

  // Asegura que exista una columna "Stock_<branchName>" para cada sede del
  // payload, insertándola justo antes de Total_Stock si hace falta.
  stockByBranch.forEach(function (entry) {
    ensureBranchColumn_(sheet, entry.branchName);
  });

  // Se relee después de insertar columnas, porque los índices pudieron cambiar.
  const headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
  const totalStockCol = headers.indexOf('Total_Stock') + 1;
  const lastUpdatedCol = headers.indexOf('Last_Updated') + 1;

  const rowIndex = findOrCreateSkuRow_(sheet, payload.sku);

  sheet.getRange(rowIndex, 2).setValue(payload.productName);
  sheet.getRange(rowIndex, 3).setValue(payload.category);

  stockByBranch.forEach(function (entry) {
    const colIndex = headers.indexOf('Stock_' + entry.branchName) + 1;
    sheet.getRange(rowIndex, colIndex).setValue(entry.quantity || 0);
  });

  sheet.getRange(rowIndex, totalStockCol).setValue(payload.totalStock || 0);
  sheet.getRange(rowIndex, lastUpdatedCol).setValue(new Date());

  return responseJSON({ status: 'SUCCESS', message: 'Inventory updated successfully' });
}

// Writes the 5 fixed headers if the INVENTORY tab is completely empty
// (getLastColumn() === 0 on a brand-new/renamed tab) — without this,
// getRange(1, 1, 1, 0) below throws "range must contain at least 1
// column" the very first time this runs against a fresh sheet.
function ensureInventoryHeaders_(sheet) {
  if (sheet.getLastColumn() === 0) {
    sheet.getRange(1, 1, 1, 5).setValues([[
      'SKU', 'Product_Name', 'Category', 'Total_Stock', 'Last_Updated'
    ]]);
  }
}

// Inserts a "Stock_<branchName>" header column right before Total_Stock,
// unless one already exists for that exact branch name.
function ensureBranchColumn_(sheet, branchName) {
  const columnHeader = 'Stock_' + branchName;
  const headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
  if (headers.indexOf(columnHeader) !== -1) return;

  const totalStockCol = headers.indexOf('Total_Stock') + 1;
  sheet.insertColumnBefore(totalStockCol);
  sheet.getRange(1, totalStockCol).setValue(columnHeader);
}

// Finds the row for a SKU, or appends a new row with just the SKU filled in.
function findOrCreateSkuRow_(sheet, sku) {
  const data = sheet.getDataRange().getValues();
  for (let i = 1; i < data.length; i++) {
    if (data[i][0] === sku) return i + 1; // 1-based index
  }
  const newRow = sheet.getLastRow() + 1;
  sheet.getRange(newRow, 1).setValue(sku);
  return newRow;
}

// Writes a header row on a completely empty append-only tab (same
// getLastColumn()===0 check as ensureInventoryHeaders_ above), so a
// freshly created/renamed tab doesn't just start filling with unlabeled
// rows.
function ensureHeaders_(sheet, headers) {
  if (sheet.getLastColumn() === 0) {
    sheet.getRange(1, 1, 1, headers.length).setValues([headers]);
  }
}

// 2. LOG OPERATIONAL TRANSACTION (Sales, Purchases, Expenses)
function handleOperationalLog(ss, payload) {
  const sheet = ss.getSheetByName('OPERATIONAL_LOGS');
  ensureHeaders_(sheet, [
    'Timestamp', 'Branch', 'Movement_Type', 'Category',
    'Description_ID', 'Amount', 'Payment_Method', 'Payment_Status', 'Cashier_User'
  ]);
  sheet.appendRow([
    new Date(),
    payload.branch,
    payload.movementType, // 'SALE', 'PURCHASE', 'EXPENSE', 'STOCK_LOSS', 'PAYMENT_CONFIRMATION'
    payload.category,
    payload.descriptionOrId,
    payload.amount,
    payload.paymentMethod,
    payload.paymentStatus || 'COMPLETED', // solo ventas DELIVERY_APP mandan PENDING_PAYMENT
    payload.cashierUser
  ]);
  return responseJSON({ status: 'SUCCESS', message: 'Transaction logged successfully' });
}

// 3. LOG CASH CLOSURE (End of Shift / Daily Z Report)
function handleCashClosure(ss, payload) {
  const sheet = ss.getSheetByName('CASH_CLOSURES');
  ensureHeaders_(sheet, [
    'Date', 'Branch', 'Total_Sales', 'Cash', 'Nequi',
    'Card', 'Delivery_Apps', 'Petty_Cash_Expenses', 'Discrepancy'
  ]);
  sheet.appendRow([
    payload.date,
    payload.branch,
    payload.totalSales,
    payload.cash,
    payload.nequi,
    payload.card,
    payload.deliveryApps,
    payload.pettyCashExpenses,
    payload.discrepancy
  ]);
  return responseJSON({ status: 'SUCCESS', message: 'Cash closure logged successfully' });
}

function responseJSON(obj) {
  return ContentService.createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON);
}
```

---

## 3. Backend integration (this repo)

The backend pushes to the webhook via a dedicated BullMQ queue + a separate
worker process — the same pattern already used for DIAN emission (see root
`CLAUDE.md`), so a slow or briefly-down webhook never blocks an API
response, and failed deliveries retry with exponential backoff instead of
silently being dropped.

* **Config:** set `GOOGLE_SHEETS_WEBHOOK_URL` (the Apps Script `/exec` URL
  from step 2) in the environment of the **worker process only** — the API
  process never calls the webhook directly, it only enqueues jobs, so it
  doesn't need this var.
* **Queue:** `backend/src/queues/sheetsQueue.ts` (`sheets-sync`), enqueued via
  `enqueueSheetsSync(action, payload)`. Job data carries the *already
  resolved* payload (branch/user names, not ids) — the worker never touches
  Mongo, it only talks to Redis and the webhook.
* **Service:** `backend/src/services/googleSheetsService.ts` — the single
  place that actually calls `fetch()` on the webhook. If credentials or a
  different transport are ever needed, change it here only.
* **Sync helpers:** `backend/src/utils/sheetsSync.ts` exports
  `syncInventoryToSheets(productId)`, `logSaleToSheets(sale)`,
  `logPurchaseToSheets(purchase)`, `logExpenseToSheets(expense)`, and
  `logCashClosureToSheets(params)`. All of them are called **fire-and-forget**
  (`.catch(err => console.error(...))`, never `await`ed) from controllers,
  right after the real mutation has already been saved to Mongo — a Sheets
  sync failure never affects the actual sale/purchase/expense/closure.
* **Worker:** `backend/src/workers/sheetsWorker.ts`, run as its own process
  (`npm run worker:sheets`), mirroring `dian-worker` in `docker-compose.yml`
  (new `sheets-worker` service there).

### What triggers what

| Event | Helper called | From |
|---|---|---|
| Sale created (admin panel) | `logSaleToSheets` + `syncInventoryToSheets` per item (stock decrements) | `adminController.createSaleAdmin` |
| Sale cancelled (admin panel) | `syncInventoryToSheets` per item (stock restored) | `adminController.cancelSaleAdmin` |
| Sale created (cajero POS, incl. offline sync) | `logSaleToSheets` | `posController.createSale` / `syncOfflineSales` |
| Purchase created (cajero, informal) | `logPurchaseToSheets` | `purchaseController.createPurchase` |
| Purchase created (admin, restocking) | `logPurchaseToSheets` per branch + `syncInventoryToSheets` once | `purchaseController.createPurchaseAdmin` |
| Purchase quantity edited/deleted (admin) | `syncInventoryToSheets` (only if quantity actually changed) | `purchaseController.updatePurchaseAdmin` / `deletePurchaseAdmin` |
| Stock top-up (Inventario, no Purchase record) | `syncInventoryToSheets` | `adminController.addProductStock` |
| Expense registered (admin panel) | `logExpenseToSheets` | `adminController.createExpense` |
| Petty-cash expense (cajero) | `logExpenseToSheets` | `posController.registerPettyCashExpense` |
| Shift closed with a final **Z** report (not an interim **X**) | `logCashClosureToSheets` | `cashClosureController.closeShift` |

Cajero sales don't currently decrement `ProductStock` at all (a known,
documented asymmetry in this system — see root `CLAUDE.md`), so they don't
trigger an inventory sync; only the flows that actually touch
`ProductStock` do.

### Reliability

Delivery is retried (5 attempts, exponential backoff starting at 3s) via
BullMQ/Redis, same as DIAN. If the webhook is down long enough to exhaust
retries, that job ends in `failed` state and is **not** automatically
retried again later (there's no reconciliation job for this queue, unlike
DIAN's) — Mongo remains the source of truth regardless; Sheets is a
real-time mirror for reporting, not a system of record.
