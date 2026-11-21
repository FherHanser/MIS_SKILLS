# Schema SAP — SBO_CANELLA

> Servidor: 128.1.200.167 | Última verificación: 2026-05-08

---

## ORDR — Órdenes de Venta (cabecera)

| Campo             | Tipo          | Notas                                           |
|:------------------|:--------------|:------------------------------------------------|
| DocEntry          | int           | PK interna                                      |
| DocNum            | int           | **Número visible = ERP_DOC_NUMBER en WMS**      |
| DocStatus         | char(1)       | 'O' = Abierta, 'C' = Cerrada                    |
| CANCELED          | char(1)       | 'Y' = Cancelada. Filtrar `= 'N'`                |
| DocDate           | datetime      | Fecha de contabilización                        |
| CardCode          | nvarchar(15)  | Código del cliente (= COD_CUSTOMER en WMS)      |
| CardName          | nvarchar(100) | Nombre del cliente                              |
| SlpCode           | int           | FK → OSLP.SlpCode (vendedor)                    |
| UserSign          | smallint      | FK → OUSR.USERID (creador)                      |
| DocTotal          | numeric       | Total del documento                             |

**UDFs de integración WMS en ORDR:**

| Campo                 | Tipo          | Descripción                                      |
|:----------------------|:--------------|:-------------------------------------------------|
| U_EstadoWMS           | nvarchar(50)  | '1' = liberado para WMS                          |
| U_OlaPicking          | nvarchar(50)  | ROUTE_ID del WMS asignado                        |
| U_RutaDespacho        | nvarchar(50)  | ROUTE_ID de despacho                             |
| U_Secuencia           | nvarchar(50)  | Secuencia de procesamiento                       |
| U_ConsolidadoEWMS     | nvarchar(50)  | ID de consolidación en EWMS                      |
| U_TableroDistribucion | nvarchar(10)  | Control tablero de distribución                  |
| U_StatusDist          | nvarchar(20)  | Estado en tablero de distribución                |
| U_TrackingDist        | nvarchar(30)  | Tracking de distribución                         |
| U_EntregaADomicilio   | nvarchar(2)   | ¿Entrega a domicilio?                            |
| U_PendienteFacturar   | nvarchar(2)   | ¿Pendiente de facturar?                          |

---

## RDR1 — Líneas de Órdenes de Venta

| Campo       | Notas                                        |
|:------------|:---------------------------------------------|
| DocEntry    | FK → ORDR.DocEntry                           |
| LineNum     | Número de línea                              |
| ItemCode    | Código de artículo (= COD_PRODUCT en WMS)    |
| WhsCode     | Bodega. 'PM122' = WMS bodega 04001           |
| Quantity    | Cantidad solicitada                          |
| LineStatus  | 'O' = Abierta, 'C' = Cerrada                 |
| LineTotal   | Monto de la línea                            |

---

## ODLN / DLN1 — Notas de Entrega

Paso intermedio entre OV y Factura. Confirma salida física.

**Flujo:** `ORDR → WMS picking → ODLN → OINV`

La ODLN actualiza `OITW.OnHand` en SAP.

**ODLN (cabecera):**
| Campo            | Notas                          |
|:-----------------|:-------------------------------|
| DocEntry         | PK interna                     |
| DocNum           | Número visible                 |
| DocDate          | Fecha                          |
| CardCode         | Cliente                        |
| CardName         | Nombre cliente                 |
| DocStatus        | 'O' / 'C'                      |

**DLN1 (líneas):**
| Campo      | Notas                                    |
|:-----------|:-----------------------------------------|
| DocEntry   | FK → ODLN.DocEntry                       |
| ItemCode   | Artículo                                 |
| WhsCode    | Bodega de despacho                       |
| Quantity   | Cantidad despachada                      |
| BaseType   | 17 = ORDR (documento origen)             |
| BaseEntry  | DocEntry del documento origen            |

---

## OINV / INV1 — Facturas de Deudores

**OINV (cabecera) — UDFs relevantes:**
| Campo                 | Tipo         | Descripción                  |
|:----------------------|:-------------|:-----------------------------|
| DocEntry              | int          | PK interna                   |
| DocNum                | int          | Número visible               |
| DocDate               | datetime     | Fecha                        |
| CardCode              | nvarchar     | Cliente                      |
| U_EstadoWMS           | nvarchar(50) | Estado WMS de la factura     |
| U_TableroDistribucion | nvarchar(10) | Control tablero distribución |
| U_StatusDist          | nvarchar(20) | Estado distribución          |

