# Notion Agreements Automation

🤝 **Automated agreement lifecycle management with e-signatures and billing**

A complete automation system that transforms Notion into a powerful contract management platform, integrating with Documenso (open-source DocuSign alternative) for e-signatures and Stripe for automated billing.

## 🎯 Overview

This system enables you to:
- Draft agreements quickly from templates with minimal inputs
- Send for signature via Documenso with one click
- Automatically trigger downstream actions on signature completion
- Generate and manage recurring monthly invoices via Stripe
- Keep all data synchronized across Notion, Documenso, and Stripe

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│     Notion      │    │      n8n        │    │   Documenso     │
│   Agreements    │◄──►│   Middleware    │◄──►│  E-Signature    │
│   Database      │    │   Workflows     │    │   Platform      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                               │
                               ▼
                       ┌─────────────────┐
                       │     Stripe      │
                       │    Billing      │
                       │   Platform      │
                       └─────────────────┘
```

### Core Components

- **Notion Database**: Central hub for agreement data and status tracking
- **n8n Workflows**: Middleware orchestration for API integration
- **Documenso**: Open-source e-signature platform
- **Stripe**: Automated billing and invoice generation

## 🚀 Features

### Agreement Management
- ✅ **Status Tracking**: Draft → Negotiating → Finalized → Closed Won/Lost
- ✅ **Relationship Management**: Links to Companies, Contacts, and Projects
- ✅ **Document Snapshots**: Captures billing information at agreement time
- ✅ **Audit Trail**: Complete history of all status changes and actions

### E-Signature Integration
- ✅ **One-Click Sending**: Generate and send agreements with single button click
- ✅ **Template Merging**: Auto-populate agreement templates with Notion data
- ✅ **Status Synchronization**: Real-time updates from Documenso webhooks
- ✅ **Document Storage**: Automatic linking of signed PDFs

### Billing Automation
- ✅ **Customer Management**: Auto-create Stripe customers from agreement data
- ✅ **Invoice Generation**: Immediate invoice creation upon signature
- ✅ **Recurring Billing**: Monthly invoices aligned to agreement start dates
- ✅ **Proration Support**: Handle mid-cycle agreement starts

### Workflow Automation
- ✅ **Project Activation**: Set related projects to "In Progress" on signature
- ✅ **Error Handling**: Robust retry logic and error notifications
- ✅ **Webhook Security**: Validated webhook signatures for all integrations

## 📊 Database Schema

### 🤝 Agreements Database

| Field | Type | Description |
|-------|------|-------------|
| Agreement Name | Title | Primary identifier for the agreement |
| Company | Relation | Link to Organizations database |
| Contact | Relation | Link to People database |
| Counterparty | Relation | Primary signer for the agreement |
| Project | Relation | Related project that will be activated |
| Amount | Number | Monthly billing amount |
| Currency | Select | Currency for billing (USD supported) |
| Agreement Date | Date | Start date for the agreement |
| Status | Status | Workflow status with automation triggers |
| Doc Provider | Select | Document provider (Documenso) |
| Envelope ID | Text | Provider-specific envelope identifier |
| Signing URL | URL | Active signing session link |
| Signed PDF URL | URL | Final signed document link |
| Billing Email (snapshot) | Email | Captured at send time |
| Billing Address (snapshot) | Text | Captured at send time |
| Notes | Text | Additional agreement details |

## 🔄 Workflows

### 1. Agreement Preparation & Sending

**Trigger**: Button click on agreement page  
**Actions**:
1. Fetch agreement and related data from Notion
2. Create Documenso envelope from template
3. Generate signing URL and send to counterparty
4. Update agreement status to "Negotiating"
5. Store envelope ID and signing URL

### 2. Signature Completion Processing

**Trigger**: Documenso webhook on signature completion  
**Actions**:
1. Update agreement status to "Finalized"
2. Store signed PDF URL
3. Update related project status to "In Progress"
4. Create/update Stripe customer
5. Generate first invoice (prorated if needed)
6. Schedule monthly recurring billing

## 🛠️ Setup Instructions

### Prerequisites

- Notion workspace with appropriate permissions
- n8n instance (local or cloud)
- Documenso instance (self-hosted or cloud)
- Stripe account with API access

### Environment Variables

```bash
# Notion Integration
NOTION_TOKEN=secret_xxx
NOTION_DATABASE_ID=your_agreements_database_id

