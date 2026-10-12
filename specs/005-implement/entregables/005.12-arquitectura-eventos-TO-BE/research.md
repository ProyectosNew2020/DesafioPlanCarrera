# Research: Safe Adaptive HTTP Pool Management

**Feature**: [TO-BE Adaptive HTTP Pool Architecture](./spec.md)  
**Research date**: 2026-10-11

## Decision 1: Propose Apache HttpClient 5 and verify the actual integration before implementation

**Decision**: Use Apache HttpClient 5 as the proposed Java HTTP client for per-instance pool adjustment and idle-connection release. Before implementation, verify the compatible version, the actual `orders-api` dependency/integration, runtime limit-update semantics, and that reducing/closing idle connections does not interrupt active requests. Prefer in-place limit updates when supported. If pool replacement is required, route new calls to the new generation, allow old in-flight work to drain, and retire it only after it is idle. Count overlapping generations against the global capacity budget.

**Rationale**: The requested proposal names Apache HttpClient 5. The available workspace contains configuration values (`maxConnections: 20`, pending-acquire timeout 5000 ms, connect timeout 1000 ms, read timeout 2000 ms), but no Java source or dependency manifest confirming the actual library/version. Apache HttpClient 5 is therefore the proposed target, not a verified existing dependency. Its lifecycle behavior must be checked in the version selected for implementation.

**Alternatives considered**:
- Treat Apache HttpClient 5 as already installed: rejected because no source or dependency evidence confirms that fact.
- Replace the HTTP client on every adjustment: rejected as the default because replacement introduces warm-up, temporary overlap, pending-acquisition, and drain semantics that must be validated.
- If compatible live pool resizing cannot be verified, hold in a safe bounded mode and revisit the client strategy before enabling enforcement; do not silently claim the requested adjustment is implemented.

