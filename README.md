# ⛓️ Wick Magic Hedge
### *The Only Bot That Holds Both Sides of the Market — And Knows When to Let Go.*

> **Simultaneous LONG + SHORT on Bitget & Bybit. Combined PnL control. Live ratio balancing.**  
> *OrderFlow precision. HCM coordination. 8 ready-to-trade presets. One strategy to rule hedge mode.*

---

Hedge Mode is the most powerful — and most dangerous — way to trade perpetual futures. Running a Long and a Short on the same asset simultaneously is not just a different strategy. **It's a fundamentally different coordination problem.** A single-direction bot cannot do it. A grid cannot do it. A standard dual-pair setup that treats each side independently will eat your margin during any sustained trend.

**Wick Magic Hedge** is the only strategy in the Gunbot ecosystem engineered from the ground up for native hedge mode. It understands that two opposing positions form a single capital unit — and it manages them as one, with combined PnL targets, L/S ratio guards, coordinated exits, and a cycle management engine that no other bot provides.

---

## 🏦 HCM — The Feature No Other Bot Has

*When you hold LONG and SHORT simultaneously, neither side's PnL means anything alone.*

A LONG losing $50 while a SHORT gains $80 is a **winning cycle**. Without combined PnL awareness, your bot will close the winning SHORT too early and let the LONG bleed. **HCM — Hedge Capital Management** treats both positions as one trade, always.

### 💹 Combined PnL Target
Set a profit target in **absolute USD** or as a **% of wallet**. When `LONG uPnL + SHORT uPnL ≥ target`, HCM closes **both sides atomically** — LONG via the master, SHORT via a direct adapter call. No lag, no partial exposure, no waiting for individual exits to line up.

### 🛑 Combined Drawdown Stop
If the combined unrealized PnL falls below `-$X`, both sides close immediately. Prevents the classic hedge failure mode: both sides bleeding simultaneously until margin is exhausted.

### ⚖️ L/S Ratio Guard
HCM monitors the real-time ratio of LONG to SHORT notional every tick. Drift outside the tolerance band:
- Over-exposed side → **entries and DCA blocked**
- Under-exposed side → **Trade Size multiplied by Rebalance TL Multiplier** for faster rebalancing

### 🧠 Dynamic Ratio (Confluence)
Enable **Dynamic Ratio** to let the HCM target ratio shift automatically with market bias:
- **Bullish confluence confirmed** → ratio target ×1.3 — LONG side allowed to run heavier
- **Bearish confluence confirmed** → ratio target ×0.7 — LONG side compressed, SHORT given room

Directional conviction changes your hedge exposure automatically, without manual tuning. Requires Institutional Trend Engine = `confluence` (default in all presets).

### 🕐 Cycle Age Tracker
Once both sides are simultaneously open, a timer starts. Configure a warning threshold (**Max Cycle Hours**) and optionally enable **Force Close on Max Age** to automatically unwind zombie cycles that have been running for days.

