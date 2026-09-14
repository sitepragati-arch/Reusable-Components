# renewal-gate

A portable renewal/paywall gate for any React app. Wrap your app root with
it, give it a `customerId` and the URL of a [`renewal-service`](../renewal-service)
deployment, and it either renders your app untouched or swaps in a
full-screen "Renewal Required" lock screen with a UPI QR code for the amount
owed.

No dependencies beyond React 18+ and `fetch`. Styling is plain inline
`style` objects — it doesn't require Tailwind, CSS Modules, or any
particular build setup, so it drops into any React app unmodified.

## Install

There's no package published yet — copy the three files in this folder
(`RenewalGate.tsx`, `RenewalRequired.tsx`, `types.ts`) into your app's
`src/renewal-gate/`.

## Use

Wrap your app root, above your router if you have one, so every route is
covered:

```tsx
import { RenewalGate } from "./renewal-gate/RenewalGate";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <RenewalGate
    apiBase="https://cf-relay-svc.swapniluser100.workers.dev"
    customerId="YOUR_APP_ID_HERE"   // this app's row key in the shared sheet
    appName="Your App Name"          // optional — falls back to the sheet's Name column
    logoUrl="/logo.png"              // optional
    supportEmail="sitepragati@gmail.com"
  >
    <App />
  </RenewalGate>
);
```

## Props

| Prop | Type | Required | Description |
|---|---|---|---|
| `apiBase` | `string` | yes | Base URL of a deployed `renewal-service` instance |
| `customerId` | `string` | yes | This app's row key in the shared spreadsheet |
| `supportEmail` | `string` | yes | Where staff send a payment screenshot for manual verification |
| `appName` | `string` | no | Shown on the lock screen; falls back to the sheet's `Name` column, then to `customerId` |
| `logoUrl` | `string` | no | Shown above the lock screen heading, if set |

## Behavior

- **Fails open.** While the check is loading, and if the service errors or
  is unreachable, `children` render normally. Only a confirmed
  `active: false` response locks the app.
- **No webhook.** The lock screen tells the person to email a payment
  screenshot to `supportEmail`. Whoever runs that inbox verifies it and
  edits `Next Renewal Date` in the spreadsheet directly — the customer taps
  **"I've Paid — Recheck Access"**, or just reloads.

## Onboarding a new app

1. Add a row to the shared spreadsheet with a fresh, unique `customerId`, a
   `Name`, and a `Next Renewal Date`.
2. Wrap that app's root as shown above, using that `customerId`.

No changes needed to `renewal-service` itself — see its own README if you'd
rather run your own copy instead of sharing an existing deployment.
