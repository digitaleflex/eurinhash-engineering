# Domain State — 12 Separated State Types

> Source: `@opencode-ai/sdk/dist/gen/types.gen.d.ts`, `@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`, `@opencode-ai/plugin/dist/index.d.ts`, `@opencode-ai/plugin/dist/tui.d.ts`
> Rule: `UNKNOWN != ZERO` — never fabricate a state value; if a type field is absent, it is `undefined`, not `""`, `0`, or `false`.

---

## 1. SESSION STATE

**Origin**: `Session` type (both v1 and v2)

### v1 `Session` (`types.gen.d.ts:465`)
```typescript
type Session = {
    id: string;
    projectID: string;
    directory: string;
    parentID?: string;
    summary?: { additions: number; deletions: number; files: number; diffs?: Array<FileDiff> };
    share?: { url: string };
    title: string;
    version: string;
    time: { created: number; updated: number; compacting?: number };
    revert?: { messageID: string; partID?: string; snapshot?: string; diff?: string };
};
```

### v2 `Session` (`v2/gen/types.gen.d.ts:64`)
```typescript
type Session = {
    id: string;
    slug: string;
    projectID: string;
    workspaceID?: string;
    directory: string;
    path?: string;
    parentID?: string;
    summary?: { additions: number; deletions: number; files: number; diffs?: Array<SnapshotFileDiff> };
    cost?: number;
    tokens?: { input: number; output: number; reasoning: number; cache: { read: number; write: number } };
    share?: { url: string };
    title: string;
    agent?: string;
    model?: { id: string; providerID: string; variant?: string };
    version: string;
    metadata?: { [key: string]: unknown };
    time: { created: number; updated: number; compacting?: number; archived?: number };
    permission?: PermissionRuleset;
    revert?: { messageID: string; partID?: string; snapshot?: string; diff?: string };
};
```

### State fields and semantics

| Field | Type | Semantics | UNKNOWN handling |
|-------|------|-----------|-----------------|
| `id` | `string` | Unique session identifier | Never absent for live sessions |
| `parentID` | `string \| undefined` | Parent session for child sessions | `undefined` means root session — NOT `""` |
| `status` | `SessionStatus` | Derived from `session.status` event | Must be one of the three discriminated variants; never fabricate |
| `time.created` | `number` | Unix ms timestamp | `undefined` if not set; never `0` as "not set" |
| `time.compacting` | `number \| undefined` | When compaction started | `undefined` when not compacting — NOT `0` |
| `time.archived` | `number \| undefined` | v2 only — when archived | `undefined` when active |
| `cost` | `number \| undefined` | v2 only — accumulated cost | `undefined` until first token; NOT `0` as placeholder |
| `summary` | `object \| undefined` | Diff summary | `undefined` if no summary generated |
| `revert` | `object \| undefined` | Revert state for rollback | `undefined` when no revert in progress |

### Derived states

- **`idle`**: `{ type: "idle" }` — no active request
- **`busy`**: `{ type: "busy" }` — request in progress
- **`retry`**: `{ type: "retry"; attempt: number; message: string; next: number; action?: {...} }` — retry scheduled

---

## 2. REQUEST STATE

**Origin**: `UserMessage`, `AssistantMessage`, `Part` types

### Request lifecycle state machine

A request transitions through these states:

```
created → parts_accumulating → step_started → (text/reasoning/tool) → step_finished → completed
                                              ↓
                                         failed/error
```

### `UserMessage` (v1 `types.gen.d.ts:39`)
```typescript
type UserMessage = {
    id: string;
    sessionID: string;
    role: "user";
    time: { created: number };
    summary?: { title?: string; body?: string; diffs: Array<FileDiff> };
    agent: string;
    model: { providerID: string; modelID: string };
    system?: string;
    tools?: { [key: string]: boolean };
};
```

