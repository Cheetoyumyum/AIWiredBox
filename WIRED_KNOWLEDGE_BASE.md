# HABBO WIRED / G-PRESETS: COMPLETE KNOWLEDGE BASE

Audience: any AI that must build, edit, validate or debug Habbo Wired rooms as G-Presets JSON.
Companion files: `wired_codes.json` (machine-readable code tables), `POKER_PROJECT_PLAN.md` (the poker project).
Confidence tags used everywhere: **[V]** verified in game or by a user export, **[I]** inferred from exports, **[U]** unverified / do not rely on it.

The single most important rule: **never guess an option code, classname or semantic. Ask the user to run a small calibration preset or export, then decode it.** Every large mistake in this project came from guessing.

---------------------------------------------------------------------
## 1. How this collaboration works (read first)
- The user owns the room and imports your JSON with G-Earth + G-Presets. You cannot run Wired. All testing happens in the user's hotel. Usage limits mean you may get only a few turns per 5 hours, so each turn must produce something small, testable and correct.
- The user wants **lean builds**: Cherb (reference gambling game) is ~615 Wired configs; Tile Theory ~241. A 2,189-furni poker build was rejected as unstable. Target a few hundred Wired furni.
- Deliver **small separately importable presets**, one subsystem at a time. Label each as VERIFIED / PROPOSED / NEEDS TESTING.
- Functional furniture (seats, coins, transaction tiles, chests, ALL Wired) is picked up by the user between attempts. Never assume old IDs or old Wired exist. Decorative layout furniture stays.
- Best calibration technique (worked every time): generate a preset with one furni per option variant, have the user import it, open each dialog or run each chat command, and report. Or have the user build the variants in the UI and export; decode the diff.
- A Python model of the game logic (a tiny VM for the variable operations plus a stack/signal scheduler) is the best way to prove logic before the user spends time. Fuzz it against an independent reference implementation. This caught real bugs (see section 9).

---------------------------------------------------------------------
## 2. JSON structure [V]
Top-level keys: `recolorables`, `wired`, `bindings`, `furni`.
- `furni`: list of `{id, name, className, location:{x,y,z}, rotation, state?}`. `name` is `className[n]` and must be unique. `id` is a preset-local instance id. Coordinates are absolute tiles relative to the import root. `z` is stack height.
- `wired` has lists `effects, variables, addons, conditions, triggers, selectors` plus the dict `variables_map` (variable name -> id).
- Every wired record: `wiredId` (= the furni `id` it belongs to), `config` (text), `options` (int array), `items`, `secondItems`, `furniSources`, `userSources`, `variableIds`, effects also have `delay`, conditions have `quantifier`, selectors have booleans `filter` and `inverse`.
- `wiredId` and `id` are the same number but different concepts. A className identifies a furniture type; id identifies an instance.
- A stack is simply all furni on the same x,y, ordered by z. Triggers fire the stack; conditions gate it; selectors choose targets; effects run in z order only if an Execute in Order add-on is present (add one whenever order matters). Positions matter: two objects are in the same stack only if they share a tile.
- Variable definitions inside a preset are CREATED on import. **Importing a preset that contains definitions for variables the room already has creates duplicates, and the game rejects duplicate names.** Test presets must either define only brand-new uniquely named variables or contain no definitions.
- Variable names must be unique across ALL variable types. User convention in this project: prefix `f` furni (ftable), `p` player (ptable), `c` context, `r` room.
- Context variable definitions in real exports have `variableId: ""`; the id is in `variables_map`. My test presets filled `variableId` and still worked for effects/add-ons [V], but mirroring real exports is safer.
- Furni "functional" classnames seen in the poker room [V in export]: chair `classic9_chair`, coin `lt_c26_coin`, chest `wf_storage_coins2`, transaction tile `cine_tile`, payment contract `wf_contract_payment`, buttons `wf_button_green`, antenna `wf_antenna2`. Availability in the hotel must be checked by the user.

