# Package Map — Dépendances et Packages

## Packages du repo hashcode_reboot

| Package | Version | Rôle |
|---------|---------|------|
| `next` | ^16.1.1 | Framework web |
| `react` / `react-dom` | ^19.0.0 | UI |
| `@prisma/client` | ^6.11.1 | ORM |
| `prisma` | ^6.11.1 | CLI ORM |
| `@upstash/ratelimit` | ^2.0.8 | Rate limiting |
| `@upstash/redis` | ^1.38.4 | Cache Redis |
| `@sentry/nextjs` | ^10.73.0 | Error tracking |
| `@vercel/speed-insights` | ^2.0.0 | Performance |
| `@tanstack/react-query` | ^5.82.0 | Server state |
| `@tanstack/react-table` | ^8.21.3 | Table UI |
| `zustand` | ^5.0.6 | Client state |
| `resend` | ^6.27.0 | Email |
| `zod` | ^4.0.2 | Validation |
| `@radix-ui/*` | ^1.1.x | Composables UI |
| `lucide-react` | ^0.525.0 | Icônes |
| `framer-motion` | ^12.23.2 | Animation |
| `tailwindcss` | ^4 | CSS |
| `typescript` | ^5 | Langage |

## Packages OpenCode runtime

| Package | Chemin local | Rôle |
|---------|-------------|------|
| `@opencode-ai/plugin` | `~/.config/opencode/node_modules/@opencode-ai/plugin` | Plugin SDK (server + TUI) |
| `@opencode-ai/sdk` | `~/.config/opencode/node_modules/@opencode-ai/sdk` | SDK client/serveur |
| `@opentui/core` | Dépendance de @opencode-ai/plugin | Moteur TUI (CliRenderer, KeyEvent, Renderable, SlotMode) |
| `@opentui/keymap` | Dépendance de @opencode-ai/plugin | Système de keybinds (Binding, Keymap) |
| `@opentui/solid` | Dépendance de @opencode-ai/plugin | Rendu JSX Solid pour TUI |
| `@opentui/keymap/extras` | Dépendance de @opentui/keymap | Extras keybinds (BindingConfig, SequenceBindingLike) |

## Types @opencode-ai/sdk

### `@opencode-ai/sdk/dist/index.d.ts`
- Exporte `createOpencode()` — crée un serveur + client
- Exporte `createOpencodeClient()` — crée un client seul
- Exporte `OpencodeClient`, `Config` (OpencodeClientConfig)
- Re-exporte de `./client.js` et `./server.js`

### `@opencode-ai/sdk/dist/v2/index.d.ts`
- Exporte `createOpencode()` — version v2 du serveur
- Exporte `createOpencodeClient()` — client v2
- Exporte `data` comme namespace (`export * as data from "./data.js"`)
- Re-exporte de `./client.js` et `./server.js`

### `@opencode-ai/sdk/dist/v2/client.d.ts`
- `OpencodeClient` — client API principal
- `createOpencodeClient(config?)` — factory
- Types : `Config`, `OpencodeClientConfig`
- Exporte depuis `./gen/types.gen.js`

### `@opencode-ai/sdk/dist/v2/data.d.ts`
- `message.user()` — envoi d'un message utilisateur
- Types : `UserMessage`, `Part`

### `@opencode-ai/sdk/dist/gen/types.gen.d.ts`
- Contient tous les types du SDK : `Event`, `Session`, `Message`, `Part`, `Provider`, `Model`, `Agent`, `Permission`, `Todo`, `File`, `Config`, `Path`, `VcsInfo`, etc.
- Types d'événements : `EventMessageUpdated`, `EventSessionStatus`, `EventPermissionUpdated`, etc.
- Types de configuration : `KeybindsConfig`, `AgentConfig`, `ProviderConfig`, `McpLocalConfig`, `McpRemoteConfig`
- Types d'erreur : `BadRequestError`, `NotFoundError`, `ProviderAuthError`, `ApiError`
- Types d'authentification : `OAuth`, `ApiAuth`, `WellKnownAuth`, `Auth`
- Types API REST : toutes les requêtes/réponses `/session`, `/config`, `/pty`, `/project`, `/path`, `/vcs`

## Types @opencode-ai/plugin

### `@opencode-ai/plugin/dist/index.d.ts`
- `Plugin` — type principal : `(input: PluginInput, options?: PluginOptions) => Promise<Hooks>`
- `PluginModule` — `{ id?: string; server: Plugin }` (server-only)
- `PluginInput` — `{ client, project, directory, worktree, experimental_workspace, serverUrl, $ }`
- `Hooks` — interface complète de tous les hooks :
  - `event`, `config`, `dispose`
  - `tool.*` (before, after)
  - `chat.message`, `chat.params`, `chat.headers`
  - `permission.ask`
  - `command.execute.before`
  - `shell.env`
  - `experimental.session.compacting`
  - `experimental.compaction.autocontinue`
  - `experimental.text.complete`
  - `tool.definition`
  - `experimental.chat.messages.transform`
  - `experimental.chat.system.transform`
  - `experimental.provider.small_model`
