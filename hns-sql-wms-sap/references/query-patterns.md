# Query Patterns — Cube_CODA

Patrones validados y listos para usar o adaptar.

---

## PATRÓN 0 — Base antifallos con ROW_NUMBER (usar como CTE base)

Deduplicar documentos y líneas antes de agregar. Obligatorio cuando el resultado
final necesita contar documentos únicos o agregar cantidades sin fanout.

```sql
WITH RawBase AS (
    SELECT
        D.ERP_DOC_NUMBER,
        D.ERP_LINE_NUMBER,
        D.ERP_DOC_TYPE,
        D.COD_PRODUCT,
        D.QTY_REQUESTED,
        D.QTY_SERVED,
        D.COD_CUSTOMER,
        P.ROUTE_ID,
        P.DATE_ROUTE,
        P.DATETIME_STARTED,
        P.DATETIME_FINISHED,
        P.ASSIGNED_TO,
        P.WAREHOUSE_ID,
        ROW_NUMBER() OVER (
            PARTITION BY D.ERP_DOC_NUMBER
            ORDER BY P.DATETIME_STARTED
        ) AS RN_DOC,
        ROW_NUMBER() OVER (
            PARTITION BY D.ERP_DOC_NUMBER, D.ERP_LINE_NUMBER
            ORDER BY P.DATETIME_STARTED
        ) AS RN_LINEA
    FROM EWMS_CANELLA.dbo.WMS_WAVE_PICKING P
    INNER JOIN EWMS_CANELLA.dbo.WMS_WAVE_DETAIL D
        ON P.ROUTE_ID = D.ROUTE_ID
    WHERE P.WAREHOUSE_ID = '04001'
      AND P.DATETIME_STARTED >= '2026-01-01'
      AND P.DATETIME_STARTED < DATEADD(DAY, 1, EOMONTH(GETDATE(), -1))
      AND P.DATETIME_FINISHED IS NOT NULL
      AND P.STATUS <> '9'
      AND D.STATUS <> '9'
      AND D.ERP_DOC_TYPE IN ('OV', 'OF', 'TR5', 'E')
)
-- RN_DOC = 1   → un registro por documento único
-- RN_LINEA = 1 → un registro por línea única de documento
```

---

## PATRÓN 1 — Fill Rate mensual

```sql
WITH RawBase AS (
    SELECT
        D.ERP_DOC_NUMBER,
        D.ERP_LINE_NUMBER,
        D.QTY_REQUESTED,
        D.QTY_SERVED,
        P.DATE_ROUTE,
        ROW_NUMBER() OVER (
            PARTITION BY D.ERP_DOC_NUMBER, D.ERP_LINE_NUMBER
            ORDER BY P.DATETIME_STARTED
        ) AS RN_LINEA
    FROM EWMS_CANELLA.dbo.WMS_WAVE_PICKING P
    INNER JOIN EWMS_CANELLA.dbo.WMS_WAVE_DETAIL D ON P.ROUTE_ID = D.ROUTE_ID
    WHERE P.WAREHOUSE_ID = '04001'
      AND P.DATETIME_STARTED >= '2026-01-01'
      AND P.DATETIME_FINISHED IS NOT NULL
      AND P.STATUS <> '9' AND D.STATUS <> '9'
      AND D.ERP_DOC_TYPE IN ('OV', 'OF', 'TR5', 'E')
),
Dedup AS (
    SELECT * FROM RawBase WHERE RN_LINEA = 1
)
SELECT
    FORMAT(DATE_ROUTE, 'yyyy-MM')          AS Mes,
    COUNT(*)                                AS Lineas,
    SUM(QTY_REQUESTED)                      AS PiezasSolicitadas,
    SUM(QTY_SERVED)                         AS PiezasServidas,
    CAST(SUM(QTY_SERVED) * 100.0
         / NULLIF(SUM(QTY_REQUESTED), 0)
         AS DECIMAL(5,2))                   AS FillRate_Pct
FROM Dedup
GROUP BY FORMAT(DATE_ROUTE, 'yyyy-MM')
-- sin ORDER BY para Power BI
```

---

## PATRÓN 2 — Tiempos reales de picking por ruta

