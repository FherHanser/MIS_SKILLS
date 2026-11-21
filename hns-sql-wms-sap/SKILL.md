---
name: "hns-sql-wms-sap"
description: "Genera consultas SQL Server para el entorno WMS/SAP de CANELLA (Cube_CODA). Usar siempre que el usuario pida una query, consulta, reporte, KPI, cruce de datos, análisis de picking, fill rate, stock, interfaces, despachos, órdenes de venta, traslados, recepciones, devoluciones, tiempos de operario, o cualquier dato que involucre las bases de datos EWMS_CANELLA (WMS) o SBO_CANELLA (SAP). También activar cuando el usuario mencione tablas como WMS_WAVE_PICKING, WMS_PICKING_TASKS, ORDR, OITW, WMS_TRANSACTIONS, WMS_INTERFACES, o cualquier tabla con prefijo WMS_ o nombres de tablas SAP de 4 letras. Activar también para diagnóstico de errores de interfaz, sincronización SAP↔WMS, consultas de Power BI contra estos servidores, diferencias de inventario, auditoría de SKUs, pallets fantasma, SKUs crónicos, clasificación de brechas SAP vs WMS, o cualquier consulta sobre las vistas VW_SBO_STOCK_ALERT_* o VW_SBO_STOCK_DIFF_*.\n"
---

# SQL WMS/SAP — Cube_CODA

## Entorno

| Servidor         | IP              | Base de datos   | Sistema |
|:-----------------|:----------------|:----------------|:--------|
| SAP (ERP)        | 128.1.200.167   | SBO_CANELLA     | SAP B1  |
| WMS (Operativo)  | 128.1.200.58    | EWMS_CANELLA    | WMS     |

**Bodega principal:** `WhsCode = 'PM122'` (SAP) = `WAREHOUSE_ID = '04001'` (WMS)

**Conexión cruzada desde WMS hacia SAP:**
```sql
[128.1.200.167].[SBO_CANELLA].[dbo].[TABLA]
```

---

## Equivalencias Clave SAP ↔ WMS

| Concepto            | SAP (SBO_CANELLA)         | WMS (EWMS_CANELLA)              |
|:--------------------|:--------------------------|:--------------------------------|
| Orden de Venta      | ORDR / RDR1               | ERP_DOC_TYPE = 'OV'             |
| Oferta de Venta     | OQUT / QUT1               | ERP_DOC_TYPE = 'OF'             |
| Traslado bodega     | OWTR / WTR1               | ERP_DOC_TYPE = 'TR5'            |
| Factura directa     | OINV / INV1               | ERP_DOC_TYPE = 'E' (raro <0.01%)|
| Kitting interno     | —                         | ERP_DOC_TYPE = 'TR3' (excluir)  |
| Mov. interno WMS    | —                         | ERP_DOC_TYPE = 'TRS' (excluir)  |
| Código de artículo  | OITM.ItemCode             | COD_PRODUCT                     |
| Código de cliente   | OCRD.CardCode             | COD_CUSTOMER                    |
| Número de documento | ORDR.DocNum               | ERP_DOC_NUMBER                  |
| Ola/Ruta            | ORDR.U_OlaPicking         | WMS_WAVE_PICKING.ROUTE_ID       |

**Filtro estándar para reportería comercial:**
```sql
ERP_DOC_TYPE IN ('OV', 'OF', 'TR5', 'E')
AND P.STATUS <> '9'
AND D.STATUS <> '9'
```

---

## Tablas más usadas — Referencia rápida

### Vistas de auditoría de inventario (EWMS_CANELLA)
| Vista                             | Uso principal                              |
|:----------------------------------|:-------------------------------------------|
| VW_SBO_STOCK_ALERT_AUDIT_V2026    | ✅ Diferencias auditadas — **usar esta**   |
| VW_SBO_STOCK_ALERT_V2026          | ⚠️ Diferencias en tiempo real (bug)        |
| VW_SBO_STOCK_ALERT_HIST_V2026     | Fotos diarias históricas (desde 2026-04-18)|
| VW_WMS_INTERFACE_ERR_BY_PROD      | Errores de interfaz agrupados por producto |

