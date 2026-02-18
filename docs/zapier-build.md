# Zapier Build Guide (Formspree + Google Sheets + Gmail)

This is the exact MVP setup for:

- Zap 1: Intake + AI + dedupe + immediate response
- Zap 2: Follow-up chain (+2h, +24h, +72h)
- Zap 3: Booking stop signal
- Zap 4: Error alerts

## Prerequisites

- Formspree form is live (already configured in website)
- Google Sheet with `Leads` tab and required columns
- Gmail connected in Zapier
- Storage by Zapier enabled
- OpenAI step enabled in Zapier

## Shared Variables

Create these with Formatter or Code steps in Zap 1:

- `created_at`: current timestamp ISO
- `service`: from `trade`
- `dedupe_key`: lowercase email; fallback to phone digits only
- `lead_id`: `LEAD-YYYYMMDD-HHMMSS-####`

### Suggested lead_id recipe

- Date token: `YYYYMMDD-HHMMSS`
- Random token: 4-digit random number
- Final: `LEAD-{{date_token}}-{{rand4}}`

## Zap 1 - Intake + AI + Immediate Reply

1. Trigger: `Formspree -> New Form Submission`
2. Formatter: normalize email lowercase and phone digits
3. Formatter: build `dedupe_key`
4. Storage by Zapier: `Get Value`
   - Key: `last_reply::{{dedupe_key}}`
5. Filter path split:
   - Path A (not duplicate): no timestamp found OR older than 24h
   - Path B (duplicate): timestamp exists and within 24h
6. AI step (OpenAI)
   - Use prompt from `ai-triage-prompt.md`
7. Formatter / validation step
   - Enforce enums and fallback defaults
8. Google Sheets: `Create Spreadsheet Row`
   - Write all lead + AI fields
   - Status:
     - Path A: `responded`
     - Path B: `duplicate_24h`
9. Path A only: Gmail send immediate response
10. Path A only: Storage by Zapier `Set Value`
   - Key: `last_reply::{{dedupe_key}}`
   - Value: current timestamp
11. Path A only: update row fields
   - `first_response_at=now`
   - `followup_stage=0`
   - `next_followup_at=now+2h`
12. High urgency alert path:
   - Filter `ai_urgency == high`
   - Send internal alert email with summary and lead details

## Zap 2 - Follow-up Chain (Delay-based)

Trigger options:

- Recommended: `Schedule by Zapier` every 15 min and process rows due for follow-up
- Alternate: `Sub-Zap/child Zap` triggered from Zap 1 payload

For simple MVP, use row-driven schedule:

1. Trigger: `Schedule by Zapier` every 15 min
2. Google Sheets: find rows where
   - `status=responded`
   - `next_followup_at <= now`
3. Loop each due row:
   - If `followup_stage=0` send follow-up #1, set stage 1, next+22h
   - If `followup_stage=1` send follow-up #2, set stage 2, next+48h
   - If `followup_stage=2` send final follow-up, set stage 3 and status `closed_no_reply`
4. Before each send, skip if status is `booked` or `closed_no_reply`

## Zap 3 - Booking Stop Signal

Trigger:

- Calendly invitee created OR Google appointment booked

Steps:

1. Find row by email (fallback phone)
2. Update status to `booked`
3. Clear `next_followup_at`

## Zap 4 - Reliability / Error Alert

1. Trigger: `Zapier Manager -> New Zap Error`
2. Gmail send internal alert with:
   - Zap name
   - step name
   - run ID
   - error message
   - timestamp
3. Optional: append to `Errors` sheet

## Dedupe Logic (24h)

- Key: `last_reply::<dedupe_key>`
- If found and timestamp < 24h old:
  - still log row
  - mark `is_duplicate_24h=true`
  - skip immediate reply

## Notes

- Email-first MVP intentionally avoids SMS compliance overhead
- For job applications, reliability proof matters more than channel count
