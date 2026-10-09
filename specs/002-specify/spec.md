# Especificación: Diagnóstico y recuperación del incidente OrionOrders

**Rama de funcionalidad**: `No aplica (el directorio del proyecto no es un repositorio Git)`

**Fecha de creación**: 2026-10-07

**Estado**: Borrador

**Solicitud original**: "Crear una especificación en `specs/` tomando en cuenta la carpeta `ORIENORDER`."

## Escenarios de usuario y pruebas *(obligatorio)*

### Historia de usuario 1 - Diagnosticar el incidente con evidencia (Prioridad: P1)

Como analista de soporte que lidera la respuesta al P1-8841, necesito correlacionar las señales de aplicación, infraestructura, dependencias, base de datos y publicación para determinar qué explica la degradación, sin confundir una alerta coincidente con la causa raíz.

**Motivo de la prioridad**: La disponibilidad y las operaciones de cuatro clientes empresariales están afectadas. Un diagnóstico basado en evidencia es necesario antes de elegir una mitigación.

**Prueba independiente**: Revisar el paquete de diagnóstico generado y comprobar que contiene una cronología, impacto, fuentes citadas y una clasificación explícita de hechos, hipótesis y conclusiones.

**Escenarios de aceptación**:

1. **Dado** que están disponibles los artefactos de P1-8841, **Cuando** el analista construye la cronología, **Entonces** cada evento relevante tiene hora, fuente y tipo de evidencia, y se distinguen el primer síntoma de las 09:42 y la declaración P1 de las 09:49.
2. **Dado** que la alerta de base de datos identifica `db-reporting-01`, **Cuando** el analista evalúa su relación con `orders-api`, **Entonces** contrasta el host con el origen de datos y con las mediciones de `orders-primary-01` antes de atribuirle causalidad.
3. **Dado** que los registros muestran espera del pool del cliente HTTP y los tiempos de red de `customer-profile` permanecen bajos, **Cuando** el analista formula una causa probable, **Entonces** registra tanto esa evidencia como las validaciones pendientes antes de tratar la hipótesis como causa confirmada.

### Historia de usuario 2 - Recuperar el servicio de forma segura (Prioridad: P1)

Como responsable de operaciones, necesito comparar mitigaciones reversibles y comprobar su efecto para restaurar el procesamiento de pedidos sin perder órdenes confirmadas ni ampliar innecesariamente el impacto.

**Motivo de la prioridad**: La recuperación de pedidos y la protección de datos son las necesidades inmediatas de los clientes afectados.

**Prueba independiente**: Evaluar el plan de mitigación y una simulación o ejecución autorizada frente a criterios de impacto, reversibilidad, validación y conservación de órdenes confirmadas.

**Escenarios de aceptación**:

1. **Dado** que existe una hipótesis sustentada sobre el cambio reciente de configuración del cliente del servicio dependiente, **Cuando** se propone una mitigación, **Entonces** se documentan el alcance, la persona responsable, la evidencia, los criterios de éxito y los criterios de reversión antes de aplicarla.
2. **Dado** que hay una versión anterior disponible y también una opción para corregir la configuración, **Cuando** se comparan las alternativas, **Entonces** se incluyen los riesgos de cada opción y se elige la intervención segura de menor alcance que pueda validarse.
3. **Dado** que se ejecutó una mitigación autorizada, **Cuando** se revisa el servicio, **Entonces** se verifica la salud de las instancias de servicio, el procesamiento de pedidos, los errores y la latencia, y no se cierra el incidente hasta cumplir los criterios de recuperación.

### Historia de usuario 3 - Prevenir la repetición y cerrar el incidente (Prioridad: P2)

Como responsable del servicio, necesito convertir el análisis en acciones, alertas y un procedimiento operativo verificable para reducir el riesgo de repetición y mejorar la respuesta de los equipos.

**Motivo de la prioridad**: Una vez protegido el servicio, corregir los vacíos de detección y de publicación disminuye la probabilidad y el tiempo de impacto de incidentes similares.

