# Modelo de datos documental: OrionOrders P1-8841

Este modelo describe los registros que componen el paquete de análisis; no implica
crear tablas, servicios ni almacenar datos de producción.

## Entidades

### Incidente

- **Identificador**: requerido; por ejemplo, `P1-8841`.
- **Severidad**: requerida; valor observado en la fuente (por ejemplo, P1).
- **Inicio**: fecha y hora con zona horaria cuando esté disponible.
- **Declaración**: fecha y hora de declaración formal, si está disponible.
- **Impacto**: clientes y síntomas observados; indicar explícitamente valores no conocidos.
- **Estado de análisis**: en curso, diagnóstico provisional o cerrado.
- **Responsables**: persona o función asignada por dominio, si se conoce.

### Evento

- **Incidente asociado**: requerido.
- **Marca de tiempo**: requerida cuando está registrada; conservar la zona horaria de origen.
- **Descripción**: observación concisa y neutral.
- **Dominio**: aplicación, infraestructura, balanceador, dependencia, base de datos,
  configuración, publicación, seguridad u operaciones.
- **Fuente**: ruta relativa exacta de `ORIONORDERS/` y, si es posible, registro o fila.
- **Clasificación**: hecho observado, hipótesis, conclusión o dato desconocido.
- **Limitación**: desfase temporal, intervalo, host distinto u otra salvedad.

### Evidencia

- **Identificador**: único dentro del paquete.
- **Evento relacionado**: uno o más eventos, si aplica.
- **Fuente**: ruta y segmento temporal o dato observado.
- **Medición**: nombre, valor, unidad y ventana, cuando exista.
- **A favor / en contra**: hipótesis a la que respalda o contradice.
- **Calidad**: directa, corroborante o contextual, con justificación.
- **Limitaciones**: datos faltantes, frecuencia de muestreo o discrepancias.

### Hipótesis

- **Enunciado**: explicación candidata, formulada sin presentarla como hecho.
- **Evidencias favorables**: referencias a evidencias.
- **Evidencias contrarias**: referencias a evidencias o anotación de que no se encontraron.
- **Certeza**: baja, media o alta, acompañada de justificación.
- **Estado**: pendiente, respaldada provisionalmente, confirmada o descartada.
- **Próxima validación**: evidencia que aún se necesita para cambiar el estado.

### Mitigación

- **Acción propuesta**: configuración acotada o reversión de versión, sin instrucción
  de ejecución automática.
- **Responsable y aprobación**: requeridos antes de cualquier ejecución real.
- **Alcance e impacto esperado**: servicios y clientes afectados.
- **Riesgo**: incluyendo riesgo a pedidos confirmados y dependencias.
- **Criterio de éxito**: señales medibles posteriores.
- **Criterio de detención/reversión**: condición observable para abortar o revertir.
- **Validación de integridad**: evidencia necesaria sobre órdenes confirmadas.

### Validación de recuperación

- **Periodo observado**: inicio, fin y zona horaria.
- **Salud del servicio**: estado de instancias y comprobaciones de salud.
- **Pedidos**: capacidad de creación y evidencia disponible sobre integridad.
- **Latencia y errores**: p95, p99, tasa 5xx y ventana de medición.
- **Resultado**: aprobado, rechazado o no verificable.
- **Aprobación**: responsable que acepta el resultado.

### Acción de seguimiento

- **Hallazgo asociado**: evidencia o requisito que origina la acción.
- **Tipo**: inmediata, correctiva o preventiva.
- **Prioridad**: explícita y motivada.
- **Responsable propuesto**: función o equipo.
- **Criterio de término**: evidencia concreta y comprobable.
- **Estado**: propuesta, aceptada, en curso o completada.

## Relaciones

- Un incidente contiene muchos eventos y evidencias.
- Una evidencia puede sustentar o contradecir varias hipótesis.
- Una hipótesis puede motivar cero o más mitigaciones propuestas.
- Una mitigación puede requerir una o más validaciones de recuperación.
- Los hallazgos de eventos, evidencias, hipótesis o mitigaciones pueden originar
  acciones de seguimiento.

## Reglas de validación

1. No registrar un evento como hecho sin fuente identificable.
2. No cambiar una hipótesis a confirmada sin evidencia de validación.
3. No declarar recuperación si faltan mediciones requeridas; usar «no verificable».
4. No marcar una mitigación como aprobada sin responsable y autorización.
5. No cerrar una acción preventiva sin su criterio verificable de término.
6. Los valores desconocidos se etiquetan como desconocidos; no se sustituyen por cero.
