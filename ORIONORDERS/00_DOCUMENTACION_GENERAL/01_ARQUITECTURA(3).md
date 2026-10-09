# OrionOrders: resumen de arquitectura

Internet
 -> Balanceador de carga
   -> Kubernetes / orders-api (12 instancias de servicio, o pods, normalmente)
      -> PostgreSQL orders-primary
      -> customer-profile-service (HTTPS)
      -> audit-service (asíncrono)

Escalado automático:
- HPA mínimo 8, máximo 30 instancias de servicio (pods).
- objetivo de CPU: 65%.

Comprobaciones de salud:
- disponibilidad: /actuator/health/readiness
- actividad: /actuator/health/liveness

El endpoint de actividad atraviesa el mismo servidor HTTP y grupo de hilos que el tráfico de negocio.

customer-profile-service:
- servicio interno externo al componente orders-api.
- latencia p95 habitual: 80-150 ms.
