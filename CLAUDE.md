# CLAUDE.md

This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with the Notion Agreements Automation project.

## Project Overview

**Notion Agreements Automation** is a complete contract lifecycle management system that integrates:
- **Notion** as the central database for agreement tracking
- **Documenso** (open-source DocuSign alternative) for e-signatures  
- **Stripe** for automated billing and invoice generation
- **n8n** as middleware for workflow orchestration

The system automates the entire flow from agreement creation through signature collection to recurring billing setup.

## Architecture

```
Notion (Database) ↔ n8n (Workflows) ↔ Documenso (E-signatures) + Stripe (Billing)
```

## Project Structure

```
notion-agreements-automation/
├── README.md                          # Main project documentation
├── CLAUDE.md                          # This file - Claude setup guide
├── docs/                              # Additional documentation
├── workflows/                         # n8n workflow JSON files
├── notion/                            # Notion templates and schemas
├── documenso/                         # Document templates
└── scripts/                           # Setup and testing utilities
```

## Core Components

### 1. Notion Database Schema

The **🤝 Agreements** database contains these key fields:
- `Agreement Name` (Title) - Primary identifier
- `Company` (Relation) - Links to Organizations database
- `Contact` (Relation) - Links to People database
- `Amount` (Number) - Monthly billing amount
- `Status` (Status) - Workflow: Draft → Negotiating → Finalized → Closed Won/Lost
- `Doc Provider` (Select) - Set to "Documenso"
- `Envelope ID` (Text) - Documenso envelope identifier
- `Signing URL` (URL) - Link for counterparty to sign
- `Signed PDF URL` (URL) - Final signed document
- Billing snapshot fields for audit trail

### 2. n8n Workflows

**Workflow 1: Agreement Preparation & Sending**
- Triggered by webhook from Notion button
- Creates Documenso envelope from template
- Updates agreement status and URLs
- Sends signing link to counterparty

**Workflow 2: Signature Completion Handler** 
- Triggered by Documenso webhook
- Updates agreement status to "Finalized"
- Creates Stripe customer and invoice
- Sets up recurring monthly billing
- Updates related project status

### 3. Integration Points

- **Notion API**: For reading/updating agreement data
- **Documenso API**: For envelope creation and management
- **Stripe API**: For customer and invoice management
- **Webhooks**: For real-time status synchronization

## Environment Setup

### Required Services

1. **Notion Workspace** with Agreements database
2. **n8n Instance** (local Docker or cloud)
3. **Documenso Instance** (self-hosted or cloud)
4. **Stripe Account** with API access

### Environment Variables

```bash
# Notion Integration
NOTION_TOKEN=secret_xxx                 # Notion API token with database access
NOTION_DATABASE_ID=xxx                  # Agreements database ID

# Documenso Integration  
DOCUMENSO_API_URL=https://your-instance.com
DOCUMENSO_TOKEN=xxx                     # API token for envelope creation
DOCUMENSO_TEMPLATE_ID=xxx               # Template ID for agreements
DOCUMENSO_WEBHOOK_SECRET=xxx            # For webhook validation

# Stripe Integration
STRIPE_SECRET_KEY=sk_live_xxx           # Stripe API key
STRIPE_WEBHOOK_SECRET=whsec_xxx         # For webhook validation

# n8n Configuration
N8N_BASE_URL=https://your-n8n.com      # Base URL for webhook endpoints
```

## Common Development Tasks

### Setting Up New Environment

1. **Clone and Initialize**
   ```bash
   git clone <repo-url>
   cd notion-agreements-automation
   ```

2. **Configure Notion Database**
   - Import database schema from `notion/database-schema.json`
   - Ensure proper relations to Companies, People, Projects databases
   - Test database structure with sample data

3. **Set up n8n Workflows**
   ```bash
   # Import workflow files
   curl -X POST http://localhost:5678/api/v1/workflows/import \
     -H "Content-Type: application/json" \
     -d @workflows/agreement-preparation.json
   ```

4. **Configure Webhooks**
   - Set Documenso webhook to: `{N8N_BASE_URL}/webhook/documenso/completion`
   - Set Stripe webhook to: `{N8N_BASE_URL}/webhook/stripe/events`
   - Test webhook delivery with test payloads

### Testing the Integration

1. **Create Test Agreement**
   ```bash
   # Use the test script
   node scripts/test-integration.js --create-test-agreement
   ```

2. **Test Workflow Triggers**
   ```bash
   # Test prepare & send
   curl -X POST {N8N_BASE_URL}/webhook/agreements/prepare-send \
     -H "Content-Type: application/json" \
     -d '{"agreement_url": "https://notion.so/xxx", "action": "prepare_and_send"}'
   ```

