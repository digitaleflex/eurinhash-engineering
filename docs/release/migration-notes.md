# Migration Notes — TUI Extraction to `@opencode-ai/tui`

## Context
The OpenCode TUI source at `packages/opencode/src/cli/cmd/tui` is being migrated to the new `packages/tui` package, published as `@opencode-ai/tui`. These notes cover what must change, what stays, and how to validate compatibility.

## Prerequisites (per `docs/architecture/upstream-migration.md`)
- OpenCode runtime version: `1.18.31`.
- Plugin API version: `1.18.30` (patch gap → compatible).
- Never modify the OpenCode binary.
- Import types only via `@opencode-ai/plugin/dist/tui.d.ts`.
- No direct imports from `@opentui/*`.

## Migration Scope
| Component | Action | Risk |
|-----------|--------|------|
| `packages/opencode/src/cli/cmd/tui` | Extract to `packages/tui` | Medium |
| `@opentui/core` | Provided by runtime | Low (runtime-bound) |
| `@opentui/keymap` | Provided by runtime | Low (runtime-bound) |
| `@opentui/solid` | Provided by runtime | Low (runtime-bound) |
| `TuiPluginApi` | Stable; remains the public contract | Low |
| `api.command` | **Deprecated** → use `api.keymap.registerLayer()` | Already migrated |
| `experimental.*` hooks | **Unstable** — track upstream changes | High |
| `@opencode-ai/plugin/dist/tui.d.ts` | Future home: `@opencode-ai/tui` | Medium |

## Migration Steps
1. **Create `packages/tui`** with `package.json` name `@opencode-ai/tui`.
2. **Re-export** TUI entry points from `packages/opencode/src/cli/cmd/tui` (shim) so existing consumers keep compiling.
3. **Move stable TUI code** into `packages/tui/src`.
4. **Re‑apply HashCode overrides** (slots, keymap, theme) on top of the upstream code — do not overwrite them.
5. **Update type imports**:
   ```ts
   // ✅ Correct after migration
   import { TuiPluginApi, TuiState } from "@opencode-ai/tui";

   // ❌ Deprecated
   import { TuiPluginApi } from "@opencode-ai/plugin/dist/tui.d.ts";
   ```
6. **Verify** with `npm run validate` and `tsc --noEmit`.
7. **Update recon**: `docs/recon/package-map.md`, `docs/recon/tui-map.md`, `docs/release/upstream-sync.md`.

## Regression Points
- Confirm all `TuiSlotPlugin` slots still render correctly.
- Confirm all `api.event.on(...)` subscriptions fire.
- Confirm custom keymap layers override upstream defaults.
- Confirm `api.lifecycle.onDispose` cleanup runs after TUI reload.

## Compatibility Policy
- **Patch versions** (1.18.x → 1.18.y): compatible, no migration needed.
- **Minor versions** (1.18 → 1.19): review type changes in `Event` union and `TuiPluginApi`.
- **Major versions** (1.x → 2.x): full re-read of plugin API + ADR for all breaking changes.

## Open Questions (from `docs/recon/open-questions.md`)
- Is the TUI source fully inside `@opencode-ai/tui` or still bridged from `@opencode-ai/opencode`?
- Are `@opentui/*` packages still required after migration?
- What is the timeline for Q2 2026 target?

## References
- `docs/architecture/upstream-migration.md` §4 — migration strategy
- `docs/architecture/tui-sdk-boundary.md` §5 — interface contracts
- `docs/recon/package-map.md` §147-151 — dependencies
- `docs/release/upstream-sync.md` — sync procedure