> ⚠️ **`VW_SBO_STOCK_ALERT_V2026` tiene un bug:** incluye `qty_errors` en la fórmula, lo que genera diferencias falsas. Usar siempre `VW_SBO_STOCK_ALERT_AUDIT_V2026` o recalcular como `qty_ewms - qty_sap - qty_reserved`.

### WMS (EWMS_CANELLA)
| Tabla                     | Uso principal              | Filas aprox. |
|:--------------------------|:---------------------------|-------------:|
| WMS_WAVE_PICKING          | Cabecera de ola (1/ruta)   | 98,000       |
| WMS_WAVE_DETAIL           | Líneas por ola (N/ruta)    | 500,000      |
| WMS_PICKING_TASKS         | Tiempos reales de picking  | 502,000      |
| WMS_PICKING_DETAIL        | Detalle de picking         | 772,379      |
| WMS_STOCK_X_LOCATION      | Stock actual por ubicación | 178,226      |
| WMS_TRANSACTIONS          | Movimientos transaccionales| 5,850,000    |
| WMS_INTERFACES            | Estado interfaz con SAP    | 114,035      |
| WMS_INTERFACES_LOG_DET    | Log detalle de interfaces  | 475,682      |
| WMS_ARRIVAL_RECEPTION     | Recepciones                | 241,548      |
| WMS_RELOCATION_TASKS      | Tareas de reubicación      | 592,876      |
| WMS_RETURNS_RECEPTION     | Devoluciones               | 5,448        |
| WMS_WAVE_PALLETS          | Pallets por ola            | 96,616       |
| WMS_WAVE_PALLETS_DETAIL   | Detalle de productos/pallet| 476,467      |
| DEF_LOCATIONS             | Catálogo de ubicaciones    | —            |

### SAP (SBO_CANELLA)
| Tabla   | Descripción                  | Filas aprox.  |
|:--------|:-----------------------------|-------------:|
| OITW    | Stock por almacén            | 19,273,702   |
| OITM    | Maestro de artículos         | 816,839      |
| OCRD    | Socios de negocio            | 254,129      |
| ORDR    | Órdenes de venta (cabecera)  | 44,167       |
| RDR1    | Órdenes de venta (líneas)    | 149,640      |
| ODLN    | Notas de entrega (cabecera)  | 133,506      |
| DLN1    | Notas de entrega (líneas)    | 395,665      |
| OINV    | Facturas (cabecera)          | 116,827      |
| INV1    | Facturas (líneas)            | 454,474      |
| OWTR    | Traslados (cabecera)         | 88,471       |
| WTR1    | Traslados (líneas)           | 215,360      |
| OQUT    | Ofertas de venta (cabecera)  | 101,919      |
| QUT1    | Ofertas de venta (líneas)    | 365,374      |
| OWHS    | Catálogo de bodegas          | 1,413        |

---

## Flujo de documentos

```
ORDR (Orden de Venta)
  │  U_EstadoWMS = '1' → liberado al WMS
  ▼
WMS_WAVE_PICKING (Ola de picking)
  │  ROUTE_ID generado → se escribe en ORDR.U_OlaPicking
  ▼
WMS_PICKING_TASKS (Ejecución física)
  │  Operarios ejecutan, se registran tiempos reales
  ▼
WMS_TRANSACTIONS (ERP_SENT / READY_SENT_ERP)
  │  WMS notifica a SAP
  ▼
ODLN (Nota de Entrega) → actualiza OITW.OnHand
  ▼
OINV (Factura)
```

**Puente bidireccional:** `WMS_ESTADOS_DESPACHOS` (tabla custom en SBO_CANELLA)
- `ERP_DOC_NUMBER` ↔ `ROUTE_ID`

---

## Reglas anti-error (LEER SIEMPRE)

