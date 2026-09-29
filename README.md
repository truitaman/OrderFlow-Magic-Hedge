# ⛓️ Wick Magic Hedge
*The Only Bot That Holds Both Sides of the Market — And Knows When to Let Go.*

> **Simultaneous LONG + SHORT on Bitget, Bybit, AsterDex, Binance Futures, Hyperliquid & dYdX v4. Combined PnL control. Live ratio balancing.**  
> *OrderFlow precision. HCM coordination. 9 ready-to-trade presets. One strategy to rule hedge mode.*

---

Hedge Mode is the most powerful — and most dangerous — way to trade perpetual futures. Running a Long and a Short on the same asset simultaneously is not just a different strategy. **It's a fundamentally different coordination problem.** A single-direction bot cannot do it. A grid cannot do it. A standard dual-pair setup that treats each side independently will eat your margin during any sustained trend.

**Wick Magic Hedge** is the only strategy in the Gunbot ecosystem engineered from the ground up for native hedge mode. It understands that two opposing positions form a single capital unit — and it manages them as one, with combined PnL targets, L/S ratio guards, coordinated exits, and a cycle management engine that no other bot provides.

---

## 🆕 What's New in v1.3.2

*Everything below is new since v1.1.7, and included in the current build.*

**⚖️ Smarter hedge management**
- **Max Unhedged** — caps the *absolute* gap between LONG and SHORT, measured in Trading Limits. Block new entries on the larger side, or let the strategy **rebalance** the gap automatically.
- **Trend-Aware Core Hedge** *(opt-in)* — when the LONG is in confirmed drawdown, the hedge can grow beyond parity, and part of the SHORT is protected from the normal profit close while the higher-timeframe trend stays bearish.
- **Target Ratio switch** — turn the L/S ratio guard off and run on the Max Unhedged gap alone.
- **Dynamic Ratio** now follows both the `confluence` and the `emadual` trend engines.

**🌐 More exchanges, more setups**
- **dYdX v4** joins Hyperliquid as a Multi-Instance Hedge exchange.
- **Multi-slot connections** — run several connections of the same exchange in one Gunbot instance, and the SHORT slave is created on the right one automatically.
- **Open Interest Guard** now works on **Hyperliquid, AsterDex and dYdX v4**.
- **Funding Rate Guard** now works on all six hedge exchanges, including **AsterDex, Bitget and Bybit**, and gates **SHORT** entries as well as LONG entries, each leg judged on its own funding direction.

**🎯 Entries**
- **Entry Filter Profile** — choose how much context confirmation an OrderFlow entry needs: `strict`, `balanced` or `rapid`. New preset **`hedge_scalp_aggressive`** ships with `rapid`.
- **SHORT OrderFlow parity** — in OrderFlow mode the SHORT re-enters exactly like the LONG, limited by the S/R zone cap and its own capital.
- **Favorable Scale-In** — add to a winning position on either leg, inside a controlled profit window.
- **Re-Entry Reference** — re-entries can measure their distance from your last partial close, so the strategy does not add straight back into a position it just took profit from.
- **ROC filter Auto-Override** — the momentum filter stands down when the Confluence trend already clearly backs your entry.
- **Self-correcting calibration** — the GA now also recalibrates early when the market runs strongly in one direction.
- **Smarter S/R** — optional 4h-only levels, and a calmer S/R Direction Exit with timeframe filter, minimum hold and imbalance mode.
- **Weekend Mode `weekends-only`** for Institution Hours.

**📈 Exits & protection**
- **Trailing Exit Min Profit (%)** — set the minimum move a trailing exit needs before it may fire.
- **Kill Position Trail Activation** — the trailing kill can be told to arm only after a profit threshold, in USD or ATR mode.
- **Partial close below entry** now works on the SHORT leg too, with the same per-fill logic as the LONG.
- **True Cost per leg** — after a below-entry partial close, each leg tracks the real cost of what is still open, shown in the sidebar and on the chart.

**📟 Sidebar**
- New tiles: **Unhedged**, **Core Hedge**, **Next Partial Close (SHORT)**, **Scale up**, **Funding Interval**, **True Cost** and **True uPnL** per leg, **Open Interest** and **OI Deviation**.

---

## 🏦 HCM — The Feature No Other Bot Has

*When you hold LONG and SHORT simultaneously, neither side's PnL means anything alone.*

A LONG losing $50 while a SHORT gains $80 is a **winning cycle**. Without combined PnL awareness, your bot will close the winning SHORT too early and let the LONG bleed. **HCM — Hedge Capital Management** treats both positions as one trade, always.

### 💹 Combined PnL Target
Set a profit target in **absolute USD** or as a **% of wallet**. When `LONG uPnL + SHORT uPnL ≥ target`, HCM closes **both sides atomically** — LONG via the master, SHORT via a direct call. No lag, no partial exposure, no waiting for individual exits to line up.

### 🛑 Combined Drawdown Stop
If the combined unrealized PnL falls below `-$X`, both sides close immediately. Prevents the classic hedge failure mode: both sides bleeding simultaneously until margin is exhausted.

