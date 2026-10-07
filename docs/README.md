# Documentación del proyecto

Este directorio contiene las especificaciones de negocio del sistema. Según la
[constitución](../.specify/memory/constitution.md), ninguna pantalla, campo o cálculo se
implementa sin estar documentado aquí primero.

## Estructura

| Ruta | Contenido |
|---|---|
| [`dominio/vision-general.md`](dominio/vision-general.md) | Cómo opera una mesa de inversiones: áreas, roles, módulos y ciclo de vida de una operación. Lectura obligatoria antes de cualquier especificación. |
| [`glossary.md`](glossary.md) | Glosario español ↔ identificadores en inglés, y mapa de compatibilidad con los campos del sistema actual. |
| `dominio/<modulo>/<producto>.md` | Especificaciones por producto (campos, reglas, cálculos, ejemplos). Se crean con `/speckit-specify`. |
| `ejemplos/` | Casos reales anonimizados con resultado esperado; son las pruebas de aceptación de los cálculos. |

## Convenciones

- Las especificaciones se escriben en español con el vocabulario de la mesa.
- Los identificadores de código (modelos, campos, endpoints) van en inglés y se registran en el
  glosario.
- Cada campo especifica: nombre en pantalla, identificador, tipo, origen (ingreso, calculado,
  referencia), reglas de validación, y si es consumido por reportes aguas abajo.
- No se incluye información sensible: nombre de la institución, datos reales de operaciones,
  contratos ni términos comerciales. El repositorio es público.