```sql
-- Tiempo real por ROUTE_ID usando WMS_PICKING_TASKS
-- MAX-MIN porque los HELPER1-10 trabajan en paralelo
SELECT
    T.ERP_DOCUMENT                              AS ROUTE_ID,
    MIN(T.DATETIME_START)                       AS Inicio_Real,
    MAX(T.DATETIME_END)                         AS Fin_Real,
    DATEDIFF(MINUTE,
             MIN(T.DATETIME_START),
             MAX(T.DATETIME_END))               AS Minutos_Reales,
    COUNT(DISTINCT T.COD_USER)                  AS Operarios
FROM EWMS_CANELLA.dbo.WMS_PICKING_TASKS T
WHERE T.DATETIME_START IS NOT NULL
  AND T.DATETIME_END IS NOT NULL
  AND T.DATETIME_END > T.DATETIME_START
GROUP BY T.ERP_DOCUMENT
```

---

## PATRÓN 3 — Cruce SAP ↔ WMS (patrón maestro)

```sql
-- Query desde SAP con JOIN cruzado al WMS
-- Ejecutar desde SBO_CANELLA
SELECT
    T0.DocNum,
    T0.DocDate,
    T0.CardCode,
    T0.CardName,
    T3.SlpName                          AS Vendedor,
    T0.U_EstadoWMS,
    T0.U_OlaPicking,
    T4.ROUTE_ID,
    T1.RESULT                           AS EstadoInterfaz
FROM SBO_CANELLA.dbo.ORDR T0
INNER JOIN SBO_CANELLA.dbo.OSLP T3
    ON T0.SlpCode = T3.SlpCode
LEFT JOIN SBO_CANELLA.dbo.WMS_ESTADOS_DESPACHOS T4
    ON T4.ERP_DOC_NUMBER = CAST(T0.DocNum AS NVARCHAR)
    AND T4.ERP_DOC_TYPE = 'OV'
LEFT JOIN [128.1.200.58].[EWMS_CANELLA].[dbo].[WMS_INTERFACES] T1
    ON T1.ROUTE_ID = T4.ROUTE_ID
WHERE EXISTS (
    SELECT 1 FROM SBO_CANELLA.dbo.RDR1 T7
    WHERE T7.DocEntry = T0.DocEntry
      AND T7.WhsCode = 'PM122'
)
  AND T0.DocStatus = 'O'
  AND T0.CANCELED = 'N'
```

---

## PATRÓN 4 — Stock actual WMS vs SAP

```sql
-- Comparar stock WMS (por ubicación) vs SAP (contable)
-- Ejecutar desde EWMS_CANELLA con linked server a SAP
SELECT
    W.COD_PRODUCT                               AS ItemCode,
    I.ItemName,
    SUM(W.QUANTITY)                             AS Stock_WMS,
    S.OnHand                                    AS Stock_SAP,
    SUM(W.QUANTITY) - S.OnHand                  AS Diferencia
FROM EWMS_CANELLA.dbo.WMS_STOCK_X_LOCATION W
INNER JOIN [128.1.200.167].[SBO_CANELLA].[dbo].[OITM] I
    ON I.ItemCode = W.COD_PRODUCT
LEFT JOIN [128.1.200.167].[SBO_CANELLA].[dbo].[OITW] S
    ON S.ItemCode = W.COD_PRODUCT
    AND S.WhsCode = 'PM122'
WHERE W.COD_WAREHOUSE = '04001'
GROUP BY W.COD_PRODUCT, I.ItemName, S.OnHand
HAVING SUM(W.QUANTITY) <> ISNULL(S.OnHand, 0)
-- sin ORDER BY para Power BI
```

---

## PATRÓN 5 — Estado de interfaces (errores de sync)

```sql
-- Ver interfaces con errores o pendientes de envío a SAP
SELECT
    I.ROUTE_ID,
    I.RESULT,
    I.DATE_CREATED,
    I.DATE_UPDATED,
    IH.ERP_DOC_TYPE,
    IH.ERP_DOC_NUMBER,
    IH.ERP_COMPANY,
    IE.ERROR_MESSAGE
FROM EWMS_CANELLA.dbo.WMS_INTERFACES I
LEFT JOIN EWMS_CANELLA.dbo.WMS_INTERFACES_HEAD IH
    ON IH.ROUTE_ID = I.ROUTE_ID
LEFT JOIN EWMS_CANELLA.dbo.WMS_INTERFACES_ERROR IE
    ON IE.ROUTE_ID = I.ROUTE_ID
WHERE I.RESULT NOT IN ('OK', 'SUCCESS')
   OR I.DATE_UPDATED IS NULL
-- sin ORDER BY para Power BI
```

---

## PATRÓN 6 — Productividad de operario

