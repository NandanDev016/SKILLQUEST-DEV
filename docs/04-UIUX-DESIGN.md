# SkillQuest — UI/UX Design

| | |
|---|---|
| **Version** | 1.0 |
| **Depends on** | [03-APP-FLOW.md](03-APP-FLOW.md) (screens & journeys) |
| **Purpose** | The design system + screen layouts the frontend is built from. Tailwind-first, so every token here maps to a class. |

---

## 1. Design Principles

1. **Game-feel, not childish.** The users are 20–22 year olds anxious about jobs. Playful energy (XP bursts, streak flames, badges) but a clean, credible product — think Duolingo's motivation with LeetCode's seriousness. Never cartoonish.
2. **The next action is always obvious.** Every screen has exactly one primary CTA. A stressed student should never wonder "what do I do now?"
3. **Progress is always visible.** XP, streak, and placement score are persistent — the student should feel momentum on every screen.
4. **Reward effort, soften failure.** Passing tests = celebration. Failing tests = helpful, never punishing (see App Flow §5, §6). Colors and copy follow this.
5. **Fast and light.** Free-tier hosting + Indian mobile networks. No heavy animation libraries, no multi-MB hero videos. Perceived speed > visual richness.

## 2. Color System

Dark-first (coding audience expects it; Monaco is dark). Light theme is a P1 stretch. All colors are Tailwind-compatible; custom values go in `tailwind.config.js` under `theme.extend.colors`.

| Token | Hex | Use |
|---|---|---|
| `bg-base` | `#0F1117` | App background |
| `bg-surface` | `#1A1D27` | Cards, panels |
| `bg-surface-2` | `#242836` | Raised elements, editor chrome, hover |
| `border-subtle` | `#2E3342` | Dividers, card borders |
| `text-primary` | `#F2F4F8` | Headings, body |
| `text-muted` | `#9AA3B2` | Secondary text, labels |
| `primary` (Quest Violet) | `#7C5CFC` | Primary buttons, active nav, focus rings |
| `primary-hover` | `#6A4AF0` | Hover state |
| `accent` (XP Gold) | `#FFC53D` | XP, coins, badges, streak flame |
| `success` | `#3DD68C` | Passed tests, completed nodes |
| `danger` | `#F2555A` | Failed tests, errors (used sparingly) |
| `info` | `#4CA5FF` | Hints, informational callouts |
| `risk-atrisk` | `#F2555A` | Admin risk tier (At Risk) |
| `risk-watch` | `#FFC53D` | Admin risk tier (Watch) |
| `risk-healthy` | `#3DD68C` | Admin risk tier (Healthy) |

**Usage discipline:** violet = "you can act on this", gold = "you earned this", green = "you succeeded", red = "something failed" (and only that — never decoration). Accessibility: all text pairs meet WCAG AA (≥4.5:1) on their background; `primary` on `bg-base` is used for large text/buttons only.

## 3. Typography

- **UI font:** Inter (self-hosted `.woff2`, not a CDN — CSP + offline resilience). Weights 400/500/600/700.
- **Code font:** JetBrains Mono for Monaco, problem code snippets, terminal output.
- **Scale (Tailwind):** `text-xs` 12 (labels) · `text-sm` 14 (secondary) · `text-base` 16 (body) · `text-lg` 18 · `text-xl` 20 (card titles) · `text-2xl` 24 (section) · `text-4xl` 36 (page hero, XP counters). Line-height `leading-relaxed` for problem statements (dense reading).

## 4. Spacing, Radius, Elevation

- **Spacing:** 4px base grid — use Tailwind scale (`p-2`=8, `p-4`=16, `p-6`=24). Card padding `p-6`; page gutters `px-4` mobile / `px-8` desktop.
- **Radius:** `rounded-lg` (8px) default, `rounded-xl` (12px) cards, `rounded-full` avatars/pills/badges.
- **Elevation:** shadows are subtle on dark — prefer a lighter surface (`bg-surface-2`) + `border-subtle` over heavy shadows. One soft shadow token for modals/popovers only.
- **Motion:** 150–200ms ease-out for hovers/transitions; XP/badge celebrations up to ~600ms. Respect `prefers-reduced-motion` — celebrations degrade to a simple fade.

