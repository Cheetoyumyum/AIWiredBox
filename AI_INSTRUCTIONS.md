# Instructions for AI builders

Read this file and the relevant knowledge-base sections before designing or editing a preset.

## Evidence discipline
- Use [V], [I], [U], and [P] confidence labels consistently.
- Never silently upgrade an inference into a verified fact.
- For uncertain options or behavior, propose a minimal calibration test.
- A JSON export proves what was configured, not necessarily what happened at runtime.
- Preserve exact user observations separately from your interpretation.

## Before building
1. Inspect the current knowledge base and relevant reference exports.
2. Identify the exact mechanics required and their confidence.
3. Check variable names globally for collisions.
4. Check stack coordinates, Z ordering, antenna placement, and source references.
5. Avoid redefining room variables already present in the live room.
6. Estimate Wired/furniture counts.
7. Prefer one small, independently importable subsystem per preset.

## Design rules
- A selector produces a target set; do not assume it creates a per-target loop.
- Keep triggering-user, users-from-signal, and users-from-selector contexts distinct.
- Use variables for state and signals for coordination.
- Initialize variables only when required; do not auto-create every missing player variable.
- Use conditions for straightforward gates rather than redundant flags.
- Geometry and Z matter; JSON list order is not a substitute for stack layout.
- Use Execute in Order where ordered effects are required.
- Avoid parallel stacks that mutate shared state without tested synchronization.
- Keep loops bounded and document their exit condition.
- Never modify a known working project unless explicitly asked.

## Test-first workflow
1. Write one falsifiable question.
2. State plausible outcomes.
3. Build the smallest test that distinguishes them.
4. Use unique temporary variable names.
5. Include only the functional Wired/furniture needed.
6. Give exact run instructions.
7. Record actual observations, not predictions.
8. Update the report, backlog, and mechanic documentation.
9. Add a regression test for important confirmed behavior.

## Preset validation
- [ ] JSON parses and required top-level keys exist.
- [ ] Every Wired instance maps to a furni instance.
- [ ] Furni IDs are unique within the preset.
- [ ] Variable names and references resolve.
- [ ] No collisions with existing room variables.
- [ ] Stack members share intended X/Y coordinates.
- [ ] Z heights reflect intended order.
- [ ] Selector source, filter, and inverse are deliberate.
- [ ] Signal antenna references and forwarded sources are deliberate.
- [ ] Recipient context is explicit.
- [ ] Loops have a proven stop condition.
- [ ] Classnames are supported by an export or verified source.
- [ ] Test instructions and expected observations are included.

## Required build summary
Purpose; verified versus assumed mechanics; furniture and Wired count; variables/scopes; trigger and stack flow; selector and recipient context; signal path; import instructions; exact tests; limitations and rollback.

A build is not working until runtime behavior has been tested in Habbo.
