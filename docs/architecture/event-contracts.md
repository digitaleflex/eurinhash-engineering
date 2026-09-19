# Event Contracts — Contrats d'événements

> **Source** : `ARCHITECTURE_FINDINGS.md`, `sdk/dist/gen/types.gen.d.ts`, `plugin/dist/tui.d.ts`, `plugin/dist/index.d.ts`
> **Principe** : Les événements sont le canal principal de communication entre le core et le TUI.

---

## 1. Vue d'ensemble

L'union `Event` du SDK définit tous les types d'événements. Le `TuiEventBus` permet aux TuiPlugins de s'abonner à un sous-ensemble typé.

### 1.1 Type `Event` (union)

```typescript
type Event =
  | EventModelsDevRefreshed
  | EventIntegrationUpdated
  | EventIntegrationConnectionUpdated
  | EventCatalogUpdated
  | EventSessionCreated
  | EventSessionUpdated
  | EventSessionDeleted
  | EventMessageUpdated
  | EventMessageRemoved
  | EventMessagePartUpdated
  | EventMessagePartRemoved
  | EventSessionNextAgentSwitched
  | EventSessionNextModelSwitched
  | EventSessionNextMoved
  | EventSessionNextPrompted
  | EventSessionNextPromptAdmitted
  | EventSessionNextContextUpdated
  | EventSessionNextSynthetic
  | EventSessionNextShellStarted
  | EventSessionNextShellEnded
  | EventSessionNextStepStarted
  | EventSessionNextStepEnded
  | EventSessionNextStepFailed
  | EventSessionNextTextStarted
  | EventSessionNextTextDelta
  | EventSessionNextTextEnded
  | EventSessionNextReasoningStarted
  | EventSessionNextReasoningDelta
  | EventSessionNextReasoningEnded
  | EventSessionNextToolInputStarted
  | EventSessionNextToolInputDelta
  | EventSessionNextToolInputEnded
  | EventSessionNextToolCalled
  | EventSessionNextToolProgress
  | EventSessionNextToolSuccess
  | EventSessionNextToolFailed
  | EventSessionNextRetried
  | EventSessionNextCompactionStarted
  | EventSessionNextCompactionDelta
  | EventSessionNextCompactionEnded
  | EventSessionNextRevertStaged
  | EventSessionNextRevertCleared
  | EventSessionNextRevertCommitted
  | EventMessagePartDelta
  | EventSessionDiff
  | EventSessionError
  | EventInstallationUpdated
  | EventInstallationUpdateAvailable
  | EventFileEdited
  | EventReferenceUpdated
  | EventPermissionV2Asked
  | EventPermissionV2Replied
  | EventPluginAdded
  | EventProjectDirectoriesUpdated
  | EventFileWatcherUpdated
  | EventPtyCreated
  | EventPtyUpdated
  | EventPtyExited
  | EventPtyDeleted
  | EventQuestionV2Asked
  | EventQuestionV2Replied
  | EventQuestionV2Rejected
  | EventTodoUpdated
  | EventLspUpdated
  | EventPermissionAsked
  | EventPermissionReplied
  | EventTuiPromptAppend2
  | EventTuiCommandExecute2
  | EventTuiToastShow2
  | EventTuiSessionSelect2
  | EventMcpToolsChanged
  | EventMcpBrowserOpenFailed
  | EventCommandExecuted
  | EventProjectUpdated
  | EventServerInstanceDisposed
  | EventInstallationUpdated;
```

### 1.2 TuiEventBus — Souscription

```typescript
interface TuiEventBus {
  on<Type extends Event["type"]>(
    type: Type,
    handler: (event: Extract<Event, { type: Type }>) => void
  ): () => void; // unsubscribe
}
```

---

## 2. Contrats d'événements par domaine

### 2.1 Session Events

| Événement | Propriétés | Description | Hook associé |
|-----------|------------|-------------|--------------|
| `EventSessionCreated` | `info: Session` | Nouvelle session créée | — |
| `EventSessionUpdated` | `info: Session` | Session mise à jour | — |
| `EventSessionDeleted` | `info: Session` | Session supprimée | — |
| `EventSessionStatus` | `sessionID: string, status: SessionStatus` | Changement statut | `hooks.event` |
| `EventSessionIdle` | `sessionID: string` | Session idle | — |
| `EventSessionCompacted` | `sessionID: string` | Compaction | `experimental.session.compacting` |
| `EventSessionDiff` | `sessionID: string` | Diff générée | — |
| `EventSessionError` | `sessionID: string, error` | Erreur session | — |

