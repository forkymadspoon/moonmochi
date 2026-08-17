# 🌙 MoonMochi Café — Fictional Demo Site

A single-file, static landing page for a **fictional** Singapore dessert café, used as a test site
for event tracking: standard events, a custom event, test/active modes and deliberate fault
scenarios.

> **Everything here is make-believe.** MoonMochi Café does not exist. All products, prices, carts,
> checkouts and "orders" are fictional. **No payment is processed** and no real business or customer
> data is used. The email typed into the Moon Club form never leaves the page and is never sent as
> an event parameter. The optional customer-data-matching demo sends **invented test values** on
> `example.com` — never put a real person's details there, since this page is public.

**Stack:** plain HTML + inline CSS + vanilla JavaScript. No backend, no database, no build step,
no dependencies. Deploys to GitHub Pages as-is.

---

## 1. Set the tracking ID

Open `index.html` and find this near the top of `<head>` (look for the comment
`>>> PASTE YOUR PIXEL ID HERE <<<`):

```js
window.MOONMOCHI_PIXEL_ID = '1073552061825203';
```

Change it to any numeric ID, or put back `'PASTE_YOUR_PIXEL_ID_HERE'` to switch the site into test
mode, where nothing is transmitted anywhere.

**Test mode vs active mode**

| | Test mode | Active mode |
|---|---|---|
| Trigger | placeholder, or any non-numeric value | ID matches `^[0-9]{8,25}$` |
| `fbevents.js` | never downloaded | loaded from `connect.facebook.net` |
| `fbq()` | never defined | defined, `init` + `PageView` sent |
| Events | logged to the browser console only | sent **and** logged to the console |

While the placeholder is in place the tracking library is completely inactive — no script, no
network request, nothing transmitted — so the whole event map can be rehearsed offline.

**Switching IDs without a redeploy.** Once the page is hosted, append `?pixel=` to the URL:

```
https://<username>.github.io/<repo>/?pixel=123456789012345   # use this ID, remembered in this browser
https://<username>.github.io/<repo>/?pixel=clear             # forget it, fall back to the ID in index.html
```

The override is stored in `localStorage` for that browser only, so several people can point the
same deployed page at their own ID. The badge in the **Fire an event** section always shows which
ID is active and where it came from.

---

## 2. Run it locally

Simplest: double-click `index.html`.

⚠️ Over `file://`, the helper extension and the tracking library will not work properly. For
active-mode testing, serve it over HTTP:

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000>. Any static server works (`npx serve`, VS Code Live Server, …).

---

## 3. Publish with GitHub Pages

```bash
git init && git add . && git commit -m "MoonMochi Café demo site"
```

Push to a GitHub repository, then: **Settings → Pages → Source: Deploy from a branch →
Branch: `main` / `(root)` → Save.**

Your site appears at `https://<username>.github.io/<repo>/` within a minute or two, served over
HTTPS — which is what the tracking library and the helper extension expect.

---

## 4. Event map

| Event | Type | Fires when | Parameters |
|---|---|---|---|
| `PageView` | standard | immediately after `fbq('init', …)` in `<head>` | — |
| `ViewContent` | standard | "Explore the menu" CTA · **ViewContent** trigger button | `content_name: 'Lunar Menu'`, `content_category: 'menu'` |
| `AddToCart` | standard | any "Add to orbit" button · **AddToCart** trigger button | `content_name`, `content_ids: [productId]`, `content_type: 'product'`, `value`, `currency: 'SGD'` |
| `Lead` | standard | Moon Club signup confirmed · **Lead** trigger button | `content_name: 'Moon Club signup'`, `method: 'hero_cta'` \| `'manual_test'` |
| `InitiateCheckout` | standard | "Begin checkout" with ≥ 1 item · **InitiateCheckout** trigger button | `contents: [productIds]`, `num_items`, `value`, `currency: 'SGD'` |
| `Purchase` | standard | after confirming the fictional checkout · **Purchase** trigger button | `content_ids: [productIds]`, `value`, `currency: 'SGD'` |
| `MochiFlavorSelected` | **custom** | a mochi product is added to the cart · **MochiFlavorSelected** trigger button | `flavor`, `source: 'menu_card'` \| `'manual_test'` |

Standard events are used wherever one fits; the custom event covers only the fictional
mochi-flavour interaction.

### Firing events manually

The **Fire an event** section (nav → *Event triggers*, or `#events`) has one CTA per event so you
can trigger them on demand while the helper extension, your events dashboard or DevTools is open.
`PageView` fires automatically on load. `InitiateCheckout` and `Purchase` use your current cart, or
a single Stardust Mochi Box as a sample basket when the cart is empty.

### Products (fictional)

| Product | ID | Price |
|---|---|---|
| Stardust Mochi Box | `MM-BOX-001` | S$18.80 |
| Lunar Matcha | `MM-DRINK-001` | S$8.50 |
| Comet Croffle | `MM-CROFFLE-001` | S$12.00 |

### Debugging API

```js
moonMochiPixelLab.state                    // mode, id, cart, totals, consent, matching data
moonMochiPixelLab.sendStandard('ViewContent', { content_name: 'Lunar Menu' })
moonMochiPixelLab.sendCustom('MochiFlavorSelected', { flavor: 'Yuzu Eclipse' })
moonMochiPixelLab.setConsent('granted')    // or 'declined'
```

---

## 5. Fault scenarios

Three controls sit under the event triggers. All are off / unanswered by default.

### 🍪 Mock consent banner

