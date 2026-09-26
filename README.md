# Iceberg: demo video script (4:00, live on Base mainnet)

## Words used in this script

| Word | Meaning | Say it |
|---|---|---|
| **AMM** | Automated Market Maker. A smart contract that holds a pool of two tokens (here ETH and USDC) and trades with anyone directly, pricing by a formula. Uniswap is one. 1inch itself is an aggregator (it routes trades to the best AMM), but a 1inch Aqua strategy can run an AMM formula, and that's what Iceberg's 1inch side is. | "A-M-M" |
| **Partially Active AMM** | Only part of the pool can trade in each block. The rest is frozen for that block and earns yield in Morpho. From the paper arXiv 2602.09887 (Ko, 2026). | |
| **LVR** | Loss-versus-rebalancing: what an LP loses because arbitrage bots trade against the pool's stale price (Milionis et al., 2022). | "L-V-R" |
| **λ** | Greek letter lambda: the fraction of the pool that can trade this block. "λ is 80%" means 80% can trade. | "LAM-duh" (silent b) |
| **Venue** | A place where traders swap against Iceberg's liquidity. There are two, running the same math. | |
| **SwapVM instruction** | 1inch's trading engine runs a strategy as a small program of instructions (opcodes). Iceberg adds its own: `PAActiveReserves` (the per-block freeze) and `ChainlinkDeviationGuard` (price check), next to 1inch's stock fee and x·y=k instructions. The 1inch track scores SwapVM use higher. | "swap-V-M" |
| **Venue 1: 1inch Aqua** | The maker ships a strategy to 1inch Aqua. Takers swap through `IcebergRouter`, where a new SwapVM instruction (`PAActiveReserves`) freezes the passive part each block. | |
| **Venue 2: Uniswap v4** | A v4 pool whose hook (`IcebergHook`) does the same per-block freeze. | |
| **1inch official router** | Not a third Iceberg venue. It's a plain strategy on 1inch's own router, backed by the same Morpho balance as the Aqua position (Aqua shared liquidity). | |
| **Keeper** | A background script. Every 20 s it copies the live ETH price, picks λ, parks or unparks idle funds in Morpho, and rebalances when the mix drifts. | |

## What `./scripts/start_local.sh` does (local fork: used by the test clip and as a fallback)

Everything runs on a private copy of Base mainnet on your laptop. Nothing touches the real chain, and your real wallet isn't used.

1. **Local Base fork.** Starts `anvil` from the latest Base block through your RPC, at `http://127.0.0.1:8555`. The real 1inch Aqua, 1inch official router, Uniswap v4 PoolManager and Morpho vaults are all there.
2. **Test wallets.** Uses anvil's public test keys (fake money, only on the fork):
   - the maker gets 20,000 USDC and 2 WETH, plus more for the v4 pool;
   - three takers (demo script, UI buttons, terminal commands) get 50,000 USDC and 10 WETH each.
3. **Price feed.** Chainlink doesn't update on a fork. A small `MirrorFeed` starts at the live Base Chainlink ETH/USD price, and the keeper keeps copying the live price into it.
4. **Venue 1: 1inch Aqua.**
   - Deploys `IcebergRouter`, the vault hooks, `IcebergParams` (λ settings) and `IcebergLens`.
   - The maker deposits 2 WETH and the matching USDC into Morpho vaults, then ships the Iceberg strategy to Aqua.
   - Also ships the small shared strategy on 1inch's official router.
5. **Venue 2: Uniswap v4.**
   - Mines a hook address with the right permission bits and deploys `IcebergHook` through the standard CREATE2 deployer.
   - Creates the pool and adds 2 WETH plus matching USDC. The idle share is parked in Morpho.
   - Takers approve the three contracts they trade through.
   - All addresses go to `deployments/local.json`.
6. **Keeper and UI.** Starts the keeper (log in `.run/keeper.log`) and the web app at `http://localhost:8788/`, with the API at `/api/status`.

**READY** means you can open the fork UI at `localhost:8788`.

## Before you hit record (15 min ahead)

