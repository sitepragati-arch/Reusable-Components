# Reusable Components

Standalone components and services meant to be shared across independent
projects, kept in one place so they have a single source of truth instead of
being copy-pasted and drifting.

## What's here

### [`renewal-gate/`](./renewal-gate)

A portable React component that gates an app behind a renewal/paywall
check — full-screen lock UI, UPI QR code, manual "I've paid" recheck.
Copy it into any React app.

### [`renewal-service/`](./renewal-service)

The stateless Cloudflare Worker `renewal-gate` talks to. Reads a shared
Google Sheet (one row per client app) and answers "is this customerId
current?" for any number of apps at once — nothing to deploy per app, just
add a row to the sheet.

Together, these let any number of independent apps — different repos,
different stacks, different teams — share one renewal-tracking spreadsheet
and one backend, each identified only by its own `customerId`.

```
renewal-service (Cloudflare Worker)
        │  reads
        ▼
  Google Sheet (one row per app)
        ▲
        │  queried by
renewal-gate (React component, one per app)
```

### [`renewal-gate-vanilla-js/`](./renewal-gate-vanilla-js)

The non-React counterpart to `renewal-gate/` — two files (a same-origin
proxy + a plain `<script>`), no build step, no npm dependency. Talks
directly to SitePragati's own public customer-info API
(`https://sitepragati.in/api/public/customer-info`), **not** the shared
Google Sheet / `renewal-service` above — a different backend entirely, not
interchangeable with the React component's data source. Reference
integration: the jewelry-shop project.

## Adding a new project

**React app:**
1. Add a row to the shared spreadsheet: a fresh `customerId`, `Name`,
   `Next Renewal Date`, `Amount`.
2. Add this repo as a git submodule in the new project and wrap its root
   with `RenewalGate` — see [`renewal-gate/README.md`](./renewal-gate/README.md)
   for the exact commands and a real example (the Chinese Food Truck app).

That's it — no changes to `renewal-service` itself.

**Anything else (static site, Cloudflare Pages, server-rendered):**
1. Ask whoever administers `sitepragati.in` to create a customer record —
   a fresh `customerId`, business name, next payment due date and amount.
2. Copy the two files from [`renewal-gate-vanilla-js/`](./renewal-gate-vanilla-js)
   into the new project — see that folder's `README.md` for the full
   walkthrough, a copy-paste template, and a testing checklist.

## Adding a new reusable component to this repo

Give it its own top-level folder with its own `README.md`, and list it above.
