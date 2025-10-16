# Notion Agreements Automation

**Core Project**: n8n workflow to send Notion agreements via Documenso for e-signature.

## What This Does

Takes a Notion agreement → Creates Documenso envelope → Sends for signature → Updates Notion status.

## Current Status

✅ **Working**: Notion integration, Documenso API calls, status updates  
❌ **Broken**: PDF generation from Notion page content

The workflow successfully:
- Fetches agreement data from Notion using proper Notion nodes
- Converts Notion blocks to HTML format  
- Calls Documenso API to create document envelope
- Updates Notion with envelope ID and signing URL

**Issue**: PDF generation step fails. Need working solution to convert Notion page content to PDF.

## PDF Generation Attempts

We've tried:
1. **Hallucinated API** - `https://api.html-css-to-pdf.com` (doesn't exist)
2. **API2PDF** - `https://v2.api2pdf.com/chrome/pdf/html` (tested working, but n8n integration fails)
3. **Need**: Working PDF generation node for n8n workflow

API2PDF test call works:
```bash
curl -X POST "https://v2.api2pdf.com/chrome/pdf/html" \
  -H "Authorization: cfbb9986-6318-4089-84e7-c368fcd9c476" \
  -d '{"html": "<html><body><h1>Test</h1></body></html>"}'
# Returns: {"Success":true, "FileUrl":"https://storage.googleapis.com/..."}
```

## Setup

```bash
# n8n Environment Variables
NOTION_TOKEN=secret_xxx
DOCUMENSO_TOKEN=xxx
API2PDF_KEY=cfbb9986-6318-4089-84e7-c368fcd9c476
```

## Key Files

- `workflows/documenso-send.json` - Original workflow (uses HTTP nodes)
- `workflows/documenso-send-updated.json` - Updated workflow (uses Notion nodes + PDF gen)
- Notion database ID: `c0923656ee1d41de882310cab0ddbd3f`

## Workflow Steps

1. **Webhook Trigger**: `/webhook/agreements/send`
2. **Get Agreement Page**: Notion node to fetch agreement
3. **Get Page Blocks**: Notion node to fetch content blocks  
4. **Convert to HTML**: Function node converts blocks to styled HTML
5. **Generate PDF**: ❌ API2PDF call (broken in n8n)
6. **Create Documenso Document**: Working API call
7. **Upload PDF**: Upload to Documenso S3 bucket
8. **Update Notion**: Store envelope ID + signing URL

## Next Steps

Fix PDF generation step to complete the automation.