**1. Record two short proof clips first.** They take minutes to run, so don't do them live:
```bash
cd ~/projects/ethglobal-online/iceberg
./scripts/test_all.sh              # record only the last ~15 lines ending in "ALL CHECKS PASSED."
REPLAY_MINUTES=1440 forge test --match-contract Replay21Sep -vv   # record the 5 result lines
```
`test_all.sh` starts its own stack with the arbitrageur on, then makes test swaps and a rebalance. That's why the fresh stack comes next. The replay runs on its own separate fork.

**2. Mainnet UI, right before recording.** Your contracts are live on Base, so every trade is a real transaction you can open on Basescan:
```bash
cd ~/projects/ethglobal-online/iceberg/frontend
set -a; source ../.env; set +a
MAINNET_UI_TRADES=1 npx next dev -p 8790      # open http://localhost:8790/?net=mainnet
```
`MAINNET_UI_TRADES=1` lets the UI's trade and Rebalance buttons sign with your key from `.env`. Only run this on your laptop, never on Vercel.

**Budget:** the wallet holds about 0.000128 ETH and 1.43 USDC. A swap costs about $0.005 of gas and a rebalance about $0.05. Keep the recording to about 10 trades and 1 rebalance, so there's ETH left for `./scripts/withdraw_mainnet.sh` (about $0.03). Trade **$0.01** at a time: the positions are about $1 each, so bigger trades move the price a lot. Buy before you sell, because the wallet has no WETH until a buy.

**Don't start the keeper on mainnet:** it would spend ETH every 20 seconds. The Keeper panel will say "waiting for keeper", which is fine.

**3. Screen layout** (1920×1080):
- **Right half:** browser at `http://localhost:8790/?net=mainnet`. The header must say **Base mainnet**, and the trade size box shows **0.01**.
- **Left top terminal (watch pane):**
  ```bash
  cd ~/projects/ethglobal-online/iceberg/frontend && set -a && source ../.env && set +a && NETWORK=mainnet npm run watch
  ```
  The UI's own feed is the **On-chain activity** section. Both show the same events, and every row in the UI has a ↗ Basescan link.
- **Left bottom terminal (commands):**
  ```bash
  cd ~/projects/ethglobal-online/iceberg/frontend && set -a && source ../.env && set +a && clear
  ```
- **Extra tabs, ready:**
  - VS Code with four tabs, scrolled to the lines in **Code tour** below: `contracts/periphery/IcebergConfig.sol`, `contracts/iceberg/PAActiveReserves.sol`, `contracts/iceberg/IcebergRouter.sol`, `contracts/v4/IcebergHook.sol`;
  - a browser tab for Basescan (links open there).

**4. Paste-ready commands** (from the commands pane):
```bash
# C1: what the chain says right now: λ, total and reachable reserves on both venues
curl -s "localhost:8790/api/status?net=mainnet" | jq '{network, block, uniswap: {lambda: .uniswapV4.lambdaPct, total: .uniswapV4.reserves, reachable: .uniswapV4.active}, aqua: {lambda: .oneInchAqua.lambdaPct, total: .oneInchAqua.balances, reachable: .oneInchAqua.active}}'
# C2: real 1-cent swap through the Uniswap v4 hook on Base mainnet (prints a basescan.org link)
NETWORK=mainnet npm run swap -- v4 buy 0.01
# C3: real 1-cent fill on the 1inch Aqua position (prints a basescan.org link)
NETWORK=mainnet npm run swap -- aqua buy 0.01
```

## The script

The opening states the problem in proper terms. Everything after it is in plain words: say it the way you'd explain it to a friend who doesn't know crypto.

**Length:** the timings assume a natural pace (about 140 words a minute) and end at **3:22**. The slack covers pauses while you point at things, so a relaxed read still ends by 3:50, under the 4:00 limit.

