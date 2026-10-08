# Release 4.18.0

## Cambios de código
- Nuevo campo opcional `salesChannel`.
- Refactor de cliente HTTP de customer-profile.
- Sin cambios declarados en SQL.

## Cambios de configuración
Valores de producción anteriores:
CUSTOMER_PROFILE_MAX_CONNECTIONS=200
CUSTOMER_PROFILE_CONNECT_TIMEOUT_MS=1000
CUSTOMER_PROFILE_READ_TIMEOUT_MS=2000

Valores release 4.18.0:
CUSTOMER_PROFILE_MAX_CONNECTIONS=20
CUSTOMER_PROFILE_CONNECT_TIMEOUT_MS=1000
CUSTOMER_PROFILE_READ_TIMEOUT_MS=2000

Commit message de Helm:
"normalize http client settings across services"

## Rollback
La versión 4.17.6 está disponible.
También es posible corregir únicamente ConfigMap/Helm values.
