# Diferencias de Inventario SAP vs WMS — Auditoría

## Vistas disponibles

| Vista                             | Estado  | Descripción                                              |
|:----------------------------------|:--------|:---------------------------------------------------------|
| VW_SBO_STOCK_ALERT_AUDIT_V2026    | ✅ Usar | Diferencias auditadas, fórmula corregida, con staging    |
| VW_SBO_STOCK_ALERT_V2026          | ⚠️ Bug  | Tiene bug qty_errors — solo usar para comparar           |
| VW_SBO_STOCK_ALERT_HIST_V2026     | ✅ Usar | Fotos diarias históricas — job nocturno. Desde 2026-04-18|
| VW_WMS_INTERFACE_ERR_BY_PROD      | ✅ Usar | Errores de interfaz agrupados por producto y bodega      |

### Columnas clave compartidas entre vistas

| Columna         | Descripción                                               |
|:----------------|:----------------------------------------------------------|
| COD_PRODUCT     | Código del artículo                                       |
| PRODUCT         | Descripción                                               |
| whs             | Bodega SAP. Filtrar siempre: `whs = 'PM122'`              |
| qty_ewms        | Stock WMS disponible (excluye staging/MOR)                |
| qty_sap         | Stock SAP OnHand en PM122                                 |
| qty_diff        | ⚠️ En V2026: con bug. En AUDIT/HIST: confiable            |
| qty_reserved    | Stock WMS con picking asignado                            |
| qty_rec         | Stock en proceso de recepción física                      |
| qty_otros       | Stock en ubicaciones MOR / staging (excluido de qty_ewms) |
| qty_errors      | Movimientos con error en interfaz WMS→SAP                 |
| stock_date      | Solo en HIST: fecha de la foto diaria                     |
| staging_detalle | Solo en AUDIT: descripción textual de ubicaciones staging |

### Fórmula correcta de diferencia real
```sql
-- NUNCA usar qty_diff de VW_SBO_STOCK_ALERT_V2026 directamente
-- SIEMPRE usar:
qty_ewms - qty_sap - qty_reserved   AS diff_real
```

---

## Sistema de clasificación de diferencias

### Clasificación por tipo (AUDITORIA_CLAUDIO_2026)

| Clasificación              | Condición                                          | Acción               |
|:---------------------------|:---------------------------------------------------|:---------------------|
| CUADRADO                   | diff_real = 0 Y qty_diff = 0                       | Sin acción           |
| FALSO_POR_ERRORES          | diff_real = 0 Y qty_diff ≠ 0                       | Era el bug qty_errors|
| STAGING_O_RUTA_ABIERTA     | diff_real ≠ 0 Y qty_otros > 0                      | Revisar operaciones  |
| ERROR_INTERFAZ_SIN_STAGING | diff_real ≠ 0 Y qty_errors ≠ 0 Y qty_otros = 0    | Revisar interfaz     |
| DIFERENCIA_PURA            | diff_real ≠ 0 Y qty_errors = 0 Y qty_otros = 0    | La más crítica       |

### Prioridad de atención

| Prioridad    | Condición                        | Significado                        |
|:-------------|:---------------------------------|:-----------------------------------|
| 1-Urgente    | diff_real ≠ 0, sin errores, sin staging | Diferencia limpia inexplicable |
| 2-Operaciones| qty_otros > 0                    | Hay stock en staging/ruta abierta  |
| 3-Interfaz   | qty_errors ≠ 0                   | Error de sync WMS→SAP              |
| 4-Sin accion | diff_real = 0                    | Sistemas cuadrados                 |

### Persistencia histórica (basada en fotos diarias)

| Clasificación  | Criterio          | Interpretación                          |
|:---------------|:------------------|:----------------------------------------|
| CRONICO        | ≥ 10 días con diff| Diferencia estructural — requiere acción|
| RECURRENTE     | 4–9 días con diff | Patrón repetitivo — monitorear          |
| ESPORADICO     | 1–3 días con diff | Puede ser transitorio                   |
| NUEVO_HOY      | No está en hist   | Primer día — observar                   |

### Clasificación combinada (AUDITORIA_SKUS_RECURRENTES)

| Clasificación       | Días  | Staging | Acción sugerida                              |
|:--------------------|:------|:--------|:---------------------------------------------|
| CRONICO_PURO        | ≥10   | No      | Conteo físico + revisar ajuste/interfaz SAP  |
| CRONICO_STAGING     | ≥10   | Sí      | Cerrar operación WMS antes de ajustar        |
| RECURRENTE_PURO     | 4–9   | No      | Monitoreo + validación física/documental     |
| RECURRENTE_STAGING  | 4–9   | Sí      | Revisar ubicación/ruta actual                |
| OCASIONAL           | 1–3   | —       | Observar siguiente foto antes de escalar     |

---

