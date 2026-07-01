# C4 print-service in Python (FastAPI + `brother_ql`), not TypeScript or Gleam

**Status:** accepted

The C4 print service is built in **Python (FastAPI)** using **`brother_ql`** to drive the networked Brother QL-1110NWB label printers. We reconsidered Node/Hono, Deno/Oak, and Gleam — the shop is TypeScript-heavy — but chose Python because the decisive part of C4 (converting the card PNG into the Brother QL raster protocol and sending it to the printer) has exactly one battle-tested implementation: `brother_ql` (Python). A TS/Gleam service would have to reimplement the QL raster protocol (real risk) or subprocess `brother_ql` anyway — which lands Python in the deployment regardless, stacking a second runtime on a three-endpoint service.

## Considered options

- **Python / FastAPI + `brother_ql` (chosen)** — native driver, one runtime, zero glue.
- **Node / Hono + `brother_ql` subprocess** — service logic in the team's language, but Python is still required for printing → two runtimes.
- **Deno / Oak** — as Node, with more friction for native/subprocess deps; no edge here.
- **Gleam** — no QL driver, immature OIDC; the same ecosystem gap that ruled it out for the CLI (ADR-0003).

## Consequences

- The stack is **Rust CLI + Python service**. The boundary is HTTP, so the split is clean; the CLI↔C4 contract (`POST /print`, `GET /boards`, Google OIDC) is unchanged.
- **TypeScript remains the right tool for the editor shims and any future UI** — this ADR is scoped to the printer-adjacent service only.
- Revisit if a mature non-Python QL raster driver ever appears.
