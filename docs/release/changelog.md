# Changelog

## v0.3.1 – Release 2026‑09‑19

### Added
- **Upstream TUI sync** – Mapping of `packages/opencode/src/cli/cmd/tui` → `packages/tui` / `@opencode-ai/tui` (see `docs/release/upstream-sync.md`).
- **Rollback strategy** – Defined three rollback points (pre‑release tag, OpenCode version downgrade, TUI plugin de‑activation) (see `docs/release/rollback-points.md`).
- **Documentation** – Created `release-plan.md`, `documentation.md`, `known-limitations.md`, `rollback-points.md`.
- **CI improvements** – Updated `npm run validate` to include OpenCode version check and TUI type‑check.

### Fixed
- **TUI migration warnings** – Updated `TuiPluginApi` type signatures to reflect upcoming `@opencode-ai/tui` contract.
- **OpenCode runtime alignment** – Ensured `OpenCode` version stays at `1.18.31` while the TUI contracts evolve independently.

### Deprecated / Removed
- None (no breaking changes introduced).

### Safety
- All new TUI entry points now go through `TuiPluginApi` (see `docs/architecture/upstream-migration.md`).
- Backward‑compatible fallback to legacy `TuiPluginApi` kept for existing integrations.

## Previous Releases
- **v0.3.0** – Initial release of the web‑app (Next.js 16.1.1) with OpenCode 1.18.31.
- **v0.2.0** – Minor UI polish, added `TuiSlotPlugin` support.
- **v0.1.0** – First public release.

## Known Limitations
- **TUI migration** – The TUI is still being extracted to `@opencode-ai/tui`; some internal packages may change before the migration completes.
- **OpenCode version** – The binary remains locked at `1.18.31`; any future upgrade requires a coordinated release.
- **External plugins** – Third‑party plugins (e.g., `better-compact`) are not managed internally; they may be removed without notice.
- **Environment variables** – `.env` files are excluded from the repo; ensure secret keys are stored securely in the CI environment.

## Future Roadmap
- Full migration of the TUI to `@opencode-ai/tui` (target Q2 2026).
- Add support for new OpenCode features (e.g., `agent.invoked` hooks) in the next release.
- Continuous monitoring of OpenCode’s `upstream-migration` checklist for compatibility breaks.
