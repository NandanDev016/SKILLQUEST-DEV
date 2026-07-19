# SkillQuest — Implementation Plan

| | |
|---|---|
| **Version** | 1.1 — review corrections: workload rebalance, staged pilots, Judge0 moved to week 6, ML protocol order |
| **Depends on** | All prior docs — this sequences the actual build |
| **Timeline** | ~14 working weeks, single semester, team of 3 |
| **Purpose** | The build order: what gets built, in what sequence, by whom, and how we know it's done. |

---

## 1. Team & Ownership

| Person | Primary lane | Owns end-to-end |
|---|---|---|
| **Nandan** (016) | ML + platform backend | Risk model (OULAD experiment, TRD §6.3), Web API, DB schema/migrations, Judge0 integration, weekly scoring job |
| **Anjith** (032) | Frontend | React app, Monaco game screen, all screens, Supabase auth, accessibility conformance |
| **Bhanushree** (005) | Placement module + content + QA | **Placement scoring end-to-end** (JD curation → skill mapping → `/ai/placement-score` → gap classification), goal-mapper NLP, level authoring lead, testing, report |

**Rebalanced deliberately.** The first draft had Nandan owning the AI service, the ML, the backend, the schema and Judge0 — a single point of failure for the whole project. Bhanushree now owns the **placement module as a complete vertical** (data curation, the FastAPI endpoint, and its UI contract), not just isolated tasks. That's a real module boundary: it touches the AI service, so knowledge of that codebase is shared, and if one person is unavailable for a week the project still moves.

**Everyone authors content.** 40–50 levels is the hidden critical path — quota is **3 levels/person/week from week 2**. Bhanushree leads and reviews; all three write.

## 2. Milestones (the checkpoints that matter)

| # | Milestone | Target | "Done" means |
|---|---|---|---|
| M0 | Foundations | End wk 2 | Repos scaffolded, Supabase up, OULAD **baselines + first RF** under the §6.3 protocol, 6 levels authored, ethics-approval question answered |
| M1 | Auth + onboarding | End wk 4 | Consent → signup → onboarding → generated roadmap. **Pilot #1 (5 students)** on the onboarding flow |
| M2 | **Playable vertical slice** | End wk 6 | User solves a real level via Judge0, earns XP atomically. **Self-hosted Judge0 load-tested.** **Pilot #2 (5 students)** on the play screen |
| M3 | Gamification + AI live | End wk 9 | XP/streaks/badges, risk scoring on real events, placement coverage + gap classification. **Pilot #3 (5 students)** |
| M4 | Content complete + polish | End wk 11 | 40+ levels live, leaderboard, all edge/empty states, a11y pass, reconcile job run |
| M5 | Tested + documented | End wk 13 | UAT with 20–30 students, metrics collected, report drafted |
| M6 | Freeze & demo | Wk 14 | Bug-fixed, demo rehearsed, report + paper finalized |

**Three 5-student pilots, not one big UAT at the end.** The original plan put all user contact in weeks 12–13, which means a fundamental usability problem — a confusing play screen, an onboarding step people abandon — would be discovered when there is no time left to fix it. Each pilot is small and cheap: 5 students, 20 minutes each, watch them use it without helping. Findings from pilots feed the next milestone; the week 12–13 UAT then measures a design that has already survived contact with real users.

**M2 is the make-or-break milestone.** If the vertical slice works by week 6, the project succeeds. Everything before M2 exists to reach it; everything after scales it. Protect this date.

## 3. Week-by-Week

### Weeks 1–2 — Foundations (→ M0)
**Shared**
- Repo scaffold: `frontend/` (Vite+React+Tailwind), `backend/` (Express+Prisma), `ai-service/` (FastAPI), `ml/`, `content/`. Prettier/ESLint/black. GitHub Actions CI (lint + smoke tests). Branch protection on `main`.
- Two Supabase projects (dev/prod); enable `vector` extension; Google OAuth client configured.
- `.env.example` for all three services; secrets shared privately (never committed).

**Nandan**
- Prisma schema from [05-BACKEND-SCHEMA.md](05-BACKEND-SCHEMA.md); first migration on dev; least-privilege DB roles.
- **OULAD experiment, in protocol order** (TRD §6.3) — this de-risks the flagship AI claim in week 1, so do it before any web work:
  1. Build the labelled windowed dataset (28-day observation, 21-day horizon, Withdrawn-only positives).
  2. Implement the **student-grouped** and **temporal** splits.
  3. Run the **three baselines first** — majority, days-since-last-activity, logistic regression. These are the numbers the RF must beat.
  4. Train the Random Forest; report PR-AUC + confusion matrix against those baselines.
  5. Write down the honest comparison, whatever it says.