### ⚖️ L/S Ratio Guard
HCM monitors the real-time ratio of LONG to SHORT notional every tick. Drift outside the tolerance band:
- Over-exposed side → **entries and DCA blocked**
- Under-exposed side → **Trade Size multiplied by Rebalance TL Multiplier** for faster rebalancing

Prefer to run on the absolute gap alone? Turn **Target Ratio Enabled** off and the ratio guard steps aside.

### 🧠 Dynamic Ratio (Trend-Aware)
Enable **Dynamic Ratio** to let the HCM target ratio shift automatically with market bias:
- **Bullish confluence confirmed** → ratio target ×1.3 — LONG side allowed to run heavier
- **Bearish confluence confirmed** → ratio target ×0.7 — LONG side compressed, SHORT given room

Directional conviction changes your hedge exposure automatically, without manual tuning. Works with Institutional Trend Engine = `confluence` (default in all presets) or `emadual`. Other engines keep the base ratio.

### 🧱 Max Unhedged Guard
The L/S Ratio Guard limits the *proportion* between the two sides. **Max Unhedged** limits the *absolute gap*: how many Trading Limits of LONG notional are allowed to sit without a matching SHORT (or the other way round). Off by default.

| Mode | What happens when the gap exceeds the limit |
|---|---|
| `off` | Nothing. No change to your current behavior. |
| `block` | New entries and DCA on the **larger** side are blocked until the gap closes. |
| `rebalance` | Same block, **plus** the strategy automatically opens the smaller side in steps to close the gap. |

- **Max Unhedged (TL)** — the allowed gap, in multiples of your largest configured Trading Limit. Default `5`, and you can type any number.
- **Hold Short or Long Exit if Unhedged** *(on by default)* — a normal exit on either leg is held back if closing it would push the gap above the limit. Emergency closes (liquidation risk, Kill Position, HCM target, drawdown or cycle age) always go through.
- **Max Corrections Per Cycle** — safety cap on how many corrective orders Rebalance can fire in one cycle.
- **Unhedged Correction Cooldown (sec)** — minimum wait between two corrective orders.

**Example:** both Trading Limits are 20 USDT and Max Unhedged is 3 TL, so the allowed gap is about 60 USDT of notional. The LONG has built up to 100 USDT while the SHORT is at 20 USDT. In `block` mode, LONG entries and DCA wait. In `rebalance` mode, the SHORT is topped up step by step, at most once per cooldown, until the gap is back inside the limit.

```text
⚖️ Unhedged    0.99 / 3.0 TL (long light)     ← current gap vs limit, and which side is light
⚖️ Unhedged    3.40 / 3.0 TL ⚠️ CANNOT HEDGE  ← over the limit and a correction was not possible
```

> **Tip:** Max Unhedged `rebalance` and the ratio guard's **Rebalance TL Multiplier** can both push size toward the same gap. Start with one of them, watch how your pair behaves, then combine.

### 🛡️ Trend-Aware Core Hedge
*A hedge that leans in when your LONG is hurting, and keeps the extra protection while the trend agrees.*

Off by default. When it is on and the LONG is in **confirmed drawdown**, the Max Unhedged rebalance engine is allowed to grow the SHORT **beyond parity**, as extra protection for the losing LONG. It reuses the same correction orders, cooldown and cap as Max Unhedged.

- The extra SHORT (the **core**) is **not closed by the normal profit target** while your higher-timeframe trend still confirms bearish. The regular scalp portion of the SHORT keeps closing on profit as usual.
- The moment the trend stops confirming bearish, the protection is released on its own.
- Emergency exits always close the **full** SHORT, core included.

Requires **Max Unhedged Mode = Rebalance** and **Institutional Trend** enabled. Without them it does nothing.

| Setting | Default | Description |
|---|---|---|
| **Trend-Aware Core Hedge** | `off` | Master switch. |
| **Core Hedge Start Drawdown (LONG ROE %)** | `5` | LONG loss at which the SHORT target starts moving beyond parity. |
| **Core Hedge Max Drawdown (LONG ROE %)** | `15` | LONG loss at which the SHORT target reaches its full ceiling. |
| **Core Hedge Ratio Ceiling** | `1.3` | How much larger than the LONG the SHORT may grow at maximum drawdown. `1.0` disables the effect. |

**Example:** the LONG holds 1,000 USDT notional and is -15% ROE. With the default ceiling of 1.3, the SHORT may grow to about 1,300 USDT. The extra part stays open while the trend is bearish, and the scalp portion keeps taking profit.

```text
🛡️ Core Hedge    inactive        ← LONG not in drawdown yet
🛡️ Core Hedge    escalating      ← drawdown threshold crossed, waiting for a correction
🛡️ Core Hedge    0.006000 protected  ← quantity currently held as core
```

> **Tip:** the default ceiling is deliberately conservative. Raise it gradually once you have seen how it behaves on your pair.

