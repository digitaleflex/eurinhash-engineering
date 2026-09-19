# Phase 11 — Release Plan

## Versioning Strategy
- **Semantic Versioning** (`MAJOR.MINOR.PATCH`) follows the existing `package.json` version (`0.3.0`).
- **OpenCode Runtime** version is pinned to `1.18.31` (see `docs/architecture/upstream-migration.md`).
- **TUI migration target**: `@opencode-ai/tui` (see upstream migration checklist).

## Release Checklist
| Step | Description | Owner |
|------|-------------|-------|
| 1. Bump version | Update `package.json` `version` field (e.g., `0.3.1`). | Release manager |
| 2. Run `npm run validate` | Type‑check + lint + unit tests. | CI |
| 3. Update `docs/release/` files | Generate `changelog.md`, `documentation.md`, `known-limitations.md`, `rollback-points.md`. | Docs team |
| 4. Tag release | `git tag -a v0.3.1 -m "Release v0.3.1"` then push tag. | Release manager |
| 5. Publish to Vercel | `vercel --prod`. | Ops |
| 6. Sync upstream TUI | Run the upstream‑sync procedure (see `docs/release/upstream-sync.md`) and commit any changes. | Integration team |
| 7. Post‑release verification | Smoke test the UI, verify that all `TuiPluginApi` entry points still resolve. | QA |

## Migration Notes (high‑level)
- **TUI → @opencode-ai/tui**: The TUI code in `packages/opencode/src/cli/cmd/tui` will be moved to `packages/tui` and the package `@opencode-ai/tui`. Refer to `docs/architecture/upstream-migration.md` §4 for compatibility rules and `docs/recon/package-map.md` §147‑151 for dependency mapping.
- **OpenCode binary**: No source changes; only type‑definition updates via `@opencode-ai/plugin/dist/tui.d.ts`.

## Rollback Points
1. **Pre‑release tag** – keep the tag (`v0.3.0`) to revert to the previous version if needed.
2. **OpenCode version rollback** – switch the runtime to `1.18.30` (previous patch) via `api.app.version` guard (see `docs/architecture/upstream-migration.md` §5.1).
3. **TUI plugin de‑activation** – use `api.plugins.deactivate("eurinhash")` to disable custom plugins before rolling back the TUI code.