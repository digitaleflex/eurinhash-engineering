# State Machine — Finite State Transitions

> Source: `@opencode-ai/sdk/dist/gen/types.gen.d.ts`, `@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`
> Rule: Each state type has its own machine. Never collapse distinct state machines.

---

## State machine principles

1. **Discriminated unions** define valid states and transitions
2. **Each domain has its own machine** — they are independent
3. **Transitions are event-driven** — events trigger state changes
4. **No global state machine** — the 12 state types are separate

---

## 1. Session State Machine

### States

```
SessionStatus = { type: "idle" } | { type: "retry"; attempt: number; message: string; next: number; action?: {...} } | { type: "busy" }
```

### Transition diagram

```
idle ──user message──► busy ──request complete──► idle
  │                        │
  │                        ├──step failure──► retry (if retryable)
  │                        │                  │
  │                        │                  └──max retries──► idle + error
  │                        │
  └──compact──► idle ──compact complete──► idle
```

### Transition table

| From | Event | To | Condition |
|------|-------|-----|-----------|
| idle | user message | busy | Message accepted |
| busy | step failed | retry | `retryable` error |
| busy | step failed | idle | Non-retryable error |
| retry | next attempt | busy | `next` timestamp reached |
| retry | max retries | idle | All retries exhausted |
| busy | request complete | idle | All messages processed |
| idle | compact | idle | Compaction starts |
| any | session.delete | terminal | Session removed |

### v2 extended states

```
session.next.* events represent sub-states within busy:
  busy → step_started → (text/reasoning/tool phases) → step_ended → busy
  busy → compaction_started → compaction_ended → busy
  busy → revert_staged → revert_committed → busy
```

---

## 2. Request State Machine

### States (per message)

```
created → parts_accumulating → (text | reasoning | tool | step) → completed
                                              ↓
                                         error
```

### Transition table

| From | Event | To | Condition |
|------|-------|-----|-----------|
| created | first part | parts_accumulating | Any part emitted |
| parts_accumulating | text.started | text_streaming | `session.next.text.started` |
| text_streaming | text.delta | text_streaming | `session.next.text.delta` |
| text_streaming | text.ended | parts_accumulating | `session.next.text.ended` |
| parts_accumulating | reasoning.started | reasoning_streaming | `session.next.reasoning.started` |
| reasoning_streaming | reasoning.ended | parts_accumulating | `session.next.reasoning.ended` |
| parts_accumulating | tool.called | tool_running | `session.next.tool.called` |
| tool_running | tool.progress | tool_running | `session.next.tool.progress` |
| tool_running | tool.success | parts_accumulating | `session.next.tool.success` |
| tool_running | tool.failed | parts_accumulating | `session.next.tool.failed` |
| parts_accumulating | step.started | step_active | `session.next.step.started` |
| step_active | step.ended | parts_accumulating | `session.next.step.ended` |
| step_active | step.failed | error | `session.next.step.failed` |
| parts_accumulating | step.failed | error | `session.next.step.failed` |
| any | error | error | `session.error` |
| completed | — | terminal | No further parts |

### v2 message-level state machine

```
SessionMessage types form a linear sequence:
  User → AgentSwitched → ModelSwitched → Shell → Assistant → Compaction → (repeats)
  Synthetic → (interleaved)
```

Each `SessionMessageAssistant` contains:
```
text_parts → reasoning_parts → tool_parts → finish
```

---

## 3. Tool State Machine

### States (`ToolState` discriminated union)

```
ToolState = ToolStatePending | ToolStateRunning | ToolStateCompleted | ToolStateError
```

### Transition diagram

```
pending ──execute──► running ──success──► completed
  │                      │
  │                      └──failure──► error
  │
  └──cancel──► error
```

### Transition table

| From | Event | To | Condition |
|------|-------|-----|-----------|
| pending | `tool.execute.before` hook | running | Hook returns allow |
| pending | `tool.execute.before` hook | pending | Hook returns deny |
| running | `tool.progress` | running | Progress update |
| running | `tool.success` | completed | Tool finished successfully |
| running | `tool.failed` | error | Tool returned error |
| running | timeout | error | Execution timed out |
| error | retry | running | Retry initiated |
| completed | — | terminal | Final state |
| error | — | terminal (unless retried) | Final state |

### v2 session message tool state machine

```
SessionMessageToolStatePending → SessionMessageToolStateRunning → SessionMessageToolStateCompleted | SessionMessageToolStateError
```

Additional v2 fields in `running`: `structured`, `content: Array<LlmToolContent>`
Additional v2 fields in `completed`: `outputPaths`, `result`, `attachments`
Additional v2 fields in `error`: `error: SessionErrorUnknown`

