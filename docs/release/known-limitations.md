# Known Limitations — Release v0.3.x

## Upstream TUI Migration
- **TUI not yet in `@opencode-ai/tui`**: The extraction from `packages/opencode/src/cli/cmd/tui` → `packages/tui` / `@opencode-ai/tui` is still in progress (`docs/recon/open-questions.md` Q19, Q27). Until finished, the local workspace references the upstream source tree, not a local `packages/` directory.
- **`@opentui/*` packages are runtime‑only**: They are provided by the OpenCode binary; they are not in the local `node_modules` (`docs/architecture/upstream-migration.md` §2.3).
- **`experimental.*` hooks are unstable**: May change between upstream versions (`docs/architecture/upstream-migration.md` §3.2).

## OpenCode Runtime
- **Version locked at `1.18.31`**: Upgrades require coordinated releases (`docs/architecture/upstream-migration.md` §1.1).
- **Compiled binary only**: No access to upstream TS sources; debugging relies on `.d.ts` types and plugin API.
- **Plugin version gap**: Plugin API `1.18.30` vs runtime `1.18.31`; patch‑compatible but must be re‑validated on any minor bump.

## Plugin Ecosystem
- **External plugins are unmanaged**: `better-compact`, `opencode-mem`, `@bluelovers/opencode-arise`, etc., can be updated or removed without notice (`docs/recon/plugin-map.md` §133-145).
- **Windows fsync bug (better-compact)**: Confirmed issue; mitigated by a local npm patch (`docs/recon/risk-register.md` R2).
- **Server plugins only**: Current local plugins (auto‑compact, guard, audit‑logger) are server‑side; no TUI plugin coverage yet.

## Environment & Secrets
- **`.env` files excluded from repo**: Keys are stored in `~/.config/opencode/`; ensure CI secrets are configured (`docs/recon/risk-register.md` R4).
- **PostgreSQL MCP disabled**: `P1000` auth error forces `enabled: false` (`docs/recon/risk-register.md` R3).

## Observability & Telemetry
- **Missing runtime metrics**: Latency, queue and throughput are not exposed by the SDK; approximations use timestamps (`docs/architecture/tui-sdk-boundary.md` §4).
- **`agent.invoked` hook not in current SDK**: `audit-logger.ts` property is ignored by the runtime.

## Agent / Worker Reliability
- **FREE model quotas**: Strict daily limits on Gemini (20 req/day) and OpenRouter (~3 req/day); fallback chain required (`docs/recon/risk-register.md` R1).
- **`free-probe.py` latency**: Sequential HTTP probes with 1.5 s sleeps can take 15‑20 s (`docs/recon/risk-register.md` R8).

## What Is NOT Supported Yet
- **Hot‑reload of TUI plugins**: The upstream behavior is not documented in the current `.d.ts`.
- **Remote workspaces**: `WorkspaceAdapter` support is theoretical only.

## Mitigations
1. Keep the upstream‑sync procedure (`docs/release/upstream-sync.md`) for every TUI update.
2. Run `npm run validate` before any release.
3. Use rollback points defined in `docs/release/rollback-points.md`.
4. Monitor `docs/recon/open-questions.md` and `docs/recon/risk-register.md` for status changes.
