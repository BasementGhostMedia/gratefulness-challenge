# The Gratefulness Challenge — Claude Code Handoff

## What This Is

A mobile app prototype built as a single self-contained HTML file, plus a small PWA shell. It simulates a complete interactive app experience in the browser — no framework, no build step, no server needed beyond a static file server (required for the service worker to register). Just serve the folder and open the HTML file in Chrome.

**File:** `index.html` is the real, canonical app file. `gratefulness-challenge-app.html` is a thin redirect stub (kept for old links/bookmarks) that meta-refreshes to `index.html` — don't edit the app in that file, it has no real content.
**Logo:** `logo.png` — referenced by the HTML (`<img>` on splash/home), `manifest.json`, and `sw.js`. Sized icon variants live in `icons/` (see PWA Support below); `apple-touch-icon` and the manifest point at those, not the raw `logo.png`.

---

## App Concept

"30 Days to a New You" — a 30-day gratitude journaling challenge app.

Users complete a daily prompt for 30 consecutive days. They earn XP, build streaks, unlock badges, grow a virtual tree avatar, and can share entries with a community feed. The core app is free forever; the only paid tier is a Family plan (monthly or annual) that adds family features and unlocks media entries.

---

## Brand

| Token | Value |
|-------|-------|
| `--yellow` | `#FCCC06` |
| `--teal` | `#00C2CB` |
| `--black` | `#000000` |
| `--white` | `#FFFFFF` |
| Font | Montserrat (Google Fonts), weights 400–900 |
| Icons | SVG stroke line art via `<symbol>`/`<use>` — no emoji, no filled icons |
| Logo | `logo.png` — used on splash screen and home avatar |

---

## Screen Structure (9 screens)

| ID | Screen | Key Content |
|----|--------|-------------|
| `s-splash` | Splash | Logo, "30 Days To A New You", Start / Sign In CTAs |
| `s-onboard` | Onboarding | 3-step how-it-works cards |
| `s-home` | Home | Greeting, streak pill, XP badge, Today's Prompt card, 30-day progress dots, tree avatar, recent entries list, badge previews, bottom nav |
| `s-journal` | Journal | Back button, today's prompt, reflection textarea, 3 gratitude inputs, photo/voice/video media buttons, Save button |
| `s-badges` | Badges | XP bar, 12-badge grid (earned vs. locked, computed from real state) |
| `s-community` | Community | Feed of user posts with likes and replies |
| `s-profile` | Profile | Tree avatar, stats (streak/XP/badges), upgrade banner, settings menu rows |
| `s-subscribe` | Plans | Free (forever, full core app), Family $9.99/mo, Family Annual $99.99/yr |
| `s-complete` | Day Complete | Celebration screen after saving a journal entry, XP earned, share buttons |

---

## Navigation System

All navigation uses a single `go(id)` JavaScript function. It handles:
- Animated slide transitions (forward slides left, back slides right)
- Touch swipe left/right between screens
- Nav dot indicator updates below the phone shell

```js
go('s-home')      // navigate to any screen by ID
```

**Critical:** All `<svg>` elements inside clickable buttons/nav items have `pointer-events:none` — this is intentional and required so taps reach the parent element's `onclick` handler.

---

## Icon System

Icons are defined as SVG `<symbol>` elements in a hidden `<svg>` block at the top of `<body>`. Reference them anywhere with:

```html
<svg width="24" height="24" style="color:var(--teal);pointer-events:none">
  <use href="#i-home"/>
</svg>
```

Available icon IDs: `i-home`, `i-pen`, `i-trophy`, `i-heart`, `i-person`, `i-flame`, `i-sprout`, `i-tree`, `i-star`, `i-sun`, `i-wave`, `i-people`, `i-mic`, `i-camera`, `i-video`, `i-gear`, `i-bell`, `i-crown`, `i-book`, `i-award`, `i-share`, `i-check`, `i-img`, `i-chat`, `i-back`, `i-right`, `i-wifi`, `i-brain`, `i-smile`, `i-sparkle`

---

## Game Mechanics — implemented with real state, not just UI mockups

State lives in `localStorage` under the key `gc-state` (see `defaultState()`, `loadState()`, `saveState()` in the `<script>` block):

