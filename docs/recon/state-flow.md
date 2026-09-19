# State Flow — Flux d'état

## Vue d'ensemble

L'état dans OpenCode est géré par le serveur backend et synchronisé vers le TUI via des événements. Le TUI expose un `TuiState` read-only qui permet d'accéder aux sessions, messages, parts, permissions, questions, LSP et MCP.

## Architecture d'état

### État côté serveur (backend)

Le serveur maintient l'état complet de chaque session :

```
Server State
├── sessions: Map<sessionID, Session>
│   ├── id, projectID, directory, parentID
│   ├── summary (additions, deletions, files, diffs)
│   ├── share (url)
│   ├── title, version
│   ├── time: { created, updated, compacting? }
│   └── revert?: { messageID, partID, snapshot?, diff? }
├── messages: Map<sessionID, Map<messageID, Message>>
│   ├── UserMessage: { id, sessionID, role, time, agent, model, system?, tools? }
│   └── AssistantMessage: { id, sessionID, role, time, error?, parentID, modelID, providerID, mode, path, summary?, cost, tokens, finish? }
├── parts: Map<messageID, Part[]>
│   ├── TextPart: { id, text, synthetic?, ignored?, time? }
│   ├── ReasoningPart: { id, text, time }
│   ├── FilePart: { id, mime, filename?, url, source? }
│   ├── ToolPart: { id, callID, tool, state, metadata? }
│   ├── StepStartPart: { id, snapshot? }
│   ├── StepFinishPart: { id, reason, snapshot?, cost, tokens }
│   ├── SnapshotPart: { id, snapshot }
│   ├── PatchPart: { id, hash, files }
│   ├── AgentPart: { id, name, source? }
│   ├── RetryPart: { id, attempt, error, time }
│   ├── CompactionPart: { id, auto }
│   └── SubtaskPart: { id, prompt, description, agent }
├── permissions: Map<sessionID, PermissionRequest[]>
├── questions: Map<sessionID, QuestionRequest[]>
├── todos: Map<sessionID, Todo[]>
├── providers: Map<providerID, Provider>
├── models: Map<modelID, Model>
└── lsp: LspStatus[]
    └── mcp: McpStatus[]
```

### État côté TUI (TuiState)

Le TUI accède à l'état via `TuiState` (read-only, live) :

```typescript
interface TuiState {
    readonly ready: boolean;
    readonly config: SdkConfig;
    readonly provider: ReadonlyArray<Provider>;
    readonly path: { state: string; config: string; worktree: string; directory: string };
    readonly vcs?: { branch?: string; default_branch?: string };
    session: {
        count: () => number;
        get: (sessionID: string) => Session | undefined;
        diff: (sessionID: string) => ReadonlyArray<TuiSidebarFileItem>;
        todo: (sessionID: string) => ReadonlyArray<TuiSidebarTodoItem>;
        messages: (sessionID: string) => ReadonlyArray<Message>;
        status: (sessionID: string) => SessionStatus | undefined;
        permission: (sessionID: string) => ReadonlyArray<PermissionRequest>;
        question: (sessionID: string) => ReadonlyArray<QuestionRequest>;
    };
    part: (messageID: string) => ReadonlyArray<Part>;
    lsp: () => ReadonlyArray<TuiSidebarLspItem>;
    mcp: () => ReadonlyArray<TuiSidebarMcpItem>;
}
```

## Cycle de vie de l'état session

```
[session.created] → Session ajoutée à l'état
    │
    ├── [message.updated] → Message ajouté/modifié
    │   └── [message.part.updated] → Part ajoutée/modifiée (delta streaming)
    │
    ├── [permission.updated] → Permission ajoutée
    │   └── [permission.replied] → Permission résolue
    │
    ├── [todo.updated] → Todos modifiés
    │
    ├── [session.status] → Status changé (idle/busy/retry)
    │
    ├── [session.compacted] → Session compactée
    │
    ├── [session.diff] → Diff calculée
    │
    └── [session.updated] → Session modifiée (titre, etc.)
        │
        └── [session.deleted] → Session retirée de l'état
```

## État des permissions

```typescript
interface Permission {
    id: string;
    type: string;
    pattern?: string | Array<string>;
    sessionID: string;
    messageID: string;
    callID?: string;
    title: string;
    metadata: Record<string, unknown>;
    time: { created: number };
}
```

