# Rollback Points — EurinHash OpenCode

## Rollback Strategy
Rollback is performed in layers: **release tag → OpenCode runtime → TUI/plugin → data/config**. The goal is to restore service with the smallest possible change.

## Rollback Points
| Point | What to restore | Command / action | Risk |
|-------|-----------------|------------------|------|
| **RP‑1** | Pre‑release code state | `git revert <commit>` or `git checkout v0.3.0` | Low |
| **RP‑2** | OpenCode runtime version | Downgrade from `1.18.31` to `1.18.30` | Medium |
| **RP‑3** | TUI plugin activation | `api.plugins.deactivate("eurinhash")` | Medium |
| **RP‑4** | TUI slots | Stop registering custom slots / remove `TuiSlotPlugin` entries | Medium |
| **RP‑5** | Keymap layer | Disable custom `api.keymap.registerLayer()` bindings | Low |
| **RP‑6** | Theme overrides | Revert to upstream/default theme | Low |
| **RP‑7** | Config changes | Restore previous `opencode.jsonc` snapshot | Medium |
| **RP‑8** | Dependency lock | Restore previous `package-lock.json` / node_modules cache | Low |
| **RP‑9** | Web release | Redeploy previous Vercel production deployment | Low |

## TUI Rollback Procedure
1. **Deactivate custom plugins**:
   ```ts
   await api.plugins.deactivate("eurinhash");
   ```
2. **Remove custom slot registrations** and keymap overrides.
3. **Reload the TUI** with the previous upstream package (see `docs/release/upstream-sync.md`).
4. **Verify** all `TuiPluginApi` entry points.

## OpenCode Rollback Procedure
1. Reinstall the previous OpenCode binary (`1.18.30`).
2. Re‑validate `@opencode-ai/plugin/dist/tui.d.ts` against the runtime.
3. Run `npm run validate`.
4. Restart the OpenCode runtime and smoke test a session.

## Rollback Triggers
Rollback immediately if any of the following occur:
- TUI does not boot.
- Slots or keymaps stop rendering.
- `TuiPluginApi` signature changes break plugin builds.
- `@opentui/*` packages disappear or are renamed.
- OpenCode runtime cannot load the plugin.

## References
- `docs/architecture/upstream-migration.md` §7 — rollback strategy
- `docs/architecture/tui-sdk-boundary.md` §6 — TUI failure semantics
- `docs/release/upstream-sync.md` — sync procedure
- `docs/release/release-plan.md` — release checklist
