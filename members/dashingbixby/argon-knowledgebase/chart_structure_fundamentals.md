# chart_structure_fundamentals.md

- **Title:** Chart Structure Fundamentals — Zero-Assumption Technical Definitions & Detectors
- **Scope:** Complete, beginner-to-programmer definitions and step-by-step detection procedures for candlesticks, timeframes, swing highs/lows, market trend vs. consolidation, support/resistance zones, level grading (freshness, double bottoms, S/R flips), range extremes vs. midpoints, liquidity sweeps, Initial Balance (IB), and Killzones.
- **Sources Used:** SRC-03 (Academy Ch. 1: Charts First)[cite: 1], SRC-06 (Academy Ch. 4)[cite: 1], SRC-08 (Academy Ch. 6)[cite: 1], SRC-12 (Academy Ch. 10)[cite: 1], SRC-25 (Academy Flowseeker Ch. 10)[cite: 1], SRC-29 (Docs Atlas Field Guide), SRC-30 (Changelog Oct 13–14, 2026).
- **Prerequisites:** `00_index.md`, `glossary.md`
- **Last Updated:** 2026-10-17

---

## Why This File Exists

Skylit's #1 rule is the **Charts First Doctrine**: *"Before a trader ever looks at a Heatseeker™ map, they must first understand how to read price structure on a chart. Heatseeker is not a signal generator. Heatseeker is a confirmation tool. Charts create the initial thesis... Without chart structure, exposure data becomes noise."* `[DOCTRINE: Academy Ch. 1, Sec. 1]`[cite: 1].

However, Skylit's Academy assumes the reader already knows what a candlestick, a swing high, or a support zone is, stating: *"If you do not have a basic understanding of market structure, technical analysis, support/resistance or liquidity, your experience trading with Skylit will be subpar."* `[DOCTRINE: Academy Ch. 1, Sec. 3]`[cite: 1].

This file supplies the exact mathematical and procedural definitions so an agent with **zero prior trading knowledge** can compute every chart-structure primitive deterministically.

---

## 1. Candlesticks (OHLCV Bars) & Timeframe Hierarchy

### 1.1 Definition
- **Plain sentence:** A candlestick (bar) summarizes how an asset's price moved during a fixed slice of time using four prices: where it started (Open), the highest price reached (High), the lowest price reached (Low), and where it finished (Close).
- **Precise version:** For a discrete time interval $[t_{\text{start}}, t_{\text{end}})$, a candlestick $C_i$ is a 6-tuple $(t_i, O_i, H_i, L_i, C_i, V_i)$ where:
  - $O_i =$ first traded price at or after $t_{\text{start}}$ `[GENERAL TA]`
  - $H_i = \max(\text{price})$ during $[t_{\text{start}}, t_{\text{end}})$ `[GENERAL TA]`
  - $L_i = \min(\text{price})$ during $[t_{\text{start}}, t_{\text{end}})$ `[GENERAL TA]`
  - $C_i =$ last traded price before $t_{\text{end}}$ `[GENERAL TA]`
  - $\text{Body}_i = |C_i - O_i|$ (Green/Bullish if $C_i > O_i$; Red/Bearish if $C_i < O_i$) `[GENERAL TA]`
  - $\text{Upper Wick}_i = H_i - \max(O_i, C_i)$ `[GENERAL TA]`
  - $\text{Lower Wick}_i = \min(O_i, C_i) - L_i$ `[GENERAL TA]`
  - $\text{Range}_i = H_i - L_i$ `[GENERAL TA]`

### 1.2 Timeframe Hierarchy & Atlas RTH Alignment
- **The Highway Analogy:** Skylit defines a strict top-down timeframe hierarchy for marking levels `[DOCTRINE: Academy Ch. 1, Sec. 4.3]`[cite: 1]:
  1. **Daily / Weekly (`1D`, `1W`) = "Major Highways":** Most significant structural levels; always mark first `[DOCTRINE: Academy Ch. 1, Sec. 4.3]`[cite: 1].
  2. **Hourly (`1H`, `4H`) = "Local Roads":** Intermediate structure (e.g., NQ hourly double bottom in Ch. 1; NQ hourly resistance in Ch. 4) `[DOCTRINE: Academy Ch. 1, Sec. 4.3 & 5; Ch. 4, Sec. 5]`[cite: 1].
  3. **5-Minute (`5m`, plus `1m`/`15m`) = "Side Streets":** Intraday execution timeframe (e.g., SPY 5-minute Atlas chart in Flowseeker Ch. 8) `[DOCTRINE: Academy Ch. 1, Sec. 4.3; Flowseeker Ch. 8, Sec. 5]`[cite: 1].
