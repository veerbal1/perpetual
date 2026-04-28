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

The goal is not "copy Drift." The goal is **operator-level understanding**:
you can explain, implement, test, debug, and defend each mechanism from first
principles.

One repo, one public program ID, one portfolio URL, one 60-second demo loop:
deposit USDC → place order → AMM fill → fee debit → funding accrual → oracle
move → margin breach → liquidation → insurance fund settlement.

When this roadmap is complete, you should be able to sit in a Jupiter Perps /
Drift-style interview and comfortably handle:
- live-coding a perp math helper with checked precision;
- tracing one order from request params to stored state to fill to PnL;
- explaining why a risk check exists and what exploit appears without it;
- reading unfamiliar Drift/Jupiter perps code without getting lost;
- naming exactly where your `mini-drift` diverges from production Drift and why.

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
│ │ │ │ │ │         R5 · R6 · R7 · R8        ◄── M1 │ │ │ │ │ │
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
│                R31      background, starts at R14         │
└────────────────────────────────────────────────────────────┘
```

Each ring's build layer is **self-contained** (compiles, tests pass, clippy
clean, commits atomic) and **composable** (the next ring plugs into it without
rewriting).

---

## 2 · The Three Tracks

Track A is the spine. Tracks B and C are proof/demo layers. They are valuable
only when they make Track A more correct, more observable, or easier to explain.

Do **not** let UI or public posting outrun protocol competence. Until M2, the
default is: protocol first, tests second, demo surface last.

| Track | What | Activates | Purpose |
|-------|------|-----------|---------|
| **A · `mini-drift` (Anchor program)** | On-chain perp protocol rebuilt ring by ring | R5 | Core artifact. Every other track exists to support this. |
| **B · `keeper/` (TS ops bot)** | Read real Drift devnet first; later crank your own funding/liquidations | Baseline-gated before R8; R10 read-only; R14 active | Shows you can operate perps, not only write handlers. |
| **C · `app/` (React)** | Thin trader/risk UI: deposit, trade, PnL, margin, liquidation | Baseline-gated before R8; serious after R18 | Demo layer. Never hides missing protocol behavior. |

No doc/blog/Loom track. The deliverable per ring is **working code + tests**.
Private scratch notes or recordings are allowed as memory aids, but they are not
portfolio checkboxes.

---

## 3 · Milestones (acceptance = runnable demo)

Each milestone is a *demo moment* you could screen-share in a hiring loop.

| # | Rings | Working title | Acceptance demo |
|---|-------|---------------|-----------------|
| **M1** | R5–R8 | *"Can trade"* | Localnet: deposit test collateral → place order → fill → open/increase/decrease/close/flip position. Every state change emits an event. |
| **M2** | R9–R14 | *"Prices and costs are alive"* | Localnet/devnet: real vAMM swap, oracle validation, spread, fee accounting, funding update. Math vectors cite Drift tests. |
| **M3** | R15–R18 | *"Can measure risk"* | Deposit/withdraw via token vault, live PnL, initial/maintenance margin, liquidation price, portfolio aggregation. |
| **M4** | R19–R21 | *"Can survive stress"* | PnL settlement, partial liquidation, bankruptcy, insurance fund residual path, plus one end-to-end liquidation bot run. |
| **M5** | R22–R26 | *"Production-shaped"* | Repeg/K maintenance, matching, market lifecycle, oracle guard rails, event audit, CI, devnet deploy, Vercel demo. |
| **M6** | R27–R31 | *"Hardened + externally credible"* | Security regression tests, invariant fuzzing, audit-derived fixes, 30-day keeper ops, merged PR or responsible disclosure. |

---

## 4 · Build Principles (non-negotiable quality bar)

Every ring's deliverable must pass these before you move on. This is the
"hire-at-Drift" bar, not the "works on my machine" bar.

1. **Compiles clean.** `anchor build` → zero warnings. `cargo clippy -- -D warnings` clean.
2. **Testing pyramid per ring.** Pure math unit tests first, then instruction tests, then end-to-end localnet/devnet smoke only when needed. Do not wait for slow `anchor test` to catch math bugs.
3. **Drift vectors where numbers matter.** For R7, R9, R11-12, R14, R16-21, R22, tests must include numeric cases lifted from Drift's `#[cfg(test)]` files. Cite `protocol-v2-master/...:line` in a comment.
4. **Property/invariant tests.** Each mechanism gets at least one invariant: no negative collateral after valid debit, `k` moves only as expected, margin requirement is monotonic in size, liquidation improves solvency, funding is capped.
5. **No silent overflow.** Zero bare `+` `-` `*` `/` on money math — only checked helpers. Zero `.unwrap()` outside tests.
6. **Custom error enum.** `MiniDriftError` grows each ring. Every rejection has a specific error, not a generic catch-all.
7. **Events for every state change.** Mirror Drift's event philosophy — emit on deposit, order open/cancel/fill, position update, fee debit, funding update, PnL settle, liquidation, bankruptcy resolution.
8. **Precision constants centralized.** `math/constants.rs` is the only place `PRICE_PRECISION`, `BASE_PRECISION`, `QUOTE_PRECISION`, `FUNDING_RATE_PRECISION`, `MARGIN_PRECISION` live. No magic numbers in business logic.
9. **Account size documented.** Every `#[account]` struct has a byte-by-byte `LEN` derivation and every array length is justified.
10. **Security checks are explicit.** Every instruction lists signer, owner, PDA seed, writable, token program, duplicate-account, and pause/status assumptions.
11. **Atomic commits.** One logical change per commit. Message format: `ring-XX: <verb> <thing>`. No "wip" commits on `main`.
12. **Foundation debt is an emergency.** R4-R6 cannot remain "typed but unproven." No R7 build begins until R4 math primitives, R5 user/collateral spine, and R6 order storage have tests, events, account-size derivations, and at least one invariant each.
13. **Simplifications are documented gaps, never accidental gaps.** Any skipped Drift field/variant needs a 3-line code comment when implemented: Drift `file:lines`, what was skipped, and whether it is a later-ring item or out of scope.

