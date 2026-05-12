# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the app

No build step. Open `index.html` directly in a browser, or serve it with any static file server:

```
npx serve .
# or
python -m http.server 8080
```

There are no dependencies to install, no package.json, no bundler.

## Architecture

The entire app lives in `index.html` — HTML structure, CSS (inline `<style>`), and JavaScript (inline `<script>`).

**Data flow:**
1. On load, `init()` reads URL query parameters (`id`, `invoiceName`, `amount`, `vendorName`, `invoiceNumber`, `invoiceDate`, `dueDate`, `category`, `submittedBy`, `status`, `webhookUrl`).
2. If no `id` or `invoiceName` is present, `loadDemo()` injects hardcoded demo data and renders without a live webhook.
3. `renderPage()` → `renderFields()` builds the invoice details grid dynamically from the `FIELDS` array.
4. On Approve/Reject, `submitAction()` POSTs a JSON payload to `webhookUrl` (a Make.com webhook). In demo mode (no `webhookUrl`), it skips the fetch and goes straight to `showSuccess()`.

**The `FIELDS` array** (top of `<script>`) is the single source of truth for which fields are displayed, their labels, types, and whether they're editable. Adding or removing a field means adding/removing one entry there.

**Edit mode:** toggled by `toggleEdit()`. In edit mode, `renderFields()` renders `<input>` elements instead of static `<div>`s. On save, values are written back to the `data` object.

**UI states** (mutually exclusive visibility):
- `#loading-state` — shown on initial load
- `#no-data-state` — shown when URL has no usable params (currently unreachable; `loadDemo()` runs instead)
- `#error-state` — shown inline on webhook fetch failure
- `#main-content` — the review form
- `#success-state` — shown after a successful submit

## Integration

This page is intended to be linked from Make.com (or a similar automation platform) with invoice data encoded in the URL query string. The `webhookUrl` param receives the Make.com webhook endpoint that processes the approval/rejection response.
