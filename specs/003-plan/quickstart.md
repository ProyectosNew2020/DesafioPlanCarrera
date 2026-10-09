# Guía de validación documental: OrionOrders P1-8841

## Propósito

Comprobar que el paquete de análisis producido a partir de `ORIONORDERS/`
conserva trazabilidad, separa evidencia de hipótesis y respeta los límites de
seguridad. Esta guía no ejecuta operaciones en producción.

## Prerrequisitos

- Acceso de lectura a `ORIONORDERS/`.
- Especificación en `specs/specify/spec.md`.
- Entregables de análisis creados según `specs/specify/plan.md` y
  `specs/plan/data-model.md`.

## Escenario 1: reconstruir cronología y evidencia

1. Abrir `ORIONORDERS/01_SOPORTE_INCIDENTE/02_P1-8841(2).txt` y verificar que
   la cronología muestra la publicación de las 09:30, primer síntoma a las
   09:42, primeros reinicios a las 09:45 y declaración P1 a las 09:49.
2. Comparar con `ORIONORDERS/01_SOPORTE_INCIDENTE/16_chat_crisis(2).txt`,
   eventos de Kubernetes y registros de aplicación.
3. En la cronología entregada, comprobar que cada hito tiene hora, fuente y
   clasificación; no corregir silenciosamente discrepancias temporales.

**Resultado esperado**: una cronología rastreable y las discrepancias indicadas.

## Escenario 2: evaluar las hipótesis de base de datos y del pool HTTP

1. Leer `ORIONORDERS/04_BASE_DE_DATOS/09_db_alerts(2).txt`.
2. Confirmar que el host de las alertas (`db-reporting-01`) se distingue del
   datasource (`orders-primary-01`) y de la medición de la base primaria:
   118/300 conexiones, CPU 44% y cero esperas por bloqueo.
3. Contrastar la configuración de
   `ORIONORDERS/05_DEVOPS_CICD_RELEASE/03_RELEASE_4.18.0(2).md` con
   `ORIONORDERS/06_CONFIGURACION/12_http_client_config(2).yaml`.
4. Revisar espera del pool y latencia de red en los registros y trazas de
   `ORIONORDERS/02_APLICACION_DESARROLLO/`.
5. Asegurar que el informe denomina la saturación del pool hipótesis respaldada,
   salvo que se incorpore evidencia adicional que confirme causalidad.

**Resultado esperado**: comparación equilibrada que no culpa la base de datos
por una alerta de host diferente ni declara la hipótesis del pool como confirmada
sin validación.

## Escenario 3: revisar seguridad y propuesta de recuperación

1. Comparar las alternativas documentadas con la disponibilidad de 4.17.6 y la
   posibilidad de ajustar solo ConfigMap/Helm.
2. Leer `ORIONORDERS/07_SEGURIDAD/17_security_notes(2).md`.
3. Comprobar que no se propone desactivar autenticación, reiniciar la base de
   datos ni editar directamente datos.
4. Confirmar que toda mitigación tiene responsable, aprobación, alcance,
   criterio de éxito, condición de detención/reversión y verificación de pedidos.

**Resultado esperado**: propuesta documental reversible, sujeta a aprobación y
sin ejecución automática.

## Escenario 4: verificar recuperación y mejoras preventivas

1. Revisar los criterios vigentes de `ORIONORDERS/01_SOPORTE_INCIDENTE/15_current_slo(2).md`.
2. Verificar que la recuperación mide disponibilidad del servicio, pedidos,
   errores 5xx, p95, p99 e integridad; diferenciar los 30 minutos propuestos del SLO.
3. Revisar `ORIONORDERS/05_DEVOPS_CICD_RELEASE/04_pipeline_4.18.0(3).log` y
   confirmar que las propuestas de calidad responden a la ausencia de validación
   de rango para `CUSTOMER_PROFILE_MAX_CONNECTIONS` y de carga similar a producción.
4. Confirmar que cada acción preventiva tiene prioridad propuesta, responsable
   propuesto y criterio verificable de término.

**Resultado esperado**: recomendaciones derivadas de brechas observadas y
criterios de éxito medibles, sin inventar resultados de recuperación.

## Comprobación final

- [ ] Cada afirmación relevante remite a una fuente identificable.
- [ ] Hechos, hipótesis, conclusiones e incertidumbres están separados.
- [ ] Se contrastan señales de todos los dominios relevantes.
- [ ] Ningún procedimiento autoriza una acción prohibida o automática de riesgo.
- [ ] Las métricas y los SLO tienen unidad y ventana de observación.
- [ ] Las órdenes confirmadas y sus verificaciones de integridad se consideran.
- [ ] Los documentos se refieren por rutas relativas correctas.
