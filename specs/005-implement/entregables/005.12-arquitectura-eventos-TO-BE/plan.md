# Implementation Plan: TO-BE Adaptive HTTP Pool Architecture

**Branch**: `005.12-arquitectura-eventos-TO-BE` | **Date**: 2026-10-11 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification for a proposed adaptive HTTP connection-pool management flow in `orders-api`.

## Summary

Produce a TO-BE architecture deliverable that extends the documented AS-IS request and audit flows with a preventive, bounded pool-management control loop. It will distinguish local pool wait from downstream latency, increase capacity only within approved per-instance and global limits, and retire only idle connections while in-flight requests complete. The proposal remains library-neutral because the supplied workspace has no application source or dependency manifest. This phase documents and validates the proposal; it does not implement Java code or authorize production changes.

## Technical Context

**Language/Version**: Java is the required execution context for the proposed mechanism; the Java version is not present in the supplied artifacts.

**Primary Dependencies**: Propose Apache HttpClient 5 for per-instance pool adjustment and idle-connection release. No application source, dependency manifest, or installed HTTP-client library is present, so its version and integration must be verified before implementation. Prometheus/Micrometer is the proposed metrics path and likewise must be matched to the existing observability stack. If the selected client requires pool replacement, use a generation-and-drain lifecycle with aggregate overlap included in the budget.

**Storage**: N/A for the architecture deliverable. Runtime metrics and decision records are assumed to use existing OrionOrders observability facilities; their product and retention are unspecified.

**Testing**: Documentation review against the requirements checklist and scenario-based validation guide. A future implementation requires controlled tests for rising/falling demand, downstream distress, in-flight requests, both capacity caps, stale signals, replica changes, and disable/rollback.

**Target Platform**: `orders-api` service running in Kubernetes, based on the supplied deployment, health-probe, and incident artifacts.

**Project Type**: Documentation-only architecture proposal; no application source code is included in the current project workspace.

**Performance Goals**: Detect sustained pressure before the approved maximum is reached; preserve current service targets of monthly availability 99.95%, p95 < 1.2 s, and p99 < 2.5 s during approved validation windows.

**Constraints**: Do not interrupt in-flight requests; never exceed approved per-instance or aggregate downstream capacity; do not enable autonomous increases before global capacity is agreed; bound pending work; avoid adjustment oscillation; preserve the synchronous `orders-api` to `customer-profile-service` request path and the separate asynchronous audit flow; all production changes require authorization and rollback planning.

**Scale/Scope**: One logical HTTP pool per applicable `orders-api` instance, with the aggregate connection budget coordinated across all instances against shared `customer-profile-service`. The supplied context documents an HPA range of 8–30 pods. Exact capacity, concurrency profile, and downstream limits must be established by owners before enabling adaptation.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle / gate | Assessment | Evidence in this plan |
|---|---|---|
| I. Evidence before action | PASS | Observe pool wait/pending separately from network latency and downstream errors; never infer downstream headroom from one local signal. |
| II. Shared ownership and disciplined classification | PASS | Global limits require agreement by `orders-api` operations and the `customer-profile-service` owner; facts and proposed behavior remain distinguished. |
| III. Safe change, rollback, and recovery readiness | PASS | Capacity changes are bounded; idle retirement excludes active work; disable and rollback behavior must be documented and tested before production. |
| IV. Quality controls and observability | PASS | Specify decision telemetry, alerts, controlled validation, and SLO checks. |
| V. Documentation and continuous improvement | PASS | Produce architecture, rationale, validation scenarios, and unresolved owner decisions as durable documentation. |
| Operational constraints | PASS | No direct production changes, data changes, authentication bypass, or unapproved capacity increase are in scope. |

**Pre-research gate result**: PASS. No constitution violation requires an exception.

## Project Structure

### Documentation (this feature)

```text
specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── checklists/
│   └── requirements.md
├── contracts/                         # Omitted: no new external API is in scope
├── arquitectura-eventos-TO-BE.md      # Architecture deliverable for the next phase
└── arquitectura-eventos-TO-BE.pdf     # Rendered deliverable, if PDF export is available
```

### Source Code (repository root)

```text
No application source tree or Java dependency manifest is present.
ORIONORDERS/ contains operational evidence, configuration snapshots, and release artifacts only.
```

**Structure Decision**: Keep planning and design documents alongside the requested `005.12-arquitectura-eventos-TO-BE` deliverable. Do not create Java source code or a public interface during this documentation-only phase. The architecture document should link to the existing `005.11-arquitectura-eventos-AS-IS` deliverable and label every new component as proposed.

## Phase 0: Research Decisions

1. Propose Apache HttpClient 5 as the Java client for per-instance pool adjustment and idle-connection release; verify the compatible version, dependency availability, and active-request behavior before implementation.
2. Propose Prometheus/Micrometer to collect pool occupancy, pending queue, latency, and errors; verify metric names, freshness, and existing registry/exporter integration.
3. Prefer in-place limit updates where supported. If not supported, use a bounded client/pool generation transition: route new work to the new generation, drain old in-flight work, and include temporary overlap in the aggregate capacity budget.
4. Model the `Pool Auto-Tuner` as a bounded, damped controller with corroborated signals, thresholds, pressure duration, cooldown, and health/max guardrails. Illustrate `20 → 30 → 40` as candidate per-instance steps only; no thresholds, health maxima, or global budget are approved by this plan.
5. Treat stale or contradictory telemetry as insufficient evidence: pause automatic increases, retain a known bounded state, and alert.

## Phase 1: Design Artifacts

1. Model pressure signals, capacity policy, connection/request lifecycle, adjustment decisions, and operational events.
2. Define internal behavior and lifecycle contracts in the design artifacts; no new external API or contract directory is warranted.
3. Provide a scenario-driven validation guide for normal demand, pressure, downstream distress, cap reached, idle retirement, active requests, stale telemetry, replica changes, and disable/rollback.
4. Re-evaluate the design against the constitution and the specification's measurable success criteria.

## Complexity Tracking

No constitution violations or additional implementation projects are proposed.

## Post-Design Constitution Check

| Principle / gate | Result | Design evidence |
|---|---|---|
| I. Evidence before action | PASS | Separate local acquisition/queue signals from downstream latency, errors, and headroom. |
| II. Shared ownership | PASS | Per-instance and aggregate limits remain explicit owner-approved policy values. |
| III. Safe change and rollback | PASS | Idle-only retirement and generation draining preserve in-flight work; bounded overlap and disable path are specified. |
| IV. Quality and observability | PASS | Pool, downstream, decision, limit, stale-signal, and idle-retirement telemetry plus staged scenario validation are defined. |
| V. Documentation and learning | PASS | Research, data model, validation guide, and future TO-BE deliverable preserve rationale and assumptions. |

**Post-design gate result**: PASS. No constitution exception is required.
