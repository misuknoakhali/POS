
    
  
# ShopDesk Retail POS

A self-hosted, single-store retail point of sale built with Node.js 24, SQLite, and a responsive browser interface. Business data stays in a local database. No runtime packages, cloud credentials, or external services are required.

## Run

```sh
cd /workspace/POS
node --version # Node 24 or newer
npm start
```

The server listens on loopback port 3000. On the server machine, open the app in a browser and create your owner account (password: at least 10 characters). There are no default credentials. Configure store name, currency, and tax in Settings before making sales, then create products and opening stock. Currency is locked after the first sale to protect historical reports.

Environment options:

| Variable            | Default                        | Purpose                                                               |
| ------------------- | ------------------------------ | --------------------------------------------------------------------- |
| `PORT`              | `3000`                         | HTTP listening port                                                   |
| `HOST`              | `127.0.0.1`                    | Bind address; use `0.0.0.0` only behind an appropriate firewall/proxy |
| `POS_DATA_DIR`      | `./data` relative to server.js | Persistent SQLite storage directory                                   |
| `POS_SECURE_COOKIE` | unset                          | Set to `1` when serving behind HTTPS                                  |

For shared access, deploy behind an HTTPS reverse proxy, set secure cookies, restrict access, and protect the initial account setup from other users. Never expose an uninitialized instance publicly. Live server processes must be restarted after environment restoration.

## Features

- First-run owner registration, password hashing with scrypt, expiring HTTP-only sessions, login throttling, same-origin write checks, and browser security headers.
- Searchable products with unique SKU/barcode, categories, sale/cost prices, stock counts, editing, and archival that preserves historical sales.
- Cashier cart with quantity controls, keyboard barcode entry (scanner sends Enter), customer selection, fixed-amount discount, tax, cash tender, and change calculation.
- Atomic SQLite checkout: server prices override browser input; insufficient stock, invalid quantities, and insufficient cash reject the whole sale.
- Cash, card, and mobile payment records. Card/mobile payments must be processed and verified on an external terminal or service before recording the sale.
- Printable receipts, searchable sales history, CSV export, and full-sale refunds with stock restoration and duplicate-refund protection.
- Customer contact directory, manual inventory adjustments with reasons, stock movement history, and low-stock alerts (five units or fewer).
- Daily sales, completed revenue, product gross profit, top sellers, and recent sales-day charts. Refunded sales are excluded. Day boundaries are UTC.
- Store preferences and authenticated business-data JSON export; responsive desktop/mobile layouts.

All monetary values are integer hundredths of the chosen currency; use currencies with two decimal places. Tax is one store-wide additive rate applied after discount. Confirm suitability for local requirements. A single owner has access to all operations.

## Tests

```sh
npm test
```

Tests launch an isolated server and temporary database. They exercise owner registration, authentication, stock adjustments, customer creation, authoritative totals, checkout rollback, reports, refunds, archival, exports, and logout. They never touch the store database. A Chromium smoke check was also performed during development for account setup, adding a product, checkout, receipt, refund, navigation, and mobile sizing.

## Backups and recovery

Stop the server before copying the entire `data/` directory (or configured `POS_DATA_DIR`) to secure backup storage. Restore the directory while the server is stopped, preserving permissions, then restart. Back up regularly and verify recovery on a separate instance. JSON exports contain business records but are not a restorable database backup; there is no JSON import endpoint. Exports include customer information and should be handled securely.

Data and passwords are not stored in Git. No sample products or credentials are inserted in the real store database.

## Scope and remaining integrations

This is a working single-store POS foundation, not a claim of complete commercial or regulatory readiness. It does not include payment-gateway charging or settlement, partial refunds, employee roles, multi-branch syncing, purchasing/supplier accounting, cash drawer reconciliation, offline synchronization, jurisdiction-specific fiscal invoices, or printer/cash-drawer drivers. Browser printing supports ordinary receipt output; hardware integration requires device-specific work. Verify accounting, tax rules, security deployment, and backups before live trading. Cashier checkout retries use an idempotency key to avoid duplicate sales. If a response is lost, retry the unchanged order or check sales history before clearing it and creating another order.

