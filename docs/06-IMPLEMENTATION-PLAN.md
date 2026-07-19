# SkillQuest — Implementation Plan

| | |
|---|---|
| **Version** | 1.0 |
| **Depends on** | All prior docs — this sequences the actual build |
| **Timeline** | ~14 working weeks, single semester, team of 3 |
| **Purpose** | The build order: what gets built, in what sequence, by whom, and how we know it's done. |

---

## 1. Team & Ownership

| Person | Primary lane | Owns end-to-end |
|---|---|---|
| **Nandan** (016) | AI service + backend | FastAPI service, dropout ML (OULAD), roadmap/placement algorithms, DB schema in Prisma, Judge0 integration |
| **Anjith** (032) | Frontend | React app, Monaco game screen, all screens from UI/UX doc, Supabase auth wiring |
| **Bhanushree** (005) | Content + NLP + QA | Level authoring (lead), goal-mapper + placement NLP, company profiles, testing, documentation/report |

**Everyone authors content.** 40–50 levels is the hidden critical path — quota is **3 levels/person/week from week 2**. Bhanushree leads/reviews; all three write.

## 2. Milestones (the checkpoints that matter)

| # | Milestone | Target | "Done" means |
|---|---|---|---|
| M0 | Foundations | End wk 2 | Repos scaffolded, Supabase up, one OULAD baseline model trained, 6 levels authored |
| M1 | Auth + onboarding | End wk 4 | User can sign up, complete onboarding, see a generated roadmap |
| M2 | **Playable vertical slice** | End wk 6 | User solves a real level via Judge0, earns XP — the core loop works end-to-end |
| M3 | Gamification + AI live | End wk 9 | XP/streaks/badges, dropout scoring on real events, placement score all working |
| M4 | Content complete + polish | End wk 11 | 40+ levels live, leaderboard, all edge/empty states, Judge0 self-hosted |
| M5 | Tested + documented | End wk 13 | UAT with 20–30 students done, metrics collected, report drafted |
| M6 | Freeze & demo | Wk 14 | Bug-fixed, demo rehearsed, report + paper finalized |

**M2 is the make-or-break milestone.** If the vertical slice works by week 6, the project succeeds. Everything before M2 exists to reach it; everything after scales it. Protect this date.

## 3. Week-by-Week

### Weeks 1–2 — Foundations (→ M0)
**Shared**
- Repo scaffold: `frontend/` (Vite+React+Tailwind), `backend/` (Express+Prisma), `ai-service/` (FastAPI), `ml/`, `content/`. Prettier/ESLint/black. GitHub Actions CI (lint + smoke tests). Branch protection on `main`.
- Two Supabase projects (dev/prod); enable `vector` extension; Google OAuth client configured.
- `.env.example` for all three services; secrets shared privately (never committed).

**Nandan**
- Prisma schema from [05-BACKEND-SCHEMA.md](05-BACKEND-SCHEMA.md); first migration on dev.
- **OULAD baseline**: download, EDA notebook, engineer the shared feature schema (TRD §6.3), train a Random Forest, record precision/recall/F1/ROC-AUC. This de-risks the flagship AI claim in week 1 — do it first.

**Anjith**
- Frontend shell: routing, layout (sidebar/bottom-tabs), Tailwind theme tokens from [04-UIUX-DESIGN.md](04-UIUX-DESIGN.md), core components (`Button`, `Card`, `XPBadge`, `StreakFlame`).
- Supabase auth: sign-up/login screen, session handling, protected routes.

**Bhanushree**
- Author the **Java + DSA skill list** (~15–20 nodes) + prerequisite edges → `content/skills.json` (unblocks the roadmap engine).
- First 6 levels with test cases (`content/levels/*.json`). Establish the level JSON format everyone follows.

**Exit check (M0):** CI green on all three services; a seeded skill graph in dev; a trained `.joblib` with documented metrics; 6 levels authored.

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
- Judge0 integration: submission endpoint, per-test-case batch runs, verdict mapping, 64 KB cap, rate limiting. Start on RapidAPI free tier.
- `submissions` + `user_levels` write; XP award on full pass; level unlock logic.

**Anjith**
- **Play screen (S6)**: Monaco (Java), problem panel, Run Tests, results drawer (`TestResultRow`), compile-error display, hint reveal. Desktop split + **mobile tab layout**.
- XP animation, level-complete → next-unlock flow.

