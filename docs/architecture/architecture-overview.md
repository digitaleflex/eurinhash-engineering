# Architecture Overview — EurinHash OpenCode

> **Source**: `ARCHITECTURE_FINDINGS.md` (Phase 0 Discovery), types `.d.ts` de `@opencode-ai/plugin` et `@opencode-ai/sdk`
> **Date**: 2026-09-19
> **Version OpenCode**: 1.18.31

---

## 1. Diagramme du système

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        OPENCODE CORE (v1.18.31)                         │
│                                                                         │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐   │
│  │   Server Side    │    │   SDK Layer      │    │   TUI Framework  │   │
│  │                  │    │                  │    │                  │   │
│  │ • Plugin (server)│◄──►│ OpencodeClient   │◄──►│ @opentui/solid  │   │
│  │ • Hooks          │    │ • createOpencode │    │ @opentui/core   │   │
│  │ • tool() def     │    │ • createServer   │    │ @opentui/keymap │   │
│  │ • BunShell       │    │ • Event types    │    │ CliRenderer     │   │
│  │ • Session mgmt   │    │ • Config types   │    │ SolidPlugin     │   │
│  └──────────────────┘    └──────────────────┘    └──────────────────┘   │
│           │                       │                       │              │
│           │         ┌─────────────┼─────────────┐         │              │
│           │         │             │             │         │              │
│           ▼         ▼             ▼             ▼         ▼              │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                    PLUGIN LAYER                                  │   │
│  │                                                                  │   │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │   │
│  │  │ Plugin (server) │  │ TuiPlugin       │  │ TuiPlugin       │  │   │
│  │  │ (guard.ts)      │  │ (eurinhash)     │  │ (better-compact)│  │   │
│  │  │                 │  │                 │  │                 │  │   │
│  │  │ Hooks:          │  │ api.state       │  │ api.state       │  │   │
│  │  │ • tool.execute. │  │ api.event       │  │ api.slots       │  │   │
│  │  │   before/after  │  │ api.ui          │  │ api.ui          │  │   │
│  │  │ • permission.   │  │ api.keymap      │  │ api.theme       │  │   │
│  │  │   ask           │  │ api.tuiConfig   │  │ api.router      │  │   │
│  │  │ • chat.message  │  │ api.kv          │  │                 │  │   │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘  │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│           │                       │                       │              │
│           ▼                       ▼                       ▼              │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                    TUI RENDERING LAYER                          │   │
│  │                                                                  │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │   │
│  │  │ app slot │  │ sidebar_ │  │ session_ │  │ home_    │       │   │
│  │  │          │  │ content  │  │ prompt   │  │ prompt   │       │   │
│  │  │ + app_   │  │ sidebar_ │  │ + prompt │  │ _footer  │       │   │
│  │  │ bottom   │  │ footer   │  │ _right   │  │          │       │   │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │   │
│  │                                                                  │   │
│  │  TuiHostSlotMap → TuiSlotPlugin → SolidJS Components             │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                    EURINHASH AGENT LAYER                          │   │
│  │                                                                  │   │
│  │  • agent/eurinhash.md (pipeline de routage)                       │   │
│  │  • scripts/route.py (EV scoring)                                 │   │
│  │  • free-models.json (statut live workers)                        │   │
│  │  • scripts/free-probe.py (rafraîchissement statut)               │   │
│  └──────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Frontières de processus

### 2.1 Server Process
**Responsabilité** : Exécution du noyau OpenCode, gestion des sessions, outils, permissions.
- **Entrées** : Requêtes HTTP (SDK), événements internes, hooks plugin
- **Sorties** : Réponses API, événements broadcastés, résultats d'outils
- **Cycle de vie** : Démarre avec `createOpencodeServer()`, s'arrête avec `close()`
- **Échec** : Erreurs remontées via `EventSessionError`, hooks `tool.execute.after` avec metadata d'erreur

