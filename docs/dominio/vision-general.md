# Visión general del dominio: la mesa de inversiones

Este documento describe cómo opera la mesa de inversiones que el sistema debe soportar.
Es la base de todas las especificaciones por producto. Fue elaborado a partir de la
descripción escrita del responsable funcional, de su planilla de campos y de las sesiones de
diseño del 6 y 8 de octubre de 2026.

> Estado: **borrador para revisión**. Pendiente de validación por el responsable funcional.
> Las dudas abiertas se marcan con **[Por confirmar]**.

## 1. Portafolios y tramos

Las reservas se administran en **tramos** de inversión. La institución usa cuatro:

| Tramo | Código | Horizonte | Mesa responsable | Producto principal |
|---|---|---|---|---|
| Tramo de inversión | `INV` | Largo plazo | Mesa de inversiones (largo plazo) | Valores (bonos) |
| Tramo de disponibilidad | `DI` | Corto plazo | Mesa de money market y FX | Depósitos |
| Tramo de liquidez | `LI` | Corto plazo | Mesa de money market y FX | Depósitos |
| Tramo overnight | `ON` | Un día | Mesa de money market y FX | Depósitos |

Para los agregados de riesgo, `INV` se contrapone a **NO INV** (la suma de `DI`, `LI` y `ON`).

El tramo de inversión se subdivide por moneda. Cada subdivisión es un **portafolio**, que el
sistema actual llama *Profit Center*, con un código compuesto por el tramo y la moneda:
`INVUSD`, `INVCAD`, `INVGBP`, `INVEUR`, `INVAUD`, `INVJPY`. Estos códigos aparecen en los
modelos y reportes existentes y **no se cambian**.

La moneda de reporte es el dólar estadounidense. Los montos en otras monedas se convierten con
un **tipo de cambio oficial de cierre del día anterior** por moneda. Ese tipo de cambio es un
dato de entrada mantenido en una tabla del sistema, no un cálculo ni una descarga automática.
**[Por confirmar]** la fuente externa que lo alimenta; por ahora se carga manualmente.

## 2. Áreas y roles

Toda mesa de inversiones se organiza en tres áreas. Cada una tiene roles con permisos
distintos y ninguna puede hacer el trabajo de la otra.

| Área | Rol en el sistema | Qué hace |
|---|---|---|
| Front office | **Trader** (especialista, gestor de portafolio, *portfolio manager*) | Crea y confirma operaciones. Crea el borrador de un instrumento nuevo. Es el responsable de la operación. |
| Front office | **Jefe de mesa** (supervisor, único por mesa) | Compara la operación con el ticket de la plataforma de negociación y la aprueba mediante el *enrich*. |
| Middle office | **Middle office** | Configura límites, nombres elegibles, plazos máximos, calendarios y tipos de cambio; define alertas; concilia aguas abajo. No opera. |
| Back office | **Back office** | Completa y confirma instrumentos. Revisa la operación, elige la instrucción de liquidación y hace el *release*, que genera el mensaje SWIFT al custodio. |

## 3. Módulos del sistema

Los módulos reflejan las áreas. Cada especificación pertenece a un solo módulo.

1. **Ejecución** (front): portafolios, productos, instrumentos, contrapartes, ingreso de
   operaciones, confirmación, *enrich*, posiciones y lotes.
2. **Cumplimiento** (middle): límites, nombres elegibles por producto, plazos máximos,
   calendarios, tipos de cambio, alertas duras y blandas evaluadas en cada transición.
3. **Liquidación** (back): revisión, instrucciones de liquidación por contraparte, moneda y
   plaza, *release* y generación del archivo de mensajes SWIFT.

## 4. Catálogos

### 4.1 Productos y clases

El **producto** define el tipo de operación y filtra el resto de los catálogos. Códigos del
sistema actual:

| Código | Producto | Tramo | Observación |
|---|---|---|---|
| `SEC` | Valores (*securities*) | INV | Primer producto del sistema. |
| `DEPORO` | Depósitos de oro | INV | Fase posterior. |
| `RPO` | Reporto (*repo*) | INV | Parecido a una venta de valor, pero con instrucción SWIFT distinta. Fase posterior. |
| `DEP` | Depósito | DI, LI, ON | Segundo producto del sistema. |

Dentro de `SEC`, el **trade class** separa los valores en dos clases. Middle office lo usa
para medir el diferencial de tasa entre ambas:

