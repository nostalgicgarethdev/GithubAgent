# Token Model

## $GitAgent Token

The `$GitAgent` token is the native token of the GitAgent protocol on Solana.

**Contract Address:**
```
TBA — not yet deployed
```

---

## How Repo Tokens Work

When $GitAgent identifies a high-signal repository, it launches a dedicated token for that repo on Solana. These are separate tokens — one per repo — not the $GitAgent token itself.

### Repo Token Lifecycle

```
Repo detected
    │
    ▼
Signal score ≥ threshold
    │
    ▼
Token deployed (pump.fun)
    │
    ├── Symbol: derived from repo name (e.g. $kairos)
    ├── Dev address: registered as fee recipient
    └── Announced on X
            │
            ▼
        Community trades the token
            │
            ▼
        Fees flow to dev address automatically
```

---

## Signal Score Threshold

Repos must reach a signal score of **75/100** or above to qualify for a token launch.

The threshold exists to:
- Prevent spam launches from low-quality repos
- Ensure meaningful community interest before a token goes live
- Protect the protocol's reputation and fee routing integrity

The threshold may be adjusted by the agent operator based on observed market conditions.

---

## Fee Model

Every repo token launched by $GitAgent has a built-in fee on trades.

| Recipient | Share |
|-----------|-------|
| Developer (repo address) | 40% of protocol fee |
| Protocol treasury | 60% of protocol fee |

### Developer Fee Routing

The developer's fee address is determined by:
1. The repository's `FUNDING.yml` on-chain address (if present)
2. A registered Solana address linked to the GitHub account
3. A fallback treasury address held until the dev claims it

Fees accumulate per token. Developers can claim at any time.

---

## $GitAgent Token Utility

The primary `$GitAgent` token represents participation in the agent protocol itself.

| Use | Description |
|-----|-------------|
| Signal amplification | Holding $GitAgent may boost a repo's signal score weighting |
| Fee access | Future: fee sharing for $GitAgent holders from protocol revenue |
| Governance | Future: voting on threshold parameters and fee splits |

---

## Disclaimer

Repo tokens are launched algorithmically by the agent. They are not endorsed by the repository's developers unless those developers explicitly opt in. Token value is speculative. Not financial advice.
