# Visión general del dominio: la mesa de inversiones

Este documento describe cómo opera la mesa de inversiones que el sistema debe soportar.
Es la base de todas las especificaciones por producto. Fue elaborado a partir de la
explicación del responsable funcional y debe ser corregido por él antes de usarse como
referencia definitiva.

> Estado: **borrador para revisión**. Pendiente de validación por el responsable funcional.

## 1. Áreas y roles

Toda mesa de inversiones se organiza en tres áreas. Cada una tiene roles con permisos
distintos y ninguna puede hacer el trabajo de la otra.

| Área | Rol en el sistema | Qué hace |
|---|---|---|
| Front office | **Trader** (especialista, gestor de portafolio) | Crea y confirma operaciones. Es el responsable de la operación. |
| Front office | **Jefe de mesa** (supervisor, único por mesa) | Revisa y aprueba las operaciones de los traders mediante el *enrich*. |
| Middle office | **Middle office** | Configura límites por emisor, instrumento y contraparte; define alertas; administra calendarios. No opera. |
| Back office | **Back office** | Crea y confirma instrumentos. Hace la segunda revisión de la operación, define instrucciones de liquidación y genera las instrucciones SWIFT. |

## 2. Módulos del sistema

Los módulos reflejan las áreas. Cada especificación pertenece a un solo módulo.

1. **Ejecución** (front): instrumentos, contrapartes, portafolios, ingreso de operaciones,
   confirmación, *enrich*, posiciones.
2. **Cumplimiento** (middle): límites, reglas por producto y plazo, calendarios, alertas
   duras y blandas evaluadas en cada transición.
3. **Liquidación** (back): validación, instrucciones de liquidación por contraparte, moneda y
   sucursal, *release* y generación del archivo de instrucciones SWIFT.

## 3. Ciclo de vida de una operación

Se conservan los nombres de estado que la mesa ya usa.

```
created ──▶ confirmed ──▶ enriched ──▶ validated ──▶ released
 trader       trader      jefe de mesa   back office   back office
(editable)   (editable     (bloqueada     (instrucciones  (archivo SWIFT
              por trader)   para front)    definidas)      generado)
```

- **created**: el trader ingresa la operación. Puede modificarla libremente.
- **confirmed**: el trader da por terminado el ingreso. Sigue pudiendo hacer *change*
  mientras el jefe de mesa no la apruebe.
- **enriched**: el jefe de mesa aprobó. La operación queda bloqueada para el front office.
- **validated**: back office revisó la operación contra el ticket de la plataforma de
  negociación y definió las instrucciones de liquidación.
- **released**: se generaron las instrucciones de liquidación (archivo de texto con mensajes
  SWIFT) que luego se cargan en la plataforma SWIFT y llegan al custodio.

Los caminos de cancelación y rechazo se definen en cada especificación. Ninguna operación se
borra físicamente: se cancela. Toda transición registra quién la hizo y cuándo.

### Operaciones padre e hijas

Algunos mercados liquidan en múltiplos fijos (por ejemplo, bonos estadounidenses en múltiplos
de 50). En ese caso la operación principal se cancela y se generan operaciones hijas, cada
una con su propia instrucción de liquidación. El vínculo padre–hijas se conserva.

## 4. Alertas (warnings)

Los controles del módulo de cumplimiento producen alertas de dos tipos:

- **Alerta dura (hard)**: bloquea la transición. Es el tipo por defecto.
- **Alerta blanda (soft)**: se muestra y se registra, pero permite continuar.

Cada control especifica de qué tipo es. Ejemplos de controles: límite por contraparte
(usado y disponible), contraparte no autorizada para el producto, plazo máximo por tipo de
operación, fecha en día no hábil, venta sin posición.

## 5. Instrumentos

Un instrumento (por ejemplo un bono o un papel comercial) debe existir en el sistema antes de
operarlo. Su ciclo de vida es independiente del de la operación:

```
created ──▶ confirmed
back office  back office
```

- El trader identifica el instrumento por su **ISIN** en la plataforma de datos de mercado.
- Back office lo crea con el siguiente **correlativo** de la serie y completa sus datos:
  identificador, emisor, moneda, convención de conteo de días, fecha de emisión, cupón,
  frecuencia de cupón, vencimiento y la convención de ajuste de fechas de cupón cuando caen
  en día no hábil.
- Una operación no puede confirmarse contra un instrumento que no esté confirmado.
- El cálculo de intereses es una fórmula exacta; los errores históricos provienen de datos
  del instrumento mal ingresados (por ejemplo un cupón de 0,06 % en lugar de 0,6 %). Por eso
  la carga automática desde el proveedor de datos por ISIN es una mejora futura prioritaria.

## 6. Posiciones

La posición de un instrumento es todo lo comprado que no ha vencido ni se ha vendido. Una
venta debe referenciar una posición (lote) existente y no puede superar la cantidad
disponible. La institución no permite ventas en corto.

## 7. Contrapartes

Cada contraparte está autorizada para tipos de producto específicos (por ejemplo, un broker
para valores; una contraparte de depósitos puede no estar habilitada para valores ni
reportos). Al ingresar una operación solo se pueden elegir contrapartes autorizadas para el
producto seleccionado.

## 8. Calendarios

El calendario de días hábiles combina los feriados locales, los del país de la moneda y los
de los mercados que la especificación indique. Una operación con fecha en día no hábil
produce una alerta dura.

## 9. Liquidación

Las instrucciones de liquidación dependen de la contraparte, la moneda y la sucursal de la
contraparte (una misma entidad puede liquidar cada moneda en una plaza distinta). El sistema
propone instrucciones por defecto y back office puede editarlas antes del *release*. Un motor
de reglas (sucursal × moneda × contraparte) es una mejora futura, no parte del MVP.

## 10. Compatibilidad con reportes existentes

La base operativa alimenta, mediante réplica, una capa de inteligencia de negocio para
límites y tableros, y middle office combina los datos de ejecución con los contables en una
herramienta de conciliación para el análisis de desempeño. Esos consumidores no van a cambiar.
Por lo tanto los campos que consumen deben conservar nombre lógico, significado y dominio de
valores. El glosario registra esa correspondencia. La mejora de este sistema está en la
interfaz, las validaciones y los controles, no en rediseñar el modelo de datos.

## 11. Alcance inicial

| Orden | Producto / módulo | Comentario |
|---|---|---|
| 1 | Compra y venta de valores (bonos y papeles comerciales), módulo de ejecución hasta *enriched* | ~90 % de las operaciones cerradas. Sin alertas en esta primera entrega. |
| 2 | Módulo de cumplimiento para valores | Límites, contrapartes por producto, calendario, posición. |
| 3 | Módulo de liquidación para valores | Validación, instrucciones, *release*, archivo SWIFT, operaciones hijas. |
| 4+ | Depósitos, cambio de moneda, reportos, futuros | Reutilizan el flujo; cambia principalmente el cálculo. |

## 12. Lo que el sistema actual no hace y este sí debe hacer

- Validar que una venta tenga posición disponible.
- Restringir contrapartes según el producto.
- Alertar operaciones en día no hábil.
- Listas desplegables filtradas por contexto en lugar de campos de texto libre con búsqueda.
- Imprimir o exportar la posición y la operación (PDF).
- Evitar operaciones duplicadas o perdidas mediante transacciones atómicas e idempotencia.
- Eliminar los parámetros de ajuste de redondeo mediante cálculo decimal exacto.
