# Launch Checklist — The Gratefulness Challenge

Companion to [PRD.md](PRD.md) and [ROADMAP.md](ROADMAP.md). Work through top to bottom; items marked **(v0.2)** apply to the prototype-hardening release, the rest gate v1.0.

## QA / Functional
- [ ] **(v0.2)** Entry saved at 11:55pm and 12:05am local time land on different days; streak math correct across the boundary (currently fails — UTC bug)
- [ ] **(v0.2)** Second save attempt same day is blocked with clear messaging
- [ ] **(v0.2)** Streak resets to 1 after a missed day; badge predicates fire exactly once
- [ ] **(v0.2)** All 30 prompts cycle correctly; Day 30 → completion state (verify by seeding `gc-state`)
- [ ] **(v0.2)** Tree stage changes at entry counts 1 / 8 / 16 / 25
- [ ] **(v0.2)** App updates propagate after a service-worker deploy (bump cache name, verify old cache purged)
- [ ] **(v0.2)** Works on iOS Safari, Android Chrome, desktop Chrome; installs to home screen on both platforms; offline launch works
- [ ] Sync: entry created offline appears on second device after reconnect (v0.5)
- [ ] Guest → account upgrade preserves entries, XP, streak, badges (v0.5)
- [ ] Notification arrives at chosen local time; none after entry done (v0.5)
- [ ] Private entry never visible in feed or via API with another user's token — test with RLS (v0.8)
- [ ] Payment lifecycle: subscribe, feature unlock, cancel, expiry re-lock (v1.0)

## Privacy, Legal, Trust (journal entries = sensitive personal data)
- [ ] Privacy policy + Terms of Service published and linked in-app
- [ ] Data export (JSON + readable text) and full account deletion, honored ≤ 30 days
- [ ] Encryption at rest; TLS everywhere; no journal content in analytics events or logs
- [ ] Community moderation live before feed opens: report, block, filter, takedown SLA
- [ ] Age gate / COPPA position decided (Family plan implies minors)
- [ ] App-store privacy labels (if distributing via stores)

## Infrastructure & Ops
- [ ] Production hosting with HTTPS + custom domain
- [ ] Supabase: RLS on every table; backups enabled; staging vs. prod projects
- [ ] Error tracking (e.g. Sentry) + uptime monitor
- [ ] Analytics events: signup, entry_saved, streak_broken, share, paywall_view, subscribe, churn
- [ ] Store secrets (Stripe, VAPID, service keys) outside the repo

## Product Polish
- [ ] Real app icons (192/512/maskable) + iOS splash screens
- [ ] Empty/edge states: first-run home, feed with no posts, expired subscription
- [ ] Loading + error states for every network call
- [ ] Accessibility pass: contrast (yellow on white is risky), focus order, labels on icon-only buttons, reduced-motion
- [ ] Copy pass on notifications and paywall

## Launch Marketing (minimum)
- [ ] Landing page with install/App Store links and email capture
- [ ] Day-30 certificate + milestone share cards render correctly (this is the growth loop)
- [ ] Beta cohort (20–50 users) completed a full 30-day cycle before public launch
