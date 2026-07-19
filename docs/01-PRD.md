# SkillQuest — Product Requirements Document (PRD)

| | |
|---|---|
| **Product** | SkillQuest — AI-Powered Gamified Skill Learning Platform |
| **Version** | 1.0 |
| **Team** | Nandan S (1SP23AD016), Anjith K.J (1SP23AD032), Bhanushree C.V (1SP23AD005) |
| **Institution** | S.E.A College of Engineering & Technology, Bangalore — Dept. of AI & DS, VTU |
| **Timeline** | Single semester (~14 working weeks) |
| **Status** | Approved scope — locked. Changes require team agreement + update to this doc. |

---

## 1. Problem Statement

Engineering students in tier 2–3 Indian colleges face three compounding problems:

1. **Generic content** — platforms like Coursera, NPTEL, and YouTube deliver the same material to every learner regardless of their goal, current level, or available time.
2. **Passive learning → dropout** — self-paced video courses have very high abandonment rates; watching is not doing.
3. **No placement visibility** — students discover their skill gaps only during placement season, when it is too late to fix them.

SkillQuest addresses all three: it learns the student's goal at onboarding, teaches through interactive coding challenges instead of videos, predicts disengagement before the student quits, and continuously shows a placement-readiness score against real Indian IT service companies.

## 2. Goals

| # | Goal | Measured by |
|---|---|---|
| G1 | Teach one complete placement-oriented skill track (Java + DSA) through hands-on coding games | ≥ 40 playable levels live at launch |
| G2 | Personalize the learning path per student | Roadmap differs based on goal, self-assessed level, and hours/week |
| G3 | Detect at-risk learners early | Dropout model trained on a public dataset (OULAD), accuracy benchmarked against published papers; live feature logging on our platform |
| G4 | Make placement readiness visible at all times | Placement Score (0–100) per company profile, updated after every completed level |
| G5 | Keep students coming back | XP, streaks, and badges implemented; engagement events logged |

## 3. Non-Goals (Out of Scope — Do Not Build This Semester)

These are deliberately excluded to keep the project deliverable in one semester. Listing them here is what protects the timeline.

- ❌ **Visual drag-and-drop puzzle game engine** (second game type) — code-to-solve only.
- ❌ **Multiple skill tracks** (web dev, ML, etc.) — Java + DSA only. The data model must *support* more tracks; we just don't author their content.
- ❌ **Free-form conversational AI chatbot for onboarding** — structured form + one free-text goal field instead.
- ❌ **Resume bullet generator** — stretch goal only if everything else ships.
- ❌ **LSTM sequence model for dropout** — Random Forest is the deliverable; LSTM is a stretch/comparison experiment for the report.
- ❌ **Mobile app** — responsive web only.
- ❌ **Peer features** (chat, forums, team quests).

## 4. Target Users

**Primary persona — "Placement-anxious 3rd/4th-year student"**
- BE student in a tier 2–3 college, CS/AI/IT branch, targeting service-company placements (Infosys, TCS, Wipro, Accenture, Cognizant).
- Knows basic C/Python syntax from coursework but cannot solve interview-style problems.
- Has 5–10 hrs/week; loses motivation with long video courses.

**Secondary persona — "Early starter"**
- 2nd-year student who wants a structured path early; same product, longer roadmap.

## 5. Core Features & Priorities

Priority key: **P0** = must ship (project fails without it) · **P1** = should ship · **P2** = stretch.

### F1 — Onboarding & Goal Capture (P0)
- Supabase Auth email/Google sign-in.
- Onboarding wizard: branch, year, self-assessed skill level (beginner/intermediate/advanced via a 5-question mini-quiz), hours available per week, target companies (multi-select), and **one free-text field**: "Describe your career goal in your own words."
- The free-text goal is sent to the AI service, which maps it to a goal category using sentence embeddings (NLP module #1).
- **Acceptance:** two students with different quiz results and hours/week receive visibly different roadmaps.

### F2 — Personalized Roadmap (P0)
- Skills are stored as a **prerequisite DAG** (directed acyclic graph): e.g., `variables → loops → functions → recursion → …`.
- The roadmap engine topologically sorts the graph, skips nodes the student tested out of, and packs levels into weeks based on hours/week.
- Rendered as an interactive node tree on the dashboard: completed / current / locked states.
- **Acceptance:** roadmap regenerates correctly when the student changes hours/week in settings.

### F3 — Code-to-Solve Game Engine (P0)
- Monaco editor in the browser; each level = problem statement + starter code + hidden test cases.
- Code execution via **Judge0** (hosted API or self-hosted) — never executed on our own backend.
- Per-level: pass/fail per test case, XP award on full pass, hint system (hint costs a small XP amount — gamified help).
- **Content commitment: 40–50 levels** across the Java + DSA track (this is a team-wide authoring task, not just engineering).
- **Acceptance:** a student can complete a level end-to-end — read problem, write code, run tests, earn XP — in under one round trip of 3 seconds per run.

### F4 — Gamification Layer (P0)
- **XP** for level completion and daily activity; **levels/ranks** derived from XP.
- **Streaks** — consecutive active days, with a warning state on the dashboard before a streak breaks.
- **Badges** — First Quest, Week Warrior (7-day streak), Code Master (complete a skill node with no hints), Placement Ready (score ≥ 75).
- **Acceptance:** every XP/streak/badge event is written to an events log (this log doubles as the dropout model's live feature source).

### F5 — Dropout Risk Prediction (P0 — the flagship AI module)
- Model: **Random Forest classifier trained on OULAD** (Open University Learning Analytics Dataset), predicting at-risk/withdrawn students from engagement features (activity frequency, gaps between sessions, assessment scores, completion rate).
- Benchmark against published OULAD papers in the report; document precision/recall/F1, not just accuracy.
- On SkillQuest, the same feature schema is computed weekly per student from the events log; the trained model scores each student and flags **At Risk / Watch / Healthy**.
- Intervention: at-risk students get an in-app nudge (encouragement message + an easier "confidence booster" level suggestion). Email nudges = P1.
- **Acceptance:** risk tier visible on an internal admin view; nudge fires when tier changes to At Risk.

### F6 — Placement Readiness Tracker (P0)
- Company skill profiles (Infosys, TCS, Wipro, Accenture, Cognizant) curated from public job descriptions.
- Score = similarity between the student's completed-skill vector and the company profile, computed with **sentence-embedding similarity** (NLP module #2), shown as 0–100 with a per-skill gap list ("You're missing: recursion, SQL basics").
- **Acceptance:** completing a level visibly moves the score; gap list updates.

### F7 — Leaderboard (P1)
- Weekly XP leaderboard among users. Ship only after F1–F6 are stable.

### F8 — Stretch (P2)
- LSTM dropout model comparison (report material), resume bullet generator, email nudges, faculty/TPO dashboard.

## 6. Success Metrics (measured in final testing, weeks 12–14)

- 40+ levels playable; full user journey (signup → roadmap → 5 levels → score update) demo-able without errors.
- Dropout model: report metrics on OULAD test split with methodology (no invented numbers — whatever we get is what we report).
- User acceptance test with **20–30 real students** from college: engagement rating, System Usability Scale (SUS) questionnaire, would-recommend %.
- API latency: p95 < 500 ms for platform APIs (Judge0 execution time excluded, reported separately).

## 7. Constraints & Assumptions

- **Budget ≈ ₹0**: free tiers only — Vercel, Render, Supabase (Postgres + Auth), Judge0 free tier (rate-limited; self-host if limits bite).
- **Team of 3**, ~14 working weeks, alongside regular coursework.
- Content authoring (problems + test cases) is on the critical path and is scheduled like an engineering task.
- OULAD is publicly available for academic use; its feature semantics transfer reasonably to our platform (documented as an assumption in the report).

## 8. Risks & Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Content authoring falls behind | No game = no product | Start authoring in week 1 in parallel with dev; 3 levels/person/week quota; reuse classic DSA problems |
| Judge0 free-tier rate limits during demo | Demo failure | Self-host Judge0 fallback; cache results for identical submissions |
| Scope creep (adding cut features back) | Nothing finishes | Any scope change requires editing §3 of this doc + all 3 members agreeing |
| Free-tier cold starts (Render) slow demos | Bad first impression | Ping/warm-up script before demos; document as infra limitation |
| Dropout model looks "disconnected" from platform to examiners | Viva risk | Live demo shows the events log → weekly features → model score pipeline on a real user |

## 9. Release Plan (single semester)

- **Weeks 1–2** — Docs (this flow), repo setup, OULAD baseline notebook, content authoring begins.
- **Weeks 3–6** — Auth + onboarding + skill graph + roadmap; game engine with Judge0; first 15 levels live.
- **Weeks 7–9** — Gamification + events log; dropout model integrated; placement scorer.
- **Weeks 10–11** — Leaderboard (P1), polish, seed 40+ levels, internal bug bash.
- **Weeks 12–13** — User testing with real students; fix findings; collect metrics.
- **Week 14** — Freeze, final report/demo prep.