---

## 4. MCP Server State Machine

### States (`McpStatus` discriminated union)

```
McpStatus = McpStatusConnected | McpStatusDisabled | McpStatusFailed | McpStatusNeedsAuth | McpStatusNeedsClientRegistration
```

### Transition diagram

```
disabled ──enable──► connecting ──connected──► connected
disabled ←─────────────────────── failed ←────┘
needs_auth ←──────────────────── failed ←────┘
needs_client_registration ←─────────────────── failed ←────┘
```

### Transition table

| From | Event | To | Condition |
|------|-------|-----|-----------|
| disabled | server enabled | connecting | `enabled: true` |
| connecting | connection success | connected | Server responds |
| connecting | connection failure | failed | Timeout or error |
| connected | connection lost | failed | Transport error |
| connected | auth required | needs_auth | Server requests auth |
| failed | retry success | connected | Connection restored |
| failed | retry failure | failed | Still failing |
| needs_auth | auth success | connected | OAuth/API auth succeeds |
| needs_auth | auth failure | failed | Auth rejected |
| needs_client_registration | registration success | connected | Client registered |
| needs_client_registration | registration failure | failed | Registration rejected |
| any | server disabled | disabled | Explicitly disabled |

### MCP configuration state

```
Config → McpLocalConfig | McpRemoteConfig
  → enabled?: boolean (default: from config)
  → timeout?: number (default: 5000ms)
  → type: "local" | "remote"
```

---

## 5. LSP State Machine

### States (`LspStatus`)

```
LspStatus = { status: "connected" } | { status: "error" }
```

### Transition diagram

```
error ──start──► connecting ──success──► connected
  ↑                │
  └────────────────┘
```

### Transition table

| From | Event | To | Condition |
|------|-------|-----|-----------|
| error | LSP started | connecting | Server binary launched |
| connecting | initialization success | connected | `initialize` response received |
| connecting | initialization failure | error | Parse error or timeout |
| connected | crash | error | Process terminated unexpectedly |
| connected | file change | connected | Diagnostics may update |
| any | config disabled | error | Explicitly disabled |

---

## 6. Permission State Machine

### States

```
PermissionAction = "ask" | "allow" | "deny"
PermissionStatus = pending (asked) → resolved (replied)
```

### Transition diagram (v2)

```
asked ──reply "once"──► resolved (one-time allow)
  │                      (tool can execute once)
asked ──reply "always"──► resolved (persistent allow)
  │                        (tool executes; rule saved)
asked ──reply "reject"──► rejected
  │                        (tool blocked)
```

### Transition table

| From | Event | To | Condition |
|------|-------|-----|-----------|
| pending | `permission.asked` | pending | New permission request |
| pending | `permission.v2.asked` | pending | New v2 permission request |
| pending | `permission.replied` (once) | resolved | One-time allowance |
| pending | `permission.replied` (always) | resolved + rule_saved | Persistent allowance |
| pending | `permission.replied` (reject) | rejected | Explicit denial |
| pending | timeout | expired | No response within timeout |
| resolved | — | terminal | Permission granted |
| rejected | — | terminal | Permission denied |

### v1 vs v2 difference

- v1: `Permission` object with `type`, `pattern`, `metadata`, `time`
- v2: `PermissionRequest` with `id`, `sessionID`, `permission`, `patterns`, `always`, `tool?`
- v2 adds `PermissionV2Reply = "once" | "always" | "reject"`

---

## 7. Model State Machine

### States

Model state is not a state machine per se, but model selection follows a pattern:

```
default → selected → active → (switch) → selected
```

### Transition table

| From | Event | To | Condition |
|------|-------|-----|-----------|
| default | `session.next.model.switched` | selected | User or agent selects model |
| selected | request starts | active | Model is in use |
| active | request completes | selected | Model idle |
| selected | `session.next.model.switched` | selected | Model changed |
| any | provider failure | error | Provider unavailable |
| error | provider restored | selected | Provider available again |

### v2 model variants

```
Model.variants?: { [key: string]: { [key: string]: unknown } }
  → variant selection is via `Session.model.variant`
  → variant changes emit `session.next.model.switched`
```

---

## 8. Stream State Machine

### States (per stream part)

```
text: not_started → streaming → ended → terminal
reasoning: not_started → streaming → ended → terminal
tool_input: not_started → streaming → ended → terminal
compaction: not_started → streaming → ended → terminal
```

### Transition table

| From | Event | To | Condition |
|------|-------|-----|-----------|
| not_started | `.started` | streaming | Stream begins |
| streaming | `.delta` | streaming | Data arrives |
| streaming | `.ended` | ended | Stream complete |
| ended | — | terminal | Final state |
| streaming | error/error | ended | Stream terminated with error |

