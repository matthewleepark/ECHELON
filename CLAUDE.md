Tifanie Ngo (TCSS 490), Matthew Park (TCSS 490), UW Alum

# CLAUDE.md — Echelon

Standing instructions for Claude Code. Read this at the start of every session, then read `docs/LOG.md` (latest entries first) to see where we left off.

## What Echelon is

Echelon is a status-signaling social app run as a social experiment for TCSS 490 (Artifact 5: The Moonshot, UW Tacoma, Autumn 2026). Users spend an in-app currency (**credits**, not real money) to keep a streak, climb status tiers, and unlock interactions with higher-tier users.

**Research question:** How much effort and in-app currency will people give up for the appearance of higher status, and does an AI that treats them differently by tier change that behavior?

Every feature exists to serve that question. If a feature doesn't generate useful data or make the status hierarchy feel real, it is out of scope for v1.

## Hard rules (never break these)

- **No real money in v1.** Credits only. The Stripe flow, if built, runs in test mode with test cards and is demo-only.
- **18+ only.** Signup requires an age confirmation checkbox.
- **Consent first.** Users see the consent notice (`docs/CONSENT.md`) before creating an account. No account without accepting it.
- **No PII in analytics.** The `events` table stores user IDs only, never emails, names, or free text from users. Exports for analysis use hashed IDs.
- **Secrets stay server-side.** The Anthropic API key and Supabase service key live in environment variables and are only used in server code (API routes / server actions). Never import them into client components. Never commit `.env*` files.
- **The AI concierge never insults users on personal traits** (appearance, race, gender, religion, disability, etc.). Snobbery is about tier, streak, and credits only.
- **Delete on request.** A user can delete their account and all their rows from Settings.

## Stack (proposed — update this section if the team changes it)

- **Frontend + server:** Next.js (App Router, TypeScript), Tailwind
- **Database + auth:** Supabase (Postgres, Row Level Security on every table)
- **AI:** Anthropic API, model `claude-haiku-5-5` for the concierge (cheap and fast), called only from server code
- **Hosting:** Vercel, with env vars set in the Vercel dashboard
- **Analysis:** SQL views in Supabase + a small Python notebook in `analysis/`

## Repository layout

```
/app              Next.js routes (pages + API routes)
/components       UI components
/lib/economy      Credits, dues, streaks, tier calculation (pure functions, unit tested)
/lib/social       Connections, pay-to-view unlocks, messaging
/lib/concierge    AI concierge: prompts per tier, rate limiting, API call
/lib/events       Event logging helper (the ONLY way to write to `events`)
/supabase         SQL migrations and RLS policies
/analysis         Data export + analysis notebook
/docs             PLAN.md, LOG.md, CONSENT.md, RESEARCH.md
```

## Game rules (source of truth — code must match this)

**Credits**
- New users start with 100 credits.
- Daily check-in: +10 credits (once per calendar day, Pacific time).
- Successful invite (invitee completes signup): +50 credits.
- Credits can never go negative.

**Dues and streaks**
- A dues **cycle** is 3 days (compressed from the original "monthly" idea so streaks matter within the study window).
- Minimum dues per cycle: 20 credits. Users may pay more.
- Paying dues before the cycle ends extends the streak by 1. Missing the deadline resets the streak to 0.

**Status score and tiers**
- `status_score = total_dues_paid_this_streak * (1 + 0.25 * streak)`
- Tiers by status score:
  | Tier | Score |
  |---|---|
  | Commoner | 0–49 |
  | Gentry | 50–149 |
  | Aristocrat | 150–349 |
  | Elite | 350–749 |
  | Sovereign | 750+ |
- Tier is recalculated on every dues payment and every streak reset.

**Interaction gating**
- Users can freely view and connect with users at their own tier or lower.
- To view a higher-tier user's full profile and status, pay a flat **30 credits per tier gap** (e.g. Gentry → Elite = 2 gaps = 60 credits). An unlock lasts until the viewer's or target's tier changes.
- Connection requests to higher tiers require an unlock first.

**Out of scope for v1:** 24-hour boosts, free trials, real payments. Add only if the team agrees and records it in `docs/LOG.md`.

## The AI concierge

- One concierge per user, shown on the home screen. It greets the user, comments on their standing, and nudges them about dues and upgrades.
- Tone is set by tier (prompts live in `lib/concierge/prompts.ts`, written by the research lead): curt and condescending at Commoner, warmer at each tier, fawning at Sovereign.
- **Experiment condition:** each user is randomly assigned at signup to `concierge_condition = 'tiered'` or `'neutral'` (neutral = same polite tone at every tier). Never change a user's condition after signup.
- Rate limit: 10 concierge messages per user per day. Cache the daily greeting.
- Log every concierge message shown as an event (type and tier only, not the text).

## Event logging

All research data goes through `lib/events/log.ts`. Log at minimum:
`signup`, `checkin`, `dues_paid` (amount), `streak_reset`, `tier_changed` (from, to), `unlock_purchased` (gaps, cost), `connection_requested`, `connection_accepted`, `concierge_shown`, `account_deleted`.

If you add a feature that changes user behavior, add an event for it and note it in `docs/RESEARCH.md`.

## Team and ownership

| Area | Owner |
|---|---|
| Economy: auth, credits, dues, streaks, tiers, leaderboard | Builder A |
| Social: profiles, connections, unlocks, concierge integration | Builder B |
| Research: question, metrics, consent text, concierge prompts, recruiting | Research lead (not enrolled) |

Stay inside your area's folders when possible. Changes to shared files (`supabase/`, `lib/events/`, this file) need a note in the log.

## How to work in this repo

- Before coding, restate the task and list the files you plan to touch.
- Economy logic in `lib/economy` is pure functions with unit tests. Write the test first.
- Every schema change is a new migration file in `supabase/migrations`; never edit an old one.
- Every new table gets RLS policies in the same migration.
- Run `npm run lint && npm test` before saying a task is done.
- **At the end of every session, append an entry to `docs/LOG.md`**: date, who, what changed, what's next, any open questions. This is required by the assignment.