### 🕐 Cycle Age Tracker
Once both sides are simultaneously open, a timer starts. Configure a warning threshold (**Max Cycle Hours**) and optionally enable **Force Close on Max Age** to automatically unwind zombie cycles that have been running for days.

> 📖 [Full HCM Documentation →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-16-hcm--hedge-capital-management)

---

## 🏗️ Master / Slave Architecture

The fundamental infrastructure that makes all of this possible.

Gunbot cycles each pair independently. In every other setup, `USDT-BTC-LONG` and `USDT-BTC-SHORT` run separate strategy instances — duplicating indicator computation, holding separate state, and sharing nothing. Wick Magic Hedge is architecturally different:

**The LONG pair is the master.** It runs all indicators, the entire OrderFlow engine, all HCM logic, all trailing state — for both directions. The SHORT pair is a **thin execution shell**: it reads state from the master via `getLedger`, renders its own sidebar and S/R chart lines, and executes SHORT orders — nothing else.

Result:
- One GA calibration serves both LONG and SHORT entries
- S/R levels computed once on the master, displayed on both charts
- SHORT trailing state persists in the LONG store — survives restarts without desync
- Full SHORT pair configuration managed by the master automatically

### 🤖 Auto-Create SHORT Slave
On first tick, the LONG master checks whether the SHORT pair exists in `config.js`. If it doesn't — or if it's misconfigured — the master creates it automatically with the minimum needed: strategy name, exchange, Candle Period, History Length, and Leverage.

On **Hyperliquid and dYdX v4** the SHORT pair is created on your second connection (the Hedge Partner Exchange), and on **multi-slot setups** the strategy resolves the right connection on its own, so a second Aster, Bitget or Bybit slot works the same way.

**Configure one pair. Let the strategy handle the rest.**

> 📖 [Master / Slave Architecture →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-2-master--slave-pair-architecture)

---

## 🌊 The OrderFlow Engine — Adapted for Both Sides

The full OrderFlow engine from Wick Magic Futures is included and adapted for bidirectional execution. One calibration. Two directions. Zero redundancy.

**⚖️ Live Order Book Imbalance** — Normalized -1 to +1 buy/sell pressure. Positive = LONG signal. Negative = SHORT signal. Same score, correct direction applied automatically.

**🧬 Self-Calibrating via Genetic Algorithm** — Evolves buy/sell thresholds against a rolling window of recent real samples. Recalibrates every 4 hours, on an ATR regime shift, and **early whenever the market runs strongly in one direction**, so thresholds never go stale in the middle of a pump or dump. One run per pair — shared between both sides.

**🏗️ Multi-Timeframe S/R Confluence** — Entries only allowed near validated structural levels. Longs blocked below HTF resistance. Shorts blocked above HTF support. Optional 4h-only levels for fresh ranges. Computed once on the master, mirrored to the SHORT chart.

**🧲 Smart S/R Bias** — Gates which *direction* is permitted based on price position relative to nearby zones. Break confirmation waits for N full closes past the zone to filter false breakouts.

**🪤 Trap Detection** — Cross-checks order book shape against real executed flow. If the book shows one direction but the tape contradicts it, the entry is blocked.

**📡 Cross-Exchange Signal Pair** — Read order book and PTH data from a more liquid exchange (e.g. Binance Futures) while executing on any supported hedge exchange.

**🏦 Institution Hours** — Full power during 7–21 UTC. Scout mode (raised conviction bar) or full block outside session hours, with a dedicated Weekend Mode (`scout`, `block`, `full` or `weekends-only`). Applies to both LONG and SHORT entries simultaneously.

**⚡ Entry Filter Profile** — Choose how much context confirmation an OrderFlow entry needs before it fires. `strict` (default) is the most selective. `balanced` relaxes some secondary confirmations. `rapid` relies on direction and the imbalance threshold alone, for maximum entry frequency. Capital limits, cooldowns and DCA caps always apply, whatever the profile. Identical for LONG and SHORT.

**🔁 Same Re-Entry Rules on Both Legs** — In OrderFlow mode the SHORT re-enters exactly like the LONG: bearish imbalance, ATR-based spacing, the same trap guard and the same **Max Re-Entries per S/R Level**, capped by its own capital. **Re-Entry Reference** can measure that spacing from your last partial close instead of the last entry.

**🎯 Trailing Exit Min Profit** — Trailing exits deliberately skip the full profit target, so a sharp reversal is not left waiting for it. By default they only need a small fee-covering minimum. Raise **Trailing Exit Min Profit (%)** to require a bigger move from your entry before a trailing exit may fire.

### 📊 ATR Strength Classifier
Reads real-time volatility regime and adjusts entry thresholds automatically. Dead market? Thresholds raised to avoid low-conviction entries. Extreme volatility (rekt regime)? Thresholds raised to avoid getting caught in whipsaws. Five regimes: Dead / Weak / Average / Strong / Rekt. Recommended ON for hedge; it is your own setting, so switching presets never turns it off. Shared across the whole Wick Magic family (Spot, Futures, Hedge), not unique to hedge mode.