```sql
WITH Tareas AS (
    SELECT
        T.COD_USER                              AS Operario,
        FORMAT(T.DATETIME_START, 'yyyy-MM')     AS Mes,
        COUNT(*)                                AS Tareas_Realizadas,
        SUM(T.QTY_SERVED)                       AS Piezas_Pickeadas,
        AVG(DATEDIFF(SECOND,
                     T.DATETIME_START,
                     T.DATETIME_END))           AS Seg_Promedio_Por_Tarea
    FROM EWMS_CANELLA.dbo.WMS_PICKING_TASKS T
    WHERE T.DATETIME_START IS NOT NULL
      AND T.DATETIME_END IS NOT NULL
      AND T.DATETIME_END > T.DATETIME_START
      AND T.DATETIME_START >= '2026-01-01'
    GROUP BY T.COD_USER, FORMAT(T.DATETIME_START, 'yyyy-MM')
)
SELECT * FROM Tareas
-- sin ORDER BY para Power BI
```

---

## PATRÓN 7 — Órdenes abiertas en SAP sin ola en WMS

```sql
-- OVs liberadas al WMS (U_EstadoWMS = '1') pero sin ROUTE_ID todavía
SELECT
    T0.DocNum,
    T0.DocDate,
    T0.CardCode,
    T0.CardName,
    T0.DocTotal,
    T0.U_EstadoWMS,
    T0.U_OlaPicking
FROM SBO_CANELLA.dbo.ORDR T0
WHERE T0.DocStatus = 'O'
  AND T0.CANCELED = 'N'
  AND T0.U_EstadoWMS = '1'
  AND (T0.U_OlaPicking IS NULL OR T0.U_OlaPicking = '')
  AND EXISTS (
      SELECT 1 FROM SBO_CANELLA.dbo.RDR1 L
      WHERE L.DocEntry = T0.DocEntry
        AND L.WhsCode = 'PM122'
        AND L.LineStatus = 'O'
  )
```

---

## PATRÓN 8 — Recepciones WMS (llegadas)

```sql
SELECT
    R.COD_WAREHOUSE,
    R.DATE_RECEPTION,
    R.COD_PRODUCT,
    R.LOT_NUMBER,
    R.QUANTITY_RECEIVED,
    R.QUANTITY_PENDING,
    R.STATUS,
    R.ERP_PURCHASE_ORDER,
    R.COD_USER
FROM EWMS_CANELLA.dbo.WMS_ARRIVAL_RECEPTION R
WHERE R.COD_WAREHOUSE = '04001'
  AND R.DATE_RECEPTION >= '2026-01-01'
-- sin ORDER BY para Power BI
```

---

## PATRÓN 9 — ROUTE_ID: extraer prefijo

```sql
-- Identificar tipo de ruta por prefijo (PTB, WP-, KT-)
CASE
    WHEN CHARINDEX('-', ROUTE_ID) > 0
        THEN LEFT(ROUTE_ID, CHARINDEX('-', ROUTE_ID) - 1)
    ELSE LEFT(ROUTE_ID, PATINDEX('%[0-9]%', ROUTE_ID) - 1)
END AS Prefijo_Ruta

-- PTB → picking general (OV, OF, TR5)
-- WP  → wave picking (OV, OF)
-- KT  → kitting interno (excluir de reportería comercial)
```

---

## PATRÓN 10 — Top 20 SKUs con mayor diferencia real (vista auditada)

```sql
SELECT TOP 20
    COD_PRODUCT,
    PRODUCT,
    qty_ewms,
    qty_sap,
    qty_ewms - qty_sap - qty_reserved   AS diff_real,
    qty_diff                             AS diff_original,
    qty_reserved,
    qty_rec,
    qty_errors,
    qty_otros,
    staging_detalle
FROM VW_SBO_STOCK_ALERT_AUDIT_V2026
WHERE whs    = 'PM122'
  AND qty_diff <> 0
-- sin ORDER BY para Power BI
```

---

## PATRÓN 11 — SKUs recurrentes cruzando histórico con auditoría actual

