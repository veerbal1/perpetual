# LEARNING STATE

## Current Position
- Ring: 8 of 26
- Active Concept: Ring 8 opening next — position mutation boundary after Ring 7's fill helpers. Ring 7 closed on 2026-05-14 as a Drift-shaped helper shell, with real position update deliberately owned by Ring 8.
- Status: READY TO OPEN RING 8 - Ring 7 helper build is closed with a clean boundary. Completed Drift-mirrored helper slices: `Order::get_base_asset_amount_unfilled`, `Order::update_open_bids_and_asks`, `controller/orders.rs::update_order_after_fill`, `controller/position.rs::decrease_open_bids_and_asks` with update flag and zero clamps, `User::decrement_open_orders`, focused fill receipt builder slice (`OrderAction`, `OrderActionExplanation`, `OrderActionRecord`, `get_order_action_record`), post-receipt counter cleanup helper `decrement_open_orders_after_full_fill`, Drift-shaped `FillMode` with `is_liquidation` / `is_ioc`, `FillMode::get_limit_price`, `state::fulfillment::PerpFulfillmentMethod`, `math::matching::do_orders_cross`, `math::matching::are_orders_same_market_but_different_sides` (perp-only simplification without `market_type`), `math::matching::is_maker_for_taker`, `math::orders::calculate_quote_asset_amount_for_maker_order`, `math::matching::calculate_fill_for_matched_orders`, `math::orders::should_expire_order`, Drift-shaped `math::orders::should_cancel_reduce_only_order`, and `controller/orders.rs::should_reward_keeper`. Latest assistant verification: `cargo fmt -- --check`, `cargo check`, full `cargo test` passed with 96 tests, and `cargo clippy -- -D warnings` clean on 2026-05-14. R4/R5 foundation debt remains tracked before full M1 demo. Parked boundaries: real position mutation R8, AMM reserve swap R9, oracle internals R10, full fee pools/referrers R13, full matching engine R23.

