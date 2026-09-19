# Documentation – Source of Truth

This document acts as the central reference for every user‑facing configuration option, layout mode, command, keybinding and telemetry field in the current release.

## Project Overview
- **Application**: Next.js (React 19 + Tailwind CSS v4 + shadcn/ui)
- **Runtime**: OpenCode 1.18.31 (`docs/architecture/upstream-migration.md`)
- **TUI target**: `@opencode-ai/tui` (extracted from `packages/opencode/src/cli/cmd/tui` → `packages/tui`)
- **Version**: `0.3.0` (may be bumped on release)

## Public API Surface
- **Web app**: Next.js routes under `src/app/` (public pages, admin panel, API routes).
- **TUI layer**: `TuiPluginApi` contract (see `docs/architecture/tui-sdk-boundary.md`).
- **Plugin contracts**: `Plugin` (server side), `TuiPlugin` (TUI side) – full listing in `docs/recon/plugin-map.md`.

## Layout Modes
| Mode | Description | Config |
|------|-------------|--------|
| Default | Standard web dashboard | `src/app/(default)/layout.tsx` |
| Admin | Protected admin panel | `src/app/admin/layout.tsx` |
| TUI | Terminal UI via `@opencode-ai/tui` | `packages/opencode/src/cli/cmd/tui` |

## Commands & Keybindings
- **Web**: Standard Next.js navigation, server actions, and API routes.
- **TUI**: Keymap registered via `api.keymap.registerLayer()` (see `docs/architecture/tui-sdk-boundary.md` §3.2 and `docs/recon/extension-points.md`).

## Telemetry & Observability
- **Error tracking**: `@sentry/nextjs` (configured in `package.json` dependencies).
- **Performance**: `@vercel/speed-insights`.
- **TUI events**: `api.event.on(...)` subscriptions (see `docs/recon/extension-points.md`).
- **Privacy**: See `docs/observability/privacy.md`.

## Documentation Directory Map
| Path | Content |
|------|---------|
| `docs/architecture/` | Architecture decisions, TUI/SDK boundary, upstream migration |
| `docs/recon/` | Maps (package, plugin, extension, risk, test) |
| `docs/state/` | State machine and failure semantics |
| `docs/observability/` | Metrics, performance, privacy, presentation |
| `docs/release/` | Release plan, changelog, upstream sync, rollback, etc. |
| `docs/qa/` | Test strategy |
| `docs/ux/` and `docs/ui/` | UX and UI guidelines |

## Outdated Documents
- None (all Phase 1‑10 documents are up to date as of 2026‑09‑19).