### `AssistantMessage` (v1 `types.gen.d.ts:98`)
```typescript
type AssistantMessage = {
    id: string;
    sessionID: string;
    role: "assistant";
    time: { created: number; completed?: number };
    error?: ProviderAuthError | UnknownError | MessageOutputLengthError | MessageAbortedError | ApiError;
    parentID: string;
    modelID: string;
    providerID: string;
    mode: string;
    path: { cwd: string; root: string };
    summary?: boolean;
    cost: number;
    tokens: { input: number; output: number; reasoning: number; cache: { read: number; write: number } };
    finish?: string;
};
```

### v2 `SessionMessage` discriminated union (`v2/gen/types.gen.d.ts:3412`)
```typescript
type SessionMessage = SessionMessageAgentSwitched | SessionMessageModelSwitched | SessionMessageUser | SessionMessageSynthetic | SessionMessageSystem | SessionMessageShell | SessionMessageAssistant | SessionMessageCompaction;
```

### Request state fields

| Field | Semantics | UNKNOWN |
|-------|-----------|---------|
| `role` | `"user"` or `"assistant"` — discriminates message type | Never unknown |
| `time.completed` | `undefined` until request finishes; NOT `0` | `undefined` |
| `error` | Present only on failure; `undefined` on success | `undefined` — never `null` |
| `finish` | `string` reason string; `undefined` if not finished | `undefined` |
| `tokens` | Always present even during streaming | May have `0` values for unstarted dimensions |

---

## 3. MODEL STATE

**Origin**: `Model`, `Provider`, `ProviderConfig` types

### `Model` (v2 `v2/gen/types.gen.d.ts:1653`)
```typescript
type Model = {
    id: string;
    providerID: string;
    api: { id: string; url: string; npm: string };
    name: string;
    family?: string;
    capabilities: {
        temperature: boolean; reasoning: boolean; attachment: boolean; toolcall: boolean;
        input: { text: boolean; audio: boolean; image: boolean; video: boolean; pdf: boolean };
        output: { text: boolean; audio: boolean; image: boolean; video: boolean; pdf: boolean };
        interleaved: boolean | { field: "reasoning" | "reasoning_content" | "reasoning_text" | string };
    };
    cost: { input: number; output: number; cache: { read: number; write: number }; tiers?: [...]; experimentalOver200K?: {...} };
    limit: { context: number; input?: number; output: number };
    status: "alpha" | "beta" | "deprecated" | "active";
    options: { [key: string]: unknown };
    headers: { [key: string]: string };
    release_date: string;
    variants?: { [key: string]: { [key: string]: unknown } };
};
```

### `Provider` (v2 `v2/gen/types.gen.d.ts:1733`)
```typescript
type Provider = {
    id: string;
    name: string;
    source: "env" | "config" | "custom" | "api";
    env: Array<string>;
    key?: string;
    options: { [key: string]: unknown };
    models: { [key: string]: Model };
};
```

### Model state dimensions

| Dimension | Field | Semantics |
|-----------|-------|-----------|
| Availability | `status` | `active`/`beta`/`alpha`/`deprecated` — never fabricate a status |
| Selection | `model.id` + `providerID` + `variant?` | Current model selection; `variant` optional |
| Cost | `cost` | Per-token pricing; may be `0` for free tiers but field always exists |
| Capability | `capabilities` | Boolean flags — `false` is a real value, not unknown |
| Limit | `limit.context` | Context window size; `0` means unbounded or unknown |

### `ModelRef` (v2 `v2/gen/types.gen.d.ts:2390`)
```typescript
type ModelRef = { id: string; providerID: string; variant?: string };
```
Used in events where only a reference to the model is needed. `variant` is `undefined` when not specified.

---

## 4. STREAM STATE

**Origin**: Part types (`TextPart`, `ReasoningPart`, `FilePart`, `ToolPart`, `StepStartPart`, `StepFinishPart`, etc.)

### Part types (v1 `types.gen.d.ts:345`)
```typescript
type Part = TextPart | { type: "subtask"; prompt: string; description: string; agent: string } | ReasoningPart | FilePart | ToolPart | StepStartPart | StepFinishPart | SnapshotPart | PatchPart | AgentPart | RetryPart | CompactionPart;
```

### Stream state per part type

