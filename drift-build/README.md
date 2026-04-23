# BUILD ROADMAP — `mini-drift`

> A ring-based study + build plan for cloning Drift v2's perp engine end-to-end,
> layer by layer, until the project is portfolio-grade and interview-ready for
> Drift / Jupiter Perps / top-tier perp DEX engineering roles.

**Companion to** `../learning/master_map.md` — the concept spiral.
**State tracker:** `../learning/learning_state.md`.
**Reference codebase:** `../protocol-v2-master/programs/drift/src/`.

---

## 0 · North Star

> **You will ship a working, tested, audited-style Solana perp DEX that a hiring
> manager at Drift or Jupiter can clone, run, and recognize — because every
> mechanism in it has a direct, documented counterpart in real Drift.**

One repo, one portfolio URL, one 60-second demo loop that shows: deposit →
place order → AMM fill → funding accrual → price move → liquidation →
insurance fund settlement.

---

## 1 · Why Rings (and how this differs from the learning spiral)

The **learning spiral** (`learning/master_map.md`) is 26 rings of pure
*understanding*: each ring adds one conceptual layer of Drift, moving from "what
is a perp" to "oracle guard rails." You don't write real Anchor code there.

This **build roadmap** shadows those 26 rings with a *production artifact* per
ring — small enough to ship in days, large enough to be real. Study a ring's
concept from Drift source, then *rebuild the same mechanism from scratch in
`mini-drift`*, with test vectors lifted from Drift's own unit tests so your
numbers provably match.

The neuron-connection goal: by R21 you should be able to point at any field on
`PerpMarket.amm` in Drift and explain both *what it does* and *why the
surrounding 20 fields exist to support it*. That only happens if you rebuild it.

```
╭────────────────────── PHASE 8 · ADVANCED ──────────────────────╮
│ ╭─────────────────── PHASE 7 · AMM HEALTH ─────────────────╮ │
│ │ ╭──────────────────── PHASE 6 · RISK ──────────────────╮ │ │
│ │ │ ╭──────────────── PHASE 5 · COSTS ─────────────────╮ │ │ │
│ │ │ │ ╭────────────── PHASE 4 · PRICING ─────────────╮ │ │ │ │
│ │ │ │ │ ╭──────────── PHASE 3 · TRADE ────────────╮ │ │ │ │ │
│ │ │ │ │ │ ╭────────── PHASE 2 · ENGINE ────────╮ │ │ │ │ │ │
│ │ │ │ │ │ │ ╭──────── PHASE 1 · WORLD ──────╮ │ │ │ │ │ │ │
│ │ │ │ │ │ │ │          R1  ·  R2             │ │ │ │ │ │ │ │
│ │ │ │ │ │ │ ╰────────────────────────────────╯ │ │ │ │ │ │ │
│ │ │ │ │ │ │           R3  ·  R4                │ │ │ │ │ │ │
│ │ │ │ │ │ ╰─────────────────────────────────────╯ │ │ │ │ │ │
│ │ │ │ │ │         R5 · R6 · R7 · R8    ◄── M1     │ │ │ │ │ │
│ │ │ │ │ ╰──────────────────────────────────────────╯ │ │ │ │ │
│ │ │ │ │       R9 · R10 · R11 · R12                   │ │ │ │ │
│ │ │ │ ╰───────────────────────────────────────────────╯ │ │ │ │
│ │ │ │           R13  ·  R14             ◄── M2         │ │ │ │
│ │ │ ╰───────────────────────────────────────────────────╯ │ │ │
│ │ │   R15 · R16 · R17 · R18    ◄── M3                   │ │ │
│ │ │   R19 · R20 · R21          ◄── M4                   │ │ │
│ │ ╰───────────────────────────────────────────────────────╯ │ │
│ │         R22  ·  R23                                       │ │
│ ╰───────────────────────────────────────────────────────────╯ │
│    R24  ·  R25  ·  R26           ◄── M5 (ship + polish)       │
╰───────────────────────────────────────────────────────────────╯

┌──── Parallel outer tracks · run alongside, not after ─────┐
│ Phase 9  · ADVERSARIAL  :  R27 → R28 → R29    ◄── M6-A    │
│ Phase 10 · REAL-WORLD   :  R30 (30d clock) · R31  ◄── M6-B│
│ Start points:  R27–R29  sequential after R21              │
│                R30      background, kicks off at R20      │
│                R31      any time from R15 onward          │
└────────────────────────────────────────────────────────────┘
```

Each ring's build layer is **self-contained** (compiles, tests pass, clippy
clean, commits atomic) and **composable** (the next ring plugs into it without
rewriting).

---

## 2 · The Four Tracks

Rings drive all four tracks in parallel — not all tracks activate at every ring.

| Track | What | Activates | Purpose |
|-------|------|-----------|---------|
| **A · `mini-drift` (Anchor program)** | The on-chain perp protocol, rebuilt ring by ring | R5 | Core hiring artifact. This is what you're really building. |
| **B · `drift-keeper` (TS bot)** | Off-chain bots run against **real Drift devnet** — filler, liquidator, funding cranker | R10 | Proves you can read Drift's live state + SDK. Hiring signal: "he's already in our codebase." |
| **C · `mini-drift-web` (React)** | Minimal UI — deposit / trade / position / funding / liq-price on your program | R8 | Makes it demoable in one link. Vercel URL on resume. |
| **D · Content & journal** | Per-ring docs in `docs/rings/`, 4 milestone blog posts, 3 Loom walkthroughs, final design journal, **daily Twitter build-in-public** | continuous | Converts tacit understanding → legible proof. Interview gold. Twitter is the Solana hiring network. |

---

## 3 · Milestones (acceptance = runnable demo)

Each milestone is a *demo moment* you could screen-share in a hiring loop.

