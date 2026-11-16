# Echelon — Project Plan

**Show and tell:** Monday, December 7 · **Due:** Sunday, December 13, 11:59 PM
**Weekly check-ins:** submit portfolio link with the Artifact 5 card updated at the end of each week.

The plan has one non-negotiable date besides the deadlines: **soft launch to real users by Sunday, November 29.** Without about a week of live data, there is no research finding to present. Everything before that date is in service of launching; everything after it is in service of the data.

---

## Week 0 — Now (Nov 15–16): Scaffold

| Task | Owner |
|---|---|
| Create GitHub repo, add CLAUDE.md, PLAN.md, LOG.md | Builder A |
| Ask professor: does this need Human Subjects review as a class project, or how should we scope it so it doesn't? | Builder A or B |
| Set up Next.js + Supabase + Vercel; deploy a "hello world" so deployment works from day one | Builder B |
| Draft research question, metrics list, and consent text (`RESEARCH.md`, `CONSENT.md`) | Research lead |

**Done when:** a blank app is live at a Vercel URL and the professor has answered the review question.

---

## Week 1 (Nov 16–22): The economy works

**Builder A**
- Supabase auth (email magic link), 18+ checkbox, consent gate
- Tables + RLS: `profiles`, `credits_ledger`, `dues_payments`, `events`
- `lib/economy`: credits, check-in, dues cycle, streak, status score, tier — all unit tested
- Event logging helper

**Builder B**
- Profile page (display name, tier badge, streak, status score)
- Leaderboard page (sorted by status score)
- Home dashboard: credits, next dues deadline countdown, pay-dues button

**Research lead**
- Finalize tier names and thresholds (playtest the math on paper: how many days to reach each tier?)
- Write the five tiered concierge prompts + the neutral prompt
- Line up recruiting channels (course Discord, friends, clubs) and write the invite message

**Done when:** a user can sign up, check in, pay dues, watch their streak and tier change, and see themselves on the leaderboard. Week check-in card submitted.

---

## Week 2 (Nov 23–29): The social layer + AI — then launch

*Thanksgiving is Nov 26. Plan for lighter days Thu–Fri and front-load the week.*

**Builder A**
- Invite links + invite bonus
- Scheduled job (Supabase cron or Vercel cron) that resets missed streaks at cycle end
- Admin view: total users, tier distribution, events per day

**Builder B**
- Browse users, tier gating, pay-to-view unlocks (30 credits per tier gap)
- Connection requests + accept/decline
- AI concierge: server route, prompts by tier, random condition assignment at signup, rate limit, caching

**Research lead**
- Test the whole app as a new user; report anything confusing
- Check concierge outputs at every tier for anything that crosses a line
- Prepare launch post

**Sunday Nov 29: soft launch.** Recruit the first wave of users.

**Done when:** real users are signed up and events are flowing into the `events` table.

---

## Week 3 (Nov 30–Dec 6): Live data + polish

**Builders A & B**
- Fix bugs reported by real users (priority over new features)
- Mobile layout polish
- Account deletion in Settings
- Stretch, only if everything above is solid: Stripe test-mode demo of a real payment flow, or the 24-hour boost as a deliberate A/B test

**Research lead**
- Second recruiting wave
- Mid-study snapshot: tier distribution, unlock rate, streak survival, tiered vs. neutral concierge
- Draft the findings slides

**Dec 6:** freeze features. Rehearse the demo.

---

## Show and tell — Monday, Dec 7

Show: the live app, the concierge at two different tiers, and preliminary findings (state clearly that the study is still running). Close with what you learned about building it with AI agents.

---

## Final week (Dec 8–13): Close the study

- Keep the app running through Dec 11, then stop data collection
- Export anonymized data (`analysis/`), run the analysis, write `docs/FINDINGS.md`
- Optionally show users a debrief page explaining the experiment and the results
- Each enrolled member updates their portfolio card (title, link, reflection) and submits on Canvas by **Dec 13, 11:59 PM**

---

## What to measure (starting list for RESEARCH.md)

- **Status spending:** share of credits spent on dues and unlocks vs. left unspent
- **Overpaying:** how often users pay above the 20-credit minimum, and by how much
- **Streak survival:** percent of streaks that survive 1, 2, 3+ cycles
- **Upward reach:** unlock purchases per user, and whether users unlock higher tiers more than they connect within their own
- **Concierge effect:** compare all of the above between the `tiered` and `neutral` groups

## Cut list (if you fall behind, cut in this order)

1. Stripe test-mode demo
2. Admin view (query Supabase directly instead)
3. Connection requests (keep unlocks; viewing a profile is enough for the experiment)
4. Neutral concierge condition (lose the comparison, keep the AI)

Never cut: event logging, consent gate, the launch date.