- **Atlas RTH Hourly Candle Alignment Rule:** When Regular Trading Hours (`Extended hours: Off`) is active on Atlas, hourly (`1H`) and multi-hour candles start at **09:30 ET** (reading `09:30`, `10:30`, `11:30`, etc.), rather than `09:00` or `10:00` `[DOCTRINE: Changelog Oct 13, 2026]`.

### 1.3 Inputs
- Raw trade ticks or aggregated OHLCV series for symbol $S$.
- Session mode (`RTH_ONLY = true/false`, where US equities RTH is `09:30:00` to `16:00:00` America/New_York) `[DOCTRINE: Changelog Oct 13, 2026]`.
- Target timeframes: `1W`, `1D`, `1H`, `15m`, `5m`, `1m`.

### 1.4 Procedure
1. Ingest OHLCV bars for `1D`, `1H`, and `5m`.
2. If `RTH_ONLY == true` and timeframe is `1H` or multi-hour (`< 1D`), anchor bar boundaries to `09:30 ET` (`09:30–10:30`, `10:30–11:30`, ..., `15:30–16:00`) `[DOCTRINE: Changelog Oct 13, 2026]`.
3. Process structure in top-down order: `1D` $\rightarrow$ `1H` $\rightarrow$ `5m` `[DOCTRINE: Academy Ch. 1, Sec. 4.3]`[cite: 1].

### 1.5 Edge Cases
- **Final RTH Hour (`15:30–16:00 ET`):** Because RTH is 6.5 hours (`09:30–16:00`), a `1H` chart anchored at `09:30 ET` has six 60-minute bars and one 30-minute bar (`15:30–16:00 ET`) `[INFERENCE]`.
- **Overnight Gaps:** When `RTH_ONLY == true`, $O_{\text{today, 09:30}} \neq C_{\text{yesterday, 16:00}}$. The price space between yesterday's RTH close (or high/low) and today's RTH open is an **overnight gap** (used in the SPY 755 gap-fill setup in `SRC-01`) `[DOCTRINE: Academy Dark Pool Course, Sec. 6]`[cite: 1].

### 1.6 Worked Example (Illustrative)
- Symbol: `SPY`, Timeframe: `5m`, Bar `09:30–09:35 ET`: Open = `580.00`, High = `581.20`, Low = `579.50`, Close = `580.80`.
- Calculation: Bullish candle ($580.80 > 580.00$); Body = $\$0.80$; Upper Wick = $581.20 - 580.80 = \$0.40$; Lower Wick = $580.00 - 579.50 = \$0.50$; Total Bar Range = $\$1.70$.

### 1.7 What It Is NOT
- A candlestick is **not** a Contract Flow bar in Flowseeker Drilldown (which measures option contract volume by bid/mid/ask) `[DOCTRINE: Academy Flowseeker Ch. 6, Sec. 4]`[cite: 1], nor is it an Atlas Flow Pane bar (which measures option dollar premium) `[DOCTRINE: Academy Flowseeker Ch. 8, Sec. 2]`[cite: 1].

### 1.8 Related Concepts
- See `session_timing.md` for RTH, Extended Hours, and Killzone windows.

---

## 2. Swing Highs and Swing Lows (Structural Building Blocks)

### 2.1 Definition
- **Plain sentence:** A **swing high** is a local peak where price moved up and then reversed lower; a **swing low** is a local trough where price moved down and then reversed higher `[DOCTRINE: Academy Ch. 1, Sec. 4.1]`[cite: 1].
- **Precise version:**
  - Skylit defines swing points qualitatively: *"A swing high is a peak where price reversed lower. A swing low is a trough where price reversed higher. These are your building blocks. Mark the ones that are obvious — if you have to squint to see it, it probably doesn't matter... The best levels are the ones you can spot in under five seconds."* `[DOCTRINE: Academy Ch. 1, Sec. 4.1 & 4.5]`[cite: 1].
  - Because Skylit does not specify the exact bar-count $k$ for automated code `[NOT SPECIFIED]`, a software agent implements a parameterized $k$-bar fractal + minimum reversal magnitude filter `[GENERAL TA]`:
    - Bar $i$ is a **Confirmed Swing High** of order $k$ if $H_i = \max_{j \in [i-k, i+k]} H_j$, $H_i > H_{i+k}$, and the subsequent drop from $H_i$ within $m$ bars is at least $\theta_{\text{ATR}} \times \text{ATR}_{14}$ `[GENERAL TA]`.
    - Bar $i$ is a **Confirmed Swing Low** of order $k$ if $L_i = \min_{j \in [i-k, i+k]} L_j$, $L_i < L_{i+k}$, and the subsequent bounce from $L_i$ within $m$ bars is at least $\theta_{\text{ATR}} \times \text{ATR}_{14}$ `[GENERAL TA]`.

