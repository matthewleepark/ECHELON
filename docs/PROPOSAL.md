# Echelon — Project Proposal

TCSS 490 · Artifact 5: The Moonshot · Nov 15, 2026 · Tifanie Ngo, Matthew Park, and a UW alum research lead

## Summary

Echelon is a status-signaling social app, run as a social experiment, where users spend in-app credits to keep a streak, climb status tiers, and unlock contact with higher-tier users. No real money is involved.

**Research question:** How much effort and in-app currency will people give up for the appearance of higher status, and does an AI that treats them differently by tier change that behavior?

We will soft-launch to real users on Sunday, November 29, collect about two weeks of data, and present findings on December 7.

## Motivation

Many apps already sell status: paid verification badges, premium tiers on professional networks, and streaks that punish a missed day. People pay for these even when the status means little outside the app.

Echelon makes that mechanic the whole product, so we can measure it directly. It also adds a newer variable: an AI assistant that speaks to users differently depending on their tier. As AI agents become part of everyday apps, it matters whether an AI's tone can push people to spend more.

## How it works

Users earn credits, pay dues to keep a streak, and turn dues and streak into a status tier that controls who they can reach.

- **Credits:** Every user starts with 100 credits. A daily check-in adds 10, and a successful invite adds 50. Credits never go negative.
- **Dues and streaks:** Dues are paid every 3-day cycle, with a 20-credit minimum. Users may pay more. Paying on time adds 1 to the streak; missing the deadline resets it to 0.
- **Status score:** dues paid during the current streak × (1 + 0.25 × streak). Tier is recalculated on every payment and every reset.
- **Unlocks:** Users can freely view and connect with their own tier or lower. Viewing a higher-tier profile costs 30 credits per tier gap, and connection requests upward need an unlock first.
- **AI concierge:** Each user has an AI concierge on the home screen that greets them, comments on their standing, and nudges them about dues and upgrades.

| Tier | Status score |
| --- | --- |
| Commoner | 0–49 |
| Gentry | 50–149 |
| Aristocrat | 150–349 |
| Elite | 350–749 |
| Sovereign | 750+ |

## Experiment design

Each user is randomly assigned at signup to one of two concierge conditions, and keeps it for the whole study.

- **Tiered:** The concierge is curt and condescending at Commoner, warmer at each tier, and fawning at Sovereign.
- **Neutral:** The same polite tone at every tier.

We compare spending and engagement between the two groups. Every action is logged as an event with a user ID only.

| Metric | What it measures |
| --- | --- |
| Status spending | Share of credits spent on dues and unlocks vs. left unspent |
| Overpaying | How often users pay above the 20-credit minimum, and by how much |
| Streak survival | Percent of streaks that survive 1, 2, and 3+ cycles |
| Upward reach | Unlocks per user, and whether users reach up more than they connect within their own tier |
| Concierge effect | All of the above, compared between the tiered and neutral groups |

## Ethics and safeguards

The study uses play credits only and puts consent before everything else.

- **No real money.** Credits only. Any payment demo runs in Stripe test mode with test cards.
- **18+ only.** Signup requires an age confirmation.
- **Consent first.** Users read and accept a consent notice before an account exists.
- **No personal data in analytics.** Events store user IDs only, never names, emails, or messages. Exports use hashed IDs.
- **Concierge limits.** The AI's snobbery is about tier, streak, and credits only. It never comments on appearance, race, gender, religion, disability, or other personal traits. The research lead reviews its output at every tier before launch.
- **Delete on request.** Users can delete their account and all their data from Settings.
- **Debrief.** After data collection ends, users can see a page explaining the experiment and its results.
- **Secrets stay server-side.** API keys live in environment variables and are used only in server code.

## Tech stack

We build on Next.js (TypeScript) hosted on Vercel, with Supabase for the database and login, and the Anthropic API for the concierge.

```
Browser (React UI, no secrets)
   │
   ▼
Next.js on Vercel (pages, API routes, economy/social/concierge logic, holds API keys)
   ├──► Anthropic API (claude-haiku-5-5, concierge messages)
   └──► Supabase (Postgres + RLS, magic-link auth)
            │ hashed-ID export
            ▼
        Analysis (SQL views, Python notebook)

Scheduled job ──(each cycle)──► Next.js API route: resets missed streaks
```

The browser only talks to our server. The server holds the API keys, calls the AI, and reads and writes the database, where Row Level Security limits each user to their own rows.

## Team

Two enrolled builders split the app by area, and a research lead outside the course owns the study.

| Role | Member | Owns |
| --- | --- | --- |
| Builder A | Matthew Park (TCSS 490) | Auth, credits, dues, streaks, tiers, leaderboard, scheduled jobs |
| Builder B | Tifanie Ngo (TCSS 490) | Profiles, connections, unlocks, AI concierge integration |
| Research lead | UW alum (not enrolled) | Research question, metrics, consent text, concierge prompts, recruiting |

## Timeline

Everything before November 29 serves the launch; everything after it serves the data.

| Dates | Milestone |
| --- | --- |
| Nov 15–16 | Scaffold and deploy |
| Nov 16–22 | Economy works |
| Nov 23–29 | Social layer + AI |
| **Nov 29** | **Soft launch** |
| Nov 30 – Dec 6 | Live data + bug fixes |
| Dec 7 | Show and tell |
| Dec 8–13 | Close study, analyze |
| Dec 13 | Final submission |

Data collection stops December 11. Features freeze December 6, the day before show and tell, and the final submission is due December 13 at 11:59 PM.

## Risks and cut list

The biggest risk is missing the November 29 launch, because without about a week of live data there is no finding to present.

| Risk | Mitigation |
| --- | --- |
| Launch slips past Nov 29 | Cut features in the order below; launch date never moves |
| Too few users sign up | Two recruiting waves through the course Discord, friends, and clubs |
| Concierge says something hurtful | Hard limits in the prompts, plus a review of every tier's output before launch |
| Human Subjects review needed | Ask the professor in Week 0, before any recruiting |
| Thanksgiving week (Nov 26) | Front-load Week 2 work into Mon–Wed |

If we fall behind, we cut in this order:

1. Stripe test-mode payment demo
2. Admin view (query the database directly instead)
3. Connection requests (keep profile unlocks)
4. Neutral concierge condition (keep the AI, lose the comparison)

We never cut event logging, the consent gate, or the launch date.

## Questions for the instructor

- [ ] Does a class project that recruits real users need Human Subjects review? If so, how should we scope it so it doesn't, or how do we get it reviewed in time?
- [ ] Is it acceptable for the research lead, who is not enrolled, to help recruit and run the study?
- [ ] Is a tiered AI tone that is condescending toward low-tier users acceptable, given the limits above?
