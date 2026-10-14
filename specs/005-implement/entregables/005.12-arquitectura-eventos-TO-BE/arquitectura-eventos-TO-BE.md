# <span style="color:#0B3C5D">OrionOrders | Arquitectura de eventos TO-BE</span>

> **Propuesta de arquitectura.** Este documento amplía la vista AS-IS con un mecanismo preventivo para la gestión del pool HTTP de `orders-api`. Los componentes nuevos y sus comportamientos se proponen; no se afirma que estén implementados ni habilitados en producción.

**Documento base:** [Arquitectura de eventos AS-IS (Markdown)](../005.11-arquitectura-eventos-AS-IS/arquitectura-eventos-AS-IS.md) · [Arquitectura de eventos AS-IS (PDF)](../005.11-arquitectura-eventos-AS-IS/arquitectura-eventos-AS-IS.pdf)

## <span style="color:#0B3C5D">1. Objetivo y alcance</span>

La propuesta es diseñar un mecanismo en Java para que `orders-api` detecte la saturación de su pool HTTP, aumente la capacidad de forma controlada y gestione las conexiones inactivas sin interrumpir las solicitudes que están en curso.

El diseño propone `Apache HttpClient 5` para la gestión ajustable del pool por instancia, sujeto a confirmar su versión y compatibilidad con la implementación real de `orders-api`. Conserva la llamada síncrona a `customer-profile-service` y mantiene separado el flujo asíncrono de auditoría. La adaptación no garantiza por sí sola la recuperación ni autoriza cambios productivos.

## <span style="color:#0B3C5D">2. Estado actual y evidencia</span>

### Hechos observados

• La evidencia del incidente registra el pool al límite (`leased=20 max=20`) y solicitudes pendientes (`pending=19`), junto con espera local de adquisición (`clientPoolWaitMs`). Esto demuestra presión observada en el pool; no demuestra capacidad libre en el servicio downstream.
• La instantánea de configuración documenta `maxConnections: 20`, `pendingAcquireTimeoutMs: 5000`, `connectTimeoutMs: 1000` y `readTimeoutMs: 2000`. No identifica por sí sola la biblioteca Java ni su semántica de ajuste y cierre.
• El material de arquitectura documenta escalado de 8 a 30 pods. Si el máximo de 20 conexiones aplica por pod, el producto aritmético condicional sería de 160 a 600 conexiones; no es una medida de demanda ni una capacidad aprobada por el downstream.

Fuentes del repositorio: [configuración HTTP](../../../../ORIONORDERS/06_CONFIGURACION/12_http_client_config(2).yaml), [registro de `orders-api`](../../../../ORIONORDERS/02_APLICACION_DESARROLLO/05_orders-api(2).log), [desglose APM](../../../../ORIONORDERS/02_APLICACION_DESARROLLO/13_apm_trace_breakdown(2).csv), [métricas de `customer-profile-service`](../../../../ORIONORDERS/02_APLICACION_DESARROLLO/08_customer_profile_metrics(2).csv) y [arquitectura actual](../../../../ORIONORDERS/00_DOCUMENTACION_GENERAL/01_ARQUITECTURA(3).md).

### Decisiones pendientes

• Confirmar la versión compatible de Apache HttpClient 5 y cómo se integra con `orders-api`; no se encontró código fuente Java ni manifiesto de dependencias que demuestre que ya esté instalado.
• Acordar con Operaciones y el propietario de `customer-profile-service` los límites por instancia y el presupuesto agregado, incluidos cambios de réplicas y cualquier solapamiento temporal de pools.
• Calibrar con pruebas aprobadas los umbrales, ventanas de observación, pasos de ajuste, periodos de estabilización, capacidad mínima y política de solicitudes pendientes. No se fijan valores numéricos no respaldados por evidencia.

## <span style="color:#0B3C5D">3. Diagrama base AS-IS tomado del PDF original</span>

La propuesta parte directamente de [arquitectura-eventos-AS-IS.pdf](../005.11-arquitectura-eventos-AS-IS/arquitectura-eventos-AS-IS.pdf), conservando sus tres zonas y los componentes, nombres y relaciones existentes. Se incluye la [imagen original extraída del PDF](arquitectura-eventos-AS-IS-base.png) y la base visual se conserva en la parte superior del diagrama ampliado siguiente; el pool HTTP original de `orders-api` está resaltado como foco de mejora.

## <span style="color:#0B3C5D">4. Diagrama TO-BE con las mejoras superpuestas</span>

