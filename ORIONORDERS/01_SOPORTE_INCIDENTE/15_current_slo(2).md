# Objetivos de nivel de servicio (SLO) vigentes

Servicio: OrionOrders
Disponibilidad objetivo: 99.95% mensual
Objetivos de latencia:
- p95 < 1.2 s
- p99 < 2.5 s

Alertas actuales:
- errores HTTP 5xx > 5% durante 5 min
- uso de CPU > 80% durante 10 min
- conexiones a la base de datos > 85%
- reinicios de instancias de servicio > 5 en 10 min

No existen alertas específicas para:
- espera por adquisición de una conexión del pool HTTP hacia customer-profile;
- saturación de conexiones del cliente HTTP;
- tiempo de espera interno antes de enviar la solicitud al servicio dependiente.
