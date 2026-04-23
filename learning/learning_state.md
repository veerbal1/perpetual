# LEARNING STATE

## Current Position
- Ring: 6 of 26
- Active Concept: 25-31 (Order struct, order types, lifecycle, reduce-only, post-only, IOC)
- Status: IN PROGRESS — delivering in 6 chunks. Chunk 1/6 delivered 2026-04-21 (the spine: Order vs PerpPosition sign-encoding split, why direction lives in its own enum on Order). Awaiting "next" for Chunk 2/6 (the 5 OrderType flavors).
- **PAUSED 2026-04-21** while user backfills R1-R5 build deliverables. Resume R6 learning + R6 build in lockstep once catch-up ships.

## Build Track
- Repo: `~/Documents/learning/perpetual/` (separate git repo, sibling to `protocol-v2/`). Initialized 2026-04-21 as empty `git init`.
- Stakes: single portfolio project targeting Drift / Jupiter / top-tier perp DEX hire. User is full-time on this.
- Four tracks: (A) `mini-drift` Anchor program R5-R21 core with test vectors lifted from Drift's source for AMM / auction / funding / margin / liquidation math; (B) `drift-keeper` TS bot vs REAL Drift devnet from R10 onward; (C) `mini-drift-web` React demo UI on Vercel from R8 onward; (D) design journal + 3 blog posts + Loom videos at each milestone.
- Milestones: M1 R5-R8 "can trade", M2 R9-R14 "real pricing", M3 R15-R18 "full risk", M4 R19-R21 "can liquidate", M5 "polish + ship".
- Quality bar per ring: compiles + tests pass + clippy clean + custom error enum + small commits. Hard-math rings MUST have test vectors matching Drift's actual outputs.
- Detailed plan lives in `drift-build/README.md` (user-authored, tutor-guided). DO NOT duplicate the plan in this file — keep this as a pointer only.
- Current build ring: R1 (catch-up). Next build deliverable: `git init` the repo, then R1 README skeleton + `docs/rings/R01-what-is-a-perp.md` in user's own words.

## Ring 6 Chunk Plan
- [x] 1/6 — Spine: Order vs PerpPosition sign encoding split; PositionDirection enum payoff
- [ ] 2/6 — The 5 OrderType flavors (Market / Limit / TriggerMarket / TriggerLimit / Oracle) with a comparison table and concrete scenarios per flavor
- [ ] 3/6 — Order struct anatomy + OrderParams (what user sends vs what gets stored); worked example "buy 2 SOL limit at $185"
- [ ] 4/6 — Lifecycle: OrderStatus (Init → Open → Filled / Canceled); note there is NO Expired terminal state — max_ts triggers cancellation
- [ ] 5/6 — The three toggles: reduce_only (bool), post_only (FOUR variants — plot twist; None/MustPostOnly/TryPostOnly/Slide), IOC (lives inside bit_flags — not a bool; OrderParamsBitFlag::ImmediateOrCancel = 0b00000001)
- [ ] 6/6 — Single ring-close scenario check (not a drill)

## ACTIVE STYLE DIRECTIVE (supersedes all prior style notes)
- Set 2026-04-20 mid-Ring-5. User reported HEADACHES from constant atomic-Socratic questioning ("answering a lot of questions after each interaction").
- NEW DEFAULT: deliver each ring as a FULL PICTURE upfront — framed, with concrete examples, tables, and a source-file pointer. Cover the whole ring's concept space in one teaching block.
- Ask at most ONE short check question at the END of the ring, only when needed to confirm a non-obvious piece landed before progressing.
- DO NOT fire atomic questions mid-ring unless the user explicitly asks to be drilled. Earlier "atomic Socratic default" (Rings 2-4) is SUPERSEDED.
- CHUNKING RULE (added 2026-04-20, mid-Ring-5 review): for LONG walkthroughs (e.g. full-diagram reviews, multi-concept syntheses), split into numbered chunks (e.g. "1 / 8") and deliver ONE chunk per turn. End each chunk with "say 'next' when ready" — user paces. Do NOT dump 2000+ word explainers in one message even when style is "full picture." Full picture per ring is fine; full picture per SINGLE concept is better for reviews.
- Still forbidden per CLAUDE.md: writing full implementation code, auto-solving user code, creating files beyond master_map.md / learning_state.md. Pseudocode + tiny syntax snippets still allowed.

## Progress Overview

