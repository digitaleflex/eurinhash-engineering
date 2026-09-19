# Upstream Sync — `packages/opencode/src/cli/cmd/tui` → `packages/tui` / `@opencode-ai/tui`

## Purpose
This document describes the **incremental upstream synchronization** process for the OpenCode TUI, which is being extracted from `packages/opencode/src/cli/cmd/tui` into the new `packages/tui` package (published as `@opencode-ai/tui`). The goal is to pull upstream improvements **without overwriting HashCode customizations**.

## Current State (as of v0.3.1)
| Item | Status |
|------|--------|
| OpenCode runtime version | `1.18.31` (`docs/architecture/upstream-migration.md`) |
| `@opentui/*` packages | Provided by OpenCode runtime (not in local `node_modules`) |
| `@opencode-ai/plugin/dist/tui.d.ts` | Stable type contract |
| TUI source location | `packages/opencode/src/cli/cmd/tui` (monorepo layout) |
| TUI target package | `packages/tui` → `@opencode-ai/tui` (migration in progress) |
| Customized TUI code | Slot overrides, theme extensions, custom keymaps (see `docs/recon/extension-points.md`) |

## Source Mapping
```
packages/opencode/src/cli/cmd/tui
├── cmd/
├── slots/          ← HashCode overrides (do NOT overwrite)
├── keymap/         ← Custom keybindings (review before sync)
├── theme/          ← Custom theme extensions (review before sync)
└── ...
```
These map to the new package layout:
```
packages/tui
├── src/
│   ├── index.ts       (entry point → @opencode-ai/tui)
│   ├── cmd/
│   ├── slots/
│   ├── keymap/
│   └── theme/
└── package.json       (name: @opencode-ai/tui)
```

## Before Every Sync — Checklist
1. **Inspect upstream changes** touching TUI, SDK, plugins, events and APIs.
2. **Identify conflicts** with HashCode customizations.
3. **Classify** changes:
   - **Safe merge** (no custom override) → pull directly.
   - **Requires adaptation** (signature change, e.g. `TuiPluginApi`) → update types, recompile.
   - **Requires architectural review** (`experimental.*` hooks, `@opentui/*` renames) → file an ADR.
4. **Update recon/ADRs** (`docs/recon/`, `docs/adr/`) when assumptions change.
5. **Merge incrementally**; one PR per concern.
6. **Run regression tests** — TUI type check + downstream plugin builds.

## Do NOT Overwrite Customized Code
- `slots/` contains HashCode slot overrides. Merge upstream only around these files.
- `keymap/` customizations must be re-reconciled with `api.keymap.registerLayer()` changes (see `docs/architecture/tui-sdk-boundary.md` §3.2).
- `theme/` extensions must be re-validated against `@opentui/solid` / `@opencode-ai/tui` theme contract before adopting upstream theme code.

## Regression Gate
| Check | Command |
|-------|---------|
| TUI type check | `tsc --noEmit` (against `@opencode-ai/tui` / `tui.d.ts`) |
| Plugin build | `tsc --noEmit` on each TuiPlugin |
| Slot tests | regression suite in `tests/integration.test.cjs` |
| Keymap smoke | `npm run validate` |

## Sync Conflicts Handling
1. Pause auto-merge for `packages/tui/**`.
2. Review the diff against `docs/recon/tui-map.md`.
3. If upstream breaks `TuiPluginApi` → treat as **architectural review**.
4. Commit only after the upstream-sync checklist above is green.

## References
- `docs/architecture/upstream-migration.md` — upstream migration constraints
- `docs/architecture/tui-sdk-boundary.md` — TUI/SDK boundary rules
- `docs/recon/package-map.md` §147-151 — `@opentui/*` / `@opencode-ai/tui` mapping
- `docs/recon/tui-map.md` — full TUI map incl. migration plan
- `docs/recon/extension-points.md` — slots, keymap, theme extension points
- `docs/recon/open-questions.md` Q19 / Q27 — TUI migration status to investigate
