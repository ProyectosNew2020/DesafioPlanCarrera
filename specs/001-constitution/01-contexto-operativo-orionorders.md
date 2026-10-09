# Contexto operativo de OrionOrders

## 1. Objetivo del caso
El servicio OrionOrders es un sistema crítico para clientes empresariales y se encuentra bajo un incidente P1 declarado tras una degradación de disponibilidad y latencia. La evidencia disponible indica que el problema se desencadenó cerca de una publicación reciente y requiere separar hechos, hipótesis y conclusiones antes de ejecutar acciones operativas.

## 2. Arquitectura y componentes relevantes
Según la documentación del proyecto, la topología es la siguiente:

- Internet -> balanceador de carga -> Kubernetes / orders-api (12 instancias de servicio, o pods, normalmente)
- orders-api accede a PostgreSQL orders-primary
- orders-api depende de customer-profile-service mediante HTTP
- hay un componente de auditoría asíncrono (audit-service)
- el escalado automático está definido con HPA mínimo 8 y máximo 30 instancias de servicio (pods), con objetivo de CPU del 65%
- las comprobaciones de salud son `/actuator/health/readiness` y `/actuator/health/liveness`

Se destaca que el endpoint de actividad atraviesa el mismo servidor HTTP y el mismo grupo de hilos que el tráfico de negocio, por lo que una degradación de carga puede afectar la salud de las instancias y acelerar sus reinicios.

## 3. Objetivos de nivel de servicio (SLO) y alertas vigentes
El servicio tiene los siguientes objetivos operativos:

- Disponibilidad objetivo: 99.95% mensual
- Latencia p95 objetivo: menos de 1.2 s
- Latencia p99 objetivo: menos de 2.5 s

Las alertas relevantes actualmente son:

- HTTP 5xx superior a 5% durante 5 minutos
- CPU superior a 80% durante 10 minutos
- conexiones a la base de datos superiores al 85%
- reinicios de instancias mayores a 5 en 10 minutos

No existe una alerta específica para:

- espera para adquirir una conexión del pool HTTP hacia customer-profile
- saturación de conexiones del cliente HTTP
- tiempo de espera interno antes de enviar la solicitud al servicio dependiente

## 4. Cronología del incidente
La secuencia documental del caso es la siguiente:

- 09:30: se completa la publicación de la versión 4.18.0
- 09:42: primera alerta de latencia
- 09:45: primeras instancias reiniciadas
- 09:47: alerta crítica de conexiones a base de datos
- 09:49: se declara P1
- 09:50: el equipo operativo observa reinicios por fallos en la comprobación de actividad
- 09:54: el administrador de bases de datos apunta a `db-reporting-01` y revisa el origen de datos de orders-api
- 10:01: en los registros aparece un aumento de `clientPoolWaitMs` con `leased=20 max=20`

Impacto reportado:

- 4 clientes empresariales con operaciones intermitentes
- p95 por encima de 8 s
- HTTP 502/503 entre 12% y 35%
- pedidos nuevos tardan o fallan
- sin evidencia inicial de pérdida de datos

## 5. Cambios de la versión 4.18.0
La versión 4.18.0 incluye:

- nuevo campo opcional `salesChannel`
- refactorización del cliente HTTP para customer-profile
- sin cambios declarados en SQL

El cambio de configuración relevante fue el siguiente:

- Antes: `CUSTOMER_PROFILE_MAX_CONNECTIONS=200`
- Después: `CUSTOMER_PROFILE_MAX_CONNECTIONS=20`
- `CUSTOMER_PROFILE_CONNECT_TIMEOUT_MS=1000`
- `CUSTOMER_PROFILE_READ_TIMEOUT_MS=2000`

La nota original de Helm indica: "normalize http client settings across services".

La versión 4.17.6 sigue disponible y también es posible corregir únicamente los valores del ConfigMap o de Helm.

## 6. Evidencia operativa y observaciones clave
La evidencia disponible sugiere varios puntos importantes:

1. La alerta de DB no parece ser la causa principal del problema.
   - La base de datos reporta 118 conexiones sobre 300 en `orders-primary-01`.
   - La CPU de la base de datos está en 44%, sin bloqueos relevantes ni esperas por bloqueo.
   - La base de datos no presenta evidencia fuerte de saturación como causa raíz del incidente.

2. El problema de conexión HTTP hacia customer-profile parece más consistente con el síntoma observado tras la publicación.
   - El cliente HTTP fue reestructurado en la versión 4.18.0.
   - El máximo del pool quedó fijado en 20 conexiones.
   - Los registros muestran un aumento de `clientPoolWaitMs`, que es una señal de espera en la cola del pool.

3. Los pods se reinician por fallos en la comprobación de actividad y no necesariamente por una falla directa de la base de datos.
   - El endpoint de actividad comparte el mismo grupo de hilos que el tráfico de producción.
   - Cuando el servicio queda saturado por espera de salida, la salud de las instancias puede degradarse.

4. El incidente implicó varios dominios, pero la evidencia no confirma pérdida de datos ni corrupción en base de datos.

## 7. Restricciones operativas y seguridad
El contexto de seguridad del proyecto establece lo siguiente:

- no se permite desactivar la autenticación para pruebas
- no se permite reiniciar la base de datos
- no se permite ejecutar cambios directos en los datos
- cualquier reversión debe preservar las órdenes ya confirmadas
- los registros entregados no contienen credenciales reales

Esto implica que cualquier corrección debe ser reversible, controlada y compatible con la integridad transaccional.

## 8. Síntesis para el trabajo posterior
La evidencia disponible indica que la versión 4.18.0 introdujo un cambio operativo de configuración en el cliente HTTP hacia customer-profile, reduciendo drásticamente el máximo de conexiones del pool. Ese ajuste puede causar espera en cola, latencia acumulada, aumento de errores 5xx y, eventualmente, reinicios de pods por fallos en la comprobación de actividad. La base de datos, aunque generó alertas, no parece ser la causa directa según la evidencia técnica entregada.

Este documento sirve como base de análisis para continuar el diagnóstico, definir planes de corrección y preparar un procedimiento operativo de recuperación para OrionOrders.
