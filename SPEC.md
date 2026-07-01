# Spec: Dantotsu Print — `dp` CLI (v1)

> Phase 1 (Specify) artifact. Design rationale lives in [DESIGN.md](./DESIGN.md);
> vocabulary in [CONTEXT.md](./CONTEXT.md); decisions in [docs/adr/](./docs/adr).
> This spec covers **the `dp` CLI (Part A, most of this doc)** and the
> **C4 print-service (Part B, at the end)**.

## Objective

Let a developer turn a code defect they're looking at into a **Defect card** —
a monochrome, peel-stick label sized to the ~90×85 mm Defect cell of a physical
Dantotsu board — in **one keystroke, ≤ 10 s**, without leaving the editor or
touching a screenshot/crop/print dance. The card shows the verbatim code with
the defect **visually marked**, plus a computer-native back-link to the exact
source. The tool exists to make filling the board *lower-friction than pasting
into Notion* (goal G1).

**Primary user:** a developer on a team that runs a daily Dantotsu board.
**Success = ** the capture step is no longer the reason people avoid the board.

## Tech Stack

- **Language:** Rust (stable), single static binary — **macOS, Linux, Windows**. See ADR-0003.
- **CLI framework:** `clap` (derive). No TUI framework (fire-and-exit).
- **Render:** `syntect` (TextMate grammars, `silicon`-style) → raster via the
  `image` crate; **bundled mono font (default JetBrains Mono, SIL OFL),
  ligatures OFF** (character-exact — see Open Q#1), rendered at **300 dpi** to
  the **90×85 mm** cell.
- **Preview:** `viuer` (inline images: Kitty / iTerm2 / Ghostty / sixel), with a
  text-grid and OS-viewer fallback.
- **Git:** shell out to `git blame`/`git rev-parse`/`git remote` (no libgit2
  dependency in v1).
- **HTTP:** `reqwest` (blocking) to POST the card to the print service.
- **Auth:** Google Workspace SSO — OAuth 2.0 loopback/browser flow (`dp login`);
  OIDC **Bearer** token cached (OS keychain, config fallback) and refreshed; the
  service validates the `theodo.com` hosted domain.
- **QR (optional):** `qrcode` crate.
- **Config:** TOML via `serde`.
- Editor shims are **not** part of this crate — they are thin per-editor
  snippets that invoke `dp` (see Project Structure → `shims/`).

## Commands

**Dev / build (this repo):**
```
Build:   cargo build --release
Test:    cargo test
Lint:    cargo clippy -- -D warnings
Format:  cargo fmt
Run:     cargo run -- <args>
```

**The `dp` surface (three commands):**

### `dp` — the hot path (capture → render → preview → emit)
```
… | dp --file <path> --mark <L:C-L:C>… [flags]
```
| Flag | Meaning | Default |
|------|---------|---------|
| `-f, --file <path>` | source file; enables git context + surrounding lines | — (else read stdin) |
| `--mark <L:C-L:C>` | emphasized defect span; **repeatable** (multi-cursor → many) | selection / whole `--lines` |
| `--lines <A-B>` | override the shown window | union of marks ± context |
| `--context <N>` | context lines around the marks | `3` |
| `--emphasis <box\|bold\|highlight\|underline>` | how spans are marked | `box` |
| `--board <id>` | target board | resolved from config |
| `--qr` | include the QR permalink | off |
| `--preview <image\|text\|os\|none>` | preview mode | `auto` (image if TTY supports, else text) |
| `-y, --yes` | skip preview + confirm | off |
| `--dry-run` | render + preview, do **not** emit | off |
| `--out <path>` | also write the PNG to disk | — |
| `--force` | print despite overflow (adds a visible truncation marker) | off |

**Flow:**
1. Resolve **board** (precedence: `--board` › repo `.dantotsu.toml` › user config). No board → abort with a hint to run `dp config`.
2. Acquire code: from `--file` (window = union of marks ± `--context`, or `--lines`), else from **stdin** (degraded: no context, no precise mark).
3. Derive **back-link**: `git blame -L` over the marked lines → *introducing* commit SHA (fallback `HEAD`); repo slug from the remote → footer `owner/repo · path · La–b · @sha`.
4. **Fit check** against the cell budget (~48 cols × ~19 rows @ 8 pt). Overflow → **abort**, reporting how far over and suggesting a narrower `--lines`; `--force` overrides with a truncation marker. Never silently shrink below the 8 pt floor.
5. **Render** the mono PNG (300 dpi, 90×85 mm): syntax-highlighted code, spans emphasized, footer band, optional QR, with the cell boundary drawn.
6. **Preview** unless `-y`: inline / text-grid / OS viewer; overflow visibly spills past the drawn boundary. Confirm `[y/N]`.
7. **Emit:** `POST {service}/print`. On success, print the resolved printer/board. On failure, exit non-zero with the service error. **No local queue in v1** (`--dry-run` to test without emitting).

### `dp config` — one-time per-dev setup
```
dp login                                # Google Workspace OAuth (browser); caches OIDC token
dp logout
dp config set service.url https://dantotsu.internal
dp config set default_board floor3-payments
dp config show
```
Writes `~/.config/dp/config.toml`; the OIDC token is cached in the OS keychain (config fallback). A committed repo-level `.dantotsu.toml` may set `board = "…"`.

### `dp doctor` — self-serve diagnostics (serves F4 at 50 boards)
Checks and reports pass/fail with hints: config present · service reachable · resolved `board → printer` (via `GET /boards/{id}`) · terminal graphics capability · `git` available · render font available.

**Print-service interface (the CLI depends on, C4 owns):**
- `POST /print` — multipart: `card` (image/png), `board` (string), `meta` (JSON: repo, path, lines, sha, ts). → `200 {printer, job_id}` or error.
- `GET /boards/{id}` — → `{printer, status}` (used by `dp doctor`).
- Auth: `Authorization: Bearer <Google OIDC id_token>`; the service validates audience + `hd=theodo.com`.

## Project Structure
```
src/
  main.rs            → clap entry; dispatch dp / config / doctor
  capture.rs         → resolve file/stdin, marks, context window
  gitctx.rs          → blame → introducing SHA, repo slug, footer string
  render/            → syntect highlight → mono PNG at cell size
    layout.rs        → cell budget, fit/overflow math, cell boundary
    emphasis.rs      → box/bold/highlight/underline over spans
    footer.rs        → back-link line + optional QR
  preview.rs         → viuer inline / text-grid / OS viewer
  emit.rs            → multipart POST to the print service
  config.rs          → TOML load/merge (flag › repo › user), board resolution
  doctor.rs          → diagnostics
tests/
  fixtures/          → sample source files + committed golden PNGs
  render_golden.rs   → golden-image tests (tolerance-compared)
  cli.rs             → arg parsing, precedence, --dry-run against a mock service
shims/               → per-editor keystroke recipes (not part of the crate)
  vim.md  jetbrains.md  vscode.md  xcode.md
docs/adr/            → decisions
```

## Code Style
Idiomatic Rust: `Result<T, DpError>` (via `thiserror`), no `unwrap()` outside tests, `snake_case`, doc-comments on public items, `clap` derive.

```rust
/// A character-precise span to emphasize on the card: (line, col) → (line, col),
/// 1-based, inclusive start / exclusive end column. Supports multi-line spans.
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub struct Span {
    pub start: (u32, u32),
    pub end: (u32, u32),
}

impl Span {
    /// Parse `L:C-L:C` (e.g. "108:12-110:4"). Column defaults to whole-line
    /// when omitted (`108-110`), for editors that cannot report columns.
    pub fn parse(s: &str) -> Result<Self, DpError> {
        // … returns DpError::BadSpan(s.into()) on malformed input
    }
}
```

## Testing Strategy
- **Framework:** `cargo test`. **Unit:** fit/overflow math, `Span::parse`, footer formatting, board-resolution precedence, blame-output parsing. **Integration:** `tests/cli.rs` drives the binary (arg parsing, precedence, `--dry-run` against a mock HTTP service); `tests/render_golden.rs` renders fixtures and compares to committed golden PNGs within a pixel tolerance.
- **Determinism:** pin font + `syntect` grammar/theme versions so golden images are stable; regenerating a golden requires explicit review.
- **Coverage:** meaningful coverage on `capture`/`render/layout`/`config` (the logic); no hard percentage gate.

## Boundaries
- **Always:** run `cargo test` + `cargo clippy -D warnings` before commit; keep rendering deterministic for goldens; validate spans/line ranges before use; resolve an explicit board before emitting.
- **Ask first:** adding a heavy render dep (Skia, headless browser); changing the print-service API contract; changing the config schema or file locations; adding a new editor shim.
- **Never:** commit auth tokens or secrets; print to a board without an explicitly resolved board id; silently shrink code below the legible floor; regenerate golden images without review.

## Success Criteria
- **F1:** on a pilot board, selection → card-in-hand ≤ 10 s; hot path is one keystroke after selection (shim-bound).
- **F2:** card is byte-verbatim to the source; fits 90×85 mm; readable at ~0.4 m at 300 dpi; overflow **aborts** with an over-by message (no silent shrink).
- **F3:** preview renders the exact print artifact with the cell boundary and visible span marks; `-y` skips it.
- **F5:** the footer resolves to the introducing commit; `--qr` opens the correct commit-pinned permalink.
- **Ops (G2/F4):** `dp doctor` correctly diagnoses a mis-set board / unreachable service / unsupported terminal; the binary builds and runs on **macOS, Linux, and Windows** dev laptops (inline preview falls back to OS-viewer / text-grid on terminals without sixel — notably older Windows consoles).
- **Emit:** a successful run prints at the correct board's printer; failure exits non-zero with an actionable message.

## Open Questions
_(Both pilot-time confirmations — neither blocks implementation.)_
1. **Exact mono font** — default **JetBrains Mono, ligatures OFF** (ligatures would obscure operator defects, break sub-span marking, and need a shaping engine → net-negative for a defect card). Confirm legibility at 300 dpi / 8 pt on real prints (pins goldens), or swap the glyph set (FiraCode et al. are fine — just keep ligatures off).
2. **Removable adhesive** — label stock is in hand; confirm it's *removable* so peel-on-resolve doesn't tear the A3.

_Resolved: **C4 (print service) owner = the Dantotsu Print lead** — same owner as this CLI, so both sides of the `POST /print` / `GET /boards` + Google-OIDC contract are co-owned; auth = Google Workspace SSO (`dp login`, OIDC Bearer, `hd=theodo.com`); cell = 90×85 mm; mono confirmed; targets = macOS + Linux + Windows; label stock available._

## Explicit v1 Non-Goals
Interactive trim/markup (deferred ratatui/GUI mode) · gap-eliding `── snip ──` between distant disjoint spans · SSO short-link redirector · sub-line marking in VS Code/Cursor (needs the thin extension) · colour on paper · editor extensions beyond keystroke shims · non-git repos · local print queue/retry.

---

# Part B — C4 print-service (v1)

Small internal service (component **C4**) that receives cards and drives the board printers. Built here, owned by the project lead. Topology: ADR-0002 · language: ADR-0004.

## Objective
Accept a rendered Defect card + a board id, and print it on that board's networked label printer — the thin router that lets the CLI stay client-side and lets 50+ printers be managed centrally.

## Tech Stack
Python · FastAPI · `brother_ql` (PNG → QL raster over the network) · `google-auth` (OIDC verification). Containerized, deployed on an internal host.

## Endpoints (contract also consumed by the CLI, Part A)
- **`POST /print`** — auth: `Authorization: Bearer <Google OIDC id_token>` (validate audience + `hd=theodo.com`). Body multipart: `card` (image/png), `board` (id), `meta` (JSON: repo, path, lines, sha, ts). Resolves board → printer, converts PNG → QL raster (102 mm continuous, auto-cut), sends via `brother_ql`. → `200 {printer, job_id}` | `4xx/5xx {error}`.
- **`GET /boards/{id}`** — → `{printer, status}` (printer reachability / label state as far as the device reports). Backs `dp doctor`.

## Registry
A config file / small store mapping `board-id → {printer_ip, model, media}`, editable by the owner. No self-service registration in v1.

## Boundaries
- **Always:** validate the OIDC token (audience + `hd`) before printing; reject unregistered boards.
- **Ask first:** changing the `/print` or `/boards` contract (it's shared with the CLI).
- **Never:** log card *contents* beyond metadata; expose the service unauthenticated.

## Success Criteria
A valid `POST /print` with a real board id prints the card on that board's QL-1110NWB; an unregistered board or an invalid/absent token is rejected with a clear error; `GET /boards/{id}` returns usable status for `dp doctor`.

## Non-Goals (v1)
Web UI · board self-registration · SSO short-link redirector (F5 later-upgrade) · multiple printers per board · retry/queue beyond one synchronous attempt.