### Testing Pyramid

| Layer | Tooling | Used for | Starts |
|-------|---------|----------|--------|
| **Pure Rust unit tests** | `cargo test` | math, casting, safe math, spread, funding, margin, liquidation | R4 |
| **Property tests** | `proptest` first; fuzz later | monotonicity, solvency, rounding, overflow edges | R8 light, R27 heavy |
| **Instruction tests** | Anchor TS tests or LiteSVM/Mollusk-style fast harness | account constraints, events, token movement, keeper permissions | R5 |
| **Localnet E2E** | `anchor test` | full user journeys | R8 |
| **Devnet smoke** | scripts + keeper | Pyth, live RPC, keeper behavior, deployment sanity | R10+ |

### Ring Closure Gate

Before a ring can be checked off:
- You can explain the concept without looking at Drift.
- You can point to the exact Drift files you studied.
- The new code imports, calls, extends, or mutates code from at least one prior ring.
- Happy path, failure path, and one invariant are tested.
- Events and account-size changes are explicit.
- Any intentional divergence from Drift is recorded in code comments or this roadmap.

### Track B/C Baseline Gate

Before opening R8, run two one-hour tests and record the band in the roadmap:
- **TypeScript/Solana:** write a Node script that connects to devnet, fetches a Pyth SOL/USD price account, decodes price + confidence, and prints them. No tutorials or copy-paste from existing project scripts.
- **React/Next.js:** scaffold a page with wallet connect and display the connected wallet's SOL balance. No tutorials.

The result changes the roadmap:
- **Band 1:** both ship cleanly in under an hour → keep Track B/C as planned.
- **Band 2:** ships in 2-3 hours with doc lookups but no panic → keep Track B/C, apply 1.5x calendar multiplier to every bot/UI estimate.
- **Band 3:** cannot ship without tutorial copy-paste or gets stuck on environment setup → Track B becomes read-only CLI/Bash/Python until a 1-week TS sprint between R14-R15; Track C becomes CLI/state inspection until R26.
- **Band 4:** cannot explain wallet adapter / Next setup basics → Track B becomes Rust CLI keeper; Track C is deferred until after protocol ships. Anchor tests + CLI walkthroughs become the demo through M4.

Regardless of band, the reading half of Track B is non-negotiable from R10:
read Drift SDK modules (`orderSubscriber/`, `dlob/`, `accounts/`) well enough
to explain what they do. If TS reading is weak, insert a 3-day TS reading ramp
before R10 opens.

### Interview-Critical Gates

These rings do not close on passing tests alone:
- **R9 AMM:** cold 5-minute explanation of reserves, peg, concentration, bounds, and net AMM exposure.
- **R10 oracle:** cold explanation of stale price, confidence, and mark/oracle divergence attacks.
- **R11-R12 spreads:** cold explanation of inventory and volatility effects on bid/ask.
- **R14 funding:** cold explanation of TWAP, asymmetry, fee-pool cap, and what happens when the cap bites.
- **R17-R18 margin:** cold explanation of IMF, strict valuation, portfolio aggregation, and why Drift is not correlation-aware.
- **R20-R21 liquidation/insurance:** cold explanation of maintenance breach → partial liquidation → bankruptcy → insurance fund → socialized loss.

Private recordings are fine for self-review, but they are calibration tools, not
portfolio deliverables.

At every milestone close M1-M6, schedule a 30-minute stress quiz:
- two fake-out checks randomly chosen from prior rings;
- one fake-out check from the current milestone;
- fail any core mechanism → milestone re-opens.

Each phase closes with one verbal design-alternatives check: name one
alternative considered, why Drift does it differently, and what `mini-drift`
intentionally chose.

### Do Not Simplify Away

These are essence, not polish. If any are missing, `mini-drift` is no longer a
serious Drift-style perp rebuild:
- concentration coefficient and bounded reserves;
- strict valuation: assets use `min(mark, oracle)`, liabilities use `max(mark, oracle)`;
- funding cap tied to fee-pool capacity;
- IMF size premium;
- portfolio margin with unsettled funding;
- bankruptcy → insurance fund → socialized loss;
- event emission on every state change;
- working liquidator keeper.

### Fake-Out Check Bank

Use these in milestone stress quizzes to expose copied-but-not-owned knowledge:
- Change one AMM trade input by 10%; predict mark direction and rough magnitude before running tests.
- Longs owe $100M funding and shorts receive $80M; name where the $20M gap comes from and what caps it.
- Explain why a 100-SOL position needs more than 10x the margin of a 10-SOL position.
- Give three oracle failure modes and the protocol guard for each.
- Name where the vAMM's actual base-asset tokens are held. Correct answer: nowhere; it is accounting exposure.
- Change one Drift test vector input; predict output direction before reading the assertion.
- Liquidation leaves a user at negative equity; walk bankruptcy, IF draw, and socialization.

### Required Invariants

These are the literal specs the implementation and R28 fuzzers attack.

| Area | Required invariants |
|------|---------------------|
| **AMM** | `base_asset_reserve * quote_asset_reserve ≈ sqrt_k^2` after scaling; reserves stay inside `[min_base_asset_reserve, max_base_asset_reserve]`; `base_asset_amount_long >= 0`; `base_asset_amount_short <= 0`; sum of user base for a market equals `-base_asset_amount_with_amm`. |
| **Funding** | `abs(funding_rate) <= cap_from_fee_pool`; cumulative funding rates move only in the valid direction per side; total long/short funding plus protocol gap never exceeds available fee-pool draw. |
| **Margin** | total collateral is nonnegative for users not being liquidated; margin requirement is monotonic in absolute position size; closing a position never increases margin requirement; post-fill user meets initial margin or the order rejects. |
| **Liquidation** | post-liquidation user is above maintenance, fully liquidated, or entering bankruptcy; liquidator fee + IF fee <= position notional; `liquidation_margin_freed` is monotonic; `BeingLiquidated` clears only when user exits danger. |
| **Insurance** | insurance fund balance never goes negative; socialized loss exactly equals residual not covered by IF; IF can only be drawn for declared bankruptcy events. |

