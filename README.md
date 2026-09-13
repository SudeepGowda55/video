# Aqueduct — 5 Minute Demo Video Script

## 0:00–1:10 — Introduction + the problem (~68 sec, ~170 words)

**[ON SCREEN: title slide, then bring up the diagram —**
**`submission-screenshots/multiplier-effect-diagram.html`** (open the published artifact link
full-screen) **— timed to land as you say "each one looks completely safe"]**

> **SAY:**
> "Hi, I'm Sudeep, and along with my teammate Vishruth, we built Aqueduct.
>
> So here's the thing — in DeFi, risk checks today only ever look at one strategy at a time, and
> that's exactly where real exposure hides. Say a maker has a $100 wallet, and commits to three
> separate strategies: $50, $40, $40. Checked alone, each one looks completely safe — 50%, 40%,
> 40% utilization, comfortably under any sane threshold. But all three are checked against the
> same full $100, as if the other two didn't exist. Add them up and that's $130 promised against
> $100 that's actually there — thirty dollars that simply isn't real.
>
> That gap stays hidden until demand hits all three strategies close together — whoever pulls
> first drains the real balance, whoever's last just fails, and bots race each other the moment
> they spot it. Three honestly-safe checks, and the maker's still over-committed by 30%. That's
> the multiplier effect, and it's the actual problem Aqueduct solves. Let's see the fix running
> for real."

---

## 1:10–5:10 — Live execution, following the page top to bottom (4:00, ~600 words)

**[ON SCREEN: open `aqueduct-protocol.vercel.app`, wallet connected, scrolled to the very top]**

This whole block follows the real page layout in order — no jumping around — with exactly ONE
deliberate callback near the end (scrolling back up after the keeper push, clearly signposted).

### 1:10–1:25 (15s) — Orient + name the mechanism

> **SAY:**
> "This is Aqueduct, live on Base Sepolia — nothing here is mocked. We added a new SwapVM
> instruction, exposure-gate-one-D, that reads a maker's real aggregate exposure from an on-chain
> oracle and derates or halts their fill. Right now this maker's exposure gauge reads 10% —
> safe."

**[Already on screen after scroll: "Why this maker is exposure-gated" panel → "Maker exposure"
section with the Exposure Gauge, reading 10%]**

### 1:25–1:45 (20s) — Cross-venue proof (predicted, not yet executed)

> **SAY:**
> "Scrolling down — here's the core claim, computed live: the same strategy backs two execution
> venues, and the predicted output is bit-for-bit identical whether it fills through SwapVM or
> through Uniswap v4. We'll prove that for real with actual swaps in a minute."

**[Scroll to "Same strategy. Same risk policy. Different execution venue." — the
"✓ EXACT MATCH — BIT-EXACT" badge is already showing, no click needed]**

### 1:45–2:20 (35s) — The Graph: owned vs borrowed, live verdict, history, MCP

> **SAY:**
> "All of this is backed by The Graph. Quick split: we made one small subgraph ourselves — that's ethonline — it tracks this maker 0x5067...132be with ten positions and drives this safety banner live. Here's one live: 100k committed against 1.39M wallet, 1000 bps, status SAFE — that's the 10% gauge. Then we borrow data from big public subgraphs like Aave and Uniswap that other teams maintain — same Messari shape, so the same query works here and there. Below is real exposure history and real indexed swaps, and over MCP an agent can just ask 'is this maker safe?' — no GraphQL needed, no fake numbers."

