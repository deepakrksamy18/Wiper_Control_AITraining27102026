# Rules and standards applied to unit test specs

Source of truth for project rules: `AGENTS.md`. The standards below are **proposed, not approved** (PRD Q11, AGENTS.md §11). Never state that a test or spec "complies with" a standard.

## Project rules (from AGENTS.md)
| Rule | Effect on the spec |
|---|---|
| §1 Preserve `PR-xxx` and `VV-xxx`; do not invent values marked `TBD` | Keep IDs exact; `TBD` stays `TBD`; rows become `Blocked` |
| §5 No inferring limits from the S/M/L label | Variant rows use synthetic validated parameter sets and a missing parameter makes the set invalid |
| §4 and §9 Deterministic, bounded control path | Spec avoids cases that require dynamic allocation, blocking, or unbounded loops; add a timing-bound case only when the story states one |
| §7 Fault handling: detection, severity, response, recovery, indication | Each supported fault gets cases for each of these five elements |
| §9.6 Explicit units | Units in names and values (`period_ms`, `current_mA`, `temperature_C`) |
| §9.7 Handle invalid, stale, out-of-range, unavailable inputs | Every input has invalid-class cases |
| §9.8 Safe initialization | Include initialization and reset cases that assert inactive motor output |
| §10 `VV-001` to `VV-010` coverage | Map rows to the matching `VV-xxx`; list gaps |
| §13 Untrusted inputs; §13.1 sensitive data | Add robustness and negative cases; fixtures are synthetic only |
| §17 Fault injection, coverage measured not estimated | Include fault-injection cases; coverage target `TBD (ASIL TBD)` |
| §20 Definition of done | Traceability and fault-path coverage are part of the spec |

## Proposed standards and how they shape the spec
| Standard (proposed) | Use in this skill |
|---|---|
| ISO 26262 (software unit verification) | Requirements-based tests, equivalence classes, boundary values, error guessing, and fault injection as selection techniques. Method strength and coverage depend on the ASIL, which is `TBD`; do not choose strength levels. Mark safety-relevant cases `safety_review = Yes`. |
| ISO/SAE 21434 | Robustness and negative cases for untrusted inputs; mark `security_review = Yes` for those. No CAL assigned. |
| ISO 24089 | If a story covers update handling (not present in the current stories), include cases for rejected, tampered, incompatible, and interrupted updates. |
| Automotive SPICE (unit verification) | Bidirectional traceability: requirement, story, AC, test case. Keep consistent with design documents. Capability level `TBD`. |
| MISRA C++ (edition TBD) | The spec must not require code that breaks the rules (no dynamic allocation, recursion, blocking, or unbounded loops in the control path). Test harness code is outside the control path but follows project conventions. |

## Which cases are safety-relevant (`safety_review = Yes`)
Cases for: loss or wrong wiping, unintended wiping, sensor failure or fallback, motor limits, stall or overload response, thermal protection and recovery, watchdog and overrun behavior, startup, reset, and brownout, configuration validation, persistent settings integrity.

## Which cases are security-relevant (`security_review = Yes`)
Cases that handle untrusted data: sensor data validation, driver input validation, configuration and calibration validation, diagnostic requests, update packages.

## Technique selection guide
| Situation | Technique |
|---|---|
| Behavior stated by an AC | Requirements-based |
| Numeric range, threshold, interval, timeout | Boundary value (below, at, above) and Equivalence class |
| Mode and state behavior (modes, fallback, thermal states) | State/transition and Decision table |
| Bad, missing, stale, or contradictory input | Equivalence class (invalid) and Error guessing |
| Detector or protective response | Fault injection |
| Untrusted data | Robustness/negative |
| Startup, reset, brownout | Fault injection with `Initialization/Reset` category |

## Not verified by a unit test spec
Timing compliance on target (WCET, 500 Hz jitter), thermal behavior on hardware, motor sizing, and hazard analysis results. Where a story needs them, record the unit-level portion and name the other verification activity in `notes`.
