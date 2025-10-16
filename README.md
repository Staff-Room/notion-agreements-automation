# Notion Agreements Automation

🤝 **n8n workflow to send Notion agreements via Documenso for e-signature**

## What It Does

```
Notion Agreement → n8n Workflow → Documenso Envelope → Email to Signer
```

1. Webhook triggered from Notion agreement page
2. Fetches agreement data from Notion API  
3. Creates Documenso envelope from template
4. Updates Notion with envelope ID and signing URL
5. Sends signing link to counterparty

## Quick Start

### 1. Setup Environment Variables in n8n

```bash
NOTION_TOKEN=secret_xxx
DOCUMENSO_API_URL=https://your-instance.com
DOCUMENSO_TOKEN=your_api_token  
DOCUMENSO_TEMPLATE_ID=your_template_id
```

### 2. Import n8n Workflow

```bash
# Import the workflow JSON
curl -X POST http://localhost:5678/api/v1/workflows/import \
  -H "Content-Type: application/json" \
  -d @workflows/documenso-send.json
```

### 3. Test the Workflow

```bash
# Trigger the webhook
curl -X POST http://localhost:5678/webhook/agreements/send \
  -H "Content-Type: application/json" \
  -d '{"agreement_url": "https://notion.so/your-agreement-id"}'
```

## Notion Database Requirements

Your Agreements database needs these fields:
- `Agreement Name` (Title)
- `Contact` (Relation to People)
- `Company` (Relation to Organizations)  
- `Amount` (Number)
- `Status` (Status: Draft/Negotiating/Finalized)
- `Envelope ID` (Text)
- `Signing URL` (URL)

## Files

- `workflows/documenso-send.json` - Main n8n workflow
- `CLAUDE.md` - Development setup guide

## How It Works

### n8n Workflow Steps

1. **Webhook Trigger** - Listens for agreement send requests
2. **Fetch Agreement** - Gets data from Notion API
3. **Fetch Contact** - Gets signer email from related Contact
4. **Create Envelope** - Sends agreement to Documenso API
5. **Update Notion** - Stores envelope ID and signing URL back to Notion

### API Calls

**Notion API**: Read agreement properties and related records  
**Documenso API**: Create envelope from template with merge fields  
**Notion API**: Update agreement with envelope details

## Testing

Create a test agreement in Notion, then trigger the workflow. Check n8n execution logs for any API errors.

## Common Issues

- **401 Unauthorized**: Check API tokens in n8n environment
- **Template not found**: Verify Documenso template ID
- **Missing contact email**: Ensure Contact relation has email field

The core workflow handles the Notion → Documenso integration. Additional workflows can handle signature completion webhooks and billing automation.