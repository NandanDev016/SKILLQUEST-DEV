# 🎮 SkillQuest

**AI-Powered Personalized Gamified Skill Learning System for Engineering Students**

Final year project — B.E. Artificial Intelligence & Data Science, S.E.A College of Engineering & Technology, Bangalore (VTU).

SkillQuest teaches Java + DSA through hands-on coding games instead of videos. It builds a personalized week-by-week roadmap from your goals, predicts when you're about to give up (ML model trained on the OULAD dataset), and shows a live Placement Readiness Score against Indian IT companies.

## 👥 Team

| Name | USN | Role |
|---|---|---|
| Nandan S | 1SP23AD016 | System architecture, Python AI microservice, dropout prediction, backend |
| Anjith K.J | 1SP23AD032 | React frontend, game engine UI, Monaco editor, Supabase auth |
| Bhanushree C.V | 1SP23AD005 | NLP onboarding, placement tracker, PostgreSQL schema design, testing, documentation |

## 📚 Project Documents

All planning documents live in [docs/](docs/) — read them in order:

1. [Product Requirements (PRD)](docs/01-PRD.md) — what we're building and what we're **not** building
2. [Technical Requirements (TRD)](docs/02-TRD.md) — architecture, stack, integration contracts
3. [App Flow](docs/03-APP-FLOW.md) — screens, journeys, edge cases
4. [UI/UX Design](docs/04-UIUX-DESIGN.md) — design system, screen layouts
5. Backend Schema *(coming next)*
6. Implementation Plan

## 🏗️ Repository Layout

```
docs/         Project documents (start here)
frontend/     React + Vite + Tailwind SPA
backend/      Node.js + Express Web API
ai-service/   Python FastAPI (NLP, roadmap, dropout, placement scoring)
ml/           OULAD notebooks & model training (not deployed)
content/      Level definitions — problems, starter code, test cases
```

## ⚡ Ground Rules

- Scope is locked in the PRD §3 (Non-Goals). Adding features back requires all 3 members to agree and the PRD to be updated first.
- `main` is deploy-on-push — work on `feat/<name>` branches and open a PR.
- Never commit secrets (`.env` is gitignored). Never execute user code outside Judge0.