| Trade class | Qué es |
|---|---|
| `GOVT` | Bonos de gobierno. |
| `BONOS` | Bonos de agencias y entidades supranacionales (SSA), con respaldo implícito o explícito de un gobierno. |

La **sucursal** (*branch*) es siempre `BR`: la institución tiene una sola. El campo existe por
compatibilidad.

### 4.2 Emisores

Un **emisor** es la entidad que emite la deuda. La lista de emisores elegibles pertenece a la
institución y la define el área de políticas de inversión; cada emisor tiene un **código de
cinco letras** ya asignado. Por emisor se configuran un plazo máximo y un monto máximo (ver
sección 8). No confundir con la contraparte: el emisor vende su deuda a través de brokers.

### 4.3 Contrapartes

La **contraparte** es el broker o dealer autorizado con el que se cierra la operación. Un
mismo bono lo pueden vender varios brokers, que el trader hace competir. Campos del sistema
actual:

| Campo | Ejemplo | Uso |
|---|---|---|
| Code | `BANKA` | Código de la contraparte. Una misma entidad puede tener un registro por plaza. |
| Center | `LDN`, `NY` | Plaza. |
| Category | `BANK` | Categoría; en la práctica todas son bancos. Campo heredado. |
| Description | `Bank A PLC` | Nombre legal. |
| Alias | `BKA` | Nombre corto, solo descriptivo. |
| Alias Type | `SEC` | En el sistema actual es un recordatorio informal de para qué producto se usa. Aquí se reemplaza por la autorización por producto (sección 8). |
| BIC | `XXXXGB2LXXX` | Identificador SWIFT; se precarga en la instrucción de liquidación. |

Una contraparte puede ser además **banco corresponsal**: un broker que tiene un contrato
firmado con la institución. Todo corresponsal es broker, pero no todo broker es corresponsal.
Los depósitos solo se hacen con corresponsales, por lo que la contraparte lleva ese indicador.

Los **custodios** (entidades que custodian los valores) son también registros de la tabla de
contrapartes. No existe una tabla de custodio por defecto por moneda: cada instrumento se
asocia a la contraparte custodia que corresponda (por ejemplo, la plaza Londres o la plaza
Nueva York de un mismo banco).

### 4.4 Instrumentos

Un **instrumento** es un bono registrado en la base interna. Debe existir y estar confirmado
antes de operarlo. Su ciclo de vida es independiente del de la operación:

```
created ──▶ confirmed
 trader      back office
(borrador)   (datos completos)
```

1. El trader identifica el bono por su **ISIN** en la plataforma de datos de mercado y crea
   el instrumento como borrador con el siguiente **código** de la serie.
2. Envía la descripción del bono a back office (hoy, una captura de pantalla por correo).
3. Back office completa los datos y confirma el instrumento.
4. El trader actualiza su operación para que los cálculos tomen los datos definitivos.

El **código** tiene tres letras y dos dígitos (`AEF14`). Es un correlativo por trade class y
moneda: cuando se agota `AEF99` sigue `AEG01`. El sistema debe continuar la serie existente
de cada combinación. **[Por confirmar]** la tabla completa de prefijos por trade class y
moneda.

| Campo | Quién lo llena | Observación |
|---|---|---|
| Código | Trader | Correlativo, ver arriba. |
| ISIN | Back office | Viene de la plataforma de datos de mercado. |
| Tipo de ID | Sistema | Siempre `ISIN`. |
| Emisor | Trader | Código de cinco letras de la lista de emisores. |
| Fecha de emisión | Back office | |
| Fecha del primer cupón | Back office | |
| Fecha de vencimiento | Back office | |
| Cupón | Back office | Tasa anual en porcentaje (`3.25`). Decimales configurables. |
| Moneda de pago | Trader | Código de tres letras. |
| Custodio | Trader | Contraparte custodia. |
| Plaza del custodio | Trader | Plaza de esa contraparte. |
| Descripción | Sistema | Emisor + cupón + vencimiento: `ACOSS 3.25 09/09/28`. |
| G/L Classification | Trader | Código contable (`10601`). **[Por confirmar]** significado y lista de valores. |
| Tipo de cupón | Back office | `Fixed` o `Floating`. |
| Frecuencia de cupón | Back office | `Annual` o `Semi annual`. |
| Convención de conteo de días | Back office | Por ahora solo `ACT/ACT`; `ACT/360` y otras se agregarán después. |
| Ajuste de fechas de cupón | Back office | Regla cuando una fecha de cupón cae en día no hábil del mercado del bono (por ejemplo, siguiente día hábil). Mueve el pago y cambia la valorización en centavos. |

