# AIWiredBox — Habbo Wired Engineering Wiki

An evidence-driven reference for building, testing, explaining, and debugging Habbo Wired systems using G-Presets JSON.

## Start here
1. [AI instructions](AI_INSTRUCTIONS.md)
2. [Wiki index](WIKI_INDEX.md)
3. [Wired knowledge base](WIRED_KNOWLEDGE_BASE.md)
4. [Testing handbook](testing/TESTING.md)
5. [Prioritized test backlog](testing/TEST_BACKLOG.md)
6. [Test report template](testing/TEST_REPORT_TEMPLATE.md)

## Ground rules
- Never invent a classname, option value, source code, or semantic.
- Separate verified facts from inference and untested hypotheses.
- Prefer the smallest importable calibration preset over a large speculative build.
- Preserve working projects. Reference projects are not permission to modify them.
- Record setup, action, expected result, actual result, and export evidence.
- Update documentation and regression tests when mechanics are confirmed or corrected.

## Confidence labels
| Label | Meaning |
|---|---|
| [V] Verified | Reproduced in Habbo or directly supported by a user-provided export and test result. State which. |
| [I] Inferred | Suggested by exports or observed behavior, not isolated in a controlled test. |
| [U] Unverified | Unknown; do not rely on it for critical logic. |
| [P] Proposed | A design recommendation, not a claim about game behavior. |

## Current focus
Build a compact test suite for selector composition, signal forwarding, recipient targeting, stack ordering, variable triggers, and movement/queues.

The catalogue and Builders Club availability can vary by hotel and season. Verify live availability separately.
