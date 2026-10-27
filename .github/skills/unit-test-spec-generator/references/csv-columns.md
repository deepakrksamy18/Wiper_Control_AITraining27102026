# CSV column definitions

File format: UTF-8, comma-separated, header row required, RFC 4180 quoting (double any `"` inside a quoted cell). Use `;` to separate multiple values in one cell. One row per test case. Columns appear in this exact order.

| # | Column | Required | Allowed values / content |
|---|---|---|---|
| 1 | `test_id` | Yes | `UT-<story number>-<nnn>`, unique, for example `UT-001-003` |
| 2 | `story_id` | Yes | `US-xxx` |
| 3 | `requirement_ids` | Yes | `PR-xxx` list; blank only with a note for stories with no PR (for example enablers) |
| 4 | `verification_ids` | No | `VV-xxx` list; blank if the PRD has none, with "no VV in PRD" in `notes` |
| 5 | `acceptance_criterion` | Yes | `AC-n` as numbered in the story, plus short text |
| 6 | `unit_under_test` | Yes | Module or function; `TBD (design pending)` if no design exists |
| 7 | `test_title` | Yes | One line |
| 8 | `technique` | Yes | `Requirements-based`; `Equivalence class`; `Boundary value`; `Error guessing`; `Fault injection`; `Robustness/negative`; `State/transition`; `Decision table` |
| 9 | `test_level` | Yes | `Unit (host)`; `Unit (target)`; `Component` (only if the story requires it) |
| 10 | `category` | Yes | `Nominal`; `Boundary`; `Invalid input`; `Fault`; `Security`; `Initialization/Reset`; `Variant` |
| 11 | `preconditions` | Yes | State before the test; `None` if none |
| 12 | `stubs_fixtures` | Yes | Stubs, mocks, synthetic data, simulated clock; only from the story's dependencies |
| 13 | `inputs` | Yes | Input values with units; `TBD` where not approved |
| 14 | `expected_result` | Yes | Observable output or state; `TBD` where not approved |
| 15 | `pass_basis` | Yes | Source of the expected result: `PR-xxx`, `AC-n`, approved calibration, or `TBD` |
| 16 | `variant_scope` | Yes | `All`; `S`; `M`; `L`; `S;M;L` |
| 17 | `asil` | Yes | Always `TBD` until an approved hazard analysis |
| 18 | `safety_review` | Yes | `Yes` or `No` |
| 19 | `security_review` | Yes | `Yes` or `No` |
| 20 | `structural_coverage_target` | Yes | `TBD (ASIL TBD)` until allocated |
| 21 | `blocking_decisions` | Yes | `Q#` ids or named missing decision; `None` if not blocked |
| 22 | `status` | Yes | `Draft` or `Blocked` |
| 23 | `notes` | No | Short remarks, gaps, assumptions marked "Assumption:" |

Header line (copy exactly):

```
test_id,story_id,requirement_ids,verification_ids,acceptance_criterion,unit_under_test,test_title,technique,test_level,category,preconditions,stubs_fixtures,inputs,expected_result,pass_basis,variant_scope,asil,safety_review,security_review,structural_coverage_target,blocking_decisions,status,notes
```

## Rules for cell content
- `status` is `Blocked` whenever any of `inputs`, `expected_result`, or `pass_basis` contains `TBD`.
- `asil` is never anything but `TBD` in this project phase.
- Numeric values appear only if they come from the PRD, the story, or approved calibration. Otherwise write `TBD`.
- Boundary rows state the boundary as a symbolic reference when the value is `TBD` (for example `just below debounce_time_ms`).
- Keep cells short; one observable result per row.
