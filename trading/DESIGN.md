# Trading engines on the Legacy

trading/DESIGN.md — the design for trading consumers: strategies that earn trust, risk that is checked rather than trusted, and no action without HAL.

Function:
    James, 2026-11-18: "what about a trading bot as a consumer", then
    "organise our consumer repo and add trading engines". This is the
    design for trading consumers on the consumer runtime: the engines,
    what each holds in the Legacy, how HAL gates every order, and how
    a bot proves it fails safely. It is a design, not a spec. Nothing is
    built until James signs it off, and nothing trades real money until
    he seals a live mandate (§6).

    Open, for James: which market and exchange (§10), the HAL mode
    (§6), and whether ccemk2 (his earlier trading stack, a reference
    under the archived the-full-stack) is the starting point.

---

## 1. What the Legacy gives a trading bot, and what it does not

It makes the bot **honest**: every price it saw, every signal, every
order, fill and refusal is recorded and hash-chained. Its strategies
earn trust only from outcomes. Risk limits are checked, not hoped
for. And no order goes out without a human answerable for it.

It does **not** make a strategy profitable. Most are not. The Legacy's
job is that when one fails, the failure is small, recorded, stopped,
and explained.

## 2. Faces: keeping what is known apart from what is believed

| Face | In trading | Where it lives |
|---|---|---|
| **FACT** | fills, positions at the daily close, balances, fees paid: what happened | daily summaries as P9 claims; every fill in the consumer's ledger |
| **OPINION** | a strategy's signal or forecast: what it believes | the consumer's store; never a P9 FACT |
| **CONTEXT** | the market regime, scheduled events, the exchange's rules | P9 claims (slow-changing) |
| **UNKNOWN** | data gaps, a stale feed, a regime nobody can classify | the consumer's store; open ones in the turn report and the Weaver's briefing |
| **COUNTER** | evidence against a strategy's thesis (it lost where it said it would win) | from outcomes (§4.3), shown with the strategy |
| FICTION | not used | — |

**Fast data is not canon.** Prices and signals arrive too fast to be
signed P9 claims. They live in the consumer's own Mongo-style store
(`$FORGE_HOME/consumers/<name>/`, Extended JSON, append-only,
hash-chained, like ai-gate). P9 holds the slow, canonical things:

- **entity:** the trading consumer (`Trader`);
- **subjects:** each instrument (`BTC-USD`), each strategy
  (`mean-reversion-1`) and the exchange;
- **claims:** instrument facts (tick size, fees), strategy parameters,
  daily FACT summaries, CONTEXT, and the mandate's terms.

## 3. The engines

Six engines in a pipeline. Each one does one job, and an order can only
pass through all of them in order:

    market data -> strategies -> trust (Leighton) -> risk -> HAL -> execution
                                                                      |
                                       ledger, store, digest <--------+

### 3.1 Market data engine
- **Adapters, read-only:** one per exchange, using only public data
  endpoints, with no key needed to read prices.
- **Recorded:** every candle or tick used is stored with its source and
  received time. A gap, a stale price (older than the strategy's
  `max_staleness`) or a clock skew is an **UNKNOWN**, and a strategy
  given UNKNOWN data must not signal.
- **Replay:** the same recorded data can drive a backtest (§7), so a
  live run can be reproduced.

### 3.2 Strategy engines
- A strategy is **declared**: `strategies/<name>.json` in the consumer
  repository, with the instrument, timeframe, parameters, horizon and
  `max_staleness`, pinned by hash in the sealed declaration. Changing
  it means a new hash and a new seal.
- Its code is a small, pure function: recorded data in, a signal out,
  with no I/O and no clock. Same data, same signal.
- A **signal** is an OPINION document: instrument, side
  (`long | short | flat`), strength 0-1, horizon, the data it saw (by
  hash), and its rationale.
