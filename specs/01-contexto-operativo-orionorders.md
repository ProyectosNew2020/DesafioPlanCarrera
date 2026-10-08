# Contexto operativo de OrionOrders

## 1. Objetivo del caso
El servicio OrionOrders es un sistema crítico para clientes empresariales y se encuentra bajo un incidente P1 declarado tras una degradación de disponibilidad y latencia. La evidencia disponible indica que el problema se desencadenó cerca de una release reciente y requiere separar hechos, hipótesis y conclusiones antes de ejecutar acciones operativas.

## 2. Arquitectura y componentes relevantes
Según la documentación del proyecto, la topología es la siguiente:

- Internet -> Load Balancer -> Kubernetes / orders-api (12 pods normativos)
- orders-api accede a PostgreSQL orders-primary
- orders-api depende de customer-profile-service mediante HTTP
- hay un componente de auditoría asíncrono (audit-service)
- el autoscaling está definido con HPA mínimo 8 y máximo 30 pods, con objetivo de CPU del 65%
- los health probes son `/actuator/health/readiness` y `/actuator/health/liveness`

Se destaca que el endpoint de liveness atraviesa el mismo servidor HTTP y el mismo thread pool que el tráfico de negocio, por lo que una degradación de carga puede afectar la salud del pod y acelerar los reinicios.

## 3. SLO y alertas vigentes
El servicio tiene los siguientes objetivos operativos:

- Disponibilidad objetivo: 99.95% mensual
- Latencia p95 objetivo: menos de 1.2 s
- Latencia p99 objetivo: menos de 2.5 s

Las alertas relevantes actualmente son:

- HTTP 5xx superior a 5% durante 5 minutos
- CPU superior a 80% durante 10 minutos
- conexiones a la base de datos superiores al 85%
- reinicios de pod mayores a 5 en 10 minutos

No existe una alerta específica para:

- espera de adquisición del pool HTTP hacia customer-profile
- saturación de conexiones del cliente HTTP
- tiempo de espera interno antes de enviar la solicitud downstream

## 4. Cronología del incidente
La secuencia documental del caso es la siguiente:

- 09:30: se completa la release 4.18.0
- 09:42: primera alerta de latencia
- 09:45: primeros pods reiniciados
- 09:47: alerta crítica de conexiones a base de datos
- 09:49: se declara P1
- 09:50: el equipo operativo observa reinicios por liveness
- 09:54: el DBA apunta a `db-reporting-01` y revisa el datasource de orders-api
- 10:01: en los logs aparece `clientPoolWaitMs` creciente con `leased=20 max=20`

Impacto reportado:

- 4 clientes empresariales con operaciones intermitentes
- p95 por encima de 8 s
- HTTP 502/503 entre 12% y 35%
- pedidos nuevos tardan o fallan
- sin evidencia inicial de pérdida de datos

## 5. Cambios de la release 4.18.0
La release 4.18.0 incluye:

- nuevo campo opcional `salesChannel`
- refactor del cliente HTTP para customer-profile
- sin cambios declarados en SQL

El cambio de configuración relevante fue el siguiente:

- Antes: `CUSTOMER_PROFILE_MAX_CONNECTIONS=200`
- Después: `CUSTOMER_PROFILE_MAX_CONNECTIONS=20`
- `CUSTOMER_PROFILE_CONNECT_TIMEOUT_MS=1000`
- `CUSTOMER_PROFILE_READ_TIMEOUT_MS=2000`

La nota de Helm indica: "normalize http client settings across services".

La versión 4.17.6 sigue disponible y también es posible corregir únicamente los valores del ConfigMap o Helm.

## 6. Evidencia operativa y observaciones clave
La evidencia disponible sugiere varios puntos importantes:

1. La alerta de DB no parece ser la causa principal del problema.
   - La base de datos reporta 118 conexiones sobre 300 en `orders-primary-01`.
   - La CPU de la base de datos está en 44%, sin bloqueos relevantes ni lock waiters.
   - La base de datos no presenta evidencia fuerte de saturación como causa raíz del incidente.

2. El problema de conexión HTTP hacia customer-profile parece más consistente con el síntoma observando la release.
   - El cliente HTTP fue refactorizado en la release 4.18.0.
   - El max pool quedó fijado en 20 conexiones.
   - Los logs muestran crecimiento de `clientPoolWaitMs`, que es una señal de cola de espera del pool.

3. Los pods se reinician por liveness y no necesariamente por una falla directa de la base de datos.
   - El endpoint de liveness comparte el mismo thread pool que el tráfico productivo.
   - Cuando el servicio queda saturado por espera de salida, la salud del pod puede degradarse.

4. El incidente implicó varios dominios, pero la evidencia no confirma pérdida de datos ni corrupción en base de datos.

## 7. Restricciones operativas y seguridad
El contexto de seguridad del proyecto establece lo siguiente:

- no se permite desactivar la autenticación para pruebas
- no se permite reiniciar la base de datos
- no se permite ejecutar cambios directos de datos
- cualquier rollback debe preservar órdenes ya confirmadas
- los logs entregados no contienen credenciales reales

Esto implica que cualquier corrección debe ser reversible, controlada y compatible con la integridad transaccional.

## 8. Síntesis para trabajo posterior
La evidencia disponible indica que la release 4.18.0 introdujo un cambio operacional de configuración en el cliente HTTP a customer-profile, reduciendo drásticamente el máximo de conexiones del pool. Ese ajuste puede causar cola de espera, latencia acumulada, aumento de 5xx y eventualmente reinicios de pod por liveness. La base de datos, aunque generó alertas, no parece ser la causa directa según la evidencia técnica entregada.

Este documento sirve como base de análisis para continuidad del diagnóstico, la definición de planes de remediación y la preparación de un runbook operativo para OrionOrders.
