---
trigger: always_on
description: Point of Sale system for **Jalisco Tienda Mexicana**, a Mexican grocery store and restaurant. The app runs as both an Electron desktop app (production) and a browser preview (GitHub Pages demo). It handles grocery/restaurant transactions with barcode scanning, item grid selection, customer lookup, payment processing, held transactions, and receipt printing.
---

# Jalisco Tienda Mexicana POS

## Project Overview

Point of Sale system for **Jalisco Tienda Mexicana**, a Mexican grocery store and restaurant. The app runs as both an Electron desktop app (production) and a browser preview (GitHub Pages demo). It handles grocery/restaurant transactions with barcode scanning, item grid selection, customer lookup, payment processing, held transactions, and receipt printing.

**Live preview:** https://graphicaljerry.github.io/Jaliscotiendamexicana/
**Figma designs:** https://www.figma.com/design/rRcPCDbbV4Vv37GRzVdUu8/Jalisco-Tienda-Mexicana

### Figma Key Frames
- **Main POS screen:** node-id=6637-3814
- **Sidebar open:** node-id=6644-4602
- **Sidebar panel detail:** node-id=6644-4954
- **Payment modal:** built via use_figma on POS System page

## Tech Stack

- **Framework:** React 18 + Vite 6
- **Desktop:** Electron 33 (with electron-builder for packaging)
- **State:** Zustand 4 (single store: `transactionStore.js`)
- **Routing:** React Router v6 (BrowserRouter with `v7_startTransition` and `v7_relativeSplatPath` future flags — previously HashRouter, migrated to BrowserRouter)
- **Styling:** Plain CSS with CSS variables (NO Tailwind, NO CSS modules)
- **Database:** better-sqlite3 (Electron only; browser uses mock API)
- **Printer:** node-thermal-printer (Electron only)
- **Inventory:** 17,698 items loaded from `src/data/inventory.json` (TGS export, ~1.5MB)

## Architecture

### File Structure
```
src/
  App.jsx                    # BrowserRouter with /, /admin, /payment-preview routes
  main.jsx                   # Entry point, installs mock API for browser
  api-mock.js                # Browser mock: categories, items, customers, barcode lookup
  data/inventory.json        # 17,698 store items (barcode, name, price, tax, ebt)
  stores/transactionStore.js # Zustand store — ALL transaction state and computed values
  hooks/
    useBarcodeScanner.js     # USB barcode scanner detection (rapid keystroke pattern)
    useKeyboardShortcuts.js  # F1-F12 + Escape key mappings
  components/
    layout/
      TopBar.jsx/css         # Dark #121212 header bar (logo, txn#, Item Grid, Admin)
      TotalsBar.jsx/css      # Bottom totals (Paid, Change, Items, Sub Total, Tax, EBT, Grand Total)
      Logo.jsx               # Color SVG logo (jalisco-logo-color.svg)
    transaction/
      TransactionScreen.jsx/css  # Main POS screen — orchestrates everything
      TransactionTable.jsx/css   # Item list table with inline editing, search, barcode input
      TransactionControls.jsx/css # Sidebar controls (Transaction Type, Tax, Discount, Output)
      FunctionBar.jsx/css        # Active transaction function keys (F1-F12 row)
      IdleFunctionBar.jsx        # Idle state keys (Sale, Return, Reload Held, etc.)
    grid/
      ItemGrid.jsx/css       # Category grid → item sub-grid for restaurant menu
    customer/
      CustomerLookup.jsx/css # Customer search modal (name, phone, email)
    payment/
      PaymentModal.jsx/css   # Payment entry (cash/credit/debit/EBT, numpad, split payments)
      PaymentPreview.jsx     # Standalone page rendering PaymentModal with sample data (for Figma capture)
      HeldTransactionsModal.jsx/css # Reload or delete held transactions
    admin/
      AdminScreen.jsx/css    # Admin panel (settings, categories, items management)
```

### Routes
- `/` — Main POS transaction screen
- `/admin` — Admin panel (password protected)
- `/payment-preview` — Standalone payment modal with sample data (for Figma capture/dev)

### State Management (transactionStore.js)

Single Zustand store manages:
- `items[]` — current transaction line items
- `customer` — selected customer (or null)
- `transactionType` — sale | return | layaway | order | quote
- `taxType` — taxable | tax_exempt | alt_tax
- `discountMode` — none | by_line | all
- `outputType` — paper_tape | invoice
- `isActive` — whether a transaction is in progress
- `transactionNumber` — auto-incrementing (starts at 300001)
- `heldTransactions[]` — transactions put on hold
- Computed: `getSubtotal()`, `getTaxTotal()`, `getGrandTotal()`, `getEbtEligibleTotal()`, `getChange()`

### Two Function Bar States

1. **Idle** (`IdleFunctionBar`): Sale, Return, Lay-A-Way, Order, Quote, Alternate Tax Rate, Discount, Cash Check, Cash Drawer, Customer Inquiry, Open Drawer, Tax Type, Print Form, Payment, Pay Out, Reload Held Transaction, Item Inquiry, Exit
2. **Active** (`FunctionBar`): Repeat Last, Return Next, Quantity, Discount, Cancel, Coupon, Customer Inquiry, Write Memo, Put on Hold, Delete Last, Item Direct, Price, Sls. Change, Finish, Item Lookup, Item Inquiry, Tax Exempt Next, Open Drawer

### Keyboard Shortcuts
- **F1-F12** mapped to function bar actions (context-dependent on idle vs active)
- **Escape** cancels active transaction, returns to idle function bar
- **Enter** in code input submits barcode/quick code
- Payment modal: F1=Cash, F2=Debit, F3=Credit, F4=EBT

### Quick Codes (typed into item code input)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Graphicaljerry) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
