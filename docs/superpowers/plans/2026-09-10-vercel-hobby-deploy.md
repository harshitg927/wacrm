# Vercel Hobby Deploy Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans (inline) — user requested immediate implementation.

**Goal:** Deploy `harshitg927/wacrm` to Vercel Hobby with a new Supabase project, Meta webhook, and external cron.

**Architecture:** Next.js on Vercel Hobby; Postgres/Auth/Storage on a new Supabase project; Meta Cloud API webhook to `*.vercel.app`; external cron hits `/api/automations/cron` and `/api/flows/cron` with `x-cron-secret`.

**Tech Stack:** Next.js 16, Supabase, Vercel Hobby, Meta WhatsApp Cloud API, external HTTP cron.

## Global Constraints

- Deploy from GitHub fork `harshitg927/wacrm` (not Connec8 upstream).
- Vercel Hobby only — no Pro cron / no reliance on 300s maxDuration.
- New Supabase project in org `bmmekstxsjwoyhmanxsa` (harshitg927's Org); cost $0/month.
- Preferred Supabase region: `ap-south-1` (Mumbai) unless create fails.
- Do not commit secrets; set them in Vercel dashboard/CLI env.
- Do not host `mcp-server/` on Vercel in this plan.

---

### Task 1: Supabase project + schema

**Files:**
- Use: `supabase/migrations/*.sql` (apply in order via MCP `apply_migration` or CLI)

- [ ] **Step 1: Create project** named `wacrm` in org `bmmekstxsjwoyhmanxsa`, region `ap-south-1`, with `confirm_cost_id` from cost confirmation ($0/month).
- [ ] **Step 2: Wait until `get_project` status is `ACTIVE_HEALTHY` (or equivalent active).**
- [ ] **Step 3: Apply all migrations** from `supabase/migrations/` in filename order via `apply_migration` (name = filename without `.sql`, query = file contents).
- [ ] **Step 4: Collect** project URL (`get_project_url`), publishable/anon key (`get_publishable_keys`), and service-role key from dashboard if MCP cannot return service role — store only in Vercel env later.
- [ ] **Step 5: Verify** `list_tables` shows core tables (e.g. profiles, conversations, contacts).

---

### Task 2: Deploy docs + secrets generation helpers

**Files:**
- Create: `docs/deployment-vercel.md`
- Create: `docs/superpowers/specs/2026-09-10-vercel-hobby-deploy-design.md` (already written)
- Modify: `README.md` — one line linking Vercel deploy doc under Documentation

- [ ] **Step 1: Write `docs/deployment-vercel.md`** covering Hobby limits, required env vars, Supabase Auth redirect URLs, Meta webhook URL, external cron header setup.
- [ ] **Step 2: Add README link** to that doc.
- [ ] **Step 3: Generate local secrets** (do not commit):

```bash
node -e "console.log('ENCRYPTION_KEY=' + require('crypto').randomBytes(32).toString('hex'))"
openssl rand -hex 32   # AUTOMATION_CRON_SECRET
```

- [ ] **Step 4: Commit docs** to local main; push to `harshitg927/wacrm` (update `origin` if needed).

---

### Task 3: Vercel project + env + deploy

**Files:** none required in repo beyond docs; optional empty/minimal `vercel.json` only if needed.

- [ ] **Step 1: Resolve Vercel `teamId`** via `list_teams` / `get_git_deployment_context`.
- [ ] **Step 2: `create_git_project`** with `repo: harshitg927/wacrm`, `teamId`, `projectName: wacrm`, `deploy: false` first if env not set — or deploy after env.
- [ ] **Step 3: Set Production env vars** via Vercel dashboard/CLI: Supabase trio, `ENCRYPTION_KEY`, `META_APP_SECRET` (user must provide Meta secret), `AUTOMATION_CRON_SECRET`, `NEXT_PUBLIC_SITE_URL` after first URL known (may need second deploy).
- [ ] **Step 4: Trigger production deploy**; confirm build succeeds via deployment logs.
- [ ] **Step 5: Configure Supabase Auth** Site URL + redirect URLs for `https://<deployment>.vercel.app/**`.

---

### Task 4: Meta WhatsApp + external cron (guided)

**Files:** checklist in `docs/deployment-vercel.md`

- [ ] **Step 1: User provides `META_APP_SECRET`** (and Meta App ID if available); set on Vercel; redeploy if needed.
- [ ] **Step 2: Meta webhook** callback = `https://<app>.vercel.app/api/whatsapp/webhook`; verify with app secret / verify token per app Settings flow.
- [ ] **Step 3: In wacrm Settings → WhatsApp**, paste access token + phone number ID + WABA ID.
- [ ] **Step 4: External cron** every 5 minutes:
  - `GET https://<app>.vercel.app/api/automations/cron` header `x-cron-secret: <AUTOMATION_CRON_SECRET>`
  - `GET https://<app>.vercel.app/api/flows/cron` same header
- [ ] **Step 5: Smoke test** login, webhook verify, cron 200.

---

## Spec coverage

| Spec item | Task |
| --- | --- |
| New Supabase + migrations | Task 1 |
| Vercel Hobby Git deploy | Task 3 |
| Env vars | Tasks 2–3 |
| External cron | Task 4 |
| Meta full path | Task 4 |
| Docs | Task 2 |
| Hobby limits documented | Task 2 |