**Prueba independiente**: Revisar el informe de cierre y comprobar que cada acción preventiva tiene evidencia que la motiva, prioridad, responsable propuesto y condición verificable de cierre.

**Escenarios de aceptación**:

1. **Dado** el diagnóstico y los límites actuales de monitorización, **Cuando** se prepara el cierre, **Entonces** se proponen señales y alertas para la saturación y espera del pool HTTP y se conservan los SLO existentes.
2. **Dado** que el flujo de integración y despliegue pasó sin validar un rango esperado para el máximo de conexiones ni ejecutar pruebas de carga representativas, **Cuando** se documentan mejoras para la publicación, **Entonces** se especifican controles verificables para ambos vacíos.
3. **Dado** que el análisis concluyó, **Cuando** se entrega el paquete del incidente, **Entonces** incluye un resumen ejecutivo, un anexo técnico, la causa raíz con su nivel de certeza, factores contribuyentes, acciones y un procedimiento de recuperación con puntos de aprobación y detención.

### Casos límite

- La alerta de conexiones DB apunta a un host distinto de la base de datos configurada para `orders-api`; el diagnóstico debe conservar y explicar esta discrepancia.
- Una señal de saturación del cliente HTTP no demuestra por sí sola que el servicio dependiente esté lento; deben compararse la espera del pool y el tiempo de red.
- Los reinicios de instancias de servicio pueden agravar la indisponibilidad sin ser la causa inicial; la cronología debe evitar invertir causa y efecto.
- Una mitigación puede mejorar errores y latencia, pero no demostrar por sí sola que se conservaron todos los pedidos confirmados; la integridad debe comprobarse por separado.
- Los datos disponibles pueden ser incompletos o tener intervalos de muestreo diferentes; las conclusiones deben indicar límites y evitar interpolar como hecho lo que no se midió.
- La solución de menor riesgo podría ser una corrección de configuración o una reversión; la selección requiere evidencia y validación, no debe fijarse de antemano.

## Requisitos *(obligatorio)*

### Requisitos funcionales

- **FR-001**: El análisis DEBE producir una cronología única del incidente que cubra la publicación, los primeros síntomas, el deterioro, la declaración P1, las acciones de respuesta y la recuperación.
- **FR-002**: Cada evento y conclusión DEBE incluir la referencia a su fuente; el informe DEBE distinguir hechos observados, hipótesis, conclusiones confirmadas y preguntas aún abiertas.
- **FR-003**: El analista DEBE correlacionar evidencia de aplicación, infraestructura, balanceador, dependencia `customer-profile`, base de datos, configuración y flujo de integración y despliegue de la publicación.
- **FR-004**: El analista DEBE evaluar la alerta de base de datos frente al host y al origen de datos correctos y documentar evidencia que apoye o contradiga una hipótesis de saturación de la base de datos.
- **FR-005**: El análisis DEBE cuantificar el impacto conocido en clientes, pedidos, latencia, errores y disponibilidad, e indicar explícitamente cuando una dimensión no pueda confirmarse.
- **FR-006**: El análisis DEBE evaluar la relación temporal y funcional entre la publicación de la versión 4.18.0, la reducción documentada del máximo de conexiones de 200 a 20 y las señales de espera del pool, sin declarar causalidad como confirmada hasta validar la evidencia disponible.
- **FR-007**: Toda mitigación propuesta DEBE identificar a la persona responsable, el impacto esperado, el riesgo, el criterio de éxito, el criterio de reversión y las comprobaciones previas y posteriores al cambio.
- **FR-008**: El proceso DEBE preservar órdenes confirmadas. No debe proponer desactivar autenticación, reiniciar la base de datos ni realizar cambios directos a datos.
- **FR-009**: La recuperación DEBE comprobar el procesamiento de pedidos, la salud del servicio, la tasa de errores y la latencia frente a los SLO vigentes antes del cierre del incidente.
- **FR-010**: El paquete de cierre DEBE incluir resumen ejecutivo, anexo técnico, causa raíz y grado de certeza, factores contribuyentes, acciones correctivas y preventivas con responsables propuestos, y lecciones aprendidas.
- **FR-011**: El procedimiento de operación DEBE describir pasos, condiciones previas, evidencia necesaria, aprobaciones humanas, criterios de detención y validación posterior; ninguna acción de riesgo puede ejecutarse automáticamente sin aprobación.
- **FR-012**: Las recomendaciones de observabilidad y del flujo de integración y despliegue DEBEN cubrir la espera y saturación del grupo de conexiones HTTP, la latencia del servicio dependiente, la salud de las instancias y la validación de rangos de configuración y carga representativa.

