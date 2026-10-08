# SLO vigente

Servicio: OrionOrders
Disponibilidad objetivo: 99.95% mensual
Latencia objetivo:
- p95 < 1.2 s
- p99 < 2.5 s

Alertas actuales:
- HTTP 5xx > 5% durante 5 min
- CPU > 80% durante 10 min
- DB connections > 85%
- pod restarts > 5 en 10 min

No existe alerta específica para:
- pending acquisition del pool HTTP a customer-profile;
- saturación de conexiones del cliente HTTP;
- tiempo de espera interno antes de enviar la solicitud downstream.