El diagrama ampliado reutiliza la ilustración AS-IS original en su parte superior. En la franja inferior agrega Prometheus/Micrometer, el `Pool Auto-Tuner` Java, las guardas y el ajuste mediante Apache HttpClient 5; no reemplaza ni redibuja los flujos existentes.

![Arquitectura AS-IS con las mejoras TO-BE superpuestas](arquitectura-eventos-TO-BE-diagrama.png)

**Lectura del flujo**

• **Camino crítico existente:** el balanceador dirige el pedido a `orders-api`; el procesamiento del pedido obtiene el perfil mediante la llamada HTTP síncrona a `customer-profile-service`.
• **Auditoría existente:** el pedido y el evento de outbox se persisten localmente; la publicación al broker y el consumo de `audit-service` permanecen asíncronos y separados de la validación síncrona del perfil.
• **Observabilidad propuesta:** Prometheus/Micrometer recolecta ocupación y cola del pool, latencia y errores, con métricas locales separadas de la salud downstream.
• **Control propuesto dentro de `orders-api`:** el `Pool Auto-Tuner` Java evalúa umbrales, duración de la presión y cooldown antes de decidir. La etapa de escalado ilustra 20 → 30 → 40 conexiones por instancia; la etapa de protección valida máximos y límites de salud.
• **Aplicación y dependencia:** Apache HttpClient 5 ajusta el pool por instancia y libera conexiones inactivas; `customer-profile-service` queda protegido por límites de concurrencia y tiempos de espera.
• **Seguridad operativa:** toda ampliación requiere topes aprobados localmente y para el conjunto de réplicas. Con señales defectuosas o una guarda incumplida se detiene el incremento y se alerta.

## <span style="color:#0B3C5D">5. Detección y aumento controlado</span>

### Señales que se deben correlacionar

• **Pool local:** conexiones activas/arrendadas e inactivas, límite efectivo, solicitudes pendientes, tiempo de adquisición y concurrencia. Estas señales identifican espera local por una conexión.
• **Servicio downstream:** latencia de la llamada una vez adquirida la conexión, errores, timeouts, throttling y señales de saturación disponibles, con su frescura y alcance.
• **Calidad de señal:** hora de observación, instancia y política/versionado, instrumentadas con Prometheus/Micrometer. Datos ausentes, vencidos, malformados o contradictorios no habilitan un incremento.

Una espera elevada del pool, por sí sola, no demuestra capacidad libre en `customer-profile-service`. El incremento requiere presión local sostenida y corroborada, señales downstream saludables y presupuesto agregado vigente.

### Política de ajuste

1. Observar primero en modo **SHADOW**: Prometheus/Micrometer recoge ocupación, cola, latencia y errores; el controlador registra qué habría decidido sin cambiar capacidad.
2. El `Pool Auto-Tuner` Java evalúa los umbrales aprobados, la duración de la presión y el cooldown antes de solicitar un ajuste.
3. Si la evidencia sostenida cumple los criterios aprobados, la progresión ilustrativa por instancia es **20 → 30 → 40**. Cada paso queda limitado por máximos configurados, salud downstream y presupuesto agregado aprobado.
4. Aplicar criterios distintos para aumento y reducción, con ventanas de observación y cooldown aprobados para reducir oscilaciones.
5. Registrar evidencia, versión de política, capacidad previa y posterior, resultado y motivo de cada decisión.
6. Habilitar modo **ENFORCING** únicamente tras confirmar la versión de Apache HttpClient 5, aprobar límites con el propietario downstream y superar validaciones controladas.

Apache HttpClient 5 es la biblioteca propuesta para modificar límites del pool por instancia y liberar conexiones inactivas. Antes de implementarlo se debe comprobar en la versión seleccionada el comportamiento con solicitudes concurrentes y cierre; toda reducción preserva solicitudes activas. Si un cambio exige reemplazar el cliente o pool, las solicitudes nuevas se enrutan a la nueva generación, la anterior deja de recibir trabajo y se retira cuando no tenga solicitudes activas. El solapamiento temporal también consume presupuesto agregado.

La progresión **20 → 30 → 40** ilustra el aumento por instancia desde la configuración observada de 20; no es una orden de configuración ni un presupuesto global aprobado. Cada paso exige señales sostenidas, estado de salud aceptable y margen dentro de los máximos local y agregado. Si una guarda falla, el `Pool Auto-Tuner` no escala y emite/expone la condición operativa.

## <span style="color:#0B3C5D">6. Reducción segura, topes y modo de protección</span>

### Retiro y continuidad

