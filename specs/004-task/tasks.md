# Tareas: diagnóstico y recuperación del incidente OrionOrders

**Entradas**: especificación `specs/specify/spec.md`, diseño `specs/plan/` y evidencia de `ORIONORDERS/`.

**Alcance**: producir entregables documentales para P1-8841. No se implementará software ni se ejecutarán cambios en producción. Los datos desconocidos se identificarán como tales; no se afirmará causa raíz confirmada ni integridad de pedidos sin evidencia.

**Pruebas**: la especificación no solicita pruebas de software; cada historia incluye criterios de revisión independiente.

## Formato

- Cada tarea sigue `- [ ] Tnnn [P] [USn] Descripción con ruta`.
- `[P]` se usa únicamente para tareas paralelizables sobre archivos distintos.
- `[USn]` relaciona las tareas de las fases de historia con `specs/specify/spec.md`.
- Los identificadores técnicos, nombres de archivos fuente y métricas se mantienen literales para su búsqueda en `ORIONORDERS/`.

## Fase 1: Preparación

**Propósito**: fijar el inventario y las restricciones comunes del análisis.

- [X] T001 [P] Crear `specs/implement/entregables/00-indice-fuentes.md` con la ruta, dominio, tipo de contenido y periodo cubierto de cada artefacto bajo `ORIONORDERS/`.
- [X] T002 [P] Crear `specs/implement/entregables/01-contexto-y-restricciones.md` con el alcance e impacto de P1-8841, los SLO de `ORIONORDERS/01_SOPORTE_INCIDENTE/15_current_slo(2).md` y las restricciones de `ORIONORDERS/07_SEGURIDAD/17_security_notes(2).md`.

---

## Fase 2: Base de evidencia

**Propósito**: construir una referencia verificable común antes de interpretar el incidente.

- [X] T003 Crear `specs/implement/entregables/02-matriz-evidencias.md` con marca de tiempo, observación, dominio, archivo fuente, clasificación (hecho, hipótesis o dato desconocido) y limitaciones; contrastar `ORIONORDERS/01_SOPORTE_INCIDENTE/02_P1-8841(2).txt`, `ORIONORDERS/01_SOPORTE_INCIDENTE/16_chat_crisis(2).txt`, fuentes de aplicación e infraestructura, alertas de base de datos y artefactos de publicación/configuración.

**Punto de control**: ningún análisis causal comienza hasta que las observaciones incluidas en la matriz tengan fuente identificable.

---

## Fase 3: Historia de usuario 1 — Diagnosticar el incidente con evidencia (Prioridad: P1) — MVP

**Objetivo**: explicar la secuencia y el impacto con afirmaciones verificables y una causa probable correctamente calificada.

**Prueba independiente**: una persona revisora reconstruye los hitos desde sus fuentes y ve separadas las observaciones, las hipótesis y las conclusiones; la alerta de `db-reporting-01` se contrasta con `orders-primary-01`.

- [X] T004 [P] [US1] Crear `specs/implement/entregables/03-cronologia-incidente.md` usando la matriz; incluir publicación 4.18.0 a las 09:30, primera alerta a las 09:42, reinicios desde las 09:45, declaración P1 a las 09:49 y señales posteriores con hora y fuente.
- [X] T005 [P] [US1] Crear `specs/implement/entregables/04-analisis-causal.md` contrastando `db-reporting-01` con el datasource `orders-primary-01`, sus mediciones 118/300 conexiones y CPU 44%, la espera `clientPoolWaitMs`, `leased=20 max=20`, la latencia de red de `customer-profile`, el cambio de conexiones de 200 a 20 y los fallos de liveness; anotar evidencia favorable, contraria y validaciones pendientes por hipótesis.
- [X] T006 [P] [US1] Crear `specs/implement/entregables/05-impacto-del-incidente.md` cuantificando clientes, pedidos, p95, errores y disponibilidad con `ORIONORDERS/03_INFRAESTRUCTURA/10_load_balancer_metrics(2).csv`, `ORIONORDERS/02_APLICACION_DESARROLLO/08_customer_profile_metrics(2).csv`, `ORIONORDERS/02_APLICACION_DESARROLLO/13_apm_trace_breakdown(2).csv` y fuentes P1; marcar cualquier dimensión no confirmable.
- [X] T007 [US1] Integrar los resultados en `specs/implement/entregables/06-informe-diagnostico.md`; citar fuentes, separar hechos e hipótesis, indicar el grado de certeza y no presentar correlación temporal como causa confirmada.

**Punto de control**: el informe explica por qué las alertas DB no demuestran saturación de la base primaria y conserva las incertidumbres.

---

## Fase 4: Historia de usuario 2 — Recuperar el servicio de forma segura (Prioridad: P1)

**Objetivo**: documentar una ruta de recuperación reversible y autorizada que proteja las órdenes confirmadas.

**Prueba independiente**: operaciones puede revisar las alternativas y los criterios para proceder, detenerse o revertir, sin que el documento autorice acciones automáticas.

