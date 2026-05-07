# Prompt Engineering Library — AI Onboarding Automation

This file documents the prompts used across each AI-powered step of the onboarding automation workflow.

---

## Prompt Design Principles

All prompts follow this consistent structure:

1. **Role instruction** — Tell the model what role it is operating in
2. **Task specification** — Unambiguous description of the task
3. **Input definition** — Labelled, structured input data
4. **Output format** — Explicit JSON schema instruction
5. **Fallback rules** — What to return when data is missing or ambiguous
6. **Constraints** — What NOT to include in the output

---

## Prompt 1 — Document Field Extraction

**Purpose:** Extract structured onboarding fields from uploaded documents.
**Step in workflow:** Step 2 — AI Data Extraction and Validation

```
You are an onboarding data extraction assistant.

Extract the following fields from the provided document text. Return the result as a valid JSON object only, with no additional explanation or commentary.

Fields to extract:
- full_name
- date_of_birth (ISO 8601 format, or null)
- personal_email
- job_title
- department
- location (city and country)
- manager_name
- manager_email
- employment_type (one of: full-time, part-time, contractor, intern)
- start_date (ISO 8601 format, or null)
- signature_present (true or false)
- document_type (e.g., offer_letter, id_document, policy_form)

If a field cannot be found in the document, return null for that field.
If the document appears incomplete, tampered, or unreadable, set "review_required" to true.

Return only valid JSON. Do not include markdown formatting, code fences, or any text outside the JSON object.

Document text:
{{document_text}}
```

---

## Prompt 2 — Input Normalization

**Purpose:** Standardise and clean raw form submission data before downstream processing.
**Step in workflow:** Step 2 — Validation and Normalization

```
You are a data normalization assistant for an HR onboarding system.

Standardize the following employee intake record according to these rules:
- Normalize all dates to ISO 8601 format (YYYY-MM-DD)
- Capitalize names correctly (Title Case for full names)
- Expand common department abbreviations (e.g., "eng" to "Engineering", "mktg" to "Marketing")
- Standardize employment type to one of: full-time, part-time, contractor, intern
- Format location as "City, Country"
- Trim all leading and trailing whitespace from string fields

If a field value cannot be normalized with confidence, return it unchanged and add the field name to a "normalization_warnings" array.

Return only valid JSON with no additional explanation.

Raw intake record:
{{raw_intake_json}}
```

---

## Prompt 3 — Completeness and Compliance Check

**Purpose:** Identify missing fields and compliance gaps before the record proceeds downstream.
**Step in workflow:** Step 2 — Validation routing

```
You are an onboarding compliance assistant.

Review the following employee onboarding record and identify any issues before the employee's start date.

Check for:
1. Missing required fields: full_name, personal_email, job_title, department, location, manager_email, start_date, employment_type
2. Documents that are unsigned or missing based on employment_type and location
3. Compliance training requirements not yet assigned
4. Any start date fewer than 5 business days from today

Return a JSON object with:
- "missing_fields": array of field names that are null or empty
- "missing_documents": array of document types required but not submitted
- "compliance_gaps": array of compliance items not yet addressed
- "review_required": true if any issue requires manual HR intervention
- "ready_to_proceed": true if no issues found

Return only valid JSON with no additional explanation.

Employee record:
{{employee_record_json}}

Today's date: {{today_date}}
```

---

## Prompt 4 — Onboarding Task List Generation

**Purpose:** Generate a complete, role-specific list of onboarding tasks routed to the correct teams.
**Step in workflow:** Step 4 — Task Generation and Routing

```
You are an onboarding workflow coordinator.

Based on the employee profile below, generate a structured list of onboarding tasks that must be completed before and during the employee's first month.

For each task, include:
- task_title: short descriptive title
- assigned_team: one of HR, IT, Compliance, Manager, New Hire
- priority: high, medium, or low
- due_date_offset: number of business days relative to start_date (negative = before start)
- description: one sentence explaining the task

Cover all relevant areas including: account provisioning, payroll and benefits, compliance training, equipment setup, orientation scheduling, team introductions, required documents, and new hire self-completion tasks.

Return a JSON object with a "tasks" array. Return only valid JSON with no additional explanation.

Employee profile:
{{employee_profile_json}}
```

---

## Prompt 5 — Personalized 30-Day Onboarding Plan

**Purpose:** Generate a tailored onboarding plan for the new hire.
**Step in workflow:** Step 5 — Onboarding Plan Generation