| # | Rings | Working title | Demo you can run |
|---|-------|---------------|------------------|
| **M1** | R5–R8 | *"Can trade"* | Localnet: `anchor test` opens + closes a perp position against a stub AMM. |
| **M2** | R9–R14 | *"Real pricing"* | Devnet: position prices off Pyth SOL/USD, spreads widen on imbalance, funding cranks hourly. |
| **M3** | R15–R18 | *"Full risk"* | Devnet: deposit USDC → open 10x SOL long → UI shows initial margin, maintenance margin, liq price, unrealized PnL live. |
| **M4** | R19–R21 | *"Can liquidate"* | Devnet: stress-move Pyth push oracle, your liquidator bot atomically takes the underwater position, insurance fund absorbs residual. |
| **M5** | R22–R26 | *"Ship"* | Vercel URL + README + 3 blogs + 3 Looms + a tagged `v1.0.0` release. |
| **M6** | R27–R31 | *"Hardened + contributed"* | Threat model + fuzz report + audit react-blog (#4) + 30d live keeper dashboard + merged PR to `drift-labs/keeper-bots-v2` (or disclosed bounty). |

---

## 4 · Build Principles (non-negotiable quality bar)

Every ring's deliverable must pass these before you move on. This is the
"hire-at-Drift" bar, not the "works on my machine" bar.

1. **Compiles clean.** `anchor build` → zero warnings. `cargo clippy -- -D warnings` clean.
2. **Tests pass.** Every math function has `#[cfg(test)] mod tests`. Every instruction has an `anchor test` case for happy path + at least one failure mode.
3. **Test vectors lifted from Drift.** For any math-heavy ring (R7, R9, R12, R14, R16, R17–18, R20), your tests must include numeric cases copied from Drift's own `#[cfg(test)]` blocks so your output provably matches. Cite the Drift file + line in a comment.
4. **No silent overflow.** Zero bare `+` `-` `*` `/` on money math — only `checked_*` / `safe_add` / `safe_mul`. Zero `.unwrap()` outside `#[cfg(test)]`.
5. **Custom error enum.** `MiniDriftError` grows each ring. Every `?` should resolve to a specific variant, never a generic `ProgramError::Custom(0)`.
6. **Events for every state change.** Mirror Drift's `OrderActionRecord` / `OrderRecord` — emit on open, fill, funding update, liquidation, settle.
7. **Precision constants centralized.** `math/constants.rs` is the only place `PRICE_PRECISION`, `BASE_PRECISION`, `QUOTE_PRECISION`, `FUNDING_RATE_PRECISION`, `MARGIN_PRECISION` live. No magic numbers in business logic.
8. **Account size documented.** Every `#[account]` struct has a comment with byte-by-byte size derivation and `LEN` constant.
9. **Atomic commits.** One logical change per commit. Message format: `ring-XX: <verb> <thing>`. No "wip" commits on `main`.
10. **Docs per ring.** Each ring closes with `docs/rings/RNN-<slug>.md` in *your own words* — Drift's mechanism, why it exists, your implementation choices, where you diverged and why.

---

## 5 · Proposed Monorepo Layout

```
mini-drift/                               ← single repo, single portfolio URL
├── README.md                             ← user-facing; 60s what+why+demo-gif
├── Anchor.toml
├── Cargo.toml                            ← workspace root
├── programs/
│   └── mini-drift/
│       └── src/
│           ├── lib.rs                    ← #[program] entrypoints
│           ├── error.rs                  ← MiniDriftError enum (grows /ring)
│           ├── state/                    ← account definitions
│           │   ├── state.rs              ← global State singleton
│           │   ├── perp_market.rs        ← PerpMarket + AMM struct
│           │   ├── spot_market.rs        ← SpotMarket (just USDC initially)
│           │   ├── user.rs               ← User, PerpPosition, SpotPosition, Order
│           │   ├── oracle.rs             ← OraclePriceData, OracleSource
│           │   └── events.rs
│           ├── math/                     ← stateless pure functions, fully tested
│           │   ├── constants.rs
│           │   ├── safe_math.rs
│           │   ├── casting.rs
│           │   ├── amm.rs
│           │   ├── amm_spread.rs
│           │   ├── auction.rs
│           │   ├── funding.rs
│           │   ├── fees.rs
│           │   ├── margin.rs
│           │   ├── pnl.rs
│           │   ├── position.rs
│           │   ├── liquidation.rs
│           │   └── oracle.rs
│           ├── controller/               ← state-mutating orchestration
│           │   ├── orders.rs
│           │   ├── position.rs
│           │   ├── amm.rs
│           │   ├── funding.rs
│           │   ├── pnl.rs
│           │   ├── liquidation.rs
│           │   ├── spot_balance.rs
│           │   └── insurance.rs
│           ├── instructions/             ← #[derive(Accounts)] + handlers
│           │   ├── admin.rs
│           │   ├── user.rs
│           │   └── keeper.rs
│           └── validation/
├── tests/                                ← Anchor mocha tests
│   ├── 01-initialize.ts
│   ├── 02-place-order.ts
│   └── ...
├── app/                                  ← mini-drift-web (Next.js + Vercel)
├── keeper/                               ← drift-keeper bot (TS, runs vs REAL Drift devnet)
├── sdk/                                  ← TS client for your program (auto-generated + hand-rolled helpers)
└── docs/
    ├── rings/                            ← RNN-<slug>.md per ring (track D)
    ├── diagrams/                         ← architecture, data flow, liq cascade
    ├── blog/                             ← 3 milestone blog posts
    └── design-journal.md                 ← long-form decisions log
```

One repo = one resume bullet. Do not split into 3 repos. The monorepo *is* the
portfolio.

---

## 6 · Ring-by-Ring Plan

**Legend per ring:**
- **Concept** — pointer to `learning/master_map.md` ring
- **Study** — exact Drift source files to dissect (relative to `protocol-v2-master/`)
- **Neurons** — which prior rings this wires into
- **Build** — the artifact to ship
- **Tests** — what must pass (with Drift-sourced vectors where marked 🎯)
- **Est.** — full-time days
- **Demo** — one line you can show a recruiter after shipping this ring

---

### PHASE 1 — THE WORLD (R1–R2)
*No Anchor code yet. Set up the repo, write the vision, own the language.*

#### Ring 1 — *What is a perp* [`learning R1`]
- **Concept:** trading, futures, perpetuals, the exchange
- **Study:** none in code yet; Drift docs at https://drift-labs.github.io/v2-teacher/ (skim intro)
- **Neurons:** none (base layer)
- **Build:**
  - `git init` the `mini-drift` repo
  - Root `README.md` skeleton with vision, monorepo layout, empty Milestone checkboxes
  - `docs/rings/R01-what-is-a-perp.md` — your own 800-word explainer: trading → futures → perp → exchange. No jargon.
- **Tests:** none
- **Est.:** 0.5 day
- **Demo:** "Here's the repo. Here's how I explain a perp in plain English."

#### Ring 2 — *Your first trade* [`learning R2`]
- **Concept:** long/short/PnL/collateral/margin/leverage
- **Study:** none in code
- **Neurons:** R1
- **Build:** `docs/rings/R02-long-short-leverage.md` — worked numeric scenarios for long $200 10 SOL @ $100, short 5 SOL @ $200, 5x vs 20x leverage PnL tables.
- **Tests:** Your worked arithmetic must match a spreadsheet (track D: commit the `.xlsx` or a Python notebook alongside).
- **Est.:** 0.5 day
- **Demo:** "Here are the 4 invariants of PnL arithmetic, with numbers."

---

### PHASE 2 — THE ENGINE (R3–R4)
*First code lands: math primitives + account scaffolding. No trading yet.*

#### Ring 3 — *The robot trader (AMM intuition)* [`learning R3`]
- **Concept:** AMM, x\*y=k, peg, price from reserves
- **Study:**
  - `programs/drift/src/math/amm.rs` — just skim `calculate_price`, `calculate_quote_asset_amount_swapped`
  - `programs/drift/src/math/constants.rs` — PRICE_PRECISION = 1e6, BASE_PRECISION = 1e9, AMM_RESERVE_PRECISION = 1e9
- **Neurons:** R2 (counterparty problem motivates AMM)
- **Build:**
  - `Cargo.toml` workspace + empty `programs/mini-drift` crate (don't `anchor init` yet; Ring 4 does that)
  - `docs/rings/R03-vamm.md` with a spreadsheet: trace 5 sequential trades through an `x*y=k` vAMM and show how price drifts
- **Tests:** none (spreadsheet only)
- **Est.:** 1 day
- **Demo:** Walk a recruiter through the spreadsheet: "Trade 1 moved price 0.4%, trade 2 moved it 0.6%, here's why."

#### Ring 4 — *Going on-chain* [`learning R4`]
- **Concept:** accounts, four pillars, fixed-point, safe math
- **Study:**
  - `programs/drift/src/math/constants.rs` (all precision constants)
  - `programs/drift/src/math/safe_math.rs` (the SafeMath trait)
  - `programs/drift/src/math/casting.rs`
- **Neurons:** R3 (precision lets x\*y=k work in integers)
- **Build:** 🚀 **First code lands.**
  - `anchor init mini-drift` (use 0.31.x)
  - `programs/mini-drift/src/math/constants.rs` — copy Drift's precision constants + cite source
  - `programs/mini-drift/src/math/safe_math.rs` — implement `SafeMath` trait for `u64`/`i64`/`u128`/`i128` with `safe_add`/`safe_sub`/`safe_mul`/`safe_div` → `Result<T, MiniDriftError::MathError>`
  - `programs/mini-drift/src/math/casting.rs` — `cast_to_u64` / `cast_to_i128` helpers
  - `programs/mini-drift/src/error.rs` — `MiniDriftError::MathError`, `InvalidCast` (first 2 variants)
- **Tests:** 🎯 unit tests for overflow/underflow/cast-boundary (one per method); vectors from `math/safe_math.rs` tests in Drift if present.
- **Est.:** 1 day
- **Demo:** "Here's `cargo test` — every checked op has overflow + underflow + boundary cases."

---

### PHASE 3 — THE TRADE (R5–R8) — **MILESTONE 1**
*First on-chain lifecycle: initialize → deposit → place order → fill → close.*

#### Ring 5 — *Your position in code* [`learning R5`]
- **Concept:** `PerpPosition` struct, signed base/quote, direction
- **Study:** `programs/drift/src/state/user.rs` — the `PerpPosition` struct definition + its `is_open_position`, `is_available` helpers
- **Neurons:** R4 (precision), R3 (base/quote from x\*y=k)
- **Build:**
  - `state/user.rs` — `User` account with `perp_positions: [PerpPosition; 8]` (Drift has more; 8 is plenty for mini)
  - `PerpPosition` with: `base_asset_amount: i64`, `quote_asset_amount: i64`, `quote_entry_amount: i64`, `quote_break_even_amount: i64`, `last_cumulative_funding_rate: i64`, `market_index: u16`
  - `instructions/user.rs::initialize_user` — creates `User` PDA seeded by `[b"user", authority, sub_account_id]`
- **Tests:** `tests/01-initialize-user.ts` — init + state checks for empty positions array
- **Est.:** 1 day
- **Demo:** "One tx creates a user PDA with room for 8 positions."

#### Ring 6 — *Placing your bet* [`learning R6`]
- **Concept:** `Order` struct, `OrderType`, `OrderParams`, lifecycle, reduce/post/IOC
- **Study:**
  - `programs/drift/src/state/user.rs` — `Order` struct + `OrderStatus`
  - `programs/drift/src/state/order_params.rs` — `OrderParams`, `OrderTriggerCondition`, `OrderParamsBitFlag`
- **Neurons:** R5 (orders produce positions)
- **Build:**
  - `state/user.rs::Order` with: `order_id`, `market_index`, `price`, `base_asset_amount`, `direction: PositionDirection`, `order_type: OrderType`, `status: OrderStatus`, `auction_duration`, `auction_start_price`, `auction_end_price`, `max_ts`, `bit_flags: u8` (reduce/post/IOC packed)
  - Reduce `OrderType` to **Market** and **Limit** only for now (Trigger in R20, Oracle skip)
  - `orders: [Order; 16]` slot on `User`
  - `instructions/user.rs::place_perp_order(params: OrderParams)` — validates, inserts into first available slot, emits `OrderRecord` event
- **Tests:** happy path + "no free slot" failure + reduce-only on empty position = reject
- **Est.:** 2 days
- **Demo:** "Place an order → event emits → next slot filled. Reduce-only correctly rejects."

#### Ring 7 — *Getting filled* [`learning R7`]
- **Concept:** Dutch auction, keepers, fill flow, fees-on-fill
- **Study:**
  - `programs/drift/src/math/auction.rs` — `calculate_auction_price`
  - `programs/drift/src/math/fees.rs` — `FeeTier`, taker fee calculation
  - `programs/drift/src/controller/orders.rs::fill_perp_order` — orchestration sketch only; you'll stub AMM until R9
- **Neurons:** R6 (orders), R3 (AMM as stub counterparty)
- **Build:**
  - `math/auction.rs::calculate_auction_price(order, slot) -> i64` — linear interp between start/end over `auction_duration` slots
  - `math/fees.rs` — fixed taker fee (10 bps) for now, refine in R13
  - `controller/orders.rs::fulfill_perp_order` — stub AMM fill: use auction price, mutate position.
  - `instructions/keeper.rs::fill_perp_order` — permissionless filler pubkey receives filler reward
- **Tests:** 🎯 `auction.rs` tests: copy Drift's auction vectors from `math/auction.rs::tests`. Assert your output matches to the last integer.
- **Est.:** 3 days
- **Demo:** "Any wallet can fill — here's a second tx from a different signer filling the order; auction price is deterministic from slot."

#### Ring 8 — *Position updates* [`learning R8`]   🏁 **M1**
- **Concept:** PositionDelta, open/increase/decrease/close
- **Study:** `programs/drift/src/controller/position.rs::update_position_and_market`
- **Neurons:** R5 (position), R7 (fills trigger updates)
- **Build:**
  - `math/position.rs::calculate_position_delta` — pure fn: given fill size/price/direction + existing position, returns new `(base, quote_entry, quote_break_even, realized_pnl)`
  - `controller/position.rs::update_position_and_market` — applies delta; handles 4 cases (open / increase / decrease / close-and-flip)
  - Close-and-flip: opening opposite side beyond current size realizes PnL on the closed portion, opens residual on the new side
- **Tests:** 🎯 four-case vector matrix lifted from `controller/position.rs::tests`; each of open/increase/decrease/flip has an exact Drift-match test
- **Est.:** 3 days
- **🏁 Demo (M1):** Vercel app (stubbed): "deposit 1000 USDC → long 5 SOL @ $180 → close → see realized PnL of $X. Numbers match my spreadsheet from R3."
- **Track C kicks off here.** Scaffold Next.js app with a single page: one wallet connect, one "place order" button, one position readout.

---

### PHASE 4 — THE PRICING ENGINE (R9–R12)
*Real AMM, real oracle, real spreads. No more stub.*

#### Ring 9 — *AMM internals* [`learning R9`]
- **Concept:** reserves, sqrt_k, base_asset_amount_with_amm, OI, bounds
- **Study:**
  - `programs/drift/src/state/perp_market.rs` — the `AMM` struct (this is the big one — spend 2 days on it)
  - `programs/drift/src/math/amm.rs` — `calculate_swap_output`, `calculate_quote_asset_swapped`, `update_amm_reserves`
  - `programs/drift/src/math/cp_curve.rs` — constant product
- **Neurons:** R3 (formula), R8 (fills mutate reserves)
- **Build:**
  - `state/perp_market.rs::PerpMarket` with embedded `AMM` struct: `base_asset_reserve`, `quote_asset_reserve`, `sqrt_k`, `peg_multiplier`, `base_asset_amount_with_amm`, `base_asset_amount_long`, `base_asset_amount_short`, `min_base_asset_reserve`, `max_base_asset_reserve`, `concentration_coef`
  - `math/amm.rs::swap_base_asset` — pure, checked, with concentration bounds
  - `instructions/admin.rs::initialize_perp_market` — seeds AMM with initial `sqrt_k` and `peg_multiplier`
  - Replace R7's stub fill with a real AMM swap
- **Tests:** 🎯 heavy vector matrix from `math/amm.rs::tests` — covers swap, bounds hit, concentration enforcement, min/max reserve guards. Every case a copy-paste match.
- **Est.:** 5 days (biggest ring so far)
- **Demo:** "Here's a fill through a real vAMM — mark price before/after, reserves before/after, match my spreadsheet."

#### Ring 10 — *Truth from outside (oracles)* [`learning R10`]
- **Concept:** OraclePriceData, OracleSource, TWAP, staleness
- **Study:**
  - `programs/drift/src/state/oracle.rs` — `OraclePriceData`, `HistoricalOracleData`, `OracleSource` enum
  - `programs/drift/src/state/oracle_map.rs`
  - `programs/drift/src/math/oracle.rs` — `oracle_validity` checks
- **Neurons:** R9 (AMM needs peg anchor)
- **Build:**
  - `state/oracle.rs::OraclePriceData { price, confidence, delay, has_sufficient_number_of_data_points }`
  - Pyth integration via `pyth-solana-receiver-sdk` — load `PriceUpdateV2` account, extract price + confidence
  - `math/oracle.rs::is_oracle_valid` — staleness + confidence-interval + divergence-vs-mark checks
  - Mark the `oracle` field on `PerpMarket` as `Pubkey` + `OracleSource::Pyth`
- **Tests:** devnet integration test: load a real Pyth SOL/USD account, assert valid, assert price in range. Also: mock an expired/high-confidence oracle → assert rejection.
- **Est.:** 4 days
- **🚀 Track B kicks off here.** Create `keeper/` — a TS bot that subscribes to **real Drift** devnet `PerpMarket.oracle` and logs current oracle price vs mark price every block. **Read-only; against real Drift.** This is your "I've been in the codebase" signal.
- **Demo:** "Tail this keeper log — that's real Drift devnet SOL/USD mark vs oracle, updated live. My program reads Pyth the same way."

#### Ring 11 — *The AMM's edge* [`learning R11`]
- **Concept:** bid/ask spread, base/max spread, long/short asymmetry
- **Study:**
  - `programs/drift/src/math/amm_spread.rs` — `calculate_spread_inventory`, `calculate_spread_leverage`
  - `programs/drift/src/state/perp_market.rs` — `AMM.base_spread`, `max_spread`, `long_spread`, `short_spread`
- **Neurons:** R9 (AMM), R10 (oracle feeds spread model)
- **Build:**
  - `math/amm_spread.rs::calculate_spread` — pure fn, static version only (base_spread + max_spread clamps)
  - `AMM` struct gains spread fields
  - Fill flow quotes `bid_price = reserve_price - short_spread/2` and `ask = + long_spread/2`
- **Tests:** 🎯 static spread vectors from `math/amm_spread.rs::tests`
- **Est.:** 2 days
- **Demo:** "Orders now quote with a spread. Here's bid/ask rendered live in the UI."

#### Ring 12 — *Living spreads* [`learning R12`]
- **Concept:** dynamic spread from inventory, volatility, revenue retreat
- **Study:** `programs/drift/src/math/amm_spread.rs` — full file: `calculate_spread_inventory_scale`, `calculate_spread_revenue_retreat`, `calculate_mark_std`
- **Neurons:** R11 (static spread), R9 (inventory), R10 (volatility via oracle_std)
- **Build:**
  - Extend `math/amm_spread.rs` to full dynamic model: inventory scale + volatility + revenue retreat
  - Revenue retreat needs `net_revenue_since_last_funding` on AMM — add the field, forward-declare (gets fed in R13/R14)
- **Tests:** 🎯 full dynamic spread vectors from Drift. These are notorious edge cases — find at least 10 and match all.
- **Est.:** 4 days
- **Demo:** "Pile in $100k of longs → ask widens, bid tightens. Demo it on localnet."

---

### PHASE 5 — ONGOING COSTS (R13–R14) — **MILESTONE 2**

#### Ring 13 — *Where money flows* [`learning R13`]
- **Concept:** fee accumulation, fee pool, total_fee_minus_distributions, fee tiers
- **Study:**
  - `programs/drift/src/state/state.rs` — `FeeStructure`, `FeeTier`
  - `programs/drift/src/state/perp_market.rs` — `PoolBalance`, `total_fee`, `total_fee_minus_distributions`, `fee_pool`
  - `programs/drift/src/math/fees.rs`
- **Neurons:** R7 (fees charged), R11-12 (revenue retreat reads these)
- **Build:**
  - `state/state.rs::State` global singleton with `FeeStructure` (4 tiers by volume)
  - `PoolBalance` struct on `PerpMarket` — `scaled_balance`, `market_index`
  - Fill path credits `total_fee`, `total_fee_minus_distributions`, and the fee pool
  - Volume-based tier: track `UserStats.taker_volume_30d` (simple EMA)
- **Tests:** fills across 4 tier boundaries; fee invariant `total_fee == sum(filler_reward + fee_pool_delta + ...)`.
- **Est.:** 3 days
- **Demo:** "Trader hits $10M volume → fee tier drops. Show the 30d EMA."

#### Ring 14 — *Staying anchored (funding)* [`learning R14`]   🏁 **M2**
- **Concept:** funding rate, mark vs oracle TWAP, asymmetry, capping, cumulative tracking
- **Study:**
  - `programs/drift/src/math/funding.rs` — `calculate_funding_rate`, `calculate_funding_rate_long_short`
  - `programs/drift/src/controller/funding.rs::update_funding_rate`
- **Neurons:** R10 (oracle TWAP), R9 (mark TWAP from reserves), R13 (capped by fee pool)
- **Build:**
  - `math/funding.rs::calculate_funding_rate` + asymmetric long/short split + capping at `total_fee_minus_distributions / OI`
  - `controller/funding.rs::update_funding_rate` — crankable, rate-limited to once per hour
  - `instructions/keeper.rs::update_funding_rate` — permissionless
  - `PerpPosition.last_cumulative_funding_rate` — settled lazily on next position touch (open/fill/close/settle)
  - `controller/funding.rs::settle_funding_payment` — per-position
- **Tests:** 🎯 funding vectors from `math/funding.rs::tests` — mark above oracle (longs pay shorts), mark below (shorts pay longs), capped case, zero-OI case.
- **Est.:** 4 days
- **🚀 Track B milestone:** your keeper bot now has a second command: `funding-crank` — watches your devnet deployment, cranks funding once per hour.
- **🏁 Demo (M2):** "Devnet recording: I open long, time-warp 2 hours, funding has accrued, next close realizes the funding payment. Numbers match my spreadsheet."
- **📝 Blog #1 publishes here:** *"Rebuilding Drift's funding rate from first principles."*

---

### PHASE 6 — RISK (R15–R21) — **MILESTONES 3 & 4**
*The largest phase. Everything you've built exists to enable safe leverage.*

#### Ring 15 — *Your collateral in code* [`learning R15`]
- **Concept:** SpotPosition, scaled_balance, get_token_value
- **Study:**
  - `programs/drift/src/state/user.rs::SpotPosition`
  - `programs/drift/src/state/spot_market.rs` (minimal — only what's needed for USDC deposit)
  - `programs/drift/src/math/spot_balance.rs::get_token_amount`
  - `programs/drift/src/controller/spot_balance.rs::update_spot_balances`
- **Neurons:** R4 (accounts)
- **Build:**
  - `state/spot_market.rs::SpotMarket` — just enough for USDC: `mint`, `vault`, `cumulative_deposit_interest`, `decimals`
  - `state/user.rs::SpotPosition { market_index, scaled_balance, balance_type: SpotBalanceType }`
  - `user.spot_positions: [SpotPosition; 8]`, index 0 = USDC always
  - `instructions/user.rs::deposit` / `withdraw` — SPL token CPI to/from market vault
  - `controller/spot_balance.rs::update_spot_balances` — the single pipe every money flow uses
- **Tests:** anchor test: deposit 1000 USDC → `scaled_balance` correct → withdraw 500 → balance + vault match.
- **Est.:** 3 days
- **Demo:** UI: "Deposit / withdraw USDC collateral."

#### Ring 16 — *PnL calculation* [`learning R16`]
- **Concept:** exit - entry, AMM-based and oracle-based valuation
- **Study:** `programs/drift/src/math/pnl.rs`, `math/position.rs::calculate_base_asset_value`
- **Neurons:** R9 (AMM valuation), R10 (oracle valuation), R15 (PnL lands in spot balance)
- **Build:**
  - `math/pnl.rs::calculate_unrealized_pnl` — AMM mark version + oracle version
  - Strict valuation: `min(mark, oracle)` for assets, `max(mark, oracle)` for liabilities
- **Tests:** 🎯 PnL vectors from `math/pnl.rs::tests`
- **Est.:** 2 days
- **Demo:** UI shows live unrealized PnL ticking with oracle.

#### Ring 17 — *Per-position margin* [`learning R17`]
- **Concept:** initial / maintenance, IMF, size premium
- **Study:** `programs/drift/src/math/margin.rs` — focus on `calculate_size_premium_liability_weight`, `calculate_margin_requirement_and_total_collateral`
- **Neurons:** R16 (position valuation), R9 (OI for IMF)
- **Build:**
  - `math/margin.rs::calculate_margin_requirement` — per-position, handling `MarginRequirementType::{Initial, Fill, Maintenance}`
  - `PerpMarket.margin_ratio_initial`, `margin_ratio_maintenance`, `imf_factor`
  - Size premium: liability weight grows with position size (the quadratic IMF bump)
- **Tests:** 🎯 four-position portfolio vectors across three margin types; Drift has great coverage for IMF scaling.
- **Est.:** 4 days
- **Demo:** UI: "At 5 SOL you need 10% margin. At 50 SOL the IMF kicks in — here's the number."

#### Ring 18 — *Portfolio margin* [`learning R18`]   🏁 **M3**
- **Concept:** aggregation, unrealized PnL weights, MarginCalculation struct
- **Study:**
  - `programs/drift/src/math/margin.rs` — `calculate_margin_requirement_and_total_collateral`
  - `programs/drift/src/state/margin_calculation.rs`
- **Neurons:** R15 (spot collateral sum), R16 (PnL), R17 (per-position), R14 (unsettled funding in PnL)
- **Build:**
  - `state/margin_calculation.rs::MarginCalculation` — struct accumulating across all perp + spot positions
  - `math/margin.rs::calculate_margin_requirement_and_total_collateral` — full aggregation
  - Enforce `initial` requirement on every order placement, `maintenance` on every settle/liq check
- **Tests:** multi-position scenarios: 2 perp positions + 1 spot borrow + unsettled funding; total collateral and requirement both line up.
- **Est.:** 4 days
- **🏁 Demo (M3):** UI shows live: total collateral, initial margin used, maintenance margin, **liquidation price**. Move the oracle → liq price moves.
- **📝 Blog #2 publishes here:** *"Portfolio margin on Solana: how Drift turns 200 oracle ticks into one number."*

#### Ring 19 — *PnL settlement* [`learning R19`]
- **Concept:** PnL pool, settling +/-, imbalance, divergence checks
- **Study:** `programs/drift/src/controller/pnl.rs::settle_pnl`, `math/pnl.rs::calculate_max_pnl_pool_excess`
- **Neurons:** R13 (fee pool funds PnL pool), R15-18 (margin check gates settle)
- **Build:**
  - `PerpMarket.pnl_pool: PoolBalance`
  - `instructions/user.rs::settle_pnl` — user-triggered on any position
  - Price divergence check: if mark diverges too far from oracle, refuse settle
  - PnL imbalance: if pool lacks capacity, settle partially
- **Tests:** positive settle, negative settle, pool-exhaustion scenario, divergence rejection.
- **Est.:** 3 days
- **Demo:** "Close a winning position → PnL moves from pool into spot balance in one tx."

#### Ring 20 — *Liquidation* [`learning R20`]
- **Concept:** trigger, partial liq, fees, status, margin_freed tracking
- **Study:**
  - `programs/drift/src/controller/liquidation.rs::liquidate_perp`
  - `programs/drift/src/math/liquidation.rs`
  - `programs/drift/src/state/user.rs` — `UserStatus` bitflags
- **Neurons:** R17-18 (trigger = maintenance breach), R15 (liquidator receives spot credit), R13 (fee split)
- **Build:**
  - `state/user.rs::User.status` bitflags — `BeingLiquidated`, `Bankrupt`
  - `math/liquidation.rs::calculate_liquidation_multiplier`, `calculate_base_asset_amount_to_cover_margin_shortage`
  - `controller/liquidation.rs::liquidate_perp` — partial, sized to `initial_pct_to_liquidate`
  - Liquidator fee + insurance fund fee split
  - `OrderType::TriggerMarket` / `TriggerLimit` also added here (for users to pre-place stops)
- **Tests:** 🎯 liquidation vectors from `math/liquidation.rs::tests` — one long, one short, one across multiple markets.
- **Est.:** 6 days
- **🚀 Track B milestone:** liquidator bot — polls all users, finds under-margin, sends atomic liq tx. Run it against **your** mini-drift, not real Drift.
- **Demo:** "Move Pyth oracle → my liquidator bot takes the position within 2 blocks → UI reflects the change."

#### Ring 21 — *Bankruptcy & insurance* [`learning R21`]   🏁 **M4**
- **Concept:** negative equity, social loss, insurance fund, contract tier
- **Study:**
  - `programs/drift/src/math/bankruptcy.rs`
  - `programs/drift/src/controller/insurance.rs`
  - `programs/drift/src/state/perp_market.rs::InsuranceClaim`, `ContractTier`
- **Neurons:** R20 (bankruptcy is liq without enough collateral), R13 (fees seed IF)
- **Build:**
  - `state/perp_market.rs::InsuranceClaim` + `ContractTier` enum (A/B/Speculative — skip Highly/Isolated)
  - Insurance fund spot vault (treat as `SpotMarket.insurance_fund_vault`)
  - `controller/insurance.rs::resolve_perp_bankruptcy` — IF pays residual; if IF insufficient, socialize loss across winning positions in same market
- **Tests:** residual covered by IF, residual exceeding IF triggers socialized loss.
- **Est.:** 5 days
- **🏁 Demo (M4):** "End-to-end devnet stress run: hour-long recording of 5 traders, oracle swings, 3 liquidations, 1 bankruptcy covered by IF, 1 partial socialization."
- **📝 Blog #3 publishes here:** *"Why insurance funds exist: anatomy of a Solana perp liquidation cascade."*

---

### PHASE 7 — AMM HEALTH (R22–R23)

#### Ring 22 — *AMM maintenance (repeg + K + JIT)* [`learning R22`]
- **Concept:** repeg, formulaic K, AMM JIT
- **Study:**
  - `programs/drift/src/math/repeg.rs`
  - `programs/drift/src/math/cp_curve.rs::update_k`
  - `programs/drift/src/math/amm_jit.rs`
  - `programs/drift/src/controller/repeg.rs`
- **Neurons:** R9 (AMM internals), R10 (oracle = repeg target), R13 (fee pool funds repeg)
- **Build:**
  - `math/repeg.rs::calculate_repeg_cost` — pure
  - `controller/repeg.rs::repeg_curve` — admin + auto variants, budgeted by fee pool
  - `math/cp_curve.rs::update_k` — auto-scale with `curve_update_intensity`
  - AMM JIT (skip or simplify — a fixed `amm_jit_intensity` participation)
- **Tests:** 🎯 repeg cost vectors; K update vectors.
- **Est.:** 4 days
- **Demo:** "Oracle drifts 2% → auto-repeg within budget. Here's the before/after."

#### Ring 23 — *Matching engine* [`learning R23`]
- **Concept:** fulfillment methods, maker/taker matching
- **Study:** `programs/drift/src/math/matching.rs`, `math/fulfillment.rs`, `controller/orders.rs::fulfill_perp_order` (full)
- **Neurons:** R6-7 (orders + fills), R11-12 (spreads)
- **Build:**
  - `math/matching.rs::do_orders_cross`, `calculate_fill_for_matched_orders`
  - Fill path tries **Match first**, falls through to **AMM**
  - Maker rebate / filler multiplier on matched fills
- **Tests:** maker sitting at $100, taker market order → match at $100 with maker rebate; no match → AMM fallback.
- **Est.:** 4 days
- **Demo:** "Two wallets — one posts limit, one markets in. Filled against each other at 0 AMM fee."

---

### PHASE 8 — ADVANCED (R24–R26) — **MILESTONE 5**
*You ship. Pick the 3–5 features that best showcase range, skip the rest.*

#### Ring 24 — *LP system* [`learning R24`]  *(optional — skip if time-constrained)*
- **Concept:** LP shares, settlement, rebase
- **Build:** BAL share token; `add_liquidity`, `remove_liquidity`, `settle_lp` controller. Minimal version.
- **Est.:** 5 days

#### Ring 25 — *Market lifecycle* [`learning R25`]
- **Concept:** MarketStatus state machine, paused ops, expiry
- **Build:**
  - `state/perp_market.rs::MarketStatus` — `Initialized → Active → ReduceOnly → Settlement → Delisted`
  - `PausedOperations` bitflags
  - Admin can pause a market; reduce-only blocks position-increasing orders
- **Est.:** 2 days
- **Demo:** "Pause a market → existing holders can only reduce. Critical for real operations."

#### Ring 26 — *Polish, infra, guard rails* [`learning R26`]   🏁 **M5**
- **Concept:** oracle guard rails, events, (optionally high-leverage mode)
- **Build:**
  - `math/oracle.rs::OracleValidity` + `OracleGuardRails` in global `State`
  - Complete event coverage audit — every state change emits an event
  - `PerpMarketMap` / `SpotMarketMap` / `OracleMap` for efficient remaining-accounts loading
  - CI: GitHub Actions running `anchor build` + `anchor test` + `cargo clippy -- -D warnings`
  - **Deploy to devnet with public program ID in README**
  - **Vercel deployment of `app/`**
  - `docs/design-journal.md` — final long-form writeup
  - 🎬 **Loom #1**: 5-min codebase tour. **#2**: 5-min liquidation cascade. **#3**: 5-min hiring pitch — "what I learned, what I'd do differently, what I'd build next."
- **Est.:** 1 week
- **🏁 Demo (M5):** Public Vercel URL + public program ID + tagged `v1.0.0` release + README with demo GIF + 3 blog posts + 3 Looms.

---

### PHASE 9 — ADVERSARIAL (R27–R29) — **MILESTONE 6-A**
*Parallel track. Sequential within phase. Runs after R21 (once the full attack surface exists). Total ~2 weeks.*

#### Ring 27 — *Threat model*
- **Concept:** enumerate the attack surface; learn to see your own code as an attacker would
- **Study:**
  - `protocol-v2-master/bug-bounty/README.md` — Drift's in-scope table + severity bands tell you what Drift itself considers critical
  - Public Drift audit reports (OtterSec / Neodyme / Zellic on GitHub)
  - Post-mortems: Mango exploit (Oct 2022), Aries Markets (2023), any perp-specific incident write-up
- **Neurons:** R9 (AMM surface), R14 (funding crank window), R17–18 (margin), R20–21 (liquidation + IF)
- **Build:** `docs/security/threat-model.md` with **15+ concrete attack scenarios** grouped by category:
  - **Oracle:** stale price, confidence-interval spoofing, cross-venue manipulation, Pyth publisher collusion
  - **Timing / MEV:** sandwich on funding crank, back-run liquidation, JIT front-run, priority-fee DoS on keepers
  - **Rounding / precision:** balance inflation via repeated dust deposits, off-by-one in IMF, rounding-direction exploit in PnL settlement
  - **Solvency:** IF drain via repeated bankruptcies, socialized-loss gaming, arb between mark and oracle at settle
  - **Logic:** reduce-only bypass, post-only slide-through, reentrancy via SPL CPI, privilege escalation on admin ix
  - For each: *attack · prerequisites · impact · mitigation in mini-drift*
- **Tests:** for each "mitigation in mini-drift," add a regression test asserting the attack fails
- **Est.:** 3 days
- **Demo:** "Here's a 20-page threat model. Pick any attack number 1–15 and I'll walk the exploit + the mitigation."

#### Ring 28 — *Fuzzing*
- **Concept:** surface bugs random inputs find that humans miss
- **Study:** `cargo-fuzz` book, `proptest` / `arbitrary` crates, the `#[cfg(test)]` blocks in Drift math modules (you've been lifting these — now you attack them)
- **Neurons:** R4 (safe math — fuzzing finds where the safety is imagined), R17–18 (margin invariants), R20 (liquidation multiplier)
- **Build:**
  - `programs/mini-drift/fuzz/` workspace with **4 fuzz targets**:
    - `fuzz_margin` — random portfolio → assert `total_collateral ≥ 0` + `margin_req` monotonic-in-size
    - `fuzz_liquidation` — random under-margined user → assert post-liq user is liquidated-to-zero OR above maintenance (never stuck between)
    - `fuzz_amm_swap` — random reserves + swap size → assert `k` invariant + concentration bounds never violated
    - `fuzz_funding` — random mark/oracle divergence → assert capped-rate invariant
  - Run each 24h on a cheap VPS. Document every crash.
  - Convert each crash into a permanent regression test with a minimal reproducer.
- **Tests:** all prior + new fuzz-derived vectors
- **Est.:** 5 days (48h compute runs in background — you don't block on it)
- **Demo:** "I ran 4 fuzzers 24h each. Found 3 edge cases — here are the minimal reproducers, here are the fixes, here are the regression tests that now pin them."

#### Ring 29 — *Audit-driven hardening*   🏁 **M6-A**
- **Concept:** read professional audit analysis cover-to-cover; apply its lessons to your own code
- **Study:**
  - Pick **one** public Drift audit report (OtterSec has several on `github.com/otter-sec`). Read end-to-end.
  - Read 2 other perp audits for calibration (Perpetual Protocol v2 audits, dYdX audits — all public)
- **Neurons:** R27 (your own threat model is about to look naive next to professionals'), R28 (fuzzing vs human review — complementary)
- **Build:**
  - `docs/security/audit-replication.md` — pick **3 findings applicable to your codebase**. For each: *auditor's words · my words · my patch*.
  - Update `docs/security/threat-model.md` with any attack categories the audit surfaced that you missed (this is usually humbling and valuable)
  - 📝 **Blog #4 publishes here:** *"What I learned reading a real perp DEX audit report."*
- **Tests:** regression test per patched finding
- **Est.:** 5 days
- **🏁 Demo (M6-A):** "I read [OtterSec]'s Drift report cover-to-cover. Three findings applied to my code too — here are the patches and the regression tests."

---

### PHASE 10 — REAL-WORLD SIGNAL (R30–R31) — **MILESTONE 6-B**
*Parallel track. R30 is a 30-day background clock that starts at R20. R31 can slot any time from R15 onward. Both complete before M6.*

#### Ring 30 — *Live keeper operations*
- **Concept:** run infrastructure against a live protocol; deal with real failure modes — RPC flakiness, oracle staleness, fill competition
- **Study:** `sdk/src/orderSubscriber/`, `sdk/src/dlob/` for real production subscriber patterns; your own `keeper/` scaffold from R10
- **Neurons:** R10 (keeper scaffold), R20 (liquidator logic)
- **Build:**
  - Dockerize `keeper/` — one-command deploy
  - Deploy to a cheap VPS (DigitalOcean $5/mo droplet works) running against **Drift devnet** (not your mini-drift)
  - Instrumentation: uptime %, fills attempted / succeeded, oracle-staleness events, RPC error rate, priority-fee spend
  - Simple public dashboard: single-page Vercel app reading a JSON stats file the bot writes every hour
  - **Weekly tweet thread** (track D amplification): *"Week N of live keeper: X fills, Y% uptime, Z interesting failure I hit."*
  - **Acceptance bar:** 30 consecutive days of ≥95% uptime, no manual intervention in any ≥48h window
- **Tests:** 48h pre-flight dry run clean before starting the 30-day clock
- **Est.:** 3 days active build, then 30 calendar days of background clock (kicks off at R20)
- **Demo:** live dashboard URL + *"32 days uptime, 847 fills, 94% success, 3 documented failure modes — here's my post-mortem on each."*

#### Ring 31 — *Upstream contribution*   🏁 **M6-B**
- **Concept:** land code in the actual Drift repo (or disclose a real bounty). **Strongest hire signal in the entire roadmap.**
- **Study:** `github.com/drift-labs/keeper-bots-v2` — open issues, CONTRIBUTING, recent PRs for style. Plus `protocol-v2-master/bug-bounty/README.md` for scope + severity.
- **Neurons:** everything (specifically R10, R14, R20, R30 — your live-keeper operation is where you'll spot what's worth fixing)
- **Build (pick one; both = legendary):**
  - **Path A · PR to `drift-labs/keeper-bots-v2`.** Start with a small quality-of-life improvement to get merged (typo, docs, metrics line). Aim for something substantive by the end. Natural targets:
    - Metrics / logging improvements you wanted while running R30
    - Reliability / performance fix you discovered in R30
    - New CLI flag or bot strategy
    - Bug fix for something you hit in production
  - **Path B · Bounty disclosure.** Find a bug in Drift's in-scope per `bug-bounty/README.md`. Disclose responsibly via Immunefi or the Drift security channel. Even a confirmed low-severity finding is a portfolio gold bar.
- **Acceptance:** merged PR URL **OR** public bounty confirmation
- **Est.:** 1 week of targeted work (most of which is *reading to find the right thing*, not writing)
- **🏁 Demo (M6-B):** "My PR to `keeper-bots-v2` was merged on [date] — here's the URL." — OR — "I responsibly-disclosed [bug class] to Drift on [date]; here's the acknowledgment / bounty."

---

## 7 · Pre-Ring Catch-Up (Current State: 2026-04-23)

You are on **learning R6**, but the build track is behind — `learning_state.md`
puts you at *"R1 catch-up"*. Before resuming R6 learning, the build track needs
to ship R1 → R5 artifacts in a compressed pass:

| Ring | Already understood (from learning) | Build artifact still owed | Est. |
|------|-------------------------------------|----------------------------|------|
| R1 | ✅ | repo init + README skeleton + `docs/rings/R01-*.md` | 0.5 day |
| R2 | ✅ | `docs/rings/R02-*.md` + leverage spreadsheet | 0.5 day |
| R3 | ✅ (derived x\*y=k independently) | `docs/rings/R03-*.md` + AMM trade-trace spreadsheet | 1 day |
| R4 | ✅ (knows checked_mul, scalers) | `anchor init` + `math/constants.rs` + `math/safe_math.rs` + `error.rs` | 1 day |
| R5 | ✅ (delivered 2026-04-20) | `state/user.rs::PerpPosition` + `initialize_user` ix + anchor test | 1 day |

**~4 days of catch-up**, then you resume **R6 learning + R6 build in lockstep**
— concept first, then ship the `Order` struct and `place_perp_order` in the
same ring, per the quality bar in §4.

From R7 onward, every ring is **learning in the morning, building in the
afternoon**.

---

## 8 · Hiring Narrative (how to frame this on a resume / in interviews)

**Resume bullet — lead line:**
> *Built `mini-drift`: a Solana perpetuals DEX with vAMM pricing, portfolio
> margin, funding, liquidations, and insurance-fund bankruptcy resolution. Full
> Anchor program (\~XX KLOC Rust), TS SDK, React UI, and keeper bot. Public devnet
> deployment + 4 technical blog posts. Math modules unit-tested with vectors
> lifted from Drift Protocol v2 to prove numerical equivalence. Threat-modeled,
> fuzz-hardened, and audit-replicated. 30+ days live keeper operations on
> Drift devnet. Accepted PR to `drift-labs/keeper-bots-v2` / disclosed bounty.*

**The 5 stories you'll tell:**

1. **"I rebuilt Drift's funding rate math from first principles."** (R14 blog.)
   Shows you reason from the cap-at-fee-pool constraint, not from a tutorial.
2. **"I can walk a liquidation from underwater oracle tick → liquidator bot →
   insurance-fund settlement in one breath."** (M4 demo.)
   Shows system-level thinking — the interview superpower.
3. **"Here's where I diverged from Drift and why."** (`docs/design-journal.md`.)
   Shows judgment. Choosing *not* to implement prediction markets or isolated
   mode is more impressive than implementing them poorly.
4. **"I threat-modeled my own code, fuzzed the margin + liquidation engines for
   48 hours each, and replicated three findings from a real Drift auditor
   report."** (R27–R29 · blog #4.) Shows security mindset — the senior → staff
   differentiator most candidates never demonstrate.
5. **"My PR is merged in `drift-labs/keeper-bots-v2` — here's the URL."** (R31.)
   Strongest hire signal in the portfolio. Not a cover-letter claim — a
   clickable artifact the hiring manager can verify in one click.

**Interview prep artifacts** (produced naturally by shipping rings):
- Per-ring journal entries in `docs/rings/` → instant study guide for Drift
  codebase deep-dives.
- Numeric test vectors → live-coding warmups.
- `design-journal.md` → tradeoff talking points.

---

## 9 · Weekly Cadence (operational)

- **Mon–Fri mornings:** learning ring (read Drift, extract concept, update `learning_state.md`).
- **Mon–Fri afternoons:** build ring (ship code, tests, docs).
- **Fri EOD:** commit + push, run `cargo clippy`, update `docs/rings/RNN-*.md`, update progress in this file's milestone checkboxes.
- **Sat:** blog draft if within 1 ring of a milestone; otherwise refactor / polish.
- **Sun:** off, or deeper reading (Paradigm blog, SBF Medium, Perp v2 whitepaper, dYdX v4 docs for contrast).

---

## 10 · References

**Primary source (always cite):**
- `../protocol-v2-master/programs/drift/src/` — Drift v2 on-chain program.
- `../protocol-v2-master/sdk/` — Drift TS SDK (for Track B).
- https://drift-labs.github.io/v2-teacher/ — official teacher docs.
- https://github.com/drift-labs/keeper-bots-v2 — reference keeper-bot implementations.

**Secondary (for the blogs):**
- Perp v2 whitepaper (Uniswap-v3-style AMM perps).
- dYdX v4 docs (contrast: off-chain orderbook).
- Paradigm "Everlasting Options" paper (funding rate origin).
- Vertex, GMX, Hyperliquid architecture posts (for contrast paragraph in each blog).

**Tooling:**
- Anchor 0.31.x
- `solana-cli` 1.18+
- `pyth-solana-receiver-sdk`
- Next.js 14 + `@drift-labs/sdk` (for `keeper/` against real Drift)
- Your own generated TS client (for `app/` against mini-drift)

---

## 11 · Progress Tracker (update at every ring close)

### Milestone checklist
- [ ] **M1** · R5–R8 · Can trade
- [ ] **M2** · R9–R14 · Real pricing *(blog #1 published)*
- [ ] **M3** · R15–R18 · Full risk *(blog #2 published)*
- [ ] **M4** · R19–R21 · Can liquidate *(blog #3 published)*
- [ ] **M5** · R22–R26 · Ship *(Vercel live, v1.0.0 tagged, 3 Looms recorded)*
- [ ] **M6** · R27–R31 · Hardened + contributed *(blog #4 published, merged PR or disclosed bounty)*

### Per-ring build status
- [ ] R1 · Repo + vision doc *(catch-up)*
- [ ] R2 · Leverage journal + spreadsheet *(catch-up)*
- [ ] R3 · vAMM trade-trace spreadsheet *(catch-up)*
- [ ] R4 · `anchor init` + math primitives + error enum *(catch-up)*
- [ ] R5 · `PerpPosition` + `initialize_user` *(catch-up)*
- [ ] R6 · `Order` struct + `place_perp_order` *(in lockstep with learning R6)*
- [ ] R7 · Dutch auction + stub fill + filler reward
- [ ] R8 · `update_position_and_market` — **🏁 M1**
- [ ] R9 · Full vAMM + swap_base_asset
- [ ] R10 · Pyth integration + read-only keeper vs real Drift
- [ ] R11 · Static spreads
- [ ] R12 · Dynamic spreads (inventory + volatility + revenue retreat)
- [ ] R13 · Fee pool + tiered fees
- [ ] R14 · Funding rate + funding-crank keeper — **🏁 M2**
- [ ] R15 · SpotMarket + deposit/withdraw
- [ ] R16 · PnL calc (AMM + oracle)
- [ ] R17 · Per-position margin + IMF
- [ ] R18 · Portfolio margin aggregation — **🏁 M3**
- [ ] R19 · `settle_pnl`
- [ ] R20 · `liquidate_perp` + liquidator keeper
- [ ] R21 · Insurance fund + social loss — **🏁 M4**
- [ ] R22 · Repeg + K update + AMM JIT
- [ ] R23 · Matching engine
- [ ] R24 · LP *(optional)*
- [ ] R25 · Market lifecycle
- [ ] R26 · Guard rails + events + CI + deployment — **🏁 M5**
- [ ] R27 · Threat model doc *(parallel · sequential after R21)*
- [ ] R28 · Fuzz harnesses + regression vectors
- [ ] R29 · Audit react-blog + 3 patched findings — **🏁 M6-A**
- [ ] R30 · Live keeper 30d + public dashboard *(parallel · background clock starts at R20)*
- [ ] R31 · Merged PR or disclosed bounty — **🏁 M6-B**

---

*Last updated: 2026-04-23 · Owner: veerbal1 · Companion docs: `../learning/master_map.md`, `../learning/learning_state.md`*
