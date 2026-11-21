# Schema WMS — EWMS_CANELLA

> Servidor: 128.1.200.58 | Última verificación: 2026-05-07

---

## WMS_WAVE_PICKING — Cabecera de ola (1 fila por ruta)

| Columna                   | Tipo          | Notas                                              |
|:--------------------------|:--------------|:---------------------------------------------------|
| ROUTE_ID                  | varchar(20)   | **PK.** Prefijos: PTB, WP-, KT-                    |
| DATE_ROUTE                | smalldatetime | Fecha de la ruta                                   |
| STATUS                    | nvarchar      | '9' = cancelado. Siempre filtrar `<> '9'`          |
| WAVE_TYPE                 | varchar       | Tipo de ola                                        |
| DATETIME_CREATED          | datetime      | Creación del registro                              |
| DATETIME_ASSIGNED         | datetime      | Asignación al operario                             |
| DATETIME_STARTED          | datetime      | ⚠️ Ciclo de vida del sistema (~26 seg mediana)     |
| DATETIME_FINISHED         | datetime      | ⚠️ NO usar para KPIs de tiempo real               |
| DATETIME_CHECKED          | datetime      | Verificación                                       |
| DATETIME_DISPATCHED       | datetime      | Despacho                                           |
| WAREHOUSE_ID              | nvarchar      | '04001' = PM122 en SAP                             |
| ASSIGNED_TO               | nvarchar      | Usuario principal asignado                         |
| HELPER1 … HELPER10        | nvarchar      | Hasta 10 operarios adicionales en paralelo         |
| TRUCK_TYPE                | varchar       | Tipo de camión                                     |
| TRUCK_ID                  | varchar       | ID del camión                                      |
| PILOT_ID                  | varchar       | ID del piloto/transportista                        |
| CARRIER_ID                | varchar       | ID del transportista                               |
| DELIVERY_ID               | varchar       | ID de entrega                                      |
| TRIP_ID                   | varchar       | ID del viaje                                       |
| WAVE_ROUTE                | varchar       | Ruta de la ola                                     |
| LOCATION_TARGET           | nvarchar      | Ubicación destino                                  |
| TARGET_WAREHOUSE          | nvarchar      | Bodega destino (relevante en traslados)            |
| PRIORITY                  | varchar       | Prioridad de la ruta                               |
| CREATED_BY                | nvarchar      | Usuario creador                                    |
| USER_AUTHORIZE            | nvarchar      | Usuario autorizador                                |
| ROUTE_REF                 | varchar       | Referencia de ruta                                 |
| DISPATCH_COMMENTS         | nvarchar      | Comentarios de despacho                            |
| TRANSPORT_COMMENTS        | nvarchar      | Comentarios de transporte                          |

---

## WMS_WAVE_DETAIL — Líneas de ola (N filas por ruta)

| Columna            | Tipo          | Notas                                              |
|:-------------------|:--------------|:---------------------------------------------------|
| ROUTE_ID           | nvarchar(20)  | **FK → WMS_WAVE_PICKING.ROUTE_ID**                 |
| DATE_ROUTE         | smalldatetime |                                                    |
| ERP_COMPANY        | nvarchar(20)  | Empresa ERP                                        |
| ERP_DOC_TYPE       | nvarchar(10)  | OV, OF, TR5, E, TR3, TRS                           |
| ERP_DOC_NUMBER     | nvarchar(20)  | = DocNum en SAP                                    |
| ERP_LINE_NUMBER    | int           | Número de línea en documento SAP                   |
| COD_PRODUCT        | nvarchar(50)  | = ItemCode en SAP                                  |
| COD_CUSTOMER       | nvarchar(20)  | = CardCode en SAP                                  |
| QTY_REQUESTED      | decimal       | Cantidad solicitada                                |
| QTY_SERVED         | decimal       | Cantidad servida (real)                            |
| QTY_REJECTED       | decimal       | Cantidad rechazada                                 |
| QTY_AUDITED        | decimal       | Cantidad auditada                                  |
| STATUS             | nvarchar      | '9' = cancelado                                    |
| LOCATION_SOURCE    | nvarchar      | Ubicación origen en almacén                        |
| LOCATION_TARGET    | nvarchar      | Ubicación destino                                  |
| TARGET_TOTE        | nvarchar      | Contenedor/tote destino                            |
| ESTIMATE_ZONE      | nvarchar      | Zona estimada                                      |
| ID_ZONE            | nvarchar      | ID de zona real                                    |
| COD_WAREHOUSE      | nvarchar      | Bodega WMS                                         |
| COD_USER           | nvarchar      | Operario que ejecutó                               |
| LOT_NUMBER         | nvarchar      | Número de lote                                     |
| EXPIRY_DATE        | datetime      | Fecha de caducidad                                 |
| DATE_CREATED       | datetime      | Creación                                           |
| DATE_LAST_UPDATED  | datetime      | Última actualización                               |