### 📉 ROC Momentum Filter
Pre-entry gate that confirms momentum is alive before allowing an order. If the move is fading — measured over configurable bars — the entry is blocked. Waits for a fresh burst instead of chasing the tail. New entries only. DCA into existing positions is never affected. Recommended ON for hedge; it is your own setting, so switching presets never turns it off. Shared across the whole Wick Magic family (Spot, Futures, Hedge), not unique to hedge mode. Running the `confluence` trend engine? Enable **Auto-Override on Confluence Trend** and the filter stands down whenever the trend already clearly backs your entry direction.

### 🚀 Favorable Scale-In
*DCA adds to a loser to improve your average. Scale-In adds to a winner to ride the trend.*

Off by default. With **Enable Favorable Scale-In** on, either leg can add to a position that is **already in profit**, inside a controlled window:

- Adds fire only while profit is **positive but below the trailing arm point**. Once the trail arms, the window closes and no more adds fire for the rest of that cycle.
- **Scale-In Max Adds** caps the adds per cycle, **per leg** (LONG and SHORT are counted separately). Default `3`, and `0` means unlimited.
- Each add uses one Trading Limit and still has to pass the capital, HCM unhedged, funding and Open Interest guards.
- The LONG scales up at the **Re-Entry Scale Spread (%)** above its last fill, the SHORT at **Scale-In Distance (SHORT %)** below its last fill. Leave either at `0` to disable that leg.
- **Scale Only After Partial Close** (on by default) holds every add back until at least one partial close has happened. Switch it off to pyramid from the first profitable move.
- **Scale-In: Allow While Underwater** (SHORT only, off by default) also allows SHORT adds on a favorable bounce while the SHORT is still net underwater, for example after DCA.

> ⚠️ Every add made in profit worsens your average entry, especially on the SHORT. A sharp reversal after several adds hurts proportionally more. Raise the cap carefully.

> 📖 [OrderFlow Engine →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-3-orderflow-trading-in-hedge-mode) · [ATR Strength →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-11-atr-strength-classifier) · [ROC Filter →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-12-roc-momentum-filter)

---

## 🛡️ Safety — Bidirectional on Every Layer

Every safety system fires on **both sides simultaneously**. This is not trivial: most checks require per-side logic that understands which position is at risk at any moment.

**🔪 Kill Position — Bidirectional USD & ATR Stop**
Monitors uPnL every tick, independently of HCM and all other logic. Two modes:
- **USD Mode** — Fixed dollar hard stop, trailing stop from peak uPnL, flat take profit
- **ATR Mode** — All thresholds auto-scale as `multiplier × ATR × position_qty`. No manual tuning per pair.

The trailing kill can also be told to **arm only after a profit threshold** (**KP Trail Activation**, in USD or as an ATR multiplier). It is never lower than the trail distance (the strategy raises it automatically), so the trail can never produce a loss exit.

When Kill Position fires: LONG exits via master, SHORT exits via slave adapter. **Both sides close in one triggered event.**

**🚨 Circuit Breaker** — Accumulates daily realized PnL. If losses exceed a configured % of capital, all new entries (LONG and SHORT) are blocked until UTC midnight. Exits always continue.

**🛡️ Liquidation Guard** — Monitors both liquidation prices in real time from the exchange WebSocket. When either side gets too close, both positions close — because a liquidation on one side destroys the entire hedge balance.

**💸 Funding Rate Guard — Side-Aware**
In hedge mode, positive funding means longs pay and shorts *receive*. A naive guard that blocks everything when funding is high is wrong. Side-Aware mode:
- **Positive funding** → only new LONG entries blocked (they pay). SHORT entries never blocked (they receive).
- **Negative funding** → only new SHORT entries blocked. LONG entries free.

The receiving side is never penalized. **Strongly recommended ON for all hedge setups.** The guard is available on all six hedge exchanges: Bitget, Bybit, AsterDex, Binance Futures, Hyperliquid and dYdX v4. On AsterDex, Bitget and Bybit the funding interval differs per symbol and is read live. Each leg is judged on its own side: the LONG is held back when longs pay, the SHORT when shorts pay, and Max Unhedged corrective orders are never blocked.

**📈 Open Interest Guard**
Two independent gates, both market-wide (not your own position):
- **Absolute OI Guard** — blocks new entries and DCA when the coin's total Open Interest notional exceeds your configured ceiling. A proxy for an over-leveraged, squeeze-prone market.
- **OI Delta Guard** — blocks new entries and DCA when Open Interest has moved sharply, up or down, within a trailing window. A sudden surge or collapse often precedes cascading liquidations.

Available on **Hyperliquid, AsterDex and dYdX v4**. In hedge mode the same coin-wide feed serves both legs, and an optional **New Entries Only** switch keeps DCA on an open position alive.

> 📖 [Kill Position →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-13-kill-position--bidirectional-hedge-edition) · [Liquidation Guard →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-14-liquidation-guard--bidirectional-protection) · [Funding Rate →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-15-funding-rate-intelligence--side-aware-)

---

## 🏛️ Institutional Trend — Direction Intelligence

