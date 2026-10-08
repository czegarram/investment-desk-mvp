# Glosario y mapa de compatibilidad

Dos tablas: el glosario de términos (español de la mesa → identificador en inglés) y el mapa
de campos consumidos por reportes aguas abajo (Principio XI de la constitución). Toda
especificación que introduzca un término o un campo nuevo debe actualizar este archivo en el
mismo cambio.

> Estado: **borrador**. Los identificadores son propuestas hasta que la primera
> especificación los fije. Los nombres de campo del sistema actual provienen de la planilla
> y las pantallas entregadas por el responsable funcional el 8 de octubre de 2026. Qué
> consumidor usa cada campo está **[por confirmar]** con middle office; mientras tanto todos
> los campos listados se consideran consumidos.

## 1. Glosario

### Organización y roles

| Término de la mesa | Identificador | Definición breve |
|---|---|---|
| Mesa de inversiones | `desk` | Unidad que gestiona el portafolio de inversiones. |
| Front office | `front_office` | Área que cierra operaciones. |
| Middle office | `middle_office` | Área que configura límites, alertas, calendarios y tipos de cambio. |
| Back office | `back_office` | Área que completa instrumentos, revisa y libera operaciones. |
| Trader / especialista / gestor de portafolio / portfolio manager | `trader` (rol) | Usuario que crea y confirma operaciones. |
| Jefe de mesa / supervisor | `desk_head` (rol) | Usuario que aprueba (*enrich*) operaciones. Uno por mesa. |
| Tramo | `tranche` | Agrupación de las reservas por horizonte: `INV`, `DI`, `LI`, `ON`. |
| Agregado de tramos | `tranche_group` | `INV` o `NO_INV`; usado por los perfiles de límite. |
| Portafolio / profit center / centro de costo | `portfolio` | Subdivisión de un tramo por moneda (`INVEUR`). Código heredado. |
| Sucursal / branch | `branch` | Plaza de la institución. Siempre `BR`. |
| Producto | `product` | Tipo de operación: `SEC`, `DEP`, `DEPORO`, `RPO`. |
| Trade class | `trade_class` | Clase del valor: `GOVT` o `BONOS`. |

### Instrumentos y emisores

| Término de la mesa | Identificador | Definición breve |
|---|---|---|
| Instrumento | `instrument` | Bono registrado en la base interna. |
| Código del instrumento | `instrument_code` | Tres letras y dos dígitos (`AEF14`); correlativo por trade class y moneda. |
| Valor / security | `security` | Instrumento negociable con ISIN. |
| Bono | `bond` | Security con cupón y vencimiento. |
| ISIN | `isin` | Identificador internacional del instrumento. |
| Tipo de ID | `identifier_type` | Siempre `ISIN`. |
| Emisor | `issuer` | Entidad que emite el instrumento. |
| Código de emisor | `issuer_code` | Cinco letras asignadas por la institución (`ACOSS`). |
| Fecha de emisión | `issue_date` | |
| Fecha del primer cupón | `first_coupon_date` | |
| Fecha de vencimiento | `maturity_date` | |
| Cupón | `coupon_rate` | Tasa anual en porcentaje. |
| Tipo de cupón | `coupon_type` | `fixed` o `floating`. |
| Frecuencia de cupón | `coupon_frequency` | `annual` o `semiannual`. |
| Convención de conteo de días | `day_count_convention` | Por ahora solo `ACT/ACT`. |
| Cuponera | `coupon_schedule` | Fechas y montos de cupón generados a partir del instrumento. |
| Ajuste de fecha de cupón | `coupon_date_adjustment` | Regla cuando una fecha de cupón cae en día no hábil. |
| Moneda de pago | `currency` | Código de tres letras. |
| Custodio | `custodian` | Contraparte que custodia los valores del instrumento. |
| Plaza del custodio | `custodian_center` | Plaza de esa contraparte. |
| Descripción | `description` | Emisor + cupón + vencimiento. |
| Clasificación G/L | `gl_classification` | Código contable heredado; significado por confirmar. |

### Contrapartes

| Término de la mesa | Identificador | Definición breve |
|---|---|---|
| Contraparte / broker / dealer | `counterparty` | Entidad con la que se cierra la operación. |
| Código de contraparte | `counterparty_code` | Un registro por entidad y plaza. |
| Plaza / center | `center` | Plaza de la contraparte (`LDN`, `NY`). |
| Categoría | `category` | Heredado; en la práctica `BANK`. |
| Alias | `alias` | Nombre corto descriptivo. |
| Tipo de alias | `alias_type` | Heredado; reemplazado por la autorización por producto. |
| BIC | `bic` | Identificador SWIFT. |
| Banco corresponsal | `is_correspondent` | Broker con contrato firmado con la institución. Requisito para depósitos. |
| Autorizada para el producto | `authorized_products` | Productos en los que la contraparte es elegible. |
| Suspendida | `is_suspended` | Contraparte con contrato pero inhabilitada temporalmente. |

