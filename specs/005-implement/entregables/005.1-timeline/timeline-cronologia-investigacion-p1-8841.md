# <span style="color:#0B3C5D">Cronología de investigación — OrionOrders P1-8841</span>

**Fecha del incidente:** 29 de septiembre de 2026  
**Ventana de evidencia:** 09:21–10:41:30  
**Primer síntoma reportado:** 09:42  
**Declaración P1:** 09:49  
**Recuperación:** no verificable con las fuentes entregadas

## <span style="color:#0B3C5D">1. Propósito y método</span>

Este documento ordena la investigación y reconstruye la secuencia del incidente
correlacionando publicación, aplicación, pool HTTP, Kubernetes, balanceador,
servicio dependiente, base de datos y comunicaciones operativas.

? **Hecho medido/registrado:** valor de telemetría, log, pipeline o evento
  directamente presente en una fuente.
? **Hecho reportado:** hito narrado en la ficha del incidente o el chat.
? **Interpretación:** explicación obtenida al correlacionar fuentes; no equivale
  a causalidad probada.

Las horas del log de aplicación incluyen el offset `-05:00`. Los CSV, eventos
Kubernetes, pipeline y chat no declaran zona horaria ni estado de sincronización
de reloj. Se conservan los valores horarios tal como fueron suministrados; las
correlaciones entre sistemas pueden tener desfase.

La cobertura tampoco es homogénea: el balanceador y customer-profile se agregan
por minuto, APM contiene muestras de trazas, el log registra solicitudes
seleccionadas y Kubernetes aporta eventos puntuales. Las cifras no comparten
necesariamente denominador ni alcance.

## <span style="color:#0B3C5D">2. Investigación organizada por fases</span>

| Fase | Ventana | Hallazgos principales |
|---|---|---|
| A. Publicación y referencia | 09:21–09:41 | Pipeline y smoke tests PASS. La release documenta el cambio del máximo de conexiones del cliente HTTP de 200 a 20. Métricas previas en línea base. |
| B. Inicio y cola local | 09:42–09:44 | Aumenta latencia; crece `clientPoolWaitMs`; pool llega a 20/20 ocupadas y aparecen pendientes. El tiempo de espera local supera el tiempo de red downstream en las muestras citadas. |
| C. Errores e instancias | 09:45–09:52 | Empiezan fallos de liveness y terminaciones después de la espera del pool; aparecen 502/503, cae el número de pods listos y el HPA cambia tamaño deseado a 18. |
| D. Diagnóstico diferencial | 09:47–10:02 | La alarma de conexiones/CPU corresponde a `db-reporting-01`, no al datasource de orders-api. La comprobación de `orders-primary-01` no muestra saturación a las 10:02. El equipo identifica el cambio de configuración y la cola. |
| E. Persistencia de degradación | 09:53–10:41:30 | Los CSV agregados terminan entre 10:14 y 10:15, pero logs de aplicación siguen hasta 10:41:30 con esperas de 9000 ms y 128 pendientes. No se observa recuperación. |

## <span style="color:#0B3C5D">3. Cronología detallada</span>

