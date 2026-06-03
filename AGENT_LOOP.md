# Agent Loop

$GitAgent runs a continuous autonomous loop. This document describes the full cycle from tweet ingestion to on-chain action and back to the feed.

---

## The Loop

```
┌─────────────────────────────────────────────┐
│                                             │
│   ┌────────┐                                │
│   │ START  │                                │
│   └───┬────┘                                │
│       │                                     │
│       ▼                                     │
│  ┌─────────────────────┐                   │
│  │ Connect X API stream │                   │
│  └──────────┬──────────┘                   │
│             │                               │
│       ┌─────▼──────┐                        │
│  ┌────┤ Tweet arrives├────┐                 │
│  │    └────────────┘     │                  │
│  │                       │                  │
│  ▼                       ▼                  │
│ No github.com       Has github.com link      │
│ link → discard      │                        │
│                     ▼                        │
│              ┌─────────────┐                 │
│              │ Seen before? │                │
│              └──┬───────┬──┘                 │
│                Yes      No                   │
│                 │       │                    │
│              discard    ▼                    │
│                   ┌──────────────┐           │
│                   │ Fetch repo   │           │
│                   │ metadata     │           │
│                   └──────┬───────┘           │
│                          │                   │
│                          ▼                   │
│                   ┌──────────────┐           │
│                   │ Score repo   │           │
│                   └──────┬───────┘           │
│                          │                   │
│               ┌──────────┴──────────┐        │
│            < 75                   ≥ 75        │
│               │                    │         │
│            discard          ┌──────▼──────┐  │
│                             │Launch token │  │
│                             └──────┬──────┘  │
│                                    │         │
│                                    ▼         │
│                             ┌──────────────┐ │
│                             │Register dev  │ │
│                             │fee address   │ │
│                             └──────┬───────┘ │
│                                    │         │
│                                    ▼         │
│                             ┌──────────────┐ │
│                             │Monitor swaps │ │
│                             └──────┬───────┘ │
│                                    │         │
│                              swap occurs     │
│                                    │         │
│                                    ▼         │
│                             ┌──────────────┐ │
│                             │Pay dev fee   │ │
│                             └──────┬───────┘ │
│                                    │         │
│                                    ▼         │
│                             ┌──────────────┐ │
│                             │Post to X     │ │
│                             └──────┬───────┘ │
│                                    │         │
│                              back to top ◄───┘
│                                             │
└─────────────────────────────────────────────┘
```

---

## Loop Timing

| Task | Frequency |
|------|-----------|
| X stream ingestion | Continuous (real-time) |
| Repo metadata fetch | On detection |
| Signal scoring | On detection |
| Token launch | On score ≥ threshold |
| Swap monitoring | Continuous (on-chain listener) |
| Fee routing | Per swap event |
| X broadcast | Per event (rate-limited) |
| Hourly summary post | Every 60 minutes |
| Repo DB cleanup | Daily |

---

## Error Handling

The agent is designed to fail gracefully:

| Failure | Behaviour |
|---------|-----------|
| X stream disconnect | Reconnect with exponential backoff (max 5 min) |
| GitHub API rate limit | Queue and retry after reset window |
| Solana RPC error | Retry up to 3x, then discard with log |
| Token launch failure | Log, skip repo, do not retry same repo for 24h |
| Fee routing failure | Retry 3x, then hold in treasury |
| X post failure | Queue post, retry after rate limit window |

All errors are logged and, where material (launch failure, fee routing failure), broadcast to the agent operator.

---

## Post Formats

### Token Launch Post
```
⚡ Token launched: $[symbol]

Repo: github.com/[user]/[repo]
Signal score: [score]/100
Dev fees: routing automatically on every trade 💸

CA: [mint_address]
```

### Fee Distribution Post
```
💸 Dev fee sent:
[amount] SOL → github.com/[user]/[repo]

Builders get paid when the community trades. 🤝
```

### Hourly Summary Post
```
📊 Last hour:
• [n] repos scanned
• [n] tokens launched  
• [amount] SOL sent to devs
• [n] swaps processed

Agent running 24/7. 🤖
```

---

## Rate Limits

The agent respects all API rate limits:

| API | Limit | Agent behaviour |
|-----|-------|-----------------|
| X filtered stream | 1 connection | Maintained continuously |
| X write API | 300 posts / 3h | Post queue with pacing |
| GitHub REST | 5000 req / h | Cached, batch fetched |
| Solana RPC | Provider dependent | Retry with backoff |
