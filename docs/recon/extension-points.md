# Extension Points — Points d'extension du système

## Vue d'ensemble

OpenCode offre de nombreux points d'extension pour personnaliser le comportement du serveur et du TUI. Ces points sont définis par les contrats `Plugin` (serveur) et `TuiPlugin` (TUI).

## Server Plugin Extension Points

### 1. Hooks de cycle de vie

| Hook | Entrée | Sortie | Description |
|------|--------|--------|-------------|
| `dispose` | — | `Promise<void>` | Nettoyage à l'arrêt |
| `config` | `Config` | `Promise<void>` | Changement de configuration |
| `event` | `{ event: Event }` | `Promise<void>` | TOUS les événements |

### 2. Hooks de chat

| Hook | Entrée | Sortie | Description |
|------|--------|--------|-------------|
| `chat.message` | `{ sessionID, agent?, model?, messageID?, variant?, message, parts }` | `{ message, parts }` | Modification du message utilisateur |
| `chat.params` | `{ sessionID, agent, model, provider, message }` | `{ temperature, topP, topK, maxOutputTokens, options }` | Paramètres LLM |
| `chat.headers` | `{ sessionID, agent, model, provider, message }` | `{ headers }` | Headers HTTP |
| `experimental.chat.messages.transform` | `{}` | `{ messages }` | Transforme les messages |
| `experimental.chat.system.transform` | `{ sessionID?, model }` | `{ system }` | Transforme le système prompt |
| `experimental.provider.small_model` | `{ provider }` | `{ model? }` | Modèle petit |

### 3. Hooks de tool

| Hook | Entrée | Sortie | Description |
|------|--------|--------|-------------|
| `tool.execute.before` | `{ tool, sessionID, callID }` | `{ args }` | Avant exécution |
| `tool.execute.after` | `{ tool, sessionID, callID, args }` | `{ title, output, metadata }` | Après exécution |
| `tool.definition` | `{ toolID }` | `{ description, parameters }` | Modification de la définition |
| `shell.env` | `{ cwd, sessionID?, callID? }` | `{ env }` | Variables d'environnement |

### 4. Hooks de permission

| Hook | Entrée | Sortie | Description |
|------|--------|--------|-------------|
| `permission.ask` | `Permission` | `{ status }` | Modification de la permission |

### 5. Hooks de commande

| Hook | Entrée | Sortie | Description |
|------|--------|--------|-------------|
| `command.execute.before` | `{ command, sessionID, arguments }` | `{ parts }` | Avant exécution commande |

### 6. Hooks expérimentaux

| Hook | Entrée | Sortie | Description |
|------|--------|--------|-------------|
| `experimental.text.complete` | `{ sessionID, messageID, partID }` | `{ text }` | Complétion texte |
| `experimental.session.compacting` | `{ sessionID }` | `{ context, prompt? }` | Directive de compaction |
| `experimental.compaction.autocontinue` | `{ sessionID, agent, model, provider, message, overflow }` | `{ enabled }` | Reprise auto après compaction |

### 7. Outils définis par le plugin

```typescript
interface Hooks {
    tool?: {
        [key: string]: ToolDefinition;
    };
}
```

Chaque clé est un nom d'outil, la valeur est un `ToolDefinition`.

### 8. Auth et Provider hooks

```typescript
interface AuthHook {
    provider: string;
    loader?: (auth, provider) => Promise<Record<string, any>>;
    methods: ({ type: "oauth" | "api", ... } | { type: "api", ... })[];
}

interface ProviderHook {
    id: string;
    models?: (provider, ctx) => Promise<Record<string, ModelV2>>;
}
```

## TUI Plugin Extension Points

### 1. Routes (`TuiPluginApi.route`)

```typescript
route: {
    register: (routes: TuiRouteDefinition[]) => () => void;
    navigate: (name: string, params?: Record<string, unknown>) => void;
    readonly current: TuiRouteCurrent;
}
```

- Enregistre des pages TUI personnalisées
- Navigation programmatique
- Route current en lecture seule

### 2. Slots (`TuiPluginApi.slots`)

```typescript
slots: {
    register: (plugin: TuiSlotPlugin) => string;
    register: <Slots>(plugin: TuiSlotPlugin<Slots>) => string;
}
```

- Enregistre des slots UI dans les zones de layout
- Types : `TuiSlotPlugin<Slots>` avec `SlotProps` pour chaque slot

### 3. Dialogs (`TuiPluginApi.ui`)