| Hora | Evento y evidencia | Dominio / fuente | Interpretación y limitación |
|---|---|---|---|
| 09:21 | Build: PASS. | CI/CD — `ORIONORDERS/05_DEVOPS_CICD_RELEASE/04_pipeline_4.18.0(3).log` | Gate de compilación superado; no demuestra el desempeño bajo carga. |
| 09:23 | 486 pruebas unitarias: PASS. | CI/CD — mismo pipeline | No se reportan pruebas de carga similares a producción. |
| 09:25 | Análisis estático: PASS. | CI/CD — mismo pipeline | El registro confirma el resultado de la etapa; no lista aquí sus reglas. |
| 09:26 | Escaneo del contenedor: PASS. | CI/CD — mismo pipeline | No valida la capacidad ni el límite del pool HTTP. |
| 09:28 | `helm template`: PASS. | CI/CD — mismo pipeline | El pipeline valida tipo y presencia, pero no un rango mínimo/máximo de `CUSTOMER_PROFILE_MAX_CONNECTIONS`. |
| 09:29 | Despliegue a producción: PASS. | CI/CD — mismo pipeline | Resultado reportado por el pipeline; no demuestra ausencia de regresiones. |
| 09:30 | Rollout 4.18.0 completo. | CI/CD e incidente — pipeline y `ORIONORDERS/01_SOPORTE_INCIDENTE/02_P1-8841(2).txt` | La ficha ubica aquí la publicación; no especifica estrategia ni duración del rollout. |
| 09:30–09:32 | Smoke `/health` PASS a las 09:31 y `create-test-order` PASS a las 09:32. | CI/CD — pipeline 4.18.0 | Pruebas puntuales; no validan carga sostenida ni integridad de órdenes de clientes. |
| Release 4.18.0 | El máximo documentado `CUSTOMER_PROFILE_MAX_CONNECTIONS` cambia de 200 a 20; connect timeout se mantiene en 1000 ms y read timeout en 2000 ms. | Publicación/configuración — `ORIONORDERS/05_DEVOPS_CICD_RELEASE/03_RELEASE_4.18.0(2).md` | Asociación temporal y funcional con el incidente. Falta historial de valor efectivo por instancia durante el despliegue. |
| 09:35–09:41 | Balanceador: p95 0.7 s, 5xx 0.4%, 253–375 solicitudes/s. Logs muestreados de orders-api: status 200 y `clientPoolWaitMs=0`. APM: espera pool 27–178 ms; customer-profile p95 87.4–137.8 ms. | Balanceador, aplicación y downstream — `10_load_balancer_metrics(2).csv`, `05_orders-api(2).log`, `13_apm_trace_breakdown(2).csv`, `08_customer_profile_metrics(2).csv` | Línea base de las series entregadas; no equivale a telemetría de todas las solicitudes. |
| 09:42 | La ficha identifica la primera alerta de latencia. Balanceador: p95 1.48 s, 5xx 4.94%, 313 solicitudes/s. Log app: status 200, total 499 ms, downstream 168 ms, espera pool 159 ms. APM: total 904 ms, pool 660 ms, red 109 ms, DB 62 ms. | Incidente, balanceador, aplicación y APM — `02_P1-8841(2).txt` y fuentes anteriores | El p95 excede el SLO de 1.2 s. El 5xx agregado está debajo de 5% en esa muestra; las señales miden aspectos distintos. |
| 09:42:20–09:42:40 | APM registra espera pool 771–913 ms, red downstream 154–155 ms y DB 68–72 ms. | APM — `ORIONORDERS/02_APLICACION_DESARROLLO/13_apm_trace_breakdown(2).csv` | En esas trazas, la espera local es el componente dominante; son observaciones puntuales. |
| 09:43:00–09:43:10 | APM muestra pool wait 435–895 ms y red 113–118 ms. | APM — mismo CSV | La diferencia entre espera de adquisición y tiempo de red aumenta en la muestra. |
| 09:43:20 | Warning `leased=20 max=20 pending=19`. Solicitud: total 1670 ms, espera pool 1359 ms, customer-profile 99 ms. | Aplicación/pool — `ORIONORDERS/02_APLICACION_DESARROLLO/05_orders-api(2).log` | Pool al máximo y cola pendiente observados; evidencia de saturación del pool en esta muestra. |
| 09:43:40 | Segundo warning: 20/20 ocupadas, 18 pendientes; espera 1307 ms y downstream 167 ms. | Aplicación/pool — mismo log | Corrobora que la saturación no aparece en una sola línea aislada. |
| 09:44:00–09:44:50 | APM: pool wait 764–1409 ms, red 101–110 ms. A las 09:44:10, log: 20/20, 25 pendientes, espera 1778 ms, downstream 90 ms. Balanceador 09:44: p95 2.17 s, 5xx 6.57%. | Aplicación, APM y balanceador — logs/CSV respectivos | La cola y espera aparecen antes de que la tasa agregada 5xx cruce 5%. La red downstream no explica por sí sola la espera local. |
| 09:45:00–09:45:10 | El pool sigue en 20/20; warnings muestran 17–24 pendientes y solicitudes con espera superior a 1 s. | Aplicación/pool — log orders-api | La espera precede al primer fallo de liveness registrado. |
| 09:45:12–09:45:14 | `orders-api-7d9c` falla liveness por timeout; Kubernetes termina el contenedor. | Kubernetes — `ORIONORDERS/03_INFRAESTRUCTURA/07_k8s_events(2).log` | Primer fallo documentado; ocurre después del inicio de la espera. Posible amplificador, no origen demostrado. |
| 09:46:03–09:46:05 | `orders-api-a12f` falla liveness por timeout y el contenedor se termina. | Kubernetes — mismo log | Segundo caso. La arquitectura indica que liveness comparte servidor HTTP y grupo de hilos con tráfico de negocio; no hay perfil de hilos que cuantifique este mecanismo. |
| 09:46:50 | Primera respuesta 503 en el log: 2520 ms total, pool wait 2231 ms, downstream 152 ms. | Aplicación — log orders-api | La espera local constituye la mayor parte del tiempo registrado para esta solicitud. |
| 09:47 | Balanceador: p95 2.89 s, 5xx 9.52%, 358 solicitudes/s. A las 09:47:00 log app: 502, pool wait 2579 ms, downstream 160 ms. Alerta DB: conexiones 92% en `db-reporting-01`. | Balanceador, aplicación, DB — `10_load_balancer_metrics(2).csv`, log app, `ORIONORDERS/04_BASE_DE_DATOS/09_db_alerts(2).txt` | Coexisten degradación del servicio y alarma de otro host; esta última no prueba saturación de la DB usada por orders-api. |
| 09:48 | Alerta DB CPU 81% en `db-reporting-01`; balanceador p95 3.0 s y 5xx 10.52%. | DB y entrada — alertas DB y CSV balanceador | Host de alerta debe contrastarse con el datasource antes de atribuir causalidad. |
| 09:48:41–09:48:43 | `orders-api-4c21` falla liveness y se termina. | Kubernetes — eventos | Tercer par explícito de fallo/terminación en los eventos entregados. |
| 09:49 | Se declara P1. Balanceador: p95 3.22 s, 5xx 10.66%, 253 solicitudes/s. Customer-profile: p95 122.1 ms y error 0.31%. | Gestión, entrada y downstream — ficha P1 y métricas CSV | Cifras de scopes distintos; no son porcentajes comparables con el mismo denominador. |
| 09:49:40–09:50:50 | Logs app muestran 502/503; pool wait 2307–3533 ms y llamadas downstream 93–141 ms en esas solicitudes. | Aplicación — log orders-api | Evidencia muestreada de espera local elevada durante errores. |
| 09:50:19–09:50:21 | `orders-api-b099` falla liveness y se termina. | Kubernetes — eventos | Cuarto par explícito de fallo/terminación. Configuración de health probe tiene timeout 1 s y umbral de 3 fallos. |
| 09:50–09:51 | Chat: desarrollo estima customer-profile cerca de 100 ms cuando recibe la llamada; operaciones indica que los errores aumentaron después de perder pods. | Operación — `ORIONORDERS/01_SOPORTE_INCIDENTE/16_chat_crisis(2).txt` | Afirmaciones del equipo compatibles con algunas métricas; no cuantifican causalidad. |
| 09:52 | Evento HPA: tamaño deseado 18 por CPU sobre objetivo; CSV de pods registra 9 listos. | Kubernetes — eventos y `ORIONORDERS/03_INFRAESTRUCTURA/06_k8s_pod_metrics(2).csv` | Tamaño deseado no equivale a pods listos; no prueba que el escalado aliviara el pool por instancia. |
| 09:52:10 | Solicitud 502: 3686 ms total, espera 3374 ms, downstream 99 ms. | Aplicación — log orders-api | La espera sigue dominando la muestra. |
| 09:53:10 | Solicitud 502: 3865 ms total, espera 3549 ms, downstream 118 ms; pool 20/20 y 50 pendientes. | Aplicación — log orders-api | Cola mayor que en las primeras advertencias. |
| 09:53:50 | Solicitud 503: espera 4318 ms, downstream 120 ms; 61 pendientes. | Aplicación — log orders-api | Continúa el crecimiento de la espera/cola. |
| 09:54 | DBA aclara que las alertas corresponden a `db-reporting-01` y solicita revisar datasource de orders-api. | Operación/DB — chat de crisis | Corrige la interpretación inicial basada en la alarma DB. |
| 09:54:10 | Solicitud 502: espera 4435 ms, downstream 89 ms; 63 pendientes. | Aplicación — log orders-api | En esta solicitud, espera local es ~50 veces el tiempo downstream medido; comparación puntual, no estadística poblacional. |
| 09:54:11 | `orders-api-83ef` registra fallo liveness; el archivo no aporta terminación asociada. | Kubernetes — eventos | Registrar como fallo de sonda, no como reinicio confirmado. |
| 09:55 | Balanceador p95 5.24 s, 5xx 17.92%, 344 solicitudes/s; alerta DB 96% sigue en reporting. Log app: status 200, total 4630 ms, espera 4387 ms, 62 pendientes. | Entrada, DB y aplicación — métricas/alertas/logs | Un 200 aislado no demuestra recuperación cuando la latencia y cola siguen elevadas. |
| 09:58 | DevOps identifica en chat el cambio de valores de cliente HTTP. Log app: total 5329 ms, espera 4995 ms, downstream 115 ms, 71 pendientes. | Publicación/aplicación — chat y log orders-api | La pista se contrasta con la release; falta verificar la configuración efectiva aplicada. |
| 09:59–10:01 | Pods listos oscilan entre 6 y 8. A las 10:00: solicitud 6270 ms, espera 6111 ms, downstream 85 ms, 87 pendientes. A las 10:01 el chat señala `clientPoolWaitMs` creciente y 20/20. | Kubernetes/aplicación — CSV pods, log y chat | Contador `cumulative_restarts` del CSV no es monotónico; no usarlo como total acumulado fiable. |
| 10:02 | DBA: `orders-primary-01` tiene 118/300 conexiones, CPU 44%, cero lock waiters. Muestra app: total 6299 ms, espera 6133 ms, downstream 96 ms; 6 pods listos. | DB, aplicación e infraestructura — alertas/nota DBA, log y CSV pods | Instantánea no respalda saturación de la primaria a esta hora; no elimina eventos fuera de la muestra. |
| 10:05 | Balanceador p95 7.79 s, 5xx 30.39%; customer-profile agregado p95 120.1 ms, error 0.07%; APM a 10:05:00: espera 6914 ms, red 154 ms, DB 34 ms. | Entrada, dependencia y APM — CSV y trazas | Métricas con distintos scopes; la traza sigue atribuyendo la mayor parte de la demora a la espera local. |
| 10:08 | Pods listos: 4. Balanceador p95 8.78 s, 5xx 33.64%. Log app: total 8777 ms, espera 8460 ms, downstream 98 ms, 120 pendientes. | Kubernetes, entrada y aplicación — CSV/logs | Degradación severa y menor capacidad lista coexisten; no se identifica por sí sola la dirección causal. |
| 10:11 | Balanceador p95 9.38 s, 5xx 35.91%; APM pool wait 9000 ms y red 125 ms. | Entrada y APM — CSV y APM | 9000 ms debe tratarse como valor observado, posiblemente techo de timeout/instrumentación. |
| 10:14 | Última muestra del CSV de pods: 4 listos y 17 en columna `cumulative_restarts`. | Kubernetes — CSV pods | El contador de reinicios es no monotónico; 4 pods listos es la observación puntual. |
| 10:15 | Última muestra balanceador: p95 10.2 s, 5xx 38%, 323 solicitudes/s. Customer-profile p95 143.2 ms, error 0.31%. Log app: 503, total 9336 ms, espera 9000 ms, 128 pendientes. | Entrada, dependencia y aplicación — CSV y log | Última muestra de agregados, no fin demostrado del incidente. |
| 10:20 | Log app: status 200, total 9356 ms, espera 9000 ms, downstream 151 ms, 128 pendientes. | Aplicación — log orders-api | Respuesta aceptada en esta muestra, pero sigue siendo lenta y con cola. |
| 10:30 | Log app: status 200, total 9221 ms, espera 9000 ms, downstream 145 ms, 128 pendientes. | Aplicación — log orders-api | No demuestra normalización. |
| 10:40 | Log app: status 502, total 9380 ms, espera 9000 ms, downstream 168 ms, 128 pendientes. | Aplicación — log orders-api | Persistencia de error en la última parte del archivo. |
| 10:41:30 | Última línea del log: status 200, total 9330 ms, espera 9000 ms; warning de pool 20/20 con 128 pendientes. | Aplicación — log orders-api | Último evento suministrado; 200 no significa recuperación con latencia/cola persistentes. No hay observación posterior. |