| Part type | `type` field | Stream semantics | Key state fields |
|-----------|-------------|-----------------|-----------------|
| Text | `"text"` | Streaming text content | `text`, `synthetic?`, `ignored?`, `time.start`, `time.end?` |
| Reasoning | `"reasoning"` | Chain-of-thought streaming | `text`, `time.start`, `time.end?` |
| File | `"file"` | File attachment in context | `mime`, `filename?`, `url`, `source?` |
| Tool | `"tool"` | Tool call lifecycle | `callID`, `tool`, `state: ToolState`, `metadata?` |
| Step start | `"step-start"` | Step boundary marker | `snapshot?` |
| Step finish | `"step-finish"` | Step completion | `reason`, `cost`, `tokens`, `snapshot?` |
| Snapshot | `"snapshot"` | Full state snapshot | `snapshot: string` |
| Patch | `"patch"` | Code patch | `hash`, `files: Array<string>` |
| Agent | `"agent"` | Sub-agent invocation | `name`, `source?` |
| Retry | `"retry"` | Retry event | `attempt`, `error: ApiError`, `time.created` |
| Compaction | `"compaction"` | Context compaction | `auto`, `overflow?`, `tail_start_id?` |
| Subtask | `"subtask"` | Sub-task creation | `prompt`, `description`, `agent`, `model?`, `command?` |

### `ToolState` (v1 `types.gen.d.ts:262`)
```typescript
type ToolState = ToolStatePending | ToolStateRunning | ToolStateCompleted | ToolStateError;
```

| Sub-state | `status` | Key fields |
|-----------|----------|-----------|
| Pending | `"pending"` | `input`, `raw` |
| Running | `"running"` | `input`, `title?`, `metadata?`, `time.start` |
| Completed | `"completed"` | `input`, `output`, `title`, `metadata`, `time.start/end`, `compacted?`, `attachments?` |
| Error | `"error"` | `input`, `error: string`, `metadata?`, `time.start/end` |

### Stream state semantics

- `delta` fields indicate partial updates; absence means no new data — NOT empty string
- `time.end` is `undefined` until the part completes; NEVER `0`
- `synthetic` and `ignored` are `boolean | undefined`; `undefined` means not applicable
- `metadata` is `{ [key: string]: unknown }`; `{}` is a valid empty object, distinct from `undefined`

---

## 5. TOOL STATE

**Origin**: `ToolPart`, `ToolState` types; `Hooks.tool` in plugin

### Tool execution hooks (`plugin/index.d.ts:179-258`)
```typescript
tool?: {
    [key: string]: ToolDefinition;
};
"tool.execute.before"?: (input: { tool: string; sessionID: string; callID: string; }, output: { args: any; }) => Promise<void>;
"tool.execute.after"?: (input: { tool: string; sessionID: string; callID: string; args: any; }, output: { title: string; output: string; metadata: any; }) => Promise<void>;
```

### Tool state transitions

```
before → pending → running → completed | error
              ↓
         cancelled (not in types — treated as error state)
```

### `ToolPart` (v1 `types.gen.d.ts:263`)
```typescript
type ToolPart = {
    id: string; sessionID: string; messageID: string; type: "tool";
    callID: string; tool: string; state: ToolState; metadata?: { [key: string]: unknown };
};
```

### v2 tool state (`v2/gen/types.gen.d.ts:3312-3367`)
```typescript
type SessionMessageToolStatePending = { status: "pending"; input: string; };
type SessionMessageToolStateRunning = { status: "running"; input: { [key: string]: unknown }; structured: { [key: string]: unknown }; content: Array<LlmToolContent>; };
type SessionMessageToolStateCompleted = { status: "completed"; input: { [key: string]: unknown }; attachments?: Array<PromptFileAttachment>; content: Array<LlmToolContent>; outputPaths?: Array<string>; structured: { [key: string]: unknown }; result?: unknown; };
type SessionMessageToolStateError = { status: "error"; input: { [key: string]: unknown }; content: Array<LlmToolContent>; structured: { [key: string]: unknown }; error: SessionErrorUnknown; result?: unknown; };
```

### Tool state semantics