The page follows the documented call order in `<head>`:

```
base snippet  →  fbq('consent', 'revoke')  →  fbq('init', …)  →  fbq('track', 'PageView')
```

Until you press **Accept**, consent is revoked and events are **held** rather than sent — they show
as `HELD — consent not granted` in the console. Accepting calls `fbq('consent', 'grant')` and the
held events flush.

This reproduces the most common "tracking is definitely installed, but the dashboard sees nothing"
report: a consent tool that never grants. **Reset consent banner** puts it back so you can re-run
the sequence from a cold load.

### 🔐 Customer data matching (fictional data)

Off by default. When on, the page inits as
`fbq('init', '<id>', { em, ph, fn, ln, ct, country })` using invented values.

- It is applied at `init`, so toggling **reloads the page**.
- Pass **plain text**, not pre-hashed values — the library normalises and SHA-256 hashes them in the
  browser before they are sent. Hashing them yourself double-hashes and breaks matching.
- This is what makes the matching panel in the helper extension populate, and what moves the match
  quality score in your dashboard. With it off, expect both to look empty — correctly so.

### 🐞 Break-it scenarios

Four buttons that each reproduce one classic symptom, with a `console.error` explaining what to
look for:

| Button | What breaks | Where you'll see it |
|---|---|---|
| Misspelled event name | sends `Purchace` | lands as a *custom* event, never counts as a conversion |
| Purchase with no value / currency | required params missing | event arrives, revenue is zero; diagnostics warn |
| Wrong parameter types | `value` a string, `currency` a number | Network tab `cd[value]` / `cd[currency]`; unusable |
| Duplicate Purchase | same event twice, ~300 ms apart | double-counted revenue; the extension flags the repeat |

---

## 6. How to validate

**Browser helper extension**
1. Install the official helper extension for your tracking platform from the Chrome Web Store.
2. Open the deployed HTTPS page (not `file://`) and **disable any ad blocker for the site** — a
   blocked `fbevents.js` is the single most common false alarm.
3. Click the extension icon, then click through the CTAs. The side panel lists each event with a
   status of Active / Inactive / Warning / Error, plus content IDs, content types, event parameters,
   matching data and the script's URL location.
4. Warnings and errors come with recommended fixes — that panel is the fastest feedback loop.

**Events dashboard → test events**
1. Open your dataset → **Test events**.
2. Paste the site URL into the test-browser-events box and open it.
3. Click through the page; events appear live within a few seconds with full parameters.

**Chrome DevTools → Network**
1. Open DevTools → **Network**, then reload.
2. Filter for `tr/?` or `facebook`.
3. Each event is a request to `/tr/`. In the query string, `id` is the tracking ID, `ev` the event
   name, and `cd[...]` the custom parameters.
4. In test mode there should be **zero** such requests — that's the check that the placeholder guard
   works.

**Chrome DevTools → Console**
- Every event is mirrored as
  `[MoonMochi] <EventName> — standard|custom event — SENT | HELD | TEST MODE — NOT SENT`
  followed by its parameters and an ISO timestamp.
- In test mode a startup `console.warn` explains why tracking is inactive.
- Fire events by hand with the debugging API above.

---

## 7. Common troubleshooting checks

- **ID still contains the placeholder** — the site stays in test mode and sends nothing. The badge
  in the **Fire an event** section says so explicitly.
- **Page not served over HTTPS** — the helper extension and test-events tools misbehave on `file://`
  and plain HTTP. Use GitHub Pages or a local server.
- **Ad blocker active** — uBlock, AdGuard, Brave Shields and similar block `fbevents.js` outright.
  Requests fail in the Network tab and the extension reports nothing. Disable for the test domain.
- **Consent or privacy tooling blocking the script** — consent platforms, Safari ITP, Firefox ETP
  and enterprise policies can suppress it. Test in a clean Chrome profile. Reproduce it here by
  declining the mock consent banner: events are held, not sent.
- **Duplicate base code installed** — a second snippet (e.g. from a tag manager) double-counts
  events. The extension flags multiple IDs / a duplicate `PageView`. This page contains exactly one
  copy of the base code; do not paste another.
- **Event or parameter name misspelled** — event names are case-sensitive: `ViewContent`, not
  `Viewcontent`. A misspelled standard event silently becomes a custom event.
- **Testing on the wrong domain** — confirm the URL under test matches the deployed one
  (`localhost` vs GitHub Pages are different domains).
- **`Purchase` firing more than once** — in the checkout flow it is guarded: it fires only after
  explicit confirmation, once per order, and the cart empties immediately after. (The manual
  **Purchase** trigger button is deliberately unguarded, so repeated clicks *will* send repeated
  events — that's how you reproduce the duplicate-Purchase symptom on purpose.) In the wild,
  duplicates usually mean a duplicate snippet or a re-submitted confirmation page.

---

## 8. Accessibility and responsiveness

- No horizontal overflow at 320–1440 px; verified at 365, 375, 768 and desktop widths.
- Single-column layout below 900 px; nav collapses to one line below 560 px.
- Interactive controls are ≥ 44 px on touch widths.
- Animations respect `prefers-reduced-motion`.

---

## 9. Files

| File | Purpose |
|---|---|
| `index.html` | The entire site: markup, inline CSS, inline JS, tracking loader, event triggers, fault controls |
| `README.md` | This document |

---

🌙 *MoonMochi Café is fictional. All products, prices and checkout interactions are make-believe,
and no payment is ever processed.*
