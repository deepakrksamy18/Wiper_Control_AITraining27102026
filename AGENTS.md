# AGENT.md — Rain-Adaptive Wiper Control System

## 1. Purpose and source of truth
This file guides AI coding agents and engineers working on the Rain-Adaptive Wiper Control System.

- Treat `wiper_control_system_product_requirements.md` as the product-level source of truth.
- Preserve requirement IDs (`PR-xxx`) and verification IDs (`VV-xxx`) for traceability.
- Do not silently invent or finalize values that the PRD marks `TBD`, says must be confirmed, or leaves as an open question.
- If implementation decisions are needed for unspecified behavior, document the assumption, its rationale, risks, and approval needed before treating it as a requirement.
- The PRD is a draft for engineering review; do not represent it as an approved production baseline.

## 2. Product scope
The system combines driver controls and rain-sensor feedback to control an automotive wiper motor. Scope includes:
- Driver input acquisition and mode selection.
- Rain sensing and calibrated intensity estimation.
- Wipe interval and speed control.
- Motor command generation and feedback monitoring where supported.
- Small (S), Medium (M), and Large (L) motor/wiper configurations.
- Torque/current/speed limits, thermal protection, fault handling, diagnostics, and embedded resource budgeting.
- Verification of control behavior, timing, and safety-related constraints.

Mechanical linkage design, detailed power-electronics design, vehicle-network protocol definition, certification/homologation, and final standards allocation are outside scope until specified in the PRD.

## 3. Required control behavior
Implement behavior only in line with approved requirements and configuration data:

- **OFF:** inhibit new wipe requests; perform a safe park sequence only if required and defined by vehicle design.
- **Manual continuous:** use the approved driver-selected continuous setting within configured motor limits.
- **Manual intermittent:** use the configured driver-selected time interval.
- **Automatic with valid sensor data:** map calibrated rain intensity to wipe interval and/or speed.
- **Automatic with invalid/stale sensor data:** use an approved fallback; provide a driver indication only if the vehicle interface supports it.
- **Overload/stall:** apply the approved protective response.
- **Overtemperature:** apply the validated thermal/derating policy while considering visibility safety.

Driver OFF must inhibit new wipe commands, subject to any explicitly defined safe park behavior. Keep driver-command priority deterministic.

## 4. Real-time execution constraints
- The PRD assumes a high-priority sensing/control task at **500 Hz (2 ms period)**.
- This is a software control-loop target, **not** a 500 Hz mechanical wiping rate.
- The task must complete within its 2 ms period under the defined worst-case workload, with engineering-approved timing margin.
- Lower-priority functions must not block the control task or cause deadline misses.
- Include watchdog supervision and defined behavior for task overrun, lockup, reset, and relevant brownout conditions.
- Measure or estimate execution time and expose timing diagnostics where feasible.
- Do not claim timing compliance without measured or otherwise justified worst-case execution-time evidence.

## 5. Motor variants and configuration safety
- Support S/M/L variants through explicit, validated configuration data.
- Do not infer ratings or safe limits from the variant label alone.
- Each approved variant parameter set must include, as applicable: motor/linkage length, voltage range, rated and maximum torque, speed, current, stall response, operating temperature, duty cycle, thermal thresholds, and motor-driver part number.
- Enforce the selected motor/driver operating envelope using available feedback and validated limits.
- Invalid, unsupported, or corrupted configuration must not enable unrestricted motor operation.
- Never fabricate supplier ratings or substitute guessed constants for missing data.

## 6. Thermal and visibility safety
- The PRD includes an eight-hour continuous-operation constraint unless a validated thermal-management solution is approved.
- Do not implement this as an arbitrary hard shutdown that could create an unsafe visibility condition.
- Thermal thresholds, derating, shutdown, and recovery behavior must be based on component specifications and safety analysis.
- Use direct temperature sensing or a validated thermal model, as applicable to the selected design.
- If the required duty cycle exceeds component ratings, request an approved thermal solution or appropriately rated components.
- Treat loss of wiping, unintended wiping, sensor failure, and thermal shutdown as hazards requiring engineering assessment.
- Do not assign an ASIL or claim ISO 26262 compliance without an approved safety analysis and allocated requirements.

## 7. Fault handling and diagnostics
For each supported fault, define and test:
- Detection condition.
- Severity/classification.
- Protective response.
- Recovery conditions.
- Diagnostic record/indication, where supported.

