# 📑 SeshTab – Intelligent Tab Manager

[![Website Status](https://img.shields.io/website?url=https%3A%2F%2Fwww.getseshtab.com&style=for-the-badge&label=getseshtab.com)](https://www.getseshtab.com)

> **Stop losing tabs. Start owning your workspace.**
> SeshTab is a browser extension that saves, organizes, and syncs your tabs — with AI-powered clustering and a command palette built for power users.

---

## 🚀 About the Project

SeshTab is a **productivity-first browser extension** available for Chrome, Firefox, and Edge. It solves the tab hoarding problem by giving users structured workspaces, smart snoozing, and optional AI organization — without requiring an account to get started.

**Core capabilities:**

- **Workspaces** — save any browser window as a named tab group; reopen it any time.
- **Sessions** — snapshot all open windows at once; restore individual windows or full sessions.
- **Tab Snooze** — hide a tab now, have it reopen automatically at a chosen time.
- **Command Palette** — fuzzy-search-driven interface for every action (save, snooze, find duplicates, close all, switch profiles).
- **Duplicate Detection** — finds and closes exact-duplicate tabs across all windows.
- **Idle/Stale Tab Suggestions** — surfaces tabs you haven't visited in a while.
- **Favorite Tabs** — pin any tab for fast access without pinning it in the browser.
- **Recently-Closed Recovery** — browse and restore recently closed tabs.

**Pro-tier capabilities:**

- **Cloud Sync** — workspaces, sessions, and snoozed tabs synced across devices via Supabase.
- **AI Tab Clustering** — GPT-4o-mini groups your open tabs into suggested workspaces automatically.
- **Unlimited Workspaces, Sessions & Snoozed Tabs** — free tier is capped at 3/3/5.
- **Workspace Profiles** — save and restore entire workspace configurations; switch between them instantly.
- **Public Sharing** — share a workspace or profile via a public link.
- **90-Day Session History** — full history accessible from the web dashboard.

This repository is a **public documentation hub** describing how the platform is built and operated.

---

## 🛠 Tech Stack

For a full technical breakdown, see [ARCHITECTURE.md](ARCHITECTURE.md).

**Extension:**
- **Build Tool:** WXT (Web Extension Toolkit) — targets Chrome MV3 and Firefox MV2 from a single codebase
- **UI:** React 19 + TypeScript
- **Styling:** Tailwind CSS 4
- **Local Storage:** Dexie 4 (IndexedDB wrapper)
- **Cloud:** Supabase Auth + PostgreSQL (Pro users)

**Web App & Backend:**
- **Framework:** Next.js 15 (App Router, Server Components, API Routes)
- **Language:** TypeScript
- **Database:** Supabase PostgreSQL with Row Level Security
- **Auth:** Supabase Auth (email OTP + OAuth)
- **AI:** OpenAI GPT-4o-mini (tab clustering)
- **Payments:** Stripe (subscriptions + webhooks)
- **Email:** Resend (transactional, onboarding)
- **Hosting:** Vercel

👉 **[Read the full Architecture Overview](ARCHITECTURE.md)**

---

## 🗺 Roadmap (High Level)

**Already shipped**

- ✅ Workspace and session management (save, restore, name, color-code)
- ✅ Tab snooze with browser Alarms API and wake notifications
- ✅ Command palette with fuzzy search
- ✅ Duplicate detection and idle tab suggestions
- ✅ Favorite tabs and recently-closed recovery
- ✅ AI tab clustering (GPT-4o-mini, Pro only, rate-limited)
- ✅ Cloud sync across devices (Supabase, Pro only)
- ✅ Workspace Profiles — save/restore full workspace states
- ✅ Public sharing for workspaces and profiles (slug-based links)
- ✅ Web dashboard for managing everything outside the popup
- ✅ Stripe subscriptions with webhook-driven plan enforcement
- ✅ Supabase RLS for per-user data isolation
- ✅ OAuth bridge between web app and extension (MV3-compatible)

**Planned / in exploration**

- ⏳ Tab groups (native browser API) integration
- ⏳ Cross-device session handoff ("continue on another device")
- ⏳ Richer snooze scheduling (recurring, weekday-based)
- ⏳ Team/shared workspaces (collaborative tab collections)
- ⏳ Browser history–aware idle tab scoring

👉 **[Read the full Roadmap](ROADMAP.md)**

---

## 🐛 Bug Reporting & Feedback

If you've found an issue in the **public docs** or have suggestions about the platform:

- Open an **Issue** in this repository, or
- Reach out via [getseshtab.com](https://www.getseshtab.com)

> **Note:** The production application codebase is private. This repository does **not** accept feature PRs for the closed-source app.

---

## 📄 License

The content of this repository (documentation, architecture overview, roadmap) is licensed under the **MIT License**.

**Important:**
The **SeshTab source code**, branding, and core service logic are **proprietary and closed-source**.
This repository serves only as a **public documentation and communication hub** for the project.

[**Visit getseshtab.com →**](https://www.getseshtab.com)
