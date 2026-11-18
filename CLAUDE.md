# CLAUDE.md

Guidance for Claude Code (claude.ai/code) in this repository: the **DHA Works admin portal** (Next.js 16, TypeScript). It was split from `apps/admin` of [DHA-Works-Platform](https://github.com/abdullah-automates31/DHA-Works-Platform) on 15 Nov 2026 with its history. The database, worker app and product documents stay in that repository; read `README.md` first.

## Rules that must not be broken

- **Never the service-role key** in this app or its environment. The portal acts as the signed-in staff member; the database decides what is allowed.
- **Every page and server action calls `requireAdmin()`** (layout and actions) or `requireSession()` for reads that the database checks again. New admin database functions live in the platform repository and need `is_admin()`, an audit row and tests there.
- No `select('*')`. Lists are paged. Signed URLs are short-lived (5 minutes). Forms use `ActionForm` (it keeps typed text across a refusal or the code page). Database refusals go through `refusal()` in `src/lib/action.ts`, which turns "confirm with a new code" into a redirect to `/confirm`.
- Security headers and the nonce CSP are built in `src/proxy.ts` and `src/lib/security-headers.ts`; do not add inline scripts or loosen `script-src`.
- Sign-in: password, Turnstile (server-side), then a six-digit code emailed by the platform Edge Function `admin-2fa` (`admin_verify_code`, migration 0055; the authenticator app of 0052 is no longer used and `/signin/enroll` is gone). The same emailed code is the 15-minute step-up on sensitive actions (`/confirm`). A production build without `TURNSTILE_SECRET_KEY` refuses every sign-in.
- Never print or commit secrets. Only `.env.example` is committed. Do not weaken a security rule to make something work: stop and explain.

## Commands

`npm install`, `npm run dev`, `npm run typecheck`, `npm run lint`, `npm test`, `npm run build`. Build needs `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY` (placeholders are fine, see `.github/workflows/ci.yml`).

## Database types

`vendor/supabase/database.types.ts` is generated in the platform repository and copied here with `npm run types:sync -- <file>`. Never hand-edit it. When a database change adds or renames an RPC the portal calls, sync the types in the same pull request as the portal change.

## Working agreements

Work on a branch and open a pull request into `main`; ask before pushing to `main`. Keep the platform repository's `docs/ADMIN-PORTAL.md` in step when behaviour changes (this repository keeps a copy in `docs/`).

## State (16 Nov 2026)

- Deployed on Vercel: https://dhaworks-admin-portal.vercel.app (project `dhaworks-admin-portal`, built from `main` of this repository). Settings in Vercel: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY` (DEV), `NEXT_PUBLIC_TURNSTILE_SITE_KEY`, `TURNSTILE_SECRET_KEY`. A push to `main` deploys. The Turnstile widget lists the hostname `dhaworks-admin-portal.vercel.app`. Add the Vercel URL to the DEV Supabase redirect URLs.
- **The sign-in code is not emailed yet:** Brevo "Authorised IPs" is switched on, so the function's calls are refused (401). The owner turns IP blocking off at app.brevo.com/security/authorised_ips and `ADMIN_MAIL_FROM` must be a verified Brevo sender (secrets live in the platform's Supabase project, never here).
- New since the split: "Let them apply again" on a rejected worker or resident (`admin_reopen_rejected`, platform migration 0054, files `ReopenForm.tsx` and `reopenAction` in `src/features/verification`). Database types are in `vendor/supabase/database.types.ts` (copy from the platform repository after a migration; never hand-edit).
- This repository is a copy of `apps/admin` of DHA-Works-Platform: a portal change is made in both (same paths under `src/`). Checks: `npm run typecheck`, `npm run lint`, `npm test` (87 tests), `npm run build`.