## 5. Core Components (build these once, reuse everywhere)

| Component | Notes |
|---|---|
| `Button` | Variants: primary (violet), secondary (surface-2 + border), ghost, danger. Sizes sm/md/lg. Loading state = spinner + disabled. |
| `Card` | `bg-surface`, `rounded-xl`, `border-subtle`, `p-6`. The workhorse. |
| `XPBadge` | Gold pill with ⚡ icon + number; animates on increase (count-up). |
| `StreakFlame` | 🔥 + day count; three states (lit / amber-pulse / grey) per App Flow §6. |
| `ProgressRing` | Circular %; used for placement score and node completion. |
| `SkillNode` | Roadmap tree node: states = locked (grey, 🔒), current (violet, pulsing), completed (green, ✓). |
| `TestResultRow` | Green ✓ / red ✗ + test name; expandable for input/expected/actual (visible tests only). |
| `BadgePopup` | Center-screen modal, badge art + name + "Nice!" dismiss; confetti (canvas-confetti, lightweight). |
| `NudgeCard` | Warm-toned dashboard card for at-risk intervention (App Flow §5). |
| `Toast` | Bottom-right; success/info/error. Auto-dismiss 4s. |
| `EmptyState` | Illustration + one line + one CTA. Used per App Flow §8. |

Component states are non-negotiable: every interactive component must define default / hover / focus-visible / active / disabled / loading. Focus-visible = 2px `primary` ring (keyboard accessibility).

## 6. Layout System

- **Desktop (≥1024px):** fixed left sidebar (240px) — logo, nav (Dashboard, Roadmap, Placement, Leaderboard, Profile), XP+streak pinned at top. Content area max-width `1200px`, centered.
- **Tablet (768–1023px):** collapsible sidebar (icon-only rail).
- **Mobile (<768px):** bottom tab bar (5 icons); XP+streak in a compact top bar. **The play screen (S6) is the hard one** — see §7.6.

## 7. Screen Layouts

