# QuickBooks Online (QBO) data exit plan (single-user)

This guide documents a practical, single-user migration path for leaving QuickBooks Online (QBO)
when you already have exported data from a previous integration. The goal is to preserve historical
records, verify completeness, and load your data into a replacement system without keeping QBO in
the loop.

## 1) Inventory what you already have

Confirm the exports you already downloaded. For a full exit, you should have at least:

- **Chart of Accounts**
- **Customers**
- **Vendors**
- **Items/Products**
- **Invoices**
- **Payments**
- **Bills**
- **Expenses**
- **Journal Entries / General Ledger**
- **Trial Balance**
- **Balance Sheet + Profit & Loss** (P&L)

Keep a small manifest (CSV or Markdown) listing:

- file name
- export date
- date range covered
- row count

## 2) Normalize your export bundle

Create a single directory and standardize naming. Example:

```
qbo-export-archive/
  00-manifest.csv
  accounts.csv
  customers.csv
  vendors.csv
  items.csv
  invoices.csv
  payments.csv
  bills.csv
  expenses.csv
  journal_entries.csv
  general_ledger.csv
  trial_balance.csv
  balance_sheet.pdf
  profit_and_loss.pdf
```

## 3) Choose your cutover strategy

For a single user, the most reliable option is a **clean cutover**:

- Import **all historical data** for reporting.
- Lock historical records to **read-only**.
- Start **new activity** in the replacement system from the cutover date.

This preserves history without forcing every legacy edge case into your new workflows.

## 4) Map QBO fields to your new schema

Even if the new app is simple, make sure you can reconstruct ledger math.
Here is a minimal mapping table for accounting accuracy:

| QBO export | Target table | Key fields to map |
| --- | --- | --- |
| Chart of Accounts | accounts | account_id, name, type, subtype, active |
| Customers | customers | customer_id, display_name, email, phone |
| Vendors | vendors | vendor_id, display_name, email, phone |
| Items | items | item_id, name, type, income_account_id, expense_account_id |
| Invoices | invoices | invoice_id, customer_id, total, status, issue_date, due_date |
| Payments | payments | payment_id, customer_id, amount, payment_date, method |
| Bills | bills | bill_id, vendor_id, total, status, issue_date, due_date |
| Expenses | expenses | expense_id, vendor_id, total, expense_date, account_id |
| Journal Entries | journal_entries | entry_id, txn_date, memo |
| General Ledger | ledger_lines | entry_id, account_id, debit, credit, txn_date |

## 5) Verify completeness before import

Run these checks on the raw export set:

- **Row counts**: make sure CSV row counts match the export UI summary.
- **Trial balance**: sum debits/credits from the general ledger; they should balance.
- **P&L cross-check**: compare income/expense totals in your ledger calculations to the P&L report.
- **Balance sheet cross-check**: ensure assets = liabilities + equity from your derived totals.

## 6) Import sequencing (recommended)

Load data in this order to minimize foreign key issues:

1. Chart of Accounts
2. Customers + Vendors
3. Items
4. Invoices + Bills
5. Payments + Expenses
6. Journal Entries + Ledger Lines

## 7) Post-import validation

- Re-run Trial Balance, P&L, and Balance Sheet **inside your new system**.
- Compare against QBO exports for the same period.
- If the totals align within rounding tolerance, freeze the historical period.

## 8) Single-user operations checklist

Since you have one user only:

- Use a **single admin account** with MFA.
- Enforce **daily backups** of your database.
- Keep the original export bundle immutable (read-only) for audit safety.

## 9) Optional: store original PDFs

Keep the QBO-generated PDFs for legal/audit reference. Store them alongside your database backups.

---

If you want, we can add a small import utility and validation scripts tailored to your export format
(e.g., CSV schema from your existing QBO integration).