---

## OQUT / QUT1 — Ofertas de Venta

Misma estructura base que ORDR / RDR1.
En WMS: `ERP_DOC_TYPE = 'OF'`

---

## OWTQ / WTQ1 — Solicitudes de Traslado

Solicita movimiento entre bodegas. 20,216 filas.
- `WhsCode` = bodega origen
- `ToWhsCod` = bodega destino

---

## OWTR / WTR1 — Transferencias de Inventario

Confirmación del traslado cuando WMS notifica que completó el movimiento.
En WMS: `ERP_DOC_TYPE = 'TR5'`

| WTR1 Campo | Notas               |
|:-----------|:--------------------|
| DocEntry   | FK → OWTR.DocEntry  |
| ItemCode   | Artículo            |
| WhsCode    | Bodega origen       |
| ToWhsCod   | Bodega destino      |
| Quantity   | Cantidad trasladada |

---

## OITW — Stock por almacén

> ⚠️ 19,273,702 filas. Siempre filtrar por `WhsCode = 'PM122'` primero.

| Campo      | Notas                                           |
|:-----------|:------------------------------------------------|
| ItemCode   | Código del artículo                             |
| WhsCode    | 'PM122' = bodega WMS principal                  |
| OnHand     | Stock físico contabilizado en SAP               |
| IsCommited | Stock comprometido por OVs abiertas             |
| OnOrder    | Stock en tránsito (órdenes de compra abiertas)  |

**Stock disponible real:**
```sql
OnHand - IsCommited  -- stock libre para nuevas OVs
```

---

## OITM — Maestro de Artículos

| Campo      | Notas                                 |
|:-----------|:--------------------------------------|
| ItemCode   | PK (= COD_PRODUCT en WMS)             |
| ItemName   | Descripción del artículo              |
| FrgnName   | Nombre alterno                        |
| ItmsGrpCod | Grupo de artículos                    |
| CodeBars   | Código de barras                      |
| SellItem   | 'Y' = ítem de venta                   |
| PrchseItem | 'Y' = ítem de compra                  |

---

## OCRD — Socios de Negocio

| Campo      | Notas                                       |
|:-----------|:--------------------------------------------|
| CardCode   | PK (= COD_CUSTOMER en WMS)                  |
| CardName   | Nombre                                      |
| CardType   | 'C' = Cliente, 'S' = Proveedor, 'L' = Lead  |
| CreditLine | Límite de crédito                           |
| Balance    | Saldo actual                                |

---

## OSLP — Vendedores

| Campo   | Notas                              |
|:--------|:-----------------------------------|
| SlpCode | PK → FK desde ORDR.SlpCode         |
| SlpName | Nombre del vendedor                |

---

## OUSR — Usuarios SAP

| Campo   | Notas                              |
|:--------|:-----------------------------------|
| USERID  | PK → FK desde ORDR.UserSign        |
| U_Name  | Nombre de usuario en sistema       |

---

## OWHS — Catálogo de Bodegas (1,413 registradas)

Tipos relevantes:
| Patrón      | Tipo                           |
|:------------|:-------------------------------|
| PM122       | Bodega principal WMS           |
| 01          | Almacén general                |
| BV001-002   | Bodegas virtuales              |
| DB001-DB240+| Distribuidores directos        |
| CY001-CY006 | Tiendas                        |
| TM001-TM022 | Talleres móviles               |

---

## WMS_ESTADOS_DESPACHOS — Tabla custom en SBO_CANELLA

Puente bidireccional SAP ↔ WMS.

| Campo          | Notas                                          |
|:---------------|:-----------------------------------------------|
| ERP_DOC_NUMBER | = ORDR.DocNum                                  |
| ERP_DOC_TYPE   | 'OV', 'OF', 'TR5', 'E'                         |
| ROUTE_ID       | = WMS_WAVE_PICKING.ROUTE_ID                    |

---

## @FEL_LOG — Facturación Electrónica

| Campo | Notas                          |
|:------|:-------------------------------|
| UUID  | Identificador único del DTE    |
| Firma | Firma electrónica              |
| Tipo  | Tipo de Documento Tributario   |

---

## JDT1 — Asientos Contables

Tabla de movimientos financieros derivados de la operación. No usar en reportería operativa.

---

## ITM1 / AIT1 — Listas de Precios

Para consultas de precios por artículo y moneda.