---------------------------------------------------------------------
## 3. Variables
- Scopes: user, furni, room, context. Context variables are temporary calculation state for one execution chain and **persist across signal hops within that chain [V: `!ctx` test printed 123]**. Use them as registers.
- **A variable must exist (be Given) before it can be changed or compared; selectors on a missing variable select nothing [V]** (the user's note: "need !s0 first, otherwise the variable you are targeting won't exist"). Give with Override=1 = create-or-assign.
- Non-permanent user variables vanish when the player leaves (useful for cleanup). Permanent ones persist.
- Values are signed 32-bit integers [V: NOT 5 = -6; 4294967289 appeared for -7 once].
- Division is integer. 7/2 = 3 [V]. Modulo works [V]. Behaviour on divide-by-zero is [U]: always guard.
- Operation codes: see `wired_codes.json` (tested: assign, add..divide, power, modulo, min/max, random, AND/OR/XOR/NOT, shift-left, bit count, get/set/clear/toggle bit, all eight next/previous bit searches). Not verified: 60 (absolute value) and 105 (shift right).
- Bit-search naming trap: "Low" = a 0 bit, "High" = a 1 bit; results are indices; -1 means none.
- Text-input variable capturer: only writes to a **context** variable [V]. Chat trigger text like `!raise #(cst_amt)`; copy the context value elsewhere if needed.
- Variable displays: a Variable stack with a display FX add-on above it shows live values (chips, pot, timers). One variable per display. [explained by user]

---------------------------------------------------------------------
## 4. Conditions [V]
`wf_cnd_var_val_match` options `[scope, cmp, 0, 0, value, valueSource, 0]`.
Comparison codes (user exported six conditions in the dialog's order): **0 `<`, 1 `=`, 2 `>`, 3 `<=`, 4 `!=`, 5 `>=`**.
Conditions are evaluated when the stack triggers, before its effects. Never rely on a condition seeing a value that an effect in the same stack will change.
`has variable` / negated variants exist (`wf_cnd_has_var`, `wf_cnd_neg_has_var`).

---------------------------------------------------------------------
## 5. Selectors [V, decoded from the Nov 2026 calibration runs]
Records `wf_slc_users_with_var` and `wf_slc_furni_with_var`: `options = [cmp, mode, 0, literal, valueSource, 0]`.
- `cmp` uses the same codes as conditions (0 `<` ... 5 `>=`). The dialog shows all six radios.
- `mode` 0 = only "has the variable" (Select by value unticked; radios greyed); 1 = select by value against the literal in `options[3]`; 2 = select by value against the variable named in `variableIds[1]` (e.g. furni `table` = user `ptable`).
- `variableIds = [variable, referenceVariable]` (reference is `'n'` or `'0'` when unused).
- `filter` = "Filter existing selection" (intersect with the previous selector's output), `inverse` = "Invert". Two selectors in a stack: the second replaces the first unless filter is true.
- Selector output is consumed by effects through source code 200 (furni) or user source 200. A selector's output is NOT the triggering user and NOT users from a signal.
- Other selectors seen: `wf_slc_furni_bytype`, `wf_slc_furni_neighborhood`, `wf_slc_furni_area`, `wf_slc_users_area`, `wf_slc_users_signal`. Use furni variable `@class_id` (variable id `-101`) with `=` literal to select by type, e.g. coin class id 18698 [V from export]. Inversion trick (user-explained): select an unreachable tile/area and invert to get everyone outside it.
- Proof results: for `ptst=2`, literal 3: code 0/mode 1 `<` selects; `=` and `>` do not; mode 0 selects every holder. For furni `ftst` vs user `ptst` (mode 2, `=`): the antenna with `ftst=2` was selected when `ptst=2`.

**Mistake history:** I first assumed selectors had only three comparisons because I put the mode number into the wrong slot. The truth is `options[0]` is the comparison and `options[1]` is the mode.

---------------------------------------------------------------------
## 6. Effects, signals, execution
- **An effect runs once per stack execution, not once per selected user [V: `!s2` with two users selected ran once].** It applies to all selected users/furni for that single operation (e.g. Show Message to a selected group works), but it cannot aggregate across them. To sum or compare values across players use per-player loops or keep running totals on a furni variable updated by each player's own action.
- **Send Signal** `[0,0]`; `items` = antenna ids; `furniSources = [100, forwardSource]`; `userSources = [forwardUsers]`. `100` in position 0 means "use the furnis picked in this effect". One effect can target up to 20 antennas (any effect that selects furni has a 20 furni limit) [explained]. Dialog checkboxes: "send signal for each furni", "...for each user" (semantics U).
- **Convention (Cherb): each stack carries its own outgoing antenna on top of it, one per stack** [V: 24 of 30 Cherb antennas are on the sending stack]. Receivers listen via `wf_trg_recv_signal`.
- Signals can carry users: later stacks can target "users from signal".
- Three recipient contexts, never confuse: triggering user (who said the command/walked), users from signal (forwarded by a previous stack), users from selector.
- **Loops:** signal-feedback loop: init stack (once) -> work stack -> increment/check -> signal back to the work stack, never to the init stack. Stop when the condition fails; a completion signal is only needed if something follows [explained, and condition-gated loops confirmed working by the user].
- **Race hazard [I from my simulator]:** if two stacks listen to one antenna with opposite conditions on a variable that one of them modifies, the second may see the modified value and also fire. Make a separate decision stack that computes a flag, then branch on the flag.
- Execute in Order add-on: use it for ordered effect sequences [V used throughout]. Execution Limit add-on exists (options seen `[100,1]`, `[1,10]`) [semantics U].
- Delay unit: seconds in the UI; stored number appears to be 0.5 s steps [I: 2 -> ~1 s, 6 -> ~3 s, matched the lock test].
- `wf_act_call_stacks` (Cherb): run another stack by pointing at its trigger furni, no signal [semantics U].
- Lock pattern that worked [V `!t7`]: negated has-variable condition on a lock variable, then Give the variable, act, and a delayed Remove Variable; spamming the command printed once per window.
- Multiple "repeat" semantics: there is no repeat effect in the reference files.

---------------------------------------------------------------------
## 7. Triggers
`wf_trg_says_something` (`[0,0,1]`, config text; use a delimiter like `!`), `wf_trg_recv_signal`, `wf_trg_click_furni`, `wf_trg_walks_on_furni`, `wf_trg_leave_room`, `wf_trg_room_initializes`, `wf_trg_periodically` (`[2]`), `wf_trg_var_changed` (events: 3, 4, 7 seen; UI offers created/updated/increased/decreased/deleted/unchanged; "unchanged" lets you trigger by writing the current value back; exact codes [U]), transaction triggers (`wf_trg_transaction_complete`). Room init does not reset variables unless Give has Override=1.

---------------------------------------------------------------------
## 8. Messages, text, display
- Private message: Show Message with visibility 0 ("only the user") and user source triggering/selected [V].
- Placeholders: `$(name)` filled by `wf_xtra_text_output_variable` / username add-ons on the same stack. `<` and `>` characters are stripped from message text [V: broke my "7<<4" labels].
- Markup (user-supplied): `[b] [i] [u]`, colour tags like `[purple]`, `[red]`, `[a]` align. Newlines work.
- ALT codes the Habbo font renders as symbols (user list): club `ALT+175`, diamond `ALT+0213`, heart `ALT+159` (also `SHIFT+\`), spade `ALT+0245`. Their unicode equivalents are [I]: `»`, `Õ`, `ƒ`, `õ`. Test by typing them in chat.
- Text Connector add-on (tutorial supplied by user): sits on the same tile as a context variable (Has Value ticked), lines `0=text`, `1=text`...; a `$(var)` placeholder with Variable display type "Textual" then prints the line for the variable's current value. Its classname is unknown [U]; configure by hand in the UI. This is how to print card names from card numbers.

---------------------------------------------------------------------
## 9. Verified mistakes (do not repeat)
1. Putting variable definitions in an import into a room that already has them -> duplicate variables.
2. Using condition codes for selectors in the wrong slot (see section 5).
3. Send Signal with `furniSources [0,0]` -> signal never fires.
4. Capturing chat text into a user variable (impossible; context only).
5. Stacking everything 0.65 high; selectors are 1.0, add-ons ~0.37.
6. Unrolling arithmetic across 8 seats and 20 temp registers produced ~2,000 furni. Prefer selectors, conditions and variable triggers.
7. Parallel test stacks fired from one signal overwrote each other's results; run tests as independent chat commands.
8. Case-colliding names (`cg_W1`, `cg_w1`) are duplicates; keep names case-distinct.
9. Two stacks on one antenna with conditions on a variable that one of them changes (race hazard, section 6).

---------------------------------------------------------------------
## 10. Algorithms already proven (Python reference + fuzz, not yet in Habbo)
- **7-card evaluator via suit rank masks**: four 13-bit masks (clubs, diamonds, hearts, spades). quads = AND of all four; trips = OR of 3-way ANDs; pairs = OR of 2-way ANDs; flush = suit mask with bit count >= 5; straight = mask AND its shifts by 1..4 (use shift-left), plus the wheel (A,2,3,4,5 = bits 12,0,1,2,3, mask 4111) as a special case; kickers via "previous high bit exclusive". Score = (category<<20)|(rank1<<16)|...|rank5; compute each category's candidate score, multiply invalid ones by a 0/1 flag, take the max with op 40. 305 straight-line ops; 0 mismatches vs brute force in 60,000 pairs. Compact it with call-stack subroutines.
- **Deck**: two 26-bit masks (deck_c = cards 0-25, deck_d = 26-51), card = suit*13+rank. Draw: random 52, select half with a 0/1 flag, index = r - 26*half, test the bit, retry on a hit; clear the bit on success.
- **Dealer/blinds**: seats are bits 1..8; "next seat" = next high bit exclusive of the current seat, wrapping to the lowest set bit. Heads-up: dealer = small blind and acts first preflop; big blind first postflop.
- **No-limit betting with short all-in rules**: per table masks `fpend` (must still act) and `fcanr` (may raise). A full raise (>= last raise size) resets both to all other active players. A short all-in adds to a `fshort` accumulator; it reopens raising only if the accumulator reaches the last raise. Call = min(need, chips). Fuzz-verified against a reference model (36k checks).
- **Side pots**: peel layers: m = smallest positive remaining contribution; layer = m * contributors; eligible = non-folded contributors; winners = best score among eligible; rake 5% (floor) on layers with 2+ contributors and an eligible player; share = net/winners; odd chips to winners clockwise from the dealer; uncalled layers (no eligible player) are refunded. Verified vs reference in thousands of cases; chips conserved.

---------------------------------------------------------------------
## 11. Reference projects and where to find them
Do not copy their ordinary furniture; use them as pattern sources. All three JSON files can be downloaded directly and parsed (they are G-Presets exports with the structure in section 2).

| Name | What it is | Link |
|---|---|---|
| **CentraDesk repository** (user's GitHub) | Holds the user's example presets | https://github.com/Cheetoyumyum/CentraDesk |
| **Cherb** (`cherb game.json`) | A gambling/profit game by the user. ~1,046 furni, ~615 Wired configs, ~164 furniture classes. Best example of efficient signal chains, variable-changed triggers (one state variable drives many stacks), negative signals, random-number architecture, selectors (`wf_slc_users_signal`, `wf_slc_furni_area`), `wf_act_call_stacks`, execution limits, and transaction/storage mechanics. Compare with it before designing anything big. | https://raw.githubusercontent.com/Cheetoyumyum/CentraDesk/master/cherb%20game.json |
| **Tile Theory** (`tile game.json`) | A working tile game by the user. ~1,241 furni, ~241 Wired configs. Shows per-player variables and progress, walk-on triggers, invisible walkable tiles, individual vs room-wide timers, random events (rain), furniture selected by type, temporary effect furniture, teleport/winner handling. Reference only; do not modify. | https://raw.githubusercontent.com/Cheetoyumyum/CentraDesk/master/tile%20game.json |
| **`pkr`** (in the same repo) | **The rejected 2,189-furni poker attempt** produced in this project (a complete but far too large build). Do NOT use it as a model. It is useful only as a record of what to avoid and of how a full import file looks. | https://raw.githubusercontent.com/Cheetoyumyum/CentraDesk/master/pkr |
| **G-Presets** (the tool that imports/exports presets) | Inspector extension by sirjonasxx for G-Earth. The README does not document option codes; codes must be calibrated (this document). | https://github.com/sirjonasxx/G-Presets |
| **Habbo Builders Club help** | What the Builders Club warehouse is; availability varies by season | https://help.habbo.com/hc/en-us/articles/360011621659-What-is-Builders-Club |
| **Ducket catalogue** | Searchable furniture catalogue; Builders Club collection | https://www.ducket.net/catalog and https://www.ducket.net/catalog/furnilines/buildersclub |
| **HabboAssets furniture explorer** | Furniture archive with classnames | https://www.habboassets.com/furniture |
| **HabStats API** | Furniture archive API | https://habstats.net/api |

Files supplied only in the chat (not online; ask the user to re-upload if you need them): the **original poker export** (`poker.json`, 718 furni, 150 Wired; the banking/seating room that the project preserves), the **autoshop export** (`Shop_wired.json`, used for the text capturer and periodic trigger records), and the calibration exports `vvalues compare.json`, `variable_selectors.json`, `operand.json`, `claude_info.json`. Third-party catalogue entries do not guarantee an item is borrowable in the live hotel.

---------------------------------------------------------------------
## 12. Open questions [U]
1. Exact variable-changed event codes and trigger semantics; call_stacks behaviour; periodic interval unit; execution limit semantics.
2. Whether a selector can compare against a **context** variable reference in mode 2 (poker design needs it for seat lookup).
3. Whether a furni selector reference to the player's own variable works on every furni class.
4. Classname and record of the Text Connector add-on.
5. Ops 60 and 105; divide-by-zero; random range endpoints.

---------------------------------------------------------------------
## Appendix A: Wired classnames seen in the supplied exports (poker, Shop, Cherb, Tile)
| className | seen in (file:count) |
|---|---|
| `wf_act_bot_move` | cherb:12 |
| `wf_act_bot_talk` | cherb:7, tile:3 |
| `wf_act_call_stacks` | cherb:1 |
| `wf_act_cancel_transaction` | poker:1, shop:2, cherb:3 |
| `wf_act_change_var_val` | poker:13, shop:12, cherb:25, tile:21 |
| `wf_act_freeze` | poker:1, cherb:6 |
| `wf_act_furni_to_furni` | cherb:8, tile:2 |
| `wf_act_furni_to_user` | cherb:6 |
| `wf_act_give_currency` | poker:1, shop:1, cherb:1 |
| `wf_act_give_furni` | cherb:8 |
| `wf_act_give_var` | poker:11, shop:9, cherb:23, tile:13 |
| `wf_act_init_transaction` | poker:1, shop:2, cherb:3 |
| `wf_act_log` | shop:1, cherb:11 |
| `wf_act_match_to_sshot` | cherb:24, tile:6 |
| `wf_act_move_rotate_user` | cherb:9, tile:2 |
| `wf_act_neg_send_signal` | poker:5, shop:4, cherb:5 |
| `wf_act_place_furni` | cherb:1, tile:2 |
| `wf_act_remove_furni` | tile:2 |
| `wf_act_remove_var` | poker:6, shop:7, cherb:23, tile:20 |
| `wf_act_send_signal` | poker:1, shop:2, cherb:35, tile:8 |
| `wf_act_set_highlight` | cherb:1, tile:4 |
| `wf_act_set_visibility` | cherb:7, tile:16 |
| `wf_act_show_message` | poker:11, shop:4, cherb:31, tile:7 |
| `wf_act_teleport_to` | poker:1, cherb:12, tile:6 |
| `wf_act_toggle_state` | poker:1, cherb:3 |
| `wf_act_unfreeze` | cherb:1 |
| `wf_act_user_to_furni` | cherb:9, tile:5 |
| `wf_cnd_actor_in_group` | cherb:7, tile:2 |
| `wf_cnd_has_var` | shop:10, cherb:6, tile:3 |
| `wf_cnd_neg_has_var` | poker:2, cherb:2, tile:5 |
| `wf_cnd_trggrer_on_frn` | poker:3, cherb:2 |
| `wf_cnd_var_val_match` | poker:14, shop:11, cherb:35, tile:5 |
| `wf_slc_furni_area` | shop:1, cherb:4, tile:2 |
| `wf_slc_furni_bytype` | poker:1, cherb:3, tile:10 |
| `wf_slc_furni_neighborhood` | poker:2, cherb:2 |
| `wf_slc_furni_with_var` | poker:5, shop:5, cherb:2 |
| `wf_slc_users_area` | cherb:15, tile:15 |
| `wf_slc_users_neighborhood` | tile:5 |
| `wf_slc_users_onfurni` | tile:1 |
| `wf_slc_users_signal` | cherb:1, tile:1 |
| `wf_slc_users_with_var` | poker:2, cherb:10 |
| `wf_trg_click_furni` | poker:2, shop:4, cherb:14, tile:1 |
| `wf_trg_click_user` | cherb:1 |
| `wf_trg_enter_room` | cherb:2, tile:1 |
| `wf_trg_leave_room` | poker:1 |
| `wf_trg_period_short` | cherb:3, tile:3 |
| `wf_trg_periodically` | shop:1, cherb:8, tile:6 |
| `wf_trg_recv_signal` | poker:6, shop:6, cherb:40, tile:6 |
| `wf_trg_room_initializes` | poker:1, cherb:1 |
| `wf_trg_says_something` | poker:1, shop:6, cherb:9, tile:4 |
| `wf_trg_transaction_complete` | poker:1, shop:1, cherb:3 |
| `wf_trg_user_performs_action` | poker:1, shop:1, cherb:1, tile:1 |
| `wf_trg_var_changed` | poker:8, cherb:9, tile:5 |
| `wf_trg_walks_on_furni` | poker:1, cherb:9, tile:8 |
| `wf_var_context` | poker:3, shop:3, cherb:3 |
| `wf_var_furni` | poker:23, shop:2, cherb:1, tile:1 |
| `wf_var_room` | poker:1, shop:2, cherb:9, tile:3 |
| `wf_var_user` | poker:9, shop:4, cherb:12, tile:12 |
| `wf_xtra_anim_time` | tile:2 |
| `wf_xtra_custom_contract` | shop:2 |
| `wf_xtra_exec_in_order` | poker:2, shop:6, cherb:12 |
| `wf_xtra_execution_limit` | poker:2, cherb:18 |
| `wf_xtra_filter_furni` | tile:2 |
| `wf_xtra_mov_carry_users` | cherb:3 |
| `wf_xtra_mov_curve` | cherb:22, tile:3 |
| `wf_xtra_mov_no_animation` | tile:3 |
| `wf_xtra_mov_physics` | cherb:38, tile:6 |
| `wf_xtra_scan_chest_furni_by_type` | shop:4 |
| `wf_xtra_text_input_variable` | shop:2, cherb:1 |
| `wf_xtra_text_output_furni_name` | shop:3, cherb:5 |
| `wf_xtra_text_output_username` | poker:1, shop:1, cherb:15, tile:2 |
| `wf_xtra_text_output_variable` | poker:3, shop:8, cherb:8 |
| `wf_xtra_unseen` | cherb:1 |
| `wf_xtra_var_leaderboard` | cherb:1, tile:1 |
| `wf_xtra_varfx_number` | poker:2, shop:4, cherb:1 |
| `wf_xtra_varfx_prog` | tile:5 |
| `wf_xtra_varfx_status` | cherb:1 |
