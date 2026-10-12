# Feature Specification: Adaptive HTTP Pool Management

**Feature Branch**: `005.12-arquitectura-eventos-TO-BE`

**Created**: 2026-10-11

**Status**: Draft

**Input**: User description: Crear una propuesta sostenible para controlar preventivamente la saturación del pool HTTP de `orders-api`, partiendo de la arquitectura AS-IS y documentando un mecanismo objetivo compatible con Java que aumente capacidad de forma controlada y gestione conexiones inactivas sin interrumpir solicitudes en curso.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Review the proposed TO-BE architecture (Priority: P1)

Como responsable de arquitectura y operación de OrionOrders, necesito revisar una propuesta TO-BE que amplíe la arquitectura AS-IS con gestión adaptativa del pool HTTP, para reducir la probabilidad de que una saturación produzca esperas, errores o degradación de pedidos.

**Why this priority**: El entregable debe explicar una estrategia sostenible, sus límites y su relación con el flujo actual antes de iniciar cualquier implementación.

**Independent Test**: Revisar la carpeta TO-BE y confirmar que el diagrama conserva el flujo AS-IS, muestra el mecanismo nuevo en el límite de `orders-api` y explica señales, decisiones, límites y resultados.

**Acceptance Scenarios**:

1. **Given** la arquitectura AS-IS y la evidencia disponible del incidente P1-8841, **When** el equipo revisa el diagrama TO-BE, **Then** identifica el pool HTTP existente, la llamada síncrona a `customer-profile-service`, la detección anticipada de presión y el ajuste controlado de capacidad sin confundirlos con el flujo asíncrono de auditoría.
2. **Given** la propuesta de gestión dinámica, **When** el propietario de `customer-profile-service` y el equipo de operaciones revisan sus límites, **Then** pueden identificar los límites por instancia y globales que deben aprobarse antes de habilitar incrementos automáticos.

---

### User Story 2 - Adapt capacity safely under rising demand (Priority: P1)

Como responsable del servicio, necesito que la solución detecte señales sostenidas de presión y aumente la capacidad dentro de límites aprobados, para atender demanda creciente antes de que la cola local deteriore la experiencia de los clientes.

**Why this priority**: La evidencia del incidente muestra que la espera local y las solicitudes pendientes crecieron al agotarse las conexiones disponibles; la capacidad debe adaptarse antes de cruzar el límite operativo.

**Independent Test**: Ejecutar una prueba de carga controlada con una rampa de demanda y revisar detección, decisiones de aumento, límites efectivos, continuidad de solicitudes activas y métricas resultantes.

**Acceptance Scenarios**:

1. **Given** que las señales de presión superan los umbrales aprobados durante el periodo sostenido definido, **When** la demanda aumenta, **Then** la solución incrementa gradualmente la capacidad antes de que se agote el límite disponible, sin superar límites por instancia ni el presupuesto global aprobado.
2. **Given** que las señales de presión son transitorias o no están corroboradas, **When** se evalúa la capacidad, **Then** la solución evita escalar repetidamente y registra la señal o decisión para su revisión.
3. **Given** que la demanda se mantiene dentro de la capacidad global aprobada, **When** la solución adapta la capacidad, **Then** las solicitudes en curso completan sin interrupción atribuible al ajuste.

---

### User Story 3 - Reclaim idle capacity and handle limits (Priority: P2)

Como responsable de operaciones, necesito recuperar conexiones que permanecen inactivas y conocer qué ocurre cuando se alcanza el límite seguro, para evitar consumo innecesario y mantener el control cuando el downstream no admite más capacidad.

**Why this priority**: La adaptación debe ser sostenible en ambos sentidos: libera capacidad sin cortes y evita que una respuesta automática amplíe el impacto sobre una dependencia compartida.

**Independent Test**: Probar una caída de demanda con conexiones activas e inactivas y, por separado, un escenario que alcance el límite aprobado; verificar cierres seguros, alertas y ausencia de crecimiento fuera de límites.

**Acceptance Scenarios**:

