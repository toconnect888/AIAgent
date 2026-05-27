# HubSpot API Reference

## Authentication
All requests require the `HUBSPOT_API_KEY` as a Bearer token:
```
Authorization: Bearer 
```

Get your API key: HubSpot → Settings → Integrations → Private Apps → Create App

---

## Search for Existing Contact by Email

```http
POST https://api.hubapi.com/crm/v3/objects/contacts/search
Content-Type: application/json

{
  "filterGroups": [{
    "filters": [{
      "propertyName": "email",
      "operator": "EQ",
      "value": "prospect@company.com"
    }]
  }],
  "properties": ["email", "firstname", "lastname", "company", "phone", "hs_lead_status"]
}
```

Response: if `results` array is non-empty, contact exists. Use `results[0].id` as contactId.

---

## Create New Contact

```http
POST https://api.hubapi.com/crm/v3/objects/contacts
Content-Type: application/json

{
  "properties": {
    "firstname": "Jane",
    "lastname": "Doe",
    "email": "jane@company.com",
    "company": "Acme Corp",
    "phone": "+1-555-0100",
    "lead_status": "Reply Today",
    "hs_lead_status": "NEW",
    "lifecyclestage": "lead"
  }
}
```

---

## Update Existing Contact

```http
PATCH https://api.hubapi.com/crm/v3/objects/contacts/{contactId}
Content-Type: application/json

{
  "properties": {
    "lead_status": "Reply Today",
    "hs_lead_status": "NEW"
  }
}
```

---

## Add a Note to a Contact

```http
POST https://api.hubapi.com/crm/v3/objects/notes
Content-Type: application/json

{
  "properties": {
    "hs_note_body": "Intent: Requested a demo after clicking CTA in email on 2026-05-26.",
    "hs_timestamp": "2026-05-26T08:00:00Z"
  }
}
```

Then associate the note with the contact:

```http
PUT https://api.hubapi.com/crm/v3/objects/notes/{noteId}/associations/contacts/{contactId}/note_to_contact
```

---

## Rate Limits
- Free/Starter: 100 requests per 10 seconds
- Batch in groups of 10 contacts, add a 1 second delay between batches
- On 429 error: wait 10 seconds and retry

---

## Custom Property: lead_status
If `lead_status` doesn't exist in your HubSpot portal, create it first:
```http
POST https://api.hubapi.com/crm/v3/properties/contacts
{
  "name": "lead_status",
  "label": "Lead Status",
  "type": "string",
  "fieldType": "text",
  "groupName": "contactinformation"
}
```
Or simply use the built-in `hs_lead_status` with values "NEW", "OPEN", "IN_PROGRESS".