
---
name: gmail-lead-scanner
description: >
  Scans Gmail inbox for B2B sales leads over the past 3 days, extracts contact details,
  scores them by urgency, exports to Excel, and uploads to HubSpot CRM. Use this skill
  whenever the user wants to find leads in their inbox, process sales emails, scan Gmail
  for prospects, export contacts to Excel, sync leads to HubSpot, or run their regular
  lead pipeline scan. Also triggers when the user says things like "check my inbox for
  leads", "run the lead scan", "find demo requests", "who signed up recently", or
  "update HubSpot with new contacts". Always use this skill for any Gmail → leads →
  HubSpot workflow, even if the user only mentions one step of the process.
compatibility:
  required_mcp: Gmail (gmailmcp.googleapis.com)
  required_tools: bash_tool, create_file, present_files
  required_secrets: HUBSPOT_API_KEY
---

# Gmail Lead Scanner Skill

Scans Gmail for B2B sales leads over the past 3 days, classifies them by urgency,
extracts contact details, exports to Excel (.xlsx), and pushes contacts to HubSpot CRM.

---

## Overview of Steps

1. **Scan Gmail** — fetch emails from the last 3 days
2. **Classify leads** — detect lead signals, score urgency, filter B2B
3. **Extract contacts** — name, email, company, phone, intent summary
4. **Export to Excel** — structured .xlsx file with all leads
5. **Upload to HubSpot** — create/update contacts via HubSpot API
6. **Report to user** — summary of what was found and actioned

---

## Step 1: Scan Gmail (last 3 days)

Use the Gmail MCP tool to search threads from the past 3 days. Run multiple searches
to cover all lead signal patterns:

```
Search queries to run (one at a time via Gmail MCP):
- "demo request" newer_than:3d
- "book a demo" newer_than:3d
- "request a demo" newer_than:3d
- "new user signup" newer_than:3d
- "interested in pricing" newer_than:3d
- "want to learn more" newer_than:3d
- "how does your product work" newer_than:3d
- "sign up" newer_than:3d
- "get started" newer_than:3d
- "contact sales" newer_than:3d
- "schedule a call" newer_than:3d
- "trial request" newer_than:3d
```

Collect all unique threads. De-duplicate by thread ID.

---

## Step 2: Classify & Filter Leads

For each email thread, apply the following classification logic:

### B2B Filter (must pass ALL of these)
- Sender domain is NOT: gmail.com, yahoo.com, hotmail.com, outlook.com, icloud.com,
  protonmail.com, aol.com, live.com, msn.com, me.com, mac.com
- Email does not appear to be a newsletter, automated marketing blast, or no-reply
- Sender appears to be a real person or a legitimate business CTA notification

### Lead Signal Detection
Classify the email as a lead if it contains ANY of:
- Demo request / book a demo / schedule a demo / request a demo
- New user signup notification (from your own product or a third-party)
- Pricing inquiry / interested in pricing / cost / quote request
- "Want to learn more" / "more information" / "tell me more"
- "How does your product work" / product questions
- Trial request / free trial / get started
- "Contact sales" / "talk to sales" / "speak to someone"
- Schedule a call / book a meeting / calendar invite request
- CRM call-to-action button click notifications

### Urgency Scoring
Assign one of two tiers:

**🔴 Reply Needed Today**
- Explicit demo booking or scheduling request
- Pricing or quote request
- Trial activation (user already signed up and is waiting)
- Any email where the prospect is clearly ready to engage NOW

**🟡 Follow Up Later**
- General interest / curiosity emails
- "Want to learn more" without a specific ask
- Signup notifications (prospect just entered the funnel)
- Ambiguous intent but from a legitimate business domain

**⚪ Flagged for Review** (ambiguous — not clearly signal or noise)
- Emails that partially match lead signals but intent is unclear
- Include these in a separate tab in the Excel export

---

## Step 3: Extract Contact Details

For each classified lead, extract:

| Field | How to extract |
|---|---|
| **Name** | From email "From" header, or signature block |
| **Email Address** | Sender email address |
| **Company Name** | From email domain (strip www/mail prefix), or signature |
| **Phone Number** | From email body/signature if present, else blank |
| **Intent Summary** | 1–2 sentence summary of what the person wants |
| **Urgency Tier** | 🔴 Reply Today / 🟡 Follow Up Later / ⚪ Review |
| **Email Date** | Date the email was received |
| **Email Subject** | Subject line of the original email |

---

## Step 4: Export to Excel

Read the xlsx SKILL.md at `/mnt/skills/public/xlsx/SKILL.md` for exact export instructions.

Structure the Excel file with **3 sheets**:

### Sheet 1: "Hot Leads — Reply Today" 🔴
Columns: Name | Email | Company | Phone | Intent Summary | Email Date | Subject

### Sheet 2: "Warm Leads — Follow Up" 🟡
Columns: Name | Email | Company | Phone | Intent Summary | Email Date | Subject

### Sheet 3: "Flagged for Review" ⚪
Columns: Name | Email | Company | Phone | Intent Summary | Email Date | Subject | Reason Flagged

Save file as: `leads_YYYY-MM-DD.xlsx` (use today's date)

---

## Step 5: Upload to HubSpot

See `references/hubspot.md` for full API details.

For each lead in Sheet 1 and Sheet 2 (not Sheet 3 — those need human review first):

1. Check if contact already exists (search by email)
2. If exists → update with latest info + add note with intent summary
3. If new → create contact with all extracted fields

Set HubSpot contact properties:
- `firstname`, `lastname` (split from Name if possible)
- `email`
- `company`
- `phone`
- `lead_status`: "Reply Today" or "Follow Up Later"
- `hs_lead_status`: "NEW"
- Notes: paste the intent summary as a HubSpot note on the contact

---

## Step 6: Report to User

After completing all steps, give the user a clean summary:

```
📬 Gmail Lead Scan Complete — [DATE]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔴 Reply Today:     X leads
🟡 Follow Up Later: X leads
⚪ Flagged:         X emails need your review
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ X contacts uploaded to HubSpot
📁 Excel exported: leads_YYYY-MM-DD.xlsx
```

Then list each "Reply Today" lead by name + company + one-line intent so the user can
act immediately without opening the file.

---

## Scheduling on a Personal Computer

To run this scan automatically every 3 days at 8am, the user needs to set up a
scheduled task. See `references/scheduling.md` for step-by-step instructions for
both Mac (launchd) and Windows (Task Scheduler).

---

## Error Handling

- **Gmail auth error**: Ask user to reconnect Gmail MCP in Claude settings
- **HubSpot API key missing**: Prompt user to add HUBSPOT_API_KEY to environment,
  or offer to skip HubSpot upload and just produce the Excel file
- **HubSpot rate limit**: Batch uploads in groups of 10 with 1s delay between batches
- **No leads found**: Report "No new leads found in the past 3 days" — do not error out
- **Partial extraction** (e.g. no phone number found): Leave field blank, do not skip lead