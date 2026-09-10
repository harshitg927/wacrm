# Deploy wacrm on Vercel (Hobby)

This fork is set up for **Vercel Hobby (free)** with an **external cron** (Vercel Cron on Hobby is too infrequent for Wait steps / flow cleanup).

## What you need

1. A Supabase project with all `supabase/migrations` applied  
2. A Meta (WhatsApp Cloud API) app + `META_APP_SECRET`  
3. A [Vercel](https://vercel.com) account linked to this GitHub repo  
4. An external cron service (e.g. [cron-job.org](https://cron-job.org))

## Environment variables

Set these in **Vercel → Project → Settings → Environment Variables** (Production + Preview as needed).

| Variable | Required | Notes |
| --- | --- | --- |
| `NEXT_PUBLIC_SUPABASE_URL` | yes | e.g. `https://xxxx.supabase.co` |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | yes | Supabase anon / publishable key |
| `SUPABASE_SERVICE_ROLE_KEY` | yes | Server-only; never expose to the client |
| `ENCRYPTION_KEY` | yes | 64 hex chars: `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"` |
| `META_APP_SECRET` | yes | Meta App → Settings → Basic |
| `NEXT_PUBLIC_SITE_URL` | recommended | `https://<your-app>.vercel.app` (no trailing slash) |
| `NEXT_PUBLIC_APP_LOCALE` | optional | Default `en` |
| `AUTOMATION_CRON_SECRET` | recommended | Protects cron routes; generate with `openssl rand -hex 32` |
| `META_APP_ID` | optional | Needed for image-header template uploads |

`NEXT_PUBLIC_*` values are inlined at **build time**. Change them → redeploy.

## Import & deploy

1. Import [harshitg927/wacrm](https://github.com/harshitg927/wacrm) in Vercel (Framework: Next.js, Root: `.`).  
2. Add the env vars above.  
3. Deploy from `main`.  
4. Note the production URL (`https://<project>.vercel.app`).

## Supabase Auth URLs

In Supabase → **Authentication → URL Configuration**:

- **Site URL:** `https://<project>.vercel.app`  
- **Redirect URLs:** include  
  - `https://<project>.vercel.app/**`  
  - `https://<project>.vercel.app/auth/callback` (if your auth callback uses that path)

Also set `NEXT_PUBLIC_SITE_URL` to the same origin and redeploy if you added it after the first build.

## WhatsApp (Meta) webhook

1. Callback URL: `https://<project>.vercel.app/api/whatsapp/webhook`  
2. Verify with your Meta app secret / verify token as configured in the app.  
3. Subscribe to message webhooks for your WABA.  
4. In the live CRM: **Settings → WhatsApp** — paste access token, phone number ID, and WABA ID.

## External cron (required for Wait steps & flow timeouts)

Both endpoints expect header `x-cron-secret: <AUTOMATION_CRON_SECRET>`:

- `GET https://<project>.vercel.app/api/automations/cron`  
- `GET https://<project>.vercel.app/api/flows/cron`

Schedule each about every **5 minutes**. Until `AUTOMATION_CRON_SECRET` is set, both return **503**.

## Hobby limits

- Function timeout ≈ **60s**. Large broadcast resume jobs that expect up to 300s may truncate.  
- Prefer an external HTTP cron; do not rely on Vercel Hobby cron frequency for automations.

## Smoke checks

1. Open the Vercel URL → signup / login works.  
2. Meta webhook verifies.  
3. Cron URLs return **200** with the correct secret (not 401/503).  
4. Send a test WhatsApp message → it appears in the inbox.
