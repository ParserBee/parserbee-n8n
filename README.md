# ParserBee workflows for n8n

Ready-to-import [n8n](https://n8n.io) workflows that send invoices, receipts, purchase orders and other documents to the [ParserBee](https://parserbee.com) API and put the extracted data where you need it: Google Sheets, a CSV file, or the response of your own webhook.

They use only n8n's built-in nodes (HTTP Request, Code, Google Drive, Gmail, Google Sheets), so there is nothing extra to install.

The folder workflow reads files from the n8n server's disk, so it needs self-hosted n8n (set the folder in **Read invoice files**; if you use `N8N_RESTRICT_FILE_ACCESS_TO`, the folder must be inside it). The other three also run on n8n Cloud.

## Workflows

| Workflow | Trigger | Output |
|---|---|---|
| [Extract invoices in a folder to CSV](workflows/extract-invoices-folder-to-csv.json) | Manual | One CSV, one row per line item |
| [Google Drive invoices to Google Sheets](workflows/google-drive-invoices-to-google-sheets.json) | New file in a Drive folder | Rows added to a sheet |
| [Gmail invoice attachments to Google Sheets](workflows/gmail-invoice-attachments-to-google-sheets.json) | New email with a PDF or image attachment | Rows added to a sheet |
| [Extract a document from a URL (webhook)](workflows/extract-document-from-url-webhook.json) | `POST` with a `file_url` | The extracted data as JSON |

## Set up

1. **Create a ParserBee account** at [app.parserbee.com](https://app.parserbee.com/create-account). New accounts get 25 free pages, no credit card.
2. **Create a Parser Template.** Upload one sample invoice and the AI builds the fields for you, or pick a ready-made one from the [template library](https://parserbee.com/templates/). Copy the template ID from the template page.
3. **Copy your API key** from **Settings** in the dashboard. It starts with `pk_`.
4. **Import a workflow** in n8n: open a new workflow, click the **...** menu, choose **Import from File**, and select a file from [workflows/](workflows). You can also paste the JSON straight onto the canvas.
5. **Add the credential.** Open the **ParserBee** HTTP Request node, and under **Credential for Header Auth** create a new one:
   - Credential name: `ParserBee API key`
   - Name: `x-api-key`
   - Value: your API key
6. **Paste your template ID** into the `template_id` field of the same node, in place of `PASTE_YOUR_TEMPLATE_ID`.
7. Connect your Google account where the workflow uses Drive, Gmail or Sheets, then click **Test workflow**.

## How the ParserBee step works

Each workflow calls one endpoint:

```
POST https://app.parserbee.com/api/v1/extract
x-api-key: pk_your_api_key
```

- To upload a file, send `multipart/form-data` with `template_id` and `file` (the binary field from the previous node).
- To extract a document that is already online, send JSON: `{ "template_id": "...", "file_url": "https://..." }`.

The response holds the extracted fields in `data`, shaped like your Parser Template:

```json
{
  "success": true,
  "request_id": "req_abc123",
  "data": {
    "invoice_number": "INV-2026-001",
    "vendor_name": "Acme Corp",
    "total_amount": 1500,
    "line_items": [
      { "description": "Consulting", "quantity": 10, "amount": 1500 }
    ]
  },
  "credits_remaining": 449,
  "usage": { "pages_processed": 1, "credits_charged": 1 }
}
```

The **One row per line item** Code node turns that into spreadsheet rows: the invoice fields are repeated on each row, and the line item fields get a `line_` prefix. It works with any Parser Template, because it takes the first list in the result as the line items.

Each extraction costs 1 page credit per page processed. Files can be PDF, PNG, JPG, WebP or HEIC, up to 50 MB; scans and phone photos work.

## Test files

No invoices at hand? The [sample-invoices](https://github.com/ParserBee/sample-invoices) repository has 18 free test invoices (digital PDFs, scans and phone photos) with the correct values for every field.

## Other ways to connect

- **Zapier:** ParserBee has a Zapier integration. See the [Zapier guide](https://parserbee.com/help/zapier-integration/).
- **Code:** [parserbee-python](https://github.com/ParserBee/parserbee-python), [parserbee-node](https://github.com/ParserBee/parserbee-node), or the [Postman collection](https://github.com/ParserBee/parserbee-postman).
- **No automation:** Data Lab in the ParserBee dashboard takes a batch of files and exports one Excel, CSV or Google Sheets file.

Choosing a tool for invoice workflows? Our guide to [invoice automation software](https://parserbee.com/blog/best-invoice-automation-software/) compares the options for small teams, and the [invoice OCR software](https://parserbee.com/blog/best-invoice-ocr-software/) test covers extraction accuracy.

## Links

- [ParserBee](https://parserbee.com)
- [API reference](https://app.parserbee.com/docs)
- Support: support@parserbee.com

## License

MIT