Relevant faults include sensor failure or implausibility, motor overload/stall, overtemperature, invalid configuration, watchdog reset, control-task overrun, startup/reset issues, and brownout where applicable.

Avoid unintended motor activation during startup, reset, brownout, or configuration failure. Do not invent vehicle diagnostic protocols or network interfaces; use only the selected architecture.

## 8. MCU and platform constraints
The PRD permits evaluation of Renesas or TI MCUs but mandates no specific part number.

Evaluate candidate hardware against:
- Worst-case control-task execution time and CPU headroom.
- RAM/flash budgets and memory protection where applicable.
- Timer/PWM, ADC, digital I/O, communication, watchdog, and safety features.
- Environmental rating, automotive qualification, lifecycle, and supply availability.
- Functional-safety capability/evidence if required by the vehicle safety concept.
- Debug, trace, calibration, and production-programming support.

Do not assume a sensor, motor driver, MCU, bus protocol, or peripheral is present unless confirmed by project configuration or an approved design decision.

## 9. Software engineering rules
When modifying or adding code:
1. Inspect the repository structure and existing conventions before editing.
2. Keep control logic deterministic and bounded in execution time.
3. Separate input acquisition, validation/filtering, mode arbitration, rain-intensity mapping, limit enforcement, motor command generation, diagnostics, and platform I/O where practical.
4. Keep calibration and variant data separate from control logic; validate it before use.
5. Avoid blocking operations, unbounded loops, and dynamic allocation in the high-priority control path unless explicitly justified and approved.
6. Use explicit units in names, interfaces, and documentation (for example, `period_ms`, `current_mA`, `temperature_C`).
7. Handle invalid, stale, out-of-range, and unavailable inputs explicitly.
8. Use safe initialization and defined reset behavior; initialize outputs so motor activation cannot occur unintentionally.
9. Add or update tests with every behavior change.
10. Preserve requirement traceability in comments, design notes, and tests where useful; avoid adding requirement IDs to unrelated code.
11. Do not claim a requirement is implemented or verified unless supported by code and test evidence.
12. Do not introduce dependencies, toolchains, coding standards, or architecture layers without checking project conventions and documenting the reason.
13. Follow the standards-driven rules in Sections 11–17 (safety, cybersecurity, updates, process, secure coding, production engineering, sensitive data).

## 10. Verification expectations
Use the PRD verification plan as the starting point. Ensure coverage for:
- `VV-001`: OFF, continuous, and intermittent modes.
- `VV-002`: rain-intensity sweep and calibrated mapping.
- `VV-003`: stale, disconnected, noisy, and out-of-range sensor data.
- `VV-004`: 500 Hz period and worst-case execution time under load.
- `VV-005`: concurrent supporting workload and resource budgets.
- `VV-006`: S/M/L configuration and invalid configuration.
- `VV-007`: overload, stall, and current-limit behavior.
- `VV-008`: approved thermal and duty-cycle profile.
- `VV-009`: watchdog reset, brownout, and startup faults.
- `VV-010`: thermal recovery and driver notification behavior.

Use unit tests for control logic, integration tests for hardware interfaces, fault-injection tests for failure handling, and target-hardware timing/thermal tests where required. Acceptance thresholds must come from approved requirements and calibration—not guesses.

## 11. Applicable standards and process framework
The project is intended to be developed against the standards below. They are **targets for engineering review**, not claims of compliance. Do not state that the product, a work product, or a change "complies with" or "is certified to" any of them without approved assessment evidence.

| Area | Standard / framework | How it applies |
|---|---|---|
| Functional safety | ISO 26262 (ASIL determined by hazard analysis) | Section 12 |
| Cybersecurity | ISO/SAE 21434 | Section 13 |
| Software updates | ISO 24089 | Section 14 |
| Process and quality | Automotive SPICE (ASPICE) | Section 15 |
| Secure coding | MISRA C / MISRA C++ | Section 16 |
| Production engineering | Static analysis, traceability, fault injection, secure release management, human review | Section 17 |

Regulatory context (for example UNECE R155 cybersecurity management and R156 software update management) may apply depending on market and vehicle type. Confirm with the project owner; do not assume.

If standards conflict with each other or with the PRD, raise it for engineering decision rather than choosing silently.

