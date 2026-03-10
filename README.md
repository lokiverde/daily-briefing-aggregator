# 📋 Daily Briefing Aggregator for AI Assistants

**An n8n workflow that pulls emails and calendar events from multiple accounts into a single API endpoint — designed for AI assistants like Claude Code, ChatGPT, or any automation tool.**

![n8n](https://img.shields.io/badge/n8n-workflow-orange)
![License](https://img.shields.io/badge/license-MIT-blue)

---

## What This Does

This workflow creates a webhook endpoint that, when called, returns:

- 📧 **Emails from the last 24 hours** from multiple Gmail and IMAP accounts
- 📅 **Today's calendar events** from multiple Google Calendars
- 🔮 **Tomorrow's first event** (so you know what's coming)
- ⚡ **Priority flagging** — VIP contacts and keyword-based urgency detection
- 📊 **Summary stats** — unread counts, next meeting countdown, priority alerts

### Example Response

```json
{
  "success": true,
  "data": {
    "generated_at": "2026-03-10T08:00:00.000Z",
    "email": {
      "primary_gmail": [...],
      "work_gmail": [...],
      "additional_imap": [...]
    },
    "priority_emails": [
      {
        "from": "boss@company.com",
        "subject": "Urgent: Need your approval",
        "priority": "VIP",
        "priority_reason": "VIP contact: Your Boss"
      }
    ],
    "calendar": {
      "today": [...],
      "tomorrow_first": { "title": "Client Call", "start": "..." }
    },
    "summary": {
      "total_unread": 12,
      "priority_count": 3,
      "meetings_today": 4,
      "next_meeting_in_minutes": 45
    }
  }
}
```

---

## Prerequisites

- **n8n instance** (self-hosted or n8n Cloud)
- **Google account(s)** with Gmail and Calendar access
- **IMAP credentials** (optional, for non-Gmail accounts)
- Basic familiarity with n8n

---

## Installation

### Step 1: Import the Workflow

1. Download `workflow.json` from this repository
2. Open your n8n instance
3. Go to **Workflows** → **Import from File**
4. Select the downloaded JSON file
5. Click **Import**

The workflow will appear with all nodes disconnected from credentials (this is intentional).

### Step 2: Set Up Google OAuth Credentials

You'll need to create OAuth2 credentials for Gmail and Google Calendar.

#### Create a Google Cloud Project (if you don't have one)

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the **Gmail API** and **Google Calendar API**
4. Go to **APIs & Services** → **Credentials**
5. Click **Create Credentials** → **OAuth client ID**
6. Select **Web application**
7. Add your n8n OAuth callback URL:
   - For n8n Cloud: `https://app.n8n.cloud/rest/oauth2-credential/callback`
   - For self-hosted: `https://your-n8n-domain.com/rest/oauth2-credential/callback`
8. Save the **Client ID** and **Client Secret**

#### Add Gmail Credentials in n8n

1. In n8n, go to **Credentials** → **New**
2. Search for **Gmail OAuth2**
3. Enter your Client ID and Client Secret
4. Click **Sign in with Google** and authorize access
5. Name it something recognizable (e.g., "Gmail - Personal")
6. Save

Repeat for each Gmail account you want to include.

#### Add Google Calendar Credentials in n8n

1. In n8n, go to **Credentials** → **New**
2. Search for **Google Calendar OAuth2**
3. Enter the same Client ID and Client Secret
4. Authorize access
5. Name it (e.g., "Google Calendar - Personal")
6. Save

### Step 3: Set Up IMAP Credentials (Optional)

If you have non-Gmail email accounts (POP3/IMAP):

1. Go to **Credentials** → **New**
2. Search for **IMAP**
3. Enter:
   - **Host**: Your mail server (e.g., `mail.yourdomain.com`)
   - **Port**: Usually 993 for SSL
   - **User**: Your email address
   - **Password**: Your password or app-specific password
4. Save

> **Note**: The IMAP node uses a community node (`n8n-nodes-imap-ai`). Install it via **Settings** → **Community Nodes** → **Install** → search for `n8n-nodes-imap-ai`.

### Step 4: Connect Credentials to Nodes

1. Open the imported workflow
2. Click on each email/calendar node
3. In the **Credential** dropdown, select the appropriate credential you created
4. For calendar nodes, also select the specific calendar from the dropdown

**Nodes to configure:**
- `Gmail — Primary` → Your main Gmail credential
- `Gmail — Work` → Your work Gmail credential
- `IMAP — Additional Email` → Your IMAP credential
- `Calendar — Primary Today` → Your main calendar credential + select calendar
- `Calendar — Work Today` → Your work calendar credential + select calendar
- `Calendar — Tomorrow First` → Same as Primary calendar

### Step 5: Customize the Code Node

The `Build Briefing Payload` node contains customizable settings.

Double-click it and edit the JavaScript:

```javascript
// === CUSTOMIZE: VIP Contacts ===
// Emails from these addresses are flagged as VIP priority
const VIP_EMAILS = [
  'boss@company.com',
  'important-client@example.com',
  'spouse@family.com'
];

const VIP_NAMES = {
  'boss@company.com': 'Your Boss',
  'important-client@example.com': 'Important Client',
  'spouse@family.com': 'Spouse'
};

// === CUSTOMIZE: High Priority Keywords ===
// Emails containing these words get flagged HIGH priority
const HIGH_PRIORITY_KEYWORDS = [
  'urgent', 'ASAP', 'time-sensitive', 'deadline', 'expires',
  'expiring', 'immediately', 'action required', 'signature needed',
  'sign by', 'due by', 'past due', 'emergency', 'critical'
];

// === CUSTOMIZE: Medium Priority Keywords ===
const MEDIUM_PRIORITY_KEYWORDS = [
  'important', 'review', 'follow up', 'reminder', 'meeting',
  'schedule', 'confirm', 'approval', 'request', 'update'
];
```

#### Industry-Specific Keyword Examples

**Real Estate:**
```javascript
const HIGH_PRIORITY_KEYWORDS = [
  'escrow', 'closing', 'contingency', 'inspection', 'appraisal',
  'title', 'amendment', 'addendum', 'counteroffer', 'extension',
  'wire instructions', 'earnest money', 'signature needed'
];
```

**Sales:**
```javascript
const HIGH_PRIORITY_KEYWORDS = [
  'contract', 'proposal', 'quote', 'decision', 'budget',
  'renewal', 'cancel', 'competitor', 'demo', 'pilot',
  'procurement', 'legal review', 'signature'
];
```

**Engineering:**
```javascript
const HIGH_PRIORITY_KEYWORDS = [
  'outage', 'incident', 'P0', 'P1', 'deploy', 'rollback',
  'security', 'vulnerability', 'breach', 'downtime',
  'production', 'hotfix', 'critical bug'
];
```

### Step 6: Activate and Test

1. Click **Save** to save the workflow
2. Toggle **Active** to ON (top right)
3. Click on the **Webhook Trigger** node
4. Copy the **Production URL**
5. Open the URL in your browser or call it with curl:

```bash
curl https://your-n8n.com/webhook/your-briefing-endpoint
```

You should see a JSON response with your emails and calendar events.

---

## Adjusting for Your Setup

### Adding More Email Accounts

1. **Duplicate a Gmail node**: Right-click → Duplicate
2. **Rename it** (e.g., "Gmail — Side Project")
3. **Connect your credential**
4. **Create a Tag node**: Duplicate an existing Tag node, change the `_source` value to something unique (e.g., `gmail_sideproject`)
5. **Connect it to the merge node**: Add another input to "Wait For All Sources" (click the node, increase "Number of Inputs")
6. **Update the Code node**: Add processing for your new source tag

### Adding More Calendars

Same process as email:
1. Duplicate a Calendar node
2. Connect your credential and select the calendar
3. Create a Tag node with unique `_source` value
4. Connect to the merge node
5. Update the Code node to process the new source

### Removing Email Accounts

If you only have one email account:
1. Delete the extra Gmail/IMAP nodes and their Tag nodes
2. Reduce inputs on the merge node
3. Update the Code node to remove references to deleted sources

---

## Connecting to AI Assistants

### Claude Code / Clawdbot

Create a skill or cron job that calls your webhook:

```bash
curl -s https://your-n8n.com/webhook/your-briefing-endpoint | jq
```

Or configure your AI to call it automatically each morning and summarize the results.

### ChatGPT / Custom GPT

Use the webhook URL as an Action in your Custom GPT:

```yaml
openapi: 3.0.0
info:
  title: Daily Briefing API
  version: 1.0.0
servers:
  - url: https://your-n8n.com
paths:
  /webhook/your-briefing-endpoint:
    get:
      operationId: getDailyBriefing
      summary: Get daily email and calendar briefing
      responses:
        '200':
          description: Briefing data
```

### Zapier / Make / Other Automation

Simply call the webhook URL as an HTTP request in your automation.

---

## Scheduled Execution (Optional)

Instead of calling the webhook on-demand, you can run it on a schedule:

1. Add a **Schedule Trigger** node alongside the Webhook Trigger
2. Set it to run at your preferred time (e.g., 7:00 AM daily)
3. Add a **Google Sheets** or **Notion** node at the end to save results
4. Your AI can then read from the spreadsheet instead of calling the webhook

---

## Troubleshooting

### "Credential not found" error
- Re-create the credential in n8n
- Make sure you've authorized the correct Google account

### Empty results from Gmail
- Check that OAuth has the `gmail.readonly` scope
- Verify the filter query (`newer_than:1d` = last 24 hours)
- Make sure there ARE emails in the last 24 hours

### Calendar not showing events
- Ensure you've selected the correct calendar in the node
- Check that the OAuth credential has `calendar.readonly` scope
- Verify there are events scheduled for today

### IMAP connection failed
- Verify server/port settings (usually port 993 for SSL)
- Check if your email provider requires app-specific passwords
- Make sure the `n8n-nodes-imap-ai` community node is installed

### Webhook returns error
- Check the execution log in n8n for specific errors
- Verify all credential connections are valid
- Test each node individually using "Execute Node"

---

## Security Notes

- **Credentials are stored in n8n**, not in this workflow file
- The workflow file contains no sensitive information
- Your webhook URL should be treated as sensitive — anyone with it can see your emails
- Consider adding authentication to your webhook (n8n supports basic auth, header auth, etc.)

---

## License

MIT License — use it however you want.

---

## Credits

Created by [Tony Self](https://techtony.co) — AI automation for business operators.

Part of **The Invisible Department** — building AI systems that work while you sleep.

---

## Questions?

- 🐦 Twitter: [@anthonyself](https://twitter.com/anthonyself)
- 💼 LinkedIn: [anthonyself](https://linkedin.com/in/anthonyself)
- 🌐 Website: [techtony.co](https://techtony.co)
