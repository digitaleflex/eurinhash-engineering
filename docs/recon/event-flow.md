# Event Flow — Flux d'événements

## Vue d'ensemble

OpenCode utilise un système d'événements typés pour la communication entre le serveur et le TUI. Tous les événements sont définis dans `@opencode-ai/sdk/dist/gen/types.gen.d.ts` via le type union `Event`. Le TUI s'y abonne via `TuiEventBus.on<Type>()`.

## Taxonomie des événements

### Type Event (union complète)

```typescript
type Event =
  | EventServerInstanceDisposed    // server.instance.disposed
  | EventInstallationUpdated       // installation.updated
  | EventInstallationUpdateAvailable // installation.update-available
  | EventLspClientDiagnostics      // lsp.client.diagnostics
  | EventLspUpdated                // lsp.updated
  | EventMessageUpdated            // message.updated
  | EventMessageRemoved            // message.removed
  | EventMessagePartUpdated        // message.part.updated
  | EventMessagePartRemoved        // message.part.removed
  | EventPermissionUpdated         // permission.updated
  | EventPermissionReplied         // permission.replied
  | EventSessionStatus             // session.status
  | EventSessionIdle               // session.idle
  | EventSessionCompacted          // session.compacted
  | EventFileEdited                // file.edited
  | EventTodoUpdated               // todo.updated
  | EventCommandExecuted           // command.executed
  | EventSessionCreated            // session.created
  | EventSessionUpdated            // session.updated
  | EventSessionDeleted            // session.deleted
  | EventSessionDiff               // session.diff
  | EventSessionError              // session.error
  | EventFileWatcherUpdated        // file.watcher.updated
  | EventVcsBranchUpdated          // vcs.branch.updated
  | EventTuiPromptAppend           // tui.prompt.append
  | EventTuiCommandExecute         // tui.command.execute
  | EventTuiToastShow              // tui.toast.show
  | EventPtyCreated                // pty.created
  | EventPtyUpdated                // pty.updated
  | EventPtyExited                 // pty.exited
  | EventPtyDeleted                // pty.deleted
  | EventServerConnected;          // server.connected
```

### GlobalEvent

```typescript
type GlobalEvent = {
    directory: string;
    payload: Event;
};
```

Chaque événement est envoyé dans un contexte de répertoire (`directory`), ce qui permet au serveur de gérer plusieurs projets.

## Flux d'événements par domaine

### Flux Messages

```
AssistantMessage produit par le LLM
    │
    ├── EventMessageUpdated → { info: Message }
    │   (message complet mis à jour)
    │
    ├── EventMessagePartUpdated → { part, delta? }
    │   (partie du message, delta pour streaming)
    │
    └── EventMessageRemoved → { sessionID, messageID }
        (message supprimé)
```

### Flux Sessions

```
Nouvelle session créée
    │
    ├── EventSessionCreated → { info: Session }
    │
    ├── EventSessionUpdated → { info: Session }
    │   (titre changé, résumé mis à jour)
    │
    ├── EventSessionDiff → { sessionID, diff: FileDiff[] }
    │
    ├── EventSessionError → { sessionID?, error? }
    │
    ├── EventSessionCompacted → { sessionID }
    │
    ├── EventSessionIdle → { sessionID }
    │
    └── EventSessionDeleted → { info: Session }
```

### Flux Permissions

```
Permission demandée par le serveur
    │
    ├── EventPermissionUpdated → { info: Permission }
    │   (nouvelle permission à trancher)
    │
    └── EventPermissionReplied → { sessionID, permissionID, response }
        (utilisateur a répondu)
```

### Flux TUI

```
TUI émet des commandes internes
    │
    ├── EventTuiPromptAppend → { text }
    │   (texte ajouté au prompt)
    │
    ├── EventTuiCommandExecute → { command }
    │   (commande TUI: session.list, session.new, etc.)
    │
    └── EventTuiToastShow → { title?, message, variant, duration? }
        (notification toast)
```

### Flux File System

```
Fichier modifié (par un tool ou watcher)
    │
    ├── EventFileEdited → { file }
    │
    └── EventFileWatcherUpdated → { file, event: "add" | "change" | "unlink" }
```

### Flux VCS