### 2.2 TUI Process
**Responsabilité** : Rendu terminal, interaction utilisateur, gestion des slots.
- **Entrées** : `TuiPluginApi` (state, event, ui, keymap), événements serveur
- **Sorties** : Composants SolidJS rendus dans les slots, commandes keymap
- **Cycle de vie** : Démarre avec `createOpencodeTui()`, gère `lifecycle.signal` et `lifecycle.onDispose`
- **Échec** : `CliRenderer` détruit → `TuiAttentionNotifySkipReason.renderer_destroyed`

### 2.3 Plugin Process
**Responsabilité** : Extension du comportement core sans modifier le source OpenCode.
- **Plugin (server)** : Exécuté côté serveur, reçoivent `PluginInput`
- **TuiPlugin** : Exécuté côté TUI, reçoivent `TuiPluginApi`
- **Cycle de vie** : Chargés au démarrage, `TuiPluginState` = "first" | "updated" | "same"
- **Échec** : Plugin désactivable via `api.plugins.deactivate(id)`

### 2.4 Agent EurinHash Process
**Responsabilité** : Orchestration des workers, scoring EV, routing des requêtes.
- **Entrées** : `route.py` (EV scoring), `free-models.json` (statut workers)
- **Sorties** : Routage des requêtes vers les workers appropriés
- **Cycle de vie** : Indépendant, polling via `scripts/free-probe.py`
- **Échec** : Fallback automatique entre workers (codestral → groq → novita → zhipu → google → ollama)

---

## 3. Dépendances de package

```
@opencode-ai/sdk/v2          ← SDK client/server/types
  ├── client.d.ts            ← OpencodeClient, createOpencodeClient
  ├── server.d.ts            ← createOpencodeServer, createOpencodeTui
  ├── data.d.ts              ← message.user() helper
  └── gen/types.gen.d.ts     ← Tous les types Event, Model, Session, etc.

@opencode-ai/plugin          ← Plugin API
  ├── index.d.ts             ← Plugin, Hooks, PluginInput, Config
  ├── tui.d.ts               ← TuiPluginApi, TuiState, TuiEventBus, TuiSlots
  ├── tool.d.ts              ← tool() definition, ToolContext
  ├── shell.d.ts             ← BunShell, BunShellPromise
  └── v2/data.d.ts           ← v2 message helper

@opentui/core                ← Framework TUI
  ├── CliRenderer            ← Rendu terminal
  ├── Renderable, SlotMode   ← Types de rendu
  └── KeyEvent, RGBA         ← Types d'événements clavier et couleurs

@opentui/keymap              ← Gestion des raccourcis
  ├── Keymap, Binding        ← Structure de keymap
  └── extras                 ← BindingLookup, formatage

@opentui/solid               ← Composants SolidJS pour TUI
  ├── SolidPlugin            ← Plugin SolidJS
  └── JSX.Element            ← Composants JSX
```

**Règle de dépendance** : Le TUI dépend du SDK (lecture state), le SDK ne dépend pas du TUI. Les plugins server dépendent du SDK. Les plugins TUI dépendent du plugin API et du SDK.

---

## 4. Responsabilités par couche

| Couche | Responsabilités | API principale |
|--------|----------------|----------------|
| **Server** | Session management, tool execution, permissions, chat messages | `PluginInput`, `Hooks` |
| **SDK** | Client/server communication, type definitions, event subscription | `OpencodeClient`, `Event` union |
| **Plugin** | Extension behavior, hooks injection, command registration | `Plugin`, `TuiPlugin`, `Hooks` |
| **TUI Framework** | Terminal rendering, slot system, keymap, theming | `@opentui/*`, `CliRenderer` |
| **TuiPlugin** | UI composition via slots, state reading, event subscription | `TuiPluginApi` |
| **EurinHash Agent** | Worker orchestration, EV scoring, model routing | `route.py`, `free-models.json` |

