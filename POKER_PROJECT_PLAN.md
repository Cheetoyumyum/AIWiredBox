# HABBO POKER PROJECT: PLAN, STATE AND HAND-OFF
Read `WIRED_KNOWLEDGE_BASE.md` and `wired_codes.json` first. Confidence tags: **[V]** verified, **[I]** inferred, **[U]** unverified, **[P]** proposed design.

## 1. Goal
Two independent No-Limit Texas Hold'em tables (8 seats each) built only from Habbo Wired via G-Presets: banking/buy-in, seating, ready check, dealer and blinds, private hole cards, full betting rules (check, call, raise, all-in, short all-in rule), board, 7-card evaluation with kickers, side pots, split pots with odd chips, 5% rake, AFK = instant fold, leave/disconnect handling. Must be reliable, exploit-resistant, and LEAN (target roughly 600-700 Wired furni including banking, hard ceiling ~1,000). The user rejected a 2,189-furni build as unstable.

## 2. Room facts
- Floor now spans x 37-55, y 3-39 for the build area (old wired lived at x 37-47, y 3-14). Absolute coordinates; import root is chosen by the user.
- Two tables: Table 1 = coin near (23,31), button near (24,31 area), chairs south; Table 2 = coin near (23,20). Historic ids (not reliable): coins 447, 457; green buttons 476, 488; bank coin 391, chest 440, transaction tile 392, contract 581; chairs 322 325 363 373 442 450 524 526 and 331 333 377 386 451 459 531 533. Classnames in `WIRED_KNOWLEDGE_BASE.md` section 2.
- The user picks up all functional furniture and all Wired between attempts. The preset must carry everything required (coins, chairs, buttons, chest, tile, contract, Wired); ids are re-assigned on import.
- Table 1 is the coin at y 31 and uses `players`; Table 2 is the coin at y 20 and uses `players_2` (derived from the old seating stacks and the user's inspector) [V]. Chairs carry furni variables `table`, `tseat`, `occupied`; the seat is joined by walking onto a chair with enough chips (old rule: chips above 2000).
- Existing banking (buy chips via contract/transaction, `!withdraw`, chip display) is working and **must not be modified**. Seating, ready, leave and seat timers may be improved.

## 3. User preferences and rules of engagement
- Lean builds; small, separately importable and testable presets; the user will not import 2,000 furni.
- Always say what is VERIFIED / PROPOSED / NEEDS TESTING. Do not guess codes; ask for exports/calibrations.
- Variable names unique across types; prefixes f/p/c/r. No duplicate variables on import. Test presets must not redefine existing variables.
- Antenna convention: one antenna per stack, on top of the stack that sends to it. Use real heights (selector 1.0, effects 0.65, add-ons ~0.37).
- Messages: private to the player for hole cards ("only the user"); card text via Text Connector (manual). `!` prefix for commands. `!raise N` captures N into a context variable.
- AFK means instant fold. Optional `!autocheck` only checks when legal, never calls.
- The user has a limited number of AI turns per 5 hours; each turn should deliver one concrete, correct step.

## 4. What is already proven (offline, in a Python model; not yet in Habbo)
See knowledge base section 10: 7-card evaluator, deck draw, dealer/blind rotation, betting rules with short all-in handling, layered side pots with rake and odd chips. A full-game simulation of an earlier design ran 3,749 random hands over two tables with every payout equal to an independent calculation and chips conserved. That design was ~1,265 effects (too big). The logic is reusable; the structure must shrink.

## 5. Verified Wired facts that shape the design
- Conditions have all six comparisons; selectors also take the same codes and have modes (has-variable / literal / variable reference) [V].
- Effects run once per stack, not per selected user [V]; aggregation across players needs loops or running totals.
- Context variables persist across signals in a chain [V]. Capturer writes to context only [V].
- Bitwise ops and random work [V]; see `wired_codes.json`.
- Signal fields: `furniSources [100, forward]` [V].

## 6. Proposed lean architecture [P]
State lives in variables; stacks are small; branches use conditions (cheap) instead of flag arithmetic.
- **Table state (furni vars on each coin)**: `phase, turn, dealer, pot, current_bet, last_raise, deck_c, deck_d, board_1..5, timer, timer_active` (existing) plus `fready, fhand, ffold, fallin, fpend, fcanr, fsb, fbb, ffirst, fshort, fbml, fbmh, fsbamt, fbbamt` as seat bitmasks (seat n = bit n, n 1..8) [P].
- **Player state (user vars)**: `chips, seat, ptable` (existing), `bet, pcontrib, card_1, card_2, pml, pmh, pscore, pwin` [P].
- **Table pointer**: furni selector `table = ptable` (mode 2) filtered by class (coin class id 18698) [selector decode V, class filter I]. One shared set of stacks serves both tables.
- **Seat lookup**: users whose seat equals a context value (selector mode 2 with a context reference) [U: must be tested]; fallback is a combined key `pgseat = seat + 8*(table-1)` compared to a literal in a loop.
- **Per-action stacks**: `!fold`, `!check`, `!call`, `!raise N`, `!allin`, each gated by conditions (turn == seat, phase in 1..4, legal amount). Raise bookkeeping uses `fpend/fcanr/fshort` as in the proven model; use conditions rather than the 0/1-flag arithmetic where possible.
- **Variable triggers** can drive phase transitions instead of signals [U until the trigger codes are tested].
- **Evaluator**: one shared stack called per player via call-stack or signal; shrink the proven 305 ops with subroutines for the repeated top-five and straight blocks (target ~230).
- **Payout**: side-pot layering needs aggregation, so use loops over seats with per-player variables, or maintain running per-table totals during betting to avoid it; choose after testing selector-by-context and call-stacks [P].
- **Timers**: a periodic trigger per table decrements `timer` while `timer_active`; expiry folds the current player (not check). Stale timers are prevented by clearing `timer_active` on every action.
- **Leave/disconnect**: on leave clear the ready bit, fold the player if in hand (chips already contributed stay), then re-run the "advance" logic.

## 7. Delivery plan (each stage its own preset)
| Stage | Content | Target furni |
|---|---|---|
| A | init (table/chair variables), ready/unready, hand start, dealer, blinds | ~70 |
| B | deck, unique draws, private hole cards, board, card text | ~90 |
| C | betting stacks, street changes, timers, AFK | ~110 |
| D | evaluator | ~230 |
| E | side pots, payout, rake, leave handling, cleanup | ~120 |
Existing bank (~150) stays. Total ~770 in the worst case; reduce further after tests.

## 8. Status at hand-off
- Done: calibration of operations, comparison codes, selectors (modes), send-signal fields, context persistence, capturer behaviour, once-per-stack effects, lock pattern.
- Not started in Habbo: Stage A and everything after.
- Last user result (selector calibration): with `ptst=2` and `ftst` 2/4: literal-mode furni selector with value 0 selected nothing; variable-reference mode (`=`) selected the antenna with `ftst=2` only. This matches the decode in the knowledge base.

## 9. Next concrete steps
1. Ask the user to run (or generate) a micro-test: selector mode 2 comparing a user variable to a **context** variable (for seat lookup) and the call-stack effect (does it share context, does it forward users). Also test the six selector comparisons with mode 1 on `ptst` (2/3/4 vs 3).
2. Build Stage A with correct selectors, `furniSources [100, ...]` signals, real stack heights, and each stack carrying its own antenna.
3. Each later stage gets: a preset, a sentence on what to type to test it, expected output, and a simulator result.
4. Maintain a changelog file of every newly verified fact and move it into the knowledge base.

## 10. Test plan for the finished game (from the original brief)
Basic play with 2 and 8 players, dealer rotation, heads-up blinds, unique cards, all actions, short all-in not reopening, minimum raise, multiple all-ins and side pots, folded contributor, best hand wins main pot but loses side pot, split pots and odd chips, uncalled bets returned, every hand category and kicker tie, wheel straight, board plays, hot-seat join, leave during hand, both tables simultaneously, duplicate triggers and stale timers do nothing, rake exactly 5% floor and never doubled, chips conserved.

## 11. Links (details in `WIRED_KNOWLEDGE_BASE.md` section 11)
- Reference presets (download and parse these): Cherb https://raw.githubusercontent.com/Cheetoyumyum/CentraDesk/master/cherb%20game.json ; Tile Theory https://raw.githubusercontent.com/Cheetoyumyum/CentraDesk/master/tile%20game.json ; repository https://github.com/Cheetoyumyum/CentraDesk
- `pkr` in that repository is the REJECTED 2,189-furni poker build; do not copy its structure.
- Tool: https://github.com/sirjonasxx/G-Presets . Catalogues: https://www.ducket.net/catalog , https://www.habboassets.com/furniture , https://habstats.net/api
- The original poker room export (banking and seating to preserve) was supplied in chat as `poker.json`; ask the user to re-upload it.

## 12. Folder suggestion for sharing
`WIRED_KNOWLEDGE_BASE.md`, `wired_codes.json`, `POKER_PROJECT_PLAN.md`, the preset files, and the user's evidence exports (`vvalues compare.json`, `operand.json`, `variable_selectors.json`, `claude_info.json`, `poker.json`, `Shop_wired.json`).
