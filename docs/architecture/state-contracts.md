# State Contracts — Contrats d'état

> **Source** : `ARCHITECTURE_FINDINGS.md`, `plugin/dist/tui.d.ts`, `sdk/dist/v2/gen/types.gen.d.ts`
> **Principe** : `TuiState` est en lecture seule. L'état de présentation ne duplique pas l'état de domaine.

---

## 1. Vue d'ensemble de `TuiState`

`TuiState` est l'unique source de vérité pour le TUI. Toutes ses propriétés sont `readonly`.

```typescript
interface TuiState {
  readonly ready: boolean;
  readonly config: SdkConfig;
  readonly provider: ReadonlyArray<Provider>;
  readonly path: {
    state: string;
    config: string;
    worktree: string;
    directory: string;
  };
  readonly vcs: {
    branch?: string;
    default_branch?: string;
  } | undefined;
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

---

## 2. Contrats par domaine

### 2.1 Session State

| Propriété | Type | Description | Lecture seule |
|-----------|------|-------------|---------------|
| `session.count()` | `() => number` | Nombre de sessions | ✅ |
| `session.get(id)` | `(id: string) => Session \| undefined` | Session par ID | ✅ |
| `session.status(id)` | `(id: string) => SessionStatus \| undefined` | Statut session | ✅ |
| `session.diff(id)` | `(id: string) => TuiSidebarFileItem[]` | Fichiers modifiés | ✅ |
| `session.todo(id)` | `(id: string) => TuiSidebarTodoItem[]` | Todos | ✅ |
| `session.messages(id)` | `(id: string) => Message[]` | Messages | ✅ |
| `session.permission(id)` | `(id: string) => PermissionRequest[]` | Permissions en attente | ✅ |
| `session.question(id)` | `(id: string) => QuestionRequest[]` | Questions en attente | ✅ |

**`Session`** : `{ id, projectID, directory, parentID?, summary?, share?, title, version, time: {created, updated, compacting?}, ... }`

**`SessionStatus`** : `{ type: "idle" } | { type: "retry"; attempt, message, next } | { type: "busy" }`

**Contrat** :
- **Input** : `sessionID: string`
- **Output** : `Session`, `SessionStatus`, ou `undefined`
- **Owner** : Server core
- **Lifecycle** : Lecture directe, pas de mutation
- **Failure** : Retourne `undefined` si session non trouvée

### 2.2 Part State

| Méthode | Type | Description |
|---------|------|-------------|
| `part(messageID)` | `(id: string) => Part[]` | Parts d'un message |

**`Part`** union : `TextPart | ReasoningPart | FilePart | ToolPart | StepStartPart | StepFinishPart | SnapshotPart | PatchPart | AgentPart | RetryPart | CompactionPart`

**`StepFinishPart`** : `{ id, sessionID, messageID, type: "step-finish", reason, snapshot?, cost, tokens: {input, output, reasoning, cache: {read, write}} }`

**`ToolPart`** : `{ id, sessionID, messageID, type: "tool", callID, tool, state, metadata? }`

**Contrat** :
- **Input** : `messageID: string`
- **Output** : `ReadonlyArray<Part>`
- **Owner** : Chat engine
- **Lifecycle** : Lecture directe, mis à jour via `EventMessagePartUpdated`
- **Failure** : Retourne tableau vide si aucune partie

### 2.3 Provider State

| Propriété | Type | Description |
|-----------|------|-------------|
| `provider` | `ReadonlyArray<Provider>` | Providers disponibles |

**`Provider`** : Info provider depuis `TuiState.provider`

**`Model`** : `{ id, providerID, api: {id, url, npm}, name, capabilities }`

**Contrat** :
- **Input** : Aucun
- **Output** : `ReadonlyArray<Provider>`
- **Owner** : Server core
- **Lifecycle** : Lecture directe
- **Failure** : Tableau vide si aucun provider

### 2.4 MCP State

| Méthode | Type | Description |
|---------|------|-------------|
| `mcp()` | `() => ReadonlyArray<TuiSidebarMcpItem>` | Liste MCP |

**`TuiSidebarMcpItem`** : `{ name, status: McpStatus["status"], error? }`

**`McpStatus`** : `McpStatusConnected | McpStatusDisabled | McpStatusFailed | McpStatusNeedsAuth | McpStatusNeedsClientRegistration`

**Contrat** :
- **Input** : Aucun
- **Output** : `ReadonlyArray<TuiSidebarMcpItem>`
- **Owner** : MCP server
- **Lifecycle** : Lecture directe, mis à jour via événements
- **Failure** : Tableau vide

### 2.5 LSP State

| Méthode | Type | Description |
|---------|------|-------------|
| `lsp()` | `() => ReadonlyArray<TuiSidebarLspItem>` | Liste LSP |

**`TuiSidebarLspItem`** : `Pick<LspStatus, "id" | "root" | "status">`

**`LspStatus`** : `{ id, name, root, status: "connected" | "error" }`

**Contrat** :
- **Input** : Aucun
- **Output** : `ReadonlyArray<TuiSidebarLspItem>`
- **Owner** : LSP server
- **Lifecycle** : Lecture directe, mis à jour via `EventLspUpdated`
- **Failure** : Tableau vide

### 2.6 Config & Path State

| Propriété | Type | Description |
|-----------|------|-------------|
| `config` | `SdkConfig` | Configuration SDK |
| `path.state` | `string` | Chemin state |
| `path.config` | `string` | Chemin config |
| `path.worktree` | `string` | Chemin worktree |
| `path.directory` | `string` | Répertoire projet |
| `vcs.branch` | `string \| undefined` | Branche courante |
| `vcs.default_branch` | `string \| undefined` | Branche par défaut |

**Contrat** :
- **Input** : Aucun
- **Output** : `SdkConfig`, chemins, VCS info
- **Owner** : Server core
- **Lifecycle** : Lecture directe
- **Failure** : Valeurs par défaut

### 2.7 Theme & UI State

| Propriété | Type | Description |
|-----------|------|-------------|
| `api.theme.current` | `TuiThemeCurrent` | Couleurs RGBA |
| `api.theme.mode()` | `"dark" \| "light"` | Mode |
| `api.tuiConfig` | `Frozen<TuiConfigView>` | Config TUI gelée |
| `api.mode.current()` | `string` | Mode actuel |

**`TuiThemeCurrent`** : `{ primary, secondary, accent, error, warning, success, info, text, textMuted, background, backgroundPanel, ... }` (RGBA values)

**Contrat** :
- **Input** : Aucun
- **Output** : Thème, config, mode
- **Owner** : TUI framework
- **Lifecycle** : Lecture directe
- **Failure** : Valeurs par défaut du thème

---

## 3. Règles d'état

### 3.1 `TuiState` est en lecture seule

```typescript
// ✅ CORRECT
const session = api.state.session.get("session-id");
const parts = api.state.part("message-id");

