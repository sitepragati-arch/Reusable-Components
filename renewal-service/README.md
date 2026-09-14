# renewal-service

A small, stateless Cloudflare Worker that answers one question for any number
of client apps: **"is this customer's renewal current?"** — by reading a
shared, publicly-viewable Google Sheet.

No database, no per-app configuration. Every client app is just a row in the
sheet, looked up by its own `customerId` at request time.

## Endpoints

### `GET /status?customerId=XXX`

```json
{
  "active": false,
  "renewal_date": "2026-09-13",
  "amount": 500,
  "name": "Chinese Food Truck Web Application",
  "customer_id": "ahJAFDCZT8Z51ms"
}
```

Fails open (`active: true`) if `customerId` has no row, or the sheet is
unreachable — a typo or a Google outage should never lock a real customer
out.

### `GET /qr?customerId=XXX`

Returns `image/svg+xml` — a UPI QR code for that row's exact `Amount`, payable
to the vendor's collection account. Usable directly in an `<img src>`.

## The sheet

One Google Sheet (**File → Share → Anyone with the link → Viewer**), one row
per client app, these exact column headers:

| Customer Id | Name | Next Renewal Date | Amount | Comments |
|---|---|---|---|---|
| any unique string | shown on the lock screen if the app doesn't override it | `DD/MM/YYYY` | number | optional |

The service reads it fresh on every request — no caching — so editing a cell
takes effect on the very next check.

## Setup

```bash
npm install
```

Copy `wrangler.toml`'s `[vars]` block and set `SHEET_ID` to your sheet's ID
(the long string in its URL between `/d/` and `/edit`).

Local dev — create `.dev.vars` (gitignored):

```
RENEWAL_UPI_ID=yourvpa@bank
RENEWAL_UPI_PAYEE_NAME=Your Business Name
```

```bash
npm run dev
```

## Deploy

```bash
npm run deploy
npx wrangler secret put RENEWAL_UPI_ID
npx wrangler secret put RENEWAL_UPI_PAYEE_NAME
```

**Naming note:** deploy this under a semantically boring `name` in
`wrangler.toml` — not anything containing *license*, *paywall*,
*subscription*, or *renewal*. Testing found ad-blocker / anti-paywall filter
lists silently drop requests (`ERR_BLOCKED_BY_CLIENT`) to subdomains shaped
like a licensing check, which — combined with the fail-open design above —
would let a blocked client bypass the check entirely without anyone
noticing. `cf-relay-svc` is the deliberately unremarkable name used for the
live instance.

## Onboarding a new client app

1. Add a row to the sheet with a fresh `customerId`.
2. Point that app's [`renewal-gate`](../renewal-gate) component at this
   service's URL and that `customerId`.

Nothing to deploy or configure here per app.
