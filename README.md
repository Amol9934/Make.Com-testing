# Invoice Review

A lightweight, single-page invoice approval interface designed to work with Make.com automations. Reviewers receive a link, inspect invoice details, optionally edit fields, add comments, and approve or reject — the decision is POSTed back to a webhook.

## Usage

Open `index.html` in a browser, or serve it statically:

```bash
npx serve .
# or
python -m http.server 8080
```

No dependencies, no build step.

## URL Parameters

The page is driven entirely by query parameters. Generate links from your automation with these fields:

| Parameter      | Description                              |
|----------------|------------------------------------------|
| `id`           | Unique invoice identifier                |
| `invoiceName`  | Display name of the invoice              |
| `amount`       | Numeric amount (displayed as ₹)          |
| `vendorName`   | Vendor / supplier name                   |
| `invoiceNumber`| Vendor's invoice reference number        |
| `invoiceDate`  | Invoice date (e.g. `2026-05-01`)         |
| `dueDate`      | Payment due date                         |
| `category`     | Expense category                         |
| `submittedBy`  | Name of person who submitted the invoice |
| `status`       | Initial status (default: `PENDING`)      |
| `webhookUrl`   | Make.com webhook URL to receive the decision |

**Example:**
```
index.html?id=INV-001&invoiceName=Software+Sub&amount=12500&vendorName=Acme&webhookUrl=https://hook.make.com/...
```

If `id` and `invoiceName` are absent, the page loads with demo data and no webhook is called.

## Webhook Payload

On approve or reject, the page POSTs JSON to `webhookUrl`:

```json
{
  "id": "INV-001",
  "status": "Approved",
  "invoiceName": "Software Sub",
  "amount": "12500",
  "vendorName": "Acme",
  "invoiceNumber": "ACM-9921",
  "invoiceDate": "2026-05-01",
  "dueDate": "2026-05-31",
  "category": "Software & Licenses",
  "comments": "Looks good.",
  "processedAt": "2026-05-12T10:30:00.000Z"
}
```

`status` is either `"Approved"` or `"Rejected"`. Edited field values are reflected in the payload.
