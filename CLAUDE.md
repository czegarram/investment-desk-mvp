# CLAUDE.md

Runtime guidance for AI agents working in this repository. The project constitution at
`.specify/memory/constitution.md` is the authority; if anything here conflicts with it, the
constitution wins and this file must be fixed.

## What this project is

An investment-desk management system (working name IMS) for a financial institution: operation
entry, approval, compliance controls and settlement for securities first, other instruments
later. Business context lives in `docs/dominio/vision-general.md`; terms in `docs/glossary.md`.

## How work flows

Spec Kit drives every feature: `/speckit-specify` → `/speckit-clarify` → `/speckit-plan` →
`/speckit-tasks` → `/speckit-analyze` → `/speckit-implement`. One feature branch per
specification. Do not write application code for a product whose specification is not in
`docs/`. If a specification has a gap, update the specification first.

## Non-negotiables to check before every change

- **Oracle** is the only database engine, in every environment. Use the Django ORM only; no
  engine-specific fields, no hand-written sequences or triggers; identifiers ≤ 30 characters.
- **Decimals** for every financial value: `Decimal` and `DecimalField` with explicit precision
  in Python, strings on the API, `decimal.js` in the frontend. Never `float` or JS `number`.
  Every calculation states its rounding rule.
- **Validation in the domain layer**, never only in the UI. Reference data comes from
  context-filtered lists. Mandatory controls: position-backed sales, counterparties by
  product, business-day calendar.
- **State machine** with the legacy names: created → confirmed → enriched → validated →
  released. Immutable after enriched. Four roles: trader, desk head, middle office, back
  office. No physical deletes. Correlativos continue existing sequences and are never reused.
- **Atomic transactions** with `select_for_update()` for sequence allocation, limit
  consumption and positions. Idempotent write endpoints. Query-count assertions on every
  endpoint. Paginated lists.
- **Legacy field compatibility**: fields consumed downstream keep their meaning; record every
  mapping in `docs/glossary.md`.
- **On-prem**: no runtime internet, no CDNs, configuration by environment variables, single
  deployable (Django serves the Vite build).
- **Nothing sensitive in the repo**: it is public. No institution name, real data, contract
  terms or commercial information.

## Stack

Backend: Python 3.12+, Django 5.x, Django REST Framework, drf-spectacular, python-oracledb
(thin mode), `uv`, `ruff`, `pytest-django`.
Frontend: React 18+, TypeScript strict, Vite, TanStack Query, React Hook Form + Zod, React
Router, Ant Design, `decimal.js`, ESLint, Prettier, Vitest, Testing Library, Playwright,
axe-core, Lighthouse CI.
Dev database: Oracle Database 23ai Free container (`container-registry.oracle.com/database/free`).

## Conventions

- UI text, specifications and glossary in Spanish (`es-PE` formats). Code identifiers in
  English, registered in the glossary.
- Commit messages in English, conventional style (`feat:`, `fix:`, `docs:`, …).
- Worked examples from the domain owner live in `docs/ejemplos/` and are the acceptance tests
  for calculations.
