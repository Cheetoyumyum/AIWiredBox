# Habbo Wired test handbook

## Purpose
Turn uncertain Wired behavior into small, reproducible experiments. Reduce live-room trial-and-error and prevent designs based on assumptions.

## A good test
- Answers one narrow question.
- Changes one variable at a time.
- Uses uniquely named temporary variables.
- Has one trigger path.
- Produces an obvious visible result: chat, variable display, movement, or state change.
- Avoids unrelated furniture and parallel listeners.
- Can be reset and repeated.

Do not combine selector behavior, signal forwarding, and recipient scope in one large game build.

## Standard workflow
1. Question: what exact behavior is unknown?
2. Hypotheses: what are the plausible outcomes?
3. Fixture: what is the minimum room/preset setup?
4. Prediction: what should happen under each hypothesis?
5. Run: follow the steps exactly.
6. Observe: record messages, selected entities, values, movement, and timing.
7. Export: save the preset if configuration details matter.
8. Classify: verified, inferred, unverified, or failed.
9. Document: update the mechanic page and backlog.
10. Regression: add a repeatable check for important behavior.

## Isolation and safety
- Use a dedicated test area or disposable room where possible.
- Prefix test variables with a unique namespace such as t26_.
- Do not redefine production variables; variable definitions can create duplicates on import.
- Do not run multiple experimental stacks from one signal unless concurrency is the subject.
- Avoid tests that transfer currency, furniture, or other value.
- For movement tests, clear the path and ensure nobody can be trapped.
- Keep a copy of the last known-good preset before importing.
- Clean up temporary variables and furniture when appropriate.

## Evidence hierarchy
1. Repeated in-game result with exact setup and steps.
2. Single in-game result with exact setup and steps.
3. Export diff isolating one UI option.
4. Existing working preset using the mechanic.
5. Inference from classname, option values, or naming.
6. Guess.

An export establishes configuration, not runtime semantics.

## Failure triage
1. Did the trigger fire?
2. Did the stack execute?
3. Did conditions pass?
4. Was the target set correct?
5. Was the correct recipient context used?
6. Did the effect run and apply?
7. Was the variable present and correctly scoped?
8. Did another stack mutate state first?
9. Were coordinates, Z order, source IDs, or antenna references wrong?

Change one thing, then repeat.

## Regression suite
Maintain repeatable tests for selector comparison/filter composition, signal forwarding/recipient scope, ordered effects, variable create/change/remove behavior, bounded-loop termination, and movement to a selected destination.
