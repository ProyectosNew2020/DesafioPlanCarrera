# Investigación y decisiones: OrionOrders P1-8841

## Decisión 1: Tratamiento de las fuentes

**Decisión**: Mantener un registro de eventos y observaciones con ruta de origen,
hora/período, dominio, clasificación y limitaciones. Citar la ruta de cada artefacto
en los documentos analíticos.

**Razón**: El escenario pide correlacionar subsistemas y evitar atribuciones sin
evidencia. Los artefactos tienen diferentes formatos y frecuencias.

**Alternativas consideradas**:
- Redactar una narrativa sin tabla de evidencias: se rechaza porque dificulta
  comprobar los hechos y distinguir una hipótesis de una observación.
- Tratar cada archivo como una conclusión independiente: se rechaza porque las
  señales contradictorias requieren correlación temporal y entre dominios.

## Decisión 2: Evaluación de la alerta de base de datos

**Decisión**: Comparar `db-reporting-01` de las alertas con el datasource de
`orders-api`, que apunta a `orders-primary-01`, e incluir la comprobación DBA
de 118/300 conexiones, CPU 44% y cero esperas por bloqueo como evidencia contra
una saturación confirmada de la base primaria.

**Razón**: Una alerta en un host distinto no demuestra degradación del origen de
datos usado por el servicio.

**Alternativas consideradas**:
- Declarar la base de datos como causa raíz por la severidad de la alerta: se
  rechaza porque el host no coincide con el datasource.
- Descartar totalmente la base de datos: se rechaza porque la evidencia solo
  reduce el respaldo de esa hipótesis, no elimina todo riesgo de base de datos.

## Decisión 3: Evaluación del cliente HTTP y servicio dependiente

**Decisión**: Contrastar `clientPoolWaitMs`, señales `leased=20 max=20`,
`customer_profile_pool_wait_ms`, latencia de red downstream y cambio de límite
de 200 a 20 conexiones de la versión 4.18.0. Formular el pool como hipótesis
fuerte, no como causalidad confirmada hasta documentar evidencia concluyente.

**Razón**: Los registros y trazas muestran crecimiento de espera mientras la
latencia habitual o medida de red de customer-profile permanece comparativamente
baja; el cambio de configuración es temporalmente cercano al inicio.

**Alternativas consideradas**:
- Atribuir el problema a lentitud del servicio dependiente: no explica por sí
  sola la baja latencia de red medida.
- Declarar el cambio de configuración causa confirmada: se rechaza hasta validar
  relación, comportamiento y efecto mediante datos suficientes.

## Decisión 4: Interpretación de los reinicios y del escalado

**Decisión**: Registrar los fallos de liveness y la pérdida de instancias como
posibles factores de amplificación. No clasificarlos como causa inicial sin
evidencia temporal que lo demuestre.

**Razón**: La arquitectura indica que liveness atraviesa el mismo servidor y
grupo de hilos de solicitudes de negocio; las métricas muestran fallos de sonda,
escalado HPA y descenso de instancias disponibles durante la degradación.

**Alternativas consideradas**:
- Tomar los reinicios como causa raíz: invierte la cronología de los primeros
  síntomas y puede confundir amplificación con origen.
- Ignorar liveness: omite un mecanismo plausible de amplificación del impacto.

## Decisión 5: Política de recuperación

**Decisión**: Documentar alternativas reversibles (ajuste acotado de ConfigMap/
Helm frente a versión 4.17.6), cada una con aprobación, riesgo, criterios de
éxito/detención y comprobación de integridad. No automatizar ni ejecutar una
alternativa en este alcance.

**Razón**: Se dispone de ambas opciones y la política prohíbe acciones inseguras
que pongan en riesgo pedidos confirmados.

**Alternativas consideradas**:
- Elegir de antemano el rollback: se rechaza porque la corrección de configuración
  también está disponible y puede ser menos amplia.
- Reiniciar la base de datos o editar datos como mitigación: prohibido por las
  restricciones del caso.

## Decisión 6: Criterios de éxito y monitoreo

**Decisión**: Mantener los SLO existentes (99.95% disponibilidad mensual,
p95 < 1.2 s, p99 < 2.5 s) y usar 30 minutos consecutivos de p95 < 1.2 s,
p99 < 2.5 s y 5xx < 5% como criterio propuesto de estabilidad para el caso.
Proponer señales de espera y saturación del pool HTTP.

**Razón**: El límite del 5% en cinco minutos es la alerta actual, no prueba por
sí solo recuperación sostenida. La ventana de 30 minutos es explícitamente una
decisión de validación y no se presenta como un SLO existente.

**Alternativas consideradas**:
- Definir nuevos SLO permanentes sin aprobación: se rechaza; están fuera de la
  especificación y requieren gobierno.
- Validar solo con `/health`: se rechaza porque no verifica pedidos, latencia,
  errores ni integridad.

## Decisión 7: Contratos de interfaz

**Decisión**: No generar `contracts/`.

**Razón**: El trabajo produce documentos de diagnóstico, análisis y operación;
no crea ni cambia interfaces o contratos públicos del servicio.

**Alternativas consideradas**:
- Crear contratos HTTP para los servicios existentes: se rechaza porque el
  alcance no requiere especificar ni modificar sus interfaces.

## Incertidumbres restantes

No bloquean el diseño documental. Los datos entregados no acreditan una
mitigación ejecutada, una causa raíz formalmente confirmada ni una reconciliación
completa de órdenes. Los entregables deben declarar esos límites en lugar de
completar vacíos con suposiciones.
