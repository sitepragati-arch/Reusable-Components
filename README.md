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

## Adding a new project

1. Add a row to the shared spreadsheet: a fresh `customerId`, `Name`,
   `Next Renewal Date`, `Amount`.
2. Copy `renewal-gate/` into the new project and wrap its root — see
   [`renewal-gate/README.md`](./renewal-gate/README.md).

That's it — no changes to `renewal-service` itself.

## Adding a new reusable component to this repo

Give it its own top-level folder with its own `README.md`, and list it above.
