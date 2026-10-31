# VeriTalent Admin

Private dashboard for the **VeriTalent** public app (the *VeriTalent* repository). It reads the same Supabase database, so everything a candidate does in the public app shows up here.

> This app exposes personal data (CVs, emails). Deploy it privately, keep the admin list short, and never share the service-role key.

## What you get

| Screen | Shows |
| --- | --- |
| **Overview** | total / new / active users, analyses, average score, paying users and conversion, subscriptions, lifetime buyers, estimated MRR, 30-day activity chart |
| **Candidates** | searchable, filterable, sortable list (plan, joined, last seen, analyses, latest role and score) with CSV export |
| **Candidate detail** | profile and consent record, every analysis (scores, matched / missing keywords, the job description used), saved CVs (full text + original file), tailored CVs, activity timeline, your private notes and tags |
| **Insights** | most-targeted roles, most-missed required keywords, most-used resume formats |
| **Activity log** | every admin action: CV views, plan grants / revocations, user deletions |
| **Actions** | grant or revoke Signature manually, delete a user and all their data (type their email to confirm) |

**Accountability:** viewing a CV goes through the server and is written to the audit log. Admin pages and functions require your email to be on the `admins` allow-list; the check happens in the database (row-level security) and again on the server.

## Setup

1. **Use the same Supabase project as the public app.** Run [`supabase/schema.sql`](supabase/schema.sql) there (the public app's copy is identical). It is safe to re-run, and it must be re-run after the public app adds tables (the latest adds `ai_usage`, the Claude credits log).
2. **Make yourself admin** (SQL editor): `insert into public.admins (email) values ('you@example.com');`
3. In Supabase *Authentication > URL Configuration*, add this app's URL (and `http://localhost:5173`) to **Redirect URLs**.
4. Copy [`.env.example`](.env.example) to `.env.local` and fill it in. The `SUPABASE_SERVICE_ROLE_KEY` is server-only: set it in Vercel's environment variables, never in a `VITE_` variable.
5. Install and run:

```bash
npm install
npm run dev          # dashboard only (reads work; the three admin actions need /api)
npm run dev:full     # dashboard + /api functions via the Vercel CLI
```

6. **Deploy to its own Vercel project** (for example `admin.yourdomain.com`). `vercel.json` already sets `noindex` and blocks framing. Consider adding Vercel *Deployment Protection* or Cloudflare Access in front, and turn on two-factor authentication on your Supabase and email accounts.

## Project structure

```
src/
  App.tsx             login gate (email link) + admin check + navigation
  pages/              Overview, Candidates, CandidateDetail, Insights, AuditLog
  ui.tsx, lib.ts      shared components and helpers
  config.ts           prices used for the revenue estimate (keep in sync with Stripe / Polar)
api/admin/
  grant.ts            grant / revoke Signature (audited)
  delete-user.ts      delete a user, their files and data (audited)
  cv.ts               fetch a CV's full text + signed file link (audited)
api/_lib/server.ts    service-role client, admin check, audit helper
supabase/schema.sql   shared database schema
```

## Notes and limits

- "Est. MRR" is computed from plan counts and `src/config.ts`. Exact revenue, refunds and payouts live in Stripe or Polar.
- Revoking Signature here does **not** cancel a Stripe / Polar subscription; the UI reminds you to cancel it there.
- The Candidates list and exports read up to 5,000 rows per export. If you grow far beyond that, add pagination to the export or a database view per report.
- Tested against mocked Supabase responses and an in-memory Postgres for the schema and security rules. Run it once against your real project before relying on it.

## Tests

This repository has no automated test suite of its own. Its pages and functions were checked against mocked Supabase responses and the shared schema's security rules were checked on an in-memory Postgres (those checks live with the public app, in its `tests/` folder). Run the dashboard once against your real project before relying on it.

## License

Copyright (c) 2026 RD. All rights reserved. This repository is private and is not licensed for use by others.