---

## 5. Interfaces stables entre couches

### 5.1 Server → SDK
- `createOpencodeClient(config)` → `OpencodeClient`
- `createOpencodeServer(options)` → `{ url, close() }`
- Types exportés : `Event`, `Session`, `Message`, `Part`, `Model`, `Provider`

### 5.2 SDK → TUI Plugin
- `TuiState` (lecture seule) : `session`, `part`, `lsp`, `mcp`, `config`, `provider`, `path`, `vcs`
- `TuiEventBus` : `on(type, handler)` → unsubscribe
- `TuiPluginApi` : Toutes les propriétés sont `readonly` sauf `keymap` et `plugins`

### 5.3 TUI Plugin → Slots
- `api.slots.register(plugin)` → `string` (slot name)
- `TuiSlotPlugin<Slots>` → composant SolidJS
- `TuiHostSlotMap` → slots pré-définis par le host

### 5.4 Plugin Server → Core
- `Hooks` interface : `event`, `config`, `tool`, `auth`, `provider`, `chat.message`, `chat.params`, `chat.headers`, `permission.ask`, `tool.execute.before/after`, `experimental.*`

---

## 6. Points d'extension

| Point d'extension | Type | Interface | Usage |
|-------------------|------|-----------|-------|
| `hooks.event` | Server Plugin | `(input: { event: Event }) => Promise<void>` | Réaction aux événements système |
| `hooks.tool.execute.before/after` | Server Plugin | Tool definition hook | Sécurité, audit, modification args |
| `hooks.permission.ask` | Server Plugin | `(input: Permission, output) => Promise<void>` | Contrôle d'accès |
| `slots.register` | TuiPlugin | `api.slots.register(TuiSlotPlugin)` | Extension UI |
| `keymap.registerLayer` | TuiPlugin | `api.keymap.registerLayer(bindings)` | Raccourcis personnalisés |
| `plugins.add` | TuiPlugin | `api.plugins.add(spec)` | Chargement dynamique |
| `experimental.provider.small_model` | Server Plugin | Hook provider | Routing modèle |
| `experimental.session.compacting` | Server Plugin | Hook compaction | Personnalisation contexte |

---

## 7. Principes architecturaux appliqués

1. **Direction de dépendance intentionnelle** : Le TUI lit `TuiState` mais n'écrit pas dans le core. Le SDK est la seule source de truth.
2. **Le TUI n'atteint pas le backend interne** : Toute donnée backend passe par `OpencodeClient` ou `TuiState` (généré par le SDK).
3. **Données manquantes exposées via le serveur** : Latence, queue, risque/sensibilité → à définir comme extensions du SDK.
4. **State de présentation ne duplique pas le domaine** : `AgentObservabilityState` est un view-model, pas une copie de `TuiState`.
5. **Compatibilité upstream préservée** : Tout plugin utilise l'API publique `@opencode-ai/plugin`, jamais de modification du binaire `.exe`.
6. **Nouvelles abstractions justifiées** : Seules les extensions qui retirent un couplage réel ou activent une UX requise sont créées.

---

## 8. Références

- `ARCHITECTURE_FINDINGS.md` — Phase 0 Discovery complète
- `~/.config/opencode/node_modules/@opencode-ai/plugin/dist/tui.d.ts` — TUI API types
- `~/.config/opencode/node_modules/@opencode-ai/plugin/dist/index.d.ts` — Plugin API + Hooks
- `~/.config/opencode/node_modules/@opencode-ai/sdk/dist/v2/client.d.ts` — SDK client types
- `~/.config/opencode/node_modules/@opencode-ai/sdk/dist/v2/server.d.ts` — SDK server types
- `~/.config/opencode/node_modules/@opencode-ai/sdk/dist/gen/types.gen.d.ts` — Tous les types Event/Model/Session
- Prompt `01-architecture-map.md` — Framework d'analyse architectural
