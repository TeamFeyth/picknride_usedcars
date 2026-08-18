# Pick & Ride Auto Sales — Used Cars Landing Page

Google Ads landing page for the used-vehicle campaign.
Destination subdomain: **call.picknrideauto.com**

Built and maintained by Feyth Marketing. Sister page: `picknride-fleet-lp`
(Commercial Fleet, `fleet.picknrideauto.com`).

---

## What's here

```
index.html        the page (self-contained; no build step)
assets/img/       photography, logo, icons — responsive WebP + JPEG fallback
assets/fonts/     self-hosted Oswald / Nunito subsets
_headers          Cloudflare Pages caching + security headers
```

---

## What this started as

The source export was a design-tool artifact, not a working page. Converting it
meant more than dropping images in:

| Was | Now |
|---|---|
| `<x-dc>` / `<helmet>` custom elements | unwrapped to real HTML |
| `{{ grid }}`, `{{ faq }}`, `{{ onSubmitLead }}` template slots | real ids and real handlers |
| `sc-camel-view-box` on 41 SVGs | `viewBox` (they were rendering blank) |
| 17 `<image-slot>` placeholders | real `<picture>` with WebP + fallback |
| `style-hover` / `style-focus` / `style-checked` | genuine CSS states |
| **0 media queries**, fixed-pixel layout | responsive at 1100 / 760 / 520px |
| no `<title>`, no meta, no OG | full SEO + social metadata |
| framework runtime `<script>` tags | removed |

Verified at 375px: no horizontal overflow and no element wider than the viewport.

---

## Go-live checklist

Edit the `window.PNR` block at the top of `index.html`.

- [ ] **`leadEndpoint`** — the deployed `pnr-lead-relay` Worker URL.
      **Until this is set the forms refuse to submit and tell the visitor to
      call instead.** They never fake a success.
- [x] **`gtmId`** — set to `GTM-N98QJB5R`, with the matching `<noscript>`
      iframe as the first element inside `<body>`.
- [ ] **CallRail** — wired to company `929436290`. Confirm it is right for this rooftop.
- [ ] **Inventory is sample data.** The six cards carry placeholder prices,
      mileage and stock. Replace with real units before spending on traffic.
- [ ] **Two vehicle photos do not exist** (see below).
- [ ] **Map** — currently an OpenStreetMap embed centred on the dealership.
      Swap for a Google Maps embed if you prefer, and confirm the pin location.

---

## Lead flow

Both forms POST to the shared `pnr-lead-relay` Worker, which converts the
submission into a DealerCenter `<ac_application>` document and posts it to the
**Prospect API**. The Worker lives in the `picknride-fleet-lp` repo under
`worker/` — one relay serves both landing pages.

| Form | Fields | `form_name` |
|---|---|---|
| Test drive (mid page) | first name, last name, phone, looking-for | `leadForm` |
| Talk to an expert (footer) | name, phone, email, message, consent | `footerForm` |

Attribution (`gclid`, `gbraid`/`wbraid`, all `utm_*`, `campaignid`, `adgroupid`,
`creative`, `keyword`, `matchtype`, `device`, geo) is captured on load,
persisted in `sessionStorage` under `pnr_attr` with **first touch winning**, and
carried into the DealerCenter `comments` field. That is what makes Google Ads
offline conversion import possible once deals close.

Spam control: a honeypot field plus a 2-second minimum time-to-submit. Both
return a normal-looking success and deliver nothing.

---

## Images

**39.7 MB → 2.6 MB** as responsive WebP with JPEG fallbacks.

Vehicle photos are matched to the make and model printed on each card:

| Card | Photo | Source |
|---|---|---|
| 2018 Toyota RAV4 | *(none)* | **no RAV4 photo exists** |
| 2017 Chevrolet Express 2500 Cargo | `veh-express-*` | `10.jpeg` |
| 2018 GMC Savana 2500 Cargo | *(none)* | **no Savana photo exists** |
| 2019 RAM 2500 Regular Cab | `veh-ram2500-*` | `3.webp` |
| 2019 Honda Civic | `veh-civic-*` | `5.webp` |
| 2021 Toyota Tacoma Double Cab | `veh-tacoma-*` | `4.webp` |

