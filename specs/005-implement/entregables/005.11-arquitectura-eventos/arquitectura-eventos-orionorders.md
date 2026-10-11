# <span style="color:#0B3C5D">OrionOrders | Arquitectura de eventos</span>

> **Diseño propuesto.** El broker y el publicador outbox no están identificados como componentes actuales; se muestran como objetivo de arquitectura.

## <span style="color:#0B3C5D">Vista de arquitectura</span>

![Diagrama de secuencia del incidente](arquitectura-eventos-orionorders.png)

> Diagrama del incidente: reducción incorrecta del pool de conexiones HTTP durante el release 4.18.0.

## <span style="color:#0B3C5D">Causa raíz | evidencia principal del incidente P1-8841</span>

**Causa principal:** reducción incorrecta del pool de conexiones HTTP en el release 4.18.0, al pasar de 200 a 20 conexiones hacia `customer-profile-service`.

La evidencia muestra que el pool alcanzó su límite (`leased=20 max=20`) y que las solicitudes comenzaron a acumular tiempos de espera elevados (`clientPoolWaitMs`), lo que contribuyó al aumento de la latencia y a la aparición de errores 502 y 503.

La saturación pudo afectar las comprobaciones de salud de los pods. Si se cumplen los umbrales de fallos configurados, Kubernetes reinicia los contenedores, lo que puede disminuir la capacidad disponible y agravar el incidente.

La acción correctiva propuesta es validar una reversión controlada del parámetro, verificar la recuperación mediante métricas y logs, e incorporar controles de configuración y pruebas de carga al pipeline de despliegue.

## <span style="color:#0B3C5D">Flujos</span>

• El cliente envía el pedido por HTTPS al balanceador y este lo dirige a `orders-api`.
• `orders-api` valida el perfil mediante una llamada **síncrona** a `customer-profile-service`. Esta dependencia permanece en el camino crítico.
• En una transacción local, `orders-api` persiste el pedido y un evento pendiente en la outbox.
• El publicador de la outbox entrega `OrderCreated` al broker y, tras recibir la confirmación, marca el registro como publicado.
• `audit-service` consume el evento y registra la auditoría. Si ocurre un fallo, vuelve a intentarlo; si se agotan los reintentos, envía el mensaje a una cola de mensajes fallidos (DLQ).

## <span style="color:#0B3C5D">Diseño mínimo del evento</span>

• **Evento:** `OrderCreated`.
• **Campos:** `eventId`, `eventVersion`, `occurredAt`, `orderId`, `correlationId`.
• **Entrega:** al menos una vez. El consumidor elimina duplicados mediante `eventId` y procesa cada evento de forma idempotente.
• **Orden:** usar `orderId` como clave de partición cuando el broker lo permita.
• **Seguridad y operación:** TLS, autenticación y permisos mínimos; alertas sobre la antigüedad de los registros de la outbox, el retraso del consumidor, los reintentos y los mensajes en la DLQ. No incluir datos personales que el consumidor no necesite.

> La arquitectura de eventos desacopla la auditoría; no elimina la llamada síncrona a `customer-profile-service` ni constituye, por sí sola, una mitigación de la causa raíz.
