# Plugin Presentation Model — Modèle de présentation plugin

> **Source** : `ARCHITECTURE_FINDINGS.md`, `plugin/dist/index.d.ts`, `plugin/dist/tui.d.ts`, `plugin/dist/tool.d.ts`
> **Principe** : Les plugins sont le mécanisme d'extension du core OpenCode sans modification du binaire.

---

## 1. Types de plugins

### 1.1 Plugin (server-side)

```typescript
type Plugin = (input: PluginInput, options?: PluginOptions) => Promise<Hooks>;
```

**`PluginInput`** :
```typescript
type PluginInput = {
    client: ReturnType<typeof createOpencodeClient>;
    project: Project;
    directory: string;
    worktree: string;
    experimental_workspace: {
        register(type: string, adapter: WorkspaceAdapter): void;
    };
    serverUrl: URL;
    $: BunShell;
};
```

**`Hooks`** — Interface de hooks côté serveur :
```typescript
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
    "permission.ask"?: (input, output) => Promise<void>;
    "command.execute.before"?: (input, output) => Promise<void>;
    "tool.execute.before"?: (input, output) => Promise<void>;
    "tool.execute.after"?: (input, output) => Promise<void>;
    "shell.env"?: (input, output) => Promise<void>;
    "experimental.chat.messages.transform"?: (input, output) => Promise<void>;
    "experimental.chat.system.transform"?: (input, output) => Promise<void>;
    "experimental.provider.small_model"?: (input, output) => Promise<void>;
    "experimental.session.compacting"?: (input, output) => Promise<void>;
    "experimental.compaction.autocontinue"?: (input, output) => Promise<void>;
    "experimental.text.complete"?: (input, output) => Promise<void>;
    "tool.definition"?: (input, output) => Promise<void>;
}
```

**Contrat** :
- **Input** : `PluginInput` (client, project, directory, worktree, serverUrl, shell)
- **Output** : `Promise<Hooks>` (callbacks de hook)
- **Owner** : Server core
- **Lifecycle** : Chargé au démarrage du serveur, hooks invoqués par le core
- **Failure** : Hook peut modifier les output, le core gère les erreurs

### 1.2 TuiPlugin (TUI-side)

```typescript
type TuiPlugin = (api: TuiPluginApi, options: PluginOptions | undefined, meta: TuiPluginMeta) => Promise<void>;
```

**`TuiPluginApi`** — Interface complète :
```typescript
interface TuiPluginApi {
    app: TuiApp;                              // { readonly version: string }
    attention: TuiAttention;                   // notify, soundboard
    command?: TuiCommandApi;                   // @deprecated
    keys: TuiKeys;                             // formatSequence, formatBindings
    keymap: TuiKeymap;                         // Keymap<Renderable, KeyEvent>
    mode: TuiModeApi;                          // current(), push()
    route: { register, navigate, readonly current };
    ui: { Dialog, DialogAlert, DialogConfirm, DialogPrompt, DialogSelect, Slot, Prompt, toast, dialog };
    readonly tuiConfig: Frozen<TuiConfigView>;
    kv: TuiKV;                                 // get, set
    state: TuiState;                           // readonly, lecture seule
    theme: TuiTheme;                           // current, mode, set, install
    client: OpencodeClient;                    // SDK client complet
    event: TuiEventBus;                        // on(type, handler)
    renderer: CliRenderer;                     // Rendu terminal
    slots: TuiSlots;                           // register(plugin)
    plugins: { list, activate, deactivate, add, install };
    lifecycle: TuiLifecycle;                   // signal, onDispose
}
```

**Contrat** :
- **Input** : `TuiPluginApi` (tout en lecture seule sauf keymap/plugins), `PluginOptions`, `TuiPluginMeta`
- **Output** : `Promise<void>`
- **Owner** : TUI framework
- **Lifecycle** : Chargé au démarrage du TUI, `api.lifecycle.signal` pour le cleanup
- **Failure** : `TuiPluginState` = "same" si pas changé, "updated" si mis à jour

### 1.3 Différence Plugin vs TuiPlugin

| Aspect | Plugin (server) | TuiPlugin |
|--------|----------------|-----------|
| **Exécution** | Côté serveur | Côté TUI |
| **Input** | `PluginInput` | `TuiPluginApi` |
| **Output** | `Promise<Hooks>` | `Promise<void>` |
| **State** | Accès complet au core | Lecture seule via `TuiState` |
| **UI** | Aucun accès direct | Slots, Dialogs, Prompt |
| **Lifecycle** | Server hooks | `TuiLifecycle` |
| **Module** | `{ server: Plugin }` | `{ tui: TuiPlugin }` |

---

## 2. Module plugin

### 2.1 Structure du fichier plugin

