# Plugin Map — Architecture des Plugins

## Vue d'ensemble

OpenCode utilise un système de plugins à deux niveaux :
1. **Server plugins** (`PluginModule` avec `server`) — exécutés côté serveur
2. **TUI plugins** (`TuiPluginModule` avec `tui`) — exécutés côté interface terminal

Les plugins sont configurés dans `opencode.jsonc` via le champ `"plugin"` (array de strings/noms/packages/chemins locaux).

## Contrat Plugin (Server)

### Type `Plugin` (@opencode-ai/plugin/dist/index.d.ts)

```typescript
type Plugin = (input: PluginInput, options?: PluginOptions) => Promise<Hooks>;

interface PluginInput {
    client: ReturnType<typeof createOpencodeClient>;
    project: Project;
    directory: string;
    worktree: string;
    experimental_workspace: {
        register(type: string, adapter: WorkspaceAdapter): void;
    };
    serverUrl: URL;
    $: BunShell;
}

interface Hooks {
    dispose?: () => Promise<void>;
    event?: (input: { event: Event }) => Promise<void>;
    config?: (input: Config) => Promise<void>;
    tool?: { [key: string]: ToolDefinition };
    auth?: AuthHook;
    provider?: ProviderHook;
    "chat.message"?: (input, output) => Promise<void>;
    "chat.params"?: (input, output) => Promise<void>;
    "chat.headers"?: (input, output) => Promise<void>;
    "permission.ask"?: (input: Permission, output) => Promise<void>;
    "command.execute.before"?: (input, output) => Promise<void>;
    "tool.execute.before"?: (input, output) => Promise<void>;
    "shell.env"?: (input, output) => Promise<void>;
    "tool.execute.after"?: (input, output) => Promise<void>;
    "experimental.chat.messages.transform"?: (input, output) => Promise<void>;
    "experimental.chat.system.transform"?: (input, output) => Promise<void>;
    "experimental.provider.small_model"?: (input, output) => Promise<void>;
    "experimental.session.compacting"?: (input, output) => Promise<void>;
    "experimental.compaction.autocontinue"?: (input, output) => Promise<void>;
    "experimental.text.complete"?: (input, output) => Promise<void>;
    "tool.definition"?: (input, output) => Promise<void>;
}
```

### PluginModule

```typescript
interface PluginModule {
    id?: string;
    server: Plugin;
    tui?: never;
}
```

## Contrat TuiPlugin (TUI)

### Type `TuiPlugin` (@opencode-ai/plugin/dist/tui.d.ts)

```typescript
type TuiPlugin = (api: TuiPluginApi, options: PluginOptions | undefined, meta: TuiPluginMeta) => Promise<void>;

interface TuiPluginModule {
    id?: string;
    tui: TuiPlugin;
    server?: never;
}
```

### TuiPluginApi (API complète du plugin TUI)

```typescript
interface TuiPluginApi {
    readonly version: string;
    app: TuiApp;
    attention: TuiAttention;
    command?: TuiCommandApi;  // @deprecated
    keys: TuiKeys;
    keymap: TuiKeymap;
    mode: TuiModeApi;
    route: { register, navigate, readonly current };
    ui: { Dialog, DialogAlert, DialogConfirm, DialogPrompt, DialogSelect, Slot, Prompt, toast, dialog };
    readonly tuiConfig: Frozen<TuiConfigView>;
    kv: TuiKV;
    state: TuiState;
    theme: TuiTheme;
    client: OpencodeClient;
    event: TuiEventBus;
    renderer: CliRenderer;
    slots: TuiSlots;
    plugins: { list, activate, deactivate, add, install };
    lifecycle: TuiLifecycle;
}
```

### Plugin lifecycle (TUI)

- `TuiPluginState` : `"first" | "updated" | "same"`
- `TuiPluginEntry` : `{ id, source, spec, target, version, modified, first_time, last_time, time_changed, load_count, fingerprint }`
- `TuiPluginStatus` : `{ id, source, spec, target, enabled, active }`

## Plugins installés (opencode.jsonc)

### Plugins locaux (chemin ./plugin/)

| Plugin | Fichier | Hooks | Rôle |
|--------|---------|-------|------|
| auto-compact.ts | `./plugin/auto-compact.ts` | `experimental.session.compacting`, `experimental.compaction.autocontinue` | Compaction intelligente avec directive de préservation contextuelle + reprise auto |
| guard.ts | `./plugin/guard.ts` | `tool.execute.before`, `tool.execute.after` | Blocage commandes destructrices (rm -rf, mkfs, dd, git push --force) + redaction secrets |
| audit-logger.ts | `./plugin/audit-logger.ts` | `tool.execute.before`, `tool.execute.after`, `agent.invoked` (non supporté SDK) | Traçabilité JSONL avec rotation 30j, redaction, flag sensibilité |

### Plugins npm (installés)

| Plugin | Package | Rôle |
|--------|---------|------|
| better-compact | `better-compact` | Pruning par étapes, préservation intention, TUI |
| opencode-mem | `opencode-mem` | Mémoire persistante entre sessions |
| envsitter-guard | `envsitter-guard` | Protection fichiers .env (lecture seule, fingerprints) |
| oh-my-opencode-slim | `oh-my-opencode-slim` | Optimisation agents (modèles FORCÉS sur FREE) |
| opencode-plugin-preload-skills | `opencode-plugin-preload-skills` | Pré-charge les skills au démarrage |
| @bluelovers/opencode-arise | `@bluelovers/opencode-arise` | Orchestrateur parallèle (agents simultanés) |