### Entidades principales *(si la funcionalidad involucra datos)*

- **Incidente**: Identificador, periodo, gravedad, impacto observado, estado y responsables.
- **Evento**: Hora, descripción, dominio de origen, fuente y relación con el incidente.
- **Evidencia**: Artefacto o medición observada, periodo de cobertura, dominio y limitaciones conocidas.
- **Hipótesis**: Explicación candidata, evidencias favorables y contrarias, nivel de certeza y validaciones pendientes.
- **Mitigación**: Acción considerada o ejecutada, responsable, alcance, riesgo, aprobación y criterios de reversión.
- **Validación de recuperación**: Resultado medido posterior a la acción, criterio evaluado, periodo observado y estado de aprobación.
- **Acción de seguimiento**: Hallazgo relacionado, prioridad, responsable propuesto y condición verificable de cierre.

## Criterios de éxito *(obligatorio)*

### Resultados medibles

- **SC-001**: El paquete del incidente registra el 100% de los hitos relevantes que aparecen en las fuentes entregadas, con hora y referencia a la fuente.
- **SC-002**: El 100% de las hipótesis presentadas incluye al menos una evidencia favorable, una evidencia contraria o la indicación explícita de que no se encontró, y su estado de validación.
- **SC-003**: Tras una mitigación aprobada, el servicio mantiene p95 inferior a 1.2 s, p99 inferior a 2.5 s y errores HTTP 5xx inferiores al 5% durante 30 minutos consecutivos de observación.
- **SC-004**: La verificación de integridad confirma cero órdenes confirmadas perdidas por la mitigación; toda discrepancia se bloquea para investigación antes del cierre.
- **SC-005**: El 100% de las acciones correctivas y preventivas del informe tiene un responsable propuesto y un criterio verificable de finalización.
- **SC-006**: Un analista que no participó en el incidente puede seguir el procedimiento operativo y determinar en cada paso si continuar, detenerse o solicitar aprobación, sin depender de instrucciones orales no documentadas.

## Supuestos

- La carpeta mencionada como `ORIENORDER` corresponde a la carpeta existente `ORIONORDERS/`; sus artefactos son la fuente de contexto para esta especificación.
- El alcance es especificar el análisis, la recuperación segura y el cierre preventivo del incidente simulado P1-8841; no incluye implementar cambios en la aplicación ni ejecutar acciones en producción.
- Se mantienen como referencia los objetivos documentados: disponibilidad mensual de 99.95%, p95 menor a 1.2 s, p99 menor a 2.5 s y alerta de 5xx superior a 5% durante 5 minutos.
- La ventana de 30 minutos de estabilidad para validar la recuperación es un criterio propuesto para este caso, no un SLO ya documentado.
- Los artefactos entregados son datos de análisis y no credenciales reales; cualquier dato adicional de producción debe tratarse según las políticas de seguridad y privacidad.
- Fuentes principales: documentación general, incidente P1-8841, conversación de crisis, métricas y registros de aplicación, trazas APM, métricas del balanceador y de las instancias, eventos de Kubernetes, alarmas de base de datos, configuración HTTP, publicación de la versión 4.18.0, registro del flujo de integración y despliegue, SLO y notas de seguridad bajo `ORIONORDERS/`.
