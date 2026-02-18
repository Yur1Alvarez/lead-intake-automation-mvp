# Test Plan and Acceptance Checklist

## Test Data

Use sanitized leads only:

- Lead A: normal estimate
- Lead B: urgent wording
- Lead C: duplicate of Lead A within 24h

## Core Tests

1. New lead end-to-end
- Submit form once
- Expected:
  - row created in `Leads`
  - AI fields populated
  - immediate email sent
  - status `responded`

2. Duplicate suppression
- Submit same email/phone within 24h
- Expected:
  - second row still logged
  - `is_duplicate_24h=true`
  - no second immediate email

3. Urgent alert
- Submit message with urgent wording
- Expected:
  - AI urgency `high`
  - internal alert email sent

4. AI failure fallback
- Temporarily break AI step or return invalid enum
- Expected:
  - fallback values applied
  - row persists
  - `error_flag=true` when applicable

5. Follow-up stage progression
- For a non-reply lead:
  - at due time stage 0 -> send #1
  - stage 1 -> send #2
  - stage 2 -> send #3 and close

6. Stop condition
- Mark lead `booked` before next run
- Expected:
  - no more follow-ups

7. Error monitoring
- Force one Zap step failure
- Expected:
  - Zapier Manager error alert sent

## Acceptance Criteria

- End-to-end intake + immediate response works in production
- Dedupe behavior confirmed in real runs
- Follow-up chain completes for at least one test lead
- Error alerting is active
- README + demo assets are ready for portfolio sharing
