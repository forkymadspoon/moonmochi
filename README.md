# 🌙 MoonMochi Café — Meta Pixel Training Environment

A single-file, static landing page for a **fictional** Singapore dessert café, built as a hands-on
sandbox for learning the Meta Pixel: standard events, a custom event, demo/live modes and
troubleshooting practice.

> **Everything here is make-believe.** MoonMochi Café does not exist. All products, prices, carts,
> checkouts and "orders" are fictional. **No payment is processed** and no real business or customer
> data is used. The email typed into the Moon Club form never leaves the page and is never sent as
> an event parameter. The optional Advanced Matching demo sends **invented test values** on
> `example.com` — never put a real person's details there, since this page is public.

**Stack:** plain HTML + inline CSS + vanilla JavaScript. No backend, no database, no build step,
no dependencies. Deploys to GitHub Pages as-is.

---

## 1. Replace the Pixel placeholder

Open `index.html` and find this near the top of `<head>` (look for the comment
`>>> PASTE YOUR PIXEL ID HERE <<<`):

```js
window.MOONMOCHI_PIXEL_ID = 'PASTE_YOUR_PIXEL_ID_HERE';
```

Replace the placeholder with your numeric Meta Pixel / Dataset ID:

```js
window.MOONMOCHI_PIXEL_ID = '123456789012345';
```

**Demo mode vs live mode**

| | Demo mode (default) | Live mode |
|---|---|---|
| Trigger | placeholder unchanged, or a non-numeric value | ID matches `^[0-9]{8,25}$` |
| `fbevents.js` | never downloaded | loaded from `connect.facebook.net` |
| `fbq()` | never defined | defined, `init` + `PageView` sent |
| Events | logged to the DevTools console only | sent to Meta **and** logged to the DevTools console |

The Pixel stays completely inactive while the placeholder is present — no script, no network
request, nothing transmitted — so you can rehearse the whole event map offline.

**Switching Pixels without a redeploy.** Once the page is hosted, append `?pixel=` to the URL:

```
https://<username>.github.io/<repo>/?pixel=123456789012345   # use this Pixel, remembered in this browser
https://<username>.github.io/<repo>/?pixel=clear             # forget it, fall back to the ID in index.html
```

The override is stored in `localStorage` for that browser only, so several people can point the
same deployed page at their own Pixel. The badge in the **Fire an event** section always shows
which ID is active and where it came from.

---

## 2. Run it locally

Simplest: double-click `index.html`.

⚠️ Over `file://`, Ads Data Advisor and the real Pixel will not work properly. For live-mode
testing serve it over HTTP:

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000>. Any static server works (`npx serve`, VS Code Live Server, …).

---

## 3. Publish with GitHub Pages

```bash
git init && git add . && git commit -m "MoonMochi Café Pixel training site"
```

Push to a GitHub repository, then: **Settings → Pages → Source: Deploy from a branch →
Branch: `main` / `(root)` → Save.**

Your site appears at `https://<username>.github.io/<repo>/` within a minute or two, served over
HTTPS — which is what the Pixel and Ads Data Advisor expect.

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
can trigger them on demand while Ads Data Advisor, Test Events or DevTools is open. `PageView` fires
automatically on load. `InitiateCheckout` and `Purchase` use your current cart, or a single
Stardust Mochi Box as a sample basket when the cart is empty.

### Products (fictional)

| Product | ID | Price |
|---|---|---|
| Stardust Mochi Box | `MM-BOX-001` | S$18.80 |
| Lunar Matcha | `MM-DRINK-001` | S$8.50 |
| Comet Croffle | `MM-CROFFLE-001` | S$12.00 |

### Debugging API

```js
moonMochiPixelLab.state                    // mode, pixel id, cart, totals, consent, advancedMatching
moonMochiPixelLab.sendStandard('ViewContent', { content_name: 'Lunar Menu' })
moonMochiPixelLab.sendCustom('MochiFlavorSelected', { flavor: 'Yuzu Eclipse' })
moonMochiPixelLab.setConsent('granted')    // or 'declined'
```

---

## 5. Troubleshooting practice

Three training controls sit under the event triggers. All are off / unanswered by default.

### 🍪 Mock consent banner

The page follows Meta's documented order in `<head>`:

```
base snippet  →  fbq('consent', 'revoke')  →  fbq('init', …)  →  fbq('track', 'PageView')
```

Until you press **Accept**, consent is revoked and the Pixel **holds** events instead of sending
them — events show as `HELD — consent not granted` in the console. Accepting calls
`fbq('consent', 'grant')` and the held events flush.

This reproduces the most common "the Pixel is definitely installed, but Events Manager sees
nothing" report: a consent tool that never grants. **Reset consent banner** puts it back so you can
re-run the sequence from a cold load.

### 🔐 Advanced Matching (fictional data)

Off by default. When on, the page inits as
`fbq('init', '<id>', { em, ph, fn, ln, ct, country })` using invented values.