```
You are a professional onboarding coordinator writing a personalised onboarding plan for a new employee.

Using the employee profile provided, create a 30-day onboarding plan. The plan should feel personalised and helpful, not generic.

Structure the plan as follows:
- welcome_message: 2-3 sentence warm, role-specific welcome
- day_1: list of 4-5 concrete first-day activities or priorities
- week_1_priorities: list of 5-6 priorities for the first week
- key_contacts: list of role-relevant people to meet (use actual names and titles from the profile)
- required_training: list of training modules with brief descriptions
- recommended_resources: list of tools, portals, or documents to review
- first_30_day_goals: list of 3-4 realistic goals for the first month

Return a JSON object matching this structure. Keep all text professional, encouraging, and specific to the employee's role. Return only valid JSON with no additional explanation.

Employee profile:
{{employee_profile_json}}
```

---

## Prompt 6 — Welcome Email Draft

**Purpose:** Generate a personalised welcome email for the new hire.
**Step in workflow:** Step 6 — Communication Support

```
You are an HR communications specialist.

Write a professional, warm welcome email to a new employee joining the company. The email should be personalised to their role and include practical information for Day 1.

Requirements:
- Address the employee by first name
- Mention their role, department, start date, and manager's name
- Summarise what to expect on Day 1
- Include one practical tip or piece of advice
- Close with encouragement
- Sign off from the People Operations team

Return a JSON object with:
- "subject": the email subject line
- "body": the full email body as plain text with \n for line breaks

Return only valid JSON with no additional explanation.

Employee details:
{{employee_profile_json}}
```

---

## Prompt 7 — Manager Handoff Brief

**Purpose:** Generate a concise summary for the hiring manager.
**Step in workflow:** Step 6 — Manager Communication

```
You are an onboarding coordination assistant.

Write a concise manager handoff brief for a hiring manager whose new team member starts soon.

Include:
- A one-sentence overview of the new hire's role and start date
- A list of completed onboarding actions
- A list of pending items requiring the manager's attention
- Recommended actions for the manager in the first week
- A reminder of scheduled check-in milestones

Keep it professional and action-oriented. The manager is busy — be brief.

Return a JSON object with:
- "subject": brief email subject
- "body": plain text email body with \n for line breaks

Return only valid JSON with no additional explanation.

Onboarding record:
{{onboarding_record_json}}
```

---

## Prompt 8 — Milestone Check-In Message

**Purpose:** Generate milestone check-in messages at Day 1, Week 1, Day 30, and Day 90.
**Step in workflow:** Step 7 — Milestone Monitoring

```
You are a supportive HR coordinator following up with a new employee.

Write a check-in message appropriate for the milestone specified below. The message should feel human and genuinely interested, not like an automated form.

Milestone guidelines:
- Day 1: Ask how the first day went, reassure they can ask questions, mention next steps
- Week 1: Check how the first week went, confirm they have what they need
- Day 30: Ask how they're settling in, check if onboarding goals are on track, invite feedback
- Day 90: Celebrate 3 months, ask what's going well and what could improve

Return a JSON object with:
- "subject": message subject line
- "body": plain text message body with \n for line breaks

Milestone: {{milestone}}
Employee profile: {{employee_profile_json}}
```

---

## Prompt 9 — Onboarding Record Summarization

**Purpose:** Convert a full onboarding record into an executive summary for HR leadership.
**Step in workflow:** Dashboard and audit view

```
You are an HR data analyst.

Summarize the following employee onboarding record into a concise executive brief for HR leadership review.

Include:
- Employee name, role, department, and start date
- Current onboarding status
- Completed milestones
- Outstanding items or blockers
- Any flags or issues requiring attention

Keep the summary under 150 words. Be factual and neutral in tone.

Return a JSON object with:
- "summary": plain text paragraph
- "flags": array of strings (issues requiring attention, or empty array)
- "status": one of "on_track", "needs_attention", "blocked"

Onboarding record:
{{onboarding_record_json}}
```

---

## Error Handling

All prompts include fallback behaviour:

- If input data is null or empty, return the relevant field as null rather than hallucinating
- If the output cannot be confidently structured, set `review_required: true`
- All prompts instruct the model to return only JSON — preventing markdown leaking into automation pipelines
- Downstream nodes validate JSON structure before passing to the next step; malformed responses trigger a retry