### ⚠️ 1. Fanout en JOINs
`WMS_WAVE_PICKING` (1 fila/ola) × `WMS_WAVE_DETAIL` (N filas/ola) = filas multiplicadas.
**Nunca agregar columnas de PICKING cuando el JOIN incluye DETAIL sin deduplicar.**
→ Ver `references/query-patterns.md` — patrón ROW_NUMBER obligatorio.

### ⚠️ 2. Tiempos de ola ≠ tiempo real
`WMS_WAVE_PICKING.DATETIME_STARTED/FINISHED` = ciclo del registro en sistema (mediana: ~26 seg). **Inútil para KPIs de productividad.**
→ Usar siempre `WMS_PICKING_TASKS.DATETIME_START / DATETIME_END`.
→ Con operarios paralelos (HELPER1-10): usar `MAX(END) - MIN(START)`, nunca `SUM`.

### ⚠️ 3. JOIN key en PICKING_TASKS
`WMS_PICKING_TASKS.ERP_DOCUMENT = WMS_WAVE_PICKING.ROUTE_ID`
**No es ERP_DOC_NUMBER.** El campo se llama `ERP_DOCUMENT` pero guarda el `ROUTE_ID`.

### ⚠️ 4. Power BI — restricciones
- Sin `ORDER BY` en queries nativas (causa error de sintaxis).
- Sin `;` antes de `WITH`.
- Wrapper correcto:
```m
let Source = Sql.Database("128.1.200.58", "EWMS_CANELLA", [Query="WITH Base AS (...) SELECT ... FROM Base"]) in Source
```

### ⚠️ 5. Tablas backup — no usar en producción
Tablas con patrón `_bak_wms_*` y `WMS_STOCK_X_LOCATION_HIST__` son snapshots históricos.

### ⚠️ 6. OITW — tabla pesada
19M de filas. Siempre filtrar por `WhsCode = 'PM122'` al inicio.

### ⚠️ 7. Bug qty_errors en VW_SBO_STOCK_ALERT_V2026
La vista original suma `qty_errors` a la diferencia, generando brechas falsas (`FALSO_POR_ERRORES`).
**Nunca usar `qty_diff` de `VW_SBO_STOCK_ALERT_V2026` como diferencia real.**
→ Usar `VW_SBO_STOCK_ALERT_AUDIT_V2026` o calcular: `qty_ewms - qty_sap - qty_reserved`.

### ⚠️ 8. Pallets fantasma en MOR — piso fijo de diferencias
**673,194 unidades** en ubicaciones MOR de bodega 04001 con +180 días sin movimiento.
Picking completado pero pallets nunca cerrados → inflan WMS artificialmente en TODA fotografía.
Representan el **84% de la diferencia estructural WMS vs SAP**.
→ Ver `references/stock-diff.md` para diagnóstico completo y query de detección.

---

## Archivos de referencia

Consultar según el tipo de query:

| Necesidad                              | Archivo                              |
|:---------------------------------------|:-------------------------------------|
| Schema completo WMS                    | `references/wms-schema.md`           |
| Schema completo SAP + UDFs             | `references/sap-schema.md`           |
| Patrones de query listos               | `references/query-patterns.md`       |
| Lógica de negocio / handshake          | `references/business-logic.md`       |
| Diferencias, auditoría, pallets, SKUs  | `references/stock-diff.md`           |

---

## KPIs de referencia validados (2026-01 a 2026-04, bodega 04001)

| Métrica                | Valor     |
|:-----------------------|:----------|
| Documentos procesados  | 18,967    |
| Fill Rate general      | 90.38%    |
| Líneas procesadas      | 88,785    |
| P50 tiempo picking     | ~2.55 min |
| P90 tiempo picking     | ~56.97 min|
| P95 tiempo picking     | ~122.61 min|

Fill Rate por mes:
- Ene: 91.31% | Feb: 96.54% | Mar: 89.78% | **Abr: 83.43% ⚠️ caída**
