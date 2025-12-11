# LineCrush Coding Conventions

**Version:** 1.0
**Last Updated:** December 2025

This document defines the coding standards and patterns used across the LineCrush codebase. All contributors should follow these conventions for consistency and maintainability.

---

## Table of Contents

1. [General Principles](#1-general-principles)
2. [Frontend (TypeScript/React/Next.js)](#2-frontend-typescriptreactnextjs)
3. [Backend (Python)](#3-backend-python)
4. [Database (PostgreSQL)](#4-database-postgresql)
5. [Git & Version Control](#5-git--version-control)

---

## 1. General Principles

### Core Mindset

- **Surgical changes over rewrites** — This is a mature production system. Default to minimal changes that preserve existing behavior.
- **Investigate before changing** — Understand the existing code before modifying. Check git history, trace data flows.
- **Single responsibility** — Each function, component, and module does one thing well.
- **Explicit over implicit** — Clear type annotations, named constants, no magic values.
- **Safe by default** — Parameterized queries, graceful error handling, logged failures.

### Competitive Moat Philosophy

**"Show results, hide methods."**

- **Prove ROI boldly** — Display performance metrics and win rates prominently.
- **Protect the how** — Never expose methodology or signal extraction logic in UI or APIs.
- **Guard data sources** — Minimize exposure of which sportsbooks/APIs we use.

### Sport Names Convention

Always use these exact sport names (case-sensitive):

```
NBA, NFL, MLB, NHL, MMA, BKFC, PGA, EPL, Boxing, Tennis, F1, WNBA, NASCAR
```

---

## 2. Frontend (TypeScript/React/Next.js)

### 2.1 File Organization

```
feature/
├── components/          # Sub-components
│   ├── SubComponent.tsx
│   └── index.ts        # Barrel export
├── hooks/              # Custom hooks
│   ├── useFeatureData.ts
│   └── index.ts
├── utils.ts            # Pure utility functions
├── types.ts            # TypeScript interfaces
├── constants.ts        # Feature constants
├── FeaturePage.tsx     # Main component
└── index.ts            # Barrel export
```

### 2.2 Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `PlayerProfile.tsx` |
| Hooks | camelCase with `use` prefix | `useBettingPreferences.ts` |
| Utilities | camelCase | `formatOdds.ts` |
| Types/Interfaces | PascalCase with Props/Data suffix | `PlayerProfileProps` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |

### 2.3 TypeScript Standards

**Strict mode enabled.** All params and returns must be typed.

```typescript
// Good: Explicit types, discriminated unions
interface PlayerEntity {
  type: 'player';
  id: string;
  name: string;
}

interface TeamEntity {
  type: 'team';
  id: string;
  name: string;
}

type BetEntity = PlayerEntity | TeamEntity;

// Type guard
const isPlayerEntity = (entity: BetEntity): entity is PlayerEntity => {
  return entity.type === 'player';
};
```

**Rules:**
- Use `unknown` over `any`; narrow with type guards
- Use `?.` and `??` for nullability; avoid `!` unless clearly safe
- Use `import type { }` for type-only imports
- Set `displayName` on memoized components

### 2.4 React Patterns

**Effects — Only for external system sync:**

```typescript
// BAD: Derived state in Effect
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(firstName + ' ' + lastName);
}, [firstName, lastName]);

// GOOD: Calculate during render
const fullName = firstName + ' ' + lastName;
```

**When to use Effects:**
- Fetching data (with cleanup)
- Subscribing to external stores
- Analytics on mount

**When NOT to use Effects:**
- Transforming data — Calculate during render
- Handling events — Use event handlers
- Expensive calculations — Use `useMemo`

**Component structure:**

```typescript
'use client';

interface FeatureComponentProps {
  id: string;
  onAction?: () => void;
}

export const FeatureComponent: React.FC<FeatureComponentProps> = React.memo(
  ({ id, onAction }) => {
    const { data, isLoading, error } = useFeatureData(id);

    if (isLoading) return <Spinner />;
    if (error) return <ErrorDisplay message={error} />;

    return (
      <div className="flex flex-col gap-4">
        {/* Content */}
      </div>
    );
  }
);

FeatureComponent.displayName = 'FeatureComponent';
```

### 2.5 Data Fetching (SWR)

Use SWR with centralized configuration:

```typescript
// lib/cache-keys.ts
export const cacheKeys = {
  playerProfile: (id: string) => `/api/player-profile?id=${id}`,
  playerVibe: (id: string) => `/api/player-vibe-report/${id}`,
};

// hooks/usePlayerData.ts
export function usePlayerData(playerId: string) {
  const { user, isLoading: userLoading } = useUser();
  const shouldFetch = user && !userLoading;

  const { data, error, mutate } = useSWR(
    shouldFetch ? cacheKeys.playerProfile(playerId) : null,
    fetcher,
    {
      revalidateOnFocus: false,
      dedupingInterval: 60000,
    }
  );

  return {
    player: data,
    isLoading: !error && !data,
    isError: !!error,
    mutate,
  };
}
```

### 2.6 Styling (Tailwind + shadcn/ui)

- Use Tailwind utility classes exclusively; no inline styles or CSS-in-JS
- Mobile-first responsive design (`sm:`, `md:`, `lg:` prefixes)
- Components from `frontend/src/components/ui` (shadcn)

```typescript
// Good
<div className="flex items-center gap-2 px-4 py-2 bg-gray-800 rounded-lg md:px-6">

// Bad
<div style={{ display: 'flex', padding: '8px' }}>
```

### 2.7 API Routes

```typescript
// app/api/feature/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getSession } from '@auth0/nextjs-auth0';

export async function GET(request: NextRequest) {
  // 1. Auth check first
  const session = await getSession();
  if (!session?.user) {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 401 }
    );
  }

  // 2. Parse params
  const { searchParams } = new URL(request.url);
  const id = searchParams.get('id');

  // 3. Query and return
  try {
    const data = await fetchData(id);
    return NextResponse.json({ data });
  } catch (error) {
    console.error('API Error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

---

## 3. Backend (Python)

### 3.1 Package Structure

```
backend/
├── service_name/
│   ├── __init__.py       # Module docstring, version
│   ├── config.py         # Environment & constants
│   ├── main.py           # CLI entry point
│   ├── database.py       # DB operations
│   ├── api_client.py     # External APIs
│   ├── core_logic.py     # Business logic
│   └── logs/             # Service logs
└── shared/
    ├── rich_logger.py    # Logging utilities
    ├── db_handler.py     # DB connection pool
    └── timing.py         # Metrics & timing
```

### 3.2 Type Hints

**All functions must have typed parameters and return values:**

```python
from typing import Dict, List, Optional, Any

async def fetch_player_data(
    pool: asyncpg.Pool,
    player_id: int,
    include_stats: bool = False
) -> Optional[Dict[str, Any]]:
    """Fetch player data from database.

    Args:
        pool: Database connection pool
        player_id: Player's unique identifier
        include_stats: Whether to include performance stats

    Returns:
        Player data dictionary or None if not found
    """
    ...
```

### 3.3 Enums & Dataclasses

```python
from enum import Enum
from dataclasses import dataclass, field
from typing import Optional

# Enums inherit from str for JSON serialization
class BetResult(str, Enum):
    WIN = "WIN"
    LOSS = "LOSS"
    PUSH = "PUSH"
    ERROR = "ERROR"

@dataclass
class GradingResult:
    """Result of grading a single bet."""
    result: BetResult
    actual_value: Optional[float] = None
    confidence: float = 0.0
    metadata: Dict[str, Any] = field(default_factory=dict)
```

### 3.4 Logging

Use the shared Rich logger:

```python
from shared.rich_logger import init_service_logger
import logging

console = init_service_logger(
    service_name="bet_grader",
    level="INFO",
    rotate=True,
    max_mb=25,
)
logger = logging.getLogger("bet_grader")

# Usage
logger.info(f"Processing {len(bets)} bets")
logger.warning(f"Deprecated function called: {func_name}")
logger.error(f"Database error: {e}", exc_info=True)
```

**Rules:**
- No emojis in logs
- Always include `exc_info=True` for errors
- Log file location: `backend/{service}/logs/`

### 3.5 Database Access

**Always use parameterized queries:**

```python
# Good: Parameterized
query = "SELECT * FROM players WHERE sport = $1 AND is_active = $2"
results = await pool.fetch(query, sport, True)

# Bad: String formatting (SQL injection risk!)
query = f"SELECT * FROM players WHERE sport = '{sport}'"
```

**Async pattern (asyncpg):**

```python
async def get_players(pool: asyncpg.Pool, sport: str) -> List[Dict]:
    query = """
        SELECT player_id, player_name, team
        FROM players
        WHERE sport = $1 AND is_active = true
        ORDER BY player_name
    """
    try:
        results = await pool.fetch(query, sport)
        return [dict(row) for row in results]
    except Exception as e:
        logger.error(f"Error fetching players: {e}", exc_info=True)
        return []  # Safe default
```

### 3.6 Error Handling

```python
# Custom exceptions with context
class PlayerStatsMissing(Exception):
    def __init__(self, player_id: str, stat_type: str):
        super().__init__(f"Missing '{stat_type}' for player '{player_id}'")
        self.player_id = player_id
        self.stat_type = stat_type

# Usage with retry
async def fetch_with_retry(url: str, max_retries: int = 3) -> Dict:
    for attempt in range(max_retries):
        try:
            response = await client.get(url)
            return response.json()
        except RateLimitException as e:
            if attempt < max_retries - 1:
                wait_time = 2 ** attempt
                logger.warning(f"Rate limited, retrying in {wait_time}s")
                await asyncio.sleep(wait_time)
            else:
                raise
```

### 3.7 Configuration

```python
# config.py
import os
from pathlib import Path
from dotenv import load_dotenv

# Find backend root
BACKEND_ROOT = next(
    (p for p in Path(__file__).resolve().parents if p.name == "backend"),
    None
)
if not BACKEND_ROOT:
    raise RuntimeError("Cannot locate backend/ directory")

# Load .env
load_dotenv(BACKEND_ROOT / ".env", override=True)

# Required (fail fast if missing)
POSTGRES_URL = os.environ.get("POSTGRES_URL_NON_POOLING")
if not POSTGRES_URL:
    raise ValueError("POSTGRES_URL_NON_POOLING must be set")

# Optional with defaults
LOG_LEVEL = os.getenv("LOG_LEVEL", "INFO")
RATE_LIMIT_DELAY = float(os.getenv("RATE_LIMIT_DELAY", "1.0"))
```

### 3.8 Entry Points

```python
"""
Bet Grader - Grade betting recommendations

Usage:
  python -m bet_grader.main              # Grade all
  python -m bet_grader.main --limit 50   # Grade 50
  python -m bet_grader.main --dry-run    # Preview only
"""

import asyncio
import argparse

def main():
    parser = argparse.ArgumentParser(description="Grade vibe bets")
    parser.add_argument("--limit", type=int, default=None)
    parser.add_argument("--dry-run", action="store_true")
    parser.add_argument("--sport", type=str, default=None)
    args = parser.parse_args()

    try:
        asyncio.run(orchestrate(args))
    except KeyboardInterrupt:
        logger.info("Interrupted by user")
    except Exception as e:
        logger.critical(f"Fatal error: {e}", exc_info=True)
        raise SystemExit(1)

if __name__ == "__main__":
    main()
```

---

## 4. Database (PostgreSQL)

### 4.1 Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Tables | lowercase_snake_case | `vibe_bets`, `calendar_events` |
| Columns | lowercase_snake_case | `player_id`, `game_date` |
| Primary keys | `id` or `{table}_id` | `player_id` |
| Foreign keys | `{referenced_table}_id` | `player_id` |
| Booleans | `is_*` or `has_*` | `is_active`, `has_media` |
| Timestamps | `*_at` suffix | `created_at`, `updated_at` |

### 4.2 Standard Columns

**All tables should include:**

```sql
id SERIAL PRIMARY KEY,
created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
updated_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
```

### 4.3 Data Types

```sql
-- IDs
id SERIAL PRIMARY KEY           -- Auto-incrementing
external_id VARCHAR(100)        -- External platform IDs

-- Text
name VARCHAR(255)               -- Names, titles
description TEXT                -- Long content
status VARCHAR(20)              -- Enum-like values

-- Numbers
count INTEGER                   -- Counts, scores
odds NUMERIC(10,2)              -- Precise decimals
score FLOAT                     -- Approximate values

-- Dates/Times
created_at TIMESTAMPTZ          -- Always with timezone
game_date DATE                  -- Date only
game_time VARCHAR(10)           -- Time as string "HH:MM"

-- Boolean
is_active BOOLEAN DEFAULT TRUE
processed BOOLEAN DEFAULT FALSE

-- JSON
raw_data JSONB                  -- Nested structures
odds_data JSONB                 -- Complex objects
player_ids INTEGER[]            -- Arrays (native type)
```

### 4.4 Indexing Strategy

```sql
-- Foreign keys (automatic in some DBs, explicit for clarity)
CREATE INDEX idx_vibe_bets_player_id ON vibe_bets(player_id);

-- Filter columns
CREATE INDEX idx_players_sport ON players(sport);
CREATE INDEX idx_vibe_bets_game_date ON vibe_bets(game_date DESC);
CREATE INDEX idx_tweets_processed ON x_tweets(processed);

-- JSONB columns (GIN index)
CREATE INDEX idx_tweets_player_ids_gin ON tweets USING gin(player_ids);

-- Composite indexes
CREATE INDEX idx_bets_sport_date ON vibe_bets(sport, game_date);
```

### 4.5 Common Query Patterns

```sql
-- Upsert (INSERT ... ON CONFLICT)
INSERT INTO players (player_id, player_name, sport)
VALUES ($1, $2, $3)
ON CONFLICT (player_id) DO UPDATE SET
  player_name = EXCLUDED.player_name,
  updated_at = NOW();

-- JSONB extraction
SELECT
  bet_structured->>'player_name' as player_name,
  (current_odds->>'value')::numeric as odds_value
FROM vibe_bets;

-- Array contains
SELECT * FROM tweets WHERE $1 = ANY(player_ids);

-- Date filtering
WHERE game_date >= CURRENT_DATE
  AND created_at > NOW() - INTERVAL '7 days';
```

---

## 5. Git & Version Control

### 5.1 Commit Message Format

```
<type>(<scope>): <imperative summary>

<body - explain why, not what>

Relates to #<issue-number>

Signed-off-by: Name <email>
```

**Types:** `feat`, `fix`, `refactor`, `perf`, `docs`, `test`, `chore`
**Scopes:** `api`, `ui`, `auth`, `bot`, `db`, `cache`, `docs`

**Example:**

```
feat(api): add multi-book odds comparison endpoint

Enables users to compare odds across sportsbooks for the same event.
Uses new SportsGamesOdds API integration with 5-minute caching.

- Add /api/odds/compare route
- Integrate OddsAPIClient with rate limiting
- Add Redis caching layer

Relates to #142

Signed-off-by: Austin Spraggins <spragginsdesigns@gmail.com>
```

### 5.2 Branch Strategy

- `main` — Production-ready code
- `Production` — Deployed to production
- Feature branches from `main`

### 5.3 Git Rules

- Always `git pull` before pushing
- Use chained commands: `git add -A && git commit -m "msg" && git push`
- Stage files intentionally; review `git status` before committing
- Use `Relates to #<id>` by default; `Closes #<id>` only when explicitly confirmed

---

## Quick Reference

### Frontend Checklist

- [ ] TypeScript strict mode, all types explicit
- [ ] Components under 200 lines
- [ ] No Effects for derived state
- [ ] SWR for data fetching with cache keys
- [ ] Tailwind only, no inline styles
- [ ] Auth check in API routes

### Backend Checklist

- [ ] Type hints on all functions
- [ ] Docstrings with Args/Returns
- [ ] Parameterized SQL queries
- [ ] Error logging with `exc_info=True`
- [ ] Config validated at startup
- [ ] `--dry-run` flag for scripts

### Database Checklist

- [ ] snake_case naming
- [ ] `created_at`/`updated_at` on all tables
- [ ] TIMESTAMPTZ for timestamps
- [ ] Indexes on filter columns
- [ ] GIN indexes for JSONB arrays

---

*LineCrush Inc. — Fresno, California*