> 📖 [Full HCM Documentation →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#16-hcm-hedge-capital-management)

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

**Configure one pair. Let the strategy handle the rest.**

> 📖 [Master / Slave Architecture →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#2-master-slave-pair-architecture)

---

## 🌊 The OrderFlow Engine — Adapted for Both Sides

The full OrderFlow engine from Wick Magic Futures is included and adapted for bidirectional execution. One calibration. Two directions. Zero redundancy.

**⚖️ Live Order Book Imbalance** — Normalized -1 to +1 buy/sell pressure. Positive = LONG signal. Negative = SHORT signal. Same score, correct direction applied automatically.

**🧬 Self-Calibrating via Genetic Algorithm** — Evolves buy/sell thresholds against up to 2000 real candles. 60 candidates × 220 generations. Recalibrates every 4 hours or on ATR regime shift. One run per pair — shared between both sides.

**🏗️ Multi-Timeframe S/R Confluence** — Entries only allowed near validated structural levels. Longs blocked below HTF resistance. Shorts blocked above HTF support. Computed once on the master, mirrored to the SHORT chart.

**🧲 Smart S/R Bias** — Gates which *direction* is permitted based on price position relative to nearby zones. Break confirmation waits for N full closes past the zone to filter false breakouts.

**🪤 Trap Detection** — Cross-checks order book shape against real executed flow. If the book shows one direction but the tape contradicts it, the entry is blocked.

**📡 Cross-Exchange Signal Pair** — Read order book and PTH data from a more liquid exchange (e.g. Binance Futures) while executing on Bitget or Bybit.

**🏦 Institution Hours** — Full power during 7–21 UTC. Scout mode (raised conviction bar) or full block outside session hours. Applies to both LONG and SHORT entries simultaneously.

### 📊 ATR Strength Classifier *(Hedge Exclusive)*
Reads real-time volatility regime and adjusts entry thresholds automatically. Dead market? Thresholds raised to avoid low-conviction entries. Extreme volatility (rekt regime)? Thresholds raised to avoid getting caught in whipsaws. Five regimes: Dead / Weak / Average / Strong / Rekt. Active in all presets.

### 📉 ROC Momentum Filter *(Hedge Exclusive)*
Pre-entry gate that confirms momentum is alive before allowing an order. If the move is fading — measured over configurable bars — the entry is blocked. Waits for a fresh burst instead of chasing the tail. New entries only. DCA into existing positions is never affected. Active in all presets.

> 📖 [OrderFlow Engine →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#3-orderflow-trading-in-hedge-mode) · [ATR Strength →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#11-atr-strength-classifier-hedge-exclusive) · [ROC Filter →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#12-roc-momentum-filter-hedge-exclusive)

---

## 🛡️ Safety — Bidirectional on Every Layer

Every safety system fires on **both sides simultaneously**. This is not trivial: most checks require per-side logic that understands which position is at risk at any moment.

**🔪 Kill Position — Bidirectional USD & ATR Stop**
Monitors uPnL every tick, independently of HCM and all other logic. Two modes:
- **USD Mode** — Fixed dollar hard stop, trailing stop from peak uPnL, flat take profit
- **ATR Mode** — All thresholds auto-scale as `multiplier × ATR × position_qty`. No manual tuning per pair.

When Kill Position fires: LONG exits via master, SHORT exits via slave adapter. **Both sides close in one triggered event.**

**🚨 Circuit Breaker** — Accumulates daily realized PnL. If losses exceed a configured % of capital, all new entries (LONG and SHORT) are blocked until UTC midnight. Exits always continue.

**🛡️ Liquidation Guard** — Monitors both liquidation prices in real time from the exchange WebSocket. When either side gets too close, both positions close — because a liquidation on one side destroys the entire hedge balance.

**💸 Funding Rate Guard — Side-Aware**
In hedge mode, positive funding means longs pay and shorts *receive*. A naive guard that blocks everything when funding is high is wrong. Side-Aware mode:
- **Positive funding** → only new LONG entries blocked (they pay). SHORT entries never blocked (they receive).
- **Negative funding** → only new SHORT entries blocked. LONG entries free.

The receiving side is never penalized. **Strongly recommended ON for all hedge setups.**

> 📖 [Kill Position →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#13-kill-position-bidirectional-hedge-edition) · [Liquidation Guard →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#14-liquidation-guard-bidirectional-protection) · [Funding Rate →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#15-funding-rate-intelligence-side-aware)

---

## 🏛️ Institutional Trend — Direction Intelligence

Five independent macro bias engines for **AUTO** direction. The strategy decides each tick: Long or Short?

| Engine | Logic | Best for |
|---|---|---|
| `confluence` *(default)* | Multi-indicator weighted score. Requires broad agreement before flipping. Sticky direction in no-man's-land zones. | **Hedge mode — recommended for all presets** |
| `emadual` | EMA33 daily + EMA200 long must agree. SuperTrend as tiebreaker on conflict. | Fast-moving pairs |
| `sma200` | Price vs 200-period SMA. Classic macro filter. | Trending markets |
| `supertrend` | SuperTrend direction, configurable period and multiplier. | Momentum markets |
| `smacross` | SMA100 vs SMA200 golden/death cross. Fewer flips than price-based. | Ranging pairs |

`confluence` is the default in all presets because a direction flip in hedge mode means stopping entries on one side and starting on the other — if that happens every 5 minutes, the strategy never builds a real position. Confluence requires multiple confirmations before flipping, giving each side time to develop.

> 📖 [Institutional Trend Mode →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#18-institutional-trend-mode)

---

## 📦 8 Ready-to-Trade Presets

One dropdown. Full configuration applied instantly. Your personal settings (Total Capital, Trade Sizes, Leverage, HCM targets) are always preserved.

### Hedge Profiles — Bitget & Bybit

| | `hedge_scalp` | `hedge_scalp_fast` | `hedge_institutional` | `hedge_inst_fast` |
|---|---|---|---|---|
| **Candle** | 5m | 5m | 15m | 15m |
| **Context** | 4h | 4h | 4h | 4h |
| **Target ROE** | 1.2% | 0.5% | 1.8% | 1.4% |
| **Min Profit** | 0.9% | 0.4% | 1.2% | 0.8% |
| **Trailing** | 0.5% | 0.4% | 0.75% | 0.5% |
| **Max DCA** | 2 | 3 | 2 | 2 |
| **Session Gate** | OFF (24/7) | OFF (24/7) | Full block | Scout |
| **ATR Strength** | ✅ | ✅ | ✅ | ✅ |
| **ROC Filter** | ✅ | ✅ | ✅ | ✅ |

### HMB Variants — Dynamic Ratio Active

`hedge_scalp_hmb` and `hedge_scalp_fast_hmb` — identical to Scalp and Scalp Fast, with **Dynamic Ratio (Confluence) ON**. HCM target ratio shifts automatically with market bias. Best used once you have HCM targets configured and confluence scores stabilized on your pair.

### Single-Direction Profiles — For Non-Hedge Exchanges

`orderflow_scalp_fast` (5m / 1h context, SuperTrend direction) and `orderflow_institutional_fast` (15m / 1h context, SMA200 direction) — full OrderFlow engine without HCM and SHORT management. Use on Binance Futures, dYdX v4, or Kraken Futures.

> 📖 [Full Preset Comparison →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#22-hedge-presets)

---

## 📟 The Hedge Sidebar — Both Positions, One View

```text
🌊 WM Hedge v1.0.6         LONG
ROC (annualized)            312.0%
📊 Market Type              🐂 BULLISH
──────────────────────────────────────
── 🔴 SHORT ──              0.0430 @ 66420.0
💰 SHORT ROE                -0.182%
📊 SHORT uPnL               -0.31 USDT
💥 SHORT Liq                72310.00
💸 Funding                  0.0100%
🏦 Wallet Bal               520.40 USDT
── 🟢 LONG ──               0.0420 @ 65800.0
──────────────────────────────────────
── 🏦 HCM ──
💹 Combined uPnL            +1.24 USDT
🎯 Target (USDT)            1.24 / $5.00 (+24%)
🛑 Drawdown Stop            1.24 / -$10.00 (0%)
⚖️ L/S Ratio                1.95 (target 2.00) ✅
🕐 Cycle Age                2.3h / 48.0h
──────────────────────────────────────
🌊 Signal (LONG)            🟢 BULLISH
⚖️ Imbalance                0.0612
🟢 Buy threshold            0.0580
🔴 Sell threshold           -0.0520
🧬 GA samples               850 / 180 ✓
🪤 Trap guard               CLEAR
📊 ATR Strength             📈 AVERAGE ×1.00
📉 ROC Filter               ✅ Active | p1 b3 ≥0.2%
⚡ Vol Gate                 P38 / 80
──────────────────────────────────────
📌 Mark Price               66380.00
💸 Funding Rate             🟢 0.0100% / 1h
⏰ Next Funding             42m 15s
🛡️ Funding Guard (LONG)     ✅ CLEAR (87% APR)
──────────────────────────────────────
🔪 Kill Position            $-0.31 / -$50.0 [USD]
🚨 Circuit Breaker          $0.00 / -$30.0
🛡️ Liq. Limit (%)           20% (SAFE ✓)
🪢 Rekt Liq. Price          61,200.00 (7.8% dist)
```

The SHORT slave shows its own sidebar with position, ROE, uPnL, trail stop, target exit price, liquidation price, and funding rate — live on every tick.

> 📖 [Sidebar Reference →](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#24-the-hedge-sidebar-dual-position-view)

---

## 🌐 Supported Exchanges

| Exchange | Status | Hedge Mode | Notes |
|---|---|---|---|
| **Bitget** | ✅ Final | ✅ Full Hedge | USDT-M & COIN-M futures, live & testnet |
| **Bybit** | ✅ Final | ✅ Full Hedge | USDT perpetuals, demo & live |
| Binance Futures | ✅ Single-direction | ❌ | Use `orderflow_scalp_fast` / `orderflow_institutional_fast` |
| dYdX v4 | ✅ Single-direction | ❌ | Same |
| Kraken Futures | ✅ Single-direction | ❌ | Same |

---

## 📖 Full Documentation

- [§1 — What is Hedge Mode?](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#1-what-is-hedge-mode)
- [§2 — Master / Slave Architecture](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#2-master-slave-pair-architecture)
- [§3 — OrderFlow in Hedge Mode](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#3-orderflow-trading-in-hedge-mode)
- [§4 — The Machine Learning Engine (GA)](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#4-the-machine-learning-engine)
- [§5 — Support & Resistance Confluence](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#5-support-resistance-confluence-hedge-adapted)
- [§6 — Smart S/R Bias](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#6-smart-sr-bias-direction-gate)
- [§7 — Trap Detection & Absorption](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#7-bull--bear-trap-detection)
- [§9 — Institution Hours](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#9-institution-hours)
- [§10 — Cross-Exchange Signal Pair](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#10-cross-exchange-signal-pair)
- [§11 — ATR Strength Classifier *(Hedge Exclusive)*](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#11-atr-strength-classifier-hedge-exclusive)
- [§12 — ROC Momentum Filter *(Hedge Exclusive)*](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#12-roc-momentum-filter-hedge-exclusive)
- [§13 — Kill Position — Bidirectional](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#13-kill-position-bidirectional-hedge-edition)
- [§14 — Liquidation Guard](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#14-liquidation-guard-bidirectional-protection)
- [§15 — Funding Rate Intelligence — Side-Aware](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#15-funding-rate-intelligence-side-aware)
- [§16 — HCM — Hedge Capital Management](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#16-hcm-hedge-capital-management)
- [§17 — SHORT Position Management](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#17-short-position-management)
- [§18 — Institutional Trend Mode](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#18-institutional-trend-mode)
- [§19 — Margin & Compound Profits](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#19-margin-compound-profits)
- [§22 — Hedge Presets](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#22-hedge-presets)
- [§23 — Full Settings Reference](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#23-editor-settings-reference)
- [§25 — OrderFlow vs Classic Branch](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#25-orderflow-vs-classic-branch-hedge)
- [§26 — Tips & Best Practices](https://github.com/truitaman/OrderFlow-Magic-Hedge/wiki#26-tips-best-practices-for-hedge-mode)

---

## 🛍️ Get the Strategy

Requires a valid **Gunbot license** (Ultimate, Unlimited, BR, or MM).

> 🛒 **Purchase Wick Magic Hedge — soon™**

👉 **Join the Telegram community:**  
https://t.me/+xQtQ9Y4AOc9lZTNk

Already running Wick Magic Futures?  
→ Hedge is a separate strategy file. Ask in Telegram for bundle options.

No Gunbot license yet?  
→ [Get Gunbot Ultimate](gunbot.com/es)
