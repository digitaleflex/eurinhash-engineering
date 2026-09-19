# TUI/SDK Boundary — Frontière TUI/SDK

> **Source** : `ARCHITECTURE_FINDINGS.md`, `tui.d.ts`, `index.d.ts`, `sdk/v2/client.d.ts`, `sdk/v2/server.d.ts`
> **Principe** : Le TUI ne doit pas atteindre dans les internaux du backend quand une frontière SDK/API existe.

---

## 1. Définition de la frontière

La frontière TUI/SDK est définie par l'interface `TuiPluginApi`. C'est le **seul** canal par lequel un TuiPlugin accède aux données et services du core OpenCode.

### 1.1 Côté TUI (en dehors du core)
```
┌──────────────────────────────────────┐
│         TuiPlugin (eurinhash)        │
│                                      │
│  api.state  ──┐                     │
│  api.event    │                     │
│  api.ui       │                     │
│  api.keymap   │                     │
│  api.slots    │                     │
│  api.theme    │                     │
│  api.kv       │                     │
│  api.client   │                     │
│  api.lifecycle│                     │
│  api.plugins  │                     │
│  api.route    │                     │
│  api.mode     │                     │
│  api.keys     │                     │
│  api.tuiConfig│                     │
│  api.renderer │                     │
└────────────────┼────────────────────┘
                 │ TuiPluginApi
                 │
                 ▼
┌──────────────────────────────────────┐
│         Frontière SDK                │
│  (OpencodeClient + TuiState)         │
└────────────────┬─────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────┐
│         OpenCode Core (server)       │
│  (binaire .exe + plugins server)     │
└──────────────────────────────────────┘
```

### 1.2 Côté SDK (à l'intérieur du core)
- `OpencodeClient` : Client pour communiquer avec le serveur
- `createOpencodeServer()` : Crée le serveur
- `createOpencodeTui()` : Crée l'environnement TUI
- Types `Event`, `Session`, `Message`, `Part`, `Model`, `Provider` : Types partagés

---

## 2. Entrées/Sorties de la frontière

### 2.1 TUI → SDK (lecture)

| Entrée TUI | Type SDK | Source | Description |
|------------|----------|--------|-------------|
| `api.state.ready` | `boolean` | `TuiState.readonly.ready` | Prêt du SDK |
| `api.state.config` | `SdkConfig` | `TuiState.readonly.config` | Configuration |
| `api.state.provider` | `ReadonlyArray<Provider>` | `TuiState.readonly.provider` | Providers disponibles |
| `api.state.session.get(id)` | `Session \| undefined` | `TuiState.session.get()` | Session par ID |
| `api.state.session.status(id)` | `SessionStatus \| undefined` | `TuiState.session.status()` | Statut session |
| `api.state.session.messages(id)` | `ReadonlyArray<Message>` | `TuiState.session.messages()` | Messages |
| `api.state.part(messageID)` | `ReadonlyArray<Part>` | `TuiState.part()` | Parts d'un message |
| `api.state.lsp()` | `ReadonlyArray<TuiSidebarLspItem>` | `TuiState.lsp()` | LSP items |
| `api.state.mcp()` | `ReadonlyArray<TuiSidebarMcpItem>` | `TuiState.mcp()` | MCP items |
| `api.state.path` | `{ state, config, worktree, directory }` | `TuiState.readonly.path` | Chemins |
| `api.state.vcs` | `{ branch?, default_branch? }` | `TuiState.readonly.vcs` | VCS info |
| `api.client` | `OpencodeClient` | `TuiPluginApi.client` | Client SDK complet |
| `api.tuiConfig` | `Frozen<TuiConfigView>` | `TuiPluginApi.tuiConfig` | Config gelée |
| `api.theme.current` | `TuiThemeCurrent` | `TuiPluginApi.theme.current` | Thème courant |
| `api.mode.current()` | `string` | `TuiPluginApi.mode.current` | Mode actuel |

### 2.2 TUI → SDK (événements)

| Événement | Type | Direction | Description |
|-----------|------|-----------|-------------|
| `api.event.on("session.status", ...)` | `EventSessionStatus` | SDK → TUI | Changement statut |
| `api.event.on("message.part.updated", ...)` | `EventMessagePartUpdated` | SDK → TUI | Nouvelle partie |
| `api.event.on("lsp.updated", ...)` | `EventLspUpdated` | SDK → TUI | Mise jour LSP |
| `api.event.on("lsp.client.diagnostics", ...)` | `EventLspClientDiagnostics` | SDK → TUI | Diagnostics |
| `api.event.on("session.created", ...)` | `EventSessionCreated` | SDK → TUI | Création session |
| `api.event.on("session.updated", ...)` | `EventSessionUpdated` | SDK → TUI | Mise jour session |
| `api.event.on("session.deleted", ...)` | `EventSessionDeleted` | SDK → TUI | Suppression session |
| `api.event.on("session.error", ...)` | `EventSessionError` | SDK → TUI | Erreur session |
| `api.event.on("session.idle", ...)` | `EventSessionIdle` | SDK → TUI | Session idle |
| `api.event.on("session.compacted", ...)` | `EventSessionCompacted` | SDK → TUI | Compaction |
| `api.event.on("file.edited", ...)` | `EventFileEdited` | SDK → TUI | Fichier édité |
| `api.event.on("file.watcher.updated", ...)` | `EventFileWatcherUpdated` | SDK → TUI | Watcher |
| `api.event.on("command.executed", ...)` | `EventCommandExecuted` | SDK → TUI | Commande |
| `api.event.on("todo.updated", ...)` | `EventTodoUpdated` | SDK → TUI | Todo |
| `api.event.on("permission.updated", ...)` | `EventPermissionUpdated` | SDK → TUI | Permission |
| `api.event.on("permission.replied", ...)` | `EventPermissionReplied` | SDK → TUI | Réponse permission |

