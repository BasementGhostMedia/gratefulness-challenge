# Gap Analysis & Technical Roadmap — The Gratefulness Challenge

**Date:** July 3, 2026 · Companion to [PRD.md](PRD.md)

## 1. Where the App Stands

The v0.1 prototype ([index.html](../index.html), 1,486 lines) is genuinely functional as a local, single-user experience: all 9 screens, navigation, and the full game loop (prompts → entry → XP → streak → badges → tree) run on real `localStorage` state, with a PWA manifest and service worker. What separates it from "an app that works" for real users falls into three buckets: **bugs in what exists**, **missing product infrastructure**, and **launch requirements**.

## 2. Bugs & Fixes in the Current Prototype (do these first — no backend needed)

| # | Issue | Where | Impact |
|---|-------|-------|--------|
| 1 | **Timezone bug:** `todayStr()` uses `toISOString()` = UTC date, while `greeting()` uses local time. A user in New York journaling at 9pm gets credited for *tomorrow* (UTC); once-per-day gating and streaks break around the UTC midnight boundary. | `index.html:1352`, streak logic `:1452-1454` | High — corrupts the core mechanic. Fix: derive dates from local time everywhere. |
| 2 | **Service worker is cache-first for everything** — after v1 is cached, users may never receive updated HTML. Also calls `cache.put()` on all requests; non-GET requests will throw. | `sw.js:22-30` | High once you start shipping updates. Fix: network-first (or stale-while-revalidate) for `index.html`; skip non-GET; keep cache-first for static assets. |
| 3 | **No way to read a past entry** — Home lists recent entries but tapping does nothing; users can't reread their own journaling. | Home screen | Medium — undermines the product's value. Add an entry-detail view. |
| 4 | **Dead controls** mislead testers: Sign In, media buttons, share buttons, settings rows, plan "Choose" buttons. | Various | Low — either wire to a "coming soon" toast or hide behind a flag. |
| 5 | **Manifest has a single 512px icon**; installability and iOS home-screen quality suffer. | `manifest.json` | Low. Add 192/512 + maskable variants and iOS splash. |
| 6 | **CLAUDE.md is stale**: `gratefulness-challenge-app.html` is now a redirect; the app lives in `index.html`. | docs | Trivial — update. |

## 3. Missing Infrastructure (prototype → product)

1. **Backend & accounts.** Everything is client-side; state is one device, wipeable by clearing browser data, and fully forgeable (streaks/XP are client-computed). Needs: auth, database, server-authoritative streak/XP/badge logic, offline-first sync.
   **Recommendation: Supabase** (already connected to your tooling) — Postgres + Auth (email/Apple/Google + anonymous guest sessions that upgrade in place), Row Level Security for private journal entries, Storage for photos, Edge Functions for streak computation and moderation hooks. Keep the vanilla-JS front end; add a small `api.js` layer — no framework rewrite required.
2. **Community feed.** Static sample posts → real shared entries with likes/replies, opt-in sharing, and moderation (report/block/filter). RLS makes "private by default" enforceable at the database layer.
3. **Push notifications.** The reminder is arguably the #1 retention feature. Web Push (VAPID) from an Edge Function cron; native push later via Capacitor wrapper.
4. **Media capture.** Photo first (file input + camera capture → Supabase Storage, compressed client-side). Voice/video later — they carry big storage/moderation costs.
5. **Payments.** Stripe Checkout + customer portal for web; entitlements table checked server-side. If distributing via app stores, StoreKit/Play Billing are mandatory for digital goods — plan for RevenueCat to unify both.
6. **Sharing/certificate.** Canvas-render a branded certificate image client-side; native share sheet. No backend needed — cheap win.

## 4. Phased Plan

**Phase 0 — Prototype hardening (1–2 weeks) → v0.2**
Fix items 1–6 above · entry-detail viewer · deploy to real hosting (GitHub Pages works today; Netlify/Vercel later) · basic analytics (e.g. Plausible/PostHog) so beta behavior is measurable.

**Phase 1 — Backend & accounts (4–6 weeks) → v0.5**
Supabase project + schema (`users`, `entries`, `challenges`, `badges_earned`, `subscriptions`) · auth incl. guest→account migration of localStorage state · server streak/XP · offline queue + sync · daily reminder Web Push. *Exit: same account works on two devices; closed beta of 20–50 users.*

**Phase 2 — Community & sharing (4 weeks) → v0.8**
Opt-in share to feed, likes, replies · report/block/moderation queue · streak-milestone share cards + Day-30 certificate · photo attachments. *Exit: feed is safe enough for strangers.*

**Phase 3 — Monetization & launch (4 weeks) → v1.0**
Stripe subscriptions + entitlement gates · data export & account deletion · privacy policy/ToS · landing page · App-store wrap via Capacitor **only if** store presence is a launch requirement, else PWA-only. *Exit: public launch.*

## 5. Decisions Needed Before Phase 1
1. PWA-only launch vs. app stores (changes payments and push architecture).
2. Confirm Supabase as backend (recommended above).
3. Answers to PRD Open Questions §8 (repeat-challenge paywall, family-plan mechanics).