## <span style="color:#0B3C5D">4. Evolución cuantitativa</span>

| Hora | p95 balanceador | HTTP 5xx balanceador | Pods listos | Señal de aplicación |
|---|---:|---:|---:|---|
| 09:41 | 0.7 s | 0.4% | 12 | Espera pool 0 ms en logs muestreados. |
| 09:42 | 1.48 s | 4.94% | 12 | APM: pool wait 660–913 ms en muestras. |
| 09:44 | 2.17 s | 6.57% | 11 | 20/20 ocupadas; 25 pendientes a las 09:44:10; espera 1778 ms. |
| 09:47 | 2.89 s | 9.52% | 10 | 502 a 09:47:00; espera 2579 ms; downstream 160 ms. |
| 09:49 | 3.22 s | 10.66% | 10 | P1 declarado; 502 a 09:49:40 y espera 2684 ms. |
| 09:55 | 5.24 s | 17.92% | 10 | Status 200, pero total 4630 ms, espera 4387 ms y 62 pendientes. |
| 10:00 | 6.57 s | 22.12% | 8 | Total 6270 ms, espera 6111 ms y 87 pendientes. |
| 10:08 | 8.78 s | 33.64% | 4 | Total 8777 ms, espera 8460 ms y 120 pendientes. |
| 10:15 | 10.2 s | 38% | 4 (último dato pods 10:14) | 503, espera 9000 ms y 128 pendientes. |
| 10:41:30 | Sin serie posterior a 10:15 | Sin serie posterior a 10:15 | Sin serie posterior a 10:14 | Último log: status 200, espera 9000 ms y 128 pendientes. |

