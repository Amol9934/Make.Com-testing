# Invoice Review

A single-page invoice approval interface for Human-in-the-Loop workflows with Make.com. Make.com builds a review URL with invoice data as query parameters and sends it to the approver. The approver clicks the link, reviews the invoice, and approves or rejects it — the decision is POSTed back to a Make.com webhook.

**Live:** https://make-com-testing.vercel.app/

## How it works

```
Make.com builds URL with encoded invoice fields
          ↓
Reviewer opens link → page reads query params → displays invoice
          ↓
Reviewer approves/rejects → POST to webhookUrl
```

## URL format

Make.com should build the URL using `encodeURL()` on each field value so spaces become `%20`:

```
https://make-com-testing.vercel.app/?id=INV-002
  &invoiceName=Office%20Supplies%20May
  &amount=3200
  &vendorName=Staples%20India
  &invoiceNumber=STP-4421
  &invoiceDate=2026-05-03
  &dueDate=2026-05-20
  &category=Operations
  &submittedBy=Priya%20Singh
  &webhookUrl=https://hook.eu1.make.com/xyz
```

Opening the page without parameters shows demo data — no webhook is called.

## URL parameters

| Parameter      | Description                                      |
|----------------|--------------------------------------------------|
| `id`           | Unique invoice identifier                        |
| `invoiceName`  | Display name of the invoice                      |
| `amount`       | Numeric amount (displayed as ₹)                  |
| `vendorName`   | Vendor / supplier name                           |
| `invoiceNumber`| Vendor's invoice reference number                |
| `invoiceDate`  | Invoice date (e.g. `2026-05-03`)                 |
| `dueDate`      | Payment due date                                 |
| `category`     | Expense category                                 |
| `submittedBy`  | Name of person who submitted the invoice         |
| `webhookUrl`   | Make.com webhook URL to receive the decision     |

## Webhook payload (sent on approve/reject)

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
  "comments": "Looks good.",
  "processedAt": "2026-05-13T10:30:00.000Z"
}
```

`status` is `"Approved"` or `"Rejected"`. If the reviewer edited any field before submitting, the updated values are sent.

## Running locally

No build step — open `index.html` directly in a browser, or:

```bash
npx serve .
```
