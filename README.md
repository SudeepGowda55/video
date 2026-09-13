# Aqueduct — ~5:25 Demo Video Script

## Before you hit record — open these tabs, in this order, and pre-paste every query

**Division of labor: Vishruth owns tabs 3–7 (all four Graph tabs, plus the MCP terminal) — he
opens them, pastes the queries, and clicks Run/executes on his own screen when you cue him. You
own everything else (tabs 1, 2, 8, 9) and drive those yourself.** Nobody types a URL or pastes a
query live on camera — everything below is pre-loaded before recording starts.

| Tab | Owner        | URL                                                                                                                | Pre-paste this query into it, don't run it yet                                                                                                                                                                                                            |
| --- | ------------ | ------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | **You**      | `https://aqueduct-protocol.vercel.app/` (wallet connected)                                                         | — (this is the main dashboard, used most of the video)                                                                                                                                                                                                    |
| 2   | **You**      | `submission-screenshots/multiplier-effect-diagram.html` (the published artifact link)                              | —                                                                                                                                                                                                                                                         |
| 3   | **Vishruth** | `https://thegraph.com/studio/subgraph/ethonline/` → Playground                                                     | `{ exposurePositions(first: 3) { id committedAmount makerWalletBalance exposureBps status updatedAt } }`                                                                                                                                                  |
| 4   | **Vishruth** | `https://thegraph.com/explorer/subgraphs/JCNWRypm7FYwV8fx5HhzZPSFaMxgkPuw4TnR3Gpi81zk?view=Query` (Aave, Ethereum) | `{ protocols(first: 1) { id protocol name slug schemaVersion network type totalValueLockedUSD cumulativeUniqueUsers } markets(first: 5, orderBy: totalValueLockedUSD, orderDirection: desc) { id name totalValueLockedUSD inputToken { symbol name } } }` |
| 5   | **Vishruth** | `https://thegraph.com/explorer/subgraphs/FUbEPQw1oMghy39fwWBFY5fE6MXPXZQtjncQy2cXdrNS?view=Query` (Uniswap, Base)  | `{ liquidityPools(first: 5, orderBy: totalValueLockedUSD, orderDirection: desc) { id name totalValueLockedUSD cumulativeVolumeUSD } }`                                                                                                                    |
| 6   | **Vishruth** | `https://thegraph.com/explorer/subgraphs/43s9hQRurMGjuYnC1r2ZwS6xSQktbFyXMPMqGKUFJojb?view=Query` (Agent0, Base)   | `{ agentRegistrationFiles(where: {active: true}, first: 2) { agentId name mcpEndpoint } agents(first: 2) { id chainId agentId owner } }`                                                                                                                  |
| 7   | **Vishruth** | Terminal, MCP server running, with a saved JSON-RPC input file ready to pipe in                                    | (see 1:45 below for the exact command)                                                                                                                                                                                                                    |
| 8   | **You**      | Terminal, this repo checked out, ready to run `forge test --summary`                                               | (see 4:30 below)                                                                                                                                                                                                                                          |
| 9   | **You**      | `https://github.com/SudeepGowda55/Aqueduct` (or the README rendered on GitHub)                                     | —                                                                                                                                                                                                                                                         |

> Two more subgraph blocks exist but aren't tied to a specific beat below — Vishruth can paste
> them into Tab 3 if you want extra Playground material to show while narrating:
> `{ exposureSnapshots(first: 3) { id } }` and `{ liquidityPools(first: 3) { id } }` and
> `{ swaps(first: 3) { id } }`.
>
> WARNING: Studio queries fail on Explorer and Explorer queries fail on Studio — each query above
> is pre-paired with the one tab it actually works on. Don't cross-paste them.

---

## 0:00–1:10 — Introduction + the problem (~68 sec, ~170 words)