| State | `status` value | Meaning |
|-------|---------------|---------|
| Pending | `"pending"` | Tool call queued, input known but not executed |
| Running | `"running"` | Tool executing; `structured`/`content` may be partial |
| Completed | `"completed"` | Tool finished successfully; `output`/`result` present |
| Error | `"error"` | Tool failed; `error` string present |

- `input` in `ToolStatePending` is `{ [key: string]: unknown }` with `raw: string`
- `input` in `ToolStateRunning/Completed/Error` is `{ [key: string]: unknown }` — the structured version
- `output` is `string` only in `ToolStateCompleted`; absent in error — NOT empty string

---

## 6. MCP SERVER STATE

**Origin**: `McpStatus` type (`v2/gen/types.gen.d.ts:1968-1985`)

```typescript
type McpStatusConnected = { status: "connected"; };
type McpStatusDisabled = { status: "disabled"; };
type McpStatusFailed = { status: "failed"; error: string; };
type McpStatusNeedsAuth = { status: "needs_auth"; };
type McpStatusNeedsClientRegistration = { status: "needs_client_registration"; error: string; };
type McpStatus = McpStatusConnected | McpStatusDisabled | McpStatusFailed | McpStatusNeedsAuth | McpStatusNeedsClientRegistration;
```

### MCP server state dimensions

| Field | Type | Semantics | UNKNOWN |
|-------|------|-----------|---------|
| `status` | `McpStatus` | Discriminated union — always one of 5 values | Must be a valid variant; never fabricate |
| `error` | `string \| undefined` | Present only in `failed`/`needs_client_registration` | `undefined` in all other states — NOT `""` |

### MCP configuration types

**v1** (`types.gen.d.ts:946-1011`):
```typescript
type McpLocalConfig = { type: "local"; command: Array<string>; environment?: { [key: string]: string }; enabled?: boolean; timeout?: number; };
type McpRemoteConfig = { type: "remote"; url: string; enabled?: boolean; headers?: { [key: string]: string }; oauth?: McpOAuthConfig | false; timeout?: number; };
```

**v2** (`v2/gen/types.gen.d.ts:1465-1506`):
```typescript
type McpLocalConfig = { type: "local"; command: Array<string>; cwd?: string; environment?: { [key: string]: string }; enabled?: boolean; timeout?: number; };
type McpRemoteConfig = { type: "remote"; url: string; enabled?: boolean; headers?: { [key: string]: string }; oauth?: McpOAuthConfig | false; timeout?: number; };
```

### MCP events
- `EventMcpToolsChanged`: `{ server: string }` — tools list changed for a server
- `EventMcpBrowserOpenFailed`: `{ mcpName: string; url: string }` — browser-based MCP failed to open
- `EventServerConnected`: `{ [key: string]: unknown }` — generic server connected

---

## 7. MCP MODEL ACCESS STATE

**Origin**: This is a derived state combining MCP server availability with model selection context.

### Definition

MCP Model Access State represents whether the current session can access models through MCP servers. It is derived from:

1. `McpStatus` — the server's connection state
2. `Provider.models` — available models per provider
3. `Agent.model` — which model an agent is configured to use

### State dimensions

| Dimension | Source | Semantics |
|-----------|--------|-----------|
| Server available | `McpStatus["status"] === "connected"` | Whether the MCP server is reachable |
| Tools available | `ToolList` from MCP | What tools the server exposes |
| Model selection | `AgentConfig.model` / `Session.model` | Which model is being used |
| Model variant | `Session.model.variant` | Specific model variant (v2) |
| Provider auth | `Provider.key?` | Whether provider credentials exist |

### UNKNOWN handling

- If `McpStatus` is `needs_auth`, model access is **blocked** — not "unknown"
- If `McpStatus` is `disabled`, the server is explicitly not available — NOT "unknown"
- `variant` being `undefined` means the default variant — NOT "no variant selected"

---

## 8. LSP STATE

**Origin**: `LspStatus` (`v2/gen/types.gen.d.ts:1957`)

```typescript
type LspStatus = {
    id: string;
    name: string;
    root: string;
    status: "connected" | "error";
};
```

