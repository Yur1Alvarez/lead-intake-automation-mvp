# AI Triage Prompt (Zapier OpenAI Step)

Use this in your OpenAI step and require JSON output.

## System / Instruction

Return ONLY valid JSON. Do not include markdown, prose, or code fences.

## User

Lead:
Name: {{name}}
Email: {{email}}
Phone: {{phone}}
Service: {{service}}
Message: {{message}}
Location: {{city}}, {{zip}}

Extract and classify. Output JSON with:
- intent: one of ["estimate","urgent","question","spam"]
- service_type: short string
- urgency: one of ["low","medium","high"]
- summary: <= 200 chars
- next_action: one of ["call_now","text_now","send_quote","request_more_info","ignore"]

## Fallback Rules (Formatter step after AI)

If value is outside enum, set defaults:

- intent -> `question`
- urgency -> `medium`
- next_action -> `request_more_info`
- service_type -> `unknown`
- summary -> `Lead received. Needs manual review.`