La **cuponera** (lista de fechas y montos de cupón) se genera automáticamente a partir de la
frecuencia, el cupón, el primer cupón y el vencimiento; nadie la carga a mano. El bono es un
instrumento determinístico.

Los errores históricos de cálculo provienen de datos del instrumento mal ingresados (un cupón
de `0.06` en lugar de `0.6`), no de la fórmula. Por eso la carga automática por ISIN desde el
proveedor de datos es una mejora futura prioritaria; tiene costo por consulta.

## 5. Ciclo de vida de una operación

Se conservan los nombres y los códigos de estado que la mesa ya usa.

```
created ──▶ confirmed ──▶ enriched ──▶ released
 trader       trader       jefe de mesa   back office
(editable)   código AE     código AR      código R
             "Pendiente"   "Confirmado"   "Enviado"
                 ▲   │
                 └───┘ change (el trader corrige y vuelve a confirmar)
```

- **created**: el trader ingresa la operación. Puede modificarla libremente.
- **confirmed** (`AE`, *awaiting enrich*): el trader dio por terminado el ingreso. Sigue
  pudiendo hacer *change* mientras el jefe de mesa no la apruebe. Si el jefe detecta una
  diferencia con el ticket, se la devuelve y el trader corrige y vuelve a confirmar.
- **enriched** (`AR`, *awaiting release*): el jefe de mesa comparó fecha valor, monto, trade
  class e instrumento (por ISIN) contra el ticket y aprobó. La operación queda bloqueada para
  el front office.
- **released** (`R`): back office revisó la operación, eligió la cuenta e instrucción de
  liquidación de la contraparte para esa moneda y liberó. Se genera el mensaje SWIFT que la
  plataforma SWIFT de la institución envía al custodio. No existe un estado intermedio de
  validación: revisión y *release* son un solo paso.

La operación registra siempre la **fecha de operación** (*trade date*), que es la fecha del
sistema, visible en pantalla porque alguna vez el sistema se levantó con una fecha atrasada.

Al confirmarse, la operación recibe un **System ID**: correlativo numérico de ocho dígitos
(`10300860`) generado por el sistema, que continúa la serie existente. Es el número que
contabilidad, auditoría y las ventas usan para referirse a la operación. **[Por confirmar]**
el significado del subnúmero de tres dígitos (`001`) que acompaña al System ID.

Los caminos de cancelación y rechazo se definen en cada especificación. Ninguna operación se
borra físicamente: se cancela. Toda transición registra quién la hizo y cuándo.

### Operaciones padre e hijas

Algunos mercados liquidan en múltiplos fijos (por ejemplo, bonos estadounidenses en múltiplos
de 50). En ese caso la operación principal se cancela y se generan operaciones hijas, cada
una con su propia instrucción de liquidación. El vínculo padre–hijas se conserva.

## 6. Ingreso de una compra o venta de valores

El sistema registra operaciones **ya ejecutadas** en la plataforma de negociación. El trader
cierra allí con la contraparte, recibe un ticket (monto, precio, fecha de liquidación, monto
neto) y transcribe la operación. La verificación previa de límites (*pre-trade*) ocurre en la
plataforma de negociación y está fuera del alcance.

El trader ingresa solo cinco datos: compra o venta, instrumento, nominal, precio y fecha
valor (más portafolio y contraparte en la cabecera). Todo lo demás se deriva o se calcula.