### LSP state dimensions

| Field | Type | Semantics | UNKNOWN |
|-------|------|-----------|---------|
| `id` | `string` | Unique LSP server identifier | Never absent |
| `name` | `string` | Display name | Never absent |
| `root` | `string` | Project root directory | Never absent |
| `status` | `"connected" \| "error"` | Connection state | Must be one of two; never fabricate |

### LSP configuration (`v2/gen/types.gen.d.ts:1607`)
```typescript
lsp?: boolean | {
    [key: string]: { disabled: true } | { command: Array<string>; extensions?: Array<string>; disabled?: boolean; env?: { [key: string]: string }; initialization?: { [key: string]: unknown } };
};
```

### LSP events
- `EventLspUpdated`: `{ [key: string]: unknown }` — generic LSP state change
- `EventLspClientDiagnostics`: `{ serverID: string; path: string }` — diagnostic update

### TUI representation (`plugin/tui.d.ts:348`)
```typescript
type TuiSidebarLspItem = Pick<LspStatus, "id" | "root" | "status">;
```

### UNKNOWN handling
- `status: "error"` means the LSP server encountered an error — NOT "not connected"
- `status: "connected"` means operational — NOT "idle"
- The `initialization` field in config is `{ [key: string]: unknown }`; if absent, the server uses defaults

---

## 9. PERMISSION STATE

**Origin**: `Permission`, `PermissionRequest`, `PermissionConfig`, hooks

### v1 `Permission` (`types.gen.d.ts:369`)
```typescript
type Permission = {
    id: string;
    type: string;
    pattern?: string | Array<string>;
    sessionID: string;
    messageID: string;
    callID?: string;
    title: string;
    metadata: { [key: string]: unknown };
    time: { created: number };
};
```

### v2 `PermissionRequest` (`v2/gen/types.gen.d.ts:2032`)
```typescript
type PermissionRequest = {
    id: string;
    sessionID: string;
    permission: string;
    patterns: Array<string>;
    metadata: { [key: string]: unknown };
    always: Array<string>;
    tool?: { messageID: string; callID: string };
};
```

### v2 `PermissionV2Asked` event (`v2/gen/types.gen.d.ts:1023`)
```typescript
type PermissionV2Asked = {
    id: string; sessionID: string; action: string;
    resources: Array<string>; save?: Array<string>;
    metadata?: { [key: string]: unknown };
    source?: PermissionV2Source;
};
```

### Permission hooks (`plugin/index.d.ts:225-227`)
```typescript
"permission.ask"?: (input: Permission, output: { status: "ask" | "deny" | "allow"; }) => Promise<void>;
```

### Permission state dimensions

| Field | Type | Semantics | UNKNOWN |
|-------|------|-----------|---------|
| `id` | `string` | Unique permission request ID | Never absent |
| `status` | `"ask" \| "deny" \| "allow"` | Response status | Must be one of three; never default |
| `always` | `Array<string>` | Rules to remember | `[]` means no persistent rules — NOT `undefined` |
| `patterns` | `Array<string>` | Matched patterns | `[]` means no specific pattern |
| `tool` | `object \| undefined` | Originating tool call | `undefined` for non-tool permissions |
| `source` | `PermissionV2Source \| undefined` | v2 origin info | `undefined` when not from a tool |

### v2 Permission reply semantics
- `"once"`: Allow this one time
- `"always"`: Allow and remember
- `"reject"`: Deny permanently

---

## 10. WORKSPACE STATE

**Origin**: `Workspace`, `WorkspaceInfo`, `Project`, `Path`, `Config`

### v2 `Workspace` (`v2/gen/types.gen.d.ts:2179`)
```typescript
type Workspace = {
    id: string;
    type: string;
    name: string;
    branch?: string | null;
    directory?: string | null;
    extra?: unknown | null;
    projectID: string;
    timeUsed: number | "NaN" | "Infinity" | "-Infinity" | "NaN" | "Infinity" | "-Infinity" | "NaN";
};
```

