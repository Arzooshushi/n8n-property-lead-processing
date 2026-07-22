# n8n-property-lead-processing
# n8n Property Lead Processing Workflow

A rule-based n8n workflow that receives a batch of property leads, standardizes raw customer data, validates required fields, assigns lead priority, prepares CRM-ready JSON payloads, and creates structured logs for rejected leads.

## Project Overview

Property businesses often receive leads from websites, advertising platforms, property portals, and external CRMs. These leads may contain inconsistent formatting, missing fields, incorrect data types, or incomplete customer information.

This workflow demonstrates how n8n can automatically:

- Process multiple leads from a single batch
- Clean and standardize customer data
- Convert raw strings into correct data types
- Validate required fields
- Route high-value and normal leads
- Generate a nested CRM-ready payload
- Create structured logs for invalid leads

## Workflow Architecture

```text
Receive Property Leads
        ↓
Mock Property Portal Data
        ↓
Split Lead Batch
        ↓
Clean and Standardize Leads
        ↓
Validate Required Fields
       / \
  Invalid  Valid
     ↓       ↓
Prepare     Check High-Value Lead
Invalid       /             \
Lead      High Value       Normal
     ↓          ↓              ↓
Build       Mark High       Mark Normal
Invalid      Value             Lead
Lead Log       \              /
                  Merge
                    ↓
             Build CRM Payload
```

## Workflow Screenshot

![Workflow Overview](screenshots/01-workflow-overview)

## Core Features

### Batch Processing

The input contains an array of multiple lead objects. The Split Out node converts the array into separate n8n items so every lead can be processed individually.

### Data Cleaning and Standardization

The workflow performs transformations including:

- Removing extra spaces from names
- Converting email addresses to lowercase
- Removing formatting characters from phone numbers
- Converting budget values from strings to numbers
- Converting verified values from strings to booleans
- Replacing missing company names with `Individual`
- Handling empty service arrays safely

![Cleaned Lead Output](screenshots/02-cleaned-lead-output.png)

### Required-Field Validation

A lead is accepted only when:

- Name is available
- Phone number is available
- Budget is a valid number

Invalid records are routed to a separate rejection path.

![Validation Routing](screenshots/03-validation-routing.png)

### Lead Priority Classification

The workflow uses a fixed business rule:

```text
Budget >= 3,000,000 → High Value
Budget < 3,000,000  → Normal
```

This is a deterministic rule-based decision and does not require AI.

### CRM Payload Generation

Valid leads are converted from flat records into structured nested JSON containing:

- Contact information
- Qualification details
- Property preferences
- Business metadata

![CRM Payload](screenshots/04-crm-payload.png)

### Invalid Lead Logging

Rejected leads receive:

- Processing status
- Specific rejection reason
- Lead source
- Execution timestamp

![Invalid Lead Log](screenshots/05-invalid-lead-log.png)

## Nodes Used

- Webhook
- Edit Fields
- Split Out
- IF
- Merge

## JSON Concepts Demonstrated

- Strings
- Numbers
- Booleans
- Null values
- Arrays
- Objects
- Arrays of objects
- Nested JSON paths
- Array indexing
- Type conversion
- Default values
- Flat-to-nested JSON transformation
- Multiple n8n items

## Test Scenarios

The workflow was tested with:

| Scenario | Expected Result |
|---|---|
| Complete high-budget lead | Accepted as High Value |
| Complete lower-budget lead | Accepted as Normal |
| Missing phone number | Rejected |
| Missing name | Rejected |
| Invalid budget | Rejected |
| Budget equal to 3,000,000 | Accepted as High Value |
| Empty services array | Processed with `No service` |
| Missing company | Converted to `Individual` |

## Sample Files

- `sample-input.json` — Example raw batch input
- `sample-valid-output.json` — CRM-ready valid records
- `sample-invalid-output.json` — Structured rejected-lead record
- `property-lead-processing-workflow.json` — Importable n8n workflow

## How to Import the Workflow

1. Download `property-lead-processing-workflow.json`.
2. Open n8n.
3. Create a new workflow.
4. Open the workflow menu.
5. Select **Import from File**.
6. Select the downloaded workflow JSON.
7. Review node settings before executing.

## Demo Input

This portfolio version uses simulated property portal data so that the workflow can be imported and tested without external accounts or credentials.

## Production Adaptation

In a real client environment, the mock-data node would be replaced by data received through a POST webhook.

Expected webhook body:

```json
{
  "source": "Property Portal",
  "campaign_id": "CMP-2026-07",
  "leads": [
    {
      "lead_id": "L-001",
      "name": "Ali Khan",
      "email": "ali@example.com",
      "phone": "03001234567",
      "budget": "5000000",
      "verified": "true",
      "company": null,
      "services": ["Buy"],
      "address": {
        "city": "Lahore",
        "country": "Pakistan"
      }
    }
  ]
}
```

The production input-mapping expressions would be:

```javascript
{{ $json.body.source }}
{{ $json.body.campaign_id }}
{{ $json.body.leads }}
```

After input normalization, the remaining cleaning, validation, routing, and payload-generation logic remains reusable.

## Workflow Type

```text
Standard Rule-Based Automation
```

AI is not required because validation and priority assignment use explicit and deterministic business rules.

## Future Improvements

- Save valid leads to a CRM
- Store invalid records in a database
- Add duplicate lead detection
- Send sales-team notifications
- Return a webhook processing summary
- Add retry and technical-error handling
- Add authentication to the webhook
- Replace simulated data with a live property portal integration

## Security Notes

This repository contains only dummy data.

It does not contain:

- API keys
- Access tokens
- Passwords
- Real customer information
- Private webhook URLs
- Client credentials

## Author

Created as part of practical n8n and AI automation engineering training.
