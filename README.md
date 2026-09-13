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

## 1:00–4:50 — Live execution + explanation, woven together (3:50, ~575 words)

**[ON SCREEN: open `aqueduct-protocol.vercel.app`, wallet connected]**

### 1:00–1:20 (20s) — Orient + name the mechanism

> **SAY:**
> "This is Aqueduct, live on Base Sepolia — nothing here is mocked. We added a new SwapVM
> instruction, `_exposureGate1D`, that reads a maker's real aggregate exposure from an on-chain
> oracle and derates or halts their fill accordingly."

**[Aggregate Exposure panel → Exposure Gauge]** Right now, the maker's exposure is 10%.

### 1:00–1:50 → 1:20–1:50 (30s) — Cross-venue proof

> **SAY:**
> "Here's the proof that matters: the same gated strategy backs two execution venues. I'll swap
> directly through SwapVM — [click] — and now through a Uniswap v4 pool sourced by our custom
> hook — [click]. Look at the outputs: bit-for-bit identical. Same gate, same math, enforced
> twice, because the v4 pool has zero liquidity of its own — every fill comes from that same Aqua
> strategy."

**[SwapVM panel swap → Uniswap v4 panel swap → Cross-Venue Proof "EXACT MATCH — BIT-EXACT" badge]**

### 1:50–2:20 (30s) — Push exposure, watch both venues react

> **SAY:**
> "Now watch what happens when risk changes. The keeper pushes a higher exposure reading — 70%.
> [click] The gate derates the fill on both venues identically, because it's the same underlying
> program running twice, not two systems that happen to agree. Push it high enough and it halts
> outright — the maker can never trade past what their own program already authorized, even from
> a malicious oracle reading."

**[Keeper panel preset → Exposure Gauge updates → one swap showing reduced output]**

### 2:20–2:45 (25s) — Strategy P: composed with a real price oracle

> **SAY:**
> "This maker also runs a more sophisticated program: price-adjusted and risk-gated together. It
> reads a real, live Chainlink ETH/USD feed to improve the taker's price, capped, and the
> exposure gate still applies on top — neither instruction can override the other's direction."

**[Strategy P panel — point at the live Chainlink price]**

### 2:45–3:15 (30s) — Dynamic fee: a second Uniswap capability

> **SAY:**
> "Separately, we built a second, independent Uniswap v4 mechanism: a swap fee that scales with
> this same live exposure, using v4's own dynamic-fee API. Here's the predicted fee from the live
> oracle, next to the persisted on-chain fee. I'll hit refresh — [click] — that's a real
> transaction pushing the fee on-chain, no swap required."

**[Dynamic Fee Pool panel — click "Push current fee on-chain"]**

### 3:15–3:35 (20s) — Maker emergency pause

> **SAY:**
> "The maker also has an independent kill switch. [click pause] Now any fill on either venue
> reverts with the exact expected error. [attempt swap, show revert] Unpausing restores it
> immediately. [click unpause]"

**[Emergency Pause panel]**

### 3:35–4:00 (25s) — The Graph: live verdict + agent tooling

> **SAY:**
> "All of this is backed by The Graph. This banner is a reasoned safety verdict computed from our
> subgraph. Below it, real indexed swaps in a Messari-standardized schema — the same query
> pattern that works on any standard DEX subgraph works here. And an AI agent can just ask 'is
> this maker safe?' over MCP and get a real answer."

**[Graph Verdict Banner → Graph Pool Activity panel → terminal running an MCP tool call]**

### 4:00–4:25 (25s) — Test suite proof

> **SAY:**
> "And it's backed by real tests, not just a working demo. Forty-nine Foundry tests across ten
> suites — including a stateful-fuzz invariant suite that ran 128,000 randomized calls checking
> committed-balance accounting never drifts. [switch to terminal] All green."

**[Switch to terminal, run `forge test --summary`, let the passing suite table sit on screen for
a beat before cutting away]**

### 4:25–4:50 (25s) — Close

> **SAY:**
> "Every number you've seen today is live on Base Sepolia, independently verifiable on-chain.
> Full developer feedback for Uniswap is in our repo's `FEEDBACK.md`. One risk guarantee, two
> execution venues, one live data layer connecting them. That's Aqueduct."

**[README or GitHub repo page]**

---

## Production notes

- Have the terminal for the MCP call ready with a saved JSON-RPC input file so the tool call
  resolves instantly on camera (avoids dead air waiting on a live subgraph round-trip).
- The two swap transactions (1:20 and 1:50) and the pause/unpause (3:15) are the segments most
  likely to run long if a transaction confirmation is slow on Base Sepolia — consider recording
  those in a separate take and cutting on the confirmation toast rather than waiting live.
- The multiplier-effect diagram for the opening: https://claude.ai/code/artifact/88d42e02-0470-45d5-bdcd-e4293fdf4081
