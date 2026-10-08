# Requisitos operativos y gates para OrionOrders

## 1. Propósito
Este documento complementa el contexto operativo del sistema y define los requisitos mínimos que deben cumplirse para operar, desplegar y recuperar OrionOrders sin comprometer disponibilidad ni integridad de datos.

## 2. Requisitos funcionales de operación

### 2.1 Diagnóstico basado en evidencia
Todo diagnóstico debe apoyarse en métricas, trazas, logs y estado de dependencias antes de tomar una decisión de corrección o rollback.

### 2.2 Gestión del incidente
- Debe existir una línea de tiempo única para cada incidente.
- Debe asignarse un responsable por dominio: aplicación, infraestructura, base de datos, seguridad y despliegue.
- Debe documentarse el impacto actual, la causa probable y las decisiones tomadas.

### 2.3 Seguridad y cumplimiento
- No se puede desactivar la autenticación para pruebas.
- No se puede ejecutar cambios directos sobre datos de producción.
- No se puede reiniciar la base de datos de forma arbitraria.
- Cualquier rollback debe preservar órdenes ya confirmadas.

### 2.4 Cambio seguro
- Los cambios deben tener alcance mínimo, rollback documentado y validación explícita.
- Deben identificarse los efectos sobre dependencias externas y el radio de impacto.
- Debe existir evidencia de salud antes y después del cambio.

## 3. Requisitos de observabilidad
El sistema debe contar con la capacidad de detectar, medir y diagnosticar los siguientes riesgos:

- saturación del pool HTTP de conexiones
- latencia acumulada a servicios dependientes
- caída de disponibilidad por liveness o thread exhaustion
- aumento de errores 5xx en producción
- degradación de CPU y picos de conexiones a la base de datos

Además, deben existir alertas para:

- errores HTTP 5xx
- subida sostenida de latencia p95/p99
- uso de conexiones del pool por encima del umbral
- reinicios de pod por dependencia operacional

## 4. Quality gates para despliegues
Todo despliegue que afecte OrionOrders debe cumplir, como mínimo, lo siguiente:

- validación del cambio en entorno de pruebas o staging
- revisión de riesgos de configuración y dependencia externa
- verificación de compatibilidad con contratos HTTP y schemas
- evidencia de salud antes y después del despliegue
- activación y revisión de alertas relevantes

## 5. Requisitos de recuperación y rollback
- Debe existir una estrategia de mitigación de bajo riesgo antes de ejecutar un cambio mayor.
- El rollback debe ser ejecutable sin afectar órdenes ya confirmadas.
- Debe registrarse la comparación entre estado previo y posterior.
- Deben mantenerse logs y métricas del periodo previo y posterior a la corrección.

## 6. SLO y vigilancia
Los niveles objetivo del servicio deben mantenerse dentro de los parámetros definidos:

- disponibilidad mensual: 99.95%
- p95 de latencia: < 1.2 s
- p99 de latencia: < 2.5 s

Cuando se excedan estos objetivos, debe activarse un procedimiento formal de mitigación y revisión.

## 7. Base de trabajo para la continuidad
La información disponible en ORIONORDERS sugiere que el equipo debe reforzar:

- control del cliente HTTP a customer-profile
- monitorización explícita del pool de conexiones
- alertas de cola de espera y saturación de dependencias
- validación de release con patrón de configuración y blast radius
- runbooks de recuperación con mínima intervención manual

Estos requisitos establecen la base para la definición de un plan de recuperación, mitigación preventiva y documentación de incidentes para OrionOrders.