### Operaciones

| Término de la mesa | Identificador | Definición breve |
|---|---|---|
| Operación | `operation` | Transacción cerrada por un trader sobre un instrumento. |
| System ID | `system_id` | Correlativo numérico de ocho dígitos asignado al confirmar; identifica la operación y, en compras, el lote. |
| Correlativo | `sequence_number` | Número secuencial que continúa la serie existente. |
| Compra / venta | `side` (`buy` / `sell`) | Sentido de la operación. |
| Nominal | `nominal_amount` | Valor nominal operado. |
| Precio | `price` | Precio en porcentaje del nominal; hasta 16 decimales. |
| Fecha de operación / trade date | `trade_date` | Fecha del sistema al registrar. |
| Fecha valor / F. Valor / liquidación | `settlement_date` | Fecha en que la operación liquida. |
| Principal | `principal_amount` | `nominal × precio / 100`, 2 decimales. |
| Interés / interés corrido | `accrued_interest` | Interés devengado hasta la fecha valor, 2 decimales. |
| Valor pagado | `settlement_amount` | `principal + interés`, 2 decimales. |
| Memo | `memo` | Texto libre. |
| Tasa / rate | `rate` | Heredado; no se usa en valores. |
| Estado | `status` | `created`, `AE`, `AR`, `R`, `cancelled`. |
| Confirmar | `confirm` (transición) | El trader termina el ingreso → `AE`. |
| Change | `amend` (transición) | El trader corrige una operación en `AE`. |
| Enrich | `enrich` (transición) | El jefe de mesa aprueba → `AR`. |
| Release | `release` (transición) | Back office revisa, elige instrucción y libera → `R`. |
| Anular / cancelar | `cancel` (transición) | Deja sin efecto una operación sin borrarla. |
| Operación padre / hija | `parent_operation` / `child_operation` | División en múltiplos de liquidación. |
| Ticket | `trade_ticket` | Confirmación de la plataforma de negociación que el trader transcribe. |

### Posiciones y lotes

| Término de la mesa | Identificador | Definición breve |
|---|---|---|
| Posición | `position` | Suma de lotes con saldo de un instrumento. |
| Lote | `lot` | Compra confirmada, identificada por su `system_id`, con saldo disponible. |
| Posición vendida | `sold_lot` | Referencia de una venta a un lote. |
| Cantidad (de la posición vendida) | `sold_quantity` | Nominal vendido de ese lote. |
| Saldo disponible | `available_quantity` | Nominal del lote aún no vendido. |

### Cumplimiento

| Término de la mesa | Identificador | Definición breve |
|---|---|---|
| Límite | `limit` | Tope configurado por middle office. |
| Position limit | `position_limit` | Exposición acumulada por nombre. |
| Concentration limit | `concentration_limit` | Exposición acumulada por producto. |
| Tradable names / nombres elegibles | `tradable_names` | Nombres elegibles por producto. |
| Plazo máximo | `max_tenor` | Días calendario desde fecha valor hasta vencimiento. |
| Categoría de riesgo | `risk_category` | Tipo de límite. |
| Perfil de riesgo | `risk_profile_code` | Código del límite (`TOTAL_ACOSS`, `LIM_ACOSS_INVI`). |
| Perfil total / hijo | `parent_profile` | El hijo referencia al total; solo el total bloquea. |
| Límite desde / hasta | `limit_from` / `limit_to` | Rango permitido, en USD nominal. |
| Usado / actual | `limit_current` | Consumo, en USD nominal. |
| Disponible | `limit_available` | `limit_to − limit_current`. |
| Estado del límite | `limit_status` | Vacío u `over`. |
| Tipo de cambio de cierre | `closing_fx_rate` | Tipo de cambio oficial del día anterior por moneda; tabla de entrada. |
| Alerta dura | `hard_warning` | Control que bloquea la transición. |
| Alerta blanda | `soft_warning` | Control que advierte y registra pero permite continuar. |
| Calendario | `calendar` | Conjunto de días hábiles por plaza o moneda. |
| Feriado | `holiday` | Día no hábil de un calendario. |

### Liquidación

| Término de la mesa | Identificador | Definición breve |
|---|---|---|
| Instrucción de liquidación | `settlement_instruction` | Cuenta y plaza con la que liquida una contraparte para una moneda. |
| Archivo SWIFT | `swift_file` | Archivo de texto con mensajes SWIFT generado en el *release*. |
| Apertura / cierre de día | `day_open` / `day_close` | Proceso diario del sistema actual (a analizar). |
| Conciliación | `reconciliation` | Cruce de datos de ejecución con contabilidad, hecho aguas abajo. |

## 2. Mapa de compatibilidad con el sistema actual

Campos del sistema actual que son consumidos por la capa de reportes, el monitoreo de límites
o la conciliación. Deben conservar significado y dominio de valores en este sistema.

### Operación (pantalla de compra/venta de valores)