### 7.1 Landing (S1)
Hero: headline "Level up from student to placement-ready" + subhead + primary CTA "Start your quest". Below: 3 feature cards (Personalized roadmap / Learn by doing / Know you're placement-ready), a "how it works" 5-step strip, footer. Keep it one-scroll; this is the pitch, not a marketing site.

### 7.2 Auth (S2)
Centered card on `bg-base`. Google button (primary path) + email/password. Toggle sign-up/login. Minimal — get them through fast.

### 7.3 Onboarding Wizard (S3)
Full-screen, one question per step, progress bar at top (Step 2 of 5). Big friendly inputs. Each step animates in from the right. **Step 5 → "Building your quest…" loader** (App Flow §2): themed animation (skill nodes assembling), 2–4s. This screen sets the emotional tone — invest design polish here.

### 7.4 Dashboard (S4) — the home base
```
┌─────────────────────────────────────────────┐
│  Top: ⚡ 1,240 XP    🔥 6-day streak   [avatar]│
├─────────────────────────────────────────────┤
│  ┌─────────────────────┐  ┌────────────────┐ │
│  │ CONTINUE YOUR QUEST │  │ Placement Score│ │
│  │ Level: Recursion #3 │  │   ◐ 62 / 100   │ │
│  │ [ Resume → ]        │  │  ▲ +4 this wk  │ │
│  └─────────────────────┘  └────────────────┘ │
│  ┌─────────────────────┐  ┌────────────────┐ │
│  │ This week's progress│  │ Recent badges  │ │
│  │ ▓▓▓▓▓░░ 5/8 levels   │  │ 🏅 🏅 🏅        │ │
│  └─────────────────────┘  └────────────────┘ │
└─────────────────────────────────────────────┘
```
The "Continue your quest" card is the single primary CTA. **When the dropout model flags At Risk, the NudgeCard replaces this card** (App Flow §5) — same slot, so the intervention is unmissable. New users see the zero-state (one big "Start your first quest").

### 7.5 Roadmap (S5)
Vertical skill-tree (top = start, down = advanced), branches where prerequisites fork. Nodes use `SkillNode` states. Current node pulses. Tap a node → side panel: concept description, levels inside it, XP available, "Start" CTA. Supports `?focus=<skillId>` deep-link from the placement tracker (auto-scroll + highlight). On mobile: the tree becomes a vertical stepper (avoid pan/zoom pain).

### 7.6 Play Screen (S6) — the core, and the responsive challenge
**Desktop:** split view — left 40% problem panel (statement, examples, constraints, in `leading-relaxed`), right 60% Monaco (JetBrains Mono, dark). Bottom drawer: "Run Tests" (primary) + results. Silent timer (analytics only — never shown).

**Mobile:** tabs — [ Problem | Code | Results ] — because a split view is unusable on a phone. Code tab is default once the student has read the problem. This is explicitly called out because it's the highest-risk layout in the app.

Results drawer uses `TestResultRow`; failed *visible* tests expand to input/expected/actual; hidden tests show pass/fail only (never leak hidden cases). Compile errors render in the drawer in mono, `danger`-tinted. "Stuck? Get a hint (−10 XP)" is `info`-styled and clearly optional.

### 7.7 Placement Tracker (S7)
Per-company cards, each a `ProgressRing` + score + trend. Tap → gap list: missing skills ranked by weight, each with "Train this →" deep-linking to `/roadmap?focus=<skillId>`. Honest framing: "You're 62% ready for Infosys" not a fake guarantee.

### 7.8 Profile (S8)
Avatar, stats (total XP, levels, best streak), full badge shelf (earned bright, locked greyed with unlock hints), and **settings**: hours/week (re-triggers roadmap re-pack with a confirm dialog per App Flow §8), target companies. No full re-onboarding in v1.

### 7.9 Leaderboard (S9, P1)
Weekly XP ranking, current user's row pinned + highlighted. `<5 users` → "Early adopter" empty framing (App Flow §8).

### 7.10 Admin (S10, internal)
Function over form — plain table, risk tiers color-coded (`risk-*` tokens), sortable by risk. Level publish toggles. "Run weekly scoring now" / "Recompute placement" buttons for live demos. No design polish budget here.

## 8. Loading, Empty & Error States (design each, don't leave blank)

| State | Treatment |
|---|---|
| Data loading | Skeleton screens (grey shimmer blocks), not spinners, for dashboard/roadmap |
| Running tests | Button spinner + "Running tests…"; editor locked |
| Judge0 down | Inline error in results drawer, retry button, reassuring copy; submission not counted |
| AI service down (onboarding) | Silent default roadmap; user sees a normal reveal |
| Empty roadmap complete | Celebration illustration + leaderboard/stretch pointer |
| Network offline | Toast + code preserved in `localStorage` |
| First-time zero states | `EmptyState` component, one CTA |

## 9. Accessibility (bake in, don't retrofit)

- WCAG AA contrast everywhere (§2 palette is pre-checked).
- Every interactive element keyboard-reachable; visible `focus-visible` ring.
- Monaco has built-in a11y; ensure the Run Tests flow is operable without a mouse.
- `prefers-reduced-motion` honored for all celebrations.
- Semantic HTML + ARIA labels on icon-only buttons (streak, XP, nav icons).
- Don't encode meaning in color alone — pair with icons (✓/✗, 🔒) so red/green failures are distinguishable for colorblind users.

## 10. Assets & Deliverables (for the report + build)

- Design in **Figma** (free): a color/type style sheet, the core components from §5, and hi-fi mockups of the 6 P0 screens. Export screenshots for the project report.
- Icons: **Lucide** (React, tree-shakeable, no external calls). Emoji for playful accents (🔥⚡🏅) — zero asset weight.
- Badge art: simple flat SVGs (can be AI-generated then hand-cleaned); keep a consistent style.
- No external font/icon CDNs — self-host everything (matches the Artifact-style CSP discipline and keeps the app fast on college wifi).
