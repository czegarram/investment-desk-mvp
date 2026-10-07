# Glosario y mapa de compatibilidad

Dos tablas: el glosario de términos (español de la mesa → identificador en inglés) y el mapa
de campos consumidos por reportes aguas abajo (Principio XI de la constitución). Toda
especificación que introduzca un término o un campo nuevo debe actualizar este archivo en el
mismo cambio.

> Estado: **borrador**. Los identificadores son propuestas hasta que la primera
> especificación los fije. Los campos del sistema actual se completarán cuando se reciba la
> documentación de campos.

## 1. Glosario

| Término de la mesa | Identificador | Definición breve |
|---|---|---|
| Mesa de inversiones | `desk` | Unidad que gestiona el portafolio de inversiones. |
| Front office | `front_office` | Área que cierra operaciones. |
| Middle office | `middle_office` | Área que configura límites, alertas y calendarios. |
| Back office | `back_office` | Área que crea instrumentos, valida y liquida operaciones. |
| Trader / especialista / gestor de portafolio | `trader` (rol) | Usuario que crea y confirma operaciones. |
| Jefe de mesa / supervisor | `desk_head` (rol) | Usuario que aprueba (*enrich*) operaciones. Uno por mesa. |
| Operación | `operation` | Transacción cerrada por un trader sobre un instrumento. |
| Correlativo | `sequence_number` | Número secuencial que identifica operaciones, instrumentos u otras entidades; continúa la serie existente. |
| Instrumento | `instrument` | Activo financiero registrado (bono, papel comercial, etc.). |
| Tipo de instrumento | `instrument_type` | Clasificación del instrumento (security, depósito, FX spot, reporto, futuro). |
| Valor / security | `security` | Instrumento negociable con ISIN: bono o papel comercial. |
| Bono | `bond` | Security con cupón y vencimiento. |
| Papel comercial | `commercial_paper` | Security de corto plazo, equivalente a un depósito con ISIN. |
| ISIN | `isin` | Identificador internacional del instrumento. |
| Emisor | `issuer` | Entidad que emite el instrumento. |
| Cupón | `coupon_rate` | Tasa anual que paga el instrumento. |
| Frecuencia de cupón | `coupon_frequency` | Periodicidad de pago del cupón (anual, semestral, …). |
| Convención de conteo de días | `day_count_convention` | Base de cálculo de intereses (por ejemplo 30/360, ACT/360). |
| Cuponera / ajuste de fecha de cupón | `coupon_date_adjustment` | Regla aplicada cuando una fecha de cupón cae en día no hábil (siguiente, anterior, …). |
| Contraparte | `counterparty` | Entidad con la que se cierra la operación. |
| Alias de contraparte | `counterparty_alias` | Nombre corto usado en el sistema actual para buscar la contraparte. |
| Producto | `product` | Tipo de operación (compra/venta de valores, depósito, reporto, …). |
| Portafolio / tramo de inversión | `portfolio` | Cartera a la que pertenece la operación. |
| Compra / venta | `side` (`buy` / `sell`) | Sentido de la operación. |
| Monto / nominal | `nominal_amount` | Valor nominal operado. |
| Precio | `price` | Precio de la operación. |
| Tasa | `rate` | Tasa de la operación (rendimiento, tasa de reporto, …). |
| Fecha de operación | `trade_date` | Fecha en que se cierra la operación. |
| Fecha de inicio / liquidación | `settlement_date` | Fecha en que la operación liquida. |
| Fecha de fin / vencimiento | `maturity_date` | Fecha de vencimiento; calculada cuando corresponde. |
| Memo | `memo` | Texto libre de la operación. |
| Posición | `position` | Tenencia vigente de un instrumento: lo comprado no vencido ni vendido. |
| Lote | `lot` | Fracción identificable de una posición, con su propio código. |
| Límite | `limit` | Tope configurado por middle office (por emisor, instrumento, contraparte). |
| Usado / disponible | `limit_used` / `limit_available` | Consumo y saldo de un límite. |
| Alerta dura | `hard_warning` | Control que bloquea la transición. |
| Alerta blanda | `soft_warning` | Control que advierte y registra pero permite continuar. |
| Plazo máximo | `max_tenor` | Plazo máximo permitido por tipo de operación. |
| Calendario | `calendar` | Conjunto de días hábiles por plaza o moneda. |
| Feriado | `holiday` | Día no hábil de un calendario. |
| Confirmar | `confirm` (transición) | El trader termina el ingreso. |
| Change | `amend` (transición) | El trader modifica una operación confirmada aún no aprobada. |
| Enrich | `enrich` (transición) | El jefe de mesa aprueba la operación. |
| Validar | `validate` (transición) | Back office revisa y define instrucciones de liquidación. |
| Release | `release` (transición) | Back office emite las instrucciones de liquidación. |
| Anular / cancelar | `cancel` (transición) | Deja sin efecto una operación sin borrarla. |
| Operación padre / hija | `parent_operation` / `child_operation` | División de una operación en múltiplos de liquidación. |
| Instrucción de liquidación | `settlement_instruction` | Cuenta y plaza con la que liquida una contraparte para una moneda. |
| Sucursal | `branch` | Plaza de la contraparte (o de la institución) usada en la instrucción. |
| Archivo SWIFT | `swift_file` | Archivo de texto con mensajes SWIFT generado en el *release*. |
| Custodio | `custodian` | Entidad que custodia los valores. |
| Apertura / cierre de día | `day_open` / `day_close` | Proceso diario del sistema actual (a analizar). |
| Conciliación | `reconciliation` | Cruce de datos de ejecución con contabilidad, hecho aguas abajo. |

## 2. Mapa de compatibilidad con el sistema actual

Campos del sistema actual que son consumidos por la capa de reportes, el monitoreo de límites
o la conciliación. Deben conservar significado y dominio de valores en este sistema.

| Campo en sistema actual | Identificador aquí | Entidad | Consumidor aguas abajo | Notas |
|---|---|---|---|---|
| _(pendiente de la documentación de campos)_ | | | | |
