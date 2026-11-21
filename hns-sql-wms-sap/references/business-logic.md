# Lógica de Negocio — Cube_CODA

## División de responsabilidades

### SAP (SBO_CANELLA) — El cerebro financiero
- Reglas maestras de negocio y contabilidad
- Stock contable (`OITW.OnHand`)
- Precios, listas de precios, monedas
- Facturación electrónica (FEL / DTE)
- Cuentas por cobrar/pagar
- Tasas de cambio

### WMS (EWMS_CANELLA) — La ejecución física
- Movimientos físicos de mercadería en almacén
- Gestión de olas y picking
- Control por lote, caducidad, ubicación exacta
- Asignación de tareas a operarios
- Medición de tiempos reales de operación

---

## Flujo completo de un despacho

```
1. SAP: Se crea ORDR (Orden de Venta)
   └─ U_EstadoWMS = '1' → liberado para WMS

2. WMS: Se genera ola de picking
   └─ WMS_WAVE_PICKING.ROUTE_ID creado
   └─ SAP actualiza ORDR.U_OlaPicking = ROUTE_ID
   └─ WMS_ESTADOS_DESPACHOS registra el vínculo

3. WMS: Operarios ejecutan picking
   └─ WMS_PICKING_TASKS registra tiempos reales
   └─ WMS_WAVE_DETAIL registra QTY_SERVED vs QTY_REQUESTED

4. WMS: Transacción enviada a SAP
   └─ WMS_TRANSACTIONS.ERP_SENT = true
   └─ WMS_INTERFACES registra RESULT

5. SAP: Se genera ODLN (Nota de Entrega)
   └─ Actualiza OITW.OnHand (descuenta stock contable)
   └─ BaseType = 17, BaseEntry = ORDR.DocEntry

6. SAP: Se genera OINV (Factura)
   └─ Basada en ODLN
   └─ FEL: genera UUID y firma en @FEL_LOG
```

---

## Handshake: cómo cruzar un documento entre sistemas

**Dirección: SAP → WMS**
```
ORDR.DocNum → WMS_ESTADOS_DESPACHOS.ERP_DOC_NUMBER (como NVARCHAR)
            → WMS_WAVE_PICKING.ROUTE_ID
            → WMS_WAVE_DETAIL.ERP_DOC_NUMBER
```

**Dirección: WMS → SAP**
```
WMS_WAVE_PICKING.ROUTE_ID → WMS_ESTADOS_DESPACHOS.ROUTE_ID
                           → WMS_INTERFACES.ROUTE_ID
                           → ORDR.U_OlaPicking
```

**⚠️ Casting obligatorio:**
```sql
-- ERP_DOC_NUMBER es NVARCHAR, DocNum es INT
ON T4.ERP_DOC_NUMBER = CAST(T0.DocNum AS NVARCHAR)
```

---

## Multialmacén y multizona

**SAP** gestiona a nivel de `WhsCode`:
- PM122 = bodega WMS principal
- BV001-002 = bodegas virtuales
- DB001-DB240+ = distribuidores directos
- CY001-CY006 = tiendas
- TM001-TM022 = talleres móviles

**WMS** gestiona a nivel granular:
- `COD_WAREHOUSE = '04001'` (equivale PM122)
- `ID_ZONE` / `ESTIMATE_ZONE` = zonas dentro del almacén
- `LOCATION_SOURCE` / `LOCATION_TARGET` = coordenadas exactas
- `LICENSE_PLATE` = licencia de pallet específico

---

## Brechas de inventario SAP vs WMS

Diferencias entre `OITW.OnHand` y `SUM(WMS_STOCK_X_LOCATION.QUANTITY)` se detectan en:
- `WMS_INTERFACES_LOG_DET` → errores de sincronización específicos
- `WMS_INTERFACES_ERROR` → errores con mensaje

Causas comunes:
1. Interfaz enviada pero SAP devolvió error (RESULT ≠ 'OK')
2. Movimiento en WMS sin `READY_SENT_ERP = 1`
3. Recepciones pendientes de confirmar en SAP
4. Devoluciones en WMS sin espejo en SAP

---

## Tipos de ERP_DOC_TYPE y su contexto

| Tipo | Descripción              | Frecuencia | Documento SAP |
|:-----|:-------------------------|:-----------|:--------------|
| OV   | Orden de Venta           | Alta       | ORDR          |
| OF   | Oferta de Venta          | Media      | OQUT          |
| TR5  | Traslado entre bodegas   | Media      | OWTR          |
| E    | Factura directa (PM122)  | < 0.01%    | OINV          |
| TR3  | Kitting interno WMS      | Interna    | Sin documento |
| TRS  | Movimiento interno WMS   | Interna    | Sin documento |

> **Reportería comercial limpia:** siempre `IN ('OV', 'OF', 'TR5', 'E')` → 18,967 documentos ene-abr 2026.

---

## Cube_CODA — Propósito

Cruces estratégicos entre ambos mundos:
- **Facturado en SAP vs despachado en WMS** → diferencias de cierre
- **Fill Rate** → QTY_SERVED / QTY_REQUESTED por período
- **Productividad por operario** → tiempos de WMS_PICKING_TASKS
- **Sincronización** → RESULT en WMS_INTERFACES vs estado en SAP
- **Stock** → OITW.OnHand vs WMS_STOCK_X_LOCATION (brechas)