1. **Given** que existen conexiones inactivas durante el plazo aprobado y otras conexiones tienen solicitudes en curso, **When** la demanda baja, **Then** solo se retiran conexiones elegibles que estén inactivas y las solicitudes activas continúan hasta completarse.
2. **Given** que la demanda supera la capacidad máxima aprobada, **When** no es posible aumentar más, **Then** la solución no rebasa los límites, no cierra conexiones activas y genera una alerta accionable con la presión observada.
3. **Given** que las señales necesarias están ausentes o desactualizadas, **When** la solución no puede decidir con evidencia suficiente, **Then** mantiene un modo seguro definido, evita ajustes agresivos y comunica la condición a operaciones.

### Edge Cases

- La demanda aumenta más rápido que el tiempo requerido para aumentar capacidad; la solución no debe exceder límites para intentar ocultar la degradación.
- El pool alcanza su máximo local o el presupuesto global del downstream mientras persiste la presión.
- Una conexión marcada para retiro vuelve a recibir una solicitud o aún mantiene una solicitud activa; no se debe interrumpir esa solicitud.
- La presión desaparece poco después de un incremento; la solución debe evitar oscilaciones frecuentes entre ampliar y reducir capacidad.
- Las métricas llegan con retraso, son incompletas o presentan valores contradictorios entre pool wait y latencia de red; no se debe atribuir automáticamente la causa al servicio downstream.
- El servicio se reinicia o se deshabilita la gestión adaptativa durante solicitudes activas; el cambio de modo no debe forzar el cierre de solicitudes en curso.
- La capacidad agregada de varias instancias se aproxima al límite que el propietario de `customer-profile-service` autorizó.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El entregable DEBE crear la carpeta `specs/005-implement/entregables/005.12-arquitectura-eventos-TO-BE` y contener una propuesta TO-BE enlazada con `005.11-arquitectura-eventos-AS-IS`.
- **FR-002**: El diagrama TO-BE DEBE conservar y distinguir los flujos existentes de pedidos y auditoría, e incorporar el flujo de observación de presión, decisión de capacidad, aplicación del ajuste y vigilancia posterior en el ámbito de `orders-api`.
- **FR-003**: La propuesta DEBE distinguir señales de espera/adquisición local del pool de latencia de red y salud de `customer-profile-service`, para no tratar una señal aislada como prueba de degradación downstream.
- **FR-004**: La solución propuesta DEBE detectar presión sostenida y señales anticipadas de agotamiento usando criterios observables y configurables que deberán validarse antes de su habilitación.
- **FR-005**: La solución propuesta DEBE aumentar capacidad en pasos controlados, únicamente ante evidencia suficiente y dentro de límites aprobados por instancia y para el conjunto de instancias.
- **FR-006**: La propuesta DEBE incluir límites globales coordinados con el propietario del downstream; si estos límites o la capacidad segura no están definidos, el aumento automático no debe habilitarse.
- **FR-007**: La reducción de capacidad DEBE retirar únicamente conexiones inactivas que cumplan la política de inactividad aprobada; las conexiones que transporten solicitudes en curso DEBEN permanecer disponibles hasta completar dichas solicitudes.
- **FR-008**: Al alcanzar los límites aprobados con demanda todavía alta, la solución DEBE detener el crecimiento, mantener las solicitudes en curso y emitir una alerta con las señales, los límites alcanzados y la acción operativa requerida.
- **FR-009**: La propuesta DEBE definir un comportamiento seguro para datos de observabilidad ausentes o desactualizados, y una forma de deshabilitar la adaptación y volver a una configuración estable sin interrumpir solicitudes en curso.
- **FR-010**: El diseño DEBE definir métricas, registro de decisiones y alertas para presión del pool, solicitudes pendientes, cambios de capacidad, conexiones inactivas retiradas, límites alcanzados y errores de ajuste.
- **FR-011**: El mecanismo propuesto DEBE estar destinado a ejecutarse en el contexto Java de `orders-api` y propone `Apache HttpClient 5` para ajustar el pool por instancia y retirar conexiones inactivas; la versión compatible y su integración DEBEN verificarse antes de implementar. La progresión ilustrativa `20 → 30 → 40` no representa límites aprobados ni autorización de producción.
- **FR-012**: El TO-BE DEBE definir validaciones controladas para carga creciente y decreciente, transiciones con solicitudes activas, límites máximos, señales defectuosas y reversión; toda prueba en producción requiere aprobación y plan reversible.
- **FR-013**: El diseño DEBE conservar la llamada síncrona existente a `customer-profile-service` en el camino crítico y no presentarla como reemplazada por mensajería o por el flujo asíncrono de auditoría.
- **FR-014**: La propuesta DEBE mantener como referencias de validación los SLO operativos vigentes: disponibilidad mensual de 99.95%, p95 menor que 1.2 s y p99 menor que 2.5 s; no debe afirmar recuperación por el mero hecho de aumentar conexiones.