| Campo en pantalla | Origen | Regla |
|---|---|---|
| B/S | Ingreso | `B` compra, `S` venta. |
| Portafolio (Profit Center) | Ingreso | Lista filtrada por producto. |
| Contraparte | Ingreso | Lista filtrada por producto (solo brokers autorizados para `SEC`). |
| Código (instrumento) | Ingreso | Lista de instrumentos confirmados. |
| Trade class | Derivado | Del instrumento. Hoy se tipea y es fuente de errores; aquí se autocompleta y se guarda. |
| Emisor | Derivado | Del instrumento. |
| ISIN | Derivado | Del instrumento. |
| Moneda | Derivado | Del instrumento. |
| Cupón | Derivado | Del instrumento. Informativo. |
| Nominal | Ingreso | Valor nominal operado. |
| Precio | Ingreso | Precio del ticket, en porcentaje del nominal. Hasta 16 decimales. |
| F. Valor | Ingreso | Fecha de liquidación. Debe ser día hábil (sección 9). |
| Principal | Calculado | `nominal × precio / 100`, redondeado a 2 decimales. |
| Interest | Calculado | Interés corrido desde el último cupón hasta la fecha valor, redondeado a 2 decimales. Cero cuando la fecha valor coincide con la emisión o con un pago de cupón. |
| Valor Pagado | Calculado | `principal + interest`, redondeado a 2 decimales. |
| Memo 1 | Ingreso | Texto libre. Se usa en depósitos; en valores suele quedar vacío. |
| Rate | Heredado | No se usa en `SEC`. Se conserva vacío por compatibilidad. |

El sistema actual tiene además un campo de **ajuste de intereses** para corregir a mano
diferencias de `0.01` causadas por la precisión decimal. Este sistema no lo tiene: calcula con
precisión exacta y la especificación fija la regla de redondeo. **[Por confirmar]** la
fórmula de interés corrido para `ACT/ACT` y el modo de redondeo, con ejemplos del responsable
funcional (ver `docs/ejemplos/`).

El rendimiento al vencimiento que muestra la plataforma de negociación es informativo y no
se calcula ni se guarda; la valorización depende del precio pagado.

## 7. Posiciones y lotes

Cada compra confirmada crea un **lote**, identificado por el System ID de la compra. La
**posición** de un instrumento es la suma de sus lotes con saldo: lo comprado que no ha
vencido ni se ha vendido.

Una **venta** usa el mismo formulario que la compra y agrega una lista dinámica de pares
**posición vendida** (System ID del lote) y **cantidad**. Reglas:

- Cada lote debe existir y corresponder al instrumento que se vende.
- La cantidad no puede superar el saldo disponible del lote.
- Una venta parcial reduce el saldo del lote; el reporte de posición muestra el lote original
  y lo que queda.
- No hay ventas en corto.

El sistema actual trata estos campos como informativos y no valida nada; middle office
descubre los errores al conciliar con contabilidad, cuando ya es tarde. Hoy el trader y el
jefe de mesa verifican la posición a mano en el módulo contable antes de vender. Validar esto
en el ingreso es una de las principales mejoras del sistema.

Los lotes permiten reconstruir la posición acumulada por instrumento y fecha de compra, y
responder a auditoría, que pide muestras por System ID y espera ver el registro, el ticket y
el respaldo de la plataforma.

## 8. Alertas y límites

Los controles del módulo de cumplimiento producen alertas de dos tipos:

- **Alerta dura (hard)**: bloquea la transición. Es el tipo por defecto.
- **Alerta blanda (soft)**: se muestra y se registra, pero permite continuar.

Las alertas se evalúan **antes de confirmar** la operación. Límites configurados por middle
office para valores:

| Límite | Qué controla |
|---|---|
| **Position limit** | Exposición acumulada por nombre (no por operación). Se organiza en un perfil **total** y perfiles hijos referenciales por tramo (`INV`, `NO INV`). Solo el total bloquea; los hijos muestran cómo se consumió. |
| **Concentration limit** | Exposición acumulada por producto (`SEC`, `DEP`, …). |
| **Tradable names** | Nombres elegibles por producto. Una contraparte con contrato puede estar suspendida y debe generar alerta. |
| **Plazo máximo** | Días calendario desde la fecha valor hasta el vencimiento, por emisor y por producto. |

Los límites se consumen en **nominal**, no a precio de mercado, y se calculan en dólares
estadounidenses convirtiendo las otras monedas con el tipo de cambio de cierre (sección 1).

**[Por confirmar]** si el *position limit* aplica al emisor, al broker, o existen ambos. En
el ejemplo recibido el nombre limitado es un emisor (`ACOSS`); en la explicación se lo llamó
contraparte. El modelo lo trata como un límite sobre un nombre con perfil de riesgo, lo que
cubre ambos casos.

Vista de límites que la mesa espera, con el ejemplo del responsable funcional:

| Risk Category | Risk Profile Code | Limit from | Limit to | Current | Available | Status |
|---|---|---|---|---|---|---|
| Position Limit | `LIM_ACOSS_INVI` | 0 | 750 000 000 | 800 000 000 | −50 000 000 | over |
| Position Limit | `TOTAL_ACOSS` | 0 | 1 500 000 000 | 1 500 000 000 | 0 | |
| Position Limit | `LIM_ACOSS_NO_INVI` | 0 | 750 000 000 | 700 000 000 | 50 000 000 | |