```typescript
// Plugin server
export const plugin: PluginModule = {
    id?: string,
    server: async (input, options) => {
        return {
            event: async ({ event }) => { /* ... */ },
            "tool.execute.before": async (input, output) => { /* ... */ },
        };
    }
};

// TuiPlugin
export const plugin: TuiPluginModule = {
    id?: string,
    tui: async (api, options, meta) => {
        // Enregistrer les slots
        api.slots.register({
            id: "eurinhash-status",
            slots: { app_bottom: () => <StatusCompact /> },
        });
        // S'abonner aux événements
        api.event.on("session.status", (event) => { /* ... */ });
        // Cleanup
        api.lifecycle.onDispose(() => { /* ... */ });
    }
};
```

### 2.2 `TuiPluginModule` vs `PluginModule`

```typescript
// Plugin server — exclut tui
type PluginModule = {
    id?: string;
    server: Plugin;
    tui?: never;
};

// TuiPlugin — exclut server
type TuiPluginModule = {
    id?: string;
    tui: TuiPlugin;
    server?: never;
};
```

---

## 3. Système de slots pour la présentation

### 3.1 `TuiSlotPlugin`

```typescript
type TuiSlotPlugin<Slots extends Record<string, object> = {}> = Omit<SlotCore<Slots>, "id"> & {
    id?: never;
};

type SlotCore<Slots extends Record<string, object>> = SolidPlugin<TuiSlotMap<Slots>, TuiSlotContext>;
```

**`TuiSlotContext`** : `{ theme: TuiTheme }` — Le thème est injecté dans chaque slot.

### 3.2 Enregistrement

```typescript
interface TuiSlots {
    register: {
        (plugin: TuiSlotPlugin): string;                    // Utiliser TuiHostSlotMap
        <Slots extends Record<string, object>>(plugin: TuiSlotPlugin<Slots>): string; // Slots personnalisés
    };
}
```

**Contrat** :
- **Input** : `TuiSlotPlugin<Slots>` — Composant SolidJS
- **Output** : `string` — Nom du slot enregistré
- **Owner** : TuiPlugin
- **Lifecycle** : Appelé au démarrage du plugin
- **Failure** : Le slot existant est remplacé

### 3.3 Slots personnalisés vs host slots

Les slots personnalisés étendent `TuiHostSlotMap` :
```typescript
type TuiSlotMap<Slots extends Record<string, object> = {}> = TuiHostSlotMap & Slots;
```

**Slots host** : `app`, `app_bottom`, `home_logo`, `home_prompt`, `home_prompt_right`, `home_bottom`, `home_footer`, `sidebar_title`, `sidebar_content`, `sidebar_footer`, `session_prompt`, `session_prompt_right`

**Slots personnalisés** : N'importe quel nom de string, mais pas un conflit avec les slots host.

---

## 4. Hooks de présentation

### 4.1 `hooks.event`

```typescript
"event"?: (input: { event: Event }) => Promise<void>;
```

**Contrat** :
- **Input** : `Event` (union complète)
- **Output** : `Promise<void>`
- **Owner** : Server plugin
- **Lifecycle** : Invoqué quand un événement est émis
- **Failure** : L'erreur est capturée par le core

### 4.2 `hooks.tool.execute.before/after`

```typescript
"tool.execute.before"?: (input: { tool, sessionID, callID }, output: { args }) => Promise<void>;
"tool.execute.after"?: (input: { tool, sessionID, callID, args }, output: { title, output, metadata }) => Promise<void>;
```

**Contrat** :
- **Input** : Tool name, sessionID, callID, args
- **Output** : Modifiable (args avant, title/output/metadata après)
- **Owner** : Server plugin (ex: guard.ts)
- **Lifecycle** : Avant/après exécution de l'outil
- **Failure** : Le core gère les erreurs

### 4.3 `hooks.permission.ask`

```typescript
"permission.ask"?: (input: Permission, output: { status: "ask" | "deny" | "allow" }) => Promise<void>;
```

**Contrat** :
- **Input** : `Permission` object
- **Output** : `{ status: "ask" | "deny" | "allow" }`
- **Owner** : Server plugin
- **Lifecycle** : Avant exécution d'une action sensible
- **Failure** : Timeout → deny

### 4.4 `hooks.experimental.provider.small_model`

```typescript
"experimental.provider.small_model"?: (input: { provider: ProviderV2 }, output: { model?: ModelV2 }) => Promise<void>;
```

**Contrat** :
- **Input** : `ProviderV2`
- **Output** : `{ model?: ModelV2 }` — Modèle alternatif
- **Owner** : Server plugin (ex: oh-my-opencode-slim)
- **Lifecycle** : Avant sélection du modèle
- **Failure** : Le modèle original est utilisé

### 4.5 `hooks.experimental.session.compacting`

```typescript
"experimental.session.compacting"?: (input: { sessionID }, output: { context: string[], prompt?: string }) => Promise<void>;
```

**Contrat** :
- **Input** : `sessionID`
- **Output** : `{ context: string[], prompt?: string }`
- **Owner** : Server plugin (ex: better-compact)
- **Lifecycle** : Avant compaction
- **Failure** : Compaction avec contexte par défaut

---

## 5. Tool Definition

### 5.1 `tool()` function

```typescript
function tool<Args extends z.ZodRawShape>(input: {
    description: string;
    args: Args;
    execute(args: z.infer<z.ZodObject<Args>>, context: ToolContext): Promise<ToolResult>;
}): { description: string; args: Args; execute: ... };
```

