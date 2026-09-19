# Presentation State — Selectors & Adapters

> Source: `@opencode-ai/plugin/dist/tui.d.ts`, `@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`
> Rule: Presentation state is DERIVED from domain state. Never a second source of truth.

---

## TuiState — The single presentation API

`TuiState` is the read-only facade that all TUI components consume. It derives from domain types but never modifies them.

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

### Selector properties

| Selector | Returns | Source type | Derived from |
|----------|---------|-------------|-------------|
| `ready` | `boolean` | — | Initialization state |
| `config` | `SdkConfig` | `Config` | Full config object |
| `provider` | `ReadonlyArray<Provider>` | `Provider[]` | All registered providers |
| `path` | `Path` object | `Path` | File system paths |
| `vcs` | `VcsInfo \| undefined` | `VcsInfo` | Git branch info |
| `session.count()` | `number` | `Session[]` | Count of active sessions |
| `session.get(id)` | `Session \| undefined` | `Session` | Session by ID; `undefined` if not found |
| `session.diff(id)` | `ReadonlyArray<TuiSidebarFileItem>` | `FileDiff[]` | Diffs for sidebar display |
| `session.todo(id)` | `ReadonlyArray<TuiSidebarTodoItem>` | `Todo[]` | Todos for sidebar |
| `session.messages(id)` | `ReadonlyArray<Message>` | `Message[]` | All messages in session |
| `session.status(id)` | `SessionStatus \| undefined` | `SessionStatus` | Current session status |
| `session.permission(id)` | `ReadonlyArray<PermissionRequest>` | `PermissionRequest[]` | Pending permissions |
| `session.question(id)` | `ReadonlyArray<QuestionRequest>` | `QuestionRequest[]` | Pending questions |
| `part(messageID)` | `ReadonlyArray<Part>` | `Part[]` | Parts for a message |
| `lsp()` | `ReadonlyArray<TuiSidebarLspItem>` | `LspStatus[]` | LSP status for sidebar |
| `mcp()` | `ReadonlyArray<TuiSidebarMcpItem>` | `McpStatus[]` | MCP status for sidebar |

### Selector semantics

**All selectors are read-only functions, not mutable state.** They derive from the underlying domain types and return new arrays/objects each call.

---

## Sidebar adapters

### `TuiSidebarMcpItem` adapter

```typescript
// Source: McpStatus
// Derived: TuiSidebarMcpItem
type TuiSidebarMcpItem = {
    name: string;           // From McpStatus config
    status: McpStatus["status"];  // Discriminated from McpStatus
    error?: string;         // Present only if McpStatusFailed or McpStatusNeedsClientRegistration
};
```

**Adapter logic**:
- Maps `McpStatus` discriminated union to a flat sidebar item
- `status` is `McpStatus["status"]` — a closed union of 5 strings
- `error` is `undefined` when status is `connected`, `disabled`, or `needs_auth`
- `error` is a non-empty string when status is `failed` or `needs_client_registration`

**UNKNOWN handling**: `error` is `undefined` when absent — NOT `""`. Status must be one of the 5 valid values.

### `TuiSidebarLspItem` adapter

```typescript
// Source: LspStatus
// Derived: TuiSidebarLspItem
type TuiSidebarLspItem = Pick<LspStatus, "id" | "root" | "status">;
```

**Adapter logic**:
- Selects 3 fields from `LspStatus`
- Drops `name` field (not needed in sidebar)
- `status` is `"connected" | "error"` — never fabricated

### `TuiSidebarTodoItem` adapter

```typescript
// Source: Todo
// Derived: TuiSidebarTodoItem
type TuiSidebarTodoItem = Pick<Todo, "content" | "status">;
```

**Adapter logic**:
- Selects only `content` and `status` from `Todo`
- Drops `priority` and `id` (not needed in sidebar)
- `status` is `string` from `Todo.status`

### `TuiSidebarFileItem` adapter

```typescript
// Source: FileDiff / VcsFileStatus
// Derived: TuiSidebarFileItem
type TuiSidebarFileItem = {
    file: string;
    additions: number;
    deletions: number;
};
```

