# BedTrack

Real-time bed availability tracking service (PHI scope).

## Layout

- `current/` — the live, deployed state. Point the governance platform's path prefix here for normal intent tracking.
  - `current/terraform/` — infrastructure as code (Lambda, SQS, DynamoDB, SNS, S3, KMS, IAM)
  - `current/docs/requirements.md` — the requirements doc matching what's actually deployed
- `alt/` — a pending requirements update, not yet promoted.
  - `alt/docs/requirements.md` — proposed next version of the requirements doc. When ready, copy this over `current/docs/requirements.md` and commit; `current/terraform/` stays as-is until it's updated to match, which is the intended drift the platform should flag.
