# LineCrush Organization Memory

**Purpose:** This file teaches AI assistants how to work with LineCrush repositories and understand the product.

---

## Company Overview

**LineCrush Inc.** is a sports betting intelligence company headquartered in **Fresno, California**.

- **Product:** [linecrush.com](https://linecrush.com) — AI-powered sports betting picks and analysis
- **Mission:** Combine advanced AI models with real-time sports data to deliver actionable betting intelligence

---

## Core Product Philosophy

### Show Results, Hide Methods

LineCrush's edge comes from our data sources AND how we extract signals. Both must be protected.

**Guiding Principles:**

1. **Prove ROI boldly** — Display performance metrics, win rates, and returns prominently
2. **Protect the how** — Never expose methodology, algorithms, or signal extraction logic
3. **Guard data sources** — Minimize exposure of which sportsbooks/APIs/data providers we use
4. **Less is more** — Fewer, focused offerings rather than feature sprawl
5. **Black box design** — Users see inputs (preferences) and outputs (picks). The transformation is invisible.

**When building features, ask:**
- Does this reveal how we generate our edge?
- Does this expose our data sources unnecessarily?
- Can a competitor reverse-engineer our methodology from this?

If yes to any → redesign or remove the exposure.

---

## Tech Stack

| Layer | Technologies |
|-------|--------------|
| **Frontend** | Next.js 14, React 18, TypeScript, Tailwind CSS, shadcn/ui |
| **Backend** | Python 3.13+, asyncio, asyncpg |
| **Database** | PostgreSQL (Neon), Redis Cloud |
| **Infrastructure** | Vercel (frontend), Ubuntu VPS (backend), AWS S3 |
| **Auth** | Auth0 |
| **Payments** | Stripe |

---

## Coding Standards

See [CODING_CONVENTIONS.md](./CODING_CONVENTIONS.md) for detailed conventions covering:
- TypeScript/React patterns
- Python backend standards
- Database naming and query patterns
- Git workflow

---

## Sport Names Convention

**Always use these exact names (case-sensitive):**

```
NBA    — Basketball
NFL    — American Football
MLB    — Baseball
NHL    — Hockey
MMA    — Mixed Martial Arts
BKFC   — Bare Knuckle Fighting Championship
PGA    — Professional Golf Association
EPL    — English Premier League (Soccer)
Boxing
Tennis
F1     — Formula 1
WNBA   — Women's Basketball
NASCAR
```

Never use generic terms like "basketball" or "football" — always the league acronym.

---

## Key Repositories

| Repository | Purpose |
|------------|---------|
| [Context-Pro-AI](https://github.com/spragginsdesigns/Context-Pro-AI) | Main application monorepo (frontend + backend) |
| [.github](https://github.com/LineCrush/.github) | Organization profile and shared configs |

---

## Development Workflow

### Mindset

1. **Surgical changes over rewrites** — Mature production system, default to minimal changes
2. **Investigate before changing** — Check git history, understand existing behavior first
3. **Commit history is king** — When something breaks, check recent commits first

### Git Conventions

- **Commit format:** `<type>(<scope>): <imperative summary>`
- **Types:** `feat`, `fix`, `refactor`, `perf`, `docs`, `test`, `chore`
- **Scopes:** `api`, `ui`, `auth`, `bot`, `db`, `cache`, `docs`
- Always include: `Signed-off-by: Austin Spraggins <spragginsdesigns@gmail.com>`

### Time & Dates

- **Year:** 2025 (assume 2025 for all time-sensitive tasks)
- **Timezone:** Use terminal `date` command when unsure of current date

---

## Social & Links

| Platform | URL |
|----------|-----|
| Website | [linecrush.com](https://linecrush.com) |
| Twitter/X | [@LineCrush](https://x.com/linecrush), [@LineCrushBot](https://x.com/linecrushbot) |
| LinkedIn | [LineCrush](https://www.linkedin.com/company/linecrush) |
| GitHub Org | [github.com/LineCrush](https://github.com/LineCrush) |
| Email | [hello@linecrush.com](mailto:hello@linecrush.com) |

### Brand Assets

- **Main Logo:** `https://contextproai-storage.s3.us-east-1.amazonaws.com/Logo/LineCrush_Simple_Logo.png`
- **Stylized Text:** `https://contextproai-storage.s3.us-east-1.amazonaws.com/Logo/LineCrush_Stylized_Text2.png`

---

## Founders

| Name | GitHub | Email |
|------|--------|-------|
| Austin Spraggins | [@spragginsdesigns](https://github.com/spragginsdesigns) | [spragginsdesigns@gmail.com](mailto:spragginsdesigns@gmail.com) |
| Jay Raynor | [@beepimajeep88](https://github.com/beepimajeep88) | [jayraynor1988@gmail.com](mailto:jayraynor1988@gmail.com) |

---

## Quick Reference

### What LineCrush Does

1. **Multi-Book Analysis** — Compare odds across major sportsbooks instantly
2. **AI-Generated Picks** — Data-driven betting recommendations backed by rigorous analysis
3. **Real-Time Updates** — Lines move fast; we keep users ahead
4. **Performance Tracking** — Transparent results users can verify

### What Makes Us Different

- Proprietary AI models analyzing thousands of data points
- Multi-source data integration (podcasts, news, social, odds feeds)
- Focus on player props and specific betting markets
- Real-time odds tracking with best-book identification

---

*LineCrush Inc. — Fresno, California*
*"Crush the line. Beat the book."*