### Key Entities

- **Señal de presión**: observación temporal de uso, espera o solicitudes pendientes que permite distinguir un evento transitorio de una tendencia de saturación.
- **Política de capacidad**: umbrales, pasos de ajuste, límites por instancia y límites globales aprobados para proteger `customer-profile-service`.
- **Conexión HTTP**: recurso de comunicación con estados de disponible, en uso o inactivo, sujeto a reglas distintas de ampliación y retiro.
- **Solicitud en curso**: petición de pedido que ya utiliza el servicio y debe completar sin que el ajuste de capacidad la interrumpa.
- **Decisión de ajuste**: incremento, reducción, pausa o rechazo de ajuste, junto con evidencia, capacidad previa/posterior y motivo.
- **Evento de operación**: alerta o registro asociado a presión sostenida, límite alcanzado, ajuste realizado, dato inválido o modo seguro.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: En el 100% de las pruebas controladas de rampa de demanda dentro del perfil aprobado, se emite una señal preventiva al cruzar el umbral configurado y antes de alcanzar el límite máximo aprobado.
- **SC-002**: En el 100% de las pruebas de aumento, reducción, deshabilitación y límite máximo, ninguna solicitud que ya esté en curso se interrumpe por una acción de administración de conexiones.
- **SC-003**: En el 100% de las ejecuciones de prueba, la capacidad total permanece dentro de los límites aprobados para cada instancia y para el conjunto de instancias.
- **SC-004**: En al menos el 95% de las pruebas con conexiones elegibles inactivas, estas se retiran dentro del plazo de inactividad configurado; ninguna conexión con solicitud activa se retira antes de finalizarla.
- **SC-005**: En pruebas dentro del presupuesto de capacidad aprobado, el servicio cumple los SLO vigentes de p95 < 1.2 s y p99 < 2.5 s durante la ventana de validación acordada.
- **SC-006**: El diagrama y la propuesta reciben revisión explícita de aplicación/operaciones y del responsable de `customer-profile-service`, y no dejan sin asignar los límites de capacidad global antes de recomendar habilitar la adaptación.

## Assumptions

- La solicitud en esta fase es producir una propuesta de arquitectura y criterios verificables en la carpeta TO-BE; la implementación Java y cualquier cambio productivo se planificarán y aprobarán después.
- Java es una restricción explícita del usuario para el contexto de ejecución; Apache HttpClient 5 es la biblioteca solicitada como propuesta, pendiente de verificar versión e integración porque no se encontró fuente Java ni manifiesto de dependencias.
- La secuencia `20 → 30 → 40` ilustra pasos candidatos por instancia desde el máximo de 20 observado; cada paso está sujeto a máximos de salud y capacidad downstream aprobada, por lo que no constituye un límite operativo autorizado.
- El propietario de `customer-profile-service` y operaciones aprobarán capacidad global, límites por instancia, umbrales, incrementos y periodos de inactividad con base en pruebas de carga; no se inventan valores de capacidad.
- El flujo síncrono `orders-api` → `customer-profile-service` continúa siendo necesario para validar el perfil en una solicitud de pedido.
- La carpeta AS-IS y la evidencia de P1-8841 son la referencia inicial; la propuesta debe diferenciar claramente hechos existentes, componentes propuestos y decisiones pendientes.
- El nombre “arquitectura de eventos TO-BE” identifica el entregable solicitado; no implica añadir un broker ni sustituir la llamada síncrona existente.
- Cualquier ejecución en producción requiere revisión, autorización, observabilidad y plan de reversión de acuerdo con la constitución del proyecto.
