# Documentación inicial del proyecto

## Propósito
Este documento marca la base documental inicial para OrionOrders y sirve como punto de entrada a la gobernanza y al trabajo operativo del equipo.

## Ubicación de referencia
- Constitución formal: `.specify/memory/constitution.md`
- Material de contexto operativo: `ORIONORDERS/`
- Carpeta de especificación: `specs/`

## Objetivos del equipo
- Mantener la estabilidad y la integridad de datos del servicio.
- Diagnosticar incidentes con evidencia y no con suposiciones.
- Aplicar cambios seguros con planificación de reversión y validación.
- Documentar incidentes, decisiones y aprendizajes para cierre operativo.

## Principios fundamentales
1. Evidencia antes de decisiones.
2. Propiedad compartida y trazabilidad del incidente.
3. Cambio seguro, reversible y validado.
4. Calidad y observabilidad como requisito de entrega.
5. Documentación continua y mejora operativa.

## Alcance inicial
La documentación inicial cubre la estructura mínima de gobierno, operación y entrega del proyecto. Para el trabajo más detallado se usarán las carpetas temáticas existentes dentro de `ORIONORDERS/` y las especificaciones que se generen en `specs/`.

## Documentación complementaria
- `specs/01-contexto-operativo-orionorders.md`: síntesis del escenario, arquitectura, cronología y hallazgos clave del caso.
- `ORIONORDERS/00_DOCUMENTACION_GENERAL/01_ARQUITECTURA(3).md`: vista general de la topología del servicio.
- `ORIONORDERS/01_SOPORTE_INCIDENTE/02_P1-8841(2).txt`: cronología del incidente P1.
- `ORIONORDERS/05_DEVOPS_CICD_RELEASE/03_RELEASE_4.18.0(2).md`: detalles de la publicación y de la reversión.
- `ORIONORDERS/07_SEGURIDAD/17_security_notes(2).md`: restricciones operativas y seguridad.

## Recomendación de uso
- Usar la constitución para decisiones de gobierno.
- Usar la carpeta `specs/` para requisitos, planes y documentación técnica específica.
- Usar `ORIONORDERS/` como repositorio de contexto operativo, análisis y referencias del caso.