**[Tab 2 — the multiplier-effect diagram, full-screen — timed to land as you say "each one looks
completely safe"]**

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

## 1:10–5:25 — Live execution, following the page top to bottom (4:15, ~620 words)

**[Switch to Tab 1 — `aqueduct-protocol.vercel.app`, scrolled to the very top]**

### 1:10–1:25 (15s) — "Why this maker is exposure-gated" panel → "Maker exposure" section with the Exposure Gauge, reading 10%

> **SAY:**
> "This is Aqueduct, live on Base Sepolia — nothing here is mocked. We added a new SwapVM
> instruction, exposure-gate-one-D, that reads a maker's real aggregate exposure from an on-chain
> oracle and derates or halts their fill. Right now this maker's exposure gauge reads 10% —
> safe."

### 1:25–1:45 (20s) — "Same strategy. Same risk policy. Different execution venue." — the "✓ EXACT MATCH — BIT-EXACT" badge already showing, no click needed

> **SAY:**
> "Scrolling down — here's the core claim, computed live: the same strategy backs two execution
> venues, and the predicted output is bit-for-bit identical whether it fills through SwapVM or
> through Uniswap v4. We'll prove that for real with actual swaps in a minute."

### 1:45–2:20 (35s) — Tab 3 (Studio) → Tab 4 (Aave) → Tab 5 (Uniswap) → Tab 6 (Agent0) → Tab 1 (Graph panels) → Tab 7 (MCP)

This one is a sequence across six screens, not a single on-screen state, so it doesn't fold into
one header line the way the others do — the header above just names the path.

**This beat is interleaved, not "say it all then click" — each line of narration is timed to
land right after the matching tab is already up on screen. Say the line, THEN cue the next tab
while it's loading/settling, not the reverse.**

> **SAY (Tab 1, before cueing anyone):**
> "All of this is backed by The Graph."

**[Cue Vishruth → Tab 3 (Studio Playground) → he clicks Run on the pre-pasted
`exposurePositions` query]**

> **SAY (while his result is on screen):**
> "Quick split: we built one subgraph ourselves — that's `ethonline` — it tracks this maker,
> `0x5067...132be`, across ten positions and drives this safety banner live. Here's one, live
> right now: 100k committed against a 1.39 million wallet, 1000 basis points, status SAFE —
> that's the same 10% you just saw on the gauge."

**[Cue Vishruth → Tab 4 (Aave Explorer) → he clicks Run]**

> **SAY:**
> "Then we compose with public subgraphs other teams maintain — Aave —"

**[Cue Vishruth → Tab 5 (Uniswap Base Explorer) → he clicks Run]**

> **SAY:**
> "Uniswap —"

**[Cue Vishruth → Tab 6 (Agent0 Base Explorer) → he clicks Run]**

> **SAY:**
> "and Agent0 — using the same Messari-standard shape, so the same query pattern that works on
> ours works on theirs too."

**[Back to you, Tab 1 → scroll past "Maker safety" → "Exposure, from The Graph" → "Pool activity,
from The Graph (Messari shape)"]**

> **SAY:**
> "Below that, real exposure history and real indexed swaps,"

**[Cue Vishruth → Tab 7 (his terminal) → he runs:**

```
node mcp/server.js < saved-mcp-input.jsonl
```

**→ hold on the `maker_safety_verdict` response]**

> **SAY:**
> "and over MCP an agent can just ask 'is this maker safe?' — no GraphQL, no fake numbers."

### 2:20–2:50 (30s) — "Swap directly via SwapVM" / "Swap via Uniswap v4" row (both default to amount 1, already matching)

> **SAY:**
> "Now let's actually do it, for real. Swap directly through SwapVM — [click, wait for the
> confirmation toast] — real output, right there. Now the same size through the Uniswap v4 pool
> sourced by our custom hook — [click, wait for the toast] — same output again. Genuinely
> executed, not just predicted — because the v4 pool has zero liquidity of its own, every fill
> comes from that same Aqua strategy."

**[Click Swap on the SwapVM panel → hold one beat on the confirmation toast → click Swap on the
Uniswap v4 panel → hold one beat on its toast → hold both toasts on screen together so the
matching output numbers are readable side by side. If Base Sepolia confirmation is slow, cut on
the toast rather than waiting live.]**

### 2:50–3:25 (35s) — "Strategy P (price + risk aware)" / "Risk-adjusted dynamic fee (Uniswap v4)" row

> **SAY:**
> "This maker also runs a more sophisticated program, reading a real live Chainlink ETH/USD feed
> to improve pricing, still gated by the same exposure check on top. And separately, a second,
> independent Uniswap v4 mechanism: a swap fee that scales with this same live exposure. Here's
> the predicted fee next to the persisted on-chain fee — I'll hit refresh — [click] — a real
> transaction, no swap required."

**[Point at the live Chainlink price, then click "Push current fee on-chain (refreshFee)" on the
dynamic-fee panel]**

### 3:25–3:40 (15s) — "Ungated vs. exposure-gated" panel

This is a read-only panel — no wallet, no transaction, nothing for Vishruth to do. You just point
at the two numbers that are already sitting on screen after the previous scroll.

> **SAY:**
> "One more comparison, and this one needs no wallet at all — both numbers come straight from
> public reads. It prices the same 100-token swap two ways: what the raw pool curve alone would
> pay out with no exposure gate, against what it actually pays out right now. At this maker's
> current 10% exposure they're identical, because 10% is well under the derate threshold — the
> gate only starts biting once exposure climbs, which is exactly what we're about to push it to."

**[Point at the two output numbers side by side — "Ungated" and "Exposure-gated" — then the
footer line underneath showing this maker's exposure percentage and committed amount for this
strategy. No clicking needed; the toggle/amount field can stay at its default.]**

### 3:40–4:05 (25s) — "Maker risk policy" / "Maker emergency halt" row

> **SAY:**
> "The maker also has an independent kill switch. [click "Pause (emergency halt)"] Now any fill
> on either venue reverts with the exact expected error. [attempt swap, show revert] Unpausing
> restores it immediately. [click "Unpause"]"

### 4:05–4:25 (20s) — "Keeper control (demo)" / "Activity" row

> **SAY:**
> "Last piece: the keeper is what pushes real exposure readings on-chain. Let's push this maker
> to 70%. [click]"

**[Click the "Derated (70%)" preset button — the "Activity" panel shows the real transaction]**

### 4:25–4:45 (20s) — Exposure Gauge (now "Derated") / "Swap directly via SwapVM" panel — the one deliberate callback

> **SAY:**
> "Scrolling back up for a second — the gauge already reads derated, and the exact same swap now
> fills for noticeably less, automatically, because it's the same program reacting to the same
> new reading. And if we scrolled back down to that comparison panel right now, the ungated and
> gated numbers would finally split apart."

**[Attempt one more swap, show the reduced output]**

### 4:45–5:05 (20s) — Terminal (Tab 8), `forge test --summary`

> **SAY:**
> "And it's backed by real tests, not just a working demo. Forty-nine Foundry tests across ten
> suites — including a stateful-fuzz invariant suite that ran 128,000 randomized calls checking
> committed-balance accounting never drifts. All green."

```
forge test --summary
```

**[Let the passing suite table sit on screen for a beat before cutting away]**

### 5:05–5:25 (20s) — Tab 9, README / GitHub repo page

> **SAY:**
> "Every number you've seen today is live on Base Sepolia, independently verifiable on-chain.
> Full developer feedback for Uniswap is in our repo's `FEEDBACK.md`. One risk guarantee, two
> execution venues, one live data layer connecting them. That's Aqueduct."

---

## Production notes

- Nobody should type a URL or paste a query live on camera — every tab in the checklist above is
  opened and pre-loaded _before_ recording starts. Vishruth drives tabs 3–7 (the four Graph tabs
  plus his MCP terminal) on his own screen when you cue him; you drive everything else (tabs 1,
  2, 8, 9) yourself — that's the only live coordination needed for the whole video.
- The swap transactions (2:20), the fee refresh (2:50), the pause/unpause (3:40), and the keeper
  push (4:05) are the segments most likely to run long if a transaction confirmation is slow on
  Base Sepolia — consider recording those in a separate take and cutting on the confirmation
  toast rather than waiting live.
- The "Ungated vs. exposure-gated" panel (3:25) is read-only and needs no wallet or transaction —
  it's the one beat in the whole live-execution block where you can just talk and point, no
  clicking, so use it as a breather between the fee-refresh click and the pause/unpause click.
- The whole demo follows the page top to bottom in one pass, with exactly one deliberate
  scroll-back-up at 4:25 to show the keeper push taking effect — that's intentional, not a
  mistake, so call it out verbally ("scrolling back up for a second") rather than cutting to it
  silently.
- The Aave/Uniswap/Agent0 Explorer links (tabs 4–6) are public third-party subgraphs Vishruth
  verified working on the day this script was written — give them one live check before
  recording, since external subgraphs can change or move without our control.
- Graph proof links for the submission description (verified live):
  - Ours (owned): https://thegraph.com/studio/subgraph/ethonline/ — v0.4.0, deployed, Base
    Sepolia, synced 100%, 330 entities.
  - Maker on-chain activity: https://sepolia.basescan.org/txs?a=0x5067591c365d7d69d76b725c2d9af7b9437132be
  - The multiplier-effect diagram: https://claude.ai/code/artifact/88d42e02-0470-45d5-bdcd-e4293fdf4081
