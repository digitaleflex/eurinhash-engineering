# Personalization (Phase 08)

## Personalization domains

- **layout mode** – Choose among `MINIMAL`, `STANDARD`, `DEVELOPER`, `OBSERVER`, `DEBUG`. Exactly five modes; `UNKNOWN` is **not** treated as `ZERO`; it means “no mode set” and is handled separately.
- **density** – Adjustable spacing/tightness of UI elements. Controlled via the same five levels (`MINIMAL` → `DEBUG`) as layout mode; see `docs/interaction/density-control.md`.
- **sidebar visibility** – Show/hide the sidebar; preference persisted per‑session.
- **telemetry visibility** – Toggle display of usage telemetry bars or logs.
- **activity verbosity** – Amount of detail shown for recent activity (`minimal`, `standard`, `verbose`).
- **theme** – Light/dark or custom color palette; stored in user preferences.
- **keybindings** – Per‑user mapping of key combinations to actions. Keymap follows `@opentui/keymap` conventions; unknown keys are rendered as `[? UNKNOWN]` rather than silently becoming `0`.
- **command aliases** – User‑defined shortcuts or aliases for frequently used commands.
- **notifications/attention behavior** – Configure how and when transient notifications appear (persistent, ephemeral, disabled).
- **persistent UI preferences** – All of the above are stored in the existing persistence layer (extended `opencode.jsonc`) and never leak into domain logic.

## Interaction rules

- Commands must have predictable focus behavior; opening the command palette always puts focus on the search field.
- `Escape`/back navigation must return focus to the previous UI region without side‑effects.
- Long‑running operations display a visible progress indicator; cancellation is allowed only after explicit confirmation.
- Dangerous actions (e.g., resetting layout, deleting preferences) require a confirmation modal that references the current preference state.
- User preferences are scoped to the individual user session and are never written to shared domain models.
- Invalid configuration defaults to safe fallbacks; the system never crashes on an unknown mode or density value.

## Persistence

- Existing persistence facilities (`opencode.jsonc`) are extended rather than replaced.
- Each personalization domain maps to a key in the store:
  - `layout.mode` → one of the five modes
  - `density.level` → one of `MINIMAL|STANDARD|DEVELOPER|OBSERVER|DEBUG`
  - `sidebar.visible` → boolean
  - `telemetry.visible` → boolean
  - `activity.verbosity` → `minimal|standard|verbose`
  - `theme` → string name or hex palette
  - `keybindings` → JSON object mapping stroke → action
  - `command.aliases` → object of alias → command name
  - `notifications.enabled` → boolean

Changes are saved on exit or when the user explicitly applies them; unsaved changes are hinted on shutdown.

## Command palette integration

The command palette (see `docs/interaction/command-palette.md`) coexists with the existing OpenCode keymap. All personalization actions (change layout, toggle density, etc.) are reachable via the palette, and their bindings are registered through `api.keymap.registerLayer()` so they do not conflict with core shortcuts. The palette respects the `UNKNOWN != ZERO` rule: if a key is not mapped, it shows `[? UNKNOWN]` and does not fall back to a default action.

## Links

- `docs/interaction/keyboard-navigation.md` – detailed keymap definitions and conventions.
- `docs/interaction/layout-modes-config.md` – layout‑mode specifics (MINIMAL/STANDARD/DEVELOPER/OBSERVER/DEBUG).
- `docs/interaction/density-control.md` – density adjustment UI and presets.
- `docs/interaction/preferences.md` – general user‑preference UI.
- `docs/interaction/command-palette.md` – keyboard‑first command palette.