### PHASE 1 - THE WORLD
- [x] Ring 1: Trading, futures, perpetual, exchange -- COMPLETED
- [x] Ring 2: Long, short, PnL, collateral, margin, leverage -- COMPLETED

### PHASE 2 - THE ENGINE
- [x] Ring 3: AMM, constant product, price from reserves, peg -- COMPLETED
- [x] Ring 4: Solana accounts, four pillars, precision, safe math -- COMPLETED

### PHASE 3 - THE TRADE
- [x] Ring 5: PerpPosition struct, base/quote, direction -- COMPLETED 2026-04-20
- [ ] Ring 6: Order struct, types, lifecycle, reduce-only, post-only -- UP NEXT
- [ ] Ring 7: Dutch auctions, keepers, fill flow, fees-on-fill
- [ ] Ring 8: Position updates: open/increase/decrease/close

### PHASE 4 - THE PRICING ENGINE
- [ ] Ring 9: AMM internals: reserves, sqrt_k, exposure, OI, bounds
- [ ] Ring 10: Oracles, price feeds, TWAP, strict pricing
- [ ] Ring 11: Spreads: what they are, bid/ask, asymmetry
- [ ] Ring 12: Living spreads: inventory, volatility, revenue retreat

### PHASE 5 - ONGOING COSTS
- [ ] Ring 13: Fee flow, accumulation, pools, total_fee_minus_distributions
- [ ] Ring 14: Funding rate, mark vs oracle, asymmetry, capping

### PHASE 6 - RISK
- [ ] Ring 15: Spot as collateral: SpotPosition, scaled_balance, get_token_value
- [ ] Ring 16: PnL calculation: exit - entry, AMM and oracle valuation
- [ ] Ring 17: Per-position margin: types, ratios, IMF, size premium
- [ ] Ring 18: Portfolio margin: PnL weights, aggregation, total collateral
- [ ] Ring 19: PnL settlement: pool, +/-, imbalance, margin check
- [ ] Ring 20: Liquidation: triggers, partial, fees, statuses
- [ ] Ring 21: Bankruptcy, social loss, insurance fund, contract tiers

### PHASE 7 - AMM HEALTH
- [ ] Ring 22: Repeg, K updates, AMM JIT
- [ ] Ring 23: Matching engine, fulfillment methods, crossing

### PHASE 8 - ADVANCED
- [ ] Ring 24: LP system: shares, sqrt_k, settlement, rebase
- [ ] Ring 25: Market lifecycle: statuses, pauses, expiry, delisting
- [ ] Ring 26: Special modes & infrastructure: high leverage, isolated, prediction, guard rails, events, full spot deep dive, DEX

## Mastery Notes

### Ring 1
- User knows long/short conceptually at a basic/UI level but does NOT actively trade perps. Do not assume trader intuition for PnL math, funding, liquidations, etc. Verify from scratch.
- Initial sticking point: "how can someone sell a promise for something they don't have?" -- landed once reframed as "a recorded bet in a database whose value tracks an external price." The word "promise" confused them; "bet" / "agreement" stuck.
- BREAKTHROUGH at Ring 1 close: user independently derived the counterparty matching problem -- "if I'm long $50, someone must be short $50; how does the protocol find exact opposite amounts among thousands of traders?" This IS the motivating question for the entire AMM (Rings 3, 9-12) and matching engine (Ring 23). They think at the system-design level, not just mechanics -- lean into this.
- Analogy that worked: "rain is independent, it happens on its own" -- user used this to explain that the underlying asset's price is separate from the bet. Reuse this framing when we introduce oracles (Ring 10).
- User prefers the word "bet" over "promise" -- stick with "bet" / "agreement" going forward.

### Ring 2
- Entered Ring 2 with the stock-market short-selling mental model ("Bob borrowed BTC, sold at $50k, rebought at $45k, kept the difference"). Classic perp-learner trap. Corrected by asking them to spot the contradiction with Ring 1 ("if longs don't touch real BTC, why would shorts?"). Landed cleanly.
- Ideas 5 (long/short) and 6 (PnL) now locked in. User understands: direction is just a sign, PnL flows between long and short as price moves, no asset is ever physically exchanged.
- Moving to idea 7 (collateral). Per the plan, this is where Ring 2 slows down -- collateral/margin/leverage are the genuinely new material for this user.
- Ring 2 closed out cleanly. User tends to give TERSE but CORRECT answers ("10,000", "trader a still alive"). Don't mistake brevity for shallow understanding -- they track the math, they just don't always spell it out. When they're terse, fill in the math on paper for the record, then keep moving.
- User asked to skip the collateral/margin check question ("i understand, let's move") but passed the follow-up leverage-math verification cleanly. They're pacing themselves; trust it but still verify at ring boundaries.
- STYLE PREFERENCE (explicit, mid-Ring-3): user wants pure atomic Socratic -- one variable per question, no multi-part questions, no explanation dumps, no recapping lists of ideas. Ask, wait, push one layer deeper. Don't spoon-feed answers. If they use jargon, make them define it first. Keep the ring framework (required by CLAUDE.md) but tighten style sharply within it.

