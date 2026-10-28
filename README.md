# Basic Google Tag Manager Setup Plan

**Digital Marketing Track – Task:** Plan tags, triggers and variables in Google Tag Manager (GTM) to track important website interactions (button clicks, form submissions, downloads) and send them to Google Analytics 4 (GA4).

**Author:** Reddi Malli Nandini
**Tools:** Google Tag Manager, Google Analytics 4

---

## Objective

Understand the fundamentals of tag management: what a **tag**, a **trigger** and a **variable** are, how they work together, and how to plan tracking *before* building it.

| Concept | What it is | Example in this project |
|---|---|---|
| **Tag** | Code that sends data somewhere (here, to GA4) | `GA4 - Event - file_download` |
| **Trigger** | The rule that decides *when* a tag fires | `Click - File Downloads` (link URL ends in `.pdf`) |
| **Variable** | A value GTM reads at the moment of the event | `{{Click URL}}`, `{{CJS - File Extension}}` |
| **Data layer** | A JavaScript array the website uses to pass information to GTM | `dataLayer.push({ event: 'form_submit_success' })` |

## Deliverables

| Deliverable | File |
|---|---|
| Measurement goals & naming conventions | [`docs/01-measurement-goals.md`](docs/01-measurement-goals.md) |
| **Tag plan** | [`docs/02-tag-plan.md`](docs/02-tag-plan.md) |
| **Trigger plan** | [`docs/03-trigger-plan.md`](docs/03-trigger-plan.md) |
| **Variable plan** | [`docs/04-variable-plan.md`](docs/04-variable-plan.md) |
| **Event tracking map** | [`docs/05-event-tracking-map.md`](docs/05-event-tracking-map.md) · [CSV](docs/event-tracking-map.csv) |
| Testing & publishing checklist | [`docs/06-testing-and-publishing.md`](docs/06-testing-and-publishing.md) |
| Demo website with GTM-ready markup | [`demo-site/index.html`](demo-site/index.html) |

## Plan at a glance

The demo site is a landing page for a fictional Hyderabad tea brand, *Charminar Chai Co.*

| User action | GA4 event | Trigger | Key event |
|---|---|---|---|
| Opens a page | `page_view` | Initialization - All Pages | – |
| Clicks a call-to-action button | `cta_click` | Click - CTA Buttons | – |
| Downloads the catalogue PDF | `file_download` | Click - File Downloads | – |
| Submits the enquiry form successfully | `generate_lead` | Custom Event - form_submit_success | ✅ |
| Clicks phone / WhatsApp / email | `contact_click` | Click - Contact Links | ✅ |
| Clicks a link to another website | `outbound_click` | Click - Outbound Links | – |
| Scrolls 50% / 90% of the page | `scroll_depth` | Scroll - 50 / 90 | – |

**Totals:** 7 tags · 7 triggers · 9 user-defined variables (+ built-in click, form, page and scroll variables)

```
User clicks "Download catalogue (PDF)"
        │
        ▼
Trigger  Click - File Downloads     ← {{Click URL}} matches \.(pdf|docx?|...)$
        │
        ▼
Tag      GA4 - Event - file_download
        │   file_name      = {{CJS - File Name}}
        │   file_extension = {{CJS - File Extension}}
        │   link_url       = {{Click URL}}
        ▼
GA4      event: file_download  →  Reports / DebugView
```

## Key decisions

1. **Measure first, tag second.** Every tag maps to a business question in [`01-measurement-goals.md`](docs/01-measurement-goals.md).
2. **Form leads use a data layer event**, pushed only after validation passes, instead of the built-in Form Submission trigger (which can fire on failed submits).
3. **No personal data goes to GA4.** The form sends only `form_id`, `form_name` and `enquiry_type`, never the visitor's name or email.
4. **No double counting.** GA4 Enhanced Measurement's outbound click, file download, scroll and form interaction toggles are switched off, because GTM tracks these.
5. **GA4 recommended event names** (`generate_lead`, `file_download`) are used where they exist; custom names use lowercase snake_case.
6. **Consistent naming** (`GA4 - Event - …`, `Click - …`, `DLV - …`) so the container stays readable as it grows.

## Repository structure

```
gtm-setup-plan/
├── README.md
├── LINKEDIN_POST.md
├── index.html                  # redirects to the demo site (for GitHub Pages)
├── docs/
│   ├── 01-measurement-goals.md
│   ├── 02-tag-plan.md
│   ├── 03-trigger-plan.md
│   ├── 04-variable-plan.md
│   ├── 05-event-tracking-map.md
│   ├── 06-testing-and-publishing.md
│   └── event-tracking-map.csv
├── demo-site/
│   ├── index.html              # GTM snippet + tracking-ready elements + live dataLayer viewer
│   └── assets/charminar-chai-catalogue.pdf
└── screenshots/
```

## Try the demo site

1. Open `demo-site/index.html` in a browser (or enable GitHub Pages: *Settings → Pages → Deploy from branch → main / root*).
2. The **dataLayer (live)** panel in the bottom-right corner shows every event as it happens: GTM's own events (`gtm.js`, `gtm.click`, `gtm.linkClick`, `gtm.scrollDepth`) and the site's `form_submit_success`.
3. To connect a real container, replace both `GTM-XXXXXXX` placeholders in `demo-site/index.html` with your GTM container ID, then build the tags, triggers and variables from the `docs/` folder.

## How to test

Use GTM **Preview** mode (Tag Assistant) and GA4 **DebugView**. The full checklist, including the "empty form must not fire `generate_lead`" test, is in [`docs/06-testing-and-publishing.md`](docs/06-testing-and-publishing.md).

## What I learned

- Tags, triggers and variables are separate on purpose: one variable (like the Measurement ID) can be reused by every tag, and one trigger can feed several tags.
- Click triggers need the right built-in variables enabled first, or `{{Click URL}}` and `{{Click Text}}` stay empty.
- A data layer push from the website is the most reliable way to track things GTM can't see on its own, such as a *successful* form submission.
- GA4 needs custom parameters registered as custom dimensions before they show up in reports.