3. **Validate End-to-End Flow**
   - Create agreement in Notion
   - Click "Prepare & Send" button
   - Complete signature in Documenso
   - Verify status updates and billing creation

### Debugging Common Issues

**Agreement Not Sending**
- Check n8n workflow execution logs
- Verify Notion token has database access
- Confirm Documenso template exists and is accessible

**Webhook Not Processing**
- Validate webhook URLs are accessible
- Check webhook secret configuration
- Review n8n error logs for parsing issues

**Billing Not Created**
- Verify Stripe API key permissions
- Check customer creation logs
- Confirm agreement has required billing fields

### Customizing the System

**Adding New Agreement Fields**
1. Update Notion database schema
2. Modify n8n workflows to handle new fields
3. Update Documenso template if needed
4. Test end-to-end flow with new fields

**Changing Billing Logic**
1. Modify the signature completion workflow
2. Update Stripe integration logic
3. Test with different billing scenarios
4. Update documentation

**Adding New Document Providers**
1. Create new workflow for provider API
2. Add provider option to Notion database
3. Update routing logic in n8n
4. Test integration thoroughly

## API Reference

### Webhook Endpoints

**Prepare & Send Agreement**
```
POST /webhook/agreements/prepare-send
{
  "agreement_url": "https://notion.so/xxx",
  "action": "prepare_and_send"
}
```

**Resend Signing Link**
```
POST /webhook/agreements/resend-link  
{
  "agreement_url": "https://notion.so/xxx",
  "action": "resend_link"
}
```

### Expected Webhook Payloads

**Documenso Completion**
```json
{
  "event": "envelope.completed",
  "envelope_id": "xxx",
  "signed_pdf_url": "https://...",
  "signer_email": "user@example.com",
  "completed_at": "2023-10-16T10:00:00Z"
}
```

**Stripe Invoice Events**
```json
{
  "type": "invoice.payment_succeeded",
  "data": {
    "object": {
      "id": "in_xxx",
      "customer": "cus_xxx",
      "status": "paid"
    }
  }
}
```

## Security Considerations

### API Key Management
- Store all secrets as environment variables
- Never commit API keys to repository
- Use separate keys for development/production
- Rotate keys regularly

### Webhook Security
- Validate all webhook signatures
- Use HTTPS for all webhook endpoints
- Implement proper error handling
- Log security events for audit

### Data Privacy
- Minimize data exposure in logs
- Use encrypted storage for sensitive data
- Implement proper access controls
- Follow data retention policies

## Troubleshooting Guide

### Common Error Messages

**"Agreement not found in Notion"**
- Verify agreement URL format
- Check Notion token permissions
- Confirm database ID is correct

**"Documenso envelope creation failed"**  
- Verify API token is valid
- Check template ID exists
- Confirm recipient email format

**"Stripe customer creation failed"**
- Verify API key permissions
- Check customer data format
- Review Stripe dashboard for errors

### Performance Optimization

**Slow Webhook Processing**
- Optimize n8n workflow steps
- Add parallel processing where possible
- Implement caching for repeated API calls

**High API Usage**
- Implement rate limiting in workflows
- Add exponential backoff for retries
- Cache frequently accessed data

### Monitoring & Alerting

**Key Metrics to Track**
- Agreement status transition rates
- Webhook processing latency
- API error rates
- Invoice generation success rates

**Alert Conditions**
- Webhook failures > 5% in 1 hour
- API response time > 5 seconds
- Agreement stuck in "Negotiating" > 7 days
- Invoice generation failures

## Development Workflow

### Making Changes

1. **Create Feature Branch**
   ```bash
   git checkout -b feature/description
   ```

2. **Test Locally**
   - Use ngrok for webhook testing
   - Test with Stripe test mode
   - Validate n8n workflow changes

3. **Deploy to Staging**
   - Update environment variables
   - Import updated workflows
   - Run integration tests

4. **Production Deployment**
   - Update production n8n instance
   - Monitor for errors
   - Validate end-to-end functionality

### Code Review Checklist

- [ ] All environment variables documented
- [ ] Webhook signatures validated
- [ ] Error handling implemented
- [ ] Tests pass in staging environment
- [ ] Documentation updated
- [ ] Security review completed

## Support Resources

- **Project Documentation**: `/docs` directory
- **n8n Documentation**: https://docs.n8n.io
- **Notion API Docs**: https://developers.notion.com
- **Documenso Docs**: https://documenso.com/docs
- **Stripe API Docs**: https://stripe.com/docs/api

When working on this project, always:
1. Test changes in a staging environment first
2. Monitor webhook processing after deployments
3. Keep environment variables secure
4. Document any new integrations or workflows
5. Follow the established error handling patterns