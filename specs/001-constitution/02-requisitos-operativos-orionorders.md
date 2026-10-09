# Requisitos operativos y controles de calidad para OrionOrders

## 1. Propósito
Este documento complementa el contexto operativo del sistema y define los requisitos mínimos que deben cumplirse para operar, desplegar y recuperar OrionOrders sin comprometer disponibilidad ni integridad de datos.

## 2. Requisitos operativos

### 2.1 Diagnóstico basado en evidencia
Todo diagnóstico debe apoyarse en métricas, trazas, registros y estado de dependencias antes de tomar una decisión de corrección o reversión.

### 2.2 Gestión del incidente
- Debe existir una línea de tiempo única para cada incidente.
- Debe asignarse un responsable por dominio: aplicación, infraestructura, base de datos, seguridad y despliegue.
- Debe documentarse el impacto actual, la causa probable y las decisiones tomadas.

### 2.3 Seguridad y cumplimiento
- No se puede desactivar la autenticación para pruebas.
- No se puede ejecutar cambios directos sobre datos de producción.
- No se puede reiniciar la base de datos de forma arbitraria.
- Cualquier reversión debe preservar las órdenes ya confirmadas.

### 2.4 Cambio seguro
- Los cambios deben tener alcance mínimo, plan de reversión documentado y validación explícita.
- Deben identificarse los efectos sobre dependencias externas y el radio de impacto.
- Debe existir evidencia del estado del servicio antes y después del cambio.

## 3. Requisitos de observabilidad
El sistema debe contar con la capacidad de detectar, medir y diagnosticar los siguientes riesgos:

- saturación del grupo de conexiones HTTP (pool)
- latencia acumulada a servicios dependientes
- caída de disponibilidad por fallos de comprobaciones de actividad o agotamiento de hilos de ejecución
- aumento de errores 5xx en producción
- degradación de CPU y picos de conexiones a la base de datos

Además, deben existir alertas para:

- errores HTTP 5xx
- subida sostenida de latencia p95/p99
- uso de conexiones del grupo por encima del umbral
- reinicios de instancias de servicio por una dependencia operativa

## 4. Controles de calidad para publicaciones
Toda publicación que afecte a OrionOrders debe cumplir, como mínimo, lo siguiente:

- validación del cambio en un entorno de pruebas o preproducción
- revisión de riesgos de configuración y dependencia externa
- verificación de compatibilidad con contratos HTTP y esquemas
- evidencia del estado del servicio antes y después de la publicación
- activación y revisión de alertas relevantes

## 5. Requisitos de recuperación y reversión
- Debe existir una estrategia de mitigación de bajo riesgo antes de ejecutar un cambio mayor.
- La reversión debe poder ejecutarse sin afectar las órdenes ya confirmadas.
- Debe registrarse la comparación entre el estado previo y el posterior.
- Deben conservarse los registros y las métricas del periodo anterior y posterior a la corrección.

## 6. Objetivos de nivel de servicio (SLO) y vigilancia
Los niveles objetivo del servicio deben mantenerse dentro de los parámetros definidos:

- disponibilidad mensual: 99.95%
- p95 de latencia: < 1.2 s
- p99 de latencia: < 2.5 s

Cuando se excedan estos objetivos, debe activarse un procedimiento formal de mitigación y revisión.

## 7. Base de trabajo para la continuidad
La información disponible en ORIONORDERS sugiere que el equipo debe reforzar:

- control del cliente HTTP hacia customer-profile
- monitorización explícita del pool de conexiones
- alertas de cola de espera y saturación de dependencias
- validación de publicaciones con revisión de configuración y alcance del impacto
- procedimientos de recuperación con la mínima intervención manual

Estos requisitos establecen la base para la definición de un plan de recuperación, mitigación preventiva y documentación de incidentes para OrionOrders.
