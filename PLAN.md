# Plan: Dantotsu Print — `dp` CLI (v1)

> Phase 2 (Plan) artifact. Implements [SPEC.md](./SPEC.md). Reviewable: read it
> and say "right approach" or "change X" before we break it into tasks (Phase 3).

## Two tracks

- **Track A — `dp` CLI** (Rust, the critical path below).
- **Track B — C4 print service** (Python/FastAPI + `brother_ql`; built here,
  owned by the project lead — ADR-0004). Runs **fully in parallel**: Track A is
  built against a **mock `POST /print` stub** that honours the SPEC service
  interface, so neither track blocks the other. The mock stub is swapped for the
  real service at the **M6 ↔ CB1** integration point — the single sync point.

## Guiding principle

**Render first, print early.** The render is both the hard part *and* the only
physical unknown (small-size 300 dpi legibility, font). So the first slice
produces a real PNG you print and eyeball on an actual board — settling Open Q#1
before anything else is built on top of it.

## Milestones (Track A) — dependency-ordered

| # | Milestone | Depends on | Verification checkpoint |
|---|-----------|:----------:|-------------------------|
| **M0** | **Skeleton** — cargo crate, `clap` surface stubs (`dp`/`config`/`doctor`), `DpError` (`thiserror`), CI (`test`/`clippy -D warnings`/`fmt`) | — | `cargo build`; `dp --help` lists the three commands |
| **M1** | **Render slice (core)** — read `--file`/`--lines`, `layout` (cell budget + fit math), syntect → mono PNG at 300 dpi / 90×85 mm (JetBrains Mono, no ligatures), `--out` to disk | M0 | **Print `card.png` on the QL-1110NWB, read it at the board → settles Q#1 (font/legibility).** Golden test: fixture → committed golden PNG |
| **M2** | **Marking + overflow** — `Span::parse` (`L:C-L:C`, repeatable, multi-line), `emphasis` (box/bold/highlight/underline), context window (union of marks ± `--context`), overflow **abort** + `--force` truncation mark + drawn cell boundary | M1 | Goldens: sub-line box, multi-span, overflow-abort (exit code + "N over" message) |
| **M3** | **Back-link** — `gitctx`: `blame -L` → introducing SHA (fallback HEAD), repo slug from remote, footer band `owner/repo · path · La–b · @sha`, optional `--qr` | M1 | On a real repo, footer resolves to the introducing commit; `--qr` opens the permalink; unit tests parse blame output |
| **M4** | **Preview** — `viuer` inline (Kitty/iTerm/Ghostty/sixel) → text-grid → OS viewer; `--preview`; confirm `[y/N]`; `-y` skips | M1 | Inline in Ghostty; text-grid fallback; `-y` non-interactive; piped (no TTY) does not hang |
| **M5** | **Config + auth** — `config` TOML load/merge, board precedence (`--board` › repo `.dantotsu.toml` › user), `dp config set/show`; Google OAuth loopback (`dp login`/`logout`), token cache (`keyring` + config fallback), Bearer attach | M0 | Precedence unit tests; `dp login` completes OAuth against a test client; token cached + refreshed |
| **M6** | **Emit** — `emit` multipart `POST /print` (PNG + board + meta JSON), non-zero exit on failure, `--dry-run` skips | M2, M3, M5, mock stub | `--dry-run`; against the mock stub returns printer/job; failure path exits non-zero with an actionable message |
| **M7** | **Doctor + shims + packaging** — `doctor` (config/service/board→printer/terminal caps/git/font); shim recipes (`vim`, `jetbrains`, `vscode`, `xcode`); cross-compile binaries (macOS/Linux/Windows), bundle font | M4, M6 | `doctor` flags a broken config; each shim fires `dp` with correct args; binaries run on all three OSes |

## Milestones (Track B — C4 print service)

| # | Milestone | Depends on | Verification checkpoint |
|---|-----------|:----------:|-------------------------|
| **CB0** | **Skeleton + registry** — FastAPI app, board→printer registry config (`board-id → {printer_ip, model, media}`), `GET /boards/{id}` returning `{printer, status}` | — | `GET /boards/floor3-payments` returns the mapped printer |
| **CB1** | **Print path** — `POST /print` (multipart PNG + board + meta) → look up printer → PNG → QL raster (102 mm continuous, auto-cut) → `brother_ql` over the network | CB0 | **End-to-end print of a posted PNG on a real QL-1110NWB** *(physical gate — needs the printer)* |
| **CB2** | **Auth + status + deploy** — Google OIDC validation (audience + `hd=theodo.com`), label-out / reachability status, containerize & deploy to an internal host | CB1 | Bad/absent token rejected; unregistered board rejected; container runs on the internal host; `dp doctor` reads real status |

## Parallelism

Critical path: **M0 → M1 → M2 → M6 → M7.** Once **M1** lands, **M3, M4, M5** are
independent and can proceed concurrently (different devs / different sessions).
**Track B (CB0 → CB1 → CB2)** runs alongside the whole thing; Track A uses the
mock stub until the **M6 ↔ CB1** integration. Shim recipes (M7) can be drafted
anytime; they're only *tested* once M1–M6 work.

## Risks & mitigations

| ID | Risk | Mitigation |
|----|------|------------|
| R1 | **300 dpi small-size legibility** (font/size) — the physical unknown | M1's early print is a gate; if 8 pt JetBrains Mono isn't legible, swap font / adjust size+budget *before* goldens bake it in |
| R2 | **Rust code→PNG fidelity** (mono metrics, no shaping) | reuse silicon's approach; pin font + syntect grammar/theme versions for deterministic goldens; ligatures-off sidesteps shaping |
| R3 | **Google OAuth loopback across 3 OSes** (browser launch, localhost redirect, keychain — esp. Windows) | maintained `oauth2` crate; device-code fallback if loopback is awkward; `keyring` with config-file fallback |
| R4 | **Terminal graphics detection** varies (weakest on older Windows consoles) | robust fallback chain inline → text-grid → OS viewer; never hang when stdout isn't a TTY |
| R5 | **CLI↔C4 contract drift** | same owner; keep the mock stub authoritative to the SPEC interface and in the test suite |
| R6 | **Overflow abort annoyance** if snippets often overflow | clear "N over, narrow --lines" message; monitor at pilot (the F1×F3 tension is already logged in DESIGN §8) |
| R7 | **Printer IP churn** — DHCP reassigns the QL's address, rotting the registry (home test *and* prod) | give each printer a **static IP / DHCP reservation**; registry keys on it. In prod the service host must also route to each board's printer subnet |

## Definition of done (v1)

All SPEC success criteria pass; `dp` builds/runs on macOS+Linux+Windows; a real
selection in Vim/JetBrains prints a legible, correctly-marked, back-linked card
at the right board via C4; `dp doctor` is a working first line of support.