Five independent macro bias engines for **AUTO** direction. The strategy decides each tick: Long or Short?

| Engine | Logic | Best for |
|---|---|---|
| `confluence` *(default)* | Multi-confirmation engine. Requires broad agreement before flipping. Sticky direction in no-man's-land zones. | **Hedge mode — recommended for all presets** |
| `emadual` | EMA33 daily + EMA200 long must agree. SuperTrend as tiebreaker on conflict. | Fast-moving pairs |
| `sma200` | Price vs 200-period SMA. Classic macro filter. | Trending markets |
| `supertrend` | SuperTrend direction, configurable period and multiplier. | Momentum markets |
| `smacross` | SMA100 vs SMA200 golden/death cross. Fewer flips than price-based. | Ranging pairs |

`confluence` is the default in all presets because a direction flip in hedge mode means stopping entries on one side and starting on the other — if that happens every 5 minutes, the strategy never builds a real position. Confluence requires multiple confirmations before flipping, giving each side time to develop.

> 📖 [Institutional Trend Mode →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-18-institutional-trend-mode)

---

## 📦 9 Ready-to-Trade Presets

One dropdown. Full configuration applied instantly. Your personal settings are always preserved: capital, trade sizes, leverage, HCM, your trend engine, Kill Position, the Funding Rate Guard, and your own filters such as ATR Strength and ROC.

### Hedge Profiles

| | `hedge_scalp` | `hedge_scalp_aggressive` | `hedge_scalp_fast` | `hedge_institutional` | `hedge_inst_fast` |
|---|---|---|---|---|---|
| **Candle** | 5m | 5m | 5m | 15m | 15m |
| **Context** | 4h | 4h | 4h | 4h | 4h |
| **Target ROE** | 1.2% | 1.2% | 0.5% | 1.8% | 1.4% |
| **Min Profit** | 0.9% | 0.9% | 0.4% | 1.2% | 0.8% |
| **Trailing** | 0.5% | 0.5% | 0.4% | 0.75% | 0.5% |
| **Re-Entries / S/R Level** | 2 | 2 | 3 | 2 | 2 |
| **Entry Filter** | strict | rapid | strict | strict | strict |
| **Session Gate** | OFF (24/7) | OFF (24/7) | OFF (24/7) | ON, scout | ON, scout |
| **Weekend Mode** | full | full | full | weekends-only | weekends-only |
| **Confluence Min Score** | 5 | 5 | 5 | 6 | 6 |
| **Partial Close** | OFF | OFF | ON | ON | ON |

> **`hedge_scalp_aggressive`** is `hedge_scalp` with the Entry Filter Profile set to `rapid`, for maximum entry frequency. In `rapid` mode the context filters (session hours, ATR Strength, ROC, absorption and S/R proximity) are bypassed on new entries. Position size, capital, leverage and cooldown are unchanged.
>
> **ATR Strength** and **ROC Filter** are your own settings, kept across preset switches. Both are recommended ON for hedge mode.

### HMB Variants — Dynamic Ratio Active

`hedge_scalp_hmb` and `hedge_scalp_fast_hmb` — identical to Scalp and Scalp Fast, with **Dynamic Ratio (Trend-Aware) ON**. HCM target ratio shifts automatically with market bias. Best used once you have HCM targets configured and confluence scores stabilized on your pair.

### Single-Direction Profiles — For Non-Hedge Exchanges

`orderflow_scalp_fast` (5m / 1h context, SuperTrend direction) and `orderflow_institutional_fast` (15m / 1h context, SMA200 direction) — full OrderFlow engine without HCM and SHORT management. Use on Kraken Futures, or anywhere you prefer one position at a time.

> 📖 [Full Preset Comparison →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-22-hedge-presets)

---

## 📟 The Hedge Sidebar — Both Positions, One View

The master sidebar shows both legs of the hedge at once — position, ROE, uPnL, liquidation price, and wallet balance for LONG and SHORT side by side — plus every guard, filter, and score driving the strategy's decisions, live on every tick. Here are two v1.3.2 examples. Prices come from live pairs on 19 September 2026, and position and PnL figures are illustrative.

### Hyperliquid — Multi-Instance Hedge

USDC-HYPE at **92.09**, LONG on one wallet (3.52 HYPE @ 92.35) and SHORT on the second (1.76 HYPE @ 91.85), sitting right on the 2.00 target ratio.

