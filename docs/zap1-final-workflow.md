# Zap 1 Final Workflow (Verified)

This document is the final, portfolio-ready description of your live Zap 1 workflow.

It reflects the behavior verified in your recent runs:
- AI fields parsed successfully (`error_flag=false` on normal runs)
- Service field populated in Sheets (`COL$H`)
- Duplicate detection working (`is_duplicate_24h=true/false`)
- Path routing by duplicate flag (not urgency)

## Scope

Zap 1 handles:
- intake from webhook
- normalization
- 24-hour dedupe
- AI triage
- row logging in Google Sheets
- customer response for new leads
- internal urgency alerts

## Data Contract

Inbound fields from website/webhook:
- `source`
- `name`
- `email`
- `phone`
- `city`
- `zip`
- `trade` (website field)
- `service` (optional fallback for manual/API tests)
- `business` (optional fallback)
- `message`

Normalized fields used by Zap:
- `email_norm` (lowercase)
- `phone_norm` (digits only)
- `service` (`trade` -> `service` -> `business`)
- `lead_id`
- `dedupe_key` (`email_norm` -> `phone_norm` -> `anon_<lead_id>`)
- `now_unix`

## Step-by-Step (Live Logic)

1. **Step 1 - Catch Hook (Webhooks by Zapier)**
   - Receives lead payload.

2. **Step 2 - Normalize (Code by Zapier)**
   - Cleans and standardizes input.
   - Builds `lead_id`, `dedupe_key`, and timestamp fields.

3. **Step 3 - Storage Get (Storage by Zapier)**
   - Key: `last_reply::{{dedupe_key}}`
   - Returns prior reply timestamp when present.

4. **Step 4 - Dedupe Check (Code by Zapier)**
   - Uses `now_unix` and Step 3 `value`.
   - Outputs:
     - `is_duplicate_24h = "true"` if `< 86400s`
     - `is_duplicate_24h = "false"` otherwise

5. **Step 5 - AI Classification (Gemini)**
   - Returns JSON fields: `intent`, `service_type`, `urgency`, `summary`, `next_action`.

6. **Step 6 - AI Cleanup (Code by Zapier)**
   - Reads `gemini_text` and `gemini_finish_reason` from Step 5 mapped text fields.
   - Removes markdown code fences.
   - Emits either:
     - cleaned text for parsing, or
     - safe fallback with error details.

7. **Step 7 - AI Validation (Code by Zapier)**
   - Parses cleaned JSON.
   - Enforces enum safety:
     - intent: `estimate|urgent|question|spam`
     - urgency: `low|medium|high`
     - next_action: `call_now|text_now|send_quote|request_more_info|ignore`
   - Outputs final AI fields + `error_flag` / `error_message`.

8. **Step 8 - Create Row (Google Sheets)**
   - Writes complete lead record:
     - lead details, AI fields, dedupe metadata, error fields.
   - Important mapping:
     - `COL$H` = normalized `service` (fixed)

9. **Step 9 - Urgency Extract (Code by Zapier)**
   - Derives `urgency_level` from validated AI urgency.

10. **Step 10 - Paths by Duplicate Flag**
   - **Path A**: `is_duplicate_24h == false` (new lead)
   - **Path B**: `is_duplicate_24h == true` (duplicate lead)

## Path A (New Lead)

- Send customer auto-reply email.
- Compute response timestamps.
- Update row:
  - `status=responded`
  - `first_response_at`
  - `followup_stage=0`
  - `next_followup_at=+2h`
- Storage Set:
  - key `last_reply::{{dedupe_key}}`
  - value current unix time
- Optional internal alert:
  - only if `ai_urgency == high`

## Path B (Duplicate Lead)

- Update row first:
  - `status=duplicate_24h`
- Optional internal alert after update:
  - only if `ai_urgency == high`
- No customer-facing immediate reply.

## Verified Outcomes

From recent run evidence:
- New lead row logged with complete fields and `status=responded`
- Follow-up submit from same email inside 24h logged with `status=duplicate_24h`
- AI fields populated correctly (`urgent`, `high`, service type present)
- `error_flag=false` on successful AI parse

## Final Acceptance Checklist

- [x] `COL$H` service is populated
- [x] AI parse succeeds (`error_flag=false`) on normal prompt
- [x] Dedupe toggles to true on follow-up within 24h
- [x] Path A handles new leads
- [x] Path B handles duplicates
- [x] Duplicate path does not send customer immediate reply
- [x] High urgency triggers internal alerts only

## Notes for Portfolio Demo

When presenting this project, describe routing as:
- **Primary split**: duplicate vs new (`is_duplicate_24h`)
- **Secondary escalation**: urgency-based internal alerts

This is the most accurate representation of your final implementation.