**Adapter logic**:
- Maps diff data to sidebar display format
- `additions` and `deletions` are always present as numbers (may be `0`)

---

## Message rendering adapters

### Message to display adapter

```typescript
// Source: Message (UserMessage | AssistantMessage)
// Derived: DisplayMessage[]
// For each Message, extract:
//   - id, role, time, agent, model
//   - For AssistantMessage: cost, tokens, finish, error
//   - For UserMessage: summary, tools, system
```

### Part to display adapter

```typescript
// Source: Part (union type)
// Derived: DisplayPart[]
// Each Part type maps to a UI component:
//   TextPart → TextBlock
//   ReasoningPart → ThinkingBlock
//   FilePart → FileAttachment
//   ToolPart → ToolCallCard
//   StepStartPart → StepBoundary
//   StepFinishPart → StepResult
//   SnapshotPart → SnapshotView
//   PatchPart → PatchView
//   AgentPart → SubAgentIndicator
//   RetryPart → RetryIndicator
//   CompactionPart → CompactionNotice
```

### Tool state to display adapter

```typescript
// Source: ToolState (4 sub-states)
// Derived: ToolDisplayProps
//   ToolStatePending → Pending icon + input preview
//   ToolStateRunning → Running spinner + title
//   ToolStateCompleted → Checkmark + output
//   ToolStateError → Error icon + error message
```

---

## Session list adapter

```typescript
// Source: Session[]
// Derived: SessionListItem[]
// For each Session:
//   - id, title, time.updated, agent, model
//   - status (from SessionStatus event)
//   - unread count (from messages)
//   - cost (from Session.cost)
```

**Selector**: `session.count()` + `session.get(id)` → `SessionListItem[]`

---

## Route adapter

```typescript
// Source: TuiRouteCurrent
// Derived: Navigation state
type TuiRouteCurrent = 
  | { name: "home" }
  | { name: "session"; params: { sessionID: string; prompt?: unknown } }
  | { name: string; params?: Record<string, unknown> };
```

**Adapter logic**:
- `name: "home"` → Home screen
- `name: "session"` → Session view with sessionID
- Other names → Plugin-defined routes

---

## Dialog stack adapter

```typescript
// Source: TuiDialogStack
// Derived: Modal overlay state
type TuiDialogStack = {
    replace: (render: () => JSX.Element, onClose?: () => void) => void;
    clear: () => void;
    setSize: (size: "medium" | "large" | "xlarge") => void;
    readonly size: "medium" | "large" | "xlarge";
    readonly depth: number;
    readonly open: boolean;
};
```

**Adapter logic**:
- Stack-based: `replace` pushes, `clear` empties
- `depth` reflects number of open dialogs
- `size` applies to the topmost dialog

---

## Theme adapter

```typescript
// Source: TuiTheme
// Derived: CSS variables, color palette
type TuiTheme = {
    readonly current: TuiThemeCurrent;  // All RGBA color values
    readonly selected: string;          // Theme name
    readonly ready: boolean;            // Theme loaded
    readonly mode: () => "dark" | "light";  // Current mode
};
```

**Adapter logic**:
- `TuiThemeCurrent` contains all color tokens as `RGBA` objects
- `mode()` returns `"dark"` or `"light"` — never unknown
- All colors are `RGBA` with r, g, b, a as numbers

### `TuiThemeCurrent` color tokens

All color tokens are `RGBA`:
- `primary`, `secondary`, `accent`, `error`, `warning`, `success`, `info`
- `text`, `textMuted`
- `background`, `backgroundPanel`, `backgroundElement`, `backgroundMenu`
- `border`, `borderActive`, `borderSubtle`
- Diff colors: `diffAdded`, `diffRemoved`, `diffContext`, etc.
- Markdown colors: `markdownText`, `markdownHeading`, etc.
- Syntax colors: `syntaxComment`, `syntaxKeyword`, etc.
- `thinkingOpacity: number`

---

## Plugin state adapter