```
Branche changée
    │
    └── EventVcsBranchUpdated → { branch? }
```

### Flux LSP

```
Diagnostic LSP reçu
    │
    ├── EventLspClientDiagnostics → { serverID, path }
    │
    └── EventLspUpdated → { [key: string]: unknown }
```

### Flux Process (PTY)

```
Terminal (PTY) créé/géré
    │
    ├── EventPtyCreated → { info: Pty }
    ├── EventPtyUpdated → { info: Pty }
    ├── EventPtyExited → { id, exitCode }
    └── EventPtyDeleted → { id }
```

### Flux Installation

```
Mise à jour disponible
    │
    ├── EventInstallationUpdated → { version }
    └── EventInstallationUpdateAvailable → { version }
```

## Sous-système d'événements internes

### TuiEventBus (TUI)

```typescript
interface TuiEventBus {
    on<Type extends Event["type"]>(
        type: Type,
        handler: (event: Extract<Event, { type: Type }>) => void
    ): () => void;
}
```

- **Typed** : le type de l'événement est déduit de `Event["type"]`
- **Unsubscribe** : retourne une fonction de cleanup
- **Usage** : `event.on("message.part.updated", (e) => { ... })`

### Event hooks dans les plugins (Serveur)

```typescript
interface Hooks {
    event?: (input: { event: Event }) => Promise<void>;
    // Hook générique, tous les événements passent par ici
}
```

### Lifecycle events

```typescript
interface TuiLifecycle {
    readonly signal: AbortSignal;
    onDispose: (fn: TuiDispose) => () => void;
}
```

- `AbortSignal` : signal d'arrêt du TUI
- `onDispose` : callbacks appelés à la fermeture

## Flux d'événements plugin (Hooks)

Les plugins reçoivent des événements via les hooks :

```
Serveur → déclenche l'event
    │
    ▼
[event] hook (si défini par un plugin)
    │
    ▼
Hook spécifique selon le type :
    ├── tool.execute.before / tool.execute.after
    ├── chat.message / chat.params / chat.headers
    ├── permission.ask
    ├── command.execute.before
    ├── shell.env
    ├── experimental.session.compacting
    ├── experimental.compaction.autocontinue
    ├── experimental.text.complete
    ├── tool.definition
    ├── experimental.chat.messages.transform
    ├── experimental.chat.system.transform
    └── experimental.provider.small_model
```

## Ordre de propagation typique

```
LLM produit un AssistantMessage
    │
    ├── 1. message.part.updated (chaque partie: text, reasoning, tool)
    │
    ├── 2. tool.execute.before (hook guard.audit-logger)
    │
    ├── 3. tool.execute.after (hook guard.redact + audit.log)
    │
    ├── 4. message.part.updated (ToolPart avec state: completed)
    │
    ├── 5. message.updated (message complet)
    │
    ├── 6. chat.params (si ajustement des paramètres)
    │
    └── 7. session.status → idle (quand la réponse est complète)
```

## Fait vs Hypothèse

| Fait confirmé | Hypothèse |
|---------------|-----------|
| Le type `Event` est une union exhaustive de tous les événements | L'ordre exact de certains événements peut varier selon le code du serveur |
| `TuiEventBus.on()` est typed et retourne un unsubscribe | Le mécanisme d'abonnement interne (EventEmitter, SSE, etc.) n'est pas dans les types |
| Le hook `event` est générique et reçoit tous les événements | Le comportement du hook `event` (async, error handling) est dans le source serveur |
| `EventMessagePartUpdated` supporte un `delta` pour le streaming | La taille et la fréquence des deltas dépendent du streaming implementation |
| `GlobalEvent` contient le `directory` pour le multi-projet | Le routage des événements entre projets est dans le source serveur |

## Événements critiques pour la performance

1. **message.part.updated** : peut être émis très fréquemment (streaming), le TUI doit gérer les mises à jour incrémentales
2. **tool.execute.before/after** : doivent être rapides car ils bloquent l'exécution
3. **lsp.client.diagnostics** : peuvent être très fréquents pendant la frappe
4. **permission.ask** : bloquant, nécessite une réponse utilisateur
5. **session.status** : déclenche des changements d'UI (busy spinner, etc.)