### Senior Calibration Protocol

Lovely is periodic calibration, not a daily co-pilot. Daily teaching/build
stays with Aman unless a calibration point is hit.

Pull Lovely back in:
- right after the Track B/C band test, to confirm classification and lock the branch;
- before opening R9, R14, and R20, to pre-flight advance-trigger criteria and proof bars;
- at every milestone close, to design stress-quiz checks and decide whether the milestone is really done;
- for the M4 solvency review: `math/liquidation.rs`, `math/bankruptcy.rs`, `controller/insurance.rs`;
- for R31 PR target selection;
- if Veerbal goes quiet for more than 2 weeks.

Aman may override a proposed gate without escalation when day-to-day evidence
shows it is wrong for Veerbal's current pacing, energy, or confusion pattern.
If that happens, record what changed and why. The plan is a tool, not a contract.

Hard rule: M4 cannot close until the solvency-math review actually happens.
Liquidation/insurance bugs are catastrophic, not merely embarrassing.

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
├── scripts/                              ← devnet/admin/operator scripts
│   ├── devnet-smoke.ts
│   └── inspect-account.ts
├── app/                                  ← mini-drift-web (Next.js + Vercel)
├── keeper/                               ← drift-keeper bot (TS, runs vs REAL Drift devnet)
└── sdk/                                  ← TS client for your program (auto-generated + hand-rolled helpers)
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
*Pure learning rings. No code, no doc deliverable.*

#### Ring 1 — *What is a perp* [`learning R1`]
- **Concept:** trading, futures, perpetuals, the exchange
- **Study:** none in code yet; optional skim of Drift teacher docs intro
- **Neurons:** none (base layer)
- **Build:** none. Concept lands in the learning track.
- **Tests:** none
- **Est.:** part of learning ring
- **Demo:** Explain a perp in plain English without using jargon.

#### Ring 2 — *Your first trade* [`learning R2`]
- **Concept:** long/short/PnL/collateral/margin/leverage
- **Study:** none in code
- **Neurons:** R1
- **Build:** none. Concept lands in the learning track.
- **Tests:** none
- **Est.:** part of learning ring
- **Demo:** "Here are the 4 invariants of PnL arithmetic, with numbers."

---

### PHASE 2 — THE ENGINE (R3–R4)
*First code lands at R4: math primitives + account scaffolding. No trading yet.*

#### Ring 3 — *The robot trader (AMM intuition)* [`learning R3`]
- **Concept:** AMM, x\*y=k, peg, price from reserves
- **Study:**
  - `programs/drift/src/math/amm.rs` — just skim `calculate_price`, `calculate_quote_asset_amount_swapped`
  - `programs/drift/src/math/constants.rs` — PRICE_PRECISION = 1e6, BASE_PRECISION = 1e9, AMM_RESERVE_PRECISION = 1e9
- **Neurons:** R2 (counterparty problem motivates AMM)
- **Build:** none. Concept lands in the learning track. Optional scratchpad: trace 5 sequential trades through `x*y=k`.
- **Tests:** none
- **Est.:** part of learning ring
- **Demo:** Explain why repeated buys move the AMM price upward.

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
- **Tests:** 🎯 unit tests for overflow/underflow/cast-boundary (one per method); vectors from `math/safe_math.rs` tests in Drift if present. Add one invariant test: any failed checked operation returns a `MiniDriftError`, never panics.
- **Est.:** 1 day
- **Demo:** "Here's `cargo test` — every checked op has overflow + underflow + boundary cases."

---

### PHASE 3 — THE TRADE (R5–R8) — **MILESTONE 1**
*First on-chain lifecycle: initialize → test-collateral → place order → fill → close.*

This phase deliberately creates a **minimal quote-collateral spine** before the
full SpotMarket lesson in R15. It is not full Drift spot; it is a tiny USDC
ledger so fills, fees, realized PnL, and funding have a real place to move.
R15 replaces the simple ledger with Drift-shaped `SpotPosition` + vault logic.

#### Ring 5 — *Your position in code* [`learning R5`]
- **Concept:** `PerpPosition` struct, signed base/quote, direction
- **Study:**
  - `programs/drift/src/state/user.rs` — `PerpPosition`, `SpotPosition`, `User`, `is_open_position`, `is_available`
  - `programs/drift/src/controller/spot_balance.rs` — only skim the idea of one shared money pipe; full details wait until R15
- **Neurons:** R4 (precision + accounts), R3 (base/quote from x\*y=k), R2 (collateral makes leverage safe)
- **Build:**
  - `state/user.rs` — `User` account with `perp_positions: [PerpPosition; 8]` (Drift has more; 8 is plenty for mini)
  - `PerpPosition` with: `base_asset_amount: i64`, `quote_asset_amount: i64`, `quote_entry_amount: i64`, `quote_break_even_amount: i64`, `last_cumulative_funding_rate: i64`, `market_index: u16`
  - `QuoteCollateral` mini-spine on `User`: one signed/unsigned quote balance field for test collateral, plus one helper boundary where fees, PnL, and funding will later flow
  - `instructions/user.rs::initialize_user` — creates `User` PDA seeded by `[b"user", authority, sub_account_id]`
  - `instructions/user.rs::deposit_test_collateral` — localnet-only/simple instruction that credits the quote-collateral spine; R15 replaces this with real SPL token vault movement
