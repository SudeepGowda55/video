# Iceberg: demo video script (4:00)

## Words used in this script

| Word | Meaning | Say it |
|---|---|---|
| **AMM** | Automated Market Maker. A smart contract that holds a pool of two tokens (here ETH and USDC) and trades with anyone directly, pricing by a formula. Uniswap is one. 1inch itself is an aggregator (it routes trades to the best AMM), but a 1inch Aqua strategy can run an AMM formula, and that's what Iceberg's 1inch side is. | "A-M-M" |
| **Partially Active AMM** | Only part of the pool can trade in each block. The rest is frozen for that block and earns yield in Morpho. From the paper arXiv 2602.09887 (Ko, 2026). | |
| **LVR** | Loss-versus-rebalancing: what an LP loses because arbitrage bots trade against the pool's stale price (Milionis et al., 2022). | "L-V-R" |
| **λ** | Greek letter lambda: the fraction of the pool that can trade this block. "λ is 80%" means 80% can trade. | "LAM-duh" (silent b) |
| **Venue** | A place where traders swap against Iceberg's liquidity. There are two, running the same math. | |
| **Venue 1: 1inch Aqua** | The maker ships a strategy to 1inch Aqua. Takers swap through `IcebergRouter`, where a new SwapVM instruction (`PAActiveReserves`) freezes the passive part each block. | |
| **Venue 2: Uniswap v4** | A v4 pool whose hook (`IcebergHook`) does the same per-block freeze. | |
| **1inch official router** | Not a third Iceberg venue. It's a plain strategy on 1inch's own router, backed by the same Morpho balance as the Aqua position (Aqua shared liquidity). | |
| **Keeper** | A background script. Every 20 s it copies the live ETH price, picks λ, parks or unparks idle funds in Morpho, and rebalances when the mix drifts. | |

## What `./scripts/start_local.sh` does

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

**READY** means you can open the UI. Both venues start with the same 2 WETH and matching USDC at the same price. That's why the two quotes in C1 match exactly.

## Before you hit record (15 min ahead)

**1. Fresh stack**, so both venues start identical:
```bash
cd ~/projects/ethglobal-online/iceberg
./scripts/stop_local.sh; SIM_ARB=0 ./scripts/start_local.sh      # wait for "READY" (~3 min)
```
`SIM_ARB=0` turns off the keeper's simulated arbitrageur. It trades every venue each tick, which pushes your trades down the activity feed and moves the two venues' prices apart before C1.
Any swap or rebalance moves each venue's reserves differently, and after that the two quotes won't match. So restart right before recording, and run C1 before any trade or rebalance.

**2. Record two short proof clips first.** They take minutes to run, so don't do them live:
```bash
./scripts/test_all.sh              # record only the last ~15 lines ending in "ALL CHECKS PASSED."
REPLAY_MINUTES=1440 forge test --match-contract Replay21Sep -vv   # record the 5 result lines
```
`test_all.sh` restarts the stack, so run step 1 again after it.

**3. Screen layout** (1920×1080):
- **Right half:** browser at `http://localhost:8788/`, zoomed to 90%, trade size box set to **10**.
- **Left top terminal (live feed):** `cd ~/projects/ethglobal-online/iceberg/frontend && npm run watch`
  - This is the **"watch pane"** in the script. It's a terminal, not part of the UI. The UI's own feed is the **On-chain activity** section (nav link "Activity"), and both show the same events.
- **Left bottom terminal (commands):** `cd ~/projects/ethglobal-online/iceberg && clear`
- **Extra tabs, ready:**
  - VS Code at `contracts/iceberg/PAActiveReserves.sol` (the `exec` function) and `contracts/v4/IcebergHook.sol` (`_getUnspecifiedAmount`);
  - the Sepolia Aqua fill: https://sepolia.etherscan.io/tx/0x321ee1482759a2393852a0b3b156bc6e65e3f897203297fe440baf296c4223b1

**4. Paste-ready commands:**
```bash
# C1: same price on both venues
for v in v4 aqua; do curl -s "localhost:8788/api/quote?venue=$v&side=buy&usd=10" | jq -r '"\(.venue): \(.amountOut) WETH"'; done
# C2: real swap through the Uniswap v4 pool
curl -s -X POST "localhost:8788/api/swap?venue=v4&side=buy&usd=10" | jq -r '.message, .tx'
# C3: real fill on the 1inch Aqua position
curl -s -X POST "localhost:8788/api/swap?venue=aqua&side=buy&usd=10" | jq -r '.message, .tx'
```

## The script

The opening states the problem in proper terms. Everything after it is in plain words: say it the way you'd explain it to a friend who doesn't know crypto.