- `AuthHook`, `ProviderHook`, `ProviderHookContext`
- `WorkspaceAdapter`, `WorkspaceInfo`, `WorkspaceTarget`
- `Config`, `PluginOptions`, `ProviderContext`
- `ToolDefinition`

### `@opencode-ai/plugin/dist/tui.d.ts`
- `TuiPlugin` — `(api: TuiPluginApi, options, meta) => Promise<void>`
- `TuiPluginModule` — `{ id?: string; tui: TuiPlugin }` (TUI-only)
- `TuiPluginApi` — API complète du plugin TUI :
  - `app`, `attention`, `command` (deprecated), `keys`, `keymap`, `mode`
  - `route` (register, navigate, current)
  - `ui` (Dialog, DialogAlert, DialogConfirm, DialogPrompt, DialogSelect, Slot, Prompt, toast, dialog)
  - `tuiConfig`, `kv`, `state`, `theme`, `client`, `event`, `renderer`, `slots`, `plugins`, `lifecycle`
- `TuiState` — accès aux sessions, messages, parts, permissions, questions, LSP, MCP
- `TuiTheme`, `TuiThemeCurrent` — système de thèmes RGBA complet
- `TuiKV` — key-value store persistent
- `TuiAttention` — notifications et sons
- `TuiDialogStack`, `TuiDialogProps` — système de dialogs
- `TuiSlotMap`, `TuiSlotPlugin`, `TuiSlotProps` — système de slots UI
- `TuiEventBus` — event bus typed
- `TuiPluginEntry`, `TuiPluginStatus`, `TuiPluginMeta` — gestion plugins
- `TuiRouteCurrent`, `TuiRouteDefinition` — routage TUI
- `TuiKeys`, `TuiKeymap`, `TuiModeApi` — keybindings et modes
- `TuiCommand`, `TuiCommandApi` (deprecated)
- `TuiPromptRef`, `TuiPromptProps`, `TuiPromptInfo` — prompt input
- `TuiToast`, `TuiAttentionNotifyInput` — notifications
- `TuiLifecycle` — signal AbortSignal + onDispose

## Plugins v2 effect (namespace interne)

`@opencode-ai/plugin/dist/v2/effect/` contient le système d'effets Effect/Ts-Effect :

| Module | Interface | Rôle |
|--------|-----------|------|
| `agent.d.ts` | `AgentHooks`, `AgentDraft` | Gestion agents v2 |
| `catalog.d.ts` | `CatalogHooks`, `CatalogDraft` | Gestion providers/models catalog |
| `command.d.ts` | `CommandHooks`, `CommandDraft` | Gestion commands v2 |
| `context.d.ts` | `PluginContext` | Contexte plugin unifié |
| `event.d.ts` | `Event`, `EventMap` | Système d'événements type-safe |
| `filesystem.d.ts` | `FileSystem` | Opérations fichier (read, list, find, glob) |
| `integration.d.ts` | `IntegrationHooks` | Intégrations |
| `path.d.ts` | `Path` | Chemins (home, data, cache, config, state, temp) |
| `plugin.d.ts` | `Plugin`, `PluginDomain`, `define()` | Plugin v2 avec Effect |
| `reference.d.ts` | `ReferenceHooks` | Références |
| `registration.d.ts` | `Registration`, `Reload`, `Hooks<T>` | Système d'enregistrement/reload |
| `skill.d.ts` | `SkillHooks`, `SkillDraft` | Skills v2 |

## Fait vs Hypothèse

| Fait confirmé | Hypothèse |
|---------------|-----------|
| @opencode-ai/sdk v2 a une API client+serveur séparée | Les types v2/effect/* sont un système interne en cours de stabilisation |
| better-compact n'est pas dans le node_modules local de hashcode_reboot | better-compact est installé dans ~/.config/opencode/node_modules/ |
| Le TUI utilise le framework @opentui (Solid-based) | @opentui/core est un package externe non vérifié localement |
| Les plugins locaux (.ts) utilisent le type `Plugin` de @opencode-ai/plugin | Les plugins npm ne sont pas vérifiés (pas dans node_modules local) |

## Dépendances critiques pour la migration TUI

Le TUI est en cours d'extraction vers `@opencode-ai/tui`. Les dépendances clés :
- `@opentui/core` → sera renommé/remplacé
- `@opentui/keymap` → sera renommé/remplacé
- `@opentui/solid` → sera renommé/remplacé
- `@opencode-ai/plugin/dist/tui.d.ts` → sera déplacé dans @opencode-ai/tui
- Le `TuiPluginApi` est le contrat principal que tout plugin TUI doit implémenter