- [X] T008 [US2] Crear `specs/implement/entregables/07-alternativas-mitigacion.md` comparando un ajuste acotado de ConfigMap/Helm con la reversión a 4.17.6 según `ORIONORDERS/05_DEVOPS_CICD_RELEASE/03_RELEASE_4.18.0(2).md`; documentar impacto, riesgo, alcance, aprobación necesaria y criterios de selección, sin elegir una opción sin evidencia.
- [X] T009 [US2] Crear `specs/implement/entregables/08-procedimiento-recuperacion.md` con condiciones previas, responsables, evidencias, aprobación humana, pasos, límites de seguridad, condiciones de detención y reversión; excluir desactivar autenticación, reiniciar la base de datos o modificar datos directamente.
- [X] T010 [US2] Crear `specs/implement/entregables/09-validacion-recuperacion.md` con comprobaciones de salud, procesamiento e integridad de órdenes, errores 5xx, p95 y p99; usar p95 < 1.2 s, p99 < 2.5 s y 5xx < 5% durante 30 minutos consecutivos como criterio propuesto, distinguirlo del SLO documentado y marcar como «no verificable» lo que carezca de medición.

**Punto de control**: toda mitigación propuesta indica responsable, aprobación, riesgo, resultado esperado, criterio de detención y comprobación de integridad; no se declara recuperación sin datos posteriores.

---

## Fase 5: Historia de usuario 3 — Prevenir la repetición y cerrar el incidente (Prioridad: P2)

**Objetivo**: recomendar mejoras trazables y consolidar un informe de cierre accionable.

**Prueba independiente**: cada acción preventiva remite a un hallazgo y cuenta con prioridad, responsable propuesto y condición verificable de cierre.

- [X] T011 [P] [US3] Crear `specs/implement/entregables/10-mejoras-observabilidad.md` proponiendo señales, paneles y alertas para espera/saturación del pool HTTP, latencia downstream, salud de instancias y SLO; fundamentar cada propuesta en `ORIONORDERS/01_SOPORTE_INCIDENTE/15_current_slo(2).md` y `specs/implement/entregables/04-analisis-causal.md`.
- [X] T012 [P] [US3] Crear `specs/implement/entregables/11-mejoras-publicacion.md` proponiendo controles de rango para `CUSTOMER_PROFILE_MAX_CONNECTIONS` y carga representativa, con base en las brechas registradas en `ORIONORDERS/05_DEVOPS_CICD_RELEASE/04_pipeline_4.18.0(3).log`.
- [X] T013 [US3] Crear `specs/implement/entregables/12-informe-cierre.md` con resumen ejecutivo, anexo técnico, causa raíz y certeza, factores contribuyentes, acciones inmediatas/correctivas/preventivas, responsables propuestos, prioridades y criterios de cierre; enlazar los entregables previos sin afirmar acciones ejecutadas que no consten en la evidencia.

---

## Fase 6: Revisión final

**Propósito**: comprobar coherencia y trazabilidad del paquete.

- [X] T014 Revisar `specs/implement/entregables/00-indice-fuentes.md` y los documentos `specs/implement/entregables/01-contexto-y-restricciones.md` a `specs/implement/entregables/12-informe-cierre.md`; corregir referencias rotas, afirmaciones sin fuente, contradicciones, términos inconsistentes y niveles de certeza superiores a la evidencia disponible.

## Dependencias y orden

- **Fase 1**: T001 y T002 son independientes y paralelizables.
- **Fase 2**: T003 requiere el inventario y contexto; bloquea US1.
- **US1**: T004, T005 y T006 dependen de T003 y pueden hacerse en paralelo; T007 integra sus resultados.
- **US2**: T008 requiere T007; T009 usa T008; T010 define la validación y puede prepararse en paralelo con T009 después de T007.
- **US3**: T011 y T012 requieren el análisis T005/T007 y pueden hacerse en paralelo; T013 integra los hallazgos de las historias.
- **Revisión final**: T014 requiere los entregables que revisa.

## Oportunidades de paralelismo

### Historia 1

```text
T004 -> specs/implement/entregables/03-cronologia-incidente.md
T005 -> specs/implement/entregables/04-analisis-causal.md
T006 -> specs/implement/entregables/05-impacto-del-incidente.md
```

Las tres tareas producen archivos diferentes y comienzan después de T003.

### Historia 3

```text
T011 -> specs/implement/entregables/10-mejoras-observabilidad.md
T012 -> specs/implement/entregables/11-mejoras-publicacion.md
```

Pueden hacerse en paralelo cuando el análisis T005 esté disponible.

## Estrategia de implementación

### MVP: US1

1. Completar T001-T003 para preparar las fuentes.
2. Completar T004-T006 en paralelo.
3. Integrar en T007.
4. Detenerse para verificar la trazabilidad y el grado de certeza antes de continuar.

### Entrega incremental

1. Completar US1 para entregar el diagnóstico verificable.
2. Completar US2 para documentar recuperación segura y validación.
3. Completar US3 para proponer prevención y cerrar el análisis.
4. Ejecutar T014 sobre el paquete completo.

## Notas

- La carpeta fuente existe como `ORIONORDERS/`; se toma como referencia de la carpeta mencionada por el usuario como `ORIENORDER`.
- No hay interfaces o contratos que deban crearse para este trabajo documental.
- Las tareas siguen la plantilla de casilla, identificador, etiquetas aplicables y ruta concreta de salida.
