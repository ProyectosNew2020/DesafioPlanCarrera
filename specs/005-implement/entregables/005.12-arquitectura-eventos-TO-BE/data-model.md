# Data Model: TO-BE Adaptive HTTP Pool

This is a conceptual operational model for the architecture proposal, not a database schema. Exact thresholds, durations, and limits remain owner-approved configuration values.

## Entities

### Pressure Observation

Represents a time-bounded view of client-pool and downstream behavior.

| Field | Meaning | Validation |
|---|---|---|
| observation time | When the values were sampled | Must be present and within the configured freshness window |
| instance identity | `orders-api` instance/pod that produced the observation | Must identify the reporting instance |
| active, idle, and leased connections | Current pool connection states | Non-negative; leased/active values must be consistent with the client’s semantics |
| effective and configured limits | Current runtime limit and approved ceiling | Positive; effective limit must not exceed its approved local ceiling |
| pending acquisitions | Requests waiting for a connection | Non-negative; measured distinctly from downstream request time |
| acquisition wait | Time waiting locally to obtain a connection | Non-negative; not interchangeable with network latency |
| downstream latency and outcomes | Request time, errors, timeouts, throttling, and saturation signals, where available | Must carry freshness and measurement scope |
| signal quality | Whether values are complete, fresh, and mutually usable | Invalid, stale, or contradictory input is insufficient evidence |

### Capacity Policy

Represents approved limits and controller parameters for the service and shared downstream.

| Field | Meaning | Validation |
|---|---|---|
| policy identity/version | Identifies the approved policy | Changes are auditable and reversible |
| per-instance minimum and maximum | Local connection bounds | Minimum is positive; maximum is not below minimum |
| aggregate downstream budget | Fleet-wide connection ceiling | Must be approved with the downstream owner before automatic increases |
| increase/decrease criteria | Sustained signal conditions and adjustment sizes | Calibrated by load tests; no invented defaults |
| sampling and hold-down periods | Conditions that damp oscillation | Positive, bounded, and explicitly approved |
| proposed controller/client | Java `Pool Auto-Tuner` and Apache HttpClient 5 integration | Proposed only; compatible client version and runtime pool-update behavior must be verified before implementation |
| illustrative increase steps | Example per-instance progression `20 → 30 → 40` | Candidate steps only; not approved ceilings or evidence of aggregate downstream capacity |
| health/concurrency guardrails | Maximum pool size, downstream health, concurrency limit, and request timeouts | Must block or pause increases when violated; values require owner approval |
| metrics instrumentation | Prometheus/Micrometer observations for occupancy, queue, latency, and errors | Metric freshness, scope, and registry/exporter integration must be validated |
| idle-retirement policy | Eligibility and minimum warm capacity | May select idle connections only; must not evict active work |
| maximum pending-work policy | Behavior at capacity ceiling | Bounded and paired with operator-visible alerting |
| enabled mode | Disabled, shadow, or enforcing | Enforcing mode requires accepted evidence, approved budgets, and authorization |

### Connection and Request Lifecycle

Represents a transport connection and work associated with it.

| Field | Meaning | Validation |
|---|---|---|
| connection generation | Pool/client generation owning the connection | Used only if the client requires pool replacement |
| state | Available, active, draining, or idle-retirement eligible | A connection with active work cannot be closed as idle |
| in-flight request count | Requests currently using the connection | Non-negative; retirement requires zero |
| last-used time | Last time the connection carried work | Used with policy freshness to determine eligibility |
| request completion state | Pending, completed, failed, or cancelled | Administrative retirement must not convert an in-flight request into a forced cancellation |

### Adjustment Decision

Represents one controller evaluation and its outcome.

| Field | Meaning | Validation |
|---|---|---|
| decision time and instance | Where and when the decision occurred | Required |
| evidence reference | Observations and policy version used | Must be sufficient and fresh for an enforcing decision |
| action | Increase, decrease/drain, hold, reject, or disable | Must be allowed by current mode and limits |
| capacity before/after | Effective limits before and after action | Must remain within local and aggregate budgets |
| reason/outcome | Why an action was taken, held, or rejected | Must be observable for audit and troubleshooting |

### Operational Event

Represents a condition requiring dashboard visibility or operator response.

| Field | Meaning | Validation |
|---|---|---|
| event type and severity | Pressure, cap reached, stale input, adjustment failure, or retirement | Severity follows the existing incident policy |
| service/instance scope | Affected local and aggregate scope | Must distinguish per-pod from fleet impact |
| evidence and policy version | Supporting observations and active limits | Required for actionable alerts |
| operator action | Review, approve, disable, or investigate | Must not silently imply that production action occurred |

## Relationships and state transitions

- Each **Pressure Observation** is evaluated against one **Capacity Policy** version.
- An evaluation may create one **Adjustment Decision** and zero or more **Operational Events**.
- A **Connection and Request Lifecycle** is affected by a decision only under these invariants: do not exceed either capacity ceiling; do not close active work; mark a connection or pool generation as draining before retirement.
- Controller modes transition as follows:
  - `DISABLED` → `SHADOW` after approved observation setup.
  - `SHADOW` → `ENFORCING` only after limits are owner-approved and validation has passed.
  - `ENFORCING` → `SAFE_HOLD` when telemetry is stale/invalid, downstream distress appears, or a limit is reached.
  - `SAFE_HOLD` → `ENFORCING` only after valid signals return and approved recovery conditions are satisfied.
  - Any mode → `DISABLED` through an operator-controlled action that leaves in-flight requests intact.
- Pool replacement, if required by the selected client, uses overlapping generations. The old generation transitions to `DRAINING`, accepts no new work, and is retired only after its active request count reaches zero; the overlap is charged to the aggregate budget.