```text
🌊 WM Hedge v1.3.2                LONG
📊 Market Type                    🐂 BULLISH
ROC (annualized)                  159.03%
────────────────────────────────────────
── 🟢 LONG ──                     3.5200 @ 92.3500
💰 LONG ROE                       -1.408%
📊 LONG uPnL                      -0.92 USDC
💥 LONG Liq                       n/a
💸 Funding                        0.0013%
🏦 Wallet Bal (LONG)              4895.27 USDC
── 🔴 SHORT ──                    1.7600 @ 91.8500
💰 SHORT ROE                      -1.306%
📊 SHORT uPnL                     -0.42 USDC
💥 SHORT Liq                      131.40
💸 Funding                        0.0013%
💼 Used Capital (SHORT)           32.42 USDC
🌊 Next Partial Close OF (SHORT)  Full Close (<BE)
🏦 Wallet Bal (SHORT)             493.50 USDC
────────────────────────────────────────
── 🏦 HCM ──
💹 Combined uPnL                  -1.34 USD
⚖️ L/S Ratio                       2.00 (target 2.00)
⚖️ Unhedged                        0.74 / 3.0 TL (short light)
────────────────────────────────────────
🌊 Signal (LONG)                  🟢 BULLISH
⚖️ Imbalance                       0.3124
🟢 Buy threshold                  0.2
🔴 Sell threshold                 -0.201
🧬 GA samples                     700 / 120 ✓
🧊 Iceberg bid                    NO
🧊 Iceberg ask                    NO
🪤 Trap guard                     CLEAR
🔬 PTH trades                     1000 (0s ago)
🔄 Rebuy enabled                  YES
🔒 Above guard                    WAITING PARTIAL
📉 Next DCA (drop)                target ~91.426500
📈 Scale up                       OFF
🏗️ S/R gate                        ON (±1%)
🏗️ S/R zone                        0/2
🔄 S/R DirExit                    OFF
🐋 Absorption                     ON
📉 ROC Filter                     ✅ Active | p1 b3 ≥0.5%
────────────────────────────────────────
Trading Limit                     220.00 USDC
🕯️ Heikin Color                    green
🏦 Total Capital                  2000.00 USDC
💼 Used Capital (LONG)            64.83 USDC
📊 Capital Used (%)               3.24%
🤖 Smart Capital                  OFF
📈 Realized P&L                   268.88 USDC
📉 Unrealized P&L                 -0.92 USDC
📊 PnL (ROE %)                    -1.41%
🪢 Rekt Liq. Price                n/a
🛡️ Liq. Limit                      n/a
🔪 Kill Position                  OFF
🚨 Circuit Breaker                OFF
⚠️ Margin Usage                    2.07%
🕒 Days Trading                   59 days
💰 Position margin                64.83 USDC
💵 Margin available               4603.40 USDC
📦 Position size                  3.52 HYPE
📉 Current Volume                 4,672.16
📊 Avg Volume                     9,399.41
🕒 Buy Cooloff                    CLEAR
📦 Total Volume                   55570.52 USDC
────────────────────────────────────────
⚙️ Mode                            💰 CLOSE
📐 Profit Mode                    Price %
🎯 Target Exit                    93.4582
🎗️ Dyn Trailing                    0.31%
🔻 Trailing Stop                  n/a
────────────────────────────────────────
🏛️ Inst. Trend                     LONG
🏛️ Bull Score                      7.5
🏛️ Bear Score                      1.5
🏛️ Grade                           A
🏛️ HTF Bias                        BULL
────────────────────────────────────────
🌊 Partial SELL                   READY
🌊 Next Partial Close OF          93.510000 (+1.54%)
────────────────────────────────────────
📌 Mark Price                     92.11
💸 Funding Rate                   0.001250%/h
📊 Cum. Funding                   +0.0245 USDC
🛡️ Funding Guard                   ✅ OK (11.0% PAY)
⏰ Next Funding                   51m 59s
📈 Open Interest                  $1,988,947,888
📉 OI Deviation 15m               -0.0%
────────────────────────────────────────
Order Type                        💸 Market
```

### AsterDex — Native Hedge (trimmed)

USDT-ETH at **2,649.83**, LONG 0.01 ETH @ 2,644.89 and SHORT 0.005 ETH @ 2,641.20 on the same account. AsterDex adds its own funding interval and Open Interest tiles. Some tiles are trimmed for length.

