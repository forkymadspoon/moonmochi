# 🌙 MoonMochi Café — Fictional Demo Site

A simple, single-file static site for a **fictional** Singapore dessert café. It exists to be a
clean, predictable page to install tracking on and fire events from.

> **Everything is make-believe.** MoonMochi Café does not exist. All products, prices, carts,
> checkouts and "orders" are fictional. **No payment is processed** and no real customer data is
> used. The email typed into the Moon Club form never leaves the page and is never sent as an
> event parameter.

Plain HTML + inline CSS + vanilla JS. No backend, no build step, no dependencies. One file:
`index.html` (~650 lines, mostly CSS).

---

## 1. The base code

It is pasted in `<head>`, between these two markers:

```html
<!-- Meta Pixel Code -->
...
<!-- End Meta Pixel Code -->
```

Pasted exactly as issued and unmodified. To use a different ID, change it on the `fbq('init', ...)`
line. To practise installing from scratch, delete everything between the markers and paste it back.

**Only one copy.** A second copy anywhere on the page double-counts every event.

If the base code is missing, the page still works: a one-line shim at the top of the main script
turns every `fbq()` call into a console warning instead of an error, so nothing breaks while you
are mid-install.

---

## 2. Where the events are

All of them are in one place — the `EVENTS` section of the script at the bottom of `index.html`.
One function per event, each a plain call:

```js
function eventViewContent() {
  var params = { content_name: 'Lunar Menu', content_category: 'menu' };
  fbq('track', 'ViewContent', params);
  log('ViewContent', params);
}
```

Standard events use `fbq('track', 'Name', params)`. The one custom event uses
`fbq('trackCustom', 'Name', params)`. To add an event, copy one of those functions and call it from
a click handler in the `WIRING` section below.

### Event map

| Event | Type | Fires when | Parameters |
|---|---|---|---|
| `PageView` | standard | page load, from the base code | — |
| `ViewContent` | standard | "Explore the menu" · test button | `content_name`, `content_category` |
| `AddToCart` | standard | any "Add to orbit" · test button | `content_name`, `content_ids`, `content_type`, `value`, `currency` |
| `Lead` | standard | Moon Club signup · test button | `content_name`, `method` |
| `InitiateCheckout` | standard | "Begin checkout" with ≥1 item · test button | `contents`, `num_items`, `value`, `currency` |
| `Purchase` | standard | confirming the fictional checkout · test button | `content_ids`, `value`, `currency` |
| `MochiFlavorSelected` | **custom** | a mochi product added to cart · test button | `flavor`, `source` |

`Purchase` in the checkout flow is guarded so it fires once per order. The **Purchase** test button
is not guarded — repeated clicks send repeated events, which is useful for reproducing
double-counting on purpose.

### Products (fictional)

| Product | ID | Price |
|---|---|---|
| Stardust Mochi Box | `MM-BOX-001` | S$18.80 |
| Lunar Matcha | `MM-DRINK-001` | S$8.50 |
| Comet Croffle | `MM-CROFFLE-001` | S$12.00 |

---

## 3. Test-event buttons

The **Fire an event** section has one button per event, so you can trigger each one on demand
without walking the whole shop flow. Every event is also logged to the console as
`[MoonMochi] <EventName> {params}`.

---

## 4. Run and publish

Locally — just double-click `index.html`. For anything involving the real network, serve over HTTP:

```bash
python3 -m http.server 8000
```

Publish with GitHub Pages: push the repo, then **Settings → Pages → Deploy from a branch →
`main` / `(root)`**. The site lands at `https://<username>.github.io/<repo>/` over HTTPS, which is
what the helper extension and test tools expect.

---

## 5. Troubleshooting

| Symptom | Check |
|---|---|
| Nothing fires at all | Is the base code actually in `<head>`? The console warns if `fbq()` is undefined. |
| Nothing arrives, but the page looks fine | **Ad blocker.** uBlock/AdGuard/Brave Shields block `fbevents.js` outright. Disable for the domain. |
| Works locally, not when shared | Serve over HTTPS, not `file://`. |
| Every event counted twice | A second copy of the base code somewhere on the page. |
| Event not recognised as standard | Spelling and case: `ViewContent`, not `Viewcontent`. A misspelled standard event silently becomes a custom one. |
| Revenue missing on `Purchase` | `value` must be a number and `currency` a 3-letter code like `SGD`. |
| Events on the wrong dataset | Check the ID on the `fbq('init', ...)` line. |

**Where to look:** the browser console for the `[MoonMochi]` lines, DevTools → Network filtered to
`tr/?` for the outgoing requests (`ev` is the event name, `cd[...]` the parameters), a pixel-helper
browser extension for per-event status, and your events dashboard's test-events view for what
actually arrived.

---

🌙 *MoonMochi Café is fictional. All products, prices and checkout interactions are make-believe,
and no payment is ever processed.*
