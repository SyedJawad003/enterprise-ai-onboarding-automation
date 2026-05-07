# Candidate Submission Template

## Candidate Information
- Full Name: Syed Jawad
- Email: syedjawaduddin100@gmail.com
- LinkedIn or Portfolio: syedjawadportfolio.netlify.app
- Submission Date: 2026-05-07

---

## Overview

This submission presents a complete AI-powered onboarding automation system designed for enterprise environments. The solution covers both the architecture design (Task 1) and a working prototype scaffold (Task 2) implemented as an exportable n8n workflow.

The system automates the full onboarding lifecycle — from new hire form submission through 90-day milestone follow-ups — using GPT-4o for data extraction, validation, personalized plan generation, and communication drafting. The workflow is orchestrated in n8n and integrates with Airtable, Gmail, Slack, and the OpenAI API.

---

## Task 1: AI-Powered Automation Design

### Workflow Logic

The onboarding automation follows a seven-step process:

1. **Intake** — New hire submits a form (Google Forms / Typeform / HRIS). A webhook fires into n8n with the submission payload.
2. **Validation** — A code node checks for required fields. Missing data triggers an automated email requesting completion and halts the workflow.
3. **AI Extraction and Normalization** — Raw intake data is sent to GPT-4o, which extracts structured fields, normalizes formats (dates, casing, department names), and flags records requiring manual review.
4. **Record Creation** — A clean onboarding record is created in Airtable with status set to `In Progress`.
5. **Plan and Task Generation** — GPT-4o generates a personalized 30-day onboarding plan and a full task list routed to HR, IT, Compliance, and the hiring manager.
6. **Communications** — GPT-4o drafts a personalized welcome email and manager handoff brief. Both are sent automatically. A Slack notification is posted to the HR channel.
7. **Milestone Tracking** — The Airtable record is updated with plan details and task count. Scheduled check-ins are triggered at Day 1, Week 1, Day 30, and Day 90.

### Where AI Is Used

**Classification:** AI determines which onboarding tasks apply based on role, employment type, location, and department — replacing static rule tables with dynamic classification.

**Document Processing:** GPT-4o extracts structured fields from uploaded documents (ID, offer letters, policy forms) and flags incomplete or unsigned documents for HR review.

**Workflow Decision Logic:** AI validates intake records and returns a `review_required` flag that routes records to manual review or automated processing. The IF node acts on this flag.

**Automatic Drafting:** AI generates all external communications — welcome emails, manager briefs, and milestone check-ins — using employee profile context. No manual template editing required.

**Recommendations and Personalization:** AI generates a 30-day onboarding plan with role-specific training recommendations, key contacts, first-week priorities, and first-month goals.

### Prompt Engineering

Nine prompts cover all AI steps in the workflow. Full prompt library is documented in `starter/prompts/prompts.md`.

Key design principles applied:

- **Role instruction first** — Each prompt opens with a clear role definition to anchor model behaviour
- **Structured output** — All prompts request JSON with defined field names, making outputs directly consumable by downstream automation nodes
- **Null-safe fallbacks** — Every prompt instructs the model to return `null` for missing data rather than hallucinate values
- **No markdown** — Prompts explicitly forbid code fences and markdown formatting to prevent JSON parsing failures
- **Temperature tuning** — Extraction prompts use temperature 0 for determinism; generation prompts use 0.4–0.6 for natural language quality

### Data Flow and Integrations

```
Google Forms / Typeform / HRIS
  ↓ Webhook
n8n Orchestration
  ↓
Code Node (validate required fields)
  ↓
OpenAI GPT-4o (extraction and normalization)
  ↓
IF Node → route on review_required flag
  ↓ (clean path)
Airtable (create onboarding record)
  ↓
OpenAI GPT-4o (generate plan and tasks)
  ↓
OpenAI GPT-4o (draft communications)
  ↓ (parallel)
Gmail (welcome email) + Gmail (manager brief) + Slack (HR notification)
  ↓
Airtable (update status and store plan)
```

