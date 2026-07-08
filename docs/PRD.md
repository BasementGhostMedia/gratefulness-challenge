# Product Requirements Document — The Gratefulness Challenge

**Version:** 1.0 · **Date:** July 3, 2026 · **Owner:** Rafael Rosario
**Status of product:** Working HTML prototype (v0.1) → this PRD defines the path to v1.0 public launch.

---

## 1. Overview

**Product:** The Gratefulness Challenge — "30 Days to a New You."
A mobile-first gratitude journaling app built around a single, focused commitment: complete one guided gratitude entry per day for 30 consecutive days.

**Problem:** People know gratitude journaling improves wellbeing, but most quit within days. Generic journaling apps offer infinite blank pages and no finish line, so there's nothing to complete and nothing to lose by skipping a day.

**Solution:** A time-boxed 30-day challenge with daily prompts, visible streaks, XP, badges, a growing tree avatar, and a supportive community — borrowing the habit mechanics that make Duolingo and fitness challenges sticky, applied to a 2-minute daily reflection.

**Positioning:** Not a general journal. A challenge you finish, with a transformation ("a new you") at the end.

---

## 2. Goals & Success Metrics

| Goal | Metric | v1 Target |
|------|--------|-----------|
| Habit formation | Day-7 retention of users who complete Day 1 | ≥ 40% |
| Challenge completion | % of starters who finish all 30 days | ≥ 15% |
| Engagement | Median entry length (reflection + gratitudes) | ≥ 120 chars |
| Monetization | Free → paid conversion within 30 days | ≥ 3% |
| Virality | % of completions that share a certificate | ≥ 25% |

North-star metric: **completed daily entries per week**.

---

## 3. Target Users & Personas

1. **The Resolution Maker** (primary) — 25–45, tries wellness habits in bursts, motivated by streaks and finish lines. Needs: low friction, daily reminder, visible progress.
2. **The Wellness Family** — a parent who wants a shared positive ritual with partner/kids. Needs: Family plan, shared feed, kid-appropriate prompts.
3. **The Journaling Veteran** — already journals, wants structure and community. Needs: quality prompts, export of their writing, privacy controls.

---

## 4. What Exists Today (v0.1 prototype)

Implemented and working, all client-side in one HTML file:

- 9 screens: Splash, Onboarding, Home, Journal, Badges, Community, Profile, Plans, Day Complete
- Navigation (tap, swipe, animated transitions), brand system (yellow/teal/black, Montserrat, line-art SVG icons)
- Real local state (`localStorage`): entries, XP (50/entry), consecutive-day streaks, 12 badges with predicate logic, 5 tree stages, 30 rotating prompts, once-per-day gating
- PWA shell: manifest + service worker offline caching

Display-only / not functional: Sign In, media capture buttons (photo/voice/video), community feed (static sample posts), subscription plans (no payments), share buttons, settings rows, push notifications.

---

## 5. v1.0 Feature Requirements

### 5.1 Accounts & Sync (P0)
- Email + Apple/Google sign-in; anonymous "guest mode" that can upgrade to an account without losing progress (migrate existing `localStorage` state on first sign-in).
- All state (entries, XP, streak, badges) synced to a backend; offline-first with sync on reconnect.
- **Acceptance:** user signs in on a second device and sees identical progress; guest data survives account creation.

### 5.2 Daily Journal (P0)
- Today's prompt (30-day rotation, as built), reflection textarea, 3 gratitude fields.
- Photo attachment (P1); voice memo (P2); video (P2 or cut from v1).
- Entries are viewable and editable same-day; read-only after (preserves challenge integrity).
- Journal content is **private by default**; sharing to community is an explicit per-entry opt-in.
- **Acceptance:** entry persists across devices; editing after midnight is blocked; a private entry never appears in any feed.

### 5.3 Streaks, XP, Badges, Tree (P0 — port existing logic to server)
- Server-authoritative streak/XP computation (client logic today is trustable only locally).
- Streaks must respect the **user's local timezone** (known bug: prototype uses UTC dates).
- One streak-freeze token per challenge (P1) to forgive a single missed day — proven to increase completion.
- **Acceptance:** streak increments correctly across DST/timezone changes; badges awarded exactly once; XP totals match entry count × 50.

### 5.4 Reminders & Notifications (P0)
- Daily reminder push at a user-chosen time (default 8pm local); streak-at-risk nudge at 9:30pm if no entry.
- Web Push for PWA; native push if/when wrapped for app stores.
- **Acceptance:** opt-in flow, per-user time setting, no notification after entry completed.

### 5.5 Community Feed (P1)
- Feed of entries users chose to share: text (+ photo), like ("gratitude"), reply.
- Moderation: report button, block user, profanity filter, admin remove. **Do not ship the feed without moderation.**
- Family circles (P1, ties to Family plan): private mini-feed for up to 6 members.
- **Acceptance:** only opted-in entries appear; reported content hidden pending review within 24h.

### 5.6 Completion & Sharing (P1)
- Day 30 celebration + generated certificate (branded image with name, dates, stats) shareable via native share sheet.
- Mid-challenge share cards for streak milestones (7/14/21).
- **Acceptance:** certificate renders with correct user data; share produces an image, not a link to private content.

### 5.7 Pricing (P1) — revised July 2026 (owner decision)
- **Free, forever:** the complete solo experience — full 30-day challenge, daily journal + gratitudes, streaks/XP/all badges/tree, community feed, and unlimited repeat runs. No trial clocks, no core features behind a paywall, usable forever.
- **Family $9.99/mo** — the only paid tier. Up to 5 family members, private family feed, shared progress view, and the premium feature set for every member: photo/voice/video entries, streak freeze (one forgiven miss per challenge), printable completion certificate.
- **Family Annual $99.99/yr** — same plan, two months free (~17% off), priority support.
- Rationale: the solo habit loop is the growth engine and stays free; monetization comes from households doing it together, with media entries as the premium hook.
- Web: Stripe. App stores: StoreKit / Play Billing (required if distributed there).
- **Acceptance:** upgrade/downgrade/cancel flows work; entitlements enforce feature gates server-side (media upload rejected server-side for free accounts).

### 5.8 Settings, Privacy & Data (P0)
- Export my data (JSON + readable text), delete my account (full erasure), notification preferences, privacy policy & ToS links.
- Journaling data is sensitive personal content: encrypt at rest, never used for ads, no third-party sale. GDPR/CCPA deletion honored within 30 days.

---

## 6. Non-Goals for v1
- AI-generated prompts or sentiment analysis
- Android/iOS fully-native rebuild (ship PWA, wrap with Capacitor if store presence needed)
- Therapist/coach marketplace, DMs between users, video entries

---

## 7. Release Plan (summary — details in ROADMAP.md)
- **v0.2 (1–2 wks):** prototype hardening — timezone fix, entry viewer, icon set, deploy
- **v0.5 (4–6 wks):** accounts + backend sync + reminders (P0 set)
- **v0.8 (4 wks):** community + sharing + moderation
- **v1.0 (4 wks):** subscriptions, data export, legal, launch

---

## 8. Open Questions
1. ~~Can a user run a second 30-day challenge for free, or is repeat-run the paywall?~~ **Resolved July 2026: repeat runs are free forever; the Family plan is the only paywall (see §5.7).**
2. Family plan: shared challenge (same start date) or independent challenges with a shared feed?
3. Certificate: real name vs. display name?
4. App store presence at launch, or PWA-only until traction?
