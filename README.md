# USN — n8n Email Workflows

All outbound email for United Scholars Network is handled by n8n workflows triggered via webhooks (except Workflow 7 which runs on a schedule).

## Workflows

| # | File | Trigger | Purpose |
|---|------|---------|---------|
| 1 | `01-application-received.json` | Webhook `/webhook/application-received` | Confirm application receipt |
| 2 | `02-partner-org-bulk-invite.json` | Webhook `/webhook/partner-invite` | Auto-approve partner org students |
| 3 | `03-application-approved.json` | Webhook `/webhook/application-approved` | Accept open applicants |
| 4 | `04-application-rejected.json` | Webhook `/webhook/application-rejected` | Reject applicants with feedback |
| 5 | `05-connection-request.json` | Webhook `/webhook/connection-request` | Mutual intro emails (parallel) |
| 6 | `06-event-rsvp.json` | Webhook `/webhook/event-rsvp` | RSVP confirmation |
| 7 | `07-pre-event-prep.json` | Schedule (daily 9 AM) | Conversation pair assignments 48h before event |
| 7B | `07b-welcome-sequence.json` | Webhook `/webhook/welcome-sequence` | 3-email onboarding drip (Day 1, 3, 7) |

## Setup

### 1. Import Workflows
In n8n, go to **Workflows > Import from File** and import each JSON file from the `workflows/` directory.

### 2. Configure Credentials
Each workflow requires these credentials (configure once, shared across all workflows):

- **SMTP** — Gmail SMTP for `membership@unitedscholarsnetwork.org`
  - Host: `smtp.gmail.com`, Port: `465`, SSL: `true`
  - Or use SendGrid/Resend SMTP credentials
- **Airtable API Token** — For application tracking (Workflows 1-4)
- **Supabase** (Workflow 7 only) — `SUPABASE_URL` and `SUPABASE_SERVICE_ROLE_KEY` as environment variables

### 3. Update Credential IDs
After creating credentials in n8n, update the credential `id` references in each workflow JSON. Look for `SMTP_CREDENTIAL_ID`, `AIRTABLE_CREDENTIAL_ID`, and `SUPABASE_CREDENTIAL_ID` placeholders.

### 4. Update Airtable Base ID
Replace `appXXXXXXXXXXXXXX` in Workflows 1-4 with your actual Airtable base ID.

### 5. Supabase RPC Functions (Workflow 7)
Create these Supabase RPC functions:
- `get_events_in_48_hours` — Returns events where `event_date` is within 48 hours of now
- `get_conversation_pairs_for_event(event_id)` — Returns conversation pairs joined with profiles for the given event

### 6. Environment Variables (Lovable/Supabase)
Store the production webhook URLs in your platform environment:

| Variable | Workflow |
|----------|----------|
| `N8N_WEBHOOK_APPLICATION_RECEIVED` | 1 |
| `N8N_WEBHOOK_PARTNER_INVITE` | 2 |
| `N8N_WEBHOOK_APPLICATION_APPROVED` | 3 |
| `N8N_WEBHOOK_APPLICATION_REJECTED` | 4 |
| `N8N_WEBHOOK_CONNECTION_REQUEST` | 5 |
| `N8N_WEBHOOK_EVENT_RSVP` | 6 |
| `N8N_WEBHOOK_WELCOME_SEQUENCE` | 7B |

Workflow 7 runs on a schedule and queries Supabase directly — no webhook URL needed.

### 7. Activate Workflows
After testing with n8n's test webhook feature, toggle each workflow to **Active**.

## Email Sender
- **From**: `membership@unitedscholarsnetwork.org`
- **From Name**: `United Scholars Network`
- All workflows use the same SMTP credential.

## Testing
Use n8n's **Test Webhook** feature to send sample payloads before connecting to the live platform. Sample payloads for each workflow are documented in the specification.
