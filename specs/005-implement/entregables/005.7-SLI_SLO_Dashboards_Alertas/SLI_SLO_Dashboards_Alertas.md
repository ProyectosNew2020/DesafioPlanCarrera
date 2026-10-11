# <span style="color:#0B3C5D">SLI/SLO, Dashboards y Alertas</span>

## <span style="color:#0B3C5D">1. Objetivo</span>

Proponer mejoras para medir la disponibilidad, fiabilidad y respuesta del servicio mediante indicadores claros, dashboards útiles y alertas accionables.

## <span style="color:#0B3C5D">2. Propuesta de SLI y SLO</span>

### <span style="color:#0B3C5D">SLI recomendados</span>

• Disponibilidad del servicio por ventana de tiempo.
• Tiempo de respuesta promedio y percentil 95.
• Tasa de errores del servicio por petición.
• Tasa de fallos en transacciones críticas.
• Tiempo de recuperación ante incidentes.

### <span style="color:#0B3C5D">SLO sugeridos</span>

• 99.9% de disponibilidad mensual.
• 95% de peticiones respondiendo en menos de 2 segundos.
• Menos del 1% de errores en transacciones críticas.
• Recuperación de incidentes en menos de 30 minutos para servicios críticos.
• Cero pérdida de datos en procesos de alta prioridad.

## <span style="color:#0B3C5D">3. Mejora en dashboards</span>

• Centralizar métricas de negocio, infraestructura y experiencia de usuario en un mismo panel.
• Separar indicadores por servicio, entorno y criticidad.
• Incluir tendencias históricas y comparativos por periodo.
• Mostrar segmentación por región, canal o tipo de usuario.
• Incorporar métricas de capacidad para anticipar saturaciones.

## <span style="color:#0B3C5D">4. Mejora en alertas</span>

• Definir alertas por severidad: crítica, alta, media y baja.
• Alertar por síntomas antes que por fallas totales.
• Incluir contexto: servicio afectado, impacto, entorno y posibles causas.
• Evitar ruido con alertas agregadas y reglas de deduplicación.
• Asignar responsables y rutas de escalamiento por tipo de incidente.

## <span style="color:#0B3C5D">5. Recomendaciones operativas</span>

• Revisar SLI/SLO cada trimestre con base en el comportamiento real del sistema.
• Entrenar al equipo para interpretar dashboards y responder rápidamente a alertas.
• Automatizar acciones de respuesta ante fallas repetitivas.
• Realizar pruebas de continuidad para validar la efectividad de alertas y recuperación.

## <span style="color:#0B3C5D">6. Conclusión</span>

La mejora de SLI/SLO, dashboards y alertas permite detectar problemas antes de que afecten a usuarios, priorizar incidentes de manera objetiva y apoyar decisiones basadas en indicadores medibles.