---

## WMS_PICKING_TASKS — Tareas reales de picking

> ✅ Esta es la tabla correcta para KPIs de tiempo y productividad.

| Columna           | Tipo      | Notas                                                  |
|:------------------|:----------|:-------------------------------------------------------|
| ERP_DOCUMENT      | varchar   | **= ROUTE_ID** (no es ERP_DOC_NUMBER)                  |
| COD_USER          | nvarchar  | Operario                                               |
| DATETIME_START    | datetime  | Inicio real de la tarea                                |
| DATETIME_END      | datetime  | Fin real de la tarea                                   |
| QTY_SERVED        | decimal   | Cantidad pickeada en esta tarea                        |
| QTY_REQUESTED     | decimal   | Cantidad solicitada                                    |
| LOCATION_SOURCE   | nvarchar  | Ubicación desde donde se pickeó                        |
| COD_PRODUCT       | nvarchar  | Producto                                               |
| STATUS            | nvarchar  | Estado de la tarea                                     |

**Tiempo real con operarios paralelos:**
```sql
SELECT ERP_DOCUMENT AS ROUTE_ID,
       DATEDIFF(MINUTE, MIN(DATETIME_START), MAX(DATETIME_END)) AS Minutos_Reales
FROM WMS_PICKING_TASKS
WHERE DATETIME_START IS NOT NULL
  AND DATETIME_END IS NOT NULL
  AND DATETIME_END > DATETIME_START
GROUP BY ERP_DOCUMENT
```

---

## WMS_PICKING_DETAIL — Detalle de picking

| Columna         | Notas                        |
|:----------------|:-----------------------------|
| ROUTE_ID        | FK → WMS_WAVE_PICKING        |
| COD_PRODUCT     | Producto                     |
| QTY_SERVED      | Cantidad                     |
| LOCATION_SOURCE | Ubicación                    |
| COD_USER        | Operario                     |
| LOT_NUMBER      | Lote                         |
| DATETIME_START  | Inicio                       |
| DATETIME_END    | Fin                          |

---

## WMS_STOCK_X_LOCATION — Stock actual por ubicación

| Columna       | Notas                                  |
|:--------------|:---------------------------------------|
| COD_PRODUCT   | = ItemCode en SAP                      |
| COD_WAREHOUSE | '04001' = PM122                        |
| LOCATION      | Coordenada exacta en almacén           |
| QUANTITY      | Cantidad en esa ubicación              |
| LOT_NUMBER    | Lote                                   |
| EXPIRY_DATE   | Vencimiento                            |
| LICENSE_PLATE | Licencia de pallet                     |

> Para stock total del artículo: `SUM(QUANTITY)` agrupando por `COD_PRODUCT`.

---

## WMS_TRANSACTIONS — Movimientos transaccionales

| Columna           | Notas                                     |
|:------------------|:------------------------------------------|
| ROUTE_ID          | FK → WMS_WAVE_PICKING                     |
| ERP_SENT          | Flag: ¿ya se envió a SAP?                 |
| READY_SENT_ERP    | Flag: ¿listo para enviar a SAP?           |
| ERP_PURCHASE_ORDER| PO de SAP asociada                        |
| ERP_DOCUMENT      | Documento ERP                             |
| TRANSACTION_TYPE  | Tipo de movimiento                        |
| DATE_CREATED      | Fecha de la transacción                   |

---

## WMS_INTERFACES / WMS_INTERFACES_HEAD / WMS_INTERFACES_ERROR

Trío de tablas para monitoreo de la sincronización WMS → SAP.

**WMS_INTERFACES** (estado general por ruta):
- `ROUTE_ID` → FK
- `RESULT` → 'OK', 'ERROR', null (pendiente)
- `DATE_CREATED`, `DATE_UPDATED`

**WMS_INTERFACES_HEAD** (detalle del documento):
- `ROUTE_ID`, `ERP_DOC_TYPE`, `ERP_DOC_NUMBER`, `ERP_COMPANY`

**WMS_INTERFACES_ERROR** (errores específicos):
- `ROUTE_ID`, `ERROR_MESSAGE`, `DATE_ERROR`

**WMS_INTERFACES_LOG / WMS_INTERFACES_LOG_DET** (log completo):
- Para diagnóstico profundo de sincronización

---

## WMS_RELOCATION_TASKS — Reubicaciones internas

| Columna           | Notas                         |
|:------------------|:------------------------------|
| TASK_ID           | PK                            |
| COD_PRODUCT       | Producto a reubicar           |
| LOCATION_SOURCE   | Ubicación origen              |
| LOCATION_TARGET   | Ubicación destino             |
| QUANTITY          | Cantidad                      |
| COD_USER          | Operario asignado             |
| DATETIME_START    | Inicio                        |
| DATETIME_END      | Fin                           |
| STATUS            | Estado                        |

---

