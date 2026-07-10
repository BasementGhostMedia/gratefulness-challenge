# Launch Checklist — The Gratefulness Challenge

Companion to [PRD.md](PRD.md) and [ROADMAP.md](ROADMAP.md). Work through top to bottom; items marked **(v0.2)** apply to the prototype-hardening release, the rest gate v1.0.

## QA / Functional
- [x] **(v0.2)** Entry saved at 11:55pm and 12:05am local time land on different days; streak math correct across the boundary (fixed: `todayStr()`/streak "yesterday" check now use `localDateStr()` instead of `toISOString()`; verified live against the actual UTC/local offset)
- [x] **(v0.2)** Second save attempt same day is blocked with clear messaging (verified: second `saveEntry()` call is a no-op, redirects to Home which shows "✓ Today's entry is saved — see you tomorrow!")
- [x] **(v0.2)** Streak resets to 1 after a missed day; badge predicates fire exactly once (verified via seeded Day-30 run: Champion + Golden Tree badges each added exactly once, no duplicates)
- [x] **(v0.2)** All 30 prompts cycle correctly; Day 30 → completion state (verified by seeding `gc-state` with 29 entries and saving Day 30 — correct final prompt, completion screen, badges)
- [x] **(v0.2)** Tree stage changes at entry counts 1 / 8 / 16 / 25 (verified: Seed→Seedling→Sapling→Tree→Golden Tree transition exactly at 1/8/16/25)
- [x] **(v0.2)** App updates propagate after a service-worker deploy (bump cache name, verify old cache purged) (verified: bumping `CACHE` to `gc-v5` and reloading purged `gc-v4` from the Cache Storage; also switched to network-first for HTML so this class of bug is much less likely going forward)
- [ ] **(v0.2)** Works on iOS Safari, Android Chrome, desktop Chrome; installs to home screen on both platforms; offline launch works — **only verified on desktop Chrome (automated) this pass**; real iOS Safari / Android Chrome device testing still needed
- [~] Sync (v0.5): wiped-device sign-in restores full progress from server — **verified**. Offline queue (`gc-pending`, flush on reconnect) is implemented but the offline→reconnect path hasn't been exercised end-to-end yet
- [x] Guest → account upgrade preserves entries, XP, streak, badges (v0.5) — verified: 2 seeded guest entries migrated via `migrate_guest_data`, streak/XP/badges recomputed server-side correctly
- [ ] Notification arrives at chosen local time; none after entry done (v0.5 — deferred to next session)
- [~] Private entry never visible via API with another user's token (RLS) — **verified at the API layer** (account B sees zero of account A's rows; direct insert/update/delete and XP forgery all blocked). Re-verify when the feed ships (v0.8)
- [ ] Payment lifecycle: subscribe, feature unlock, cancel, expiry re-lock (v1.0)

## Privacy, Legal, Trust (journal entries = sensitive personal data)
- [ ] Privacy policy + Terms of Service published and linked in-app
- [~] Data export (JSON + readable text) and full account deletion, honored ≤ 30 days — local export + device reset are implemented in Profile; server-side account deletion still needs a backend/admin flow
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
- [~] Real app icons (192/512/maskable) done (`icons/`); iOS splash screens still needed
- [~] Empty/edge states: first-run home, feed with no posts, expired subscription — first-run home and feed empty states are implemented; expired subscription still pending
- [~] Loading + error states for every network call — journal save/auth have user-facing errors and save disabled state; broader network error audit still pending
- [~] Accessibility pass: contrast (yellow on white is risky), focus order, labels on icon-only buttons, reduced-motion — keyboard focus, reduced-motion, and selected labels improved; full screen-reader/contrast audit still pending
- [ ] Copy pass on notifications and paywall

## Launch Marketing (minimum)
- [ ] Landing page with install/App Store links and email capture
- [ ] Day-30 certificate + milestone share cards render correctly (this is the growth loop)
- [ ] Beta cohort (20–50 users) completed a full 30-day cycle before public launch