### v2 `Project` (`v2/gen/types.gen.d.ts:1994`)
```typescript
type Project = {
    id: string; worktree: string; vcs?: ProjectVcs; name?: string; icon?: ProjectIcon;
    commands?: ProjectCommands; time: ProjectTime; sandboxes: Array<string>;
};
```

### `Path` (v2 `v2/gen/types.gen.d.ts:1895`)
```typescript
type Path = { home: string; state: string; config: string; worktree: string; directory: string };
```

### `WorkspaceInfo` (`plugin/index.d.ts:11`)
```typescript
type WorkspaceInfo = { id: string; type: string; name: string; branch: string | null; directory: string | null; extra: unknown | null; projectID: string; };
```

### Workspace state dimensions

| Field | Type | Semantics | UNKNOWN |
|-------|------|-----------|---------|
| `id` | `string` | Workspace identifier | Never absent |
| `type` | `string` | Workspace type | Never absent |
| `branch` | `string \| null` | Git branch | `null` when not a git repo — NOT `""` |
| `directory` | `string \| null` | Workspace path | `null` when not set |
| `name` | `string` | Display name | Never absent |
| `vcs` | `"git" \| undefined` | Version control type | `undefined` when no VCS |
| `sandboxes` | `Array<string>` | Sandbox identifiers | `[]` when none |

### Workspace events
- `EventWorkspaceReady`: `{ name: string }`
- `EventWorkspaceFailed`: `{ message: string }`
- `EventWorkspaceStatus`: `{ workspaceID: string; status: "connected" | "connecting" | "disconnected" | "error" }`
- `EventWorktreeReady`: `{ name: string; branch?: string }`
- `EventWorktreeFailed`: `{ message: string }`

---

## 11. GIT STATE

**Origin**: `VcsInfo`, `VcsFileStatus`, `VcsFileDiff`, `File`, `FileDiff`, `EventVcsBranchUpdated`, `EventFileEdited`, `EventFileWatcherUpdated`

### `VcsInfo` (v2 `v2/gen/types.gen.d.ts:1902`)
```typescript
type VcsInfo = { branch?: string; default_branch?: string };
```

### `FileDiff` (v1 `types.gen.d.ts:32`)
```typescript
type FileDiff = { file: string; before: string; after: string; additions: number; deletions: number };
```

### `VcsFileStatus` (v2 `v2/gen/types.gen.d.ts:1906`)
```typescript
type VcsFileStatus = { file: string; additions: number; deletions: number; status: "added" | "deleted" | "modified" };
```

### `File` (`types.gen.d.ts:1393`)
```typescript
type File = { path: string; added: number; removed: number; status: "added" | "deleted" | "modified" };
```

### Git state dimensions

| Field | Type | Semantics | UNKNOWN |
|-------|------|-----------|---------|
| `branch` | `string \| undefined` | Current branch | `undefined` when no branch detected |
| `default_branch` | `string \| undefined` | Default branch name | `undefined` when no repo |
| `status` | `"added" \| "deleted" \| "modified"` | File change type | Must be one of three; never fabricate |
| `additions` | `number` | Added lines | `0` is valid; not unknown |
| `deletions` | `number` | Deleted lines | `0` is valid; not unknown |
| `before` | `string` | Previous file content/hash | Empty string when new file |
| `after` | `string` | New file content/hash | Empty string when deleted file |

### Git events
- `EventVcsBranchUpdated`: `{ branch?: string }`
- `EventFileEdited`: `{ file: string }`
- `EventFileWatcherUpdated`: `{ file: string; event: "add" | "change" | "unlink" }`
- `EventSessionDiff`: `{ sessionID: string; diff: Array<FileDiff> }`
- `EventSessionCompacted`: `{ sessionID: string }`

### UNKNOWN handling
- `branch?: string` — `undefined` means no branch detected; NOT `""`
- `VcsInfo.branch` is `string \| undefined` in v2; `string` (required) in v1 `VcsInfo`
- File watcher events: `"add"`, `"change"`, `"unlink"` are exhaustive; no fallback

---

## 12. UI STATE

