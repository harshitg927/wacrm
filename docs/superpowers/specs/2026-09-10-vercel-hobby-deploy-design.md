# Vercel Hobby deploy for wacrm (design)

**Date:** 2026-09-10  
**Repo:** https://github.com/harshitg927/wacrm (fork)  
**Plan:** Approach 2 — Vercel Hobby + external cron + new Supabase project + full WhatsApp path

## Goals

Deploy this WhatsApp CRM fork to Vercel’s free (Hobby) tier with a new Supabase backend, Meta Cloud API webhook wired to the production URL, and an external scheduler for automation/flow crons.

## Architecture

| Piece | Choice |
| --- | --- |
| App host | Vercel Hobby, Git-linked to `harshitg927/wacrm`, production branch `main` |
| Public URL | Default `*.vercel.app` (custom domain later) |
| Database / Auth / Storage | New Supabase project; apply all `supabase/migrations` |
| WhatsApp | Meta Cloud API → `https://<app>.vercel.app/api/whatsapp/webhook` |
| Cron | External pinger (~5 min) for `GET /api/automations/cron` and `GET /api/flows/cron` with header `x-cron-secret` matching `AUTOMATION_CRON_SECRET` |
| MCP server package | Out of scope (not hosted on Vercel) |

## Repo changes in scope

- Minimal Vercel deploy documentation (`docs/deployment-vercel.md`)
- Optional lightweight `vercel.json` only if needed for framework/build clarity (no Vercel Cron — Hobby interval is insufficient)
- Leave `output: "standalone"` unchanged (Docker path)

## Environment variables (Vercel Production)

**Required**

- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY`
- `ENCRYPTION_KEY` (64 hex chars)
- `META_APP_SECRET`

**Recommended**

- `NEXT_PUBLIC_SITE_URL` = production `https://<app>.vercel.app`
- `NEXT_PUBLIC_APP_LOCALE` = `en`
- `AUTOMATION_CRON_SECRET` (long random; required for Wait steps / flow timeout sweep)

**Optional later**

- `META_APP_ID`, AI tuning vars, `ALLOWED_INVITE_HOSTS`

## Setup sequence

1. Create Supabase project → apply migrations → note URL + anon + service-role keys  
2. Generate `ENCRYPTION_KEY` and `AUTOMATION_CRON_SECRET`  
3. Push any deploy docs to the fork  
4. Import repo on Vercel, set env vars, deploy  
5. Configure Supabase Auth Site URL + redirect allow-list for the Vercel host  
6. Configure Meta webhook (callback URL + verify token / app secret)  
7. In app Settings → WhatsApp, save access token + phone number IDs  
8. Schedule external cron jobs with `x-cron-secret`

## Constraints / known limits (Hobby)

- Serverless max duration ~60s; large broadcast resume (`maxDuration = 300`) may truncate  
- No native high-frequency Vercel Cron on Hobby — use external cron  
- `NEXT_PUBLIC_*` values are build-time; changing them requires redeploy

## Success criteria

- App loads on `*.vercel.app`; signup/login works against Supabase Auth  
- Meta webhook verification succeeds; inbound messages appear in inbox  
- Cron endpoints return 200 when called with the correct `x-cron-secret` (not 503/401)

## Out of scope

- Custom domain / DNS  
- Vercel Pro / native Vercel Cron  
- Hostinger path changes  
- Deploying `mcp-server/`