### Ring 3
- User independently derived FOUR major concepts: (1) x*y=k with y/x = price, correctly mapping y=USDC, x=BTC; (2) virtual reserves insight -- "AMM does not actually have any BTC reserve, just collateral" -- caught this on their own, usually a taught concept not a derived one; (3) arbitrage as a gap-closer -- "won't traders long to capture the 50k vs 55k gap?"; (4) peg-induced instant PnL redistribution -- "if peg moves, longs win instantly and shorts lose instantly."
- Teaching pattern that worked: stress-test the user's "simple oracle-only AMM" hypothesis with a concrete pile-in scenario ($100 long x 1M traders x 20% move = $20M owed). Made them FEEL why a curve is needed. They derived the mechanism from the pain.
- CONFUSION MODE: user shuts down with "no idea what you are telling" when a scenario has too many variables at once (1M traders + $5M + oracle ticks in one question). Fix: one variable at a time. One trader, one move. Scale up in separate turns.
- Self-aware: user explicitly flagged "partially understand, not solid" and later "what if there are hidden parts I don't know" -- honor this, do audits at ring close. They prefer honesty over false completion.
- Mini-callbacks landed well: when user asked "is this x*y?" mid-derivation, parking the formula for one more atomic step ("price moves which direction?") forced them to verbalize the mechanism BEFORE seeing the formula name. Do this again.
- Vocabulary introduced at Ring 3 close: `mark_price` (the AMM's quoted price = (y/x) * peg_multiplier), `slippage` (teaser -- own trade moves price; full math Ring 9).
- Forward refs planted for Rings 9 (price impact), 10 (oracles), 11 (bid/ask spread), 14 (funding rate), 22 (repeg cost/triggers), 24 (where virtual reserves come from / LPs). Don't let these become vapor -- resurface each when its ring opens.

### Ring 5 (delivered & closed 2026-04-20)
- Delivered as full-picture lecture per new style directive (see ACTIVE STYLE DIRECTIVE at top). Single end-of-ring check: "short 5 SOL at $200 → what are base and quote?" User answered instantly: "-5, +1000". Clean close, no wobble.
- New style (full picture + single check) REPLACES the atomic-Socratic default going forward. User explicitly reported headaches from prior constant questioning — respect the shift.
- The full-picture delivery included: (a) PnL math showing only entry/size/direction matter, (b) signed encoding trick (base carries size AND direction), (c) quote = cash side with OPPOSITE sign, IN/OUT convention, (d) quote stores NOTIONAL not collateral (collateral lives in SpotPosition, Ring 15), (e) quote_entry_amount vs quote_break_even_amount (equal at open, diverge with fees — forward ref Ring 7), (f) PositionDirection enum exists for Order flow (Ring 6) but NOT on PerpPosition.
- Source file revealed: `programs/drift/src/state/user.rs` (PerpPosition struct).
- User's comfort zone with this style: confirmed. Continue pattern for Ring 6+.

### Ring 5 (prior mid-flight attempt — SUPERSEDED by 2026-04-20 fresh delivery; kept below as behavioral/style pattern data only)
- Opening move: "minimum info to compute PnL a week later" -- user initially listed collateral + leverage + entry + direction. Used the PnL arithmetic check ($200 on long 10 SOL, $100->$120) to make them SEE that collateral/leverage didn't appear in the formula. They self-identified entry + direction as the ones they actually used. Worked cleanly.
- Size was the hidden one -- they implicitly used "× 10" but didn't list "size" originally. Caught it when asked where the 10 came from. Small gap, fast fix.
- Signed encoding (+10 long, -10 short): derived instantly and unprompted. "+means long, - means short" -- no resistance.
- Quote-as-notional-not-collateral: user asked the right question ("is quote direct collateral or leverage included amount?") -- that's a real design-level instinct. Framed the $100 vs $1000 scenario; they picked $1000 correctly.
- Sign of quote: FIRST INSTINCT WRONG -- guessed long=+quote, short=-quote. Before pushing further, user flagged CONFUSION explicitly: "im not seeing full picture... we studying in chunks and a lot of questions. Do you think about this?" -- good self-advocacy. Honor it.
- RESPONSE: zoomed out and delivered the full two-field layout as a table (base_asset_amount + quote_asset_amount, signed, opposite signs, IN=+ OUT=-). Did NOT force the derivation when they were confused. This user tolerates atomic style well on solid ground (Rings 3-4) but asks for the map when genuinely lost. Track this -- pattern is "atomic until stuck, then give the frame."
- STYLE CALIBRATION: atomic Socratic works, but not as a religion. When user says "I'm confused / missing context," deliver the frame immediately, then return to atomic. Mid-ring reframes are fine -- they are NOT admissions of failure, they are part of the method for this user.
- Still pending in Ring 5: (a) confirm the sign-flip intuition is now locked (ask a short verification), (b) introduce quote_entry_amount vs quote_break_even_amount (they differ because of fees; forward ref Ring 7), (c) note that PositionDirection enum exists for ORDER flow but is NOT stored on PerpPosition (the signed base IS the direction storage).

### Ring 4
- User entered with working Solana baseline: already knew PDAs, program ownership, lamports-as-SOL-atomic-unit. Did NOT need to teach these -- confirmed and leveraged directly.
- Derived fixed-point representation unprompted: "multiply by a large scaler, store as big int, divide back on use." One atomic push was enough.
- Also derived the tradeoff unprompted: "bigger scaler = more precision, more bytes" -- correct. Add overflow risk as my own overlay.
- Nailed the scaler-matches-native-unit insight: "SOL = 10^9 lamports, can't split further, so BASE_PRECISION = 10^9." This is the real reason BASE ≠ QUOTE precision; they got it on first ask.
- Caught the numerical guardrail: when I gave price × size ~ 10^15, they noted "this will not overflow" before I baited them into it. Track this -- they check magnitudes, don't just trust handed numbers.
- Knew `checked_*` functions by name. Also independently proposed "don't calc everything in one line, break into 2-at-a-time intermediates" -- that's a real Drift pattern. Confirmed their instinct.
- Minor calibration: they said `checked_mul` "throws error." Actual return is Option<T>; error conversion via `ok_or(ErrorCode)?`. Noted but did not belabor.
- Pacing: Ring 4 moved fast (6-7 atomic beats total). Continue this tempo when user has existing adjacent knowledge; slow back down on genuinely new material (Ring 5 entry-vs-break-even, Ring 7 auctions, etc.).
- Four pillars delivered as a table at the end -- user grouped fine. No confusion about State vs PerpMarket vs SpotMarket vs User. They correctly placed AMM reserves in PerpMarket on first ask.

## Vocabulary Unlocked
- trading, buying, selling
- future (a bet on what a price WILL be)
- perpetual (a bet that never expires)
- exchange / marketplace / middleman (the program that holds money, tracks bets, forces payouts)
- bet / agreement (the record that tracks a position's value)
- long (bet that price goes up)
- short (bet that price goes down)
- PnL / profit and loss (money flowing between sides as price moves; drains from loser's collateral, adds to winner's)
- collateral (locked deposit guaranteeing you can pay if you lose; USDC on Drift)
- margin / margin requirement (required collateral-to-size ratio, enforced before a bet can open; stored as margin_ratio_initial per market)
- leverage (bet size / collateral ratio; inverse of margin; e.g. 5% margin = up to 20x leverage)
- liquidation (teaser only: forcible bet closure before collateral hits zero; full topic in Ring 20)
- AMM (automated market maker -- always-available robot counterparty)
- reserves / virtual reserves (x = base, y = quote; purely accounting numbers on a perp AMM, no actual BTC anywhere)
- constant product / x*y=k (the curve: as one reserve drops, the other must rise to keep product constant; price never runs out, just gets arbitrarily expensive)
- liquidity (implicitly: the size of the reserves; thicker reserves = less price impact per trade)
- peg / peg_multiplier (scalar knob in the price formula: mark_price = (y/x) * peg_multiplier; aligns AMM baseline to real-world oracle price)
- mark price (the AMM's quoted price from its formula)
- slippage (teaser only: own trade moves the price against you; full math Ring 9)
- arbitrage (user-derived: traders capture the AMM-oracle gap, which also closes the gap)
- funding rate (teaser only: indirect ongoing gap-closer; full topic Ring 14)
- repeg / repeg cost (teaser only: peg updates cost the AMM when net-exposed, paid from fee pool; full mechanics Ring 22)
- account (the universal Solana storage unit; everything -- wallets, tokens, program state -- is an account)
- PDA (program-derived address; account owned by a program with no private key, program writes freely)
- four pillars (Drift's top-level account layout: State singleton, PerpMarket per perp, SpotMarket per collateral token, User per trader subaccount)
- State / PerpMarket / SpotMarket / User (the four account types; PerpMarket holds the AMM reserves + peg + sqrt_k for its market)
- fixed-point / scaler (store decimal numbers as integers by multiplying by a fixed power of 10)
- PRICE_PRECISION = 10^6 (Drift's standard multiplier for prices)
- BASE_PRECISION = 10^9 (matches SOL's native 10^9 lamport granularity)
- QUOTE_PRECISION = 10^6 (matches USDC's native 10^6 atomic granularity)
- overflow / silent wrapping (u64 max ~1.8×10^19; Rust release-mode overflow wraps silently -- catastrophic for money math)
- checked_mul / checked_add / checked_sub / checked_div (arithmetic that returns Option<T>; None on overflow)
- ok_or(ErrorCode::MathError)? (Drift's idiom: convert Option → Result → bubble up)
- u64 (the 64-bit unsigned integer type; holds up to ~1.8×10^19)
- position / PerpPosition (the stored record of a bet; lives on the User account)
- base_asset_amount (signed i64 on PerpPosition; sign = direction, magnitude = size in base-asset atomic units)
- quote_asset_amount (signed i64 on PerpPosition; the cash side of the swap; always OPPOSITE sign to base; stores NOTIONAL not collateral)
- signed encoding (collapsing "size + direction" into a single signed integer; Drift uses this across position records)
- IN/OUT sign convention (from the position's perspective: received = +, gave up = −; applies to both base and quote)
- notional (size × entry price; what was exchanged in the swap, NOT what was deposited)
- quote_entry_amount (quote snapshot at open, BEFORE any fees)
- quote_break_even_amount (the quote level needed to net-zero including fees paid; equals entry at open, drifts as fees accumulate — Ring 7)
- entry price (derivable: quote_entry_amount / base_asset_amount)
- break-even price (derivable: quote_break_even_amount / base_asset_amount)
- PositionDirection enum (Long/Short; used on the Order struct in Ring 6; NOT stored on PerpPosition — sign of base_asset_amount IS the direction there)

## Next Up
- Ring 6 — Placing your bet. Concepts 25-31: Order struct, OrderType enum (Market / Limit / TriggerMarket / TriggerLimit / Oracle — the Oracle type sources price from a feed, forward ref Ring 10), OrderParams (what the user submits), Order lifecycle (Open → Filled / Canceled / Expired), reduce-only (can only shrink a position), post-only (must rest as maker — full maker/taker distinction Ring 23), IOC (immediate-or-cancel).
- Delivery: full-picture per ACTIVE STYLE DIRECTIVE. Frame the motivating problem (you have a position record from Ring 5 — how do you CREATE one?), give a summary table of order types, walk the lifecycle, explain the three toggles (reduce-only / post-only / IOC) concretely, point at source files (`programs/drift/src/state/user.rs` for Order struct, `state/order_params.rs` for OrderParams). End with ONE retention check, not a drill.
- Vocabulary to unlock during Ring 6: order, limit, market order, trigger, reduce-only, post-only, IOC, OrderType, OrderParams, OrderStatus, maker / taker (teaser — full mechanics Ring 23), Oracle-type order (teaser — price source Ring 10).
- Forward refs to plant during Ring 6: Ring 7 (Dutch auction gives market orders a price, taker/maker fees on fill), Ring 10 (Oracle order type's price source), Ring 23 (maker/taker matching engine).
- Forward refs still outstanding from earlier rings: Ring 7 (entry vs break-even via fees — hit it in Ring 7), Ring 9 (price impact, sqrt_k, open interest), Ring 10 (oracle mechanics), Ring 11 (bid/ask spread), Ring 14 (funding rate, last_cumulative_funding_rate on PerpPosition), Ring 22 (repeg), Ring 24 (LPs / virtual reserve origination).
