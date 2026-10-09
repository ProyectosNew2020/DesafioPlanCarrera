# Plan de trabajo: Diagnóstico y recuperación del incidente OrionOrders

**Funcionalidad**: [Especificación](../specify/spec.md)
**Plan técnico compatible con Spec Kit**: [plan de implementación](../specify/plan.md)
**Fecha**: 2026-10-07
**Estado**: Diseño completado

## Objetivo

Producir una explicación trazable del incidente P1-8841 y un conjunto de
entregables operativos seguros: línea de tiempo, matriz de evidencia, análisis
causal, impacto, opciones de mitigación, procedimiento de recuperación y acciones
preventivas.

## Alcance y límites

- Analizar el material suministrado en `ORIONORDERS/`.
- Distinguir observación, hipótesis, conclusión y dato no disponible.
- No modificar la aplicación, la configuración, el clúster ni la base de datos.
- No ejecutar rollback ni otra acción de producción.
- No desactivar autenticación, reiniciar la base de datos ni realizar cambios
  directos de datos.
- Preservar órdenes confirmadas y declarar las verificaciones de integridad que
  la evidencia existente no permite realizar.

## Enfoque

1. Inventariar las fuentes y registrar período, dominio y limitaciones.
2. Normalizar hitos en una cronología y mantener la cita de cada fuente.
3. Contrastar hipótesis de base de datos, cliente HTTP downstream y reinicios,
   evitando anclaje a una alarma aislada.
4. Cuantificar impacto con métricas disponibles y marcar las brechas.
5. Comparar mitigaciones reversibles sin recomendar ejecución automática.
6. Definir validaciones de recuperación a partir de SLO documentados.
7. Derivar propuestas de observabilidad y de controles del flujo de publicación
   de brechas demostradas por los artefactos.
8. Revisar coherencia y trazabilidad antes del informe final.

## Resultados del diseño

- [Decisiones y análisis de fuentes](research.md)
- [Modelo de datos documental](data-model.md)
- [Guía de validación](quickstart.md)
- No se crea `contracts/`: el alcance es documental y no incorpora ni cambia
  interfaces públicas o contratos de integración.

## Criterios de salida

- Todo evento y afirmación técnica relevante apunta a una fuente disponible.
- Hechos, hipótesis, conclusión provisional e incertidumbres aparecen separados.
- Se evalúan señales de aplicación, infraestructura, base de datos, servicio
  dependiente y publicación/configuración.
- Las acciones propuestas identifican alcance, aprobación, criterio de éxito,
  condición de detención y reversión, y no transgreden las restricciones operativas.
- Las medidas preventivas tienen prioridad sugerida, responsable propuesto y
  condición verificable de cierre.

## Comprobación de la constitución

| Principio | Comprobación de diseño | Estado |
|---|---|---|
| Evidencia antes que acción | El modelo separa observaciones y evaluaciones causales e incluye fuente y certeza. | Cumple |
| Propiedad compartida | La cronología agrupa dominios y permite registrar responsables y traspasos. | Cumple |
| Cambio seguro | Las opciones de recuperación requieren autorización humana y comprobación posterior. | Cumple |
| Calidad y observabilidad | Las recomendaciones se derivan de vacíos observados en SLO, métricas y pipeline. | Cumple |
| Aprendizaje y documentación | Se prevé un paquete de cierre con acciones medibles. | Cumple |
| Restricciones operativas | Se mantiene fuera de alcance toda operación directa en producción o datos. | Cumple |

**Puerta posterior al diseño**: aprobada; no quedan decisiones técnicas sin resolver
para un entregable estrictamente documental.
