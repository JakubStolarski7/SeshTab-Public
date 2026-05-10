# 🗺️ SeshTab Roadmap & Vision

> **Status:** Active Development
> **Focus:** From "tab saver" → full **workspace OS for the browser** (Sessions + Snooze + AI + Cloud).

This document outlines where SeshTab is **today** and where it is heading.
The original "save tabs" idea has evolved into a full productivity layer on top of the browser.

---

## ✅ Phase 1: Foundation (Completed)

*Build the core extension infrastructure.*

- [x] **Local Workspace Manager** — save and restore named tab groups, no account required.
- [x] **Session Snapshots** — multi-window capture and restore.
- [x] **Tab Snooze** — browser Alarms API integration, wake tabs at a chosen time.
- [x] **Command Palette** — fuzzy-search-driven access to every feature.
- [x] **Duplicate Detection** — find and close exact-duplicate tabs across all windows.
- [x] **Idle/Stale Tab Suggestions** — surface tabs that haven't been visited recently.
- [x] **Favorite Tabs** — pin frequently-used tabs for fast access.
- [x] **Recently-Closed Recovery** — restore tabs closed in the current session.
- [x] **Free Tier Limits** — 3 workspaces / 3 sessions / 5 snoozed tabs enforced locally.

---

## ✅ Phase 2: Cloud & Pro Platform (Current Core — Shipped)

*From local-only → synced, AI-assisted, shareable.*

### ☁️ 1. Cloud Sync & Auth

- [x] **Supabase Auth Integration** — email OTP + OAuth; MV3-compatible via custom storage adapter.
- [x] **OAuth Bridge** — seamless web → extension session handoff without re-login.
- [x] **Real-Time Cloud Sync (Pro)** — workspaces, sessions, and snoozed tabs synced across devices.
- [x] **Auto-Snapshot Push** — debounced tab/window state pushed to Supabase on change.
- [x] **Soft Delete & Recovery** — deleted workspaces recoverable from the web dashboard.

### 🤖 2. AI Tab Clustering

- [x] **AI Organize (Pro)** — GPT-4o-mini clusters open tabs into suggested workspaces.
- [x] **Structured Output** — clusters returned as JSON with group names and confidence scores.
- [x] **Rate Limiting** — 20 AI requests/min per user to control costs.
- [x] **User Approval Flow** — suggestions shown in popup; user chooses which groups to save.

### 🗂️ 3. Workspace Profiles & Sharing

- [x] **Workspace Profiles (Pro)** — save entire workspace configurations; switch between them instantly.
- [x] **Public Workspace Sharing** — share a workspace via a slug-based public URL.
- [x] **Public Profile Sharing** — share a profile (workspace collection) via a public URL.
- [x] **Save-from-Share** — visitors can import a shared workspace into their own local storage.

### 💳 4. Subscriptions & Billing

- [x] **Stripe Integration** — subscription checkout, upgrades, downgrades.
- [x] **Webhook-Driven Plan Enforcement** — plan status kept in sync via Stripe events.
- [x] **Web Dashboard** — manage workspaces, sessions, snoozed tabs, profiles, and billing outside the popup.
- [x] **90-Day Session History (Pro)** — full history accessible from the dashboard.

### 📬 5. Emails & Automation

- [x] **Onboarding Email** — triggered via cron on new signup.
- [x] **Transactional Emails** — subscription events via Resend.

---

## 🔮 Phase 3: Richer Browser Integration (Planned)

*From "manages tabs" → "understands your workflow".*

- [ ] **Native Tab Groups Integration** — read and write Chrome/Firefox native tab groups alongside workspaces.
- [ ] **Cross-Device Session Handoff** — "continue on another device" — push the current window to another machine.
- [ ] **Recurring Snooze** — snooze a tab to repeat daily, weekly, or on specific weekdays.
- [ ] **Browser History–Aware Idle Scoring** — smarter stale-tab detection using visit frequency.
- [ ] **Keyboard-First Navigation** — full keyboard control of the popup without mouse.

---

## 🔮 Phase 4: Collaboration & Teams (Exploration)

*From "personal productivity" → "shared workspaces".*

- [ ] **Team Workspaces** — real-time shared workspaces for small teams or pair-programming sessions.
- [ ] **Classroom / Bootcamp Mode** — instructor shares a workspace set; students clone it with one click.
- [ ] **Richer Sharing Controls** — view-only vs. editable shared workspaces; expiry links.
- [ ] **Activity Feed** — see which shared workspace tabs teammates are visiting.

---

## 📝 How to Use This Roadmap

This roadmap is **deliberately high-level** and covers:

- What is **already in production**.
- What is **being iterated on** for better UX, reliability, and performance.
- What is **planned** but not committed to specific dates.

If you have ideas, questions, or feature requests, open an Issue or reach out via [getseshtab.com](https://www.getseshtab.com).
