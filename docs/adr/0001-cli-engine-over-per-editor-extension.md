# CLI engine with per-editor keystroke shims, not a per-editor extension

**Status:** accepted

Dantotsu Print's capture layer is a single `dp` CLI engine, invoked by a thin per-editor keystroke shim (Vim map, JetBrains External Tool, VS Code/Cursor keybinding→task; Xcode falls back to clipboard). We chose this over a native extension per editor because the engineering fleet is polyglot (VS Code, Cursor, JetBrains, Vim, Xcode) and maintaining five extensions would multiply operating burden against F4 — while one engine plus thin shims stays editor-agnostic.

## Considered options

- **Per-editor extensions** (Polacode-style). Richest UX, free git context — but one codebase per editor to maintain.
- **CLI engine + keystroke shims (chosen).** One engine, thin shims; editor-agnostic; the CLI resolves git context itself.
- **Web paste-in.** Zero install, but reintroduces the Notion-like friction this project exists to remove.

## Consequences

- Weaker in-editor preview — mitigated by a cell-accurate terminal/OS preview (F3a), inline in graphics-capable terminals (Ghostty, iTerm2, …).
- Xcode is degraded to clipboard-only (no precise line range).
- Syntax colours come from the CLI's own tokenizer, not the editor's live theme.
- A VS Code/Cursor extension may be added later as a fast-follow for a richer preview — one codebase covers the largest slice of the fleet.