**`SessionStatus`** :
```typescript
{ type: "idle" } | { type: "retry"; attempt: number; message: string; next: number } | { type: "busy" }
```

**Contrat** :
- **Input** : `sessionID: string` (pour la plupart)
- **Output** : `Session` ou `SessionStatus`
- **Owner** : Server core
- **Lifecycle** : Émission immédiate après le changement d'état
- **Failure** : `EventSessionError` avec `error` message

### 2.2 Message & Part Events

| Événement | Propriétés | Description | Hook associé |
|-----------|------------|-------------|--------------|
| `EventMessageUpdated` | `messageID: string` | Message mis à jour | — |
| `EventMessageRemoved` | `sessionID: string, messageID: string` | Message supprimé | — |
| `EventMessagePartUpdated` | `part: Part, delta?: string` | Partie mise à jour | — |
| `EventMessagePartRemoved` | `sessionID: string, messageID: string` | Partie supprimée | — |
| `EventMessagePartDelta` | `partID, delta` | Delta de texte | — |
| `EventSessionNextStepStarted` | `sessionID, ...` | Début d'étape | — |
| `EventSessionNextStepEnded` | `sessionID, ...` | Fin d'étape | — |
| `EventSessionNextToolCalled` | `sessionID, ...` | Outil appelé | `hooks.tool.execute.before/after` |
| `EventSessionNextToolSuccess` | `sessionID, ...` | Outil réussi | — |
| `EventSessionNextToolFailed` | `sessionID, ...` | Ooutil échoué | — |
| `EventSessionNextTextStarted` | `sessionID, ...` | Début de texte | — |
| `EventSessionNextTextDelta` | `sessionID, ...` | Delta de texte | — |
| `EventSessionNextTextEnded` | `sessionID, ...` | Fin de texte | — |
| `EventSessionNextReasoningStarted` | `sessionID, ...` | Début reasoning | — |
| `EventSessionNextReasoningDelta` | `sessionID, ...` | Delta reasoning | — |
| `EventSessionNextReasoningEnded` | `sessionID, ...` | Fin reasoning | — |
| `EventSessionNextToolInputStarted` | `sessionID, ...` | Début input outil | — |
| `EventSessionNextToolInputDelta` | `sessionID, ...` | Delta input | — |
| `EventSessionNextToolInputEnded` | `sessionID, ...` | Fin input | — |

**Contrat** :
- **Input** : `messageID` ou `sessionID` + `part`
- **Output** : `Part` mis à jour
- **Owner** : Server core (chat engine)
- **Lifecycle** : Émission pendant le stream de réponse
- **Failure** : `EventSessionError`

### 2.3 Permission Events

| Événement | Propriétés | Description | Hook associé |
|-----------|------------|-------------|--------------|
| `EventPermissionUpdated` | `permission` | Permission mise à jour | — |
| `EventPermissionReplied` | `permission` | Permission réponse | `hooks.permission.ask` |
| `EventPermissionV2Asked` | `...` | Permission v2 demandée | — |
| `EventPermissionV2Replied` | `...` | Permission v2 répondue | — |
| `EventPermissionAsked` | `...` | Permission demandée | — |
| `EventQuestionV2Asked` | `...` | Question posée | — |
| `EventQuestionV2Replied` | `...` | Question répondue | — |
| `EventQuestionV2Rejected` | `...` | Question rejetée | — |

**Contrat** :
- **Input** : `Permission` object
- **Output** : `{ status: "ask" | "deny" | "allow" }`
- **Owner** : Server core + Plugin guard
- **Lifecycle** : Avant exécution d'une action sensible
- **Failure** : Timeout → deny automatique

### 2.4 LSP Events

| Événement | Propriétés | Description |
|-----------|------------|-------------|
| `EventLspUpdated` | `[key: string]: unknown` | LSP mis à jour |
| `EventLspClientDiagnostics` | `serverID: string, path: string` | Diagnostics LSP |

**Contrat** :
- **Input** : `serverID`, `path`
- **Output** : Diagnostic LSP
- **Owner** : LSP server
- **Lifecycle** : Push quand un diagnostic change
- **Failure** : `EventSessionError` si le serveur LSP crashe

### 2.5 MCP Events

| Événement | Propriétés | Description |
|-----------|------------|-------------|
| `EventMcpToolsChanged` | `...` | Outils MCP changés |
| `EventMcpBrowserOpenFailed` | `...` | Navigateur MCP échoué |

### 2.6 File & VCS Events