### Plugins retirés (et pourquoi)

| Plugin | Raison |
|--------|--------|
| DCP | Mauvais nom de paquet (remplacé par better-compact) |
| supermemory | Cloud, doublon de opencode-mem |
| lazy-skills / skillful | Remplacés par preload-skills |
| opencode-router | Bridges inutilisés |
| opencode.nvim | Pas d'usage Neovim |
| oh-my-openagent | Chevauche oh-my-opencode-slim + superviseur |
| context-summarizer | Stub, remplacé par auto-compact.ts |
| opencode-antigravity-multi-auth | Dépôt archivé (2026-09-19) |
| @f97/opencode-morph-fast-apply | Nécessite MORPH_API_KEY |

## Plugins v2 Effect (namespace interne)

Le système `@opencode-ai/plugin/dist/v2/effect/` utilise **Effect-TS** pour un plugin system déclaratif :

| Module | Interface | Fonction |
|--------|-----------|----------|
| `plugin.d.ts` | `Plugin<R>`, `define()`, `PluginDomain` | Plugin avec Effect, add/remove |
| `context.d.ts` | `PluginContext` | Contexte unifié : options, agent, aisdk, catalog, command, integration, plugin, reference, skill |
| `registration.d.ts` | `Registration`, `Reload`, `Hooks<T>` | Système d'enregistrement/reload |
| `event.d.ts` | `Event`, `EventMap` | Subscribe typed aux événements SDK |
| `agent.d.ts` | `AgentHooks`, `AgentDraft` | CRUD agents v2 |
| `catalog.d.ts` | `CatalogHooks`, `CatalogDraft` | CRUD providers/models v2 |
| `command.d.ts` | `CommandHooks`, `CommandDraft` | CRUD commands v2 |
| `skill.d.ts` | `SkillHooks`, `SkillDraft` | CRUD skills v2 |
| `reference.d.ts` | `ReferenceHooks` | Références |
| `filesystem.d.ts` | `FileSystem` | read/list/find/glob |
| `path.d.ts` | `Path` | home/data/cache/config/state/temp |
| `integration.d.ts` | `IntegrationHooks` | Intégrations |
| `aisdk.d.ts` | `AISDKHooks` | AISDK integration |

## Extension points (Points d'extension)

### Server-side hooks (Plugin)
1. `event` — intercepte TOUS les événements
2. `config` — intercepte les changements de configuration
3. `tool.execute.before/after` — intercepte l'exécution d'outils
4. `chat.message` — intercepte les messages chat
5. `chat.params` — modifie les paramètres LLM
6. `chat.headers` — modifie les headers HTTP
7. `permission.ask` — modifie les permissions
8. `command.execute.before` — intercepte les commandes
9. `shell.env` — modifie l'environnement shell
10. `tool.definition` — modifie les définitions d'outils
11. `experimental.session.compacting` — personnalise la compaction
12. `experimental.compaction.autocontinue` — contrôle la reprise auto
13. `experimental.text.complete` — complète le texte
14. `experimental.chat.messages.transform` — transforme les messages
15. `experimental.chat.system.transform` — transforme le système prompt
16. `experimental.provider.small_model` — modifie le petit modèle

### TUI extension points (TuiPluginApi)
1. `route.register` — enregistre des routes TUI personnalisées
2. `slots.register` — enregistre des slots UI personnalisés
3. `keymap` — keybindings personnalisés
4. `ui.Dialog/Alert/Confirm/Prompt/Select/Slot/Prompt` — composants UI
5. `toast` — notifications
6. `plugins.activate/deactivate/add/install` — gestion dynamique des plugins
7. `kv` — stockage persistant
8. `theme` — thèmes personnalisés
9. `event.on` — abonnement aux événements
10. `lifecycle.onDispose` — cleanup
11. `mode.push` — modes personnalisés
12. `attention.notify` — notifications système

## Fait vs Hypothèse

| Fait confirmé | Hypothèse |
|---------------|-----------|
| Le contrat Plugin est clair dans les .d.ts | Le mécanisme de chargement des plugins .ts locaux n'est pas visible dans les types |
| Les plugins v2/effect utilisent Effect-TS | La stabilité et la documentation du système v2/effect est incomplète |
| `better-compact` est un plugin npm installé | Son code source n'est pas dans le node_modules local de hashcode_reboot |
| Les plugins locaux utilisent `satisfies Plugin` | Les chemins relatifs dans opencode.jsonc sont résolus par le runtime |
| `envsitter-guard`, `oh-my-opencode-slim`, etc. sont des packages npm | Leurs sources ne sont pas disponibles localement |

## Plugin development flow

```
Écrire le plugin (.ts ou package)
    │
    ▼
Configurer dans opencode.jsonc ("plugin": [...])
    │
    ▼
Runtime charge et initialise le plugin
    │
    ▼
Plugin retourne ses Hooks
    │
    ▼
Hooks sont appelés aux moments appropriés
    │
    ▼
Plugin peut aussi s'abonner à event ou config
```

## Risques liés aux plugins

1. **better-compact** : bug Windows fsync (EPERM) → nécessite un patch npm
2. **audit-logger.ts** : `agent.invoked` n'est pas dans le SDK actuel → property ignorée
3. **@bluelovers/opencode-arise** : package externe, dépendance non vérifiée localement
4. **oh-my-opencode-slim** : force des modèles, si mal configuré → utilisation de modèles payants
5. **Plugins v2/effect** : système interne en cours de développement, API instable
