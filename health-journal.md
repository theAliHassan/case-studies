# Health Journal

A single-user personal tracker, built as a web app that installs on a phone. It logs meals, estimates macros with an AI model, and charts progress.

**Stack:** Next.js (App Router), TypeScript, Supabase (Postgres, Auth, Storage), Google Gemini, Tailwind, shadcn/ui, Recharts, Vitest. **Code:** private. **Built:** 14–25 June 2026 (178 commits, about 19,000 lines of TypeScript, 26 database migrations).

## The problem

Tracking food meant a daily loop: describe a meal to an AI chat, copy the macros out, add them to a running total, and repeat. The app turns that into one flow: type or photograph a meal, review the estimate, save, and see today's totals.

## Design

- **Server-first.** Server components render the dashboard; client components handle the logging form, the chat and the charts. Route handlers form the API layer.
- **One AI wrapper.** Every model call goes through a single provider interface, so the model is swappable and its name comes from configuration, never from code.
- **Prompts are data.** The prompts that drive meal analysis, chat and weekly summaries are database rows, editable from a settings screen with version history.
- **Ownership by design.** All data lives in the user's own Supabase project, with row-level security on its tables. There is no third-party analytics, and the app has export and import.
- **Append-only history** for targets and goals, using database triggers, so past totals are never silently rewritten.
- **A written decision trail.** 48 numbered decision records (ADRs) explain the choices and the trade-offs, and a "backlog" file stops new ideas from creeping into the current scope.

## Decisions that mattered

- **Cut the first version to a short list of core features.** Meal logging, a macro dashboard, AI chat and progress charts, among others. Everything else went to a backlog, and later work was driven by problems found along the way.
- **One AI call per meal.** Early on, meal parsing used a regular-expression matcher plus a second AI call to scale saved favorite foods. The matcher failed on word order and could match one input to several favorites. I replaced both with a single call that receives the favorites as context, and the server validates the match it returns. The trade-off is that every meal analysis now costs one model call, which is negligible for one user and would need caching at scale.
- **Treat prompt examples as data that can be wrong.** A worked example in a prompt contained wrong macro math, and the model copied it. I fixed it through a migration that versions the prompt, and added a rule about scaling math.
- **Make failures traceable.** Structured logging with a request ID on every server action, plus retries with backoff and a friendly "busy" state when the AI service returns a 503.
- **Correct dates by construction.** All timezone arithmetic is centralized in one module, because timezone handling is easy to get subtly wrong.

## Results and limits

- Core flow works end to end: text and photo meal logging, editable AI estimates, a daily macro dashboard, weekly AI summaries, progress charts and data export.
- AI macro estimates are estimates, and the app shows a disclaimer on every one. I have not measured their accuracy against laboratory values.
- Automated tests are light (two test files), so most verification has been manual testing.
- It is built for one user. There is no multi-user hardening, billing or support.

## What I would change

- Add automated tests around the meal-parsing flow and the date handling, since those are where the bugs were.
- Measure the AI's macro estimates against labelled foods, so that the accuracy is a number.
- Move the remaining hard-coded settings into the settings screen.