- `permission.ask` hook → peut modifier en "ask" | "deny" | "allow"
- `EventPermissionUpdated` → notification TUI
- `EventPermissionReplied` → réponse utilisateur

## État des todos

```typescript
interface Todo {
    content: string;
    status: string; // pending, in_progress, completed, cancelled
    priority: string; // high, medium, low
    id: string;
}
```

- `EventTodoUpdated` → notification TUI avec tous les todos de la session

## État VCS

- `EventVcsBranchUpdated` → branche changée
- `EventFileEdited` → fichier édité
- `EventFileWatcherUpdated` → watcher: add/change/unlink
- `TuiState.path` → chemins (state, config, worktree, directory)
- `TuiState.vcs` → branche et default_branch

## État LSP et MCP (sidebar)

### LSP (`TuiSidebarLspItem`)
```typescript
type TuiSidebarLspItem = Pick<LspStatus, "id" | "root" | "status">;
// LspStatus: { id, name, root, status: "connected" | "error" }
```

### MCP (`TuiSidebarMcpItem`)
```typescript
type TuiSidebarMcpItem = {
    name: string;
    status: McpStatus["status"];
    error?: string;
};
// McpStatus: connected | disabled | failed | needs_auth | needs_client_registration
```

## Persistence de l'état

- **État serveur** : en mémoire (Node.js), pas de persistence sauf via la DB Prisma
- **Configuration** : `~/.config/opencode/opencode.jsonc`
- **State persistants** : `TuiKV` (key-value store persistant dans le TUI)
- **Route-usage** : `~/.config/opencode/route-usage.json` (feedback Bayes)
- **Free-models** : `~/.config/opencode/free-models.json` (statut live + latence)
- **Audit logs** : `~/.config/opencode/logs/audit-YYYY-MM-DD.jsonl`

## Configuration runtime (Config)

Le `Config` est le point central de configuration :

```typescript
interface Config {
    $schema?: string;
    theme?: string;
    keybinds?: KeybindsConfig;
    logLevel?: "DEBUG" | "INFO" | "WARN" | "ERROR";
    tui?: { scroll_speed?, scroll_acceleration?, diff_style? };
    command?: { [key: string]: { template, description?, agent?, model?, subtask? } };
    watcher?: { ignore?: string[] };
    plugin?: Array<string>;
    snapshot?: boolean;
    share?: "manual" | "auto" | "disabled";
    autoupdate?: boolean | "notify";
    disabled_providers?: string[];
    enabled_providers?: string[];
    model?: string;
    small_model?: string;
    username?: string;
    agent?: { [key: string]: AgentConfig };
    provider?: { [key: string]: ProviderConfig };
    mcp?: { [key: string]: McpLocalConfig | McpRemoteConfig };
    formatter?: false | { [key: string]: { disabled?, command?, extensions? } };
    lsp?: false | { [key: string]: { disabled?, command?, extensions?, env?, initialization? } };
    instructions?: string[];
    layout?: LayoutConfig;
    permission?: { edit?, bash?, webfetch?, doom_loop?, external_directory? };
    tools?: { [key: string]: boolean };
    enterprise?: { url?: string };
    experimental?: { hook?, chatMaxRetries?, disable_paste_summary?, batch_tool?, openTelemetry?, primary_tools? };
}
```

## Fait vs Hypothèse

| Fait confirmé | Hypothèse |
|---------------|-----------|
| TuiState est read-only et live via les événements | Le mécanisme de synchronisation exact (pull vs push) est dans le source |
| L'état des sessions est en mémoire côté serveur | La persistence exacte des sessions après un redémarrage n'est pas documentée |
| Les permissions sont gérées par des hooks | La UI exacte d'affichage des permissions n'est pas dans les types |
| Les todos sont par session | Le comportement multi-sessions pour les todos n'est pas clair |
| La config est chargée au démarrage et peut être mise à jour via le hook `config` | Le mécanisme de reload sans restart n'est pas documenté |

## Points de synchronisation critiques

1. **Message streaming** : les `delta` dans `EventMessagePartUpdated` permettent le rendu incrémental
2. **Session status** : `SessionStatus` (idle/busy/retry) est la source de vérité pour l'UI
3. **Cost tracking** : `AssistantMessage.cost` et `AssistantMessage.tokens` sont accumulés par part
4. **Compaction** : `CompactionPart` + `experimental.compaction.autocontinue` contrôlent le cycle de vie du contexte
5. **Permission flow** : `permission.ask` → hook → `allow/deny/ask` → `permission.replied`