- **Tests:** `tests/01-initialize-user.ts` — init + empty positions + zero collateral; deposit test collateral updates only the user's quote-collateral field and emits an event.
- **Est.:** 1.5 days
- **Demo:** "One tx creates a user PDA, another credits test collateral. Later rings have a real money pipe to touch."

#### Ring 6 — *Placing your bet* [`learning R6`]
- **Concept:** `Order` struct, `OrderType`, `OrderParams`, lifecycle, reduce/post/IOC
- **Study:**
  - `programs/drift/src/state/user.rs` — `Order` struct + `OrderStatus`
  - `programs/drift/src/state/order_params.rs` — `OrderParams`, `OrderTriggerCondition`, `OrderParamsBitFlag`
- **Neurons:** R5 (orders produce positions and will later reserve/debit collateral), R4 (fixed-size arrays and account sizing)
- **Build:**
  - `state/user.rs::Order` with: `order_id`, `market_index`, `price`, `base_asset_amount`, `direction: PositionDirection`, `order_type: OrderType`, `status: OrderStatus`, `auction_duration`, `auction_start_price`, `auction_end_price`, `max_ts`, `bit_flags: u8` (reduce/post/IOC packed)
  - Define all Drift-shaped `OrderType` variants you learned, but only **Market** and **Limit** are executable. Unsupported variants return `MiniDriftError::UnsupportedOrderType` until their later rings.
  - `OrderParams` mirrors Drift's request-vs-storage split. Optional params normalize into stored defaults at insertion time.
  - `orders: [Order; 16]` slot on `User`
  - `instructions/user.rs::place_perp_order(params: OrderParams)` — validates, inserts into first available slot, emits `OrderRecord` event
- **Tests:** happy path + "no free slot" failure + reduce-only on empty position = reject + unsupported trigger/oracle order rejected with specific error. Add one serialization/account-size test for the fixed order array.
- **Est.:** 2 days
- **Demo:** "Place an order → event emits → next slot filled. Reduce-only correctly rejects."

#### Ring 7 — *Getting filled* [`learning R7`]
- **Concept:** Dutch auction, keepers, fill flow, fees-on-fill
- **Study:**
  - `programs/drift/src/math/auction.rs` — `calculate_auction_price`
  - `programs/drift/src/math/fees.rs` — `FeeTier`, taker fee calculation
  - `programs/drift/src/controller/orders.rs::fill_perp_order` — orchestration sketch only; you'll stub AMM until R9
- **Neurons:** R6 (orders), R5 (collateral spine receives fee debit), R3 (AMM as stub counterparty)
- **Build:**
  - `math/auction.rs::calculate_auction_price(order, slot) -> i64` — linear interp between start/end over `auction_duration` slots
  - `math/fees.rs` — fixed taker fee (10 bps) + fixed filler reward for now, refined in R13
  - `controller/orders.rs::fulfill_perp_order` — stub AMM fill: use auction price, debit fee from quote-collateral spine, mutate position through the R8 position helper boundary
  - `instructions/keeper.rs::fill_perp_order` — permissionless filler pubkey receives accounting credit/event; SPL transfer waits until the real vault path
- **Tests:** 🎯 `auction.rs` tests: copy Drift's auction vectors from `math/auction.rs::tests`. Add fee-debit invariant: collateral decreases by exactly fee + realized loss, never by more.
- **Est.:** 3 days
- **Demo:** "Any wallet can fill — here's a second tx from a different signer filling the order; auction price is deterministic from slot."

#### Ring 8 — *Position updates* [`learning R8`]   🏁 **M1**
- **Concept:** PositionDelta, open/increase/decrease/close
- **Study:** `programs/drift/src/controller/position.rs::update_position_and_market`
- **Neurons:** R5 (position), R7 (fills trigger updates)
- **Pre-flight:** run the Track B/C Baseline Gate from §4 before opening this ring. Lock the resulting band into the roadmap so bot/UI scope is not discovered late at R10.
- **Build:**
  - `math/position.rs::calculate_position_delta` — pure fn: given fill size/price/direction + existing position, returns new `(base, quote_entry, quote_break_even, realized_pnl)`
  - `controller/position.rs::update_position_and_market` — applies delta; handles 4 cases (open / increase / decrease / close-and-flip)
  - Close-and-flip: opening opposite side beyond current size realizes PnL on the closed portion, opens residual on the new side
- **Tests:** 🎯 four-case vector matrix lifted from `controller/position.rs::tests`; each of open/increase/decrease/flip has an exact Drift-match test. Add one round-trip invariant: open then close at same price changes collateral only by fees.
- **Est.:** 3 days
- **🏁 Demo (M1):** Localnet/Anchor test: "deposit 1000 test-USDC → long 5 SOL @ $180 → close/flip → realized PnL and collateral delta match the spreadsheet."
- **Track C starts only as a readout stub here.** One page may display user/order/position state, but no serious UI polish until R18 risk numbers exist.

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
  - Plant the LP forward reference explicitly: the initialized virtual reserves are conceptually backed by LP capital; R24 closes that loop
- **Tests:** 🎯 heavy vector matrix from `math/amm.rs::tests` — covers swap, bounds hit, concentration enforcement, min/max reserve guards. Add property tests: reserves stay inside min/max bounds; invalid swaps fail before mutating state.
- **Est.:** 10-14 days (biggest ring so far; do not compress the vector matrix)
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
  - `MockOracle`/admin-set oracle for deterministic local tests first; this keeps all R10-R21 tests reproducible without live RPC
  - Pyth adapter via `pyth-solana-receiver-sdk` second — load `PriceUpdateV2` account, extract price + confidence
  - `math/oracle.rs::is_oracle_valid` — staleness + confidence-interval + divergence-vs-mark checks, returning an explicit validity enum
  - Mark the `oracle` field on `PerpMarket` as `Pubkey` + `OracleSource::Pyth`
