# Investment Desk MVP

Sistema de gestión para una mesa de inversiones (nombre de trabajo: **IMS**): ingreso y
aprobación de operaciones, controles de cumplimiento y liquidación. Primer alcance: compra y
venta de valores.

## Cómo está organizado

| Ruta | Qué es |
|---|---|
| [`.specify/memory/constitution.md`](.specify/memory/constitution.md) | Constitución del proyecto: principios no negociables, stack y proceso. |
| [`docs/`](docs/README.md) | Especificaciones de negocio: visión general del dominio, glosario y especificaciones por producto. |
| [`CLAUDE.md`](CLAUDE.md) | Guía operativa para agentes de IA que trabajan en el repositorio. |
| `backend/`, `frontend/` | Código de la aplicación (pendiente de scaffolding). |

## Stack

Python 3.12 + Django 5 + Django REST Framework sobre Oracle Database; frontend React +
TypeScript con Ant Design, comunicándose solo por API REST. Detalle y justificación en la
constitución.

## Proceso

El desarrollo sigue [Spec Kit](https://github.com/github/spec-kit): cada producto se
especifica, planifica y descompone en tareas antes de implementarse.