**`ToolContext`** :
```typescript
type ToolContext = {
    sessionID: string;
    messageID: string;
    agent: string;
    directory: string;
    worktree: string;
    abort: AbortSignal;
    metadata(input: { title?, metadata? }): void;
    ask(input: AskInput): Promise<void>;
};
```

**Contrat** :
- **Input** : `description`, `args` (Zod schema), `execute` function
- **Output** : `ToolResult` (string ou `{ title, output, metadata, attachments }`)
- **Owner** : Server plugin
- **Lifecycle** : Appelé quand l'outil est invoqué
- **Failure** : `ToolResult` avec error information

---

## 6. Modèle de présentation pour EurinHash

### 6.1 Architecture de présentation

```
┌──────────────────────────────────────────────────┐
│            EurinHash TuiPlugin                    │
│                                                  │
│  ┌────────────────────────────────────────────┐  │
│  │  TuiPlugin entry point                     │  │
│  │                                            │  │
│  │  api.slots.register({                      │  │
│  │    app_bottom: () => <StatusCompact />,    │  │
│  │    sidebar_footer: ({ session_id }) =>     │  │
│  │      <StatusDetailed sessionID={session_id} />, │  │
│  │    app: () => <StatusDebug />,             │  │
│  │  });                                       │  │
│  │                                            │  │
│  │  api.event.on("session.status", handler);  │  │
│  │  api.event.on("message.part.updated", h);  │  │
│  │  api.event.on("lsp.updated", handler);     │  │
│  │                                            │  │
│  │  api.lifecycle.onDispose(cleanup);         │  │
│  └────────────────────────────────────────────┘  │
│                                                  │
│  ┌────────────────────────────────────────────┐  │
│  │  View-Model (AgentObservabilityState)      │  │
│  │                                            │  │
│  │  ← TuiState (lecture seule)                │  │
│  │  ← Calculs (tokens, coût, durée)           │  │
│  │  ← Événements (mises à jour temps réel)    │  │
│  └────────────────────────────────────────────┘  │
│                                                  │
│  ┌────────────────────────────────────────────┐  │
│  │  Composants SolidJS                        │  │
│  │                                            │  │
│  │  StatusCompact · StatusDetailed · StatusDebug│  │
│  │  ContextMeter · McpStatusList · LspStatusList│  │
│  │  PerformancePanel · SecurityPanel          │  │
│  └────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────┘
```

### 6.2 Plugins existants et leurs rôles

| Plugin | Type | Rôle | Présentation |
|--------|------|------|-------------|
| `better-compact` | TuiPlugin | Pruning contexte | `showToast` + summary sections |
| `auto-compact.ts` | TuiPlugin | Compaction auto | `experimental.session.compacting` |
| `guard.ts` | Plugin server | Sécurité | `tool.execute.before/after`, `permission.ask` |
| `audit-logger.ts` | Plugin server | Traçabilité | Hooks `experimental.*` |
| `envsitter-guard` | TuiPlugin | Protection .env | `tool.execute.before` |
| `opencode-mem` | TuiPlugin | Mémoire persistante | — |
| `oh-my-opencode-slim` | TuiPlugin | Optimisation agents | `provider.small_model` |
| `opencode-arise` | TuiPlugin | Agents parallèles | — |
| `opencode-plugin-preload-skills` | TuiPlugin | Pré-charge skills | — |

### 6.3 Cycle de vie du plugin

```
Chargement:
  1. Plugin trouvé (source: "file" | "npm" | "internal")
  2. TuiPluginState = "first" (premier chargement) ou "updated" (mis à jour)
  3. TuiPlugin entry point invoqué
  4. api.slots.register() → slots enregistrés
  5. api.event.on() → subscriptions actives

Exécution:
  6. Events déclenchées → handlers appelés
  7. Slots rendus dans le terminal
  8. api.ui.toast/dialog → interactions utilisateur
  9. api.keymap.registerLayer() → raccourcis actifs

Cleanup:
  10. api.lifecycle.signal.abort → signal déclenché
  11. api.lifecycle.onDispose(fn) → cleanup exécuté
  12. TuiPluginState = "same" ou "updated" (rechargement)
```

---

## 7. Validation du modèle

```
CHECKLIST — Plugin Presentation Model Validation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
☐ Les plugins utilisent l'API publique @opencode-ai/plugin
☐ TuiState est utilisé en lecture seule uniquement
☐ Les slots sont enregistrés via api.slots.register()
☐ Les événements sont souscrits via api.event.on()
☐ Le cleanup est fait via api.lifecycle.onDispose()
☐ Les hooks server utilisent l'interface Hooks
☐ Les tool definitions utilisent la fonction tool()
☐ Pas de modification du binaire .exe OpenCode
☐ Les types viennent des .d.ts du SDK
☐ Les plugins sont isolés (un plugin ne modifie pas un autre)
☐ TuiPluginModule exclut server, PluginModule exclut tui
☐ Les hooks expérimentaux sont documentés et versionnés
```