### Ordering guarantee
- Deltas within a stream MUST be applied in order
- Skipping a delta breaks the stream state
- `time.start` is always before `time.end`

---

## 9. Workspace State Machine

### States

```
WorkspaceStatus = "connected" | "connecting" | "disconnected" | "error"
```

### Transition diagram

```
disconnected ──connect──► connecting ──success──► connected
  │                         │
  └─────────────────────────┴──error──► error
  │                                    │
  └────────────────────────────────────┘
```

### Transition table

| From | Event | To | Condition |
|------|-------|-----|-----------|
| disconnected | connect request | connecting | `workspace.status` → `"connecting"` |
| connecting | success | connected | `workspace.ready` event |
| connecting | failure | error | `workspace.failed` event |
| connected | disconnect | disconnected | Explicit disconnect |
| connected | error | error | Connection lost |
| error | reconnect | connecting | Retry initiated |

### Worktree sub-machine

```
Worktree states mirror workspace but are worktree-specific:
  worktree.ready → worktree.failed → worktree.ready (retry)
```

---

## 10. Git State Machine

### States

Git state is event-driven, not a finite state machine, but branch changes follow:

```
branch_a → branch_switch → branch_b
```

### Transition table

| From | Event | To | Condition |
|------|-------|-----|-----------|
| any branch | `vcs.branch.updated` | new branch | Branch changed |
| any branch | `file.watcher.updated` | same branch | File change (not branch change) |
| any | `session.diff` | same branch | Diff computed |

### File status state machine

```
untracked → added (on git add)
added → modified (on file change)
modified → deleted (on file removal)
deleted → untracked (on git rm)
```

---

## 11. UI State Machine

### States

UI state is not a single state machine but multiple sub-machines:

**Route state**: `home → session → (session | home)`
**Dialog state**: `closed → open → closed` (stack-based)
**Plugin state**: `first | updated | same`
**Theme state**: `dark | light`

### TuiState machine

```
ready: false → ready: true (after initialization)
  → session.count() changes → session.get() changes
  → lsp() changes → mcp() changes
```

### TuiPluginState transitions

| From | Event | To | Condition |
|------|-------|-----|-----------|
| first | plugin installed | same | First install |
| same | plugin updated | updated | Content changed |
| updated | plugin unchanged | same | Content reverted |

---

## 12. Compaction State Machine

### States

```
SessionNextCompactionStarted → SessionNextCompactionDelta* → SessionNextCompactionEnded
```

### Transition table

| From | Event | To | Condition |
|------|-------|-----|-----------|
| not_compacting | compaction.started | compaction_started | Trigger condition met |
| compaction_started | compaction.delta | compaction_started | Delta arrives |
| compaction_started | compaction.ended | not_compacting | Compaction complete |
| compaction_started | error | error | Compaction failed |

### Auto vs manual
- `reason: "auto"` — triggered by context limits
- `reason: "manual"` — triggered by user command

---

## Cross-machine interactions

```
Session State Machine ──triggers──► Request State Machine
  │                                        │
  ├──triggers──► Tool State Machine         │
  │                                        │
  ├──triggers──► MCP Server State Machine   │
  │                                        │
  ├──triggers──► LSP State Machine          │
  │                                        │
  ├──triggers──► Permission State Machine   │
  │                                        │
  └──triggers──► Stream State Machine       │
                                              │
Model State Machine ──selects──► Request State Machine
                                              │
Workspace State Machine ──provides──► Session State Machine
Git State Machine ──provides──► Session State Machine
```

### Interaction rules
1. **Session → Request**: Session must be `busy` before request starts
2. **Request → Tool**: Request emits tool events; tool state is independent
3. **MCP → Tool**: MCP server must be `connected` for MCP-based tools
4. **LSP → Workspace**: LSP runs within workspace context
5. **Permission → Request**: Permission must be resolved before tool execution
6. **Stream → Request**: Stream parts accumulate into the request message

---

## State machine invariants

1. **SessionStatus is always valid**: Must be one of the three discriminated variants
2. **ToolState is always valid**: Must be one of the four sub-states
3. **McpStatus is always valid**: Must be one of the five sub-states
4. **LspStatus is always valid**: Must be `"connected"` or `"error"`
5. **Part types are exhaustive**: `Part` union covers all part types
6. **Message roles are exhaustive**: `Message = UserMessage | AssistantMessage`
7. **Error types are exhaustive**: All error types are listed in union
8. **No silent state transitions**: Every state change is triggered by an event
9. **`UNKNOWN != ZERO`**: Missing fields are `undefined`, never `0` or `""`