### 2.3 TUI → Core (via SDK)

| Action | Méthode | Description |
|--------|---------|-------------|
| Navigation | `api.route.navigate(name, params)` | Navigation TUI |
| Plugin install | `api.plugins.install(spec, options)` | Installation plugin |
| Plugin activate | `api.plugins.activate(id)` | Activation |
| Plugin deactivate | `api.plugins.deactivate(id)` | Désactivation |
| Plugin add | `api.plugins.add(spec)` | Ajout plugin |
| Toast | `api.ui.toast(input)` | Notification |
| Dialog | `api.ui.dialog` | Pile de dialogues |
| Attention | `api.attention.notify(input)` | Notification sonore |
| KV | `api.kv.get/set(key, value)` | Stockage clé-valeur |

### 2.4 Core → TUI (via SDK)

| Action | Méthode | Description |
|--------|---------|-------------|
| Session status | `api.event.on("session.status", handler)` | Push statut |
| Part update | `api.event.on("message.part.updated", handler)` | Push partie |
| Config change | `hooks.config` | Notification config |
| Tool execute | `hooks.tool.execute.before/after` | Hook outil |
| Permission | `hooks.permission.ask` | Demande permission |
| Chat message | `hooks.chat.message` | Message chat |
| Compaction | `hooks.experimental.session.compacting` | Hook compaction |
| Auto-continue | `hooks.experimental.compaction.autocontinue` | Hook auto-continue |

---

## 3. Règles de la frontière

### 3.1 Le TUI NE DOIT PAS

1. **Importer directement des types internes du server** — Tous les types doivent venir du SDK (`@opencode-ai/sdk/v2`)
2. **Appeler des méthodes internes du serveur** — Toute interaction passe par `OpencodeClient` ou `TuiPluginApi`
3. **Modifier l'état du core** — `TuiState` est `readonly`, les modifications passent par les hooks
4. **Accéder au binaire `.exe`** — Le source OpenCode est compilé, pas accessible
5. **Contourner le SDK pour lire des données** — Même si une donnée est disponible dans le TUI, elle doit provenir du SDK

### 3.2 Le TUI DOIT

1. **Lire `TuiState` via `api.state`** — Jamais directement
2. **S'abonner aux événements via `api.event.on()`** — Jamais de polling direct
3. **Utiliser `api.client` pour les opérations asynchrones** — `OpencodeClient` est le seul client
4. **Utiliser `api.slots.register()` pour l'UI** — Jamais de rendu direct hors des slots
5. **Utiliser `api.keymap.registerLayer()` pour les raccourcis** — Jamais de keymap interne
6. **Utiliser `api.lifecycle.signal` pour le cleanup** — `onDispose` pour la déregistration

---

## 4. Données manquantes et contournements

| Donnée | Disponible dans le SDK ? | Contournement |
|--------|--------------------------|---------------|
| Latence | ❌ Non | Calcul via `time.updated - time.created` |
| Queue | ❌ Non | Compteur de messages/parts |
| Throughput | ❌ Non | Calcul : tokens / elapsed |
| Risque | ❌ Non | À définir dans `guard.ts` ou eurinhash |
| Sensibilité | ❌ Non | À définir dans `guard.ts` ou eurinhash |
| Coût estimé vs réel | ✅ Via `StepFinishPart.cost` | Direct |
| Tokens cache read/write | ✅ Via `StepFinishPart.tokens.cache` | Direct |
| Context limit | ⚠️ Via `Model.capabilities` | Calcul |

**Principe** : Les données manquantes doivent être exposées via le serveur API et le SDK généré, jamais importées directement.

---

## 5. Lifecycle de la frontière

```
Initialisation:
  1. createOpencodeServer() → serveur démarré
  2. createOpencodeTui() → TUI initialisé
  3. TuiPluginApi disponible
  4. api.state.ready = true
  5. Plugins chargés (TuiPluginState = "first" | "updated" | "same")

Exécution:
  6. api.event.on(type, handler) → subscriptions actives
  7. api.state.session.get(id) → lecture en cours
  8. api.slots.register(plugin) → slots rendus
  9. api.keymap.registerLayer(bindings) → raccourcis actifs

Cleanup:
  10. api.lifecycle.signal.abort → signal déclenché
  11. api.lifecycle.onDispose(fn) → cleanup exécuté
  12. api.plugins.deactivate(id) → plugin désactivé
  13. serveur.close() → serveur arrêté
```

---

## 6. Échec de la frontière

| Échec | Comportement | Mitigation |
|-------|-------------|------------|
| `TuiState` non prêt | `api.state.ready = false` | Attendre avant lecture |
| Événement non trouvé | `api.event.on` ne déclenche pas | Vérifier le type Event |
| Plugin malveillant | `api.plugins.deactivate(id)` | Isolation via sandbox |
| Renderer détruit | `TuiAttentionNotifySkipReason.renderer_destroyed` | Rechargement automatique |
| Écart version plugin/API | `TuiPluginState` différent | Compatibility layer v1/v2 |

---

## 7. Validation de la frontière

```
CHECKLIST — TUI/SDK Boundary Validation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
☐ Le TuiPlugin n'importe aucun type interne du server
☐ Toutes les lectures de state passent par api.state
☐ Tous les événements passent par api.event.on()
☐ Aucun accès direct au binaire .exe
☐ api.client utilisé pour toute opération asynchrone
☐ api.slots.register() utilisé pour tout rendu UI
☐ api.lifecycle.signal utilisé pour le cleanup
☐ TuiState est readonly (pas d'écriture)
☐ Les données manquantes sont exposées via le SDK
☐ Les hooks server ne modifient pas le state TUI directement
```