**Anjith**
- Frontend shell: routing, layout (sidebar/bottom-tabs), Tailwind theme tokens from [04-UIUX-DESIGN.md](04-UIUX-DESIGN.md), core components (`Button`, `Card`, `XPBadge`, `StreakFlame`).
- Supabase auth: sign-up/login screen, session handling, protected routes.

**Bhanushree**
- Author the **Java + DSA skill list** (~15–20 nodes) with `tags`, plus prerequisite edges → `content/skills.json` (unblocks the roadmap engine).
- Draft `goal_profiles` weight vectors (TRD §6.2) — the thing that makes goal personalization real.
- First 6 levels with test cases (`content/levels/*.json`). Establish the level JSON format everyone follows.
- **Ask the guide whether department/ethics approval is needed before student testing** (Backend Schema §5.1). Week 1, not week 12 — approval can take time and it gates every pilot.

**Exit check (M0):** CI green on all three services; a seeded skill graph in dev; baselines + RF metrics recorded under the stated protocol; 6 levels authored; ethics question answered.

### Weeks 3–4 — Auth, Onboarding, Roadmap (→ M1)
**Nandan**
- Web API: JWT verification middleware, `profiles` create-on-login, users/onboarding endpoints, events-write helper (used everywhere from here on).
- AI service: `/ai/goal-map` (fastembed + category embeddings) and `/ai/roadmap` (Kahn topo-sort + weekly bin-pack). Internal-key auth between services.

**Anjith**
- Onboarding wizard (5 steps, resume-by-step), mini-quiz, "Building your quest…" loader.
- Roadmap screen: skill-tree render, node states, side panel.

**Bhanushree**
- Levels to 12. Draft company skill profiles (`content/companies.json`) from live JDs.
- Write onboarding quiz questions (per-topic test-out mapping, App Flow §2).

**Exit check (M1):** two different onboarding inputs produce two visibly different roadmaps; AI-service-down falls back to a default roadmap.

### Weeks 5–6 — The Vertical Slice (→ M2, the critical milestone)
**Nandan**
- Judge0 integration: submission endpoint, runtime language-id resolution, per-test-case batch runs, verdict mapping, 64 KB cap, rate limiting.
- **Self-host Judge0 and load-test it this milestone** (5 concurrent users × 5 test cases, record p95). The RapidAPI free tier is ~50 requests/day ≈ 10 level attempts — it cannot survive even a 5-student pilot, so this moves from week 10 to now.
- `submissions` + `user_levels` write; **atomic XP award** (conditional transition + one transaction, Schema §3.7); level unlock logic.

**Anjith**
- **Play screen (S6)**: Monaco (Java), problem panel, Run Tests, results drawer (`TestResultRow`), compile-error display, hint reveal. Desktop split + **mobile tab layout**.
- XP animation, level-complete → next-unlock flow.

**Bhanushree**
- Levels to 18. Manually test every authored level against its own reference solution (catch broken test cases now, not during UAT).
- **Run Pilot #2**: 5 students attempt one level each, observed, no help given. Log where they hesitate.

**Exit check (M2):** on a phone and a laptop, a user reads a problem, writes Java, runs tests, sees per-case results, and earns XP — with zero manual steps, on self-hosted Judge0, with XP that cannot double-award. **Demo this to your guide.**

### Weeks 7–9 — Gamification + AI Modules Live (→ M3)
**Nandan**
- Streak engine (active day = a `level_submit`), badge evaluation inside the XP transaction, `/ai/risk-score` wired to the **external scheduler** job (GitHub Actions → authenticated endpoint, advisory lock, idempotent), tier → `profiles.risk_tier` + `dropout_scores` history with full reproducibility metadata.
- Nudge lifecycle: `nudges` rows for shown/clicked/dismissed, linked to the triggering prediction.
- Admin endpoints incl. "run scoring now" trigger and the `reconcile` job.

**Bhanushree** (owns this module end-to-end)
- `/ai/placement-score`: JD curation with provenance → human-confirmed skill mapping → weighted coverage → gap classification (`available_now` / `external_track`).

