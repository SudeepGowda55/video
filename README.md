# Iceberg: demo video script (4:00)

## Words used in this script

| Word | Meaning | Say it |
|---|---|---|
| **AMM** | Automated Market Maker. A smart contract that holds a pool of two tokens (here ETH and USDC) and trades with anyone directly, pricing by a formula. Uniswap is one, and a 1inch Aqua strategy works the same way. | "A-M-M" |
| **Partially Active AMM** | Only part of the pool can trade in each block. The rest is frozen for that block and earns yield in Morpho. From the paper arXiv 2602.09887 (Ko, 2026). | |
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

| Time | Screen | Do | Say (voice-over) |
|---|---|---|---|
| **0:00–0:20** | UI, top of page | Slowly move the mouse over the headline, then the **19.3%** tile | "Every block, arbitrage bots take money from liquidity providers. A 2026 research paper, Partially Active AMMs, proposes letting only a fraction lambda of a pool trade in each block. I built it and called it **Iceberg**: only the tip is exposed, and the rest sits under water earning yield in Morpho." |
| **0:20–0:45** | UI, **Venues** section | Point at the blue / grey / green bars on both cards, then the **1inch official router** card | "One kernel, two venues. A **1inch Aqua** position and a **Uniswap v4 hook**. Blue is what an arbitrageur can reach this block, grey is frozen, green is earning in Morpho. On Aqua, the maker holds only vault shares, so nothing leaves their wallet until a trade fills. The same balance even backs a strategy on 1inch's own official router." |
| **0:45–1:05** | VS Code | Show `PAActiveReserves.exec` for 8 s, then `IcebergHook._getUnspecifiedAmount` for 8 s | "On 1inch it's a new SwapVM instruction on the official Aqua registry: on the first fill of a block it freezes the passive part, and 1inch's own curve trades only what's left. On Uniswap it's a hook on OpenZeppelin's custom-curve base, with the same math. The same trade gets the same price on both." |
| **1:05–1:25** | UI, **Keeper** panel | Point at the sentence, then the green bar in the chart | "Lambda isn't fixed. A keeper replays the last six hours of **real ETH prices** at the pool's fee and picks the lambda that loses least. The paper assumes zero fees. I measured that at high fees partial activity stops helping, so the keeper opens the pool up to its 80% cap." |
| **1:25–1:45** | Terminal (bottom) | Paste **C1** | **While pressing Enter:** "Same trade on both venues, from the API…"<br>**After the output:** "…identical price." (point at the two numbers) |
| **1:45–2:10** | Terminal → UI → watch pane | Paste **C2**. Point at the **UI pop-up**, the feed row, then the watch pane | **While pressing Enter:** "A real swap through the Uniswap v4 pool on a Base mainnet fork."<br>**After the pop-up:** "The UI sees it instantly: that block's split, lambda, and exactly how much was frozen. The terminal feed shows the same event." |
| **2:10–2:30** | Terminal → UI | Paste **C3**. In **On-chain activity**, point at the rows tagged *terminal / API*: "Aqua fill withdrew … WETH from its Morpho vault" and "Aqua proceeds … USDC deposited back into Morpho" | **While pressing Enter:** "Now 1inch Aqua."<br>**After the rows appear:** "The fill withdrew exactly what it needed from the Morpho vault and put the proceeds straight back, in one transaction." |
| **2:30–2:50** | UI → watch pane | Click **Buy on 1inch official router**, then point at the new line in the watch pane | **While clicking:** "And the other direction: I click in the UI…"<br>**After the watch-pane line:** "…and the terminal feed shows it. That's Aqua's shared liquidity, the same Morpho balance filling on 1inch's unmodified router." |
| **2:50–3:10** | UI → watch pane | Click **Rebalance now**, then point at the watch pane lines: *retired → shipped → λ carried* | **While clicking:** "A partially active pool lags the market, so its mix drifts."<br>**After the watch-pane lines (10–20 s; pause or cut the wait):** "The keeper retires the strategy, rebalances through Uniswap and re-ships it balanced at the live price, without paying arbitrageurs to do it." |
| **3:10–3:35** | UI, **Replay** section (nav "Replay") | 1. Section title. 2. The **plain v4** and **hook λ100%** bars (same height). 3. The two **green** bars (hook λ50%, λ39%). 4. The **cyan Aqua λ50%** bar next to hook λ50%. 5. The command box on the right. (Optional: cut 5 s to the recorded forge replay output.) | 1. "This is the proof: a real day of ETH prices, replayed minute by minute on a Base fork, with a bot arbitraging each pool every minute."<br>2. "A normal Uniswap v4 pool loses about a dollar to the bots. Iceberg at 100% loses the same, so the test is fair."<br>3. "Expose only half the pool, and the loss drops **17.6%**. At 39%, it drops **19.3%**."<br>4. "And the 1inch Aqua version loses exactly the same as the Uniswap hook: one kernel, two venues."<br>5. "Anyone can rerun this with one command." |
| **3:35–3:50** | `test_all` clip, then the Sepolia tab | Show "52 tests passed … ALL CHECKS PASSED", then the Etherscan transaction | "52 Solidity tests, including 1inch's own invariant suite, a 25-point end-to-end check, deployed and trading on Sepolia, and the Base mainnet run rehearsed with a real wallet." |
| **3:50–4:00** | UI, **Limitations** section | Scroll to it and hold | "Honest limits: the gain depends on the fee, and fewer active reserves means worse prices for ordinary traders, which is why lambda has a floor. Iceberg: only the tip is exposed. Thanks." |

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