**Bhanushree**
- Levels to 18. Manually test every authored level against its own reference solution (catch broken test cases now, not during UAT).

**Exit check (M2):** on a phone and a laptop, a user reads a problem, writes Java, runs tests, sees per-case results, and earns XP — with zero manual steps. **Demo this to your guide.**

### Weeks 7–9 — Gamification + AI Modules Live (→ M3)
**Nandan**
- Streak engine (active day = a `level_submit`), badge evaluation on events, `/ai/dropout-score` wired to the weekly job (in-process `node-cron`), tier → `profiles.risk_tier` + `dropout_scores` history.
- `/ai/placement-score` (weighted coverage + embedding match), `placement_scores` history.
- Admin endpoints incl. "run scoring now" triggers.

**Anjith**
- Dashboard (all cards, zero-state, NudgeCard swap-in), placement tracker screen (rings, gap list, "Train this →" deep-links), badge popups + confetti, profile/settings (hours-change → roadmap re-pack).
- Minimal admin table.

**Bhanushree**
- Levels to 30. Finalize company weights. Begin the report (lit survey, methodology, screenshots).

**Exit check (M3):** admin "run weekly scoring" flags a seeded at-risk user → nudge card appears on their dashboard; completing a level moves the placement score live.

### Weeks 10–11 — Content Complete + Polish (→ M4)
- **Levels to 40–50** (all hands). Leaderboard (P1). Every loading/empty/error state from UI/UX §8. Skeleton loaders.
- **Self-host Judge0** (Oracle Cloud free VM / college server) — free-tier limits will break UAT/demo otherwise. Non-negotiable before week 12.
- Warm-up cron (keeps Render warm + Supabase unpaused). Internal bug bash: all three play through everything.

**Exit check (M4):** full app usable end-to-end with 40+ levels; Judge0 self-hosted and stable under repeated runs.

### Weeks 12–13 — Testing + Documentation (→ M5)
- **UAT with 20–30 real students**: observed sessions, SUS questionnaire, engagement + would-recommend data. This is your *real* results section — no invented numbers.
- Fix UAT findings (prioritize demo-path bugs). Measure API p95 latency.
- Report: results, dropout-model evaluation vs published OULAD papers, screenshots, architecture. Draft the paper.

**Exit check (M5):** UAT metrics collected; report complete through results.

### Week 14 — Freeze & Demo (→ M6)
- Code freeze (bug fixes only). Rehearse the demo on the *real* deployed app with warm-up done. Seed a compelling demo account (mid-roadmap, some badges, a movable placement score). Finalize report + paper + presentation.

## 4. Critical Path & Dependencies

```
Supabase + Prisma schema ─┬─> Web API endpoints ─┬─> Play screen ──> M2
OULAD model (wk1) ─────────┘                       │
Skill graph JSON ──> Roadmap engine ───────────────┘
Judge0 (RapidAPI) ─────────────────────────> Judge0 self-hosted (wk10) ──> UAT
Level authoring (continuous, 3/person/week) ──────────────────────────────> M4
Events log (from wk3) ──> dropout features ──> dropout scoring ──> M3
```

**The two things that sink this project if they slip:** (1) the M2 vertical slice, (2) level authoring falling behind. Both are front-loaded and quota'd for that reason.

## 5. Risk Triggers → Cut List

If behind schedule, cut in this order (all already P1/P2, so scope stays intact):
1. Leaderboard (F7)
2. Email nudges (in-app nudge stays)
3. Light theme (dark-only ships)
4. LSTM comparison experiment (RF is the deliverable)
5. Reduce levels 50 → 40 (never below 40 — it's a PRD success metric)

**Never cut:** the vertical slice, the dropout model, placement scoring, or real UAT. Those are the project's identity.

## 6. Definition of Done (every feature)

- Works on desktop **and** mobile (play screen especially).
- Loading + empty + error states handled (not just the happy path).
- Writes the right `events` rows.
- Reviewed in a PR by at least one teammate.
- No secrets committed; inputs validated.

## 7. Cadence

- **Weekly 30-min sync**: demo progress against the milestone, re-quota levels, surface blockers.
- **Guide review** at M1, M2, M4 — bring the working app, not slides.
- One shared board (GitHub Projects) mirroring these weeks; every task tagged to an owner and a milestone.