## 12. Functional safety — ISO 26262
- The ASIL for each safety goal is **determined by the hazard analysis and risk assessment (HARA)**. Never assign, assume, or downgrade an ASIL in code, comments, tests, or documents without an approved HARA.
- Until the HARA is approved, label any safety classification as `ASIL: TBD` and treat safety-relevant behavior conservatively.
- Candidate hazards to be assessed (not pre-classified): loss of wiping, unintended wiping, wrong wipe speed/interval for conditions, sensor failure or implausibility, thermal shutdown, and unintended motor activation at startup/reset/brownout.
- Safety work products are expected to flow: HARA → safety goals → functional safety concept → technical safety requirements → software safety requirements → design → verification. Preserve this traceability.
- Code implementing a safety requirement must reference its requirement ID and be traceable to tests.
- Apply freedom-from-interference: non-safety functions must not corrupt, block, or delay safety-related functions (memory protection, partitioning, timing protection as the platform allows).
- Safety mechanisms (watchdog, plausibility checks, current/temperature limits, safe-state entry) need defined diagnostic coverage assumptions, fault reaction time, and verification evidence.
- Software tool confidence (compilers, static analyzers, test tools) must be assessed when required by the allocated ASIL.
- Any change touching safety-related behavior must be flagged for safety-engineer review.

## 13. Cybersecurity — ISO/SAE 21434
- Security activities follow a TARA (threat analysis and risk assessment) → cybersecurity goals → cybersecurity requirements → design → verification flow. Do not invent a Cybersecurity Assurance Level (CAL) or risk rating without an approved TARA.
- Treat all external inputs as untrusted: rain-sensor data, vehicle-network messages, diagnostic requests, calibration/configuration data, and update packages. Validate range, length, integrity, and freshness before use.
- Minimize attack surface: disable or lock debug/test interfaces (for example JTAG/SWD, bootloader, diagnostic services) in production configurations unless an approved, authenticated mechanism exists.
- Use approved, vetted cryptographic libraries and algorithms only. Never implement custom cryptography.
- Do not add network, diagnostic, or remote interfaces that are not part of the approved architecture.
- Maintain an inventory of third-party components (SBOM), track known vulnerabilities, and document the vulnerability handling and incident response path.
- Security-relevant changes require security review, and cybersecurity requirements must be traceable to tests (including negative and fuzz/robustness tests where applicable).

### 13.1 Sensitive data handling
Treat the following as sensitive and handle them under the project's data classification and access rules:
- **Secrets and keys:** passwords, tokens, API keys, certificates, private/signing keys, symmetric keys, seed values, and diagnostic/security-access credentials.
- **Vehicle and personal data:** VIN, ECU serial numbers, vehicle/driver identifiers, location, timestamps linkable to a person, and any data that could identify a driver. Apply data minimization and applicable privacy law (for example GDPR) if such data is ever collected.
- **Proprietary and confidential data:** calibration data, tuning parameters, variant configurations, supplier datasheets and ratings under NDA, safety/security analyses, and unreleased source.

Rules:
- Never commit secrets, keys, credentials, production calibration data, or real vehicle/personal data to the repository, issues, commit messages, logs, or test fixtures. Use clearly fake, synthetic data for tests and examples.
- Keep signing and production keys in a hardware security module or approved key-management system, never in source control or build scripts.
- Do not log sensitive data. Diagnostic and debug output must not expose keys, identifiers, or confidential calibration.
- Do not paste confidential, NDA-covered, or proprietary content into external tools, services, or prompts unless approved. AI agents must not reproduce supplier-confidential values or guess missing ones.
- Run secret scanning in CI and pre-commit; treat any exposed secret as compromised and report it for rotation.
- Define retention, access control, and secure disposal for diagnostic records, test logs, and field data if they contain sensitive content.
- Protect stored sensitive data in ECU memory (read protection, integrity protection, and encryption where the cybersecurity concept requires it).

## 14. Software updates — ISO 24089
- Any software update capability (flash programming, over-the-air, or workshop update) must follow an approved software update engineering concept; do not add update mechanisms outside it.
- Updates must be authenticated and integrity-checked before installation. Reject unsigned, tampered, corrupted, or incompatible packages.
- Enforce version and compatibility checks (hardware variant S/M/L, calibration set, software version) before activating an update, and apply anti-rollback protection where required by the security concept.
- Define safe behavior for interrupted or failed updates (power loss, brownout, reset): the ECU must retain or recover to a known-good state, and the motor must not activate unintentionally during an update.
- Do not perform updates while the vehicle is in a state where wiping loss could be unsafe unless the approved concept allows it.
- Maintain update-related records: software identification and version, release and compatibility information, update package manifest, and update history/diagnostics.
- Updates to safety-related or security-relevant software require impact analysis and appropriate re-verification before release.

