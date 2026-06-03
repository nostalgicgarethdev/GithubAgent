# Fee Distribution

$GitAgent automatically routes a portion of every token's trading fees to the developers behind the repository that inspired the launch.

---

## How Developers Get Paid

### Step 1 — Token Launch
When $GitAgent launches a token for a repository, it records the **developer fee address** before the token goes live. No trade can happen without the address being set.

### Step 2 — Trade Occurs
Every buy or sell on the repo token generates a fee.

### Step 3 — Fee Router Executes
The Fee Router module monitors on-chain swap events. On each confirmed swap, it:
1. Calculates the developer's share
2. Executes a transfer instruction to the dev address
3. Logs the payment
4. Triggers a broadcast post on X

---

## Fee Address Resolution

$GitAgent resolves developer fee addresses in this order:

```
1. FUNDING.yml in the repo root
   └── Look for: solana: <address>

2. Registered address via GitAgent claim portal (future)

3. GitHub account linked Solana address (future)

4. Unclaimed treasury
   └── Fees held until developer claims them
```

### FUNDING.yml Example

Developers can opt in and set their Solana address by adding to their repo:

```yaml
# .github/FUNDING.yml
custom:
  - "solana:YourSolanaAddressHere"
```

$GitAgent scans for this file during repo evaluation. If found, the address is used immediately at launch.

---

## Claiming Unclaimed Fees

If a developer's repo gets a token launched before they've registered an address, fees accumulate in the protocol treasury tagged to that repo.

To claim:
1. Verify GitHub ownership of the repository
2. Submit a Solana address via the claim process (details TBA)
3. Protocol releases all accumulated fees to that address

Unclaimed fees are held indefinitely — they do not expire.

---

## Fee Distribution Summary

| Event | Action |
|-------|--------|
| Token launched | Dev address registered |
| Swap occurs | Dev fee calculated + sent |
| Fee sent | X post broadcast |
| Address not found | Fee held in treasury |
| Dev claims | Treasury releases to dev |

---

## Developer Rights

Developers whose repositories are tokenized by $GitAgent:
- **Do not need to do anything** to receive fees if a Solana address is registered
- Can **opt out** by opening an issue or contacting the agent operator — the token will be delisted and fees returned
- Are **not responsible** for the token or its market behavior
- **Did not endorse** the token launch unless they explicitly state otherwise

---

## Transparency

Every fee distribution is:
- Recorded on-chain (Solana transaction)
- Posted to X by the Broadcaster module
- Logged in the public agent activity feed

Anyone can verify payments at any time.
