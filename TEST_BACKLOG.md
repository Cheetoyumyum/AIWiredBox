# Test backlog

Do not mark a test complete without an evidence report. Statuses: READY, NEEDS FIXTURE, AWAITING RUN, PASS, FAIL, BLOCKED.

## Priority 0 — foundational behavior

| ID | Question | Minimal experiment | Status |
|---|---|---|---|
| SEL-001 | Does a second selector replace the previous selection by default, and does Filter=true intersect it? | Select two distinct groups; compare effect targets with filter off/on. | Partially verified; regression needed |
| SEL-002 | Does inverse apply before or after filter in a multi-selector stack? | Two overlapping sets; test all filter/inverse combinations. | READY |
| SEL-003 | Does selector output persist for the next effect, and clear on a new trigger? | Two ordered effects; trigger twice with changed population. | NEEDS FIXTURE |
| SIG-001 | Which user/furni contexts are forwarded by Send Signal? | Known triggering user and selected furni; receiver reports each recipient context. | READY |
| SIG-002 | Does one signal to multiple antennas preserve the same forwarded sets? | One sender, two antennas, separate receiver outputs. | READY |
| SIG-003 | What happens when multiple receivers listen and mutate shared state? | Two receivers, shared counter, ordered logs; repeat. | READY |
| REC-001 | Distinguish triggering user, users from signal, and users from selector. | Three known user groups; send a private marker for each recipient mode. | READY |
| EXE-001 | Confirm effect ordering with Execute in Order and changed Z heights. | Two effects write distinct markers; swap Z and compare. | NEEDS FIXTURE |
| VAR-001 | Map variable-changed event codes to UI event types. | Isolated listeners for create, update, increase, decrease, delete, unchanged. | READY |
| VAR-002 | Does variable-change trigger fire on create, override, same-value write, and delete? | One variable and one listener per operation. | READY |

## Priority 1 — reusable building blocks

| ID | Question | Minimal experiment | Status |
|---|---|---|---|
| SIG-004 | Does context state persist through every signal hop, and how does delay affect it? | Bounded three-hop counter; print context at each receiver. | Partially verified; regression needed |
| LOOP-001 | What is the safest bounded loop pattern? | Counter loop with visible count and hard maximum; prove initializer is not re-entered. | READY |
| SEL-004 | Can a furni selector compare its variable against a context variable in mode 2? | One context value, two furni values, one selector. | READY |
| SEL-005 | Does users-from-signal selector select only forwarded users? | Signal one of two users; inspect selector output. | NEEDS FIXTURE |
| MOV-001 | How do invisible walkable blocks, a selector, and a repeater create an autoline? | Three-block straight queue, one player, no other game logic. | READY |
| MOV-002 | Does movement preserve the triggering user through signal/call-stack hops? | Move one player, then target/message at receiver. | READY |
| ADD-001 | What are the Execution Limit option semantics? | Small repeated-trigger test with each option variant and visible counter. | READY |
| ADD-002 | What are Animation Time and movement add-on semantics? | One furniture move; change one option per import. | NEEDS FIXTURE |

## Priority 2 — advanced behavior

| ID | Question | Minimal experiment | Status |
|---|---|---|---|
| CALL-001 | Does call-stacks preserve triggering-user, signal, selector, and context state? | Caller/receiver reports each context; invoke from chat and signal. | READY |
| TXT-001 | What is the Text Connector class and line-index behavior? | Context variable plus manually configured connector; test 0, 1, missing, out-of-range. | NEEDS FIXTURE |
| TXT-002 | How are multiple text output add-ons evaluated on one stack? | Message with username, variable, and furni-name placeholders. | READY |
| TRG-001 | What are the interval unit and minimum for periodically trigger? | Export several UI intervals and time visible ticks. | READY |
| TRG-002 | Compare period-short and periodically triggers. | Same counter, isolated trigger stacks. | READY |
| TXN-001 | Document transaction lifecycle and cancellation. | Non-value fixture; inspect init, complete, cancel paths. | NEEDS FIXTURE |
| SEL-006 | Determine selector entity limits and behavior above 20. | Test 1, 20, and 21 candidates with harmless marker effect. | READY |

## Adding tests
Use a stable ID (AREA-NNN), one narrow question, a minimal experiment, and a status. Keep failed tests; record what they ruled out.