// ❌ INTERDIT
api.state.session.set(...); // Pas de mutation
api.state.ready = false;    // Pas de réécriture
```

### 3.2 L'état de présentation ne duplique pas l'état de domaine

- `AgentObservabilityState` (view-model) → dérivé de `TuiState`, pas une copie
- Les calculs (tokens/session, coût/part) sont des dérivations, pas des stocks
- Le `StepFinishPart.cost` est la source de vérité, pas un cache local

### 3.3 Données manquantes

| Donnée | Disponible dans `TuiState` ? | Contournement |
|--------|------------------------------|---------------|
| Latence | ❌ | Calcul via timestamps |
| Queue | ❌ | Compteur de messages |
| Throughput | ❌ | Calcul : tokens / elapsed |
| Risque | ❌ | À définir dans `guard.ts` / eurinhash |
| Sensibilité | ❌ | À définir dans `guard.ts` / eurinhash |

**Principe** : Les données manquantes doivent être exposées via le serveur API et le SDK généré.

---

## 4. Événements de changement d'état

Le `TuiState` ne réagit pas directement — les changements sont signalés via les événements :

```
TuiState (lecture) ←── State mis à jour par le core
                        │
                        ▼
api.event.on("session.status", handler) → Mise à jour view-model
api.event.on("message.part.updated", handler) → Mise à jour tokens/coût
api.event.on("lsp.updated", handler) → Mise à jour LSP status
api.event.on("mcp.*", handler) → Mise à jour MCP status
```

**Contrat** :
- Le TUI lit `TuiState` au moment du rendu
- Les événements déclenchent des recalculs du view-model
- Le view-model est un dérivé de `TuiState`, jamais une source indépendante

---

## 5. Contrats de type par sous-système

### 5.1 Session

```typescript
type Session = {
  id: string;
  projectID: string;
  directory: string;
  parentID?: string;
  summary?: { title?: string; body?: string; diffs: FileDiff[] };
  share?: string;
  title: string;
  version: string;
  time: { created: number; updated: number; compacting?: boolean };
};
```

### 5.2 Message

```typescript
type UserMessage = {
  id: string;
  sessionID: string;
  role: "user";
  time: { created: number };
  summary?: { title?, body?, diffs: FileDiff[] };
  agent: string;
  model: { providerID: string; modelID: string };
  system?: string;
  tools?: { [key: string]: boolean };
};
```

### 5.3 Part (union)

```typescript
type Part = TextPart | ReasoningPart | FilePart | ToolPart | StepStartPart
          | StepFinishPart | SnapshotPart | PatchPart | AgentPart
          | RetryPart | CompactionPart;
```

### 5.4 Model

```typescript
type Model = {
  id: string;
  providerID: string;
  api: { id: string; url: string; npm: string };
  name: string;
  capabilities: ModelCapabilities;
};
```

---

## 6. Validation des contrats d'état

```
CHECKLIST — State Contracts Validation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
☐ TuiState est utilisé en lecture seule uniquement
☐ Aucune mutation directe de TuiState
☐ Les view-models sont dérivés de TuiState, pas des copies
☐ Les données manquantes sont exposées via le SDK
☐ Les calculs (tokens, coût) utilisent StepFinishPart comme source de vérité
☐ Les événements déclenchent des recalculs du view-model
☐ Les types Part sont utilisés via l'union du SDK
☐ Les sessions sont récupérées via session.get(id), pas par indexation
☐ Les providers sont lus via provider, pas par accès direct
☐ Les chemins sont lus via path, pas par process.cwd()
```
