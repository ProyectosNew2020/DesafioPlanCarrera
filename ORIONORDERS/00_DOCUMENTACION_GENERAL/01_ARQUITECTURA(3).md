# OrionOrders - arquitectura resumida

Internet
 -> Load Balancer
   -> Kubernetes / orders-api (12 pods normalmente)
      -> PostgreSQL orders-primary
      -> customer-profile-service (HTTPS)
      -> audit-service (async)

Autoscaling:
- HPA mínimo 8, máximo 30 pods.
- objetivo CPU 65%.

Health probes:
- readiness: /actuator/health/readiness
- liveness: /actuator/health/liveness

El endpoint de liveness atraviesa el mismo servidor HTTP/thread pool que el tráfico de negocio.

customer-profile-service:
- servicio externo interno.
- latencia habitual p95: 80-150 ms.
