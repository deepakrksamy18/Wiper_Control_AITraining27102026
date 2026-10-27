---
name: unit-test-spec-generator
description: 'Generate unit test specifications in CSV format for the Rain-Adaptive Wiper Control System from user stories, design documents, and the PRD, applying project rules (AGENTS.md) and proposed automotive standards. Use for: unit test spec, test case spec CSV, test design, boundary and fault-injection cases, requirement-to-test traceability, VV-xxx coverage, ISO 26262 unit verification, MISRA C++ test considerations. Produces specs only, not test code.'
argument-hint: 'Story IDs (for example US-001 US-008) or "all"; optional design-doc path'
---

# Unit Test Spec Generator (CSV)

Produces one CSV unit test specification per user story. The CSV is the input for later test implementation and for requirement analysis. It contains no test code.

## When to Use
- A user story or design document exists and unit test cases must be specified.
- Traceability from acceptance criteria to `PR-xxx` and `VV-xxx` to test cases is needed.
- Test coverage gaps or blocked tests (open `TBD` values) must be made visible.

Do not use for integration, system, HIL, or timing-on-target specs without saying so; this skill covers unit-level logic only. Target-hardware cases may be listed with `test_level` set accordingly, but only when the story requires them.

## Inputs (read in this order)
1. Design document in `docs/design docs/US-xxx-*.md`, if it exists.
2. `docs/requirements/user_stories.md` (acceptance criteria, dependencies, "Independent test", open decisions).
3. `docs/wiper_control_system_product_requirements.md` (`PR-xxx`, `VV-xxx`, `Q1`-`Q11`).
4. `AGENTS.md` (project rules; wins over this skill on conflict).
5. [rules and standards](./references/rules-and-standards.md) (how the rules shape test cases).
6. [CSV column definitions](./references/csv-columns.md) and [template](./assets/unit-test-spec-template.csv).

## Output
- `docs/verification/unit-test-specs/US-xxx-unit-test-spec.csv` (one per story, columns exactly as in the column reference).
- `docs/verification/unit-test-specs/unit-test-spec-index.csv`, one row per story: `story_id, spec_file, test_count, blocked_count, ac_count, ac_without_test, review_flags`.

If the user wants a different folder, use theirs and say so.

## Procedure
1. **Scope.** Resolve the requested story IDs (or all). If a story is missing, ambiguous, or contradicts the PRD, stop on it and ask the user. Do not resolve silently.
2. **Read** the inputs above. Note each acceptance criterion (AC-1, AC-2, ... in story order), the dependencies, and open decisions.
3. **Identify units.** Name each unit under test (module or function) from the design document. If no design document exists, use the story's component description and record `unit_under_test` as `TBD (design pending)`.
4. **Derive test cases** per acceptance criterion using these techniques (see the rules reference):
   - Requirements-based (nominal) cases.
   - Equivalence classes and boundary values for every range, interval, threshold, and timeout.
   - Invalid, stale, out-of-range, missing, and contradictory inputs.
   - Fault injection and protective-response checks where the story has fault behavior.
   - Robustness and negative cases for untrusted inputs (sensor, driver, configuration, calibration, diagnostic, update data).
   - Initialization, reset, and no-unintended-motor-activation cases where outputs are involved.
   - Variant cases (S, M, L) only for stories whose variant scope is S/M/L, using synthetic validated parameter sets, never inferred ratings.
5. **Make each case independently runnable.** List the stubs, mocks, and synthetic fixtures needed, using only the story's listed dependencies. No case may need another story to be implemented.
6. **Handle unknowns.** Where a threshold, value, timing, or behavior is `TBD`, write `TBD` in `expected_result` or `inputs`, set `status` to `Blocked`, and put the open question (`Q#`) or missing decision in `blocking_decisions`. Never fill in a value.
7. **Flag reviews.** Set `safety_review` and `security_review` to `Yes` for safety-relevant or security-relevant cases (see the rules reference). Keep `asil` as `TBD`.
8. **Write the CSV** following the column reference (UTF-8, header row, RFC 4180 quoting, `;` between multiple values in one cell).
9. **Validate** (see below), fix issues, then write the index CSV.
10. **Report** to the user: files created, counts, blocked cases grouped by `Q#`, acceptance criteria without tests, questions raised, and what was not validated.

## Validation checklist
Run and report each item:
- Every AC has at least one test row; list any that do not.
- Every `PR-xxx` and `VV-xxx` in a row exists in the PRD and in the story. A requirement with no `VV-xxx` stays blank and the row notes "no VV in PRD".
- Every row traces to a story, an AC, and a unit under test.
- Each range or threshold has boundary cases (just below, at, just above) or the row is `Blocked` with a reason.
- Every `TBD` is in `blocking_decisions` and `status` is `Blocked`; no invented numeric values, ratings, or thresholds.
- Inputs and outputs use explicit units in names (`period_ms`, `current_mA`, `temperature_C`).
- Test IDs are unique and follow `UT-<story number>-<nnn>` (for example `UT-001-003`).
- No secrets, real vehicle or personal data, or confidential supplier values; fixtures are synthetic.
- No case requires dynamic allocation, blocking operations, or an unbounded loop in the control path of the unit under test.
- CSV is well formed. Check with PowerShell:
  `Import-Csv <file> | Measure-Object` and confirm the header equals the column reference and no row has empty `test_id`.

## Rules to enforce (summary)
- Preserve `US-xxx`, `PR-xxx`, and `VV-xxx` exactly. Do not invent requirement or verification IDs.
- Do not assign an ASIL or claim compliance with any standard. Standards in AGENTS.md §11-17 are proposed.
- Structural coverage targets are `TBD` until the hazard analysis allocates an ASIL. Report measured coverage later; never estimate it in the spec.
- Pass criteria come from approved requirements and calibration, not guesses.
- Safety-relevant cases (protective responses, safe states, thermal, overload, startup, watchdog) and security-relevant cases (untrusted input handling) are flagged for engineering review.
- A test case never claims a requirement is verified; the spec only states what will be tested.

## Completion criteria
The skill is done when the CSVs exist, the validation checklist is reported with results, blocked cases and open questions are listed, and the user has been told what is unverified.

## Example prompts
- `/unit-test-spec-generator US-001 US-008`
- `/unit-test-spec-generator all`
- "Create unit test specs for US-023 using the design doc in docs/design docs."