| Time | Screen | Do | Say (voice-over) |
|---|---|---|---|
| **0:00–0:29** | UI, top of page | Move the mouse over the headline, then the **19.3%** tile | "Automated market makers, or AMMs, like Uniswap hold billions for liquidity providers, about four billion on Uniswap alone. But a pool's price only moves when someone trades. When ETH moves on Binance, the pool is stale for a moment, and arbitrage bots trade against the old price. Researchers call this **loss-versus-rebalancing**, or LVR: a hidden cost on every pool, every block. I built **Iceberg** to shrink it." |
| **0:29–0:50** | UI, **Venues** section | Point at the blue, grey and green bars on the Uniswap card, then the 1inch card | "The idea comes from a 2026 research paper: don't put all your money on the counter. **Blue** can trade right now. **Grey** is locked for this block. **Green** earns interest in Morpho, a savings vault. Only the tip is exposed, like an iceberg, live on Base mainnet on both **Uniswap** and **1inch Aqua**." |
| **0:50–1:06** | VS Code | Four quick shots, see **Code tour** below the table: `program()` → `PAActiveReserves.exec` → `IcebergRouter._runOpcode` → `IcebergHook._getUnspecifiedAmount` | "On 1inch, I wrote two custom **SwapVM instructions**: one freezes the passive part each block, one checks the price against Chainlink. On Uniswap, the same freeze runs in a **v4 hook** that parks idle funds in Morpho." |
| **1:06–1:20** | UI, venue cards → Basescan | Point at **λ** on both venue cards, then in **On-chain activity** click the ↗ on "keeper set λ = …%" to open it on Basescan | "How much stays on the counter? A keeper decides from the last six hours of real prices and publishes it on-chain, here on Basescan. Wild market: lock more. Calm market: open up, to 80%. Never below 40%, so normal traders always find deep liquidity." |
| **1:20–1:32** | Terminal (bottom) | Paste **C1** | **While pressing Enter:** "This is live on Base mainnet. I ask what's on-chain right now…"<br>**After the output:** "…on both venues, only part of the pool can be reached this block." |
| **1:32–1:48** | Terminal → Basescan → UI | Paste **C2**, then click the **basescan.org** link it prints. Back in the UI, point at the new rows in **On-chain activity** | **While pressing Enter:** "Now a real one-cent trade on Uniswap, on mainnet."<br>**After the link opens:** "There it is on Basescan. The website shows it too, with how much was open and how much was locked this block." |
| **1:48–2:04** | Terminal → Basescan | Paste **C3**, click its **basescan.org** link, and scroll to the token transfers | **While pressing Enter:** "Same on **1inch Aqua**."<br>**After the link opens:** "Real token transfers, on-chain: the WETH came out of the Morpho vault only for this trade, and my USDC went straight back in, earning interest." |
| **2:04–2:20** | UI → watch pane → Basescan | In the UI, click **Buy on 1inch official router** (size 0.01), point at the new line in the watch pane, then click its ↗ in the UI feed | **While clicking:** "Now the other way: I click on the website…"<br>**After the watch-pane line:** "…and my terminal shows it, and so does Basescan. That's **Aqua's shared liquidity**: one pile of money backing two strategies, including one on 1inch's own router." |
| **2:20–2:35** | UI → watch pane → Basescan | Click **Rebalance now**, point at the watch pane lines, then click the ↗ on "shipped Aqua strategy" in the UI feed | **While clicking:** "The pool's mix of ETH and dollars slowly drifts."<br>**After the lines appear (cut the wait in editing):** "One click: the keeper retires the strategy and re-ships it at today's price, on-chain." |
| **2:35–3:03** | UI, **Replay** section | 1. Section title. 2. The **plain v4** and **hook λ100%** bars. 3. The two **green** bars. 4. The **cyan Aqua** bar. 5. The command box on the right. | 1. "Does it work? I replayed one real day of ETH prices, minute by minute, with a bot attacking every pool."<br>2. "A normal Uniswap pool lost about a dollar. Iceberg fully open lost the same, so the test is fair."<br>3. "Half open: **17.6%** less loss. At 39%: **19.3%** less."<br>4. "And the 1inch version matched the Uniswap one exactly."<br>5. "Anyone can rerun it with one command." |
| **3:03–3:13** | `test_all` clip, then the README | `test_all` clip, then the README's **Live on Base mainnet** row | "52 contract tests and a full end-to-end check pass. It's deployed on Base mainnet and Sepolia, and every transaction is on the explorer." |
| **3:13–3:22** | UI, top of page | Scroll back to the top and hold on the **19.3%** tile | "Less lost to bots, more earned in Morpho, on both Uniswap and 1inch. Iceberg: only the tip is exposed. Thanks." |

### Code tour

Open each file at these lines before recording, then click through the tabs while you talk. Highlight the lines with your mouse.

