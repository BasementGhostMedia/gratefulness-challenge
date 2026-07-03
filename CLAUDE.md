# The Gratefulness Challenge — Claude Code Handoff

## What This Is

A mobile app prototype built as a single self-contained HTML file, plus a small PWA shell. It simulates a complete interactive app experience in the browser — no framework, no build step, no server needed beyond a static file server (required for the service worker to register). Just serve the folder and open the HTML file in Chrome.

**File:** `gratefulness-challenge-app.html`
**Logo:** `logo.png` — referenced by the HTML (`<img>` on splash/home, `apple-touch-icon`), `manifest.json`, and `sw.js`. *(Note: an earlier version of this doc called this asset `B&W logo. Transparent.png` — the code was updated to reference `logo.png` directly; keep using that filename.)*

---

## App Concept

"30 Days to a New You" — a 30-day gratitude journaling challenge app.

Users complete a daily prompt for 30 consecutive days. They earn XP, build streaks, unlock badges, grow a virtual tree avatar, and can share entries with a community feed. There's a subscription model with Individual, Family, and Annual plans.

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
| `s-subscribe` | Plans | Individual $9.99/mo, Family $19.99/mo, Annual $79.99/yr |
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
- **Subscription (display only, not wired to payments):** Individual $9.99/mo · Family $19.99/mo · Annual $79.99/yr.

`updateUI()` re-renders Home, Journal, Profile, and Badges from `state` on load and after every save. Saving is once-per-day (`alreadyDoneToday()` blocks a second entry the same day).

---

## PWA Support

- `manifest.json` — installable web-app manifest (name, theme color `#FCCC06`, icon `logo.png`).
- `sw.js` — service worker that caches `gratefulness-challenge-app.html` and `logo.png` for offline use. Registers via `navigator.serviceWorker.register('./sw.js')` at the bottom of the HTML's `<script>`.
- **Service worker registration requires the file to be served over `http(s)://`** — it will silently fail to register when opened directly via `file://`. Use a static server (e.g. `npx serve`, `python3 -m http.server`) to test PWA behavior.

---

## Current State

- All 9 screens built and navigable
- Navigation works (click, swipe, nav dots, back buttons)
- Brand colors and Montserrat font applied throughout
- Line-art SVG icons throughout (no emoji)
- Real logo (`logo.png`) referenced on splash and home avatar
- **Real data persistence via `localStorage`** — streaks, XP, entries, and earned badges survive a reload
- 30-day prompt rotation, badge-earning logic, and tree-stage progression are functionally complete
- Community feed and "Recent Entries" beyond the current session are still hardcoded/sample data

---

## What's Not Built Yet

- Actual media capture (camera/mic/video) — buttons are present but not wired
- User authentication
- Push notifications
- Real community feed (live/shared data — currently static sample posts)
- Sharing / certificate generation
- Payment processing for subscriptions (plan selection UI only)
- App store submission

---

## How to Run

1. Serve this folder with a static file server (needed for the service worker to register) — e.g. `npx serve` or `python3 -m http.server`, then open `gratefulness-challenge-app.html` in Chrome.
   - Opening the file directly via `file://` also works for the UI/nav/localStorage, but the service worker will not register.
2. The phone shell renders at 390×844px centered on the page.
3. Click any nav item, button, or use the dots below the phone to navigate.
4. Swipe left/right on touch screens.

No build step, no npm install required.

---

## File Layout

```
The Gratefulness Challenge/
├── gratefulness-challenge-app.html   ← entire app (HTML + CSS + JS in one file)
├── manifest.json                     ← PWA manifest
├── sw.js                             ← service worker (offline caching)
├── logo.png                          ← brand logo (PNG with transparency)
├── sessionhandoff.docx               ← original brief with full feature spec
└── CLAUDE.md                         ← this file
```

---

## Prompting Tips for Claude Code

- "Add a [screen name] screen" — it should follow the existing `<div class="screen" id="s-xxx">` pattern
- "The nav on [screen] is broken" — check that `pointer-events:none` is on all SVGs inside clickable elements
- "Add real data persistence" — already done via `localStorage` (`gc-state`); extend the existing `state` object and `updateUI()` rather than introducing a new storage layer
- "Make the [X] screen match the brand" — refer to the Brand section above for exact hex values and font weights