**Anjith**
- Dashboard (all cards, zero-state, NudgeCard swap-in), placement tracker screen (rings, gap list, "Train this →" deep-links), badge popups + confetti, profile/settings (hours-change → roadmap re-pack).
- Minimal admin table.

- Levels to 30. Begin the report (lit survey, methodology, screenshots). **Run Pilot #3.**

**Exit check (M3):** admin "run weekly scoring" flags a seeded at-risk user → nudge card appears on their dashboard and the impression is logged; completing a level moves coverage live; every prediction row carries its model version and window dates.

### Weeks 10–11 — Content Complete + Polish (→ M4)
- **Levels to 40–50** (all hands). Leaderboard (P1). Every loading/empty/error state from UI/UX §8. Skeleton loaders.
- **Accessibility pass** against UI/UX §9: keyboard-only run through the full play flow, target sizes, focus management, `aria-live` announcements verified with a screen reader.
- Warm-up cron (keeps Render warm + Supabase unpaused). Run the **reconcile job** and fix any cached-field drift. Internal bug bash: all three play through everything.

**Exit check (M4):** full app usable end-to-end with 40+ levels; keyboard-only completion of a level possible; no XP/streak drift.

### Weeks 12–13 — Testing + Documentation (→ M5)
- **UAT with 20–30 real students** (consent first): observed sessions, SUS questionnaire, engagement + would-recommend data, nudge interaction counts. This is your *real* results section — no invented numbers.
- Fix UAT findings (prioritize demo-path bugs). Measure API p95 **and** code-execution p95 separately.
- Report: results, risk-model evaluation vs baselines and published OULAD papers, screenshots, architecture.
- **Write the limitations section deliberately** — transfer assumption from OULAD, no control group, n≈30, single institution, self-selected participants. A stated limitation is a strength in a viva; an unstated one is an ambush.

**Exit check (M5):** UAT metrics collected; report complete through results and limitations.

### Week 14 — Freeze & Demo (→ M6)
- Code freeze (bug fixes only). Rehearse the demo on the *real* deployed app with warm-up done. Seed a compelling demo account (mid-roadmap, some badges, a movable placement score). Finalize report + paper + presentation.

## 4. Critical Path & Dependencies

```
Supabase + Prisma schema ─┬─> Web API endpoints ─┬─> Play screen ──> M2
OULAD baselines+RF (wk1) ──┘                       │
Skill graph + goal weights ──> Roadmap engine ─────┘
Judge0 self-hosted + load-tested (by wk6) ────────> pilots + UAT
Level authoring (continuous, 3/person/week) ──────────────────────> M4
Events log (from wk3) ──> risk features ──> weekly scoring ──> M3
Ethics approval question (wk1) ───────────────────> gates all pilots
```

**The three things that sink this project if they slip:** (1) the M2 vertical slice, (2) level authoring falling behind, (3) Judge0 capacity — the free tier cannot support even one pilot, so it must be self-hosted by week 6. All three are front-loaded for that reason.

## 5. Risk Triggers → Cut List

If behind schedule, cut in this order (all already P1/P2, so scope stays intact):
1. Leaderboard (F7)
2. Email nudges (in-app nudge stays)
3. Light theme (dark-only ships)
4. LSTM comparison experiment (RF is the deliverable)
5. Reduce levels 50 → 40 (never below 40 — it's a PRD success metric)

**Never cut:** the vertical slice, the risk model *with its baselines and honest protocol*, placement coverage, the pilots, or real UAT. Those are the project's identity — and a risk model without its baselines is not a result, it's a number.

**Also never cut (they cost hours, not weeks):** consent + withdrawal, per-prediction reproducibility metadata, atomic XP, and the accessibility pass. These are what make the work defensible rather than merely finished.

## 6. Definition of Done (every feature)

- Works on desktop **and** mobile (play screen especially).
- Loading + empty + error states handled (not just the happy path).
- Keyboard-operable, labelled, 44×44 targets, errors announced (UI/UX §9).
- Writes the right `events` rows.
- Rewarded actions are atomic and safe to retry.
- Reviewed in a PR by at least one teammate.
- No secrets committed; inputs validated.
- Any user-facing claim it makes is one the implementation actually supports.

## 7. Cadence

- **Weekly 30-min sync**: demo progress against the milestone, re-quota levels, surface blockers.
- **Guide review** at M1, M2, M4 — bring the working app, not slides.
- One shared board (GitHub Projects) mirroring these weeks; every task tagged to an owner and a milestone.
