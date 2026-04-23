# LEARNING STATE

## Current Position
- **Ring: 1 of 31** (learning spiral = 26; build-only rings 27–31 for adversarial + real-world phases)
- **Active Concept:** 1–4 (trading, futures, perpetual, exchange)
- **Status:** FRESH START — reset 2026-04-23. Prior R1–R5 learning progress wiped by user choice. Behavioral calibration + vocabulary preserved below (tutor memory, not progress).
- **Mode (new, per updated CLAUDE.md):** learning + building in lockstep. Every ring ships BOTH a concept understanding AND a code artifact from `drift-build/README.md`. User writes every line of code; tutor guides with "now write X" prompts.

---

## Build Track (pointer only — full plan lives in `drift-build/README.md`)

- **Repo:** `~/Documents/learning/perpetual/mini-drift/` (to be `git init`'d as part of R1 build deliverable).
- **Stakes:** single portfolio project targeting Drift / Jupiter / top-tier perp DEX engineering hire. User is full-time on this.
- **Four tracks:**
  - (A) `mini-drift` Anchor program — activates R5, ships through R26
  - (B) `drift-keeper` TS bot against **real Drift devnet** — activates R10
  - (C) `mini-drift-web` React UI — activates R8
  - (D) Content: per-ring `docs/rings/RNN-*.md`, 4 milestone blogs, 3 Looms, **daily Twitter build-in-public**
- **Milestones:** M1 R5–R8 (can trade) · M2 R9–R14 (real pricing) · M3 R15–R18 (full risk) · M4 R19–R21 (can liquidate) · M5 R22–R26 (ship) · M6 R27–R31 (hardened + contributed).
- **Quality bar per ring:** compiles clean · `cargo clippy -- -D warnings` · tests pass · custom error enum · atomic commits. Hard-math rings MUST have test vectors matching Drift's actual outputs (🎯 markers in the build README).
- **Current build ring:** **R1** (fresh start; lockstep with learning R1). Next build deliverable: `git init` the repo, root `README.md` skeleton, `docs/rings/R01-what-is-a-perp.md` in user's own words.

---

## ACTIVE STYLE DIRECTIVE (preserved across reset — supersedes all prior style notes)

- **Set 2026-04-20 mid-Ring-5.** User reported HEADACHES from constant atomic-Socratic questioning. This directive replaced the atomic-Socratic default from Rings 2–4.
- **DEFAULT delivery:** each ring = a FULL PICTURE upfront — framed, with concrete examples, tables, and source-file pointers. Cover the whole ring's concept space in one teaching block.
- **Check question:** at most ONE short check question at the END of the ring, only when needed to confirm a non-obvious piece landed before progression.
- **No atomic drilling:** do NOT fire rapid one-variable questions mid-ring unless the user explicitly asks to be drilled.
- **Chunking rule:** for LONG walkthroughs (full-diagram reviews, multi-concept syntheses), split into numbered chunks ("1 / 8") and deliver ONE chunk per turn. End each chunk with "say 'next' when ready" — user paces. No 2000+ word explainers in one message. Full picture per ring is fine; full picture per SINGLE concept is better for reviews.
- **Build phases (NEW — added 2026-04-23 with build lockstep):** use the **"Now write X"** prompt format. State the goal, name the file, name the connection to prior rings (concept-side AND code-side), then wait while user writes. Review their diff. NEVER write their code. If stuck, climb the hint ladder from CLAUDE.md EDGE CASES (reframe → prior ring → plain English → pseudocode as last resort).
- **Forbidden per CLAUDE.md:** writing any code (Rust / TS / config / tests / docs), creating files beyond the three managed files (`master_map.md`, `learning_state.md`, `drift-build/README.md`), auto-solving user's code.

---

## Progress Overview

### PHASE 1 — THE WORLD
- [ ] **Ring 1**: Trading, futures, perpetual, exchange — **UP NEXT**
- [ ] Ring 2: Long, short, PnL, collateral, margin, leverage

### PHASE 2 — THE ENGINE
- [ ] Ring 3: AMM, constant product, price from reserves, peg
- [ ] Ring 4: Solana accounts, four pillars, precision, safe math

### PHASE 3 — THE TRADE
- [ ] Ring 5: PerpPosition struct, base/quote, direction
- [ ] Ring 6: Order struct, types, lifecycle, reduce-only, post-only
- [ ] Ring 7: Dutch auctions, keepers, fill flow, fees-on-fill
- [ ] Ring 8: Position updates: open/increase/decrease/close — 🏁 **M1**

### PHASE 4 — THE PRICING ENGINE
- [ ] Ring 9: AMM internals: reserves, sqrt_k, exposure, OI, bounds
- [ ] Ring 10: Oracles, price feeds, TWAP, strict pricing
- [ ] Ring 11: Spreads: what they are, bid/ask, asymmetry
- [ ] Ring 12: Living spreads: inventory, volatility, revenue retreat

### PHASE 5 — ONGOING COSTS
- [ ] Ring 13: Fee flow, accumulation, pools
- [ ] Ring 14: Funding rate, mark vs oracle, asymmetry, capping — 🏁 **M2**

### PHASE 6 — RISK
- [ ] Ring 15: Spot as collateral: SpotPosition, scaled_balance
- [ ] Ring 16: PnL calculation
- [ ] Ring 17: Per-position margin: IMF, size premium
- [ ] Ring 18: Portfolio margin: aggregation, total collateral — 🏁 **M3**
- [ ] Ring 19: PnL settlement
- [ ] Ring 20: Liquidation
- [ ] Ring 21: Bankruptcy, social loss, insurance fund — 🏁 **M4**

### PHASE 7 — AMM HEALTH
- [ ] Ring 22: Repeg, K updates, AMM JIT
- [ ] Ring 23: Matching engine, fulfillment methods

### PHASE 8 — ADVANCED
- [ ] Ring 24: LP system (optional)
- [ ] Ring 25: Market lifecycle: statuses, pauses, expiry
- [ ] Ring 26: Guard rails, events, CI, deployment — 🏁 **M5**

### PHASE 9 — ADVERSARIAL (build-only; extends learning spiral)
- [ ] Ring 27: Threat model (15+ attack scenarios)
- [ ] Ring 28: Fuzzing (cargo-fuzz harnesses)
- [ ] Ring 29: Audit react-blog + 3 patched findings — 🏁 **M6-A**

### PHASE 10 — REAL-WORLD SIGNAL (build-only; extends learning spiral)
- [ ] Ring 30: Live keeper 30-day uptime + dashboard (background clock starts at R20)
- [ ] Ring 31: Merged PR to `drift-labs/keeper-bots-v2` OR disclosed bounty — 🏁 **M6-B**

---

## Mastery Notes (preserved across reset — tutor calibration from prior run)

These describe HOW this user learns best. They are not progress. Keep them referenced at session start and apply when teaching.

### Prior Ring 1
- User knows long/short at a basic/UI level but does NOT actively trade perps. Don't assume trader intuition for PnL math, funding, liquidations. Verify from scratch.
- Sticking point resolved by reframing "promise" as **"bet" / "agreement in a database whose value tracks an external price."** User prefers "bet" and "agreement" over "promise" — use that vocabulary.
- BREAKTHROUGH at Ring 1 close: user independently derived the counterparty matching problem ("if I'm long $50, someone must be short $50 — how does the protocol match exact opposites among thousands?"). This is the motivating question for the entire AMM + matching engine. **They think at system-design level, not just mechanics — lean into this.**
- Analogy that worked: "rain is independent, it happens on its own" — reuse when introducing oracles (Ring 10).

### Prior Ring 2
- Entered with stock-market short-selling mental model (borrow-and-sell). Classic perp-learner trap. Corrected by asking them to spot the contradiction with Ring 1.
- User is **TERSE BUT CORRECT** ("10,000", "trader a still alive"). Don't mistake brevity for shallow understanding — track the math, fill it in on paper for the record, keep moving.
- Paced themselves with "I understand, let's move"; passed verification anyway. Trust self-pacing but still verify at ring boundaries.

### Prior Ring 3
- User independently derived FOUR major concepts: (1) x\*y=k with y/x = price (correctly mapping y=USDC, x=BTC); (2) virtual reserves ("AMM does not actually have any BTC reserve, just collateral" — derived, not taught); (3) arbitrage as gap-closer; (4) peg-induced instant PnL redistribution.
- **Teaching pattern that works:** stress-test their hypothesis with a concrete scale-up scenario ("$100 long × 1M traders × 20% move = $20M owed"). They derive from feeling the pain.
- **CONFUSION MODE:** user shuts down with "no idea what you are telling" when a scenario has too many variables at once. Fix: one variable at a time, one trader, one move. Scale up in separate turns.
- Self-aware — flags "partially understand, not solid" and "what if there are hidden parts I don't know." Honor this. Audit at ring close.

### Prior Ring 4
- **User has Solana baseline:** PDAs, program ownership, lamports-as-atomic-unit of SOL. Do NOT re-teach these.
- Derived fixed-point representation unprompted: "multiply by scaler, store as big int, divide on use." Derived the tradeoff unprompted: "bigger scaler = more precision, more bytes."
- Knew `checked_*` functions by name. Independently proposed "break into 2-at-a-time intermediates" — matches Drift pattern.
- Pacing: Ring 4 moved fast (6–7 atomic beats total). Continue fast tempo when user has adjacent knowledge; slow down on genuinely new material.

### Prior Ring 5
- Delivered as full-picture lecture (2026-04-20). Single end-of-ring check: "short 5 SOL at $200 → base and quote?" User answered "-5, +1000" instantly. Clean close.
- This is where the current style directive was cemented. User explicitly confirmed the full-picture-per-ring + single-check format works.

---

## Vocabulary Unlocked (preserved — user already knows these terms)

These are banked from the prior run. Don't re-teach at "Ring 1 beginner" level. Ring 1 this time around is a re-anchor + first build deliverable, not a true beginner introduction.

- trading, buying, selling
- future (a bet on what a price WILL be)
- perpetual (a bet that never expires)
- exchange / marketplace / middleman (the program that holds money, tracks bets, forces payouts)
- bet / agreement (the preferred framing for a position)
- long (bet that price goes up)
- short (bet that price goes down)
- PnL / profit and loss
- collateral (locked deposit guaranteeing payout on loss; USDC on Drift)
- margin / margin requirement (required collateral-to-size ratio)
- leverage (bet size / collateral ratio)
- liquidation (teaser only — Ring 20)
- AMM (automated market maker — always-available robot counterparty)
- reserves / virtual reserves (x = base, y = quote; purely accounting numbers on a perp AMM)
- constant product / x\*y=k
- liquidity
- peg / peg_multiplier (mark_price = (y/x) × peg_multiplier)
- mark price
- slippage (teaser only — Ring 9)
- arbitrage (user-derived)
- funding rate (teaser only — Ring 14)
- repeg / repeg cost (teaser only — Ring 22)
- account (Solana storage unit)
- PDA (program-derived address)
- four pillars (State singleton, PerpMarket, SpotMarket, User)
- State, PerpMarket, SpotMarket, User
- fixed-point / scaler
- PRICE_PRECISION = 10^6
- BASE_PRECISION = 10^9
- QUOTE_PRECISION = 10^6
- overflow / silent wrapping
- checked_mul / checked_add / checked_sub / checked_div
- `ok_or(ErrorCode)?`
- u64
- position / PerpPosition
- base_asset_amount (signed i64; sign = direction, magnitude = size)
- quote_asset_amount (signed i64; always OPPOSITE sign to base; stores NOTIONAL not collateral)
- signed encoding (direction + size collapsed into one signed integer)
- IN/OUT sign convention (received = +, gave up = −)
- notional (size × entry price)
- quote_entry_amount (snapshot at open, before fees)
- quote_break_even_amount (level needed to net-zero including fees)
- entry price, break-even price (derivable)
- PositionDirection enum (Long/Short; used on Order, NOT on PerpPosition)

---

## Next Up

### Learning side — Ring 1 re-anchor
- Concepts 1–4: trading, futures, perpetual, exchange.
- Format: full-picture per ACTIVE STYLE DIRECTIVE. Since vocabulary above is banked, this is a **fast re-anchor** — frame the 4 pieces as a system ("marketplace + agreement + never-expiring + middleman"), emphasize the counterparty-matching motivating question the user independently derived last time, end with ONE verification question.
- Do NOT belabor. User is not a beginner.

### Build side — Ring 1 first code-adjacent deliverable
At Ring 1 learning close, **pivot immediately to the Ring 1 Build section of `drift-build/README.md`** — this is the new lockstep mode. Specifically:
1. Prompt: *"Now run `git init` inside `~/Documents/learning/perpetual/mini-drift/`."*
2. Prompt: *"Now write the root `README.md`."* Give them the skeleton structure from `drift-build/README.md` §5 (vision, monorepo layout, empty milestone checkboxes), but they write it.
3. Prompt: *"Now write `docs/rings/R01-what-is-a-perp.md`."* 800 words in their own words. This is the writing-reps muscle memory for the interview talking points later.
4. Close Ring 1 in both tracks: tick checkbox in this file AND in `drift-build/README.md` §11. Then preview Ring 2's connection to Ring 1.

### Session-level open for the new chat
When the user opens a fresh session, greet per CLAUDE.md INITIALIZATION PROTOCOL: confirm you've read all three files (`learning/master_map.md`, `learning/learning_state.md`, `drift-build/README.md`), note that this is a post-reset re-anchor at Ring 1 with build lockstep, and ask: *"Ready to re-anchor Ring 1?"*
