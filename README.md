# Dantotsu Print

Tooling that moves code-quality defects from the IDE onto the physical
"Daily — Dantotsu and Problem Solving" board with minimal friction, preserving
the Dantotsu ritual of *visualizing the defect* — which for developers means
seeing the actual lines of code.

A developer turns the code they're looking at into a **Defect card** — a
monochrome, peel-and-stick label sized to the ~90×85 mm **Defect** cell of a
physical A3 board — in one keystroke, without leaving the editor. The card
shows the verbatim code with the defect visually marked, plus a back-link to
the exact source (`path·lines·@sha` + QR). The machine prints only the Defect
cell; the human handwrites the causes, countermeasures, owner, and status.

## Status

**Design / spec stage — no code yet.** The vocabulary, design, spec, and key
decisions are settled (see [Documentation](#documentation)); implementation has
not started. The build plan is in [PLAN.md](./PLAN.md).

## How it works

Two components, split over an HTTP boundary:

- **`dp` CLI** (Rust) — invoked by a thin per-editor keystroke shim. Resolves
  git context, renders the selected lines to a PNG client-side, previews it in
  the terminal, then `POST`s the card image + target board to the print service.
- **C4 print service** (Python / FastAPI + `brother_ql`) — a thin router that
  forwards the raster to that board's networked Brother QL-1110NWB label
  printer, and owns the board→printer registry, retries, and print status.

Target: capture → card in **one keystroke, ≤ 10 s**, for a fleet of ~400
engineers and 50+ boards. Rationale for every choice above lives in the
[ADRs](#decisions-adrs).

## Getting started

There is nothing to build or run yet. Once the two tracks in
[PLAN.md](./PLAN.md) land, install and usage instructions will live here — track
progress there in the meantime.

<!-- docs:start -->
## Documentation

Read in this order — language first, then why, what, and how.

| Document | What it is |
| --- | --- |
| [CONTEXT.md](./CONTEXT.md) | **Ubiquitous language** — the project glossary. The shared vocabulary (Board, Defect, Defect card, yokoten…) that should appear verbatim in code, tests, commits, and docs. |
| [DESIGN.md](./DESIGN.md) | **Design (QFD)** — decomposes the goals into functions, approaches, components, and the tradeoffs taken, with strength matrices and the deployment context (~400 engineers, 50+ boards). |
| [SPEC.md](./SPEC.md) | **Spec (Phase 1)** — what we're building: the `dp` CLI (Part A) and the C4 print service (Part B), with the capture objective (one keystroke, ≤ 10 s) and the service interface. |
| [PLAN.md](./PLAN.md) | **Plan (Phase 2)** — how we'll build it: two parallel tracks (Rust `dp` CLI, Python print service) meeting at a single integration point, guided by *render first, print early*. |

### Decisions (ADRs)

Architectural decisions and the tradeoffs behind them — see [docs/adr/](./docs/adr).

| ADR | Decision |
| --- | --- |
| [0001](./docs/adr/0001-cli-engine-over-per-editor-extension.md) | CLI engine with per-editor keystroke shims, not a per-editor extension. |
| [0002](./docs/adr/0002-central-print-service-over-direct.md) | Central print service routing to networked label printers, not direct-to-printer or CUPS. |
| [0003](./docs/adr/0003-rust-plain-cli-not-tui.md) | `dp` CLI in Rust, as a plain command (not a TUI). |
| [0004](./docs/adr/0004-c4-service-in-python.md) | C4 print service in Python (FastAPI + `brother_ql`), not TypeScript or Gleam. |
<!-- docs:end -->