- **XP System:** 50 XP per entry (`saveEntry()`), running total shown on Home, Badges, and Profile.
- **Streaks:** Consecutive-day count computed in `saveEntry()` by comparing `lastEntryDate` to yesterday's date; resets to 1 if a day is missed. Shown with the flame icon.
- **Tree Stages:** `TREES` array — Seed → Seedling → Sapling → Tree → Golden Tree, gated on total entry count (0 / 1 / 8 / 16 / 25).
- **12 Badges:** `BADGES` array, each with a `check(state)` predicate evaluated on every save: First Light, On Fire, Mind Full, Flow State, Golden, Champion, Golden Tree, Family, Share Joy, Voice, Snapshot, Legend.
- **30 daily prompts:** `PROMPTS` array, indexed by entry count, cycling through Day 1–30 content.
- **Pricing (display only, not wired to payments):** the core app is **free forever** (full challenge, streaks/XP/badges/tree, community, repeat runs). The only paid tier is **Family — $9.99/mo or $99.99/yr** (up to 5 members, private family feed, shared progress) which also unlocks the premium features: photo/voice/video entries (journal media buttons carry a crown badge and a `familyFeature()` toast), streak freeze, printable certificate.

`updateUI()` re-renders Home, Journal, Profile, and Badges from `state` on load and after every save. Saving is once-per-day (`alreadyDoneToday()` blocks a second entry the same day).

---

## PWA Support

- `manifest.json` — installable web-app manifest (name, theme color `#FCCC06`), with `icons/icon-192.png` / `icon-512.png` (purpose `any`) and `icon-192-maskable.png` / `icon-512-maskable.png` (purpose `maskable`, logo scaled to ~70% with white padding so Android's circular mask doesn't clip it). Regenerate with `sips` from `logo.png` if the logo changes — see git history for the exact commands.
- `sw.js` — service worker with a **network-first** strategy for navigations (so a new deploy is visible immediately) and **cache-first** for static assets (logo, icons). Registers via `navigator.serviceWorker.register('./sw.js')` at the bottom of the HTML's `<script>`.
- **Bump the `CACHE` version string in `sw.js` on every deploy that changes cached assets** — otherwise returning visitors can get served a stale cache. (This bit us once already — see git log.)
- **Service worker registration requires the file to be served over `http(s)://`** — it will silently fail to register when opened directly via `file://`. Use a static server (e.g. `npx serve`, `python3 -m http.server`) to test PWA behavior.
- The service worker deliberately **never caches `*.supabase.co` requests** — API data must stay live.

---

## Backend (Supabase) — added in v0.5

Project: `gratefulness-challenge` (ref `uemgwlrimezwzrsubxdb`, us-east-1, free tier) in "BasementGhostMedia's Org". The client config (URL + publishable key, safe to embed) is in the `── BACKEND (Supabase) ──` section of the app's `<script>`; supabase-js v2 is loaded via a pinned jsdelivr UMD `<script>` in `<head>`.

**Two modes:**
- **Guest (no account):** identical to the old behavior — everything in `localStorage` (`gc-state`), client-side streak/XP/badge logic (`localApplySave()`).
- **Signed in:** server-authoritative. `saveEntry()` calls the `save_entry` RPC; streak/XP/badges/once-per-day are computed in Postgres and the client applies the returned truth (`applyServerSave()`). Boot hydrates from the server (`loadRemote()`); `localStorage` is just an offline cache. Entries saved while offline queue in `gc-pending` and flush on reconnect (`flushPending()`).

