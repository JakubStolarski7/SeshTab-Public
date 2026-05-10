# 🏗️ SeshTab — Technical Architecture

> **Public technical overview.**
> This document describes the architecture and technology choices of **SeshTab** for employers, contributors, and technical users.
> It does not expose business logic, proprietary AI prompts, or implementation details.

---

## 📋 Project Overview

**SeshTab** is a **browser extension + web app** for tab and session management with:

- **Workspace Manager** — save/restore named tab groups per window
- **Session Snapshots** — multi-window capture and restore
- **Tab Snooze** — time-based tab dismissal using browser Alarms API
- **AI Tab Clustering** — GPT-4o-mini groups open tabs into suggested workspaces
- **Cloud Sync** — real-time cross-device sync via Supabase (Pro)
- **Workspace Profiles** — collections of workspaces that can be switched and shared
- **Production-grade platform** — subscriptions, webhook-driven plan enforcement, transactional emails, RLS-backed data isolation

The product is monetized via a freemium model (free tier + Pro at $4/month via Stripe).
The extension runs on Chrome MV3 and Firefox MV2 from a shared codebase built with WXT.

---

## 🛠️ Tech Stack

| Layer | Technology | Rationale |
| :--- | :--- | :--- |
| **Extension Build** | **WXT** (Web Extension Toolkit) | Single-codebase builds for Chrome MV3 and Firefox MV2; HMR in dev, automated manifest generation. |
| **Extension UI** | **React 19 + TypeScript** | Component-driven popup UI; strong typing across the whole extension. |
| **Extension Styling** | **Tailwind CSS 4** | Utility-first styling, consistent design tokens across popup and web app. |
| **Local Storage** | **Dexie 4** (IndexedDB) | Typed schema, migrations, and a Promise-based API over raw IndexedDB. |
| **Web Framework** | **Next.js 15** (App Router) | Server Components, API Routes, Vercel-native deployment. |
| **Language** | **TypeScript** | End-to-end type safety across extension, web app, and API routes. |
| **Database** | **PostgreSQL** (Supabase) | Managed Postgres with Row Level Security for per-user data isolation. |
| **Auth** | **Supabase Auth** | Email OTP + OAuth; custom browser-storage adapter for MV3 service workers. |
| **Payments** | **Stripe** | Subscriptions, webhook-driven plan status, upgrade/downgrade flows. |
| **AI** | **OpenAI GPT-4o-mini** | Tab clustering — lightweight model balances cost and response quality. |
| **Email** | **Resend** | Transactional and onboarding emails. |
| **Hosting** | **Vercel** | Next.js-optimized, API routes, Cron Jobs. |

**Key Libraries**

- `dexie` — IndexedDB ORM for local-first storage.
- `@supabase/supabase-js` — client-side auth and database access.
- Custom browser storage adapter — allows Supabase SDK to work inside MV3 service workers (which have no `localStorage`).
- In-memory rate limiter (`Map`-based, per-instance) on the API routes.

---

## 🔄 High-Level Architecture

The platform has two surfaces:

1. **Browser Extension** — the primary user-facing product (popup + background service worker).
2. **Web App** — dashboard, auth bridge, API backend, public share pages.

### Monorepo Layout

```
SeshTab/
├── apps/
│   ├── extension/          # WXT extension
│   │   ├── entrypoints/    # background.ts, popup/, content.ts
│   │   └── utils/          # Business logic (db, sync, auth, snooze, …)
│   └── web/                # Next.js 15 app
│       ├── src/app/        # Pages, API routes
│       └── src/lib/        # Supabase, Stripe, utilities
├── supabase/               # DB migrations
└── package.json            # npm workspaces root
```

---

### 1. Extension Architecture

**Entrypoints**

| File | Role |
|------|------|
| `background.ts` | MV3 service worker: tab/window lifecycle, snoze alarms, OAuth message bridge, auto-snapshot debounce |
| `popup/App.tsx` | React UI: command palette, workspace/session/profile CRUD, AI clustering, auth state |
| `content.ts` | Content script (minimal, reserved for future web-page interaction) |

**Local Database (Dexie / IndexedDB)**

Six tables:

| Table | Contents |
|-------|----------|
| `sessionSnapshots` | Auto-captured window/tab state (debounced every 2 s) |
| `workspaces` | Named tab groups saved by the user |
| `snoozedTabs` | Tabs queued for wake-up with alarm ID |
| `savedSessions` | Multi-window snapshots |
| `favoriteTabs` | Starred tabs |
| `profiles` | Workspace collections (Pro) |

**Tab Snooze Flow**

1. User selects a tab and chooses a wake time in the popup.
2. Extension stores the tab record in `snoozedTabs` and creates a browser alarm (`seshtab-snooze-{id}`).
3. Background worker listens for `onAlarm`; when it fires it opens the tab URL and removes the record.
4. Cloud sync (`pushSnoozedTab`) mirrors the record to Supabase for Pro users.

**Auto-Snapshot Flow**

- Background worker registers listeners for `tabs.onCreated`, `onRemoved`, `onUpdated`, `windows.onFocusChanged`.
- Each event triggers a debounced (2 s) `captureSnapshot()` that writes the current window/tab state to `sessionSnapshots`.
- Pro users also get the snapshot pushed to Supabase (`pushSessionSnapshot`).

---

### 2. Web App Architecture

**Key Routes**

| Route | Purpose |
|-------|---------|
| `/` | Landing page |
| `/login`, `/signup` | Supabase Auth UI |
| `/auth/extension-callback` | OAuth bridge: web → extension handoff |
| `/dashboard` | Main hub with workspace overview |
| `/dashboard/workspaces` | Manage all cloud workspaces |
| `/dashboard/snoozed` | View active snoozed tabs |
| `/dashboard/sessions` | 90-day session history (Pro) |
| `/dashboard/profiles` | Manage workspace profiles (Pro) |
| `/dashboard/billing` | Upgrade / manage subscription |
| `/shared/[slug]` | Public workspace share page |
| `/p/[slug]` | Public profile share page |
| `/pricing` | Free vs Pro comparison |

**API Routes**

| Route | Purpose |
|-------|---------|
| `POST /api/cluster` | AI tab clustering (Pro, rate-limited at 20 req/min) |
| `POST /api/checkout/stripe` | Create Stripe checkout session |
| `POST /api/webhooks/stripe` | Handle Stripe events (subscription lifecycle) |
| `POST /api/workspaces/save-shared` | Save a public workspace into user's local DB |
| `POST /api/cron/onboarding` | Trigger onboarding email sequence |

---

### 3. AI Tab Clustering Flow

1. User triggers "ai organize" in the command palette (Pro only).
2. Extension calls `POST /api/cluster` with the list of open tab titles and URLs, plus the user's Supabase access token.
3. API route:
   - Validates auth and Pro plan via Supabase.
   - Checks in-memory rate limit (20 req/min per user).
   - Sends the tab list to OpenAI GPT-4o-mini with a structured prompt.
   - Returns clusters as JSON (group name + tab indices + confidence score).
4. Popup renders the suggestions; user approves and saves chosen groups as new workspaces.

---

### 4. Sync Architecture (Pro)

**Push flows** (extension → Supabase)

| Flow | Trigger |
|------|---------|
| `pushSessionSnapshot` | Auto, debounced on tab/window change |
| `pushInitialData` | On first login |
| `pushFullSync` | Manual "Sync from Extension" action |
| `pushWorkspace` | On workspace create/update |
| `pushSnoozedTab` | On snooze |
| `pushProfile` | On profile create/update |

**Pull flows** (Supabase → extension)

| Flow | Trigger |
|------|---------|
| `pullWorkspacesFromSupabase` | Dashboard restore action, popup mount |
| `purgeSnoozedTabOrphans` | On popup mount — cleans up cloud-deleted records |

Soft-delete (`deleted_at` timestamp) on workspaces allows dashboard recovery without permanent loss.

---

### 5. Auth & OAuth Bridge

SeshTab runs in Chrome MV3 where service workers have no `localStorage`. A custom Supabase storage adapter persists the session in `browser.storage.local` instead.

**Login flow:**

1. Extension opens `https://www.getseshtab.com/login?next=/auth/extension-callback?ext_id={runtime.id}`.
2. User authenticates (email OTP or OAuth).
3. Web app redirects to `/auth/extension-callback` with session cookies.
4. Page calls `chrome.runtime.sendMessage(ext_id, { type: 'SESHTAB_AUTH_SUCCESS', session })`.
5. Background worker stores the session via the custom adapter.
6. Supabase SDK handles token refresh automatically on future requests.

---

### 6. Subscription & Plan Enforcement

**Stripe Webhook events handled:**

| Event | Action |
|-------|--------|
| `checkout.session.completed` | Set user plan to `pro` in Supabase `profiles` |
| `customer.subscription.updated` | Downgrade to `free` if status is canceled/unpaid |
| `customer.subscription.deleted` | Downgrade to `free` |

**Free-tier hard limits (enforced locally + via Supabase):**

| Resource | Free | Pro |
|----------|------|-----|
| Workspaces | 3 | Unlimited |
| Snoozed tabs | 5 | Unlimited |
| Saved sessions | 3 | Unlimited |
| Cloud sync | ✗ | ✓ |
| AI clustering | ✗ | ✓ |
| Profiles | ✗ | ✓ |
| Public sharing | ✗ | ✓ |
| Session history | — | 90 days |

---

## 🔒 Security & Data

- **RLS:** Supabase Row Level Security ensures users can only read/write their own rows.
- **Stripe:** Webhook signature validated on every event; `client_reference_id` = Supabase user ID.
- **Privileged URLs:** Extension filters out `chrome://`, `chrome-extension://`, `data:`, and `javascript:` URLs before opening or saving.
- **CORS:** Extension origin whitelist restricts `/api/cluster` to `https://www.getseshtab.com` only.
- **Rate limiting:** AI clustering and Stripe checkout routes guarded per-user; prevents cost amplification.
- **Code length guards:** Not applicable here (no code execution), but tab-list payload sizes are bounded.

---

## 📁 Configuration Types (Example)

*Note: This is an illustrative TypeScript interface. Actual business logic is private.*

```typescript
// Example: how plan limits are typed and checked
type PlanSlug = "free" | "pro";

interface PlanLimits {
  maxWorkspaces: number | "unlimited";
  maxSnoozed: number | "unlimited";
  maxSessions: number | "unlimited";
  cloudSync: boolean;
  aiClustering: boolean;
  profiles: boolean;
  sharing: boolean;
}
```

This architecture continues to evolve, but the core principles remain:
- **Local-first** — full offline capability on the free tier
- **Privacy-conscious** — cloud sync is opt-in, no tab data sent anywhere without user action
- **Cost-aware** — AI calls are Pro-gated, rate-limited, and use a lightweight model
- **MV3-compatible** — no persistent background pages; everything works in a service worker
