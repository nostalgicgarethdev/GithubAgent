# Architecture Overview

$GitAgent is a pipeline of loosely-coupled modules that run continuously. Each module has a single responsibility and communicates via an internal event queue.

---

## High-Level Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                      $GitAgent Pipeline                      │
│                                                             │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐             │
│  │  Scanner  │───▶│Evaluator │───▶│ Launcher │             │
│  │  (X API)  │    │ (Scorer) │    │(Solana)  │             │
│  └──────────┘    └──────────┘    └──────────┘             │
│        │                               │                    │
│        ▼                               ▼                    │
│  ┌──────────┐                   ┌──────────┐              │
│  │  Repo DB  │                   │Fee Router│              │
│  │ (cache)   │                   │ (on-chain)│             │
│  └──────────┘                   └──────────┘              │
│                                        │                    │
│                                        ▼                    │
│                                  ┌──────────┐              │
│                                  │Broadcaster│             │
│                                  │ (X / API) │             │
│                                  └──────────┘              │
└─────────────────────────────────────────────────────────────┘
```

---

## Modules

### Scanner
- Connects to the X/Twitter filtered stream API
- Parses tweets in real-time for `github.com` links
- Deduplicates repos seen within a rolling 24h window
- Pushes new repo events to the evaluation queue

**Input:** Raw tweet stream  
**Output:** `{ repo_url, tweet_id, engagement_snapshot, timestamp }`

---

### Evaluator (Signal Scorer)
- Pulls repo metadata from GitHub API (stars, forks, last commit, language)
- Combines GitHub signals with X engagement metrics
- Computes a composite score (0–100)
- Repos above the launch threshold are forwarded to the Launcher

**Scoring factors:**
| Factor | Weight |
|--------|--------|
| Tweet engagement (likes + RTs) | 35% |
| Repo stars | 20% |
| Mention velocity (tweets/hour) | 25% |
| Repo recency (last commit) | 10% |
| Unique mentioner count | 10% |

**Input:** `repo_url + engagement_snapshot`  
**Output:** `{ score, repo_url, dev_address, launch: bool }`

---

### Launcher
- Receives high-score repos from the Evaluator
- Checks the Repo DB — skips already-launched repos
- Deploys a token on Solana (pump.fun compatible)
- Registers the developer's on-chain address as fee recipient
- Writes launch record to Repo DB

**Input:** Scored repo event  
**Output:** `{ token_mint, token_symbol, dev_address, tx_hash }`

---

### Fee Router
- Monitors on-chain swap events for all launched tokens
- On each swap, calculates the developer fee share
- Executes transfer instruction to the registered dev address
- Emits fee event to the Broadcaster

**Fee split:**
- Developer: configurable (default 40% of protocol fee)
- Protocol: remainder

---

### Broadcaster
- Listens for events from all other modules
- Formats and posts updates to X via the API
- Rate-limits posts to stay within API quotas
- Queues posts if rate limit is hit; flushes when clear

**Post triggers:**
- New repo detected (high-signal)
- Token launched
- Developer fee paid
- Hourly summary

---

### Repo DB
- Simple persistent store (SQLite or Redis)
- Tracks: seen repos, launched tokens, dev addresses, fee totals
- Prevents duplicate launches
- Powers the hourly summary stats

---

## Event Flow (Sequence)

```
Tweet arrives
    │
    ▼
Scanner detects github.com link
    │
    ▼
Evaluator scores repo
    │
    ├─ Score < threshold → discard
    │
    └─ Score ≥ threshold
            │
            ▼
        Launcher deploys token
            │
            ├──▶ Repo DB records launch
            │
            └──▶ Fee Router registers dev address
                        │
                        ▼
                  Swap occurs on-chain
                        │
                        ▼
                  Fee Router pays dev
                        │
                        ▼
                  Broadcaster posts to X
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Stream ingestion | X API v2 filtered stream |
| Repo metadata | GitHub REST API v3 |
| On-chain execution | Solana / Web3.js |
| Token launch | pump.fun protocol |
| Persistence | SQLite (local) / Redis (prod) |
| X posting | X API v2 OAuth2 |
| Runtime | Node.js / Python (agent core) |
