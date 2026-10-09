# Publicación de la versión 4.18.0

## Cambios de código
- Nuevo campo opcional `salesChannel`.
- Reestructuración del cliente HTTP de customer-profile.
- Sin cambios declarados en SQL.

## Cambios de configuración
Valores anteriores en producción:
CUSTOMER_PROFILE_MAX_CONNECTIONS=200
CUSTOMER_PROFILE_CONNECT_TIMEOUT_MS=1000
CUSTOMER_PROFILE_READ_TIMEOUT_MS=2000

Valores de la versión 4.18.0:
CUSTOMER_PROFILE_MAX_CONNECTIONS=20
CUSTOMER_PROFILE_CONNECT_TIMEOUT_MS=1000
CUSTOMER_PROFILE_READ_TIMEOUT_MS=2000

Mensaje de confirmación original de Helm:
"normalize http client settings across services"

## Reversión
La versión 4.17.6 está disponible.
También es posible corregir únicamente los valores de ConfigMap o Helm.
