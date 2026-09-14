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

There's no published package — pull the files from this repo instead of
copy-pasting them, so bug fixes (like the QR-centering fix below) reach every
app that uses the gate. Two ways to do that, in order of preference:

### Option A — git submodule (recommended)

This is how the reference integration (the Chinese Food Truck app) does it.
From your app's repo root:

```bash
git submodule add https://github.com/sitepragati-arch/Reusable-Components.git src/reusable-components
```

This clones the whole `Reusable-Components` repo into
`src/reusable-components/` as a **submodule** — a pinned reference to a
specific commit, not a copy. Your build tool (Vite, CRA, etc.) sees plain
`.tsx` files on disk and compiles them like any other source file; nothing
extra to configure.

If your build setup typechecks the whole `src` tree (e.g. a bare `tsc -b`
with `"include": ["src"]`), exclude the sibling
[`renewal-service/`](../renewal-service) folder — it's a separate Cloudflare
Worker project with its own dependencies (`hono`, `qrcode-generator`) that
your frontend doesn't have installed:

```json
{
  "include": ["src"],
  "exclude": ["src/reusable-components/renewal-service"]
}
```

Two things to know once it's in:

- **A fresh clone** of your app's repo needs one extra step before the
  submodule's files exist on disk: `git submodule update --init --recursive`.
- **Pulling in updates** (bug fixes, new features) from this repo is a
  two-step, explicit action — nothing updates silently:
  ```bash
  git submodule update --remote src/reusable-components
  git add src/reusable-components && git commit -m "Update renewal-gate submodule"
  ```

### Option B — copy the files

If your project can't use git submodules, copy the three files in this
folder (`RenewalGate.tsx`, `RenewalRequired.tsx`, `types.ts`) into your app's
`src/renewal-gate/`. You lose the "one source of truth" benefit — future
fixes here won't reach your copy unless you re-copy them by hand.

## Use

Wrap your app root, above your router if you have one, so every route is
covered. Adjust the import path to wherever you installed the files
(`./reusable-components/renewal-gate/RenewalGate` for the submodule path
above, `./renewal-gate/RenewalGate` if you copied the files directly):

```tsx
import { RenewalGate } from "./reusable-components/renewal-gate/RenewalGate";

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

### Passing in your app's own branding

If your app already tracks its own name/logo somewhere (a settings table, a
context provider), don't hardcode them as literals above — wrap `RenewalGate`
in a small bridge component that reads your app's state and passes it
through. This is exactly what the Chinese Food Truck app does
([`AppRenewalGate.tsx`](https://github.com/swapniluser100-byte/Chinese-Food-Truck-web-application/blob/main/frontend/src/AppRenewalGate.tsx)):

```tsx
import type { ReactNode } from "react";
import { RenewalGate } from "./reusable-components/renewal-gate/RenewalGate";
import { useBranding } from "./BrandingContext";

const RENEWAL_API_BASE = "https://cf-relay-svc.swapniluser100.workers.dev";
const CUSTOMER_ID = "YOUR_APP_ID_HERE";
const SUPPORT_EMAIL = "sitepragati@gmail.com";

export function AppRenewalGate({ children }: { children: ReactNode }) {
  const { settings } = useBranding();

  return (
    <RenewalGate
      apiBase={RENEWAL_API_BASE}
      customerId={CUSTOMER_ID}
      appName={settings.app_name}
      logoUrl={settings.logo_data_url ?? undefined}
      supportEmail={SUPPORT_EMAIL}
    >
      {children}
    </RenewalGate>
  );
}
```

Then wrap the app root with `<AppRenewalGate>` instead of `<RenewalGate>`
directly. Everything else about installing and updating the underlying
component (submodule commands, typecheck exclusion) is unchanged — this
bridge is just your own app-specific file, not part of this repo.

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

## Styling notes

Styling is plain inline `style` objects on purpose, so the component drops
into any React app unmodified — no Tailwind, CSS Modules, or particular
build setup required. One consequence: if your host app uses **Tailwind**,
its preflight reset sets `img { display: block }`, which would otherwise
break simple `text-align: center` centering (that only affects inline
content, not block elements). The QR image already accounts for this with
explicit `display: block; margin: 0 auto`, so it centers correctly whether
or not your app resets `<img>`. If you fork this component, keep that
pattern for any other image you add.

## Onboarding a new app

1. Add a row to the shared spreadsheet with a fresh, unique `customerId`, a
   `Name`, and a `Next Renewal Date`.
2. Install the component (git submodule, or copy — see **Install** above).
3. Wrap that app's root as shown above, using that `customerId`.

No changes needed to `renewal-service` itself — see its own README if you'd
rather run your own copy instead of sharing an existing deployment.
