---

description: "Actionable tasks for the TO-BE adaptive HTTP pool architecture deliverable"
---

# Tasks: Adaptive HTTP Pool Management

**Input**: Design documents in `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/`

**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `quickstart.md`

**Scope**: Documentation-only architecture proposal. Do not implement Java code or authorize production changes in this feature.

**Organization**: Tasks are grouped by the three user stories in `spec.md`. All story tasks update the same architecture document and therefore follow the specified order.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: The task can run in parallel with another task because it works on a different output and has no unfinished prerequisite.
- **[Story]**: Maps the task to its user story in `spec.md`.
- Every task names its target file path.

## Phase 1: Setup

**Purpose**: Create the final architecture deliverable scaffold.

- [X] T001 Create `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md` with the Spanish document title, purpose, proposal-versus-current-state legend, and a relative link to `specs/005-implement/entregables/005.11-arquitectura-eventos-AS-IS/arquitectura-eventos-AS-IS.md`.

---

## Phase 2: Foundational

**Purpose**: Establish the evidence and decision boundaries used by every story.

- [X] T002 Add a current-state evidence and open-decisions section to `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md`, citing the relevant configuration and incident evidence under `ORIONORDERS/`, distinguishing observed facts from proposed behavior, and stating that the Java HTTP client/version, approved downstream budget, and tuning values are not established.

**Checkpoint**: The document has a traceable AS-IS baseline and does not present assumptions or pending owner decisions as facts.

---

## Phase 3: User Story 1 - Review the proposed TO-BE architecture (Priority: P1)

**Goal**: Let architecture and operations reviewers understand how the proposal extends the existing order and audit flows.

**Independent Test**: Review `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md` and confirm its diagram retains the synchronous `orders-api` to `customer-profile-service` request path and separate asynchronous audit path, while placing the proposed pool control loop within `orders-api` and showing both local and aggregate capacity limits as approval gates.

### Implementation for User Story 1

- [X] T003 [US1] Extract the original diagram from `specs/005-implement/entregables/005.11-arquitectura-eventos-AS-IS/arquitectura-eventos-AS-IS.pdf` and extend it in `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE-diagrama.png`, preserving the AS-IS image and highlighting its existing HTTP pool while adding the proposed control loop.
- [X] T004 [US1] Explain in `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md` how the TO-BE overlay extends `specs/005-implement/entregables/005.11-arquitectura-eventos-AS-IS/arquitectura-eventos-AS-IS.pdf`; preserve the existing synchronous profile and asynchronous audit flows and label new behavior as proposed.

**Checkpoint**: The TO-BE diagram and narrative are independently reviewable against User Story 1's acceptance scenarios.

---

## Phase 4: User Story 2 - Adapt capacity safely under rising demand (Priority: P1)

**Goal**: Describe an evidence-based, damped increase in capacity that protects active requests and the shared downstream.

**Independent Test**: Review the capacity-control section in `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md` against rising-load and transient-load scenarios; verify the signals are distinguishable, increases require sustained corroborated evidence, and neither local nor aggregate approved limits can be exceeded.

### Implementation for User Story 2

- [X] T005 [US2] Document in `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md` how local pool acquisition wait, pending acquisitions, active/idle/leased connections, and effective limits are distinguished from downstream latency, errors, timeouts, throttling, and health signals.
- [X] T006 [US2] Specify in `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md` the proposed sustained-pressure evaluation, gradual bounded increases, separate increase/decrease criteria, hold-down or damping, and the Java execution context; keep thresholds, steps, intervals, and library choice pending evidence and owner approval.
- [X] T007 [US2] Define in `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md` the per-instance and aggregate `customer-profile-service` ceilings, including HPA replica changes and temporary pool-generation overlap; state that automatic increases remain disabled until the downstream owner approves the budget and its enforcement behavior.

**Checkpoint**: The capacity increase proposal can be reviewed without assuming downstream headroom or inventing operating limits.

---

## Phase 5: User Story 3 - Reclaim idle capacity and handle limits (Priority: P2)

**Goal**: Describe safe idle-connection retirement and controlled behavior at capacity ceilings or when observations are unusable.

**Independent Test**: Review `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md` against falling-demand, active-request, ceiling, stale-telemetry, and disable/rollback scenarios; verify only idle connections are retired, in-flight requests are preserved, and alerts and safe behavior are specified.