| Campo en sistema actual | Identificador aquí | Entidad | Consumidor aguas abajo | Notas |
|---|---|---|---|---|
| System ID# | `system_id` | operation | BI, conciliación, auditoría | 8 dígitos. Subnúmero de 3 dígitos por confirmar. |
| Trade Date | `trade_date` | operation | BI, conciliación | Fecha del sistema. |
| Status | `status` | operation | BI | `AE`, `AR`, `R`. |
| Branch | `branch` | operation | BI | Siempre `BR`. |
| Profit Center | `portfolio` | operation | BI, modelos de Excel | `INVUSD` … `INVJPY`. |
| Product | `product` | operation | BI, conciliación | `SEC`. |
| Counterparty | `counterparty` | operation | BI, límites | Código y plaza. |
| Rate | `rate` | operation | — | Vacío en `SEC`. Se conserva. |
| B/S | `side` | operation | BI, conciliación | `B` / `S`. |
| Código | `instrument` | operation | BI, conciliación | Código del instrumento. |
| Trade Class | `trade_class` | operation | BI (diferencial de tasas) | Derivado del instrumento y guardado. |
| Issuer | `issuer_code` | operation | BI, límites | Derivado y guardado. |
| ISIN | `isin` | operation | conciliación | Derivado y guardado. |
| Nominal | `nominal_amount` | operation | BI, límites, conciliación | Base de consumo de límites. |
| Precio | `price` | operation | conciliación | Hasta 16 decimales. |
| F. Valor | `settlement_date` | operation | BI, conciliación | |
| Moneda | `currency` | operation | BI, conciliación | Derivada y guardada. |
| Principal | `principal_amount` | operation | conciliación | 2 decimales. |
| Interest | `accrued_interest` | operation | conciliación | 2 decimales. |
| Valor Pagado | `settlement_amount` | operation | conciliación | 2 decimales. |
| Memo 1 | `memo` | operation | — | |
| Cupón | `coupon_rate` | operation | — | Derivado y guardado. |
| Posición Vendida N | `sold_lot.lot` | sold_lot | conciliación | Solo en ventas; lista dinámica. |
| Cantidad N | `sold_lot.sold_quantity` | sold_lot | conciliación | Solo en ventas. |
| Ajuste de intereses | — | — | — | **No se replica.** Reemplazado por cálculo exacto. |

### Instrumento

| Campo en sistema actual | Identificador aquí | Entidad | Consumidor aguas abajo | Notas |
|---|---|---|---|---|
| Code | `instrument_code` | instrument | BI, conciliación | `AEF14`. |
| ISIN Nbr | `isin` | instrument | conciliación | |
| Tipo de ID | `identifier_type` | instrument | — | Siempre `ISIN`. |
| Issuer | `issuer_code` | instrument | BI, límites | |
| Issue Date | `issue_date` | instrument | conciliación | |
| 1st coupon | `first_coupon_date` | instrument | conciliación | |
| Maturity Date | `maturity_date` | instrument | BI, límites (plazo) | |
| Coupon | `coupon_rate` | instrument | conciliación | |
| Type | `coupon_type` | instrument | — | `Fixed` / `Floating`. |
| Cpn Freq | `coupon_frequency` | instrument | conciliación | `Annual` / `Semi annual`. |
| Day Cnt | `day_count_convention` | instrument | conciliación | `ACT/ACT` por ahora. |
| Payment Instrument | `currency` | instrument | BI, conciliación | Moneda de pago. |
| Custodio | `custodian` | instrument | liquidación | Contraparte. |
| Custodio Plaza | `custodian_center` | instrument | liquidación | |
| Description | `description` | instrument | BI | Generada. |
| G/L Classification | `gl_classification` | instrument | contabilidad | Significado por confirmar. |

### Contraparte

| Campo en sistema actual | Identificador aquí | Entidad | Consumidor aguas abajo | Notas |
|---|---|---|---|---|
| Code | `counterparty_code` | counterparty | BI, límites | |
| Center | `center` | counterparty | liquidación | |
| Category | `category` | counterparty | — | Heredado. |
| Description | `description` | counterparty | BI | |
| Alias | `alias` | counterparty | — | |
| Alias Type | `alias_type` | counterparty | — | Heredado; la autorización real es `authorized_products`. |
| BIC | `bic` | counterparty | liquidación | |

### Límites (vista de middle office)

| Campo en sistema actual | Identificador aquí | Entidad | Consumidor aguas abajo | Notas |
|---|---|---|---|---|
| Risk Category | `risk_category` | limit | BI | |
| Risk Profile Code | `risk_profile_code` | limit | BI | |
| Limit from | `limit_from` | limit | BI | USD nominal. |
| Limit to | `limit_to` | limit | BI | USD nominal. |
| Current | `limit_current` | limit | BI | Calculado. |
| Available | `limit_available` | limit | BI | Calculado. |
| Status | `limit_status` | limit | BI | `over` o vacío. |