**Origin**: `TuiState`, `TuiPluginState`, `TuiTheme`, `TuiKV`, `TuiAttention`, `Config`

### `TuiState` (`plugin/tui.d.ts:287`)
```typescript
type TuiState = {
    readonly ready: boolean;
    readonly config: SdkConfig;
    readonly provider: ReadonlyArray<Provider>;
    readonly path: { state: string; config: string; worktree: string; directory: string };
    readonly vcs: { branch?: string; default_branch?: string } | undefined;
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
};
```

### `TuiSidebarMcpItem` (`plugin/tui.d.ts:343`)
```typescript
type TuiSidebarMcpItem = { name: string; status: McpStatus["status"]; error?: string };
```

### `TuiSidebarLspItem` (`plugin/tui.d.ts:348`)
```typescript
type TuiSidebarLspItem = Pick<LspStatus, "id" | "root" | "status">;
```

### `TuiPluginState` (`plugin/tui.d.ts:417`)
```typescript
type TuiPluginState = "first" | "updated" | "same";
```

### `TuiPluginStatus` (`plugin/tui.d.ts:435`)
```typescript
type TuiPluginStatus = { id: string; source: TuiPluginEntry["source"]; spec: string; target: string; enabled: boolean; active: boolean };
```

### UI state dimensions

| Dimension | Source | Fields | Semantics |
|-----------|--------|--------|-----------|
| Ready | `TuiState.ready` | `boolean` | Whether TUI is initialized |
| Config | `TuiState.config` | `SdkConfig` | Full configuration object |
| Providers | `TuiState.provider` | `ReadonlyArray<Provider>` | Available providers |
| Session list | `TuiState.session.count()` | `number` | Active session count |
| Current session | `TuiState.session.get()` | `Session \| undefined` | `undefined` when no session selected |
| MCP items | `TuiState.mcp()` | `ReadonlyArray<TuiSidebarMcpItem>` | Sidebar MCP status |
| LSP items | `TuiState.lsp()` | `ReadonlyArray<TuiSidebarLspItem>` | Sidebar LSP status |
| Theme | `TuiTheme` | `current`, `selected`, `ready` | UI theme state |
| Plugins | `TuiPluginStatus[]` | `id`, `source`, `spec`, `target`, `enabled`, `active` | Plugin lifecycle |
| KV | `TuiKV` | `get`, `set`, `ready` | Persistent key-value store |
| Attention | `TuiAttention` | `notify`, `soundboard` | Notification system |
| Route | `TuiRouteCurrent` | `name`, `params` | Current navigation route |
| Dialog | `TuiDialogStack` | `size`, `depth`, `open` | Modal dialog stack |

### `TuiRouteCurrent` (`plugin/tui.d.ts:13`)
```typescript
type TuiRouteCurrent = { name: "home" } | { name: "session"; params: { sessionID: string; prompt?: unknown } } | { name: string; params?: Record<string, unknown> };
```

### UNKNOWN handling in UI state
- `TuiState.session.get(sessionID)` returns `Session | undefined` — `undefined` means no session; NOT a default session
- `TuiState.session.status(sessionID)` returns `SessionStatus | undefined` — `undefined` means status unknown
- `TuiState.vcs` is `{ branch?: string; default_branch?: string } | undefined` — outer `undefined` means VCS not available
- `TuiPluginState` is a closed union: `"first" | "updated" | "same"` — no fallback
- `TuiTheme.mode()` returns `"dark" | "light"` — never unknown

---

## Cross-cutting state principles

1. **`UNKNOWN != ZERO`**: Every field that can be absent uses `undefined` (or the discriminated union pattern). `0`, `""`, and `false` are real values, not placeholders for missing data.
2. **Discriminated unions**: `SessionStatus`, `McpStatus`, `ToolState`, `PermissionAction` all use discriminant fields (`type`, `status`) to enable exhaustive pattern matching.
3. **Optional fields**: Fields marked `?` in TypeScript are genuinely optional — their absence is semantically meaningful, not a bug.
4. **Immutability**: TUI state is `readonly`; plugins receive snapshots, not mutable references.