## WMS_ARRIVAL_RECEPTION — Recepciones

| Columna              | Notas                              |
|:---------------------|:-----------------------------------|
| COD_WAREHOUSE        | '04001'                            |
| DATE_RECEPTION       | Fecha de recepción                 |
| COD_PRODUCT          | Producto recibido                  |
| LOT_NUMBER           | Lote                               |
| QUANTITY_RECEIVED    | Cantidad recibida                  |
| QUANTITY_PENDING     | Cantidad pendiente de procesar     |
| ERP_PURCHASE_ORDER   | Orden de compra en SAP             |
| STATUS               | Estado de la recepción             |
| COD_USER             | Operario receptor                  |

---

## WMS_RETURNS_RECEPTION — Devoluciones

| Columna          | Notas                     |
|:-----------------|:--------------------------|
| COD_PRODUCT      | Producto devuelto         |
| COD_CUSTOMER     | Cliente que devuelve      |
| QUANTITY         | Cantidad                  |
| REASON           | Motivo de devolución      |
| DATE_RECEPTION   | Fecha                     |
| STATUS           | Estado                    |

---

## Catálogos

### ERP_DOC_TYPE
| Valor | SAP      | Reportería comercial |
|:------|:---------|:---------------------|
| OV    | ORDR     | ✅ Incluir           |
| OF    | OQUT     | ✅ Incluir           |
| TR5   | OWTR     | ✅ Incluir           |
| E     | OINV     | ✅ Incluir (raro)    |
| TR3   | Kitting  | ❌ Excluir           |
| TRS   | Interno  | ❌ Excluir           |

### STATUS en WAVE_PICKING y WAVE_DETAIL
| Valor | Significado |
|:------|:------------|
| 9     | Cancelado / Anulado — siempre excluir |
| Otros | Activo / procesado |

### ROUTE_ID — Prefijos
| Prefijo | Tipo                   | Excluir |
|:--------|:-----------------------|:--------|
| PTB     | Picking general        | No      |
| WP-     | Wave picking           | No      |
| KT-     | Kitting interno        | Sí      |

---

## DEF_LOCATIONS — Catálogo de ubicaciones físicas

| Campo          | Tipo        | Descripción                              |
|:---------------|:------------|:-----------------------------------------|
| LOCATION       | varchar     | Código de la ubicación física            |
| COD_WAREHOUSE  | varchar     | Bodega. '04001' = PM122                  |
| LOCATION_TYPE  | varchar     | Tipo numérico (ver catálogo)             |

### LOCATION_TYPE — valores relevantes

| Tipo | Clasificación              | En stock disponible |
|:-----|:---------------------------|:--------------------|
| 1    | Almacenamiento (004A)       | ✅ Sí               |
| 2    | Picking face               | ✅ Sí               |
| 3    | Recepción                  | ❌ No (qty_rec)     |
| 4    | Staging salida             | ❌ No (qty_otros)   |
| 5    | Consolidación              | ❌ No (qty_otros)   |
| 8    | MOR / rutas activas        | ❌ No (qty_otros)   |
| 9    | Rampa / despacho           | ❌ No (qty_otros)   |
| 33   | Empaque especial           | ❌ No (qty_otros)   |

**Filtro stock disponible:**
```sql
AND l.LOCATION_TYPE NOT IN ('8','9','33','4','5','3')
AND l.LOCATION NOT IN ('04001PM122','004DESGLOSE01','04001SOBRANTE','04FALTANTE01')
```

---

## WMS_WAVE_PALLETS — Pallets por ola

| Columna       | Notas                              |
|:--------------|:-----------------------------------|
| ROUTE_ID      | FK → WMS_WAVE_PICKING.ROUTE_ID     |
| WAVE_PALLET_ID| PK del pallet (ej: 'LPS0006-49')   |
| STATUS        | Estado del pallet                  |
| DATE_CREATED  | Fecha de creación                  |

## WMS_WAVE_PALLETS_DETAIL — Productos por pallet

| Columna             | Notas                          |
|:--------------------|:-------------------------------|
| WAVE_PALLET_ID      | FK → WMS_WAVE_PALLETS          |
| ROUTE_ID            | FK → WMS_WAVE_PICKING          |
| COD_PRODUCT         | Producto                       |
| QTY                 | Cantidad asignada              |
| QTY_CHECKED         | Cantidad verificada            |
| STATUS              | Estado de la línea             |
| SOURCE_LOCATION     | Ubicación origen               |
| SOURCE_TOTE         | Tote origen                    |
| TARGET_TOTE         | Tote destino                   |
| CONSOLIDATION_LOCATION | Ubicación de consolidación  |
| COD_USER            | Operario                       |
| COD_CUSTOMER        | Cliente                        |
| LOT_NUMBER          | Lote                           |
| DATE_CREATED        | Creación                       |
| DATE_LAST_UPDATED   | Última actualización           |