| System | Role |
|---|---|
| Google Forms / Typeform | New hire intake |
| n8n | Workflow orchestration |
| OpenAI GPT-4o | Extraction, classification, generation |
| Airtable | Onboarding record and status tracking |
| Gmail | Welcome email, manager brief, missing info requests |
| Slack | HR channel notifications |
| Google Calendar | Orientation scheduling (planned extension) |
| Jira / ClickUp | Task creation and assignment (planned extension) |

### Business Impact

**Efficiency:** Estimated 70% reduction in manual coordination time. IT provisioning triggers automatically 5 days before start. No data re-entry between systems.

**Accuracy:** AI validation catches missing fields and document issues before Day 1. Normalized records eliminate downstream formatting errors.

**Personalization:** Every new hire receives a role-specific plan. Manager communications include contextual details about the specific hire.

**HR Time Savings:** HR focus shifts from logistics coordination to exception handling. Milestone tracking is fully automated.

**New Hire Experience:** Structured welcome communication from Day 0 creates a professional first impression. Timely check-ins surface issues early.

---

## Task 2: Implementation Demo

### Demo Type

n8n workflow export JSON

### Files Included

- `starter/workflows/onboarding-workflow.json` — Full n8n workflow export with 17 nodes covering the complete automation flow
- `starter/prompts/prompts.md` — Complete prompt engineering library (9 prompts with design rationale)
- `starter/design-solution.md` — Full architecture and design documentation

### Flow of Data

1. Webhook node receives POST payload from new hire form
2. Code node validates required fields; flags missing data
3. HTTP Request node sends raw data to OpenAI GPT-4o for extraction and normalization
4. Code node parses JSON response; IF node routes based on `review_required` flag
5. **Happy path:** Airtable creates record → OpenAI generates plan and tasks → OpenAI drafts communications → parallel Gmail and Slack nodes fire → Airtable updates status → Webhook responds with success
6. **Incomplete path:** Gmail sends missing-info request to candidate → Webhook responds with field list

### Pain Points Solved

- **Eliminates manual data re-entry** — intake data flows directly to all systems without HR copy-pasting
- **Removes dependency on individual HR workload** — communications and task creation happen automatically
- **Catches issues before Day 1** — validation and document checks happen at submission time, not during orientation
- **Standardizes the experience** — every new hire gets the same quality of onboarding regardless of who handles the process

---

## Assumptions

1. The automation platform is n8n. The workflow JSON is importable directly and the logic is portable to Zapier or Make.
2. OpenAI API credentials are configured in n8n as an HTTP Header Auth credential with Bearer token format.
3. The Airtable base ID is stored as an n8n environment variable `AIRTABLE_BASE_ID`.
4. Gmail and Slack credentials are configured as OAuth2 connections in n8n.
5. Mock data and placeholder integrations are used where production credentials are not available.
6. The workflow covers core orchestration logic. Production deployment would add dedicated error-handling workflows and full HRIS integration.

---

## Setup Instructions

### To import and run the n8n workflow:

1. Open your n8n instance (or start one with `npx n8n`)
2. Go to **Workflows → Import from File**
3. Upload `starter/workflows/onboarding-workflow.json`
4. Configure credentials in n8n:
   - HTTP Header Auth: **OpenAI API Key** (Bearer token)
   - Gmail OAuth2
   - Slack OAuth2
   - Airtable API Token
5. Set environment variable `AIRTABLE_BASE_ID` to your Airtable base ID
6. Activate the workflow
7. Test by sending a POST request to the webhook URL with the sample payload below

### Sample test payload:

```json
{
  "full_name": "Sarah Johnson",
  "personal_email": "sarah.johnson@gmail.com",
  "job_title": "Product Manager",
  "department": "Product",
  "location": "London, UK",
  "manager_name": "David Lee",
  "manager_email": "david.lee@company.com",
  "employment_type": "full-time",
  "start_date": "2024-06-03",
  "required_systems": ["Jira", "Confluence", "Slack", "Google Workspace"]
}
```

---

## Optional Notes

The n8n workflow is structured to be readable as a standalone architecture reference even without running it — each node includes a `notes` field describing its purpose. The prompt library in `prompts.md` is directly copy-paste usable in any LLM-integrated automation tool.

The highest-value production extension would be adding a dedicated error-handling workflow in n8n and automating the Day 1, Week 1, and 30-day milestone check-ins using the start date from the Airtable record.

