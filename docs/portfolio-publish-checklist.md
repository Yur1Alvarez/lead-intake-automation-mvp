# Portfolio Publish Checklist

Use this checklist before pushing public updates.

## Security

- [ ] No live webhook secrets committed (Zapier hook URLs, API keys, tokens)
- [ ] No private credentials in docs, code, or screenshots
- [ ] No real customer PII in sample data
- [ ] Contact emails are business-safe/public-facing only

## Workflow Evidence

- [ ] New lead run: Path A (`is_duplicate_24h=false`) verified
- [ ] Duplicate run: Path B (`is_duplicate_24h=true`) verified
- [ ] AI success run: `error_flag=false` verified
- [ ] AI fallback run: row still created with `error_flag=true` verified
- [ ] Duplicate path sends no immediate customer auto-reply

## Documentation

- [ ] `README.md` reflects current architecture and routing logic
- [ ] `docs/zap1-final-workflow.md` matches live Zap behavior
- [ ] `docs/test-plan.md` includes tested scenarios and outcomes
- [ ] `docs/demo-script.md` matches current implementation

## Repo Readiness

- [ ] Screenshots added under `docs/screenshots/` (sanitized)
- [ ] Commit messages clearly explain changes
- [ ] Push to GitHub branch completed

## Recommended Commit Sequence

```bash
git add README.md docs/zap1-final-workflow.md docs/portfolio-publish-checklist.md pages/index.html
git commit -m "docs: finalize portfolio workflow and sanitize public form endpoint"
git push
```