```text
🌊 WM Hedge v1.3.2                LONG
📊 Market Type                    🐂 BULLISH
ROC (annualized)                  79.38%
────────────────────────────────────────
── 🟢 LONG ──                     0.0100 @ 2644.8900
💰 LONG ROE                       0.932%
📊 LONG uPnL                      0.05 USDT
💥 LONG Liq                       2,118.35
💸 Funding                        0.0100%
🏦 Wallet Bal                     329.01 USDT
── 🔴 SHORT ──                    0.0050 @ 2641.2000
💰 SHORT ROE                      -1.634%
📊 SHORT uPnL                     -0.04 USDT
💥 SHORT Liq                      3,149.60
💸 Funding                        0.0100%
💼 Used Capital (SHORT)           2.65 USDT
⚙️ Next Partial Close (SHORT)      Full Close (<BE)
────────────────────────────────────────
── 🏦 HCM ──
💹 Combined uPnL                  +0.01 USD
⚖️ L/S Ratio                       2.00 (target 2.00)
────────────────────────────────────────
🌊 Signal (LONG)                  🟢 BULLISH
⚖️ Imbalance                       0.3186
🟢 Buy threshold                  0.2
🔴 Sell threshold                 -0.2012
🧬 GA samples                     700 / 120 ✓
🪤 Trap guard                     CLEAR
🔬 PTH trades                     500 (32s ago)
🔒 Above guard                    WAITING PARTIAL
📉 Next DCA (drop)                target ~2604.431788
📈 Scale up                       OFF
🏗️ S/R gate                        ON (±1%)
🏗️ S/R zone                        0/2
🐋 Absorption                     ON
💰 Compound                       +0.00 → 18.00
📊 ATR Strength                   🟢 AVERAGE ×1.00
📉 ROC Filter                     ✅ Active | p1 b3 ≥0.2%
────────────────────────────────────────
Trading Limit                     18.00 USDT
🏦 Total Capital                  300.00 USDT
💼 Used Capital (LONG)            5.29 USDT
📊 Capital Used (%)               1.76%
📈 Realized P&L                   11.91 USDT
🪢 Rekt Liq. Price                2118.35 (25.1% away)
🛡️ Liq. Limit                      25.1% / 20% → 2542.02
⚠️ Margin Usage                    3.10%
📦 Position size                  0.01 ETH
────────────────────────────────────────
🎯 Target Exit                    2,676.6287
🎗️ Dyn Trailing                    0.30%
🔻 Trailing Stop                  n/a
────────────────────────────────────────
🏛️ Inst. Trend                     LONG
🏛️ Bull Score                      9
🏛️ Bear Score                      0
🏛️ Grade                           A+
🏛️ HTF Bias                        BULL
────────────────────────────────────────
📌 Mark Price                     2,650.25
💸 Funding Rate                   0.010000%
⏱️ Funding Interval                8h
📊 Cum. Funding                   +0.0000 USDT
🛡️ Funding Guard                   ✅ OK (11.0% PAY)
⏰ Next Funding                   413m 45s
📈 Open Interest                  $246,802,086
📉 OI Deviation 15m               -0.0%
```

After a below-entry partial close, each leg also shows **🧾 True Cost** and **💵 True uPnL** tiles, and a dashed **🧾 TCB** line appears on its chart.

The SHORT slave shows its own sidebar with position, ROE, uPnL, trail stop, target exit price, liquidation price, and funding rate — live on every tick.

> 📖 [Sidebar Reference →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-24-the-hedge-sidebar--dual-position-view)

---

## 🌐 Supported Exchanges

| Exchange | Status | Hedge Mode | Notes |
|---|---|---|---|
| **Bitget** | ✅ Final | ✅ Native Hedge | USDT-M & COIN-M futures, live & testnet — LONG and SHORT on one connection |
| **Bybit** | ✅ Final | ✅ Native Hedge | USDT perpetuals, demo & live — LONG and SHORT on one connection |
| **Aster Futures** | ✅ Final | ✅ Native Hedge | USD perpetuals, live — LONG and SHORT on one connection |
| **Binance Futures** | ✅ Final | ✅ Cross-Quote Hedge | Two pairs, one shared Multi-Assets margin pool — see setup below |
| **Hyperliquid** | ✅ Final | ✅ Multi-Instance Hedge | Two wallets / connections — see setup below |
| **dYdX v4** | ✅ Final | ✅ Multi-Instance Hedge | Two wallets / connections, same mechanism as Hyperliquid. All orders are placed as limit-taker, because the exchange has no market orders |


Hedge Mode now runs on six exchanges, using three different mechanisms depending on what each exchange allows. The next section walks through all three.

---

## 🔀 Hedge Topologies & Multi-Pair Setup

Not every exchange lets you hold LONG and SHORT the same way. Bitget and Bybit hold both positions on one connection natively. Binance Futures, Hyperliquid and dYdX v4 need a second pair (or a second wallet) to represent the opposing side — and each does it differently.

| Topology | Exchanges | Pair naming |
|---|---|---|
| **Native Hedge** | Bitget, Bybit | Single pair — the exchange holds both sides internally |
| **Cross-Quote Hedge** | Binance Futures | Two pairs, distinguished by quote asset |
| **Multi-Instance Hedge** | Hyperliquid, dYdX v4 | Two wallets, same pair name on both |

> 📖 [Full Setup Guide — Binance Cross-Quote & Multi-Instance (Hyperliquid, dYdX v4) →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-27-multi-exchange-hedge-setups--binance-futures-hyperliquid--dydx-v4)

---

## 📖 Full Documentation