**Schema (all RLS-protected):**
- `profiles` — 1:1 with `auth.users` (auto-created by trigger `handle_new_user`): name, streak, xp, last_entry_date, badges_earned[], shares, invites, challenge_count. Clients can only directly UPDATE the `name` column (column-level grant); all stats change only via RPCs.
- `entries` — day (1–30), entry_date, prompt, reflection, gratitudes jsonb. `UNIQUE(user_id, entry_date)` + `UNIQUE(user_id, day)`. Clients can only SELECT their own; INSERT/UPDATE/DELETE are revoked entirely (immutable journal, writes only through RPCs).
- RPCs (SECURITY DEFINER, scoped to `auth.uid()`, authenticated-only): `save_entry(p_local_date, p_prompt, p_reflection, p_gratitudes)` and `migrate_guest_data(p_name, p_entries)` (one-time guest import, refuses if the account already has entries). `compute_badges(uuid)` is internal (not callable by clients).
- Security advisors show two expected WARNs (the two RPCs are intentionally client-callable SECURITY DEFINER functions — that's the design since direct table writes are revoked).

**Auth:** email + password only. **"Confirm email" is currently ON** (Supabase default) — real users get a confirmation email before they can sign in; the client shows "Check your email" after signup. Toggle it off in Dashboard → Authentication for instant beta signups (test accounts were confirmed via SQL). Guest→account migration: on first sign-in, if local entries exist and the account is empty, they're pushed via `migrate_guest_data` and the old local state is kept at `gc-state-backup`.

---

## Current State

- All 9 screens built and navigable
- Navigation works (click, swipe, nav dots, back buttons)
- Brand colors and Montserrat font applied throughout
- Line-art SVG icons throughout (no emoji)
- Real logo (`logo.png`) referenced on splash and home avatar; proper multi-size + maskable app icons in `icons/`
- **Real data persistence via `localStorage`** — streaks, XP, entries, and earned badges survive a reload
- 30-day prompt rotation, badge-earning logic, and tree-stage progression are functionally complete
- **Dates are computed in local time** (`localDateStr()`), not UTC — streaks and once-per-day gating no longer break near midnight for users outside UTC
- **Real accounts + cross-device sync (v0.5):** email/password sign-in via Supabase, server-authoritative streak/XP/badges, guest→account data migration, offline save queue — see the Backend section
- Past entries are viewable read-only via a detail modal (`viewEntry(day)` / `closeEntryModal()`), triggered from Home's Recent Entries and Profile's "Past Journals" row
- Every not-yet-built control (media buttons, share buttons, most Profile menu rows, Subscribe's CTA) shows a "Coming soon" toast (`comingSoon()` / `toast()`) instead of silently doing nothing
- Community feed and "Recent Entries" beyond the current session are still hardcoded/sample data

---

## What's Not Built Yet

- Actual media capture (camera/mic/video) — buttons show a "coming soon" toast
- Google/Apple OAuth sign-in (email+password works; see Backend section)
- Push notifications
- Real community feed (live/shared data — currently static sample posts)
- Sharing / certificate generation
- Payment processing for the Family plan (plan selection UI only, CTA shows "coming soon")
- App store submission

See `docs/PRD.md`, `docs/ROADMAP.md`, and `docs/LAUNCH-CHECKLIST.md` for the full phased plan to v1.0 — this file only tracks the current prototype's implementation, not the roadmap.

---

## How to Run

1. Serve this folder with a static file server (needed for the service worker to register) — e.g. `npx serve` or `python3 -m http.server`, then open `index.html` (or just the root URL) in Chrome.
   - Opening the file directly via `file://` also works for the UI/nav/localStorage, but the service worker will not register.
2. The phone shell renders at 390×844px centered on the page.
3. Click any nav item, button, or use the dots below the phone to navigate.
4. Swipe left/right on touch screens.

No build step, no npm install required.

---

## File Layout

```
The Gratefulness App 0.1 2026/
├── index.html                         ← entire app (HTML + CSS + JS in one file) — edit this one
├── gratefulness-challenge-app.html     ← redirect stub to index.html (legacy filename, don't edit)
├── manifest.json                       ← PWA manifest
├── sw.js                               ← service worker (network-first HTML, cache-first assets)
├── logo.png                            ← brand logo source (flat RGB, white background)
├── icons/                              ← generated icon sizes (192/512, incl. maskable variants)
├── docs/                               ← PRD.md, ROADMAP.md, LAUNCH-CHECKLIST.md (path to v1.0)
├── sessionhandoff.docx                 ← original brief with full feature spec
└── CLAUDE.md                           ← this file
```

**Note:** this working copy is a git repo pushed to `github.com/BasementGhostMedia/gratefulness-challenge` and deployed live via GitHub Pages. A separate copy lives at `~/Documents/Claude/Projects/The Gratefulness Challenge/` (uses the original `gratefulness-challenge-app.html` filename directly, no redirect stub) — keep both in sync when making changes.

---

## Prompting Tips for Claude Code

- "Add a [screen name] screen" — it should follow the existing `<div class="screen" id="s-xxx">` pattern
- "The nav on [screen] is broken" — check that `pointer-events:none` is on all SVGs inside clickable elements
- "Add real data persistence" — already done via `localStorage` (`gc-state`); extend the existing `state` object and `updateUI()` rather than introducing a new storage layer
- "Make the [X] screen match the brand" — refer to the Brand section above for exact hex values and font weights