- It is applied at `init`, so toggling **reloads the page**.
- Pass **plain text**, not pre-hashed values — `fbevents.js` normalises and SHA-256 hashes them in
  the browser before they are sent. Hashing them yourself double-hashes and breaks matching.
- This is what makes the *Advanced Matching* panel in Ads Data Advisor populate, and what moves
  *Event Match Quality* in Events Manager. With it off, expect both to look empty — correctly so.

### 🐞 Break-it scenarios

Four buttons that each reproduce one classic symptom, with a `console.error` explaining what to
look for:

| Button | What breaks | Where you'll see it |
|---|---|---|
| Misspelled event name | sends `Purchace` | lands as a *custom* event, never counts as a conversion |
| Purchase with no value / currency | required params missing | event arrives, revenue is zero; Diagnostics warns |
| Wrong parameter types | `value` a string, `currency` a number | Network tab `cd[value]` / `cd[currency]`; Meta can't use it |
| Duplicate Purchase | same event twice, ~300 ms apart | double-counted revenue; Data Advisor flags the repeat |

---

## 6. How to validate

**Meta Ads Data Advisor (Chrome extension — this replaced Meta Pixel Helper)**
1. Install "Meta Ads Data Advisor" from the Chrome Web Store. If you already had Pixel Helper it
   updates to this automatically. Optionally sign in with your Facebook profile to unlock the
   setup-automation and audit features.
2. Open the deployed HTTPS page (not `file://`) and **disable any ad blocker for the site** —
   blocked `fbevents.js` is the single most common false alarm.
3. Click the extension icon, then click through the CTAs. The side panel lists each event with a
   status of Active / Inactive / Warning / Error, plus content IDs, content types, event
   parameters, Advanced Matching data and the Pixel's URL location.
4. Warnings and errors come with Meta's own recommended fixes — that panel is the fastest
   feedback loop for hands-on troubleshooting.

**Meta Events Manager → Test Events**
1. Events Manager → your dataset → **Test events**.
2. Paste the site URL into "Test browser events" and open it.
3. Click through the page; events appear live within a few seconds with full parameters.

**Chrome DevTools → Network**
1. Open DevTools → **Network**, then reload.
2. Filter for `tr/?` or `facebook`.
3. Each event is a request to `facebook.com/tr/`. In the query string, `id` is the Pixel ID,
   `ev` the event name, and `cd[...]` the custom parameters.
4. In demo mode there should be **zero** such requests — that's the check that the placeholder
   guard works.

**Chrome DevTools → Console**
- Every event is mirrored as
  `[MoonMochi] <EventName> — standard|custom event — SENT TO META | DEMO MODE — NOT SENT`
  followed by its parameters and an ISO timestamp.
- In demo mode a startup `console.warn` explains why the Pixel is inactive.
- Fire events by hand with the debugging API above.

---

## 7. Common troubleshooting checks

- **Pixel ID still contains the placeholder** — the site stays in demo mode and sends nothing.
  The badge in the **Fire an event** section says so explicitly.
- **Page not served over HTTPS** — Ads Data Advisor and Test Events misbehave on `file://` and
  plain HTTP. Use GitHub Pages or a local server.
- **Ad blocker active** — uBlock, AdGuard, Brave Shields and similar block `fbevents.js` outright.
  Requests fail in the Network tab and Ads Data Advisor reports nothing. Disable for the test domain.
- **Consent or privacy tooling blocking the script** — CMPs, Safari ITP, Firefox ETP and
  enterprise policies can suppress the Pixel. Test in a clean Chrome profile. Reproduce it here by
  declining the mock consent banner: events are held, not sent.
- **Duplicate Pixel code installed** — a second snippet (e.g. from a tag manager) double-counts
  events. Ads Data Advisor flags multiple Pixel IDs / a duplicate `PageView`.
- **Event or parameter name misspelled** — event names are case-sensitive: `ViewContent`, not
  `Viewcontent`. A misspelled standard event silently becomes a custom event.
- **Testing on the wrong domain** — confirm the URL in Test Events matches the deployed one
  (`localhost` vs GitHub Pages are different domains).
- **`Purchase` firing more than once** — in the checkout flow it is guarded: it fires only after
  explicit confirmation, once per order, and the cart empties immediately after. (The manual
  **Purchase** trigger button is deliberately unguarded, so repeated clicks *will* send repeated
  events — that's how you reproduce the duplicate-Purchase symptom on purpose.) In the wild,
  duplicates usually mean a duplicate snippet or a re-submitted confirmation page.

---

## 8. Files

| File | Purpose |
|---|---|
| `index.html` | The entire site: markup, inline CSS, inline JS, Pixel loader, event triggers, training controls |
| `README.md` | This document |

---

🌙 *MoonMochi Café is fictional. All products, prices and checkout interactions are make-believe,
and no payment is ever processed.*