- [§1 — What is Hedge Mode?](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-1-what-is-hedge-mode)
- [§2 — Master / Slave Architecture](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-2-master--slave-pair-architecture) 
- [§3 — OrderFlow in Hedge Mode](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-3-orderflow-trading-in-hedge-mode) 
- [§4 — The Machine Learning Engine (GA)](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-4-the-machine-learning-engine) 
- [§5 — Support & Resistance Confluence](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-5-support--resistance-confluence-hedge-adapted) 
- [§6 — Smart S/R Bias](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-6-smart-sr-bias-direction-gate)
- [§7 — Trap Detection & Absorption](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-7-bull--bear-trap-detection)
- [§9 — Institution Hours](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-9-institution-hours)
- [§10 — Cross-Exchange Signal Pair](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-10-cross-exchange-signal-pair)
- [§11 — ATR Strength Classifier](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-11-atr-strength-classifier) 
- [§12 — ROC Momentum Filter](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-12-roc-momentum-filter) 
- [§13 — Kill Position — Bidirectional](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-13-kill-position--bidirectional-hedge-edition)
- [§14 — Liquidation Guard](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-14-liquidation-guard--bidirectional-protection)
- [§15 — Funding Rate Intelligence — Side-Aware](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-15-funding-rate-intelligence--side-aware-)
- [§16 — HCM — Hedge Capital Management](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-16-hcm--hedge-capital-management)
- [§17 — SHORT Position Management](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-17-short-position-management)
- [§18 — Institutional Trend Mode](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-18-institutional-trend-mode)
- [§19 — Margin & Compound Profits](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-19-margin--compound-profits)
- [§22 — Hedge Presets](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-22-hedge-presets)
- [§23 — Full Settings Reference](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-23-editor-settings-reference)
- [§25 — OrderFlow vs Classic Branch](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-25-orderflow-vs-classic-branch-hedge)
- [§26 — Tips & Best Practices](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-26-tips--best-practices-for-hedge-mode)
- [§27 — Multi-Exchange Hedge Setups (Binance Futures, Hyperliquid & dYdX v4)](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-27-multi-exchange-hedge-setups--binance-futures-hyperliquid--dydx-v4)
- [§28 — Open Interest Guard (Hyperliquid, AsterDex & dYdX v4)](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-28-open-interest-guard-hyperliquid-asterdex--dydx-v4)
- [§29 — Safe Config Example](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-29-safe-config-example)
- [§30 — Version History](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-30-version-history)
- [§31 — Entry Filter Profile](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-31-entry-filter-profile--configuring-entry-aggressiveness)

---

## 🧭 Recommended Setup & Best Practices

A handful of habits separate a hedge that runs itself from one that needs constant babysitting: symmetric wallets on Hyperliquid and dYdX v4, conservative leverage (2–5x swing, 5–10x scalp), understanding the `HEDGE_RATIO` band before touching it, switching on Max Unhedged in `block` mode before you try `rebalance`, and running one exchange family per Gunbot instance.

> 📖 [Full Tips & Best Practices →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-26-tips--best-practices-for-hedge-mode)

---

## 🧾 Safe Config Example

A complete, ready-to-run `config.js` for a Hyperliquid multi-instance hedge — both connections, both pairs, all strategy settings — is available as a starting template. Download it, rename to `config.js`, fill in your own wallet and API details, and start Gunbot.

> 📖 [Full Safe Config Guide & Download →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-29-safe-config-example)

---

## 🆕 Version History — v1.3.2

- **dYdX v4** as a full Multi-Instance Hedge exchange, with limit-taker order handling and live Open Interest
- **Max Unhedged** (Block and Rebalance) with exit hold, corrections cap and cooldown
- **Trend-Aware Core Hedge**, an opt-in extra SHORT that stays protected while the trend is bearish
- **SHORT OrderFlow parity** — the SHORT re-enters like the LONG in OrderFlow mode
- **Favorable Scale-In** on both legs, with a profit window, a per-leg cap and an underwater option for the SHORT
- **Re-Entry Reference** (`auto` / `last_entry`)
- **Entry Filter Profile** (`strict`, `balanced`, `rapid`) and the new `hedge_scalp_aggressive` preset
- **Open Interest Guard** on AsterDex and dYdX v4, on top of Hyperliquid
- **Funding Rate Guard** on all six hedge exchanges and on the SHORT leg, with the per-symbol funding interval read live
- **Dynamic Ratio** now follows `confluence` and `emadual`, plus a Target Ratio on/off switch
- **ROC filter Auto-Override** on the Confluence trend, and **early GA recalibration** in one-directional markets
- **Trailing Exit Min Profit (%)** and **Kill Position Trail Activation**
- **SHORT leg:** partial close below entry, a Next Partial Close tile, and Stop After Full Close support
- **True Cost per leg**, with True Cost / True uPnL tiles and TCB chart lines
- **Multi-slot exchange connections** for Aster, Hyperliquid, dYdX v4 and more, with automatic SHORT slave creation
- **S/R and session upgrades:** 4h-only levels, Direction Exit timeframe, minimum hold and imbalance mode, Weekend Mode `weekends-only`

For the full version-by-version changelog, check [Section 30 of the wiki](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#-30-version-history) or the [Telegram community](https://t.me/+xQtQ9Y4AOc9lZTNk).

---

## 🛍️ Get the Strategy

Requires a valid **Gunbot license** (Defi or Unlimited).

> 🛒 **Get Wick Magic Hedge** : https://checkout.gunbot.com/crazymop/wmhedge

👉 **Join the Telegram community:**  
https://t.me/+xQtQ9Y4AOc9lZTNk

No Gunbot license yet?  
→ [Get Gunbot Ultimate](gunbot.com/es)