| Composant | Description |
|-----------|-------------|
| `Dialog` | Dialog générique (size: medium/large/xlarge) |
| `DialogAlert` | Alert avec confirmation |
| `DialogConfirm` | Confirmation avec cancel |
| `DialogPrompt` | Saisie utilisateur |
| `DialogSelect` | Sélection dans une liste |
| `Slot` | Slot personnalisé dans le layout |
| `Prompt` | Zone de saisie (avec ref, hint, etc.) |

### 4. Keymap (`TuiPluginApi.keymap`)

```typescript
keymap: Keymap<Renderable, KeyEvent>;
keys: {
    formatSequence: (parts) => string;
    formatBindings: (bindings) => string | undefined;
};
```

- Enregistrement de keybindings
- Formatage des séquences de touches

### 5. Mode (`TuiPluginApi.mode`)

```typescript
mode: {
    current: () => string;
    push: (mode: string) => () => void;
};
```

- Gestion des modes d'interface
- Push/pop de modes

### 6. KV Store (`TuiPluginApi.kv`)

```typescript
kv: {
    get: <Value>(key, fallback?) => Value;
    set: (key, value) => void;
    readonly ready: boolean;
};
```

- Stockage persistant clé-valeur
- Prêt au démarrage

### 7. Theme (`TuiPluginApi.theme`)

```typescript
theme: {
    current: TuiThemeCurrent;
    selected: string;
    has: (name) => boolean;
    set: (name) => boolean;
    install: (jsonPath) => Promise<void>;
    mode: () => "dark" | "light";
    readonly ready: boolean;
};
```

### 8. Plugins (`TuiPluginApi.plugins`)

```typescript
plugins: {
    list: () => ReadonlyArray<TuiPluginStatus>;
    activate: (id) => Promise<boolean>;
    deactivate: (id) => Promise<boolean>;
    add: (spec) => Promise<boolean>;
    install: (spec, options?) => Promise<TuiPluginInstallResult>;
};
```

- Gestion dynamique des plugins TUI

### 9. Attention (`TuiPluginApi.attention`)

```typescript
attention: {
    notify: (input) => Promise<TuiAttentionNotifyResult>;
    soundboard: TuiAttentionSoundboard;
};
```

- Notifications avec sons
- Soundboard pour customiser les sons

### 10. Toast (`TuiPluginApi.ui.toast`)

```typescript
toast: (input: { variant?, title?, message, duration? }) => void;
```

### 11. Lifecycle (`TuiPluginApi.lifecycle`)

```typescript
lifecycle: {
    readonly signal: AbortSignal;
    onDispose: (fn) => () => void;
};
```

### 12. Event Bus (`TuiPluginApi.event`)

```typescript
event: {
    on: <Type extends Event["type"]>(type, handler) => () => void;
};
```

## V2 Effect Extension Points (namespace interne)

| Point d'extension | Module | Interface |
|-------------------|--------|-----------|
| Agent CRUD | `v2/effect/agent` | `AgentHooks`, `AgentDraft` |
| Catalog CRUD | `v2/effect/catalog` | `CatalogHooks`, `CatalogDraft` |
| Command CRUD | `v2/effect/command` | `CommandHooks`, `CommandDraft` |
| Skill CRUD | `v2/effect/skill` | `SkillHooks`, `SkillDraft` |
| Reference | `v2/effect/reference` | `ReferenceHooks` |
| Filesystem | `v2/effect/filesystem` | `FileSystem` |
| Path | `v2/effect/path` | `Path` |
| Integration | `v2/effect/integration` | `IntegrationHooks` |
| AISDK | `v2/effect/aisdk` | `AISDKHooks` |
| Event stream | `v2/effect/event` | `Event`, `EventMap` |
| Plugin management | `v2/effect/plugin` | `Plugin`, `PluginDomain` |

## Fait vs Hypothèse

| Fait confirmé | Hypothèse |
|---------------|-----------|
| Les hooks server et TUI sont bien typés dans les .d.ts | Le comportement exact de chaque hook (ordre, error handling) est dans le source |
| Les V2 effect modules offrent un système CRUD complet | Le système v2/effect est en cours de développement |
| Les TUI slots permettent l'extension du layout | Les slots exacts disponibles dépendent du TUI version |
| `tool` peut définir des outils custom | L'API de ToolDefinition exacte est dans tool.d.ts |
| Le système de plugins est dynamique (activate/deactivate/add/install) | Le comportement exact du hot-reload n'est pas dans les types |

## Extension points non documentés

1. **`experimental.provider.small_model`** — le hook pour les petits modèles n'est pas documenté dans le prompt 00-deep-recon.md
2. **`experimental.text.complete`** — la complétion de texte est un hook récent
3. **V2 effect modules** — le système d'effets interne est en cours de stabilisation
4. **WorkspaceAdapter** — l'adaptateur de workspace permet des cibles remote/local personnalisées