**[Show Studio Playground exposurePositions{ committedAmount makerWalletBalance exposureBps status } screenshot, then scroll past "Maker safety" → "Exposure, from The Graph" → "Pool activity, from The Graph
(Messari shape)" → cut to a terminal running a saved MCP tool-call input, showing the response]**

### 2:20–2:50 (30s) — Actually swap on both venues

> **SAY:**
> "Now let's actually do it, for real. Swap directly through SwapVM — [click, wait toast] — 0.42 Aqua out. Now the same size through the Uniswap v4 pool sourced by our custom hook — [click, wait toast] — 0.42 out again. Same output, genuinely executed, not just predicted — because the v4 pool has zero liquidity of its own, every fill comes from that same Aqua strategy. Pause on the two toasts side by side."

**[Scroll to the "Swap directly via SwapVM" / "Swap via Uniswap v4" row → click Swap on each →
hold 1 beat on each confirmation toast → point back at the matching output amounts. If Base Sepolia is slow, cut on toast, do not wait live.]**

### 2:50–3:25 (35s) — Strategy P + the dynamic-fee pool

> **SAY:**
> "This maker also runs a more sophisticated program, reading a real live Chainlink ETH/USD feed
> to improve pricing, still gated by the same exposure check on top. And separately, a second,
> independent Uniswap v4 mechanism: a swap fee that scales with this same live exposure. Here's
> the predicted fee next to the persisted on-chain fee — I'll hit refresh — [click] — a real
> transaction, no swap required."

**[Scroll to the "Strategy P (price + risk aware)" / "Risk-adjusted dynamic fee (Uniswap v4)"
row — point at the live Chainlink price, then click "Push current fee on-chain (refreshFee)" on
the dynamic-fee panel]**

### 3:25–3:50 (25s) — Maker emergency pause

> **SAY:**
> "The maker also has an independent kill switch. [click "Pause (emergency halt)"] Now any fill
> on either venue reverts with the exact expected error. [attempt swap, show revert] Unpausing
> restores it immediately. [click "Unpause"]"

**[Scroll to the "Maker risk policy" / "Maker emergency halt" row]**

### 3:50–4:10 (20s) — Keeper pushes real exposure

> **SAY:**
> "Last piece: the keeper is what pushes real exposure readings on-chain. Let's push this maker
> to 70%. [click]"

**[Scroll to the "Keeper control (demo)" / "Activity" row — click the "Derated (70%)" preset
button, the "Activity" panel shows the real transaction]**

### 4:10–4:30 (20s) — The one deliberate callback: scroll back up

> **SAY:**
> "Scrolling back up for a second — the gauge already reads derated, and the exact same swap now
> fills for noticeably less, automatically, because it's the same program reacting to the same
> new reading."

**[Scroll back up to the Exposure Gauge (now showing "Derated") and/or the "Swap directly via
SwapVM" panel — attempt one more swap, show the reduced output]**

### 4:30–4:50 (20s) — Test suite proof

> **SAY:**
> "And it's backed by real tests, not just a working demo. Forty-nine Foundry tests across ten
> suites — including a stateful-fuzz invariant suite that ran 128,000 randomized calls checking
> committed-balance accounting never drifts. All green."

**[Cut to terminal, run `forge test --summary`, let the passing suite table sit on screen for a
beat before cutting away]**

### 4:50–5:10 (20s) — Close

> **SAY:**
> "Every number you've seen today is live on Base Sepolia, independently verifiable on-chain.
> Full developer feedback for Uniswap is in our repo's `FEEDBACK.md`. One risk guarantee, two
> execution venues, one live data layer connecting them. That's Aqueduct."

**[README or GitHub repo page]**

---

## Production notes

- Have the terminal for the MCP call ready with a saved JSON-RPC input file so the tool call
  resolves instantly on camera (avoids dead air waiting on a live subgraph round-trip).
- The swap transactions (2:20), the fee refresh (2:50), the pause/unpause (3:25), and the keeper
  push (3:50) are the segments most likely to run long if a transaction confirmation is slow on
  Base Sepolia — consider recording those in a separate take and cutting on the confirmation
  toast rather than waiting live.
- The whole demo now follows the page top to bottom in one pass, with exactly one deliberate
  scroll-back-up at 4:10 to show the keeper push taking effect — that's intentional, not a
  mistake, so call it out verbally ("scrolling back up for a second") rather than cutting to it
  silently.
- The multiplier-effect diagram for the opening: https://claude.ai/code/artifact/88d42e02-0470-45d5-bdcd-e4293fdf4081
- Graph proof links for description (only working, verified today):
  - Ours (owned): https://thegraph.com/studio/subgraph/ethonline/ — v0.4.0 DEPLOYED Base Sepolia SYNCED 100% 330 entities
  - Maker on-chain (working): https://sepolia.basescan.org/txs?a=0x5067591c365d7d69d76b725c2d9af7b9437132be

## Copy-paste queries — USE ONLY WHERE LABELED

> WARNING: Studio queries FAIL on Explorer and Explorer queries FAIL on Studio. Paste each block ONLY at its URL below.

```graphql
"USE ONLY at https://thegraph.com/studio/subgraph/ethonline/ > Playground"
{ makers(first: 5) { id positionIds } }
```

```graphql
{
  exposurePositions(first: 3) {
    id
    committedAmount
    makerWalletBalance
    exposureBps
    status
    updatedAt
  }
}
```

```graphql
{ exposureSnapshots(first: 3) { id } }
```

```graphql
{ liquidityPools(first: 3) { id } }
```

```graphql
{ swaps(first: 3) { id } }
```

## Copy-paste queries (Explorer — borrowed)

Aave ETH: https://thegraph.com/explorer/subgraphs/JCNWRypm7FYwV8fx5HhzZPSFaMxgkPuw4TnR3Gpi81zk?view=Query
```graphql
{
  protocols(first: 1) {
    id
    protocol
    name
    slug
    schemaVersion
    network
    type
    totalValueLockedUSD
    cumulativeUniqueUsers
  }
  markets(first: 5, orderBy: totalValueLockedUSD, orderDirection: desc) {
    id
    name
    totalValueLockedUSD
    inputToken { symbol name }
  }
}
```

Uniswap Base: https://thegraph.com/explorer/subgraphs/FUbEPQw1oMghy39fwWBFY5fE6MXPXZQtjncQy2cXdrNS?view=Query
```graphql
{
  liquidityPools(first: 5, orderBy: totalValueLockedUSD, orderDirection: desc) {
    id
    name
    totalValueLockedUSD
    cumulativeVolumeUSD
  }
}
```

Agent0 Base: https://thegraph.com/explorer/subgraphs/43s9hQRurMGjuYnC1r2ZwS6xSQktbFyXMPMqGKUFJojb?view=Query
```graphql
{
  agentRegistrationFiles(where: {active: true}, first: 2) {
    agentId
    name
    mcpEndpoint
  }
  agents(first: 2) {
    id
    chainId
    agentId
    owner
  }
}
```
