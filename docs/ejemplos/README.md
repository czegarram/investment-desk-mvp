# Ejemplos de aceptación

Casos reales anonimizados con el resultado esperado. Son las pruebas de aceptación de los
cálculos: cada especificación referencia los ejemplos que la validan y la implementación debe
reproducirlos exactamente.

## Reglas

- Los instrumentos son datos públicos de mercado (ISIN, emisor, cupón, fechas). No se
  incluyen nombres de la institución, de personas, de contrapartes reales ni códigos de
  usuario.
- Cada ejemplo indica su fuente (ticket de la plataforma de negociación, pantalla del sistema
  actual, planilla del responsable funcional) y su estado de validación.
- Los montos se escriben con el formato del sistema: separador de miles coma, decimal punto,
  dos decimales en montos monetarios.

## Índice

| Archivo | Producto | Caso | Estado |
|---|---|---|---|
| [`valores/compra-bono-nueva-emision.md`](valores/compra-bono-nueva-emision.md) | SEC | Compra de un bono en su fecha de emisión, sin interés corrido | Validado contra ticket y pantalla del sistema actual |

## Pendientes de obtener del responsable funcional

- Compra de un bono con interés corrido (`ACT/ACT`, cupón anual).
- Compra con cupón semestral.
- Venta parcial de un lote y venta sobre dos lotes.
- Fecha valor desplazada por feriado local y por feriado de Estados Unidos.
- Operación que excede el perfil de límite total.