## Ring 6 Chunk Plan
- [x] 1/6 — Spine: Order vs PerpPosition sign encoding split; PositionDirection enum payoff
- [x] 2/6 — The 5 OrderType flavors (COMPLETE, sub-chunked):
  - [x] 2a — Motivation: an order needs a "rule label" called OrderType; coffee-shop analogy; five rules total (delivered 2026-04-28)
  - [x] 2b — Market rule: fill now, no price commitment, no wait condition; Dutch auction is forward-ref Ring 7 (delivered 2026-04-28)
  - [x] 2c — Limit rule: "or better" clause (ceiling for buys / floor for sells), two cases (CROSS or REST), trade-off table, maker/taker teaser for Ring 23 (delivered 2026-04-28)
  - [x] 2d — TriggerMarket: armed/triggered two-state pattern, Above/Below condition, stop-loss + take-profit examples, brief note on TriggeredAbove/TriggeredBelow as "already fired" memory state (full lifecycle treatment deferred to Chunk 4/6) (delivered 2026-04-28)
  - [x] 2e — TriggerLimit: same arming as 2d, TWO prices (trigger + limit), flash-crash failure mode (limit floor protects price BUT can strand the position), TriggerMarket vs TriggerLimit comparison table (delivered 2026-04-28)
  - [x] 2f — Oracle rule: brief aside on "oracle = live external truth price" (callback to user's own Ring 1 'rain is independent' analogy + Ring 3 peg); Oracle order = Market with auction prices as OFFSETS not absolutes; volatility/stale-price motivation; bot/market-maker use case; full oracle mechanics deferred to Ring 10 (delivered 2026-04-28)
  - [x] 2g — Synthesis: 5-row comparison table, two-family tree (ACTIVE NOW vs ARMED & WAITING; auction vs price-pinned), use-case cheat sheet, mental model "5 rules → 3 underlying machines (auction / resting-limit / armed-watcher wrapper)" (delivered 2026-04-28). 2/6 COMPLETE.
- [x] 3/6 — Order struct anatomy + OrderParams (COMPLETE, sub-chunked):
  - [x] 3a — Motivation: WHY two structs? OrderParams = request form; Order = stored record. Bank-deposit-slip analogy. Source files: `state/order_params.rs` + `state/user.rs`. Stored Orders live in User account fixed-size array (callback to Ring 4 four-pillars) (delivered 2026-04-28)
  - [x] 3b — OrderParams walkthrough: 17 fields organized into 5 clusters (Identity-7, Toggles-3, Trigger-2, Auction-3, Oracle+lifetime-2). Concrete OrderParams literal for "Limit Long 2 SOL @ $185 on SOL-PERP" with PRICE_PRECISION/BASE_PRECISION values. Toggle deep-dive deferred to Chunk 5/6, lifecycle (max_ts) deferred to Chunk 4/6 (delivered 2026-04-28)
  - [x] 3c — Order walkthrough: what's IN the stored record. Highlighted protocol-stamped fields (slot, order_id, status, base_asset_amount_filled, quote_asset_amount_filled, existing_position_direction, posted_slot_tail) NOT present in OrderParams. User locked: Params = ask, Order = ask + memory/progress, PerpPosition = one per-market net position folder. (delivered 2026-04-29)
  - [x] 3d — Side-by-side delta: what's added/normalized between submission and storage (Option<T> → defaults, post_only enum → bool, IOC bit_flag → bool). User needed extra post-only examples; locked: post-only prevents immediate fill at placement but can fill later. (delivered 2026-04-29)
  - [x] 3e — Worked example: user submits OrderParams { Limit, Long, 2 SOL, price $185, market_index SOL-PERP } → protocol stores Order { same fields + order_id, slot, status=Open, fills=0, existing_position_direction }. Added fixed-shelf examples for unfilled, partial, and filled orders. (delivered 2026-04-29)
- [x] 4/6 — Lifecycle: OrderStatus (Init → Open → Filled / Canceled); note there is NO Expired terminal state — max_ts triggers cancellation. User locked: Canceled stops remaining waiting amount but does not undo fills; expiry via max_ts becomes Canceled, not Expired; only Open orders count as waiting. (delivered 2026-04-29)
- [x] 5/6 — The three toggles: reduce_only (bool), post_only (FOUR variants — plot twist; None/MustPostOnly/TryPostOnly/Slide), IOC (lives inside bit_flags — not a bool; OrderParamsBitFlag::ImmediateOrCancel = 0b00000001). User locked: reduce-only moves toward zero and cannot flip; post-only must wait first and has None/Must/Try/Slide request styles; IOC fills now and cancels leftover. (delivered 2026-04-29)
- [x] 6/6 — Single ring-close scenario check (not a drill). User answered reduce-only correctly: short 7 against long 5 can reduce to zero but cannot open short 2. Clarified one slip: placing the order does not zero the existing position; fills move the position. (delivered 2026-04-29)

## Ring 7 Chunk Plan
- [x] 1/8 - Place vs fill recap: placing stores a waiting order; filling changes real position state.
- [x] 2/8 - Filler/keeper model: caller wakes the program; program checks rules and never trusts caller-provided safety.
- [x] 3/8 - Dutch auction setup: oracle -> market side info -> start/end boundary -> final gap -> percent gap -> duration.
- [x] 4/8 - Auction price at fill time: read saved auction fields, calculate elapsed slots, clamp elapsed by duration, move price by direction, then tick-round safely.
- [x] 5/8 - Limit/fill safety: Long fills at or below wall; Short fills at or above wall; auction price can be the temporary wall when no explicit user limit exists.
- [x] 6/8 - Partial fill storage: total order size stays fixed; `base_asset_amount_filled` tracks progress; unfilled = total - filled.
- [x] 7/8 - Drift-shaped fill update order: position update must succeed before order progress can move; open bids/asks decrease as waiting exposure fills.
- [x] 8/8 - Event mental model: fill event is a receipt emitted after successful state changes.
- [x] Gate before Build Slice 1 - R4/R5 foundation debt intentionally parked for this order-progress-only slice; it is saved for later, not ignored.
- [x] Build Slice 1 - CLOSED 2026-05-14: order-progress helpers, fill receipt shape, counter cleanup, FillMode basics, route-label/crossing/role-gate helpers, matched-fill math, cleanup decision helpers, keeper reward gate, and tests. Completed: unfilled amount including reduce-only safety, partial/full fill order progress, open bid/ask decrease with Drift-style update flag and clamps, `User::decrement_open_orders`, focused `OrderActionRecord` receipt builder for current AMM-style fill slice, `decrement_open_orders_after_full_fill` with partial/full/underflow tests, `FillMode` enum plus `is_liquidation` / `is_ioc`, `Order.slot`, `Order::has_auction`, `math::auction::calculate_auction_price`, simplified `Order::get_limit_price`, `FillMode::get_limit_price`, `state::fulfillment::PerpFulfillmentMethod`, `math::matching::do_orders_cross`, `math::matching::are_orders_same_market_but_different_sides`, `Order::is_limit_order`, `Order::is_auction_complete`, `Order::is_resting_limit_order`, `math::matching::is_maker_for_taker`, `math::orders::calculate_quote_asset_amount_for_maker_order`, `math::matching::calculate_fill_for_matched_orders`, `math::orders::should_expire_order`, Drift-shaped `math::orders::should_cancel_reduce_only_order` using `get_base_asset_amount_unfilled(... ) < step_size`, and `controller/orders.rs::should_reward_keeper`. Parked by design: emit inside a real Drift-shaped fill path and position-update success boundary; R8 opens next.
- [x] Drift-alignment gate before any new R7 flow wrapper - do not invent mini-only helpers or fake fill orchestration. Build only exact Drift helper slices, or park missing dependencies until their owning ring.
- [x] Study Slice 2 - CLOSED 2026-05-09: route-selection neurons + source-code bridge of `determine_perp_fulfillment_methods` (locked 2026-05-08 via 9-chunk Hinglish bank-scene walkthrough; user passed integration check Carol-out-of-wall → `[AMM(None)]`-only). On 2026-05-09 also locked: `OrderActionRecord` Option<T> philosophy + always-present vs sometimes-present split, header fields (ts/action/action_explanation/market_index/market_type), filler block with owner-cancel vs keeper-cancel correction (filler/filler_reward depend on who triggered, fill_record_id is fill-specific only), money flow core with `maker_fee: i64` = rebate sign neuron and Drift's `Some(-maker_rebate.cast()?)` internal trick, surface-level fees-on-fill picture, and 4-path order-death taxonomy (full fill / owner cancel / keeper expiry via `should_expire_order` / keeper reduce-only invalidation). Forward-parked with their owning rings: detailed receipt fields (`referrer_reward` Ring 13, `quote_asset_amount_surplus` Ring 9/13, `spot_fulfillment_method_fee` Ring 26, taker/maker order snapshot blocks, oracle/bit_flags/reduce-only-context/builder/trigger_price), full fee math + fee-pool splits + tiers (Ring 13), full cancel-flow code (later build slice when needed), AMM fairness question (Ring 9/11/12/13).

## Parked Questions
- 2026-05-08 — User asked: "If AMM fills a user's cheaper limit order, then later fills a higher-priced order, is that fair?" Current-slice answer belongs to Ring 7 Study Slice 2: AMM can only fill inside the user's price wall and its own route checks. Deeper answer belongs to Ring 9 (AMM reserve/price impact and inventory), Ring 11-12 (spread/protection), and Ring 13 (fees): how AMM prices, protects itself, and whether fills are economically fair/sustainable.

## ACTIVE STYLE DIRECTIVE (supersedes all prior style notes)
- Set 2026-04-20 mid-Ring-5. User reported HEADACHES from constant atomic-Socratic questioning ("answering a lot of questions after each interaction").
- NEW DEFAULT: deliver each ring as a FULL PICTURE upfront — framed, with concrete examples, tables, and a source-file pointer. Cover the whole ring's concept space in one teaching block.
- Ask at most ONE short check question at the END of the ring, only when needed to confirm a non-obvious piece landed before progressing.
- DO NOT fire atomic questions mid-ring unless the user explicitly asks to be drilled. Earlier "atomic Socratic default" (Rings 2-4) is SUPERSEDED.
- CHUNKING RULE (added 2026-04-20, mid-Ring-5 review): for LONG walkthroughs (e.g. full-diagram reviews, multi-concept syntheses), split into numbered chunks (e.g. "1 / 8") and deliver ONE chunk per turn. End each chunk with "say 'next' when ready" — user paces. Do NOT dump 2000+ word explainers in one message even when style is "full picture." Full picture per ring is fine; full picture per SINGLE concept is better for reviews.
- FINER CHUNKING (added 2026-04-28 mid-Ring-6): user explicitly said "this ring should be compiled by every bit in my brain. Use easy to understand vocabulary." After Chunk 2/6 delivered all 5 order types in one big table, user pushed back asking for ONE concept per turn. Going forward: each top-level chunk in the ring plan is broken into atomic SUB-CHUNKS (e.g. 2a, 2b, 2c). One sub-chunk = ONE small idea (~150-250 words), plain English first, source-code naming AFTER the idea lands. End with "say 'next' when ready". A long table or comparison is the LAST sub-chunk after all rows are taught individually. This applies for the rest of Ring 6 and likely future rings too — keep watching the user's signal.
- VOCAB GUARD (reinforced 2026-04-28): in early sub-chunks of any concept, prefer plain English ("the rule label", "wait in line") over the code term. Introduce the actual struct field / enum name only AFTER the concept clicks. Source-file pointers come at the END of the ring or sub-section, not at the top.
- FRUSTRATION GUARD (added 2026-04-30 during Ring-6 build): user got overwhelmed and cried when multiple layers were stacked at once (`is_available` / `is_for`, Drift invariant, market-index-0 edge case, Rust `Option`, `iter().position`, and mutation path). New hard rule for build tutoring: never combine a new protocol invariant with new Rust syntax in the same teaching chunk. Teach in this order: (1) plain object/locker analogy, (2) one concrete state table, (3) one helper's input/output only, (4) only then the Rust syntax, (5) only then composition with another helper. If user shows frustration, immediately stop coding, validate the load, summarize the one current idea, and offer a reset/pause. Do not ask the user to continue writing code while emotionally overloaded.
- DRIFT-ALIGNMENT GUARD (added 2026-05-01): user explicitly does not want invented mini-project shortcuts. Before build prompts, compare the exact Drift source slice and name every field/helper the invariant depends on. If mini-drift is missing a field, either add that field now or park it explicitly with the future ring that owns it; never silently ignore it as "simplified."
- DRIFT-STRUCTURE GUARD (added 2026-05-02): user explicitly wants `mini-drift` to follow Drift's file structure, code organization, and architecture as closely as possible, not only the concepts. Going forward, prefer Drift-shaped modules such as `controller/orders.rs`, `controller/position.rs`, `state/user.rs`, `state/order_params.rs`, and `math/*` as soon as a mechanism belongs there. If we temporarily keep something simpler, name it as temporary and move it before it grows. Never let "mini" become an excuse for student-project architecture.
- NO-INVENTED-FLOW GUARD (added 2026-05-05): do not create new controller flow helpers just to connect partial pieces if Drift does not have that shape. Helper slices may be built when they directly mirror Drift behavior. Missing dependencies must be parked with their owning ring instead of replaced by fake mini flow.
- SIDE-QUESTION GUARD (added 2026-05-04): when user asks a side/random question during a chunk chain, answer it briefly, explicitly label it as a side answer, then return to the exact previous teaching flow. Do not continue teaching in the side-question direction unless user clearly says to switch topics. After side answer, restate: "We were at [previous chunk/neuron]. Now we return there."
- QUESTION PARKING RULE (added 2026-05-08): when user asks a question, first decide whether it belongs to the current slice. If it belongs to a later ring/slice, do not answer deeply now; say exactly where it will be covered, save the question in `learning_state.md`, and answer it when that ring/slice opens. If it is safe to answer now, answer in the current tiny-chunk style.
- ONE-SCENE METHOD (added 2026-05-08 during Ring 7 route selection): when a concept has multiple moving pieces, use one stable scene before code labels. Name the actors and exact numbers, teach one rule from that scene, then ask one short check. Do not mix several examples, changing labels, or changing prices in the same explanation. This worked well for `AMM(Some(111))`: Alice + AMM current price + Carol fixed maker price.
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
- [x] Ring 6: Order struct, types, lifecycle, reduce-only, post-only -- COMPLETED 2026-05-03
- [x] Ring 7: Dutch auctions, keepers, fill flow, fees-on-fill -- COMPLETED 2026-05-14 as helper shell with strict R8/R9/R13/R23 boundaries
- [ ] Ring 8: Position updates: open/increase/decrease/close -- READY TO OPEN

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

### Ring 7
- Roadmap gap found 2026-05-04 during Dutch auction study: teaching `calculate_auction_price` without first teaching how Drift derives `auction_start_price`, `auction_end_price`, and `auction_duration` created a knowledge gap. Ring 7 roadmap updated to include `controller/orders.rs::get_auction_params`, `math/auction.rs::calculate_auction_prices`, `state/order_params.rs::derive_market_order_auction_params` / `get_auction_duration`, tick standardization, price-band validation, and `PerpMarket::amm_can_fill_order`.
- User correctly questioned unrealistic training numbers (`100 -> 108`) and asked how Drift bounds real auction prices. Locked correction: simple market-order auction boundaries are oracle anchored; default examples should use Drift-shaped small bands (e.g. oracle $100, 0.5% band around $100 / $100.50) unless deliberately using toy numbers for arithmetic.
- Full Ring 7 audit completed 2026-05-04 against Drift sources. Additional gaps patched into roadmap: keeper fill entry, place-and-take entry, `FillMode`, final fill-price validation, cleanup paths (`should_expire_order`, reduce-only-now-invalid cancellation), flat keeper reward rules, open bid/ask decrease on fill, `OrderActionRecord`, and the strict R7->R8 boundary. Key invariant: do not mark `Order.base_asset_amount_filled` / `Filled` unless the R8-owned position update succeeds, because Drift mutates position before order progress in `fulfill_perp_order_with_amm`.
- Correction added 2026-05-04: do not mix Drift's simple `calculate_auction_prices` fallback with the placement-time `OrderParams::update_perp_auction_params` / `derive_market_order_auction_params` sanitization path. Teach them as two steps: (1) derive/sanitize params from market/oracle/limit, (2) `get_auction_params` reads final stored boundary and minimum duration.
- First Ring 7 learning slice locked 2026-05-04: user can explain auction boundary, final gap, duration clamp, elapsed-slot price movement, direction-based tick rounding, fill-price safety, partial fills, `base_asset_amount_filled`, unfilled amount, open bids/asks decrease, `open_orders` count behavior, and event-as-receipt. Current build should start tiny with order-progress helpers, not full AMM/maker/fee logic.
- Ring 7 Build Slice 1 progress on 2026-05-05: user built/tested Drift-mirrored helper slices for `Order::get_base_asset_amount_unfilled` including reduce-only cap/same-direction/zero-position cases, `controller/orders.rs::update_order_after_fill` for partial/full fill order progress, `controller/position.rs::decrease_open_bids_and_asks` with Drift-style update flag and zero clamps, and `User::decrement_open_orders` with saturating behavior. Full `cargo test` passed: 33 tests. Important correction: do not invent mini-only flow helpers such as fake post-position-update wrappers; park real orchestration until Drift dependencies are ready.
- Ring 7 Build Slice 1 receipt progress on 2026-05-06: user built/tested the focused fill receipt shape in `state/events.rs`: `OrderAction::Fill`, `OrderActionExplanation::OrderFilledWithAMM`, `OrderActionRecord`, and `get_order_action_record`. Neuron locked: `filler` = who woke the fill, `taker` = whose order was filled, event action amount = this fill, copied order snapshot = progress after update. User also understood derive abilities at a practical level (`PartialEq` enables `==`, `Debug` prints in tests, `Default` chooses a starting variant, etc.). Focused test and user-reported full `cargo test` passed. Important correction: no fake emit wrapper and no fake fill flow; real emit belongs inside the real Drift-shaped fill path later.
- Ring 7 Build Slice 1 cleanup progress on 2026-05-06: user built/tested `controller/orders.rs::decrement_open_orders_after_full_fill`. Neuron locked: counter cleanup only runs after full fill, partial fill leaves counters unchanged, full fill decrements `user.open_orders` and the matched `perp_position.open_orders`, and zero position counter returns `MathError` instead of underflowing. Focused cleanup tests and user-reported full `cargo test` passed. Boundary remains clean: helper is not a fake fill wrapper and must later be called only after the real receipt emission in Drift-shaped fill flow.
- Ring 7 FillMode progress on 2026-05-06: user studied the three-label split: `OrderType` = order rule, `FillMode` = fill attempt context, `FulfillmentMethod` = actual route (`AMM` or `Match`). User built Drift-shaped `state/fill_mode.rs` with `Fill`, `PlaceAndMake`, `PlaceAndTake(bool, u8)`, `Liquidation`, plus `is_liquidation` and `is_ioc` tests. Neuron locked: instruction door determines FillMode; caller cannot freely inject a different mode through the wrong door.
- Ring 7 `FillMode::get_limit_price` progress on 2026-05-08: user built/tested the helper chain `Order.slot` -> `Order::has_auction` -> `math::auction::calculate_auction_price` -> simplified `Order::get_limit_price` -> `FillMode::get_limit_price`. Neuron locked: normal fill modes use the fixed order wall; `PlaceAndTake(_, percentage)` uses percentage to create pretend slots and a pretend current slot only when an auction exists; percentages above 100 cap at 100; no-auction `PlaceAndTake` falls back to fixed price. Important Rust neuron: cast `u8` duration/percentage to `u64` before multiplying, because `40u8 * 25u8` overflows before division. Verification: `cargo fmt -- --check` and full `cargo test` passed with 53 tests.
- Ring 7 Study Slice 2 route-selection progress on 2026-05-08: user locked the core route neurons in simple language. Price wall comes first; then Drift checks possible routes. Opposite maker side is necessary but not enough; prices must cross. For a maker match, final matched price is the maker's posted price. `AMM(Some(price))` means AMM gets a turn before a worse maker but stops at that maker price; `AMM(None)` means no extra maker stop line, while the user's wall still applies. Empty route list means no fill on this attempt, not order deletion. Teaching method that worked: one stable scene, one rule, one check.
- Ring 7 OrderActionRecord field tour 2026-05-09 (Study Slice 2 closing arc). Locked: Chunk 1 (Option<T> philosophy + always-present vs sometimes-present split), Chunk 2 (header: ts/action/action_explanation/market_index/market_type), Chunk 3 (filler block with owner-cancel vs keeper-cancel correction — filler/filler_reward depend on who triggered, fill_record_id is fill-specific only), Chunk 4 (money flow core + `maker_fee: i64` = rebate sign neuron, with Drift's `Some(-maker_rebate.cast()?)` internal trick). Mid-tour user pushed back on Chunk 5 surface tour ("ye last wala boring tha aur baad mai detail mai cover krna hai to skip kr do") and again later on overall scope ("bro utna karo jitna aaj jaroorat hai. baki ja b jaroorat hogi tab krr lenge"). I parked all forward-ref fields and shifted to a single compact final chunk: surface fees-on-fill (taker pays, maker gets rebate, rest to Drift fee pool — deep math Ring 13) + 4-path order-death taxonomy (full fill / owner cancel / keeper expiry via `should_expire_order` / keeper reduce-only invalidation, code-level work deferred to later build slice when needed). STUDY SLICE 2 CLOSED 2026-05-09. Two cross-session memories saved today: `feedback_skip_forward_ring_fields.md` (struct-field tour rule) and `feedback_just_in_time_scope.md` (broader minimum-needed-today rule). For future me: do not surface-tour all fields of any struct again; classify by ring before chunking.
- Ring 7 Study Slice 2 source-code bridge completed 2026-05-08: 9-chunk Hinglish bank+greeter+vending-machine+Carol scene walkthrough of `math/fulfillment.rs::determine_perp_fulfillment_methods` end-to-end (job-as-planner → inputs → direction flip L35 → AMM matching-tag pick L37-40 → pre-sorted maker line walk L42 → wall-cross check L44 → maker-vs-AMM comparison L53-55 + leashed `AMM(Some(maker_price))` push + `amm_price = maker_price` leash update L59-60 → unconditional `Match(..., maker_price)` push L64 → post-loop `AMM(None)` push L75-94). User passed the integration check: Carol-out-of-wall scenario produces `[AMM(None)]` only — the `Match` is excluded by the wall check inside the loop, and `AMM(None)` is still pushed because the post-loop AMM check is independent of whether any maker iteration completed (and `amm_price` stays at its original value when the comparison block never runs). Mode that worked best: stable bank+greeter+machine+Carol scene held across all 9 chunks with exact small numbers ($110/$112/$115); code shown only AFTER each scene step landed; Hinglish from Chunk 5 onward at user request. Cross-session memory `feedback_scene_story_mode.md` saved at user-memory level.
- Ring 7 route helper build progress on 2026-05-10: user built Drift-shaped `state::fulfillment::PerpFulfillmentMethod` with `AMM(Option<u64>)` and `Match(Pubkey, u16, u64)`, then built `math::matching::do_orders_cross` matching Drift's maker-direction comparison rules. Tests now cover AMM(Some), AMM(None), Match shape, maker Short crossing true/false, and maker Long crossing true/false. Neuron locked: `PerpFulfillmentMethod` names possible roads; `do_orders_cross` is the gate that decides whether a price route may be added. Full route planner remains intentionally parked because it needs maker ordering, AMM price, AMM availability, route ordering, and R8/R9/R23-owned dependencies. Assistant ran full `cargo test`: 60 passed.
- Ring 7 matching role-gate build progress on 2026-05-13: after a 2-day break and a short Malaysia-trip aside, user resumed with Hinglish/movie-scene mode because the dry gate style felt boring. User built/tested `are_orders_same_market_but_different_sides`, `Order::is_limit_order`, `Order::is_auction_complete` with Drift's strict `elapsed > auction_duration` rule, `Order::is_resting_limit_order` including the TriggerLimit edge, and `math::matching::is_maker_for_taker`. Neuron locked: `is_maker_for_taker` is Drift's judge for maker status, not a price check; post-only taker means "I want to wait," maker must be resting liquidity, and if both orders are resting then older ready time (`slot + auction_duration`) wins. Assistant verification: `cargo check` warning-free and full `cargo test` 80 passed. Teaching mode that worked better today: "Solana Bazaar / Drift judge / waiting stall" scene instead of repetitive checklist gates.
- Ring 7 close progress on 2026-05-14: user completed the matched-fill and cleanup helper arc. New shipped helpers: `math::orders::calculate_quote_asset_amount_for_maker_order` (one shared quote bill; maker Long floors, maker Short ceils), `math::matching::calculate_fill_for_matched_orders` (smaller maker/taker base amount + maker-price quote), `math::orders::should_expire_order`, Drift-shaped `math::orders::should_cancel_reduce_only_order` (Open + reduce-only + current unfilled under `step_size`), and `controller/orders.rs::should_reward_keeper` (external keeper reward gate, no self-reward). Important correction: reduce-only cleanup should reuse `Order::get_base_asset_amount_unfilled(Some(existing_base_asset_amount))` instead of manually duplicating direction checks. Final verification: `cargo fmt -- --check`, `cargo check`, `cargo test` 96 passed, and `cargo clippy -- -D warnings` clean. Ring 7 is closed as a helper shell; Ring 8 owns actual position mutation before any real fill orchestration.

### Ring 6
- Build-side Drift-alignment gap found 2026-04-30 before `place_perp_order`: mini-drift initially used `market_index == market_index` and `base_asset_amount == 0` to find/allocate perp position shelves. User correctly spotted the market-index-0 ambiguity and the "flat but used folder" problem. Roadmap updated: Ring 6 now explicitly requires `PerpPosition::is_available()` and `is_for(market_index)` before order placement, matching Drift's invariant that market label + availability state define whether a shelf is real.
- Position shelf repair completed and committed 2026-05-01. User built Drift-shaped helper chain: `is_open_position`, `has_open_order`, `has_unsettled_pnl`, `is_being_liquidated`, `is_available`, `is_for`, and `force_get_perp_position_index`. Rust tests covered: existing market folder wins, available folder is reused and relabeled, and no free slot returns exact `NoPerpPositionSlotAvailable`.
- Teaching style that worked extremely well on 2026-05-01: tiny chunks, easy vocabulary, one neuron per message, explicit repetition. User explicitly said they deeply understood because of this format. Continue this style for order-slot storage and `place_perp_order`.
- Open bids/asks neuron clarified 2026-05-02: Ring 6 only stores waiting exposure on `PerpPosition` (`open_bids` for future long/buy pressure, `open_asks` as negative future short/sell pressure). Full risk/margin use of these fields is intentionally parked for Rings 17-18, where they help compute worst possible position if waiting orders fill.
- Ring 6 core build completed and pushed 2026-05-03. User built `Order::is_available`, `get_available_order_index`, `force_get_available_order_index`, `Order::new_from_params`, Drift-shaped `controller/orders.rs::place_perp_order`, `controller/position.rs::increase_open_bids_and_asks`, `state/events.rs::OrderRecord`, and `instructions/user.rs::handle_place_perp_order`.
- Ring 6 tests now cover Market/Limit storage, unsupported Oracle/Trigger rejection, no order slot, no position slot, reduce-only without position, reduce-only same direction, reduce-only opposite direction, open bid/ask accounting, existing position direction snapshot, and order-slot helpers. Final checks passed before push: `cargo check`, `cargo test` (20 tests), and `cargo clippy -- -D warnings`.
- Important neuron locked on 2026-05-03: placing an order does not change `base_asset_amount`; it stores a waiting order, increments open-order/open-bid/open-ask tracking, and emits an event. Fill flow in Ring 7/8 is what will change the actual position size.
- Security neuron added at Ring 6 close: public Anchor instruction gathers context and accounts, but controller owns logic. `PlacePerpOrder` uses `has_one = authority` so a signer cannot place orders through someone else's `User` account.

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
- order / Order (a waiting request to trade; stored in the User account's fixed order array)
- OrderParams (the user request form) vs Order (the stored protocol record with status, id, progress, and snapshots)
- order slot (one fixed shelf in the `orders` array; Open means busy, non-Open means reusable)
- OrderStatus (Init, Open, Filled, Canceled; Open orders are waiting)
- OrderType (Market, Limit, TriggerMarket, TriggerLimit, Oracle; Ring 6 executes only Market/Limit)
- reduce_only (order rule: later fill may only move the position toward zero; placement itself does not reduce)
- open_bids / open_asks (waiting long/buy pressure and waiting short/sell pressure on a PerpPosition)
- OrderRecord (event receipt that copies and announces the stored order)
- OrderAction / OrderActionExplanation (event labels: what happened, and why/how it happened)
- OrderActionRecord (fill/cancel/etc. receipt; current mini slice supports AMM-style Fill)
- filler vs taker (filler wakes the fill transaction; taker owns the order being filled)
- counter cleanup (after full fill, user-level and market-folder open-order counters must decrease without underflow)
- FillMode (context label for a fill attempt: `Fill`, `PlaceAndMake`, `PlaceAndTake`, `Liquidation`)
- price wall (the worst allowed fill price for an order at a given moment)
- pretend current slot (PlaceAndTake's percentage-derived slot used to read an auction price immediately)
- FulfillmentMethod (actual fill route label: `AMM` or `Match`; Ring 7 built route-label helpers, not full execution)
- price crossing (buyer wall and seller price overlap enough for a fill)
- route list (ordered possible fill attempts; not a guarantee that every route executes)
- `AMM(Some(price))` (AMM route with an extra maker-price stop line)
- `AMM(None)` (AMM route with no extra maker stop line; user price wall still applies)
- empty route list (no valid fill in this attempt; an open order may keep waiting)
- instruction handler vs controller (instruction gathers accounts/time/user key; controller owns state-changing order logic)
- matched fill math (maker/taker trade size is the smaller remaining base amount; quote bill uses maker price)
- step_size (minimum valid trade unit; reduce-only cleanup cancels if remaining fillable amount is below it)
- keeper reward gate (external keeper can be rewarded for cleanup; self-cleanup does not receive keeper reward)

- Ring 8 next - open position mutation: connect Ring 7's `(base_filled, quote_filled)` result into PerpPosition changes. First target is the smallest Drift-shaped position delta helper before any full fill wrapper.
- Delivery: keep the tiny-neuron style. One helper contract at a time: plain meaning -> state table -> helper input/output -> Rust syntax -> tests.
- Forward refs to keep parked: Ring 9 (AMM reserve swap and price impact), Ring 10 (Oracle order type's price source), Ring 13 (deep fee model and fee pools), Ring 23 (full maker/taker matching engine).
- Forward refs still outstanding from earlier rings: Ring 9 (price impact, sqrt_k, open interest), Ring 10 (oracle mechanics), Ring 11 (bid/ask spread), Ring 14 (funding rate, last_cumulative_funding_rate on PerpPosition), Ring 22 (repeg), Ring 24 (LPs / virtual reserve origination).