| Shot | File and lines | What's on screen | Say during this shot |
|---|---|---|---|
| 1 (4 s) | [IcebergConfig.sol:49-57](../contracts/periphery/IcebergConfig.sol#L49-L57), `program()` | The whole 1inch strategy as a SwapVM program, 5 instructions in order: `Salt` → **`ChainlinkDeviationGuard`** (my opcode **0x21**: refuses to trade if the pool price is more than 1% from Chainlink) → **`PAActiveReserves`** (my opcode **0x92**: the per-block freeze) → `FeeFlatIn` (1inch's stock 5 bps fee) → `XYCSwap` (1inch's stock x·y=k curve) | "On 1inch, I wrote two custom **SwapVM instructions**…" |
| 2 (5 s) | [PAActiveReserves.sol:94-120](../contracts/iceberg/PAActiveReserves.sol#L94-L120), `exec()` | On the block's first fill it splits each balance into λ·R active and the rest passive, stores the split for this block, and hands **only the active part** to the next instruction (`ctx.swap.balanceIn = activeIn`) | "…one freezes the passive part each block, one checks the price against Chainlink." |
| 3 (2 s) | [IcebergRouter.sol:17-21](../contracts/iceberg/IcebergRouter.sol#L17-L21), `_runOpcode()` | 1inch's official Aqua router, unchanged, plus my two opcodes. Any other opcode falls through to 1inch's own code | (no words, just the click) |
| 4 (5 s) | [IcebergHook.sol:191-212](../contracts/v4/IcebergHook.sol#L191-L212), `_getUnspecifiedAmount()` | The v4 hook's pricing: `_refreshSplit()` (same per-block freeze), `xycOut` on the active reserves only, and `_unparkInSwap` pulling from Morpho mid-swap when needed | "On Uniswap, the same freeze runs in a **v4 hook** that parks idle funds in Morpho." |

### How to read the Replay chart

A real day (24 hours ending 21 Sep 2026, ETH $2,635 → $2,800) was replayed on a Base fork. Five pools started with the same money and the same 5 bps fee, and every minute a bot traded each pool to the real price. Each bar is how much that pool's LP lost to the bots. **Lower is better.**

| Bar | Loss | Meaning |
|---|---|---|
| plain v4 (grey) | $1.077 | normal Uniswap v4 pool, the baseline |
| hook λ100% (dark grey) | $1.076 | Iceberg fully active matches plain v4, so the test is fair |
| hook λ50% (green) | $0.888 | 17.6% less loss |
| hook λ39% (green) | $0.869 | 19.3% less loss |
| Aqua λ50% (cyan) | $0.888 | same as hook λ50%: same math on both venues |

## Recording tips

- **Budget:** about 10 trades of $0.01 and 1 rebalance. Keep ETH for `./scripts/withdraw_mainnet.sh`.
- **Timing:** in the five action scenes (1:20–2:35), say the first half while you press Enter or click, and the second half once the result is on screen. Results take about 2–3 s, and the rebalance 10–20 s. Scenes without an action have nothing to wait for, so just talk over them.
- **Easiest option:** record the screen silently first, then add the voice-over while watching the recording, so every "after" line lands on its result.
- **Numbers on screen are small:** the mainnet positions are about $1 each, so reserves change only slightly. The point on mainnet is that every step is a real transaction on Basescan. Read λ off the screen, and keep the replay percentages as scripted.
- **If a mainnet command fails:** check the wallet still has ETH (`cast balance 0x3AaAe578f1F6bBE9705363DE4354d64d6a09C8B7 --ether --rpc-url https://mainnet.base.org`). As a fallback, record the same scenes on the local fork: `SIM_ARB=0 ./scripts/start_local.sh`, UI at `localhost:8788`, and the fork commands `curl -s -X POST "localhost:8788/api/swap?venue=v4&side=buy&usd=10" | jq -r '.message, .tx'` (same for `venue=aqua`).
- **Editing:** record the UI/terminal blocks as separate takes and cut them together. Splice in the code flash and proof clips afterwards.
- **Fallback:** `cd frontend && PAUSE=1 npx tsx scripts/demo.ts` runs all 10 demo steps with a pause between each.