## 15. Process and quality — ASPICE
- Follow the project's ASPICE-aligned lifecycle: system requirements → system architecture → software requirements → software architecture and detailed design → implementation → unit/integration/qualification testing → system testing.
- Maintain **bidirectional traceability** between requirements, design, code, tests, and results (`PR-xxx` ↔ design ↔ code ↔ `VV-xxx` ↔ test evidence).
- Keep supporting processes active: quality assurance, configuration management, change request management, problem resolution, and review/verification records.
- Every change should reference a requirement, change request, or issue; keep changes small, reviewable, and attributable.
- Record review outcomes, findings, and resolutions. Do not close findings without evidence.
- The target ASPICE capability level is **TBD**; do not claim an achieved level.
- Keep work-product consistency: when requirements, design, code, or tests change, update the others in the same change set or record why not.

## 16. Secure coding — MISRA C / C++
- Use the MISRA guideline set approved for the project. The exact edition is **TBD** (for example MISRA C:2012 with amendments or MISRA C:2023 for C; MISRA C++:2023 for C++). Do not mix or assume editions without confirmation.
- Treat Mandatory and Required rules as blocking. Advisory-rule violations need justification.
- Every deviation must be recorded with rule ID, location, rationale, risk assessment, and approver (deviation record). Use narrowly scoped, documented suppressions only; no blanket suppression.
- Compiler warnings are enabled at a high level and treated as errors in CI; the toolchain and flags are pinned and documented.
- Avoid undefined, unspecified, and implementation-defined behavior; use fixed-width integer types, explicit casts, and bounded buffers.
- No dynamic memory allocation, recursion, or unbounded loops in the control path; avoid it elsewhere unless approved.
- Validate all inputs and check all return values; do not ignore error codes.
- Prefer simple, statically analyzable constructs. Keep complexity (cyclomatic, nesting, function size) within project limits.
- Do not copy third-party or generated code into safety-related paths without license, quality, and MISRA assessment.

## 17. Production engineering
The following are required gates before a change is considered release-ready:

- **Static analysis:** run MISRA-checking static analysis, compiler warnings, and defect/security analysis in CI. No new unwaived findings. Record the analyzer, version, and configuration used.
- **Traceability:** requirements-to-code-to-test traceability is generated and checked in CI; gaps are reported, not hidden.
- **Fault injection:** exercise sensor faults, stale/out-of-range data, motor overload/stall, overtemperature, configuration corruption, watchdog/reset, brownout, and update interruption. Verify the defined protective response and recovery for each (see `VV-003`, `VV-007`, `VV-009`).
- **Testing and coverage:** unit, integration, and target-hardware tests with structural coverage measured to the level required by the allocated ASIL (TBD until HARA). Report measured coverage; do not estimate it.
- **Secure release management:** releases come from a tagged, reviewed baseline built reproducibly in a controlled CI environment with pinned toolchains; artifacts are signed with protected keys; an SBOM, release notes, known issues, and deviation/waiver list are produced; no debug or test backdoors in production builds.
- **Human review:** all changes require review by a qualified human. Safety-related and security-related changes require review by the corresponding safety or security engineer. AI-generated code and documents must be reviewed with the same rigor and are never self-approved.
- **Release approval:** release requires an explicit sign-off confirming open findings, waivers, and unresolved `TBD` items are acknowledged.

## 18. Recommended folder structure
This is a **recommendation**. Check existing repository conventions first and do not restructure without approval.

