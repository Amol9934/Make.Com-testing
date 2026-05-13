# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the app

No build step. Open `index.html` directly in a browser, or serve it with any static file server:

```bash
npx serve .
# or
python -m http.server 8080
```

No dependencies to install.

## Architecture

The entire app is a single `index.html` file — HTML structure, CSS (inline `<style>`), and JavaScript (inline `<script>`). It is deployed as a static site on Vercel.

**Data flow:**
1. Make.com builds a URL with invoice fields as query parameters (spaces encoded as `%20` using `encodeURL()`).
2. On load, `init()` reads the query string with `new URLSearchParams(window.location.search)`. `URLSearchParams.get()` automatically decodes percent-encoded values — no extra `decodeURIComponent` needed.
3. If no `id` or `invoiceName` is present, `loadDemo()` injects hardcoded demo data and renders without a live webhook.
4. `renderPage()` → `renderFields()` builds the invoice details grid from the `FIELDS` array.
5. On Approve/Reject, `submitAction()` POSTs a JSON payload to `webhookUrl`. In demo mode (no `webhookUrl`), it skips the fetch and calls `showSuccess()` directly.

**The `FIELDS` array** (top of `<script>`) is the single source of truth for which fields are displayed, their labels, types, and whether they're editable. Adding or removing a field means one entry there.

**Edit mode:** toggled by `toggleEdit()`. In edit mode, `renderFields()` renders `<input>` elements instead of static `<div>`s. On save, values are written back to the `data` object.

**UI states** (mutually exclusive `display` toggling):
- `#loading-state` — shown on initial load
- `#main-content` — the review form
- `#success-state` — shown after a successful submit
- `#error-state` — shown inline on webhook fetch failure
- `#no-data-state` — shown when URL has no usable params (currently falls through to demo mode)

## Integration

Make.com builds the review URL using `encodeURL()` on each field value so spaces become `%20`. Example:

```
https://make-com-testing.vercel.app/?id=INV-002&invoiceName=Office%20Supplies%20May&amount=3200&vendorName=Staples%20India&invoiceNumber=STP-4421&invoiceDate=2026-05-03&dueDate=2026-05-20&category=Operations&submittedBy=Priya%20Singh&webhookUrl=https://hook.eu1.make.com/xyz
```

On Approve/Reject the page POSTs this JSON to `webhookUrl`:

```json
{
  "id": "INV-002",
  "status": "Approved",
  "invoiceName": "Office Supplies May",
  "amount": "3200",
  "vendorName": "Staples India",
  "invoiceNumber": "STP-4421",
  "invoiceDate": "2026-05-03",
  "dueDate": "2026-05-20",
  "category": "Operations",
  "comments": "",
  "processedAt": "2026-05-13T10:30:00.000Z"
}
```