```typescript
// Source: TuiPluginEntry[]
// Derived: TuiPluginStatus[]
type TuiPluginStatus = {
    id: string;
    source: "file" | "npm" | "internal";
    spec: string;
    target: string;
    enabled: boolean;
    active: boolean;
};
```

**Adapter logic**:
- Filters to only relevant fields
- `TuiPluginState` ("first" | "updated" | "same") is metadata, not in `TuiPluginStatus`

---

## KV adapter

```typescript
// Source: TuiKV
// Derived: Persistent state
type TuiKV = {
    get: <Value = unknown>(key: string, fallback?: Value) => Value;
    set: (key: string, value: unknown) => void;
    readonly ready: boolean;
};
```

**Adapter logic**:
- `get` returns `fallback` if key not found — NOT `undefined`
- `ready` indicates whether KV is initialized

---

## Attention adapter

```typescript
// Source: TuiAttention
// Derived: Notification system
type TuiAttention = {
    notify(input: TuiAttentionNotifyInput): Promise<TuiAttentionNotifyResult>;
    soundboard: TuiAttentionSoundboard;
};
```

**Sound names** (closed set): `default`, `question`, `permission`, `error`, `done`, `subagent_done`

**Notification skip reasons** (closed set): `attention_disabled`, `empty_message`, `blurred`, `focused`, `focus_unknown`, `renderer_destroyed`

---

## Input prompt adapter

```typescript
// Source: TuiPromptInfo
// Derived: Prompt display
type TuiPromptInfo = {
    input: string;
    mode?: "normal" | "shell";
    parts: Array<FilePart | AgentPart | TextPart>;  // Omitted id/messageID/sessionID
};
```

**Adapter logic**:
- `parts` strips internal IDs for display
- `mode` is `undefined` when not specified — NOT `"normal"`

---

## Keybind adapter

```typescript
// Source: KeybindsConfig (from Config)
// Derived: TuiKeymap
// Keybinds are parsed into keymap bindings
```

**Keybind categories** (from `KeybindsConfig`):
- Navigation: `session.page.up`, `session.page.down`, `messages_first`, `messages_last`, etc.
- Session: `session.new`, `session.list`, `session.share`, `session.interrupt`, `session.compact`
- Agent: `agent.cycle`, `agent_list`
- Model: `model_list`, `model_cycle_recent`
- Input: `input_clear`, `input_paste`, `input_submit`, `input_newline`
- History: `history_previous`, `history_next`
- Terminal: `terminal_suspend`, `terminal_title_toggle`

---

## Presentation invariant: NEVER fabricate

| Presentation field | What to do when unknown | NEVER |
|-------------------|------------------------|-------|
| `session.get(id)` returns `undefined` | Show "No session" | Show empty session object |
| `session.status(id)` returns `undefined` | Show "Unknown" | Show `{ type: "idle" }` |
| `part(messageID)` returns `[]` | Show empty state | Show placeholder parts |
| `lsp()` returns `[]` | Show "No LSP" | Show fake LSP entry |
| `mcp()` returns `[]` | Show "No MCP" | Show fake MCP entry |
| `config` field missing | Use default | Fabricate default |
| `provider` returns `[]` | Show "No providers" | Show mock provider |
| `vcs` is `undefined` | Show "No VCS" | Show `{ branch: "" }` |

---

## Selector composition patterns

```typescript
// Pattern 1: Session + Status + Messages
const sessionView = (state: TuiState, id: string) => ({
    session: state.session.get(id),
    status: state.session.status(id),
    messages: state.session.messages(id),
});

// Pattern 2: MCP + LSP sidebar
const sidebar = (state: TuiState) => ({
    mcp: state.mcp(),
    lsp: state.lsp(),
    todos: state.session.todo(activeSessionId),
    files: state.session.diff(activeSessionId),
});

// Pattern 3: Current session overview
const overview = (state: TuiState) => {
    const sessions = state.session.count();
    // Derive from all selectors, never store separately
    return { sessionCount: sessions, ... };
};
```

### Rule: composition never creates new state
Selectors compose existing selectors. The result is always derivable from `TuiState`.