Los valores de pods listos son instantáneas; los tiempos de espera y pendientes
son muestras de logs, no percentiles globales. El valor de espera de 9000 ms
puede representar un límite de timeout o instrumentación.

## <span style="color:#0B3C5D">5. Correlación entre dominios</span>

| Dominio | Evidencia | Conclusión permitida |
|---|---|---|
| Publicación/configuración | 4.18.0 se completa a las 09:30; nota registra reducción de máximo de 200 a 20. | Cambio temporalmente asociado; confirmar el valor efectivo y hacer una prueba controlada antes de declararlo causa formal. |
| Aplicación/pool HTTP | Desde 09:43:20, `leased=20 max=20`, cola pendiente creciente y esperas de cientos de milisegundos hasta 9000 ms. | Saturación observada y espera local dominante en solicitudes citadas. |
| Customer-profile | Métrica agregada p95 generalmente ~87–143 ms; errores menores de 0.4% en la ventana disponible. | No explica por sí sola las esperas locales de varios segundos; no descarta episodios no capturados. |
| Kubernetes | Fallos/terminaciones de liveness desde 09:45; pods listos disminuyen; HPA deseado 18 a las 09:52. | Posible amplificación posterior. No demuestra causa inicial ni que el escalado haya aliviado el pool. |
| Base de datos | Alertas de 92–96% conexiones y CPU 81% en `db-reporting-01`; orders-api usa `orders-primary-01`. A las 10:02: 118/300, CPU 44%, cero bloqueos. | Evidencia no respalda saturación de la primaria a las 10:02; la comprobación es puntual. |
| Balanceador | p95 de 0.7 s a 09:41 y 10.2 s a 10:15; 5xx de 0.4% y 38%. | Deterioro agregado hasta 10:15; no hay métricas de balanceador posteriores. |
| Integridad de pedidos | Ficha refiere ausencia de evidencia inicial de pérdida; smoke crea una orden de prueba antes del incidente. | No es conciliación del periodo afectado. Pérdidas, duplicados y hora de recuperación no verificables. |

