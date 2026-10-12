# Validation Quickstart: Adaptive HTTP Pool Architecture

This feature phase produces an architecture proposal, not runnable Java code. The supplied workspace contains no `orders-api` source tree or build manifest. Use this guide with an approved test environment and load harness when the implementation exists; do not run load or fault injection against production without explicit authorization.

## Prerequisites

- Review the [TO-BE specification](./spec.md), [research decisions](./research.md), and [conceptual data model](./data-model.md).
- Confirm that the proposed Apache HttpClient 5 dependency/version is compatible with the actual Java `orders-api` and test live adjustment, idle release, and active-request drain semantics.
- Confirm Prometheus/Micrometer registry/exporter availability and the mapping from pool occupancy, queue, latency, and error measurements.
- Obtain approved per-instance and aggregate `customer-profile-service` capacity limits, along with controller windows, steps, and idle-retirement policy.
- Use a non-production environment with request-level correlation, pool metrics, downstream health metrics, and representative load.
- Confirm that production changes are out of scope for this validation and that test rollback/disable steps are available.

## Run the validation scenarios

For every scenario, record the policy version, test demand, replica count, pool observations, downstream outcomes, controller decision, completed requests, and SLO measurements. Compare outcomes with [data-model.md](./data-model.md).

| Scenario | Setup and action | Expected result |
|---|---|---|
| Baseline / shadow mode | Run representative steady traffic with adjustment in shadow mode. | Decisions are logged but no pool limits change; local pool wait remains distinguishable from downstream latency. |
| Sustained rising demand | Increase load gradually until approved sustained-pressure criteria are met. | A preventive signal occurs before the approved maximum; any increase is bounded, logged, and does not exceed local or aggregate limits. |
| Illustrative steps | From the observed per-instance value of 20, exercise 20 → 30 → 40 only when each step is configured and approved. | The tuner evaluates threshold, pressure duration, and cooldown; maximum and health guardrails block unsafe steps. These candidate values do not set or imply the aggregate downstream budget. |
| Brief burst | Apply a short burst that does not meet the sustained condition. | No repeated or unnecessary capacity change; decision is held or logged as transient. |
| Downstream distress | Increase downstream latency or inject approved downstream errors/throttling while local wait is present. | Controller freezes increases or backs off according to policy; it does not infer spare downstream capacity from local pool wait. |
| Idle retirement with active work | Reduce demand while some connections are active and others idle past the approved threshold. | Eligible idle connections retire; active requests finish; active connections are not forcibly closed. |
| Local and aggregate ceilings | Exercise capacity to each approved ceiling, including HPA scaling and any overlapping pool generations. | No ceiling is exceeded; pending work follows its bound; an actionable cap-reached event is emitted. |
| Stale or contradictory observations | Omit, delay, corrupt, or make local/downstream measurements inconsistent. | Automatic increase pauses, known bounded state remains, telemetry/health alert is visible, and no false recovery is reported. |
| Disable or rollback | Disable enforcement during active requests; if using generations, switch new requests to the stable generation. | New decisions stop; requests already in progress complete; old connections are retired only after draining. |
| Replica-count change | Repeat pressure and ceiling scenarios from minimum to maximum configured HPA replicas. | Aggregate connection use stays within the owner-approved budget across replica count changes. |

## Acceptance checks

- All requests active at the start of increase, decrease, drain, disable, and rollback scenarios complete without controller-driven interruption.
- No test run exceeds either approved local limits or aggregate downstream budget.
- Pressure, adjustment, hold, stale-input, cap, and idle-retirement decisions are attributable to fresh evidence and a policy version.
- No increase occurs from stale or contradictory telemetry or while downstream distress is present.
- Every proposed increase honors the configured per-instance maximum, downstream health/concurrency limits, request timeouts, and the owner-approved aggregate budget.
- Within the approved downstream budget, validation checks the current OrionOrders targets: monthly availability 99.95%, p95 < 1.2 s, and p99 < 2.5 s.
- The responsible teams review and approve thresholds, limits, and the result before recommending enforcement.

## Current incident evidence for architecture walkthrough

Use the documented sample `leased=20 max=20 pending=19` at 09:43:20 together with `clientPoolWaitMs` and the corresponding downstream request duration to review which signals the proposed controller should compare. This is an architecture walkthrough only; the sample does not authorize an increase or establish spare downstream capacity.