• Solo se considera elegible una conexión inactiva que cumpla el periodo y la capacidad mínima aprobados.
• Una conexión con solicitudes activas no se cierra para reducir capacidad. Si se usa drenado por generaciones, la generación anterior deja de aceptar trabajo nuevo y se retira únicamente al completar las solicitudes que ya atiende.
• No se asigna un plazo de drenado que fuerce cierres y, a la vez, afirme continuidad garantizada. La semántica debe verificarse para el cliente HTTP seleccionado.

### Cuando se alcanza un límite o falla la telemetría

• En el tope local o agregado se detienen los incrementos, se mantienen las solicitudes en curso, se aplica la política aprobada de trabajo pendiente y se genera una alerta accionable con señales, alcance y límite alcanzado.
• Ante telemetría ausente, vencida o contradictoria, el controlador pasa a **SAFE_HOLD**: pausa incrementos automáticos, conserva una configuración conocida y acotada, y notifica a Operaciones.
• La desactivación o reversión es operada y auditable. Detiene nuevas decisiones adaptativas sin cerrar solicitudes activas; si hay generaciones, el cambio conserva la ruta estable hasta completar el drenado seguro.

## <span style="color:#0B3C5D">7. Observabilidad y validación</span>

Registrar y visualizar, como mínimo:

• Uso y límite del pool, solicitudes pendientes y tiempo de adquisición local.
• Latencia, errores, timeouts, throttling y salud downstream disponibles, diferenciados de la espera local.
• Decisiones de incremento, reducción, espera o rechazo; versión de política, evidencia, límites y resultado.
• Conexiones inactivas retiradas, generaciones en drenado, solapamiento contabilizado, límites alcanzados y fallos de ajuste.
• Modo del controlador (**DISABLED**, **SHADOW**, **ENFORCING** o **SAFE_HOLD**), frescura de señales y alertas operativas.

La validación se realiza primero en un entorno no productivo aprobado y luego por etapas con autorización, monitoreo y reversión preparada. Antes de ejecutar las pruebas, los responsables deben acordar y registrar el perfil de carga, el número de ejecuciones por escenario, las ventanas de medición y el método para atribuir interrupciones al controlador. No ejecutar carga o inyección de fallos en producción sin aprobación explícita.

| Escenario | Resultado requerido |
|---|---|
| Carga creciente sostenida | Señal preventiva antes del máximo aprobado; aumento gradual sin rebasar topes locales ni agregado. |
| Pico transitorio | Sin escalado repetitivo; registrar la decisión transitoria. |
| Espera local con downstream degradado | Congelar o limitar incrementos; no inferir capacidad downstream por la espera local. |
| Menor demanda con trabajo activo | Retirar solo conexiones elegibles inactivas; retirar al menos el 95% dentro del periodo de inactividad configurado; completar solicitudes activas. |
| Tope local/agregado y cambio de réplicas | No exceder presupuesto; acotar trabajo pendiente y emitir alerta accionable. |
| Telemetría ausente o contradictoria | Entrar en SAFE_HOLD, pausar incrementos y comunicar la condición. |
| Desactivación o reversión durante solicitudes | Detener decisiones nuevas y permitir completar trabajo en curso. |
| Cambio de generación del pool, si aplica | Enrutar llamadas nuevas a la generación nueva, drenar la anterior y contabilizar el solapamiento. |

En el 100% de las pruebas controladas, no debe haber interrupciones atribuibles al controlador ni excedentes de los topes aprobados, y cada decisión debe ser trazable a señal fresca y versión de política. Al menos el 95% de las conexiones inactivas elegibles debe retirarse dentro del periodo configurado. Durante las ventanas aprobadas también se revisan los SLO operativos vigentes: disponibilidad mensual de **99.95%**, p95 **< 1.2 s** y p99 **< 2.5 s**. Un aumento de conexiones, por sí solo, no demuestra recuperación ni cumplimiento de SLO.

## <span style="color:#0B3C5D">8. Aprobaciones antes de habilitar cambios</span>

• **Aplicación:** identificar cliente HTTP Java, versión, límites modificables, semántica de retiro inactivo, drenado y cierre.
• **Operaciones:** validar métricas, ventanas, umbrales, alertas, modo de despliegue y procedimiento de reversión.
• **Propietario de `customer-profile-service`:** aprobar presupuesto agregado, política ante saturación y comportamiento ante cambios de réplicas.
• **Responsables de validación:** acordar y registrar el perfil de carga, el número de ejecuciones por escenario, las ventanas de medición y el método de atribución de interrupciones; revisar evidencia de pruebas controladas y autorizar explícitamente cualquier etapa de producción.

Hasta cerrar estas decisiones, la propuesta permanece en modo documental/shadow; no se habilitan aumentos automáticos ni se presume un máximo downstream seguro.