### 2.2 Inputs
- OHLCV bar series on timeframe $TF \in \{\text{1D}, \text{1H}, \text{15m}, \text{5m}\}$.
- Fractal lookback/lookforward window $k$ (Default `[GENERAL TA]`: $k = 3$ to $5$ bars for minor swings; $k = 10$ to $20$ bars for major "obvious" swings; exact value `[NOT SPECIFIED]` in Skylit docs).
- Average True Range $\text{ATR}_{14}$ on timeframe $TF$ to measure reversal magnitude `[GENERAL TA]`.

### 2.3 Procedure (Major vs. Minor Swing Detection & Deduplication)
1. **Compute ATR(14):** For each bar $i$, compute True Range $TR_i = \max(H_i - L_i, |H_i - C_{i-1}|, |L_i - C_{i-1}|)$ and $\text{ATR}_{14, i} = \text{SMA}_{14}(TR)$ `[GENERAL TA]`.
2. **Identify Candidate Swing Highs/Lows:**
   - Scan closed bars up to current bar $t - k$ (a swing point at $i$ is only confirmed once $k$ bars have closed after $i$, preventing lookahead bias) `[GENERAL TA]`.
3. **Filter for "Obvious" (Major) vs. Minor Swings:**
   - Skylit states: *"levels created by large, dramatic reversals carry more weight than those formed by shallow ones, because a bigger reaction signals stronger conviction from market participants"* and *"sharp reversals carry more weight than slow, grinding reactions"* `[DOCTRINE: Academy Ch. 1, Sec. 4]`[cite: 1].
   - Compute **Reversal Magnitude** $\Delta_{\text{rev}}$: the maximum price excursion away from $H_i$ (or $L_i$) before $H_i$ (or $L_i$) is breached `[GENERAL TA]`.
   - Compute **Reversal Speed** $v_{\text{rev}} = \Delta_{\text{rev}} / N_{\text{bars}}$ over the first 3–5 bars after $i$ `[GENERAL TA]`.
   - Classify as **Major Swing** if $\Delta_{\text{rev}} \ge 1.5 \times \text{ATR}_{14}$ and $k \ge 5$; classify as **Minor Swing** if $0.75 \times \text{ATR}_{14} \le \Delta_{\text{rev}} < 1.5 \times \text{ATR}_{14}$; discard as noise if $\Delta_{\text{rev}} < 0.75 \times \text{ATR}_{14}$ `[GENERAL TA]` *(exact multipliers `[NOT SPECIFIED]` in Skylit docs)*.