## <span style="color:#0B3C5D">6. Interpretación y certeza</span>

1. **El síntoma precede a los reinicios observados:** primera alerta/espera a
   las 09:42; primer timeout de liveness a las 09:45:12. Por esta cronología,
   los reinicios no son el iniciador demostrado.
2. **La espera del pool precede a los errores agregados:** los spans de APM
   empeoran a las 09:42; la saturación 20/20 está registrada a las 09:43:20;
   el balanceador supera 5% de 5xx a las 09:44.
3. **La espera local predomina en las muestras de errores:** a las 09:53–09:54
   hay 3.5–4.4 s de espera frente a 89–120 ms de llamada downstream registrada.
4. **La alerta DB corresponde a otro host:** la alerta apunta a
   `db-reporting-01`; la comprobación puntual de `orders-primary-01` no muestra
   saturación a las 10:02.
5. **No puede inferirse la recuperación al final de una serie:** los CSV
   agregados terminan entre 10:14 y 10:15, pero orders-api sigue registrando
   espera/cola hasta las 10:41:30.

**Conclusión provisional:** la saturación del pool HTTP de orders-api hacia
customer-profile está fuertemente respaldada. El cambio documentado de 200 a
20 conexiones en 4.18.0 es temporal y funcionalmente consistente con los logs,
pero su causalidad no está formalmente confirmada sin la configuración
efectivamente cargada por instancia y una validación controlada.

Los fallos de liveness, la caída de pods listos y el escalado HPA pueden haber
amplificado el impacto; su contribución no está cuantificada. La evidencia
disponible no respalda la DB primaria como causa. No se demuestra mitigación,
recuperación sostenida ni integridad completa de las órdenes.

## <span style="color:#0B3C5D">7. Límites y evidencia pendiente</span>

? Historial de ConfigMap/Helm y valor activo de pool por instancia durante el
  rollout y el incidente.
? Métricas completas por pool: conexiones ocupadas/libres, pendientes, tiempo
  de adquisición, errores y duración de arriendo.
? Correlación de pods terminados con carga, hilos HTTP, liveness, readiness y
  tráfico; validar la anomalía del contador `cumulative_restarts`.
? Telemetría agregada posterior a las 10:15 para establecer la hora de
  recuperación y evaluar p95, p99 y 5xx.
? Conciliación autorizada de órdenes aceptadas, confirmadas, fallidas y
  duplicadas; no modificar datos directamente.



