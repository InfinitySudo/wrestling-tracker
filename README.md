# Wrestling Performance Tracker

> A multi-club PWA for wrestling coaches and athletes. Norm tracking, performance analysis, live UWW scoreboard, social profiles, share cards, payment-enabled training camps. Built for "Constant Wrestling" but designed to onboard any club.

**Stack:** React 19 · Vite · Tailwind v4 · Radix UI · TanStack Query · Framer Motion · Capacitor (iOS/Android shell) · React Router v7 · FastAPI · SQLite · Square Payments

> This is a **showcase repository** describing the architecture and engineering decisions of a private production codebase. Source code is not included.

---

## What it does

Coaches need to track athlete progression against age- and weight-class norms. Athletes want a profile they can show recruiters or share on social. Clubs want a leaderboard. Tournaments need a live scoreboard that follows UWW rules. This app gives all of that a single home.

- **Norms tracking** — push-ups, pull-ups, sprint times, technique benchmarks. Logged per athlete, plotted over time, compared against age/weight-class norms. Norm-attempt history table with regex-filtered numeric values (legacy data drifted to VARCHAR).
- **Performance analysis** — trend lines, percentile-vs-norm, regression alerts (athlete dropped from Tier-2 to Tier-3 in last 30 days).
- **Athlete profile** with socials, photo, weight class, club affiliation.
- **Share card** — `html2canvas`-rendered image of an athlete's stats, formatted for Instagram / Telegram.
- **QR check-in** for in-person events (`html5-qrcode`).
- **Multi-club** — clubs are tenants; an athlete can move clubs without losing history. Cross-club "People sheet" for super-admin firehose view.
- **PWA + Capacitor** — installable on phones, optional native iOS/Android shell.

### UWW-compliant live scoreboard (May 2026)
Heracles-style live table for tournament use:
- Period count-down with P shot-clock and P-strike counters.
- Match result codes: FALL, VSU (technical superiority), VPO (decision), INJ (injury time).
- Auto tech-superiority after a configurable point gap.
- Auto end after Period 2 with appropriate result code.
- Cast-to-TV mode for venue displays.

### Sparring live scoring + birthday daemon + i18n
- Sparring live scoring mode for practice tracking.
- Birthday daemon (cron) — auto-greets athletes in club Telegram.
- 9-language UI: en / ru / uk / pl / ar / fa / zh / ja / pa with RTL support for Arabic / Farsi. Header dropdown; sub-components use their own `useTranslation` hook.

### Payment-enabled training camps
- Square Checkout integration.
- Honor-system "I Paid / Confirm" flow as a fallback for athletes paying in cash or e-transfer (default link points to Constant Wrestling).

### Super-admin + role discipline
- `users.is_super_admin` flag for cross-club ops (currently 2 IDs).
- Password reset, user groups, club policy + signing flow.
- React `<select>` value mismatch gotcha documented (a select with a `value` that has no matching `option` silently renders the first option — UI lied about the user's role).

## Architecture

```
React 19 PWA ──► FastAPI backend (separate process) ──► SQLite
     │
     ├── TanStack Query (server-state cache)
     ├── React Router v7 (data routes)
     ├── Radix UI (accessible primitives)
     ├── Framer Motion (layoutId pill bottom-nav with auto-scroll-to-active)
     └── react-i18next (9 languages, RTL)

Capacitor wrapper ──► native iOS/Android build (`vite build && npx cap sync`)
Square Checkout ──► training-camp registrations
Telegram Bot API ──► birthday daemon, score broadcasts
```

## What's interesting

- **Domain-first design.** Wrestling has weird structure: weight classes are non-linear, norms vary by federation, athletes change classes mid-season. The data model holds historical class membership, not just current.
- **Offline-tolerant PWA.** Coaches at a tournament have flaky wifi. Service worker caches the last-seen athlete roster and norms reference data; logged measurements queue and sync when connectivity returns.
- **PWA stale-bundle recovery** — a hard lesson: don't auto-reload on `controllerchange` / `sw-activated`. React Query `onError` + 10-minute cooldown instead. Auto-reloads at the wrong moment lost coach input mid-tournament.
- **Share-card pipeline.** `html2canvas` is the one part that genuinely fights you — fonts, scaled images, gradient backgrounds. Documented the gotchas in the code.
- **Multi-tenant from day one.** Club is a first-class entity; a single user can coach in two clubs, an athlete can transfer.
- **Mobile-first nav.** Framer-motion `layoutId` pill bottom-nav with scroll, safe-area, and auto-scroll to the active item. The kind of detail tournament coaches notice within 10 seconds.

## Scale & status

- ~240 React components / hooks / pages.
- Active beta with multiple wrestling clubs.
- Capacitor build configured for both iOS and Android.
- Constant Wrestling branding as the default tenant; UI re-skins per club.

## What I'd build next

- Coach scheduling + attendance integrated with norms (so missing 3 sessions auto-flags a regression).
- Tournament-bracket integration (USA Wrestling / TrackWrestling import).
- Stripe Connect for club subscriptions ($9–49/club/mo).

---

## About this repo

This is a **portfolio showcase**. The production code, athlete data, and credentials are private. If you're a hiring manager, recruiter, or wrestling-club operator and want a deeper technical walkthrough — happy to do a live call.

**Author:** Artem Borysiuk — solo founder + wrestling coach who got tired of Excel.
