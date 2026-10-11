# <span style="color:#0B3C5D">Causa raiz OrionOrders P1-8841</span>

<div style="text-align: justify;">

El análisis del incidente P1-8841 identifica como causa principal una reducción incorrecta del pool de conexiones HTTP en el release 4.18.0, al pasar de 200 a 20 conexiones hacia `customer-profile-service`.

La evidencia muestra que el pool alcanzó su límite (`leased=20 max=20`) y que las solicitudes comenzaron a acumular tiempos de espera elevados (`clientPoolWaitMs`), lo que contribuyó al aumento de la latencia y a la aparición de errores 502 y 503.

La saturación también pudo afectar las comprobaciones de salud de los pods, provocando reinicios de Kubernetes y agravando la pérdida de capacidad.

La acción correctiva propuesta es validar una reversión controlada del parámetro, verificar la recuperación mediante métricas y logs, e incorporar controles de configuración y pruebas de carga al pipeline de despliegue.

</div>