| Événement | Propriétés | Description |
|-----------|------------|-------------|
| `EventFileEdited` | `file: string` | Fichier édité |
| `EventFileWatcherUpdated` | `file: string` | Watcher mis à jour |
| `EventVcsBranchUpdated` | `branch: string` | Branche VCS |
| `EventProjectUpdated` | `...` | Projet mis à jour |
| `EventProjectDirectoriesUpdated` | `...` | Dossiers projet |

### 2.7 Installation & Plugin Events

| Événement | Propriétés | Description |
|-----------|------------|-------------|
| `EventInstallationUpdated` | `version: string` | Installation mise à jour |
| `EventInstallationUpdateAvailable` | `version: string` | Mise à jour disponible |
| `EventPluginAdded` | `...` | Plugin ajouté |

### 2.8 TUI-Specific Events

| Événement | Propriétés | Description |
|-----------|------------|-------------|
| `EventTuiPromptAppend2` | `...` | Prompt TUI |
| `EventTuiCommandExecute2` | `...` | Commande TUI |
| `EventTuiToastShow2` | `...` | Toast TUI |
| `EventTuiSessionSelect2` | `...` | Sélection session |
| `EventCommandExecuted` | `command: string` | Commande exécutée |

### 2.9 Server Events

| Événement | Propriétés | Description |
|-----------|------------|-------------|
| `EventServerInstanceDisposed` | `directory: string` | Instance server détruite |
| `EventModelsDevRefreshed` | `...` | Modèles rafraîchis |
| `EventIntegrationUpdated` | `...` | Intégration mise à jour |
| `EventIntegrationConnectionUpdated` | `...` | Connexion intégration |
| `EventCatalogUpdated` | `...` | Catalogue mis à jour |
| `EventReferenceUpdated` | `...` | Référence mise à jour |
| `EventFileWatcherUpdated` | `file: string` | Watcher fichier |
| `EventPtyCreated/Updated/Exited/Deleted` | `...` | Processus PTY |

---

## 3. Utilisation dans les plugins

### 3.1 Via `TuiPluginApi.event`

```typescript
const unsubscribe = api.event.on("session.status", (event) => {
  console.log(`Session ${event.sessionID} status: ${event.status.type}`);
});

// Cleanup
unsubscribe();
```

### 3.2 Via `Hooks.event` (server plugin)

```typescript
const plugin: Plugin = async (input, options) => {
  return {
    event: async ({ event }) => {
      if (event.type === "session.status") {
        // Réaction au changement de statut
      }
    }
  };
};
```

---

## 4. Contrats de propriétés par type d'événement

### 4.1 `EventSessionStatus`
```typescript
{ type: "session.status", sessionID: string, status: SessionStatus }
```
- **Owner** : Server core
- **Emission** : Immédiate après changement de statut
- **Consommateurs** : TuiPlugins, UI components
- **Failure** : `EventSessionError`

### 4.2 `EventMessagePartUpdated`
```typescript
{ type: "message.part.updated", part: Part, delta?: string }
```
- **Owner** : Chat engine
- **Emission** : Pendant le stream de réponse LLM
- **Consommateurs** : TuiPlugins, rendering engine
- **Failure** : `EventSessionError`

### 4.3 `EventLspClientDiagnostics`
```typescript
{ type: "lsp.client.diagnostics", serverID: string, path: string }
```
- **Owner** : LSP server
- **Emission** : Quand un diagnostic change
- **Consommateurs** : LSP status UI
- **Failure** : `EventSessionError`

---

## 5. Souscription et lifecycle

```typescript
interface TuiEventBus {
  /**
   * S'abonner à un type d'événement spécifique.
   * Le handler est typé via Extract<Event, { type: Type }>.
   * Retourne une fonction d'unsubscribe.
   */
  on<Type extends Event["type"]>(
    type: Type,
    handler: (event: Extract<Event, { type: Type }>) => void
  ): () => void;
}
```

**Règles** :
1. Le type doit être un membre de `Event["type"]` (union exhaustive)
2. Le handler est automatiquement typé grâce à `Extract`
3. L'unsubscribe est retourné et doit être appelé dans `onDispose`
4. Les handlers sont synchrones (pas de `async` dans le handler)

---

## 6. Références

- `sdk/dist/gen/types.gen.d.ts` — Définition complète de l'union `Event`
- `plugin/dist/tui.d.ts` — `TuiEventBus` interface
- `plugin/dist/index.d.ts` — `Hooks.event` type
- `ARCHITECTURE_FINDINGS.md` §3 — Système d'événements
