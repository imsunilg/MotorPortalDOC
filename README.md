# MotorPortalDOC

Documentation for **Motor Portal** — a bulk motor-insurance policy issuance
system built across four repositories:

- [MotorPortalWEB](https://github.com/imsunilg/MotorPortalWEB) — Angular 22 SPA (operator-facing UI)
- [MotorPortalAPI](https://github.com/imsunilg/MotorPortalAPI) — .NET 8 Web API (business logic, orchestration)
- [MotorPortalDB](https://github.com/imsunilg/MotorPortalDB) — PostgreSQL schema, functions/procedures, seed data
- **MotorPortalDOC** (this repo) — architecture, ER diagram, API reference, setup guide, changelog

All three application repos are complete and have been verified end-to-end,
including a real cross-repo integration pass driven through a headless
browser. This repo documents the system as it actually shipped.

## Start here

New to the system? Read in this order:

1. [`docs/architecture.md`](docs/architecture.md) — component diagram, the
   4-repo layout, and why it's split that way.
2. [`docs/setup-guide.md`](docs/setup-guide.md) — clone, install, run all
   three application repos from zero.
3. [`docs/smoke-test.md`](docs/smoke-test.md) — the full manual checklist
   to verify the running system end to end.

## Documentation index

| Document | What it covers |
|---|---|
| [`docs/architecture.md`](docs/architecture.md) | Component diagram (WEB → API → DB), the 4-repo layout, and the rationale for splitting it that way |
| [`docs/er-diagram.md`](docs/er-diagram.md) | The real 15-entity ER diagram, verified against the actual (lowercase snake_case) table/column names in MotorPortalDB |
| [`docs/batch-lifecycle.md`](docs/batch-lifecycle.md) | The 6 user-facing pipeline stages and the real 9-state backend status lifecycle enforced by `sp_advance_batch_status`, including how invalid records are isolated per-record without blocking a batch |
| [`docs/api-reference.md`](docs/api-reference.md) | Every real MotorPortalAPI endpoint — method, path, auth, request/response shape, purpose — read directly from the shipped controllers and DTOs |
| [`docs/setup-guide.md`](docs/setup-guide.md) | From-zero instructions: clone all 4 repos, install prerequisites, migrate the database, run the API and the web app, seeded login credentials |
| [`docs/changelog.md`](docs/changelog.md) | Dated, plain-language summary of what each build phase delivered across all four repos, including the 2 bugs found and fixed during the final integration pass |
| [`docs/smoke-test.md`](docs/smoke-test.md) | Manual, step-by-step end-to-end verification checklist for the whole running system (~15-20 minutes) |

## Reference design documents

These describe the system's **original intent**, written before the build.
Where the shipped code has since diverged from them (exact endpoint paths,
exact table/column casing, exact status values), the documents above and
the actual code are authoritative — these are kept for historical/planning
context.

| Document | What it covers |
|---|---|
| [`Motor-Portal-HLD.md`](Motor-Portal-HLD.md) | High-level design: system overview, architecture, key modules, batch lifecycle, NFRs, deployment view, risks |
| [`Motor-Portal-LLD.md`](Motor-Portal-LLD.md) | Low-level design: schema, API and sequence-level detail |
| [`Motor-Portal-Solution-Architecture.md`](Motor-Portal-Solution-Architecture.md) | Solution architecture: principles, C4 context/container views, deployment/integration/security architecture, technology rationale |
| [`Motor_Portal_ER_Diagram.docx`](Motor_Portal_ER_Diagram.docx) | Original ER diagram (Word) |
| [`Motor_Portal_Application.pptx`](Motor_Portal_Application.pptx) / [`.pdf`](Motor_Portal_Application.pdf) / [`.doc`](Motor_Portal_Application.doc) | Original application overview deck |
| [`motor-portal-vendor-demo-appstyle.html`](motor-portal-vendor-demo-appstyle.html) | Original vendor-demo-styled UI reference |

## Progress

- [x] Bootstrap (ground rules, README, .gitignore)
- [x] `architecture.md`, `er-diagram.md`, `batch-lifecycle.md`
- [x] `api-reference.md`
- [x] `setup-guide.md`, `changelog.md`
- [x] Final polished READMEs across all 4 repos
