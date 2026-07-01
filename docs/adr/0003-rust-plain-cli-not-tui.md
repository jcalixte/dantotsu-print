# `dp` CLI in Rust, as a plain command (not a TUI)

**Status:** accepted

The `dp` capture CLI is built in **Rust** as a plain one-shot command (`clap`), rendering code → PNG via a `syntect`/`silicon`-style pipeline and previewing inline via an image protocol (`viuer`). We chose Rust over TypeScript/Node and Gleam because it ships as a **single static binary** — the easiest thing to distribute across a polyglot 400-engineer fleet (F4) — and has the **best-fit rasterization ecosystem**, which is the hard part of this tool. We deliberately do **not** use a TUI framework (ratatui/ink): `dp` is fire-and-exit (capture → render → preview → confirm / `-y` → emit), so a persistent interactive TUI would be machinery without payoff and would break piping and the non-interactive `-y` path. This also matches the QFD priority order, where F3 (preview/trim) is the lowest-weighted function and conflicts with F1.

## Considered options

- **Rust plain CLI (chosen)** — single binary; `syntect`/`silicon` + `viuer`.
- **TypeScript / Node** — familiar for a JS shop, `shiki` highlighting, but painful PNG rasterization (`node-canvas` native dep or headless browser) and runtime-dependent distribution.
- **Gleam** — pleasant language, but the image / terminal-graphics ecosystem is essentially absent; runtime-dependent.

## Consequences

- The print-service (C4) is independent and stays **Python / `brother_ql`**; the CLI just POSTs an image.
- Maintainers must be Rust-comfortable — the accepted bus-factor cost.
- If F3 later grows into *interactive* trimming (live line-range scrubbing), **ratatui becomes the right tool for that mode** — added then as a scoped `dp trim -i`, not the v1 hot path.