### Implementation for User Story 3

- [X] T008 [US3] Document in `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md` the idle-retirement eligibility rules and active-request lifecycle; if the actual Java client requires pool replacement, describe routing new work to a new generation, draining the old generation until idle, and counting overlap against the aggregate budget.
- [X] T009 [US3] Specify in `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md` ceiling behavior, bounded pending work, actionable alerts, stale/missing/contradictory telemetry safe-hold behavior, and operator disable/rollback without closing active requests.
- [X] T010 [US3] Add a validation matrix to `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md` covering rising and falling load, downstream distress, local and aggregate ceilings, active requests during drain/disable, stale signals, replica-count changes, and rollback; include the approved SLO targets from `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/spec.md` without claiming that capacity growth alone guarantees recovery.
- [ ] T014 Confirm with application/operations and the `customer-profile-service` owner the review status and validation protocol (load profile, runs per scenario, measurement windows, and interruption-attribution method); record approvals, pending decisions, and agreed values in `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md` without inventing limits or test parameters.

**Checkpoint**: The retirement, limit, degraded-observation, and validation behaviors can be reviewed independently against User Story 3's acceptance scenarios.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Export and verify the complete architecture package.

- [ ] T011 [P] Check for an existing PDF export workflow and, if available, render `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md` to `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.pdf`; if no supported workflow is available, record that limitation in `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/tasks.md` rather than introducing an unapproved dependency.
- [ ] T012 [P] Review `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md` against FR-001–FR-014 and SC-001–SC-006 in `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/spec.md`; correct missing requirements, broken relative links, unsupported numeric defaults, or confusion between observed facts and proposed components.
- [ ] T013 Compare `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.pdf` with `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/arquitectura-eventos-TO-BE.md`; verify that the original AS-IS PDF diagram is visibly preserved in the TO-BE overlay and that headings, links/references, limits, and SLOs remain legible and consistent.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: T001 creates the architecture document scaffold.
- **Foundational (Phase 2)**: T002 depends on T001 and establishes the evidence baseline for all stories.
- **User Stories (Phases 3–5)**: US1 follows T002; US2 follows US1; US3 follows US2 because each increment extends the same architecture document and diagram.
- **Polish (Phase 6)**: T014 follows T010 and records the agreed review status and validation protocol; T011 and T012 follow T014 and can run in parallel; T013 follows both and checks the final package.

### User Story Dependencies

- **User Story 1 (P1)**: Depends on the setup and evidence baseline; creates the architecture flow and explanation.
- **User Story 2 (P1)**: Depends on US1's architecture boundary; adds the safe adaptive increase policy.
- **User Story 3 (P2)**: Depends on US2's capacity policy; adds safe retirement, ceiling, and fallback behavior.

### Parallel Opportunities

- After T014 records the review status and validation protocol, **T011 (PDF export)** and **T012 (requirements review)** can run in parallel because they work on separate outputs/read-only review.
- Story implementation tasks are intentionally sequential because they extend the same architecture document and depend on the flow and capacity policy established by earlier stories.

### Dependency Graph

```text
T001 -> T002 -> T003 -> T004 -> T005 -> T006 -> T007 -> T008 -> T009 -> T010 -> T014
                                                                                       |-> T011 --|
                                                                                       |-> T012 --|-> T013
```

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete T001 and T002 to establish the document and evidence boundaries.
2. Complete T003 and T004 to deliver the proposed flow and its relationship to the AS-IS architecture.
3. Stop and validate User Story 1 independently using its diagram-and-boundaries acceptance criteria.
4. Treat this as an architecture-review MVP only; do not present it as an implemented or production-ready controller.

### Incremental Delivery

1. Add US1's architecture flow and review it with architecture/operations.
2. Add US2's bounded increase policy and review owner approvals for capacity limits.
3. Add US3's safe idle retirement, ceiling handling, rollback, and validation matrix.
4. Export and cross-check the final Markdown/PDF package where PDF tooling is available.

## Notes

- Application tests are outside this documentation-only scope; T014 records the agreed protocol required before future controlled validation.
- The Markdown architecture document is canonical; no Java source, external API contract, arbitrary threshold, or production change is in scope.
- `[P]` tasks operate on separate outputs after the architecture content is complete.