- **The AI never makes a signal.** ai-gate can analyse ("what might
  explain this week?"), but its answers are assumptions, and an
  assumption can never be a strategy's input. A rule enforces it (§5).

### 3.3 Trust engine (Leighton λ, per strategy)
- When a signal's horizon passes, its **outcome** is measured against
  recorded prices. It becomes one P14 observation on HAL's ladder:

  | Outcome after the horizon | value |
  |---|---|
  | right, and beat its fees | 1.50 |
  | right, but not after fees | 1.00 |
  | wrong | 0.60 |
  | wrong, and hit its stop | 0.40 |

- λ per strategy and instrument: `leighton_engine_v3.compute_lambda`,
  weighted fusion, with the strategy's own `k` per day declared in its
  file (a fast strategy forgets fast). It is never stored, always
  reproducible.
- A strategy with no outcomes yet is **unrated** and trades on paper
  only, whatever the mandate says (§6).
- A losing streak lowers λ on its record alone. Below 0.60 the strategy
  is **quarantined**: its signals are still recorded and scored, so it
  can earn its way back, but none can become an order.
- COUNTER: each wrong outcome is listed with the strategy as evidence
  against its thesis.

### 3.4 Risk engine
Deterministic checks. Every one must pass, or the order is refused,
with the reason recorded:

- **Categorical (P12 Rules v3):** allowed instruments, allowed sides,
  no trading in an UNKNOWN regime, no order whose input is an
  assumption. These are the consumer's `rules/risk.json`, pinned in the
  declaration.
- **Numeric (the mandate's limits, §6):** maximum position per
  instrument, maximum order size, maximum daily loss, maximum open
  exposure, maximum orders per day, leverage (default: none).
- **Sizing:** the order size is the strategy's strength times the
  mandate's maximum, scaled by the strategy's λ tier, as the mandate
  says (for example SENIOR 100%, TRUSTED 50%, NOVICE 0%, which means
  paper only). The bot never decides to use λ this way: James does,
  in the mandate he seals.
- **Preview:** the Echo's question, "what would this order do?",
  against current positions and limits, is printed with every proposal.

### 3.5 HAL: the human answerable (§6)
No order reaches execution without a valid HAL seal: either a seal for
that order, or a sealed mandate the order fits inside.

### 3.6 Execution engine
- **Paper** (the default): fills simulated at recorded prices, with the
  exchange's fees and a declared slippage. It is the same code path as
  live except the last call.
- **Live:** the exchange adapter, used only under a sealed mandate with
  `"live": true`. Every order is idempotent (a client order id derived
  from the order's hash), so a retry can never double an order.
- Every order, fill, partial, cancel and refusal goes to the consumer's
  own **P4 ledger**, hash-chained.
- **Kill switch:** `python consumer.py trading stop <name>` writes a
  stop document. The engine checks it before every order. The same
  stop is written automatically when any limit is hit, the feed goes
  stale, a fill doesn't match its order, or the ledger fails to verify.
  After a stop, only James can resume, by sealing a new mandate.

## 4. In the Legacy, day to day

- **The turn report** (`consumer.py run trader`): positions, the day's
  P&L, each strategy's λ and tier, quarantined strategies, refusals and
  why, open UNKNOWNs, mandate time left, and **what needs James**.
- **Daily close:** a FACT summary (positions, balance, P&L, fees) is
  drafted to the Cockpit inbox. James commits it, and it becomes signed
  canon in P9.
- **The Weaver:** strategies whose λ is fading, and open UNKNOWNs, in
  the morning briefing.
- **P17 Mediator:** the trading digest (which predicates and faces the
  consumer uses) joins the others.
- **The Legacy page:** a Trading tab, view-only, like the Story tab
  (served by the consumer runtime). It never places orders. On the VPS
  it is view-only and keyless.

## 5. Rules that make it safe by construction

1. **No order without HAL** (a per-order seal, or a mandate it fits).
2. **No signal from an assumption:** AI output can inform James, never
   an order.
3. **No order from an unrated or quarantined strategy,** except on paper.
4. **No order on UNKNOWN data** (gap, stale, skewed).
5. **No live order without a sealed live mandate.** Paper is the default.
6. **Any surprise stops everything** (§3.6). Resuming is James's act.
7. **Keys:** exchange keys are trade-only, **withdrawals disabled**,
   held on the laptop only, loaded like the Forge key, never logged,
   and never on the VPS.

## 6. HAL: two modes, James chooses per mandate

**Per order:** each proposed order is a pending document. James
approves or refuses it with `consumer.py trading seal <order-id>
approve|refuse --lambda L`, a HAL v2 seal like ai-gate's. This is
safe and slow, fit for a few trades a day.

**Mandate:** James seals one bounded policy capsule. Within it, the bot
acts on its own, and every order still passes the risk engine and is
logged:

    {"scp_id": "trader/mandate-v1", "declaration": {
       "type": "trading_mandate", "live": false,
       "instruments": ["BTC-USD"], "strategies": ["mean-reversion-1"],
       "max_order": "50 USD", "max_position": "200 USD",
       "max_daily_loss": "20 USD", "max_orders_per_day": 10,
       "leverage": "none", "sizing_by_tier": {"SENIOR": 100, "TRUSTED": 50, "NOVICE": 0},
       "expires": "2026-12-18T00:00:00Z"},
     "signature": {...}}

- It is sealed with the Forge key (P3 / P2), and **HAL records it:** a
  HAL v2 approve seal on the mandate itself, with James as the
  Consequence Authoriser.
- When the mandate expires, a limit is hit, or a stop is written, the
  bot stops. There is no automatic renewal.
- Amounts are strings with a currency, never floats.
- **The first mandate is paper.** A live mandate is a separate decision,
  and the design recommends it only after a strategy has reached
  TRUSTED or better on paper over a declared period.

## 7. Proving it fails safely

- **Backtest:** recorded or historical data replayed through the same
  engines (paper execution), giving the same outcomes, λ and refusals
  as live would have. A backtest's outcomes are tagged and never count
  towards live λ.
- **The Saboteur:** trading threats in `threats_v0_4/`. The bot must
  stop loudly, and never trade, on each of:
  - a corrupted price;
  - a stale feed;
  - a duplicated fill;
  - a fill for an order it never sent;
  - a reordered ledger;
  - a clock jump;
  - a tampered strategy file (hash mismatch);
  - an expired mandate;
  - a forged seal.
- **Tests** for every engine. No test touches a network or a real key.

## 8. Phases

| Phase | What runs | Gate to the next |
|---|---|---|
| 0. Backtest | recorded data, paper execution | strategies rated; every Saboteur threat stops it |
| 1. Paper, live data | real prices, simulated fills, a paper mandate | a strategy reaches TRUSTED over a declared period |
| 2. Live, tiny | a live mandate with small limits, one instrument | James's seal, and a review of phase 1's record |
| 3. Live | larger limits, more strategies | each step is a new sealed mandate |

## 9. What it needs from the stack (gaps)

- **The consumer runtime's library shape** (consumer SPEC §10): a
  trading consumer runs a loop and has strategy code, so it imports the
  runtime's pieces rather than being purely declared.
- **P12 numeric matchers:** P12 v3 matches subject, face and
  object-contains only. Numeric limits live in the risk engine for now.
  A P12 v4 with numeric comparisons would move them into Rules.
- **A scheduled loop:** the runtime runs one turn by hand. Trading needs
  a tick loop (`trading run <name> --interval 60s`), with the stop
  checked every tick.
- **HAL roles:** HAL v2 records `separation: none` (one operator). A
  live mandate is exactly where a second human (attested separation)
  would add real safety. That is noted, not required.

## 10. Open decisions (James)

1. **Market and exchange:** crypto, stocks or forex, and which
   exchange's API. The adapters are per exchange.
2. **HAL mode for the first run:** per order, or a paper mandate. Paper
   comes first either way.
3. **ccemk2:** use it as the starting point (rebuilt on the Legacy), or
   keep it as reference only, like the-full-stack.
4. **Order of work:** the design recommends the story consumer first.
   It proves ai-gate, the runtime, HAL and Leighton with nothing at
   stake, and the trading consumer then inherits a runtime that is
   already tested.
