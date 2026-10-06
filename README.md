# VoiceFlow AI — Voice Agent Dashboard

A responsive admin dashboard for creating, configuring and monitoring AI voice agents. Built with React and Tailwind CSS as a **frontend-only demo**: all data is generated locally and persisted in the browser, so it runs with no backend.

> **Demo login:** `priya@northwind.io` / `demo1234`

## Features

- **Authentication (demo)** — login form with validation, protected routes, redirect back to the page you came from, and sign out.
- **Dashboard** — KPI cards with period-over-period trends, 14-day call chart, recent calls and per-agent performance.
- **Voice Agents** — create, edit, activate/deactivate and delete agents, with search, status filters, duplicate-name validation, confirmation dialogs and toast feedback.
- **Agent detail** — per-agent stats, call volume chart, recent calls and system prompt.
- **Call History** — search, agent / status / date filters, "load more" pagination, a call details modal with transcript, and **CSV export** of every call matching the current filters.
- **Analytics** — 7 / 30 / 90-day ranges, calls-over-time line chart, status donut and sortable agent table.
- **Settings** — editable profile, light / dark / system theme, notification preferences and a "reset demo data" action.
- **Global search & notifications** — topbar search across agents and calls, plus a notification feed that respects your preferences.
- **Polish & accessibility** — loading skeletons, empty and error states, an error boundary, a branded 404 page, and a layout that works from mobile to desktop. Dialogs trap and restore focus, search is a keyboard-navigable combobox, menus and radio groups support arrow keys, there is a skip-to-content link, and colours were checked for WCAG AA contrast in light and dark themes.

## Tech stack

React 19 · Vite · Tailwind CSS v4 · React Router v7 · lucide-react · Oxlint. Plain JavaScript, React Context for state, no other runtime dependencies.

## Getting started

Requires Node.js 20+.

```bash
npm install
npm run dev      # start the dev server
npm run build    # production build in dist/
npm run preview  # preview the production build
npm run lint     # run Oxlint
```

## Project structure

```
src/
  components/   ui/ (Card, Button, Modal, Toggle, …), layout/ (Sidebar, Topbar, menus),
                agents/, calls/, dashboard/, analytics/, auth/
  context/      Auth, Agents, Settings, Notifications and Toast providers (+ hooks)
  data/         Seed data: agents, call log, aggregated analytics, navigation, settings
  hooks/        useSimulatedLoading, useElementWidth, useDismiss
  layouts/      AuthLayout, DashboardLayout
  pages/        Login, Dashboard, Agents, AgentDetail, Calls, Analytics, Settings, NotFound
  utils/        analytics aggregation, agent metrics, filters, formatters, storage
```

## How the data fits together

There is a single source of truth: the **call log** (`src/data/callsData.js`), about six months of generated calls per agent.

- `analyticsData.js` aggregates that log into daily per-agent stats.
- `utils/agentMetrics.js` derives each agent's totals, success rate, trends and recent calls from those stats.
- Only agent *configuration* (name, voice, prompt, status, …) is stored and editable, so Dashboard, Agents, Calls and Analytics always agree — deleting an agent removes its calls from every page.
- Dates are anchored to a fixed "now" (`REFERENCE_NOW`) so the demo looks the same whenever you open it.

## Architecture & decisions

- **React Context instead of Redux.** The state is small and mostly independent (auth, agents, settings, notifications, toasts), so one provider per concern is simpler than a store. Each context lives in a plain `*.js` file with a `use…` hook, separate from its provider component, so Fast Refresh and the lint rule `only-export-components` stay happy.
- **One source of truth, derived everywhere else.** Only agent *configuration* is stored and editable. Metrics, charts, analytics and notifications are computed from the call log, so pages can't disagree and deleting an agent updates every view.
- **A fixed "now".** `REFERENCE_NOW` anchors all seeded dates and relative times ("2 hr ago", "Last 7 days"), which keeps the demo deterministic and screenshots reproducible.
- **Small hand-built SVG charts** (line chart, donut) instead of a charting library: no extra dependency, and they redraw to the container's real width via `useElementWidth`.
- **Browser storage behind one helper** (`utils/storage.js`) that namespaces keys and never throws. If storage is unavailable the app still works and shows a toast.
- **Loading is simulated in one place** (`useSimulatedLoading`) so it is obvious where a real data fetch would go.
- **Accessibility lives in shared components**: `Modal` (focus trap/restore, labelled dialog), `SegmentedControl` (radio group with arrow keys), `useDismiss` (outside click and Escape, returning focus to the trigger). Features reuse them rather than re-implementing behaviour.
- **Dark mode re-maps the Tailwind palette variables** in `index.css` instead of adding `dark:` variants to every component; a few overrides at the bottom of that file handle cases where a re-mapped colour would hurt contrast.

## Persistence

State is kept in `localStorage` under the `voiceflow:` prefix: `agents`, `settings` (profile, theme, notification preferences), `notifications` (read/dismissed) and `session`. Use **Settings → Reset demo data** to restore the defaults.

## Deployment

The app is a static SPA and deploys as-is to Vercel (`npm run build`, output `dist/`). `vercel.json` rewrites every path to `index.html`, so refreshing or deep-linking to a route such as `/agents` works. On other hosts, add the equivalent fallback (for Netlify, a `public/_redirects` file containing `/* /index.html 200`).

## Limitations

- Authentication is a UI gate only — credentials are hard-coded and nothing is verified server-side. Don't use it for real security.
- There is no backend or real telephony; "loading" delays are simulated.

## Ideas for next steps

Connect a real API and auth provider, add automated tests, route-level code splitting, server-side pagination for large call logs, and real-time call status.