El tramo de inversión está excedido, pero lo que se valida es el total, que está justo en el
límite: una compra adicional de 50 millones produce la alerta dura.

## 9. Calendarios

La fecha valor debe ser día hábil en **tres calendarios a la vez**: el local (Perú), el de
Estados Unidos (back office no opera cuando ese mercado cierra) y el del mercado de la moneda
del instrumento. Los bonos europeos liquidan normalmente a dos días hábiles.

Ejemplo: una operación cerrada el 7 de octubre liquidaría el 9; como el 8 es feriado local,
pasa al 12; como el 12 es feriado en Estados Unidos, pasa al 13.

Una operación con fecha valor en día no hábil produce una alerta dura. El sistema actual
intentó agregar calendarios por moneda y no lo resolvió del todo.

## 10. Liquidación

Las instrucciones de liquidación dependen de la contraparte, la moneda y la plaza de la
contraparte (una misma entidad puede liquidar cada moneda en una plaza distinta). Back office
elige la instrucción al hacer el *release*. El sistema propone instrucciones por defecto (a
partir del BIC y la plaza de la contraparte) y back office puede editarlas. Un motor de reglas
(plaza × moneda × contraparte) es una mejora futura, no parte del MVP.

Los reportos usan mensajes SWIFT distintos de los de valores; se especifican en su propio
producto.

## 11. Compatibilidad con reportes existentes

La base operativa alimenta, mediante réplica, una capa de inteligencia de negocio para
límites y tableros, y middle office combina los datos de ejecución con los contables en una
herramienta de conciliación para el análisis de desempeño y los modelos de Excel. Esos
consumidores no van a cambiar. Por lo tanto los campos que consumen deben conservar nombre
lógico, significado y dominio de valores: códigos de portafolio, producto, trade class,
sucursal, estado, System ID, lotes y los campos de la operación y del instrumento. El
glosario registra esa correspondencia.

La primera versión **replica el procedimiento actual** para no invalidar manuales ni
reportes. Las validaciones adicionales se agregan sobre ese flujo sin cambiarlo.

## 12. Alcance y orden

| Orden | Producto / módulo | Comentario |
|---|---|---|
| 1 | Compra y venta de valores (`SEC`), módulo de ejecución hasta *enriched* | ~90 % de las operaciones cerradas. Incluye instrumentos, contrapartes, portafolios, lotes. |
| 2 | Módulo de cumplimiento para valores | Límites, nombres elegibles, plazo máximo, calendario, tipo de cambio. Pantallas del módulo de riesgos aún pendientes de recibir. |
| 3 | Módulo de liquidación para valores | Revisión, instrucciones, *release*, archivo SWIFT, operaciones hijas. |
| 4 | Depósitos (`DEP`) | Operación más sencilla; usa bancos corresponsales. |
| 5+ | Reportos, depósitos de oro, cambio de moneda, futuros | Reutilizan el flujo; cambia el cálculo y la instrucción SWIFT. |

Antes de programar cada producto se validan **mockups** del flujo con el responsable
funcional.

## 13. Lo que el sistema actual no hace y este sí debe hacer

- Validar que una venta referencie lotes existentes, del mismo instrumento y con saldo.
- Derivar trade class, emisor, ISIN, moneda y cupón del instrumento en lugar de tipearlos.
- Restringir contrapartes según el producto y alertar contrapartes suspendidas.
- Alertar operaciones con fecha valor en día no hábil de cualquiera de los tres calendarios.
- Listas desplegables filtradas por contexto en lugar de campos de texto libre con búsqueda.
- Imprimir o exportar la posición y la operación (PDF).
- Evitar operaciones duplicadas o perdidas mediante transacciones atómicas e idempotencia.
- Eliminar el ajuste manual de intereses mediante cálculo decimal exacto y precio de hasta
  16 decimales.
- Mostrar los campos de venta solo en ventas (hoy aparecen vacíos también en compras).
- Autenticación integrada con la cuenta institucional o con recuperación de contraseña sin
  intervención del jefe de back office; sesiones que no quedan bloqueadas al cerrar la
  ventana; certificados vigentes; sin modo de compatibilidad de navegador.