# Documenso Integration
DOCUMENSO_API_URL=https://your-documenso-instance.com
DOCUMENSO_TOKEN=your_api_token
DOCUMENSO_TEMPLATE_ID=your_template_id
DOCUMENSO_WEBHOOK_SECRET=your_webhook_secret

# Stripe Integration
STRIPE_SECRET_KEY=sk_live_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx

# n8n Configuration
N8N_BASE_URL=https://your-n8n-instance.com
```

### Installation Steps

1. **Clone Repository**
   ```bash
   git clone https://github.com/staffroom/notion-agreements-automation.git
   cd notion-agreements-automation
   ```

2. **Configure Notion Database**
   - Import the provided database schema
   - Set up appropriate relations to Companies, People, and Projects

3. **Set up n8n Workflows**
   - Import workflow JSON files from `/workflows` directory
   - Configure environment variables in n8n
   - Test webhook endpoints with ngrok during development

4. **Configure Documenso**
   - Create agreement template with merge fields
   - Set up webhook endpoint pointing to n8n
   - Test envelope creation and signing flow

5. **Configure Stripe**
   - Set up webhook endpoints for invoice events
   - Configure payment terms and tax settings
   - Test customer and invoice creation

## 📁 Repository Structure

```
notion-agreements-automation/
├── README.md                          # This file
├── CLAUDE.md                          # Claude AI setup instructions
├── docs/                              # Additional documentation
│   ├── api-reference.md               # API endpoint documentation
│   ├── webhook-specs.md               # Webhook payload specifications
│   └── troubleshooting.md             # Common issues and solutions
├── workflows/                         # n8n workflow definitions
│   ├── agreement-preparation.json     # Prepare & send workflow
│   └── signature-completion.json      # Signature processing workflow
├── notion/                            # Notion templates and schemas
│   ├── database-schema.json           # Agreements database schema
│   └── sample-pages.md               # Sample agreement templates
├── documenso/                         # Documenso templates
│   └── agreement-template.pdf         # Base agreement template
└── scripts/                           # Utility scripts
    ├── setup-webhooks.sh              # Webhook configuration script
    └── test-integration.js            # End-to-end testing script
```

## 🧪 Testing

### Manual Testing Flow

1. Create test agreement in Notion with sample data
2. Click "Prepare & Send" button
3. Verify Documenso envelope creation
4. Complete signature in Documenso test environment
5. Verify status updates and Stripe customer/invoice creation

### Automated Testing

```bash
# Run integration tests
npm test

# Test webhook endpoints
npm run test:webhooks

# Validate n8n workflows
npm run test:workflows
```

## 🔒 Security Considerations

- **API Key Management**: All sensitive credentials stored as environment variables
- **Webhook Validation**: All incoming webhooks validated with signatures
- **Data Privacy**: Minimal data exposure in logs and error messages
- **Access Control**: Proper permissions on n8n instance and webhook endpoints

## 📈 Monitoring & Logging

### Key Metrics to Monitor

- Agreement status transition success rates
- Documenso envelope creation/completion rates
- Stripe customer and invoice creation success
- Webhook processing latency and error rates

### Logging Strategy

- All API calls logged with request/response data
- Envelope IDs and agreement URLs tracked throughout workflow
- Error notifications via email/Slack for critical failures

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- 📧 **Email**: support@staffroom.ai
- 📚 **Documentation**: [Full Documentation](./docs/)
- 🐛 **Issues**: [GitHub Issues](https://github.com/staffroom/notion-agreements-automation/issues)

## 🙏 Acknowledgments

- [Notion](https://notion.so) - Database and workspace platform
- [Documenso](https://documenso.com) - Open-source e-signature solution
- [n8n](https://n8n.io) - Workflow automation platform
- [Stripe](https://stripe.com) - Payment processing and billing