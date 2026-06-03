# FAQ

---

**What is $GitAgent?**

$GitAgent is an autonomous on-chain agent that monitors X/Twitter for GitHub repository links, launches tokens for high-signal repos on Solana, routes trading fees to the developers, and posts every action back to X.

---

**Who runs $GitAgent?**

The agent runs autonomously. It is not controlled by any central team on a trade-by-trade basis. Parameters (score threshold, fee splits) can be updated by the agent operator but day-to-day actions are fully automated.

---

**My repo got a token. Did I ask for this?**

No. Token launches are algorithmic and based on public X/Twitter engagement. Developers do not need to opt in for a launch to happen.

If you want to **claim your dev fees**, see [Fee Distribution](FEE_DISTRIBUTION.md).

If you want to **opt out**, open an issue in this repository with your GitHub repo URL and a Solana address for any accumulated fees. The token will be delisted and fees returned.

---

**How do I receive my developer fees?**

Add a Solana address to your repo's `.github/FUNDING.yml`:

```yaml
custom:
  - "solana:YourSolanaAddressHere"
```

$GitAgent scans for this during evaluation. If the token is already live, fees will route to your address from the next swap onward. Any prior unclaimed fees can be claimed — see [Fee Distribution](FEE_DISTRIBUTION.md).

---

**What makes a repo get tokenized?**

A composite signal score of 75/100 or above. The score weighs:
- X/Twitter engagement (likes, retweets, replies)
- GitHub repo health (stars, recent activity)
- Mention velocity
- Unique mentioner count

See [Architecture](ARCHITECTURE.md) for the full scoring breakdown.

---

**Can any repo get a token?**

Any public GitHub repo that appears in X/Twitter tweets can be evaluated. Private repos, forks with no original activity, and repos flagged as spam or malicious are filtered out.

---

**Is $GitAgent the same as the repo tokens it launches?**

No. `$GitAgent` is the protocol's own token. The tokens launched for individual repos (e.g. `$kairos`, `$hermesagent`) are separate — one per repo. $GitAgent launching a repo token does not mean those tokens are affiliated with the $GitAgent protocol beyond the fee routing mechanism.

---

**Is this financial advice?**

No. All token launches are algorithmic. Token values are speculative. Do your own research.

---

**Where can I follow the agent's activity?**

On X/Twitter: search `$GitAgent` or follow `@GitAgentAI`.

Every launch, fee payment, and hourly summary is posted automatically.

---

**How do I report a bug or a bad launch?**

Open an issue in this repository. For urgent matters (incorrect fee address, harmful repo tokenized), tag the issue `urgent`.