4. **Merge Nearby Candidates:** If two swing highs (or two swing lows) sit within a cluster tolerance $\epsilon_{\text{zone}}$ (e.g., $\le \$0.50$ on SPY/QQQ or $\le \$5.00$ on SPX, matching Skylit's deflection zone width `[INFERENCE]`), merge them into a single **Structural Zone** $[P_{\text{low}}, P_{\text{high}}]$ rather than drawing two separate lines `[DOCTRINE: Academy Ch. 1, Sec. 4: "Support is a price zone... Keep It Simple"]`[cite: 1].

### 2.4 Edge Cases
- **Equal Highs / Equal Lows (Ties):** If $H_i == H_{i+1}$ at a peak, treat the pair as a single swing high anchored at the later timestamp $i+1$ `[GENERAL TA]`. If two distinct swing lows occur separated by an intermediate swing high and sit within $\epsilon_{\text{zone}}$ of each other, classify the structure as a **Double Bottom** (see Section 4.4) `[DOCTRINE: Academy Ch. 1, Sec. 4.2]`[cite: 1].
- **Unconfirmed Right-Hand Side:** At the live right edge of the chart (fewer than $k$ bars since the high/low), a peak is an *unconfirmed intraday extreme*, not yet a historical swing point `[GENERAL TA]`.

### 2.5 Worked Example (Illustrative)
- On a `1H` QQQ chart ($\text{ATR}_{14} = \$2.00$), bar $i=50$ reaches $H_{50} = 612.00$. The preceding 5 bars and following 5 bars all have highs $< 612.00$. Over the next 4 bars, QQQ drops to $607.20$ ($\Delta_{\text{rev}} = \$4.80 = 2.4 \times \text{ATR}_{14}$).
- Result: $612.00$ is confirmed at bar $55$ as a **Major `1H` Swing High**.

### 2.6 What It Is NOT
- A single tall candle wick inside a choppy midday range that reverses by only $0.2 \times \text{ATR}$ is **not** a major swing point ("if you have to squint to see it, it probably doesn't matter") `[DOCTRINE: Academy Ch. 1, Sec. 4.1]`[cite: 1].

### 2.7 Related Concepts
- Used directly to build **Support and Resistance Zones** (Section 4) and **Trend vs. Range State** (Section 3).

---

## 3. Market Structure State: Trend vs. Range (Consolidation)

### 3.1 Definition
- **Plain sentence:** A market is **trending up** when it makes a sequence of higher swing highs and higher swing lows, **trending down** when it makes lower swing highs and lower swing lows, and **ranging (consolidating)** when it oscillates horizontally between a fixed upper boundary (Range High) and lower boundary (Range Low) `[DOCTRINE: Academy Ch. 1, Sec. 2 & 4]`[cite: 1].
- **Precise version:** Given the last two confirmed Major Swing Highs ($SH_{n-1}, SH_n$) and last two confirmed Major Swing Lows ($SL_{n-1}, SL_n$) on timeframe $TF$:
  - **Uptrend (`TREND_UP`):** $SH_n > SH_{n-1} + \epsilon_{\text{zone}}$ AND $SL_n > SL_{n-1} + \epsilon_{\text{zone}}$ (`Higher Highs + Higher Lows`) `[DOCTRINE: Academy Ch. 1, Sec. 2]`[cite: 1].
  - **Downtrend (`TREND_DOWN`):** $SH_n < SH_{n-1} - \epsilon_{\text{zone}}$ AND $SL_n < SL_{n-1} - \epsilon_{\text{zone}}$ (`Lower Highs + Lower Lows`) `[DOCTRINE: Academy Ch. 1, Sec. 2]`[cite: 1].
  - **Range-Bound (`RANGE`):** Swing highs stall near a common ceiling ($\vert{}SH_n - SH_{n-1}\vert{} \le \epsilon_{\text{zone}}$) and/or price oscillates inside a bounded horizontal channel $[R_{\text{low}}, R_{\text{high}}]$ without establishing progressive higher/lower swings `[DOCTRINE: Academy Ch. 1, Sec. 4 & 5]`[cite: 1].

### 3.2 Inputs
- Ordered sequence of confirmed Major Swing Highs and Swing Lows on timeframe $TF$.
- Current spot price $P_{\text{spot}}$.

### 3.3 Procedure (Range Geometry & Extreme vs. Midpoint Classification)
When market structure is in a `RANGE` (or bounded between two structural levels $R_{\text{low}}$ and $R_{\text{high}}$):
1. **Define Range Boundaries:**
   - $\text{Range High } (R_{\text{high}}) = \text{upper structural resistance boundary}$ `[DOCTRINE: Academy Ch. 1, Sec. 4]`[cite: 1].
   - $\text{Range Low } (R_{\text{low}}) = \text{lower structural support boundary}$ `[DOCTRINE: Academy Ch. 1, Sec. 4]`[cite: 1].
   - $\text{Total Width } (W) = R_{\text{high}} - R_{\text{low}}$.
   - $\text{Midpoint } (R_{\text{mid}}) = \frac{R_{\text{high}} + R_{\text{low}}}{2}$ `[DOCTRINE: Academy Ch. 1, Sec. 4 & 5]`[cite: 1].
2. **Compute Relative Location $L_{\%} \in [0, 1]$:**
   $$L_{\%} = \frac{P_{\text{spot}} - R_{\text{low}}}{R_{\text{high}} - R_{\text{low}}}$$
3. **Classify Price Location Zone:**
   - Skylit states: *"A range consists of three primary zones: 1. Range High, 2. Range Low, 3. Midpoint. Range highs and lows represent extremes. The midpoint represents equilibrium... We trade extremes. We avoid midpoints."* `[DOCTRINE: Academy Ch. 1, Sec. 4]`[cite: 1].
   - Because Skylit requires a **minimum 3:1 Reward-to-Risk** to the opposite side of the range `[DOCTRINE: Academy Ch. 3, Sec. 6; Ch. 6, Sec. 9]`[cite: 1], mathematically price must be within the outer **25%** of the range (or within the node Deflection Zone of $R_{\text{low}}$ / $R_{\text{high}}$) to achieve $\ge 3:1$ R:R across the range `[INFERENCE]`:
     - **`AT_RANGE_LOW` (Bullish Extreme):** $P_{\text{spot}} \in [R_{\text{low}} - \delta_{\text{deflect}}, R_{\text{low}} + \max(\delta_{\text{deflect}}, 0.20 \times W)]$.
     - **`AT_RANGE_HIGH` (Bearish Extreme):** $P_{\text{spot}} \in [R_{\text{high}} - \max(\delta_{\text{deflect}}, 0.20 \times W), R_{\text{high}} + \delta_{\text{deflect}}]$.
     - **`AT_MIDPOINT` (No-Trade Equilibrium Zone):** $L_{\%} \in (0.25, 0.75)$ and not touching an intermediate structural node `[INFERENCE]`.

### 3.4 Edge Cases
- **Momentum Inside a Range Midpoint:** If price is moving rapidly (large candles) through $R_{\text{mid}}$, Skylit explicitly warns: *"Mistake 3 — Confusing Momentum With Structure. Just because price is moving quickly does not mean a trade exists. Structure determines opportunity, not speed."* `[DOCTRINE: Academy Ch. 1, Sec. 5]`[cite: 1].

### 3.5 Worked Example (`[DOCTRINE: Academy Ch. 1, Sec. 5 Case Study]`)
- **Inputs:** SPX has a clearly defined range `[DOCTRINE: Academy Ch. 1, Sec. 5]`[cite: 1]:
  - $\text{Range High } (R_{\text{high}}) = 6795$
  - $\text{Midpoint } (R_{\text{mid}}) = 6782$ *(note: mathematical center of 6795 and 6770 is 6782.5; Skylit rounds to 6782)*[cite: 1]
  - $\text{Range Low } (R_{\text{low}}) = 6770$
- **Event:** Price moves downward toward `6770` (e.g., spot = `6771`, $L_{\%} = (6771 - 6770)/25 = 4\%$).
- **Classification:** `AT_RANGE_LOW`. The agent forms the structural hypothesis: *"Price may react at this level — range low on SPX"* and now queries Heatseeker to confirm `[DOCTRINE: Academy Ch. 1, Sec. 5]`[cite: 1].

### 3.6 What It Is NOT
- A trend is **not** a single fast 5-minute candle; a range extreme is **not** any arbitrary price where a large options print appeared `[DOCTRINE: Academy Ch. 1, Sec. 5; Flowseeker Ch. 1, Sec. 3]`[cite: 1].

### 3.7 Related Concepts
- See `no_trade_conditions.md` (Gate 1: Midpoint Prohibition) and `entries_stops_targets_and_rr.md`.

---

## 4. Support & Resistance (S/R) Zones, Level Grading & Exceptions

### 4.1 Definition
- **Plain sentence:** **Support** is a price zone ("floor" / area of memory) where buyers previously stepped in and pushed price higher; **Resistance** is a price zone ("ceiling" / area of memory) where sellers previously stepped in and pushed price lower `[DOCTRINE: Academy Ch. 1, Sec. 4]`[cite: 1].
- **Precise version:** A SupportZone or ResistanceZone $Z$ is a price interval $[P_Z - \epsilon_Z, P_Z + \epsilon_Z]$ anchored at a confirmed Major Swing Low (Support) or Major Swing High (Resistance), or a Prior Day High/Low/Close (`PDH`, `PDL`, `PDC`) or gap boundary, characterized by 5 grading attributes `[DOCTRINE: Academy Ch. 1, Sec. 4; Ch. 10, Sec. 6]`[cite: 1].

### 4.2 Inputs
- Confirmed Swing Highs/Lows across `1D`, `1H`, and `5m` timeframes.
- Historical bar series since the swing point was formed (to count subsequent tests/taps).
- Current timestamp $t_{\text{now}}$.

### 4.3 Procedure: Grading a Level's Strength
Skylit defines five criteria that make a chart level "Good" (Strong) vs. "Weak" `[DOCTRINE: Academy Ch. 1, Sec. 4]`[cite: 1]. An agent scores each candidate level $Z$ across these five dimensions:

1. **Freshness (Touch Count $N_{\text{touches}}$ Since Creation):**
   - *"A level that has not been tapped yet is stronger than one that has been tested multiple times. Each time price touches a level, it degrades — the reaction becomes weaker because the orders sitting there get absorbed. Fresh levels carry the most potential energy."* `[DOCTRINE: Academy Ch. 1, Sec. 4.2]`[cite: 1].
   - Count how many distinct times price returned to $[P_Z - \epsilon_Z, P_Z + \epsilon_Z]$ after the swing point was confirmed:
     - $N_{\text{touches}} = 0 \implies$ **Fresh** (Highest grade) `[DOCTRINE: Academy Ch. 1, Sec. 4.2]`[cite: 1].
     - $N_{\text{touches}} = 1 \implies$ **Second Tap / Double Bottom or Top Candidate** (Playable exception; see 4.4) `[DOCTRINE: Academy Ch. 1, Sec. 4.2]`[cite: 1].
     - $N_{\text{touches}} \ge 2 \implies$ **Degraded / Weak** (Avoid fading) `[DOCTRINE: Academy Ch. 1, Sec. 4.2; Ch. 6, Sec. 6]`[cite: 1].
2. **Magnitude of the Original Reversal ($\Delta_{\text{rev}}$):**
   - *"Levels created by large, dramatic reversals carry more weight than those formed by shallow ones"* `[DOCTRINE: Academy Ch. 1, Sec. 4]`[cite: 1].
3. **Clean Price Action / Sharpness of Reversal ($v_{\text{rev}}$):**
   - *"Sharp reversals carry more weight than slow, grinding reactions"* vs. *"Choppy, unclear reactions around it"* `[DOCTRINE: Academy Ch. 1, Sec. 4]`[cite: 1].
4. **Multiple Timeframe Alignment (`MTF_Align`):**
   - *"The level shows up on more than one timeframe"* (e.g., visible on both `1D`/`1H` and `5m`, vs. *"Only visible on a very low timeframe"*) `[DOCTRINE: Academy Ch. 1, Sec. 4]`[cite: 1].
5. **Recency ($\Delta t = t_{\text{now}} - t_{\text{created}}$):**
   - *"A support level from last week is more relevant than one from six months ago."* `[DOCTRINE: Academy Ch. 1, Sec. 4.4]`[cite: 1]. Weight recent swing points ($\le 5\text{--}20$ trading days) above levels $\ge 180$ days old `[INFERENCE]`.

### 4.4 The Two Doctrine Exceptions to Degraded Levels
Skylit explicitly defines two exceptions where a previously touched level remains strong `[DOCTRINE: Academy Ch. 1, Sec. 4.2]`[cite: 1]:
1. **Exception 1 — Double Bottoms (and Double Tops):**
   - *"Double bottoms — a second tap at the same low can produce a strong reversal. This is the one scenario where a re-test is worth playing."* `[DOCTRINE: Academy Ch. 1, Sec. 4.2; Ch. 1, Sec. 5 NQ hourly double bottom; Ch. 10, Sec. 12 SPX double bottom]`[cite: 1].
   - **Rule:** When $N_{\text{touches}} == 1$ at a major support low (the 2nd overall touch of that price), flag as `DOUBLE_BOTTOM_CANDIDATE` (valid bullish thesis) `[DOCTRINE: Academy Ch. 1, Sec. 4.2]`[cite: 1].
2. **Exception 2 — S/R Flips (Support-Becomes-Resistance / Resistance-Becomes-Support):**
   - *"S/R flips — when a former resistance level becomes support (or vice versa), the level takes on new meaning. It's essentially fresh in its new role."* `[DOCTRINE: Academy Ch. 1, Sec. 4.2]`[cite: 1].
   - **Rule:** If price breaks decisively through a ResistanceZone $Z$ and closes beyond it by $> \epsilon_Z$, convert $Z$ into a SupportZone and **reset its touch counter in its new role to $N_{\text{touches\_new\_role}} = 0$ (Fresh)** `[DOCTRINE: Academy Ch. 1, Sec. 4.2]`[cite: 1]. (Example in `SRC-12`: Previous Day Low `PDL` on SPX at `6775` acting as overhead resistance after a breakdown `[DOCTRINE: Academy Ch. 10, Sec. 6]`[cite: 1].)

### 4.5 Edge Cases
- **Spider-Web Charts (Too Many Lines):** If the detector identifies $> 5$ S/R zones within the day's Expected Move, keep only the top 3–5 highest-graded zones (prioritizing `1D`/`1H`, Fresh, and `PDH`/`PDL`/Gap levels) `[DOCTRINE: Academy Ch. 1, Sec. 4: "New traders tend to draw too many lines... Stick to the obvious levels"]`[cite: 1].

### 4.6 Worked Example (`[DOCTRINE: Academy Ch. 10, Sec. 6 & 12]`)
- **Case A (S/R Flip):** SPX broke below Previous Day Low (`6775`). When SPX rallies back up to `6775` from below, `6775` acts as a fresh **S/R Flip Resistance** level `[DOCTRINE: Academy Ch. 10, Sec. 6]`[cite: 1].
- **Case B (Double Bottom):** SPX drops to `6775`, bounces, and later returns to `6775` for a second touch, forming a **Double Bottom** on support $\rightarrow$ valid bullish thesis when paired with an SPX Reverse Rug at `6775` `[DOCTRINE: Academy Ch. 10, Sec. 12]`[cite: 1].

### 4.7 What It Is NOT
- Support/Resistance is **not** an exact penny line; it is a **price zone** ("not a guarantee, but an area where demand/supply has shown up before") `[DOCTRINE: Academy Ch. 1, Sec. 4]`[cite: 1].

### 4.8 Related Concepts
- Combines directly with **Heatseeker Floors, Ceilings, and King Nodes** (`heatseeker_node_core_attributes.md`).

---

## 5. Liquidity Sweeps, Initial Balance (IB) & Killzones

### 5.1 Definitions
1. **Mapped Liquidity Sweep (Summer Bootcamp 4th Gate):**
   - **Plain sentence:** Price briefly pokes below a prior swing low (or above a prior swing high) to trigger resting stop-loss orders and then closes back inside the range `[DOCTRINE: Academy Flowseeker Ch. 8, Sec. 5 & Ch. 10, Sec. 2; GENERAL TA]`[cite: 1].
   - **Precise version:** In Flowseeker Ch. 10, Skylit notes that the stricter Summer Bootcamp checklist adds a fourth gate to the A+ setup: *"a mapped liquidity sweep — all four gates, or no trade"* `[DOCTRINE: Academy Flowseeker Ch. 10, Sec. 2]`[cite: 1]. In Flowseeker Ch. 8's case study, this is illustrated on a 5-minute Atlas chart as: *"price sells off into 578 [range low / floor] and wicks below it, then closes back above."* `[DOCTRINE: Academy Flowseeker Ch. 8, Sec. 5]`[cite: 1].
   - **Detector Rule:** At a support/floor level $P_{\text{floor}}$, candle $C_i$ (or a 2–3 bar sequence) has $L_i < P_{\text{floor}}$ (sweeping below the level/prior low) followed by a candle close $C_i \ge P_{\text{floor}}$ `[INFERENCE from Flowseeker Ch. 8, Sec. 5]`[cite: 1].
2. **Initial Balance (`IB` Indicator on Atlas):**
   - **Plain sentence:** The high-to-low price range established during the first 15, 30, or 60 minutes of a major trading session (New York, London, or Asia) `[DOCTRINE: Changelog Oct 13, 2026]`.
   - **Precise version:** For session $S \in \{\text{NY}, \text{London}, \text{Asia}\}$ and duration $D \in \{15\text{m}, 30\text{m}, 60\text{m}\}$:
     - $\text{IB High} = \max(H)$ over $[t_{\text{session\_open}}, t_{\text{session\_open}} + D)$ `[DOCTRINE: Changelog Oct 13, 2026]`.
     - $\text{IB Low} = \min(L)$ over $[t_{\text{session\_open}}, t_{\text{session\_open}} + D)$ `[DOCTRINE: Changelog Oct 13, 2026]`.
     - $\text{IB Midpoint} = (\text{IB High} + \text{IB Low}) / 2$ `[DOCTRINE: Changelog Oct 13, 2026]`.
     - Also draws Fibonacci retracement lines ("shaded golden zone", typically $50\%\text{--}61.8\%$ `[GENERAL TA]`, exact ratios `[NOT SPECIFIED]` in changelog) and an optional extension ladder `[DOCTRINE: Changelog Oct 13, 2026]`.
3. **Killzones (`Killzones` Indicator on Atlas):**
   - **Plain sentence:** Time-boxed intraday windows (`ASIA`, `LNDN`, `NYAM`, `NYPM`) drawn on charts $\le 30\text{m}$ with horizontal lines marking each session's high and low `[DOCTRINE: Changelog Oct 13, 2026]`.

### 5.2 Inputs
- Intraday OHLCV bars (`1m`, `5m`, `15m`, `30m`).
- Session open timestamps and user-selected IB duration (`15m`, `30m`, or `60m`).

### 5.3 Procedure
1. **Track Killzone Highs/Lows:** On any chart with timeframe $\le 30\text{m}$, track the running High and Low of `ASIA`, `LNDN`, `NYAM`, and `NYPM` windows `[DOCTRINE: Changelog Oct 13, 2026]`.
2. **Lock Initial Balance:** At $t_{\text{open}} + D$ (e.g., `10:00 ET` for a 30m NY IB or `10:30 ET` for a 60m NY IB), lock $\text{IB High}$, $\text{IB Low}$, and $\text{IB Midpoint}$ for the rest of the session `[DOCTRINE: Changelog Oct 13, 2026]`.
3. **Check Liquidity Sweep Gate (if Bootcamp 4-Gate Mode is enabled):** When price tests a major S/R + Node extreme, verify whether price wicked beyond the level/swing point and closed back inside `[DOCTRINE: Academy Flowseeker Ch. 8, Sec. 5; Ch. 10, Sec. 2]`[cite: 1].

### 5.4 Edge Cases
- **Timeframes $> 30\text{m}$:** The Atlas `Killzones` indicator only renders on `30m` charts and under (`1m`, `5m`, `15m`, `30m`) `[DOCTRINE: Changelog Oct 13, 2026]`.

### 5.5 Worked Example (`[DOCTRINE: Academy Flowseeker Ch. 8, Sec. 5]`)
- SPY is trading on a `5m` Atlas chart with a week-long range of `578` to `590` and a Heatseeker floor node at `578`.
- Price sells off into `578`, wicks below `578` (e.g., low of `577.65`), and the 5-minute candle closes back above `578` (e.g., `578.20`) while a tall ask-side call bar prints in the Flow Pane `[DOCTRINE: Academy Flowseeker Ch. 8, Sec. 5]`[cite: 1].
- Both the Charts First Floor Hold and the Bootcamp Liquidity Sweep gate are satisfied.

### 5.6 What It Is NOT
- A candle that breaks below `578` and **closes and holds** at `576.80` is **price acceptance below support** (invalidation), **not** a liquidity sweep `[DOCTRINE: Academy Flowseeker Ch. 7, Sec. 6; Ch. 8, Sec. 5]`[cite: 1].

### 5.7 Related Concepts
- See `session_timing.md` and `charts_first_workflow.md`.

---

## Agent Checklist (`chart_structure_fundamentals.md`)

1. **Ingest Multi-Timeframe Bars:** Load `1D` (highways), `1H` (local roads, anchored at `09:30 ET` if RTH-only), and `5m` (side streets) `[DOCTRINE: Academy Ch. 1, Sec. 4.3; Changelog Oct 13, 2026]`[cite: 1].
2. **Detect & Grade S/R Zones:** Extract confirmed Swing Highs/Lows, `PDH`/`PDL`, and Gap boundaries; score each by Freshness ($N_{\text{touches}} == 0$, or Double Bottom $N_{\text{touches}} == 1$, or S/R Flip reset), Reversal Magnitude, Sharpness, MTF Alignment, and Recency `[DOCTRINE: Academy Ch. 1, Sec. 4]`[cite: 1].
3. **Classify Structure State:** Determine whether the active chart is `TREND_UP`, `TREND_DOWN`, or `RANGE` `[DOCTRINE: Academy Ch. 1, Sec. 2 & 5]`[cite: 1].
4. **Evaluate Location Gate:** Compute whether current spot price is at a structural extreme (`AT_RANGE_LOW`, `AT_RANGE_HIGH`, or testing a fresh `1D`/`1H` S/R zone) vs. `AT_MIDPOINT`. If `AT_MIDPOINT`, halt and output `NO_TRADE_MIDPOINT` `[DOCTRINE: Academy Ch. 1, Sec. 4 & 5]`[cite: 1].