## Hallazgo estructural — Pallets Fantasma en MOR

### Diagnóstico validado (2026-05-12)

| Métrica                    | Valor          |
|:---------------------------|:---------------|
| Total filas afectadas      | 168,407        |
| Total unidades WMS retenidas | 1,233,903    |
| Unidades en bodega 04001   | **673,194**    |
| Pallets fantasma (04001)   | 16,355         |
| Productos distintos (04001)| 12,414         |
| Promedio antigüedad        | 302 días       |
| Antigüedad máxima          | 475 días       |
| **Impacto en diferencia**  | **84% del total WMS vs SAP** |

**Causa raíz:** picking completado → verificación y despacho nunca ejecutados → pallet nunca cerrado → stock nunca liberado de ubicación MOR → infla WMS permanentemente.

**Rutas más afectadas:** MOR20, MOR07, MOR51, MOR28

**Efecto en fotografías diarias:** estas unidades aparecen en TODA foto a hora 0 porque nunca cambian. Son un "piso fijo" de diferencia.

### Query: detectar pallets fantasma

```sql
SELECT
    s.COD_PRODUCT,
    LEFT(s.LOCATION, 5)          AS ruta_mor,
    s.LOCATION                   AS ubicacion_fisica,
    s.QUANTITY                   AS unidades_wms,
    s.STATUS,
    s.WAVE_PALLET_ID             AS pallet_id,
    s.DATETIME_LAST_TRANS,
    DATEDIFF(DAY, s.DATETIME_LAST_TRANS, GETDATE()) AS dias_sin_movimiento,
    CASE
        WHEN DATEDIFF(DAY, s.DATETIME_LAST_TRANS, GETDATE()) <= 3   THEN '1-Reciente (0-3d)'
        WHEN DATEDIFF(DAY, s.DATETIME_LAST_TRANS, GETDATE()) <= 30  THEN '2-Normal   (4-30d)'
        WHEN DATEDIFF(DAY, s.DATETIME_LAST_TRANS, GETDATE()) <= 90  THEN '3-Alerta   (31-90d)'
        WHEN DATEDIFF(DAY, s.DATETIME_LAST_TRANS, GETDATE()) <= 180 THEN '4-Critico  (91-180d)'
        ELSE                                                             '5-FANTASMA (+180d)'
    END                          AS estado_antiguedad
FROM WMS_STOCK_X_LOCATION s
WHERE s.LOCATION LIKE 'MOR%'
  AND s.QUANTITY  > 0
  AND s.STATUS    = '1'
-- sin ORDER BY para Power BI
```

### Query: resumen por bodega

```sql
SELECT
    wp.WAREHOUSE_ID,
    COUNT(DISTINCT s.WAVE_PALLET_ID)  AS pallets_fantasma,
    COUNT(DISTINCT s.COD_PRODUCT)     AS productos_distintos,
    SUM(s.QUANTITY)                   AS unidades_retenidas,
    MIN(s.DATETIME_LAST_TRANS)        AS movimiento_mas_antiguo,
    AVG(DATEDIFF(DAY, s.DATETIME_LAST_TRANS, GETDATE())) AS promedio_dias
FROM WMS_STOCK_X_LOCATION s
INNER JOIN WMS_WAVE_PALLETS wpal ON wpal.WAVE_PALLET_ID = s.WAVE_PALLET_ID
INNER JOIN WMS_WAVE_PICKING wp   ON wp.ROUTE_ID = wpal.ROUTE_ID
WHERE s.LOCATION LIKE 'MOR%'
  AND s.QUANTITY  > 0
  AND s.STATUS    = '1'
  AND DATEDIFF(DAY, s.DATETIME_LAST_TRANS, GETDATE()) > 180
GROUP BY wp.WAREHOUSE_ID
-- sin ORDER BY para Power BI
```

---

## Queries de auditoría

### Resumen general de diferencias (vista auditada)

```sql
SELECT
    COUNT(*)                                                AS total_productos,
    SUM(CASE WHEN qty_diff <> 0 THEN 1 ELSE 0 END)         AS productos_con_diferencia,
    SUM(CASE WHEN qty_diff  > 0 THEN 1 ELSE 0 END)         AS wms_mayor_sap,
    SUM(CASE WHEN qty_diff  < 0 THEN 1 ELSE 0 END)         AS sap_mayor_wms,
    SUM(qty_diff)                                           AS diferencia_neta,
    SUM(ABS(qty_diff))                                      AS diferencia_absoluta,
    SUM(qty_ewms)                                           AS total_wms,
    SUM(qty_sap)                                            AS total_sap,
    SUM(qty_reserved)                                       AS total_reservado,
    SUM(qty_rec)                                            AS total_en_recepcion,
    SUM(qty_errors)                                         AS total_errores
FROM VW_SBO_STOCK_ALERT_AUDIT_V2026
WHERE whs = 'PM122'
```

### Clasificador de diferencias reales vs ruido

```sql
SELECT
    COD_PRODUCT,
    PRODUCT,
    qty_ewms,
    qty_sap,
    qty_reserved,
    qty_errors,
    qty_otros,
    qty_diff                                        AS diff_original,
    qty_ewms - qty_sap - qty_reserved               AS diff_real,
    (qty_ewms - qty_sap - qty_reserved) - qty_diff  AS distorsion_por_errores,
    CASE
        WHEN (qty_ewms - qty_sap - qty_reserved) = 0 AND qty_diff = 0
            THEN 'CUADRADO'
        WHEN (qty_ewms - qty_sap - qty_reserved) = 0 AND qty_diff <> 0
            THEN 'FALSO_POR_ERRORES'
        WHEN (qty_ewms - qty_sap - qty_reserved) <> 0 AND qty_otros > 0
            THEN 'STAGING_O_RUTA_ABIERTA'
        WHEN (qty_ewms - qty_sap - qty_reserved) <> 0 AND qty_errors <> 0 AND qty_otros = 0
            THEN 'ERROR_INTERFAZ_SIN_STAGING'
        WHEN (qty_ewms - qty_sap - qty_reserved) <> 0 AND qty_errors = 0 AND qty_otros = 0
            THEN 'DIFERENCIA_PURA'
        ELSE 'REVISAR'
    END AS clasificacion,
    CASE
        WHEN (qty_ewms - qty_sap - qty_reserved) = 0  THEN '4-Sin accion'
        WHEN qty_errors = 0 AND qty_otros = 0          THEN '1-Urgente'
        WHEN qty_otros  > 0                            THEN '2-Operaciones'
        WHEN qty_errors <> 0                           THEN '3-Interfaz'
        ELSE '4-Sin accion'
    END AS prioridad
FROM VW_SBO_STOCK_ALERT_V2026
WHERE whs = 'PM122'
-- sin ORDER BY para Power BI
```

### SKUs crónicos — matriz histórica

```sql
SELECT
    COD_PRODUCT,
    PRODUCT,
    COUNT(DISTINCT stock_date)                              AS dias_en_historia,
    COUNT(DISTINCT CASE WHEN qty_diff <> 0
          THEN stock_date END)                              AS dias_con_diferencia,
    MIN(stock_date)                                         AS primera_foto,
    MAX(stock_date)                                         AS ultima_foto,
    MIN(qty_diff)                                           AS diff_minima,
    MAX(qty_diff)                                           AS diff_maxima,
    AVG(CAST(qty_diff AS FLOAT))                            AS diff_promedio
FROM VW_SBO_STOCK_ALERT_HIST_V2026
WHERE whs = 'PM122'
GROUP BY COD_PRODUCT, PRODUCT
-- sin ORDER BY para Power BI
```

### SKUs con diferencia real y sin errores de interfaz (los más críticos)

```sql
WITH ultima_foto AS (
    SELECT
        COD_PRODUCT, PRODUCT,
        qty_ewms, qty_sap,
        qty_ewms - qty_sap  AS diff_cruda,
        qty_errors, qty_reserved, qty_rec,
        qty_diff            AS diff_auditada,
        stock_date
    FROM VW_SBO_STOCK_ALERT_HIST_V2026
    WHERE whs = 'PM122'
      AND stock_date = (SELECT MAX(stock_date) FROM VW_SBO_STOCK_ALERT_HIST_V2026 WHERE whs = 'PM122')
),
historial AS (
    SELECT
        COD_PRODUCT,
        COUNT(DISTINCT stock_date)  AS dias_en_hist,
        MIN(stock_date)             AS primera_vez,
        MAX(stock_date)             AS ultima_vez
    FROM VW_SBO_STOCK_ALERT_HIST_V2026
    WHERE whs = 'PM122' AND qty_diff <> 0
    GROUP BY COD_PRODUCT
)
SELECT
    f.COD_PRODUCT, f.PRODUCT,
    f.stock_date        AS fecha_foto,
    f.qty_ewms, f.qty_sap,
    f.diff_cruda        AS diff_wms_vs_sap,
    f.qty_errors, f.qty_reserved, f.qty_rec,
    h.dias_en_hist,
    h.primera_vez, h.ultima_vez,
    CASE
        WHEN h.dias_en_hist >= 10           THEN 'CRONICO'
        WHEN h.dias_en_hist BETWEEN 4 AND 9 THEN 'RECURRENTE'
        ELSE                                     'ESPORADICO'
    END AS persistencia
FROM ultima_foto f
INNER JOIN historial h ON h.COD_PRODUCT = f.COD_PRODUCT
WHERE f.diff_cruda  <> 0
  AND f.qty_errors   = 0
  AND h.dias_en_hist >= 3
-- sin ORDER BY para Power BI
```

