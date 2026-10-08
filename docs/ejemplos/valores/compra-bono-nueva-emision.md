# Compra de un bono en su fecha de emisión

**Producto:** SEC · **Trade class:** BONOS · **Fuente:** ticket de la plataforma de
negociación y pantalla de la operación en el sistema actual, 2 de septiembre de 2026.
**Estado:** validado; los montos coinciden en ambas fuentes.

## Instrumento

| Campo | Valor |
|---|---|
| Código | `AEF14` |
| ISIN | `FR0129816759` |
| Emisor | `ACOSS` (agencia francesa; trade class BONOS) |
| Descripción | `ACOSS 3.25 09/09/28` |
| Moneda | `EUR` |
| Tipo de cupón | Fixed |
| Cupón | `3.25` |
| Frecuencia de cupón | Annual |
| Convención | `ACT/ACT` |
| Fecha de emisión | 2026-09-09 |
| Fecha del primer cupón | 2027-09-09 |
| Fecha de vencimiento | 2028-09-09 |
| Precio de emisión | `99.859` |

## Operación

| Campo | Valor | Origen |
|---|---|---|
| B/S | `B` | Ingreso |
| Portafolio | `INVEUR` | Ingreso |
| Contraparte | Broker autorizado para SEC | Ingreso |
| Código | `AEF14` | Ingreso |
| Trade class | `BONOS` | Derivado |
| Emisor | `ACOSS` | Derivado |
| ISIN | `FR0129816759` | Derivado |
| Moneda | `EUR` | Derivado |
| Cupón | `3.25` | Derivado |
| Fecha de operación | 2026-09-02 | Sistema |
| Nominal | `10,000,000.00` | Ingreso |
| Precio | `99.859` | Ingreso |
| F. Valor | 2026-09-09 | Ingreso |

## Resultado esperado

| Campo | Valor | Cálculo |
|---|---|---|
| Principal | `9,985,900.00` | 10,000,000 × 99.859 / 100 |
| Interest | `0.00` | La fecha valor coincide con la fecha de emisión: 0 días devengados |
| Valor Pagado | `9,985,900.00` | 9,985,900.00 + 0.00 |

Al confirmar, la operación recibe un System ID de ocho dígitos y queda en estado `AE`. Esa
compra es un lote con saldo disponible de `10,000,000.00`.

## Notas

- El rendimiento que muestra la plataforma (`3.324`) es informativo; el sistema no lo
  calcula ni lo guarda.
- Este caso no ejercita el interés corrido. Sirve para validar el principal, la derivación
  de campos desde el instrumento y la creación del lote.
