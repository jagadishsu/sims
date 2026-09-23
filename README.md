# Bachat Kosh (बचत कोष)

A community savings & investment management system for small savings
groups (dhukuti/cooperative-style groups) in Nepal. Members make monthly
deposits into a shared fund, and the fund's committee invests the pooled
money in NEPSE-listed shares — tracking units, purchase price, current
value, bonus shares, and rights shares over time.

Built as a self-contained web app: one HTML/JS front end talking to a
small PHP + MySQL backend, meant to run on a single PC via XAMPP.

## Features

- **Member accounts** — full profile per member (name, member number,
  phone, address, join date, status), with a private login PIN so
  members can check their own savings without admin help.
- **Deposits** — record each member's monthly deposit (amount, date,
  mode: cash/bank/mobile wallet, note); view, print, and reprint any
  individual receipt, or a full account statement per member.
- **Investments** — track the fund's share purchases on NEPSE: symbol,
  company, sector, purchase date, units, purchase price, and current
  price, with running gain/loss.
  - **Bonus shares** — record free bonus units issued on a holding.
  - **Rights shares** — record paid rights units, added to cost basis.
  - **Sell / close a holding** and see realised gain or loss.
- **Reports** — fund-wide summary (total deposits, invested amount,
  current portfolio value, gain/loss), printable.
- **Two login roles**
  - **Admin** — full access: manage members, record deposits and
    investments, view all reports, change settings/password, backup
    and restore the whole database.
  - **Member** — read-only view of their own deposit history and
    running balance, and can change their own login PIN.
- **Printing** — deposit receipts, member statements, portfolio
  summary, and fund summary all have dedicated print layouts.
- **Backup & restore** — download the entire database as a single
  JSON file at any time, and restore from it later (including
  recovering a wiped or brand-new install — see below).
- Currency and numbering formatted for Nepal (NPR, lakh/crore
  grouping, Nepali-style amount-in-words on receipts).

## How it's built

- **Front end**: a single `index.html` — vanilla JavaScript (no
  build step, no frameworks), rendering the whole app from a small
  in-memory state object and talking to the backend over `fetch()`.
- **Back end**: plain PHP (`api/*.php`) using PDO, one file per
  endpoint, returning JSON. Sessions handle login state; passwords
  and member PINs are stored as hashes, never in plain text.
- **Database**: MySQL/MariaDB (`api/schema.sql`) — six tables:
  `settings`, `members`, `deposits`, `investments`,
  `investment_bonus`, `investment_rights`.

```
bachat-kosh/
├── index.html          # the entire front end
├── SETUP.md            # step-by-step XAMPP installation guide
├── e2e-test.js         # automated end-to-end test (Playwright)
└── api/
    ├── schema.sql            # database schema (import this once)
    ├── db.php                # DB connection + shared helpers
    ├── bootstrap.php         # loads app state (members, deposits, investments…)
    ├── setup.php              # first-run setup (create org + admin password)
    ├── login.php / logout.php
    ├── members.php            # member CRUD
    ├── reset_pin.php          # admin resets a member's PIN
    ├── member_pin.php         # member changes their own PIN
    ├── deposits.php           # deposit CRUD
    ├── investments.php        # investment CRUD
    ├── investment_action.php  # bonus / rights / price update / sell
    ├── settings.php           # org name, toggles
    ├── change_password.php    # admin password change
    ├── backup.php             # download full backup as JSON
    └── restore.php            # restore from a backup JSON file
```

## Getting started

This app needs a web server with PHP and a MySQL/MariaDB database — it
will **not** work if you just double-click `index.html`. See
**[SETUP.md](SETUP.md)** for the full step-by-step guide to installing
it under XAMPP. In short:

1. Copy this folder into XAMPP's `htdocs`.
2. Import `api/schema.sql` in phpMyAdmin (creates the `bachat_kosh` database).
3. Start Apache and MySQL in the XAMPP control panel.
4. Visit `http://localhost/bachat-kosh/` and complete the one-time setup
   screen (savings group name + admin password).

## Using the app

- **First run**: the setup screen asks for your group's name and an
  admin password, and offers to load sample data so you can see how
  everything looks before entering real records.
- **Admin**: add members, record their monthly deposits, log share
  purchases and any bonus/rights issues, update current share prices,
  and print receipts/statements/reports as needed.
- **Members**: given a Member No. and PIN (set or reset by the admin),
  they can sign in on the same page to see their own deposit history
  and balance, and change their PIN.
- **Backup regularly**: Settings → "Download backup" saves a dated
  `.json` file with every record in the database. Keep copies
  somewhere safe (USB drive, cloud folder) — it's the only way to
  recover data or move it to a new PC.
- **Restore**: Settings → "Restore from backup" replaces all current
  data with a backup file's contents. If the database was ever wiped
  or this is a fresh install, the *setup screen* also offers a
  "Restore backup" option, which restores the data and signs you in
  as admin in one step.

## Security notes

- Admin passwords and member PINs are stored as hashes (never
  plaintext) using PHP's `password_hash()`.
- This is meant for **single-PC, local use** by one savings group —
  it does not implement the kind of hardening (rate limiting, HTTPS,
  audit logs, multi-tenant isolation) needed for a public-internet
  deployment. Keep it on your local network / XAMPP install.
- Anyone with admin access can download a full backup, which includes
  password/PIN hashes — treat backup files as sensitive and don't
  share them.

## Testing

`e2e-test.js` is a Playwright script that drives a full user journey
against a live copy of the app — setup, members, deposits,
investments (including bonus/rights/sale), reports, settings, PIN
reset, backup download, logout/login, and a full "erase all data →
restore from backup" recovery cycle. Run it against a running PHP
server (`php -S 127.0.0.1:8899` from this folder) with:

```
npm install playwright
node e2e-test.js
```
