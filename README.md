# Personal Finance Dashboard

A single-page personal finance dashboard: expense tracking from a Google Sheet, a monthly salary
flow summary, net worth tracking from INDmoney, and savings targets linked to individual schemes.

It is written as one self-contained `index.html` meant to run as a **Claude Artifact** (the page
uses the artifact runtime, `window.claude`, for storage and connector access). It does not run as a
plain static site, because storage and connector calls come from that runtime.

## Features

- **Summary** – pick a month and see where the salary goes (investments, rent, spends, remaining)
  as a flow chart. Salary, investments and rent are editable in the UI and saved per month
  (a month inherits the latest earlier month's amounts). Also shows spends over the last 6 months,
  this month vs. last month (same days), and net worth over the last 6 months.
- **Expenses** – transactions read from a Google Sheet and copied into the app's own database.
  Add, edit, hide or delete rows without touching the sheet. Select rows to **split** a
  transaction (only your share counts) or mark it **for someone else** (not counted as spend).
- **Net worth** – total, invested, returns and allocation from INDmoney, refreshed at most once a
  day, with a daily history.
- **Targets** – named goals with a date and amount, linked to individual schemes (not asset types);
  progress is the live value of the linked schemes.

## How data flows

| Data | Where it lives |
| --- | --- |
| Transactions | Google Sheet (read only), copied into the artifact database |
| Edits, hidden rows, split / "for someone else" flags, added expenses | Artifact database |
| Salary / investments / rent per month, targets and links | Artifact database |
| Net worth and holdings | Read from INDmoney, last snapshot cached in the artifact database |

The sheet is never written to. The repository contains no financial data, credentials or keys.

## Required capabilities

Publish the page as an artifact with these runtime capabilities:

- `db` and `user`
- `mcp` with connectors:
  - `Google Sheets` – tool `get_values`
  - `IndMoney` – tools `networth_snapshot`, `networth_holdings`

## Setup

1. Create a Google Sheet tab with a header row `Amount, Category, Date, Payment method, Merchant,
   Source Key`. Dates are `dd/mm/yy`; the Source Key must be a unique id per row.
2. In `index.html`, set the `SH` constant near the top of the script:
   `id` (spreadsheet id from its URL), `tab` (tab name) and `gid` (numeric tab id, used only for the
   "Open sheet" link).
3. Publish `index.html` as an artifact with the capabilities above, connect Google Sheets and
   INDmoney in your Claude connectors, and open the artifact.
4. Open the Summary tab and enter your salary, investments and rent.

## Spend rules

Spends exclude the `Investments` category (toggle on the Expenses tab) and, on the Summary, the
`Rent` category (rent comes from the monthly plan). Transactions marked "for someone else" never
count, and split transactions count `amount ÷ people`.