### Auditoría de pallet específico (ciclo de vida completo)

```sql
-- Reemplazar 'LPS0006-49' con el WAVE_PALLET_ID a investigar

-- BLOQUE 1: Stock actual en ese pallet
SELECT '1-STOCK ACTUAL' AS bloque,
    s.LOCATION, s.COD_PRODUCT, s.QUANTITY, s.STATUS,
    s.WAVE_PALLET_ID, s.ROUTE_ID, s.DATETIME_LAST_TRANS,
    DATEDIFF(DAY, s.DATETIME_LAST_TRANS, GETDATE()) AS dias_sin_movimiento
FROM WMS_STOCK_X_LOCATION s
WHERE s.WAVE_PALLET_ID = 'LPS0006-49';

-- BLOQUE 2: Ruta/ola asociada
SELECT '2-RUTA ASOCIADA' AS bloque,
    wp.ROUTE_ID, wp.DATE_ROUTE, wp.STATUS AS estado_ruta,
    wp.DATETIME_CREATED, wp.DATETIME_DISPATCHED,
    wp.ASSIGNED_TO, wp.PILOT_ID, wp.TRUCK_ID
FROM WMS_WAVE_PALLETS wpal
INNER JOIN WMS_WAVE_PICKING wp ON wpal.ROUTE_ID = wp.ROUTE_ID
WHERE wpal.WAVE_PALLET_ID = 'LPS0006-49';

-- BLOQUE 3: Productos asignados en la ola
SELECT '3-DETALLE OLA' AS bloque,
    wpd.ROUTE_ID, wpd.WAVE_PALLET_ID, wpd.COD_PRODUCT,
    wpd.QTY, wpd.STATUS AS estado_linea, wpd.COD_USER,
    wpd.SOURCE_LOCATION, wpd.COD_CUSTOMER, wpd.QTY_CHECKED
FROM WMS_WAVE_PALLETS_DETAIL wpd
WHERE wpd.WAVE_PALLET_ID = 'LPS0006-49';

-- BLOQUE 4: Transacciones generadas
SELECT '4-TRANSACCIONES' AS bloque,
    t.TRANSACTION_NUMBER, t.TRANSACTION_TYPE, t.TRANSACTION_DATETIME,
    t.COD_PRODUCT, t.QTY, t.LOCATION_SOURCE, t.LOCATION_TARGET,
    t.COD_USER, t.ERP_SENT, t.ERP_DOCUMENT
FROM WMS_TRANSACTIONS t
WHERE t.LICENSEPLATE = 'LPS0006-49';

-- BLOQUE 5: Log de interfaz SAP
SELECT '5-LOG INTERFAZ SAP' AS bloque,
    il.LOG_DATETIME, il.INTERFACE_TYPE, il.RESULT,
    il.ERP_NUMBER, il.COD_PRODUCT, il.COMMENTS
FROM WMS_INTERFACES_LOG il
WHERE il.TRANSACTION_NUMBER = 'LPS0006-49'
   OR il.ERP_NUMBER         = 'LPS0006-49';
```

---

## DEF_LOCATIONS — Tipos de ubicación

Tabla catálogo que define el tipo de cada ubicación física.

| Campo          | Descripción                                         |
|:---------------|:----------------------------------------------------|
| LOCATION       | Código de la ubicación                              |
| COD_WAREHOUSE  | Bodega ('04001')                                    |
| LOCATION_TYPE  | Tipo numérico (ver catálogo abajo)                  |

### LOCATION_TYPE — Catálogo validado

| LOCATION_TYPE | Clasificación | Incluir en stock disponible |
|:--------------|:--------------|:----------------------------|
| 1             | Almacenamiento normal (004A) | ✅ Sí             |
| 2             | Picking face              | ✅ Sí               |
| 3             | Recepción                 | ❌ No (qty_rec)     |
| 4             | Staging salida            | ❌ No (qty_otros)   |
| 5             | Consolidación             | ❌ No (qty_otros)   |
| 8             | MOR / rutas               | ❌ No (qty_otros)   |
| 9             | Rampa / despacho          | ❌ No (qty_otros)   |
| 33            | Empaque especial          | ❌ No (qty_otros)   |

**Filtro para stock disponible (igual que usa el view):**
```sql
INNER JOIN DEF_LOCATIONS l ON s.LOCATION = l.LOCATION
                           AND l.COD_WAREHOUSE = '04001'
WHERE l.LOCATION_TYPE NOT IN ('8','9','33','4','5','3')
  AND l.LOCATION NOT IN ('04001PM122','004DESGLOSE01','04001SOBRANTE','04FALTANTE01')
```

**Filtro para ubicaciones staging:**
```sql
WHERE l.LOCATION_TYPE IN ('8','9','33','4','5')
```