- **Tests:** deterministic local tests for fresh/stale/high-confidence/divergent oracle. Devnet smoke test loads real Pyth SOL/USD and asserts valid + price in sane range.
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
  - Extend `math/amm_spread.rs` in three small commits: inventory scale, volatility scale, revenue retreat
  - Revenue retreat needs `net_revenue_since_last_funding` on AMM — add the field, forward-declare (gets fed in R13/R14)
- **Tests:** 🎯 full dynamic spread vectors from Drift. These are notorious edge cases — find at least 10 and match all.
- **Est.:** 7-8 days
- **Demo:** "Pile in $100k of longs → ask widens, bid tightens. Demo it on localnet."

---

### PHASE 5 — ONGOING COSTS (R13–R14) — **MILESTONE 2**

#### Ring 13 — *Where money flows* [`learning R13`]
- **Concept:** fee accumulation, fee pool, total_fee_minus_distributions, fee tiers
- **Study:**
  - `programs/drift/src/state/state.rs` — `FeeStructure`, `FeeTier`
  - `programs/drift/src/state/perp_market.rs` — `PoolBalance`, `total_fee`, `total_fee_minus_distributions`, `fee_pool`
  - `programs/drift/src/math/fees.rs`
- **Neurons:** R7 (fees charged), R5 (quote-collateral spine), R11-12 (revenue retreat reads these)
- **Build:**
  - `state/state.rs::State` global singleton with `FeeStructure` (4 tiers by volume)
  - `PoolBalance` struct on `PerpMarket` — `scaled_balance`, `market_index`
  - Fill path debits the user's quote-collateral spine and credits `total_fee`, `total_fee_minus_distributions`, and the fee pool
  - Volume-based tier: track `UserStats.taker_volume_30d` (simple EMA)
- **Tests:** fills across 4 tier boundaries; fee invariant `user_debit == fee_pool_delta + filler_reward + protocol_distribution_delta`. Revenue-retreat input updates from actual fee accounting.
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
  - `controller/funding.rs::settle_funding_payment` — per-position, flowing through the same quote-collateral helper used by R7/R13
- **Tests:** 🎯 funding vectors from `math/funding.rs::tests` — mark above oracle (longs pay shorts), mark below (shorts pay longs), capped case, zero-OI case. Add invariant: total funding transfer across longs/shorts plus protocol gap equals the accounted market revenue delta.
- **Est.:** 4 days
- **🚀 Track B milestone:** your keeper bot now has a second command: `funding-crank` — watches your devnet deployment, cranks funding once per hour.
- **External contribution background begins here:** start `keeper-bots-v2` contribution scanning, 2 hours/week. The first goal is not a heroic PR; it is learning Drift's review/contribution loop.
- **🏁 Demo (M2):** "Devnet run: I open long, time-warp 2 hours, funding has accrued, next close realizes the funding payment. Numbers match my spreadsheet."

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
- **Neurons:** R4 (accounts), R5 (quote-collateral spine becomes real `SpotPosition` storage), R7/R13/R14 (fees and funding already use the money pipe)
- **Build:**
  - `state/spot_market.rs::SpotMarket` — USDC collateral market: `mint`, `vault`, `cumulative_deposit_interest`, `decimals`
  - `state/user.rs::SpotPosition { market_index, scaled_balance, balance_type: SpotBalanceType }`
  - `user.spot_positions: [SpotPosition; 8]`, index 0 = USDC always
  - `instructions/user.rs::deposit` / `withdraw` — SPL token CPI to/from market vault
  - Delete the R5 placeholder balance path. Every controller that previously touched it now calls `controller/spot_balance.rs::update_spot_balances`
- **Tests:** anchor test: deposit 1000 USDC → `scaled_balance` correct → withdraw 500 → balance + vault match. Regression: R7/R13/R14 fee and funding tests still pass without changing their public controller calls. Add code search check: no controller writes the old placeholder directly.
- **Est.:** 3 days
- **Demo:** UI: "Deposit / withdraw USDC collateral."

#### Ring 16 — *PnL calculation* [`learning R16`]
- **Concept:** exit - entry, AMM-based and oracle-based valuation
- **Study:** `programs/drift/src/math/pnl.rs`, `math/position.rs::calculate_base_asset_value`
- **Neurons:** R9 (AMM valuation), R10 (oracle valuation), R15 (PnL lands in spot balance)
- **Build:**
  - `math/pnl.rs::calculate_unrealized_pnl` — AMM mark version + oracle version
  - Strict valuation: `min(mark, oracle)` for assets, `max(mark, oracle)` for liabilities
- **Tests:** 🎯 PnL vectors from `math/pnl.rs::tests`. Add invariant: closing a position at its entry mark has zero realized PnL before fees/funding.
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
- **Tests:** 🎯 margin/IMF vectors from `math/margin/tests.rs`; add monotonicity property: increasing absolute position size never lowers required margin for the same market config.
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
- **Tests:** multi-position scenarios: 2 perp positions + 1 spot borrow/deposit + unsettled funding; total collateral and requirement both line up. Add rejection test: an order that passes naive per-position margin but fails portfolio margin is rejected.
- **Est.:** 4 days
- **🏁 Demo (M3):** UI shows live: total collateral, initial margin used, maintenance margin, **liquidation price**. Move the oracle → liq price moves.

#### Ring 19 — *PnL settlement* [`learning R19`]
- **Concept:** PnL pool, settling +/-, imbalance, divergence checks
- **Study:** `programs/drift/src/controller/pnl.rs::settle_pnl`, `math/pnl.rs::calculate_max_pnl_pool_excess`
- **Neurons:** R13 (fee pool funds PnL pool), R15-18 (margin check gates settle)
- **Build:**
  - `PerpMarket.pnl_pool: PoolBalance`
  - `instructions/user.rs::settle_pnl` — user-triggered on any position
  - Price divergence check: if mark diverges too far from oracle, refuse settle
  - PnL imbalance: if pool lacks capacity, settle partially