```text
.
├── AGENTS.md
├── README.md
├── wiper_control_system_product_requirements.md   # PRD (source of truth)
├── docs/
│   ├── requirements/       # system and software requirements, PR-xxx
│   ├── architecture/       # system/software architecture and design
│   ├── safety/             # HARA, safety concept, FMEA/FTA, safety case (ISO 26262)
│   ├── security/           # TARA, cybersecurity concept, threat models (ISO/SAE 21434)
│   ├── update/             # software update concept and procedures (ISO 24089)
│   ├── process/            # ASPICE plans: project, QA, configuration, change, problem resolution
│   ├── verification/       # test plans, VV-xxx mapping, reports, fault-injection results
│   ├── traceability/       # trace matrices and reports
│   └── decisions/          # decision log, assumptions, open decisions, MISRA deviations
├── config/
│   ├── variants/           # validated S/M/L parameter sets
│   ├── calibration/        # calibration data (no confidential production values in public repos)
│   └── schemas/            # configuration validation schemas
├── src/
│   ├── app/                # input, validation, mode arbitration, rain mapping, limits, motor command, diagnostics, fault handling
│   ├── hal/                # hardware abstraction (timer/PWM, ADC, GPIO, comms, watchdog)
│   ├── platform/           # startup, scheduler, bootloader/update, memory protection
│   └── common/             # shared types, units, utilities
├── test/
│   ├── unit/
│   ├── integration/
│   ├── fault_injection/
│   ├── timing/             # 500 Hz period and WCET tests
│   ├── hil/                # hardware-in-the-loop / target tests
│   └── fixtures/           # synthetic data only
├── tools/
│   ├── static_analysis/    # MISRA and analyzer configuration, deviation records
│   ├── traceability/       # trace generation and checking scripts
│   └── scripts/            # build, flash, and helper scripts (no secrets)
├── ci/                     # pipeline definitions: build, analysis, tests, secret scan, release
├── release/                # release notes, SBOM, update manifests (no private keys)
└── build/                  # generated output (gitignored)
```

Keep secrets, private keys, and production calibration out of the repository; store them in approved secure locations.

## 19. Open decisions — do not guess
The PRD leaves these decisions unresolved:
- Vehicle supply voltage and electrical architecture (e.g. 12 V or 24 V).
- Rain sensor, interface, update rate, and diagnostics.
- Exact scope of the 500 Hz requirement.
- S/M/L motor and linkage details and supplier ratings.
- Availability of park-position, current, and temperature feedback.
- Driver switch positions, intermittent control, and automatic-mode sensitivity behavior.
- Rain-sensor failure fallback.
- Meaning of the eight-hour duty-cycle constraint and visibility-safe response.
- MCU-hosted application workload, CPU/RAM/flash use, and scheduling.
- Vehicle communication and diagnostic protocol.
- Applicable environmental, functional-safety, cybersecurity, and regulatory requirements.
- ASIL(s) from the HARA, and the structural coverage and tool-qualification level they require.
- Cybersecurity Assurance Level(s) and TARA outcomes; applicability of UNECE R155/R156.
- Software update method (flash, OTA, workshop), anti-rollback policy, and failed-update recovery behavior.
- Target ASPICE capability level.
- MISRA C / C++ edition, static analysis tools, and deviation approval process.
- Data classification, privacy obligations, and key-management approach.

When work depends on one of these, identify the dependency and ask for an engineering decision or record a clearly marked temporary assumption. Do not present an assumption as an approved requirement.

## 20. Definition of done
A change is ready for review when:
- The behavior maps to one or more approved requirements or is explicitly labeled as a proposal.
- Input validation, boundary cases, and relevant fault paths are covered.
- Tests are added or updated and results are reported accurately.
- Real-time and memory impact are considered for embedded changes.
- Motor and thermal limits use approved configuration data.
- No unresolved TBD is silently converted into a fixed production value.
- Documentation and requirement/test traceability are updated.
- Any safety-relevant behavior is flagged for appropriate engineering review.
- Static analysis and MISRA checks pass, with any deviations recorded and approved.
- Security impact is assessed, and no secrets or sensitive data are introduced.
- Fault-injection tests cover the changed behavior where applicable.
- Human review is complete, including safety/security review where required.

## 21. Agent response format
For substantial implementation work, report:
1. **Summary** — what changed.
2. **Requirements covered** — relevant `PR-xxx` and `VV-xxx` IDs.
3. **Design decisions and assumptions** — distinguish approved facts from proposals.
4. **Tests** — commands run, outcomes, and gaps.
5. **Risks / open decisions** — unresolved inputs, safety/security reviews, or hardware dependencies.
6. **Standards impact** — ISO 26262, ISO/SAE 21434, ISO 24089, ASPICE, and MISRA aspects affected, static analysis results, deviations, and sensitive-data considerations.

Be concise, factual, and explicit about what has not been validated on target hardware.