**Sources**:
- Project snapshot: [`12_http_client_config(2).yaml`](../../../../ORIONORDERS/06_CONFIGURACION/12_http_client_config(2).yaml)
- [Apache HttpClient 5 `PoolingHttpClientConnectionManager`](https://github.com/apache/httpcomponents-client/blob/master/httpclient5/src/main/java/org/apache/hc/client5/http/impl/io/PoolingHttpClientConnectionManager.java)
- [JDK `HttpClient` API](https://docs.oracle.com/en/java/javase/21/docs/api/java.net.http/java/net/http/HttpClient.html)
- [Reactor Netty `ConnectionProvider`](https://projectreactor.io/docs/netty/release/api/reactor/netty/resources/ConnectionProvider.html)
- [OkHttp `ConnectionPool`](https://square.github.io/okhttp/5.x/okhttp/okhttp3/-connection-pool/)

## Decision 2: Adapt against corroborated local and downstream signals

**Decision**: Propose Prometheus/Micrometer instrumentation to track local pool occupancy, pending queue/acquisition wait, active/idle/leased connections, effective limit, and request concurrency separately from downstream request latency, errors, timeouts, throttling, and (where available) saturation. The Java `Pool Auto-Tuner` evaluates configured thresholds, pressure duration, and cooldown. Increase only when sustained local pressure is corroborated, health guardrails pass, and downstream capacity is approved. Freeze or back off increases when downstream distress appears.

**Rationale**: The incident records `leased=20 max=20 pending=19` and elevated `clientPoolWaitMs`, while sampled downstream call durations are substantially lower. That supports a local wait hypothesis but does not prove spare aggregate capacity at the shared dependency.

**Alternatives considered**:
- Increase on a single pool-wait spike: rejected because a transient observation can trigger unnecessary expansion and overload the dependency.
- Use downstream latency alone: rejected because it cannot distinguish time waiting locally for a connection from time spent in the downstream service.
- Use static `20 → 30 → 40` values as production defaults: rejected because these are illustrative per-instance steps only; max/health guardrails and the aggregate downstream budget still require approval.

**Sources**:
- Project evidence: [`05_orders-api(2).log`](../../../../ORIONORDERS/02_APLICACION_DESARROLLO/05_orders-api(2).log), [`13_apm_trace_breakdown(2).csv`](../../../../ORIONORDERS/02_APLICACION_DESARROLLO/13_apm_trace_breakdown(2).csv), [`08_customer_profile_metrics(2).csv`](../../../../ORIONORDERS/02_APLICACION_DESARROLLO/08_customer_profile_metrics(2).csv)
- [Netflix concurrency-limits design overview](https://github.com/Netflix/concurrency-limits)
- [Envoy adaptive concurrency filter](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/adaptive_concurrency_filter)

## Decision 3: Bound and damp the adaptation loop

**Decision**: Use sustained observation windows, different criteria for increasing and decreasing, bounded adjustment steps/frequency, a recovery hold-down, and jitter for periodic per-instance actions. The TO-BE must leave threshold values, steps, window lengths, and cooldowns for owner-approved load-test calibration.

**Rationale**: Damping reduces oscillation and synchronized changes across replicas. There are no project measurements that justify concrete controller constants.

**Alternatives considered**:
- Use one threshold for both expansion and reduction: rejected because noisy signals can cause repeated reversals.
- Put arbitrary default numeric thresholds in the architecture: rejected because the configuration and capacity evidence do not establish safe values.

**Sources**:
- [Envoy adaptive concurrency filter](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/adaptive_concurrency_filter)
- [Google SRE: Addressing Cascading Failures](https://sre.google/sre-book/addressing-cascading-failures/)

## Decision 4: Protect the dependency with both local and aggregate capacity ceilings

**Decision**: Define a hard per-instance ceiling and a coordinated aggregate budget owned with the `customer-profile-service` team. Include HPA scaling and temporary old/new pool overlap in aggregate calculations. Do not enable automatic increases while the aggregate budget or its enforcement/failure behavior is unapproved.

**Rationale**: OrionOrders documents an HPA range of 8–30 pods and a configured pool maximum of 20. If the setting is per pod, the theoretical aggregate configured maximum ranges from 160 to 600 connections; this is conditional arithmetic, not measured usage or evidence that the downstream can support that load.

**Alternatives considered**:
- Rely on per-instance limits only: rejected because aggregate connections rise with replica count.
- Assume the old configured value (200) is safe to restore or expand: rejected because the downstream owner's capacity limit is absent.

**Sources**:
- Project evidence: [`01_ARQUITECTURA(3).md`](../../../../ORIONORDERS/00_DOCUMENTACION_GENERAL/01_ARQUITECTURA(3).md), [`03_RELEASE_4.18.0(2).md`](../../../../ORIONORDERS/05_DEVOPS_CICD_RELEASE/03_RELEASE_4.18.0(2).md)
- [Envoy upstream circuit breaking](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/circuit_breaking)
- [Envoy upstream connection pooling](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/connection_pooling)

## Decision 5: Drain before retirement; bound pending work at the ceiling

**Decision**: Stop assigning new work to a connection or old pool generation before retirement, let active requests finish, and close only when idle. At a ceiling, stop increasing, keep active requests intact, bound pending work according to an approved policy, and raise an actionable alert. If a drain deadline is ever allowed to force closure, document it as a behavior that can interrupt remaining requests and do not claim that it satisfies the no-interruption criterion.

**Rationale**: Idle cleanup and capacity adjustment are distinct operations. Protocol/client shutdown semantics vary; a generic pool replacement cannot be assumed to drain active work forever without consequences. HTTP/2 `GOAWAY` offers a protocol-level way to stop new streams while existing streams finish, but it applies only when the actual protocol and client behavior support it.

**Alternatives considered**:
- Close all old connections immediately on resize: rejected because active requests could be interrupted.
- Leave pending work unbounded at the cap: rejected because it perpetuates queue growth and hides overload.

**Sources**:
- [RFC 9113 §6.8: GOAWAY](https://www.rfc-editor.org/rfc/rfc9113.html#section-6.8)
- [Google SRE: Handling Overload](https://sre.google/sre-book/handling-overload/)
- [Envoy upstream circuit breaking](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/circuit_breaking)

## Decision 6: Missing or stale input pauses automatic increases

**Decision**: Treat absent, stale, malformed, or contradictory telemetry as insufficient evidence. Pause automatic increases, retain a known bounded configuration, emit an operational signal, and expose an operator-controlled disable/rollback path that does not close active requests.

**Rationale**: The project requires evidence before action and a safe change/rollback. Invalid observations cannot be interpreted as available capacity.

**Sources**:
- [Google SRE: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/)
- Feature requirements FR-003, FR-006, and FR-009.

## Decision 7: Validate in shadow/staged mode before enforcement

**Decision**: First compare the proposed controller's decisions against observed load without changing capacity (shadow observation). Then validate approved increase/decrease behavior under controlled load and fault scenarios; stage any production rollout with approvals, monitoring, and rollback readiness.

**Rationale**: This provides evidence to tune thresholds and estimate downstream impact before enabling automated changes. Production validation remains subject to explicit authorization.

**Sources**:
- [Google SRE: Handling Overload](https://sre.google/sre-book/handling-overload/)
- Feature requirements FR-012 and SC-002–SC-006.