```sql
WITH params AS (
    SELECT CAST(0 AS BIT) AS incluir_hoy, CAST(GETDATE() AS DATE) AS fecha_hoy
),
hist AS (
    SELECT
        h.cod_product,
        MAX(h.product)                                              AS product,
        COUNT(DISTINCT CASE WHEN qty_diff <> 0 THEN stock_date END) AS dias_con_diff,
        MIN(CASE WHEN qty_diff <> 0 THEN stock_date END)            AS primera_fecha_diff,
        MAX(CASE WHEN qty_diff <> 0 THEN stock_date END)            AS ultima_fecha_diff,
        MIN(h.qty_diff)                                             AS diff_min_hist,
        MAX(h.qty_diff)                                             AS diff_max_hist,
        CAST(AVG(CAST(h.qty_diff AS FLOAT)) AS DECIMAL(18,2))      AS diff_prom_hist
    FROM VW_SBO_STOCK_ALERT_HIST_V2026 h
    CROSS JOIN params p
    WHERE h.whs = 'PM122' AND h.qty_diff <> 0
      AND (p.incluir_hoy = 1 OR h.stock_date < p.fecha_hoy)
    GROUP BY h.cod_product
),
audit AS (
    SELECT cod_product, product, qty_ewms, qty_sap, qty_diff,
           qty_reserved, qty_rec, qty_otros, qty_errors, staging_detalle
    FROM VW_SBO_STOCK_ALERT_AUDIT_V2026
    WHERE whs = 'PM122' AND qty_diff <> 0
)
SELECT
    h.cod_product,
    COALESCE(a.product, h.product)      AS product,
    h.dias_con_diff,
    h.primera_fecha_diff,
    h.ultima_fecha_diff,
    h.diff_min_hist,
    h.diff_max_hist,
    h.diff_prom_hist,
    a.qty_diff                          AS diff_actual,
    a.qty_ewms, a.qty_sap,
    a.qty_reserved, a.qty_rec, a.qty_otros, a.qty_errors,
    CASE
        WHEN h.dias_con_diff >= 10 AND a.staging_detalle IS NULL     THEN 'CRONICO_PURO'
        WHEN h.dias_con_diff >= 10 AND a.staging_detalle IS NOT NULL THEN 'CRONICO_STAGING'
        WHEN h.dias_con_diff BETWEEN 4 AND 9 AND a.staging_detalle IS NULL  THEN 'RECURRENTE_PURO'
        WHEN h.dias_con_diff BETWEEN 4 AND 9 AND a.staging_detalle IS NOT NULL THEN 'RECURRENTE_STAGING'
        ELSE 'OCASIONAL'
    END AS clasificacion,
    CASE
        WHEN a.staging_detalle IS NOT NULL THEN 'STAGING_O_RUTA_ABIERTA'
        WHEN a.qty_rec > 0                 THEN 'RECEPCION_PENDIENTE_UBICAR'
        WHEN a.qty_reserved > 0            THEN 'RESERVA_DE_RECEPCION'
        ELSE 'DIFERENCIA_PURA_ENTRE_SISTEMAS'
    END AS causa_probable,
    a.staging_detalle
FROM hist h
INNER JOIN audit a ON a.cod_product = h.cod_product
-- sin ORDER BY para Power BI
```

---

## PATRÓN 12 — Stock disponible real por SKU (fórmula del view)

```sql
-- Calcula el stock disponible igual que lo hace VW_SBO_STOCK_ALERT_AUDIT_V2026
-- Útil para validar un SKU específico directamente desde las tablas base
-- Reemplazar '113138' con el COD_PRODUCT a validar

SELECT
    '113138'                                AS cod_product,
    -- Stock WMS disponible (excluye staging)
    (
        SELECT SUM(s.QUANTITY)
        FROM WMS_STOCK_X_LOCATION s
        INNER JOIN DEF_LOCATIONS l ON s.LOCATION = l.LOCATION
                                   AND l.COD_WAREHOUSE = '04001'
        WHERE s.COD_PRODUCT = '113138'
          AND l.LOCATION_TYPE NOT IN ('8','9','33','4','5','3')
          AND l.LOCATION NOT IN ('04001PM122','004DESGLOSE01','04001SOBRANTE','04FALTANTE01')
    )                                       AS qty_wms,
    -- Stock SAP PM122
    (
        SELECT OnHand
        FROM [128.1.200.167].SBO_CANELLA.dbo.OITW
        WHERE ItemCode = '113138' AND WhsCode = 'PM122'
    )                                       AS qty_sap,
    -- Errores de interfaz activos
    (
        SELECT ISNULL(qty, 0)
        FROM VW_WMS_INTERFACE_ERR_BY_PROD
        WHERE COD_PRODUCT = '113138' AND erp_whs = 'PM122'
    )                                       AS qty_errors
```