- **Tests:** positive settle, negative settle, pool-exhaustion scenario, divergence rejection. Add accounting invariant: user spot delta + PnL pool delta + residual unsettled PnL equals pre-settle PnL.
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
  - No trigger-order implementation here. Liquidation is protocol safety; user stop-loss orders are activated later after the matching/oracle paths are stable.
- **Tests:** 🎯 liquidation vectors from `math/liquidation.rs::tests` — one long, one short, one across multiple markets. Add invariant: after partial liquidation, the user is either closer to maintenance safety or enters bankruptcy; never silently worsens.
- **Est.:** 8-10 days
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
- **Tests:** residual covered by IF, residual exceeding IF triggers socialized loss. Add invariant: total deficit after resolution is zero or explicitly recorded as socialized loss.
- **Est.:** 5 days
- **🏁 Demo (M4):** "End-to-end devnet stress run: 5 traders, oracle swings, 3 liquidations, 1 bankruptcy covered by IF, 1 partial socialization."

---

### PHASE 7 — AMM HEALTH (R22–R23)

#### Ring 22 — *AMM maintenance (repeg + K)* [`learning R22`]
- **Concept:** repeg, formulaic K updates
- **Study:**
  - `programs/drift/src/math/repeg.rs`
  - `programs/drift/src/math/cp_curve.rs::update_k`
  - `programs/drift/src/controller/repeg.rs`
- **Neurons:** R9 (AMM internals), R10 (oracle = repeg target), R13 (fee pool funds repeg)
- **Build:**
  - `math/repeg.rs::calculate_repeg_cost` — pure
  - `controller/repeg.rs::repeg_curve` — admin + auto variants, budgeted by fee pool
  - `math/cp_curve.rs::update_k` — auto-scale with `curve_update_intensity`
  - AMM JIT is explicitly deferred/optional; do not bundle it with repeg/K unless the core path is already clean
- **Tests:** 🎯 repeg cost vectors; K update vectors. Add invariant: repeg cannot spend more than its fee-pool budget.
- **Est.:** 4 days
- **Demo:** "Oracle drifts 2% → auto-repeg within budget. Here's the before/after."

#### Ring 23 — *Matching engine* [`learning R23`]
- **Concept:** fulfillment methods, maker/taker matching
- **Study:** `programs/drift/src/math/matching.rs`, `math/fulfillment.rs`, `controller/orders.rs::fulfill_perp_order` (full), optional skim `math/amm_jit.rs`
- **Neurons:** R6-7 (orders + fills), R11-12 (spreads)
- **Build:**
  - `math/matching.rs::do_orders_cross`, `calculate_fill_for_matched_orders`
  - Fill path tries **Match first**, falls through to **AMM**
  - Maker rebate / filler multiplier on matched fills
  - Optional extension: simplified AMM JIT participation after basic maker/taker matching is correct
- **Tests:** maker sitting at $100, taker market order → match at $100 with maker rebate; no match → AMM fallback.
- **Est.:** 4 days
- **Demo:** "Two wallets — one posts limit, one markets in. Filled against each other at 0 AMM fee."

---

### PHASE 8 — ADVANCED (R24–R26) — **MILESTONE 5**
*You ship the production-shaped core. Scope stays slim, but no core Drift mechanic gets handwaved.*

#### Ring 24 — *LP system* [`learning R24`]  *(mandatory, slim)*
- **Concept:** LP shares, settlement, rebase
- **Study:**
  - `programs/drift/src/controller/lp.rs`
  - `programs/drift/src/math/lp_pool.rs`
  - `programs/drift/src/state/perp_market.rs` — LP-related AMM fields
- **Neurons:** R9 (virtual reserves and `sqrt_k`), R22 (K updates affect LP value), R13 (fees/revenue affect LP economics)
- **Build:** mandatory slim version: `add_liquidity`, `remove_liquidity`, `settle_lp`, `base_asset_amount_per_lp`, `quote_asset_amount_per_lp`. Skip LP rebase and tokenized LP shares, but document the skipped Drift code with file/line comments when implementing.
- **Tests:** LP share/reserve invariant; add/remove round trip before price move; LP settlement after AMM inventory changes.
- **Est.:** 7-8 days

#### Ring 25 — *Market lifecycle study* [`learning R25`]  *(folded into R26 build)*
- **Concept:** MarketStatus state machine, paused ops, expiry
- **Study:** `programs/drift/src/state/perp_market.rs` market status fields, `state/paused_operations.rs`, `instructions/constraints.rs`
- **Build:** no standalone checkpoint. Fold implementation into R26 so market lifecycle, pause controls, guard rails, events, CI, and deployment ship as one production-controls package.
- **Est.:** 1 day study + design pass

#### Ring 26 — *Polish, infra, guard rails* [`learning R26`]   🏁 **M5**
- **Concept:** oracle guard rails, events, remaining order types, account maps, deployment polish
- **Build:**
  - `math/oracle.rs::OracleValidity` + `OracleGuardRails` in global `State`
  - `state/perp_market.rs::MarketStatus` — `Initialized → Active → ReduceOnly → Settlement → Delisted`
  - `PausedOperations` bitflags and instruction-level pause/status constraints
  - Admin can pause a market; reduce-only blocks position-increasing orders
  - Activate previously unsupported `TriggerMarket`, `TriggerLimit`, and `Oracle` order paths only if R6-R23 order/fill behavior is stable; otherwise document them as unsupported with explicit errors
  - Complete event coverage audit — every state change emits an event
  - `PerpMarketMap` / `SpotMarketMap` / `OracleMap` for efficient remaining-accounts loading
  - CI: GitHub Actions running `anchor build` + `anchor test` + `cargo clippy -- -D warnings`
  - **Deploy to devnet with public program ID in README**
  - **Vercel deployment of `app/`**