### Why two cards have no photo

The asset set contains ten vehicle photos — Nissan Titan XD, Chevrolet
Silverado, RAM 2500, Toyota Tacoma, Honda Civic, Chevrolet Express (passenger
and cargo), Mercedes Sprinter, Ford Transit, RAM ProMaster. **None of them is a
Toyota RAV4 or a GMC Savana.**

Putting any other vehicle under those headings would advertise a vehicle the
dealership is not selling, on a licensed dealer's paid landing page. Those two
slots therefore render an explicit "Photo coming soon" tile until either the
real photos arrive or the listings are replaced with actual inventory.

### Customer gallery

The grid is 4×2 = **8 tiles**, but the build doc lists **nine** photos
(`c3 c9 c2 c4 c5 c6 c7 c8 c1`). The first eight are used in that order so both
rows stay full; `c1` is optimised and committed as `gal-9-*` but is not placed.
Say the word and the grid can go 3×3 to fit all nine.

### Hero photography

The hero uses `Ad 1.jpeg` as the build doc specifies. Worth knowing: the model
is barefoot in `Ad 1`–`Ad 5`, `Ad 7` and `Ad 8`. For a page selling vehicles to
Houston buyers that reads off-message. `Ad 6` — the aerial lot shot showing real
inventory, model in shoes — is used for the "Why Pick & Ride" background and is
the stronger trust image if you want to swap the hero too.

---

## Hosting

Deploy is a direct upload of a **staged public-only directory**, never the repo
root — everything uploaded to Pages is publicly fetchable:

```bash
rm -rf dist && mkdir dist
cp index.html _headers dist/ && cp -r assets dist/assets
npx wrangler pages deploy dist --project-name=picknride-usedcars-lp --branch=main
```

`picknrideauto.com` runs on GoDaddy nameservers, so point the subdomain with:

```
CNAME   call   picknride-usedcars-lp.pages.dev
```

---

## Bilingual (EN/ES)

English is the default. Spanish is opt-in via the toggle in the header and in
the footer bar, and the choice is remembered in `localStorage` under
`pnr_lang`. A visitor with no stored preference always gets English, including
when their browser is set to Spanish.

Switching walks the DOM and swaps any string found in `DICT`, keeping the
English original in a lookaside so switching back is exact rather than a
re-translation. Anything absent from `DICT` is left alone: phone numbers,
prices, mileage, vehicle names, the address.

To reword or add Spanish copy, edit `DICT` in the i18n `<script>` block. That
is the only place to change.

### Two things that are deliberate

- **Checkbox `value` attributes are never translated.** Those values go to
  DealerCenter, so the CRM must receive `Truck` regardless of the language the
  visitor read the page in. Only the visible label changes.
- **Customer reviews stay in the language the customer wrote them in.** They
  are signed with real names; translating a signed review puts words in a real
  person's mouth. If the client wants them translated, they should be labelled
  as translations.

The form now also sends `preferred_language: 'Spanish'` when the page is in
Spanish. The Worker already supported the field but nothing was populating it,
so DealerCenter had no way to know which language to call the lead back in.

### Careful: HTML entities merge text nodes

`Pick &amp; Ride` looks like three fragments in the source but is **one** text
node in the DOM. A dictionary written by reading the HTML silently fails on
every string containing `&`. `tests/test_i18n_coverage.py` walks the real DOM
and fails the build if any visible string has no translation and is not on the
allow-list. Run it after touching copy.

## Tests

```bash
pip install playwright && python3 -m playwright install chromium
python3 -m http.server 8789 &          # from this directory
python3 tests/test_page.py http://127.0.0.1:8789/           # 42 assertions
python3 tests/test_i18n_coverage.py http://127.0.0.1:8789/  # translation coverage
```