| Time | Screen | Do | Say (voice-over) |
|---|---|---|---|
| **0:00–0:25** | UI, top of page | Move the mouse over the headline, then the **19.3%** tile | "Automated market makers like Uniswap hold billions for liquidity providers. But a pool's price only moves when someone trades against it. Every time ETH moves on Binance or Coinbase, the pool is stale for a moment, and arbitrage bots race to trade against the old price. Researchers call this **loss-versus-rebalancing**, or LVR: a hidden cost on every pool, every block, that can eat much of what LPs earn in fees. I built **Iceberg** to shrink it." |
| **0:25–0:50** | UI, **Venues** section | Point at the blue, grey and green bars on the Uniswap card, then the 1inch card | "The idea comes from a 2026 research paper: don't put all your money on the counter. **Blue** is the part that can trade right now. **Grey** is locked for this block. **Green** sits in Morpho, a savings vault, earning interest. Like an iceberg, only the tip is exposed. It works the same on **Uniswap** and on **1inch Aqua**: two of the biggest names in crypto trading." |
| **0:50–1:05** | VS Code | Show `PAActiveReserves.exec` for 6 s, then `IcebergHook._getUnspecifiedAmount` for 6 s | "On 1inch, it's a new instruction for their trading engine. On Uniswap, it's a hook. Same math in both." |
| **1:05–1:25** | UI, **Keeper** panel | Point at the "Why λ = …" sentence | "How much goes on the counter? A small assistant, the keeper, decides every 20 seconds. It looks at how wild prices were over the last six hours. Wild market: lock more. Calm market: put more out, up to 80%. Never below 10%, so normal traders can always trade." |
| **1:25–1:45** | Terminal (bottom) | Paste **C1** | **While pressing Enter:** "Let me show it working. I ask both places for the price of a ten-dollar trade…"<br>**After the output:** "…same answer from both." |
| **1:45–2:10** | Terminal → UI → watch pane | Paste **C2**. Point at the pop-up, then **reserves** and **last split block** on the Uniswap card, then the watch pane | **While pressing Enter:** "Now a real ten-dollar trade on Uniswap."<br>**After the pop-up:** "The website picked it up straight away: how much was open to trade and how much was locked this block. And my terminal shows the same trade." |
| **2:10–2:30** | Terminal → UI | Paste **C3**. In **On-chain activity**, point at the *terminal / API* rows "withdrew … from its Morpho vault" and "deposited back into Morpho" | **While pressing Enter:** "Same thing on 1inch."<br>**After the rows appear:** "Watch the money: it left the savings vault only at the moment of the trade, and what I paid went straight back in. It earns interest until the very second it's needed." |
| **2:30–2:50** | UI → watch pane | Click **Buy on 1inch official router**, then point at the new line in the watch pane | **While clicking:** "Now the other way round. I click a button on the website…"<br>**After the watch-pane line:** "…and my terminal shows it. That's 1inch's own trading engine using the same savings: one pile of money serving two strategies." |
| **2:50–3:10** | UI → watch pane | Click **Rebalance now**, then point at the watch pane lines | **While clicking:** "Because the pool trades less, its mix of ETH and dollars slowly drifts."<br>**After the lines appear (10–20 s; pause or cut the wait):** "One click: the assistant takes the pool down, fixes the mix, and puts it back up at today's price." |
| **3:10–3:35** | UI, **Replay** section | 1. Section title. 2. The **plain v4** and **hook λ100%** bars. 3. The two **green** bars. 4. The **cyan Aqua** bar. 5. The command box on the right. | 1. "Does it actually work? I replayed one real day of ETH prices, minute by minute, with a bot attacking every pool."<br>2. "A normal Uniswap pool lost about a dollar. Iceberg with everything on the counter lost the same, so the test is fair."<br>3. "Keep half on the counter: **17.6%** less loss. Keep 39%: **19.3%** less."<br>4. "And the 1inch version lost exactly the same as the Uniswap version."<br>5. "Anyone can rerun this with one command." |
| **3:35–3:50** | `test_all` clip, then the Sepolia tab | Show "ALL CHECKS PASSED", then the Etherscan transaction | "It's tested: 52 smart-contract tests and a full end-to-end check. It's live on the Sepolia test network, and I rehearsed the real Base run with my own wallet." |
| **3:50–4:00** | UI, **Limitations** section | Scroll to it and hold | "The honest trade-off: normal traders also see less money on the counter, so big trades get a slightly worse price. That's why the assistant never locks too much. Iceberg: only the tip is exposed. Thanks." |

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

- **Order matters:** run **C1 before Rebalance now**. A rebalance changes the Aqua reserves, and the two prices would then differ slightly.
- **Timing:** in the five action scenes (1:25–3:10), say the first half while you press Enter or click, and the second half once the result is on screen. Results take about 2–3 s, and the rebalance 10–20 s. Scenes without an action have nothing to wait for, so just talk over them.
- **Easiest option:** record the screen silently first, then add the voice-over while watching the recording, so every "after" line lands on its result.
- **Numbers on screen will differ** from the script (λ, prices, hashes follow the live market). Read them off the screen, and keep the replay percentages as scripted.
- **If a command fails:** stop recording, run `./scripts/stop_local.sh && SIM_ARB=0 ./scripts/start_local.sh`, and retake that scene.
- **Editing:** record the UI/terminal blocks as separate takes and cut them together. Splice in the code flash and proof clips afterwards.
- **Fallback:** `cd frontend && PAUSE=1 npx tsx scripts/demo.ts` runs all 10 demo steps with a pause between each.