- **Est.:** 7-10 days
- **🏁 Demo (M5):** Public Vercel URL + public program ID + tagged `v1.0.0` release + README demo GIF.

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
- **Build:** **15+ concrete attack scenarios → 15+ regression tests in `programs/mini-drift/tests/security/`**, grouped by category:
  - **Oracle:** stale price, confidence-interval spoofing, cross-venue manipulation, Pyth publisher collusion
  - **Timing / MEV:** sandwich on funding crank, back-run liquidation, JIT front-run, priority-fee DoS on keepers
  - **Rounding / precision:** balance inflation via repeated dust deposits, off-by-one in IMF, rounding-direction exploit in PnL settlement
  - **Solvency:** IF drain via repeated bankruptcies, socialized-loss gaming, arb between mark and oracle at settle
  - **Logic:** reduce-only bypass, post-only slide-through, reentrancy via SPL CPI, privilege escalation on admin ix
  - Each test name encodes the attack; each test's setup proves the prereqs; each assertion proves the mitigation. The test file IS the threat model.
- **Tests:** the build IS the tests
- **Est.:** 3 days
- **Demo:** "Pick any test in `tests/security/` — I'll walk the exploit it's pinning down and the mitigation in code."

#### Ring 28 — *Fuzzing*
- **Concept:** surface bugs random inputs find that humans miss
- **Study:** `cargo-fuzz` book, `proptest` / `arbitrary` crates, the `#[cfg(test)]` blocks in Drift math modules (you've been lifting these — now you attack them)
- **Neurons:** R4 (safe math — fuzzing finds where the safety is imagined), R17–18 (margin invariants), R20 (liquidation multiplier)
- **Build:**
  - Target the Required Invariants from §4 directly. Each fuzzer has a named invariant to break, not a vague "random inputs" goal
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
- **Neurons:** R27 (your own attack tests will look naive next to professionals'), R28 (fuzzing vs human review — complementary)
- **Build:**
  - Pick **3 findings applicable to your codebase**. For each: write the patch in mini-drift + a regression test in `tests/security/` whose name cites the audit (e.g. `ottersec_2023_<finding_id>_<short_desc>`).
  - If the audit surfaces an attack category your R27 tests missed, add the new tests too.
- **Tests:** one regression test per patched finding (named after the audit's finding ID)
- **Est.:** 5 days
- **🏁 Demo (M6-A):** "Three Drift audit findings reapplied here — point at the test, I'll walk the original auditor's analysis and my patch."

---

### PHASE 10 — REAL-WORLD SIGNAL (R30–R31) — **MILESTONE 6-B**
*Parallel track. R30 is a 30-day background clock that starts at R20. R31 is not a final boss; contribution scanning starts at R14 and becomes increasingly concrete.*

#### Ring 30 — *Live keeper operations*
- **Concept:** run infrastructure against a live protocol; deal with real failure modes — RPC flakiness, oracle staleness, fill competition
- **Study:** `sdk/src/orderSubscriber/`, `sdk/src/dlob/` for real production subscriber patterns; your own `keeper/` scaffold from R10
- **Neurons:** R10 (keeper scaffold), R20 (liquidator logic)
- **Build:**
  - Dockerize `keeper/` — one-command deploy
  - Deploy to a cheap VPS (DigitalOcean $5/mo droplet works) running against **Drift devnet** (not your mini-drift)
  - Instrumentation: uptime %, fills attempted / succeeded, oracle-staleness events, RPC error rate, priority-fee spend
  - Simple public dashboard: single-page Vercel app reading a JSON stats file the bot writes every hour
  - **Acceptance bar:** 30 consecutive days of ≥95% uptime, no manual intervention in any ≥48h window
- **Tests:** 48h pre-flight dry run clean before starting the 30-day clock
- **Est.:** 3 days active build, then 30 calendar days of background clock (kicks off at R20)
- **Demo:** live dashboard URL + *"32 days uptime, 847 fills, 94% success, 3 documented failure modes — here's my post-mortem on each."*

#### Ring 31 — *Upstream contribution*   🏁 **M6-B**
- **Concept:** land code in the actual Drift repo (or disclose a real bounty). **Strongest hire signal in the entire roadmap.**
- **Study:** `github.com/drift-labs/keeper-bots-v2` — open issues, CONTRIBUTING, recent PRs for style. Plus `protocol-v2-master/bug-bounty/README.md` for scope + severity.
- **Neurons:** everything (specifically R10, R14, R20, R30 — your live-keeper operation is where you'll spot what's worth fixing)
- **Build / contribution ladder:**
  - **Start at R14:** block 2 hrs/week reading `keeper-bots-v2`; keep a private rough-edge list for PR candidates.
  - **Tier 1 by R16:** tiny docs/README/CLI-flag clarification PR. Goal: learn the contribution loop and review dynamics.
  - **Tier 2 by R20:** observability PR: priority-fee spend logging, oracle-staleness metric, RPC error counter, or similar ops-facing improvement.
  - **Tier 3 by R26:** reliability PR based on a real failure hit while running your keeper: retry/backoff, stale-account handling, or RPC failure hardening.
  - **Path B · Bounty disclosure:** parallel option, not a replacement. Find a real in-scope issue and disclose responsibly.
- **Acceptance:** merged PR URL **OR** public bounty confirmation
- **Est.:** 1 week of targeted work (most of which is *reading to find the right thing*, not writing)
- **🏁 Demo (M6-B):** "My PR to `keeper-bots-v2` was merged on [date] — here's the URL." — OR — "I responsibly-disclosed [bug class] to Drift on [date]; here's the acknowledgment / bounty."

---

## 7 · Pre-Ring Catch-Up (Current State: 2026-04-28)

R1-R3 are pure-learning rings — no build artifact owed.

R4-R6 structs and constants are typed in `mini-drift/`. Instructions, tests,
and the orchestrating `User` account are not yet shipped:

| Ring | What's on disk | What's still owed |
|------|----------------|-------------------|
| R4 | `math/constants.rs`, `math/safe_math.rs`, `error.rs` typed | unit tests for every checked op (overflow / underflow / boundary) |
| R5 | `PerpPosition` struct typed | `User` account · quote-collateral spine · `initialize_user` ix · `deposit_test_collateral` ix · anchor test |
| R6 | `Order`, `OrderType` (Market/Limit), `OrderStatus`, `PositionDirection`, `OrderParams` typed | `User.orders: [Order; 16]` · `place_perp_order` ix · `OrderRecord` event · happy path + reject tests |

Hard gate: do not start R7 build until R4-R6 have tests, events, account-size
derivations, and at least one invariant each. R7 fills need a real place to
debit fees.

From R7 onward, every ring is **learning in the morning, building in the
afternoon**.

---

## 8 · Weekly Cadence (operational)

- **Mon–Fri mornings:** learning ring (read Drift, extract concept, update `learning_state.md`).
- **Mon–Fri afternoons:** build ring (ship user-written code + tests).
- **Fri EOD:** commit + push, run `cargo clippy`, tick the ring's checkbox in §10.
- **From R10 onward:** consume-only market presence. Lurk Drift/Jupiter Discord, follow perp engineers, and study technical threads. No posting/replies yet.
- **From R14 onward:** block 2 hrs/week for `keeper-bots-v2` reading and PR candidate notes. This is background work, not a reason to slow Track A.
- **From R20 onward:** 1 hr/week technical outbound: thoughtful Discord replies, Solana hackathon presence, or one substantive perp insight. No low-signal posting.
- **Sat:** refactor, polish, or deeper reading.
- **Sun:** off, protected.

Daily minimum during build catch-up:
- one Drift source file opened;
- one `mini-drift` artifact advanced;
- one test or invariant added;
- one private recall note if something surprised you.

---

## 9 · References

**Primary source (always cite):**
- `../protocol-v2-master/programs/drift/src/` — Drift v2 on-chain program.
- `../protocol-v2-master/sdk/` — Drift TS SDK (for Track B).
- https://drift-labs.github.io/v2-teacher/ — official teacher docs.
- https://github.com/drift-labs/keeper-bots-v2 — reference keeper-bot implementations.

**Secondary (background reading):**
- Perp v2 whitepaper (Uniswap-v3-style AMM perps).
- dYdX v4 docs (contrast: off-chain orderbook).
- Paradigm "Everlasting Options" paper (funding rate origin).
- Vertex, GMX, Hyperliquid architecture posts.

**Tooling:**
- Anchor 0.31.x
- `solana-cli` 1.18+
- `pyth-solana-receiver-sdk`
- `proptest` for light property tests
- LiteSVM or Mollusk-style fast instruction tests where practical
- Next.js 14 + `@drift-labs/sdk` (for `keeper/` against real Drift)
- Your own generated TS client (for `app/` against mini-drift)

---

## 10 · Progress Tracker (update at every ring close)

### Milestone checklist
- [ ] **M1** · R5-R8 · Can trade
- [ ] **M2** · R9-R14 · Real pricing
- [ ] **M3** · R15-R18 · Full risk
- [ ] **M4** · R19-R21 · Can liquidate
- [ ] **M5** · R22-R26 · Ship
- [ ] **M6** · R27-R31 · Hardened + contributed

### Per-ring build status
- [ ] R1 · *(learning only, no build)*
- [ ] R2 · *(learning only, no build)*
- [ ] R3 · *(learning only, no build)*
- [ ] R4 · `anchor init` + math primitives + error enum *(structs typed; tests still owed)*
- [ ] R5 · `PerpPosition` + quote-collateral spine + `initialize_user` *(struct typed; account + ix + test still owed)*
- [ ] R6 · `Order` struct + `place_perp_order` *(structs typed; ix + tests still owed; in lockstep with learning R6)*
- [ ] R7 · Dutch auction + stub fill + filler reward
- [ ] R8 · `update_position_and_market` — **🏁 M1**
- [ ] R9 · Full vAMM + swap_base_asset
- [ ] R10 · Pyth integration + read-only keeper vs real Drift
- [ ] R11 · Static spreads
- [ ] R12 · Dynamic spreads (inventory + volatility + revenue retreat)
- [ ] R13 · Fee pool + tiered fees
- [ ] R14 · Funding rate + funding-crank keeper — **🏁 M2**
- [ ] R15 · real SpotMarket/vault deposit/withdraw replacing quote-collateral internals
- [ ] R16 · PnL calc (AMM + oracle)
- [ ] R17 · Per-position margin + IMF
- [ ] R18 · Portfolio margin aggregation — **🏁 M3**
- [ ] R19 · `settle_pnl`
- [ ] R20 · `liquidate_perp` + liquidator keeper
- [ ] R21 · Insurance fund + social loss — **🏁 M4**
- [ ] R22 · Repeg + K update
- [ ] R23 · Matching engine *(AMM JIT optional)*
- [ ] R24 · LP *(mandatory slim; no rebase)*
- [ ] R25 · Market lifecycle study *(implemented in R26)*
- [ ] R26 · Guard rails + remaining order types + events + CI + deployment — **🏁 M5**
- [ ] R27 · Security regression tests *(parallel · sequential after R21)*
- [ ] R28 · Fuzz harnesses + regression vectors
- [ ] R29 · 3 patched audit findings + regression tests — **🏁 M6-A**
- [ ] R30 · Live keeper 30d + public dashboard *(parallel · background clock starts at R20)*
- [ ] R31 · `keeper-bots-v2` PR ladder or disclosed bounty *(starts R14, completes M6-B)*

---

*Last updated: 2026-04-28 · Owner: veerbal1 · Companion docs: `../learning/master_map.md`, `../learning/learning_state.md`*
