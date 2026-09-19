# Event Catalog — 25+ Events with Full Semantics

> Source: `@opencode-ai/sdk/dist/gen/types.gen.d.ts`, `@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`, `@opencode-ai/plugin/dist/index.d.ts`
> Rule: Every event documents producer, payload, consumer, ordering guarantees, replay behavior, and failure mode.

---

## Event taxonomy

Events are categorized by domain:

| Category | Event count | Examples |
|----------|------------|----------|
| Session lifecycle | 5 | `session.created`, `session.updated`, `session.deleted`, `session.status`, `session.idle` |
| Message lifecycle | 4 | `message.updated`, `message.removed`, `message.part.updated`, `message.part.removed` |
| Request/stream | 30+ | `session.next.*` (v2), `message.part.delta` |
| Tool | 6 | `session.next.tool.called`, `.progress`, `.success`, `.failed`, `.input.started`, `.input.ended` |
| Permission | 4 | `permission.asked`, `permission.replied`, `permission.v2.asked`, `permission.v2.replied` |
| MCP | 3 | `mcp.tools.changed`, `mcp.browser.open.failed`, `server.connected` |
| LSP | 2 | `lsp.updated`, `lsp.client.diagnostics` |
| File/Git | 5 | `file.edited`, `file.watcher.updated`, `vcs.branch.updated`, `session.diff`, `session.compacted` |
| Todo | 1 | `todo.updated` |
| Command | 1 | `command.executed` |
| TUI | 6 | `tui.prompt.append`, `tui.command.execute`, `tui.toast.show`, `tui.session.select`, `tui.prompt.append2`, `tui.command.execute2`, `tui.toast.show2` |
| Workspace | 5 | `workspace.ready`, `workspace.failed`, `workspace.status`, `worktree.ready`, `worktree.failed` |
| Installation | 2 | `installation.updated`, `installation.update-available` |
| Pty | 4 | `pty.created`, `pty.updated`, `pty.exited`, `pty.deleted` |
| Question | 5 | `question.asked`, `question.replied`, `question.rejected`, `question.v2.asked`, `question.v2.replied`, `question.v2.rejected` |
| Plugin | 1 | `plugin.added` |
| Project | 1 | `project.updated` |
| Error | 1 | `session.error` |
| Sync | 16+ | `sync.*` (v2 durable events) |
| Server | 2 | `server.connected`, `server.instance.disposed`, `global.disposed` |

---

## 1. Session lifecycle events

### `session.created`
- **Producer**: Session manager
- **Payload** (`EventSessionCreated`): `{ info: Session }`
- **Consumer**: All subscribers, TUI sidebar
- **Ordering**: First event for a new session; precedes all other session events
- **Replay**: Replay creates a new session record; idempotent
- **Failure mode**: If session creation fails, `session.error` is emitted instead

### `session.updated`
- **Producer**: Session manager
- **Payload** (`EventSessionUpdated`): `{ info: Session }`
- **Consumer**: TUI sidebar, session list
- **Ordering**: After `session.created`; whenever session metadata changes
- **Replay**: Updates existing session; idempotent
- **Failure mode**: If update fails, `session.error` is emitted

### `session.deleted`
- **Producer**: Session manager
- **Payload** (`EventSessionDeleted`): `{ info: Session }`
- **Consumer**: All subscribers, TUI sidebar
- **Ordering**: Terminal event for a session; after `session.updated`
- **Replay**: Removes session from store; idempotent
- **Failure mode**: If deletion fails, `session.error` is emitted

### `session.status`
- **Producer**: Session runtime
- **Payload** (`EventSessionStatus`): `{ sessionID: string; status: SessionStatus }`
- **Consumer**: TUI status bar, session manager
- **Ordering**: Transitions between `idle`, `busy`, `retry`
- **Replay**: Latest status wins; stale status discarded
- **Failure mode**: If status update fails, session remains in previous state

### `session.idle`
- **Producer**: Session runtime when request completes with no further action
- **Payload** (`EventSessionIdle`): `{ sessionID: string }`
- **Consumer**: TUI prompt, session manager
- **Ordering**: After `session.status` → `{ type: "idle" }`
- **Replay**: Re-marks session as idle; idempotent

### `session.compacted`
- **Producer**: Compaction service
- **Payload** (`EventSessionCompacted`): `{ sessionID: string }`
- **Consumer**: Session manager, TUI
- **Ordering**: After `session.next.compaction.ended`
- **Replay**: Re-marks session as compacted; idempotent

### `session.error`
- **Producer**: Session runtime
- **Payload** (`EventSessionError`): `{ sessionID?: string; error?: ProviderAuthError | UnknownError | MessageOutputLengthError | MessageAbortedError | ApiError }`
- **Consumer**: Error handler, TUI toast
- **Ordering**: Can occur at any point; terminal for the current operation
- **Replay**: Must be re-emitted on replay; cannot be reconstructed
- **Failure mode**: If error itself cannot be serialized, fallback to `UnknownError`

---

## 2. Message lifecycle events

### `message.updated`
- **Producer**: Message store
- **Payload** (`EventMessageUpdated`): `{ info: Message }`
- **Consumer**: Message list, TUI
- **Ordering**: After message creation; whenever message metadata changes
- **Replay**: Updates message; idempotent

### `message.removed`
- **Producer**: Message store
- **Payload** (`EventMessageRemoved`): `{ sessionID: string; messageID: string }`
- **Consumer**: Message list, TUI
- **Ordering**: After message creation; terminal for the message
- **Replay**: Removes message; idempotent

### `message.part.updated`
- **Producer**: Part accumulator
- **Payload** (`EventMessagePartUpdated`): `{ part: Part; delta?: string }`
- **Consumer**: Message rendering, TUI
- **Ordering**: Sequential per message; parts arrive in creation order
- **Replay**: Accumulates parts; idempotent
- **Failure mode**: If part is malformed, `session.error` may follow

### `message.part.removed`
- **Producer**: Part store
- **Payload** (`EventMessagePartRemoved`): `{ sessionID: string; messageID: string; partID: string }`
- **Consumer**: Message rendering
- **Ordering**: After part addition
- **Replay**: Removes part; idempotent

### `message.part.delta` (v2 only)
- **Producer**: Streaming engine
- **Payload** (`EventMessagePartDelta`): `{ sessionID: string; messageID: string; partID: string; field: string; delta: string }`
- **Consumer**: Streaming renderer
- **Ordering**: Sequential within a part; `field` identifies which sub-field changed
- **Replay**: Deltas must be applied in order; skipping causes corruption
- **Failure mode**: If delta is lost, part state diverges; may require re-sync

---

## 3. Request/stream events (v2 `session.next.*`)

### `session.next.agent.switched`
- **Producer**: Agent router
- **Payload** (`SessionNextAgentSwitched`): `{ timestamp: number; sessionID: string; messageID: string; agent: string }`
- **Consumer**: TUI agent indicator
- **Ordering**: Before first assistant message for the new agent
- **Replay**: Re-records switch; idempotent

### `session.next.model.switched`
- **Producer**: Model selector
- **Payload** (`SessionNextModelSwitched`): `{ timestamp: number; sessionID: string; messageID: string; model: ModelRef }`
- **Consumer**: TUI model indicator
- **Ordering**: Before request to the new model
- **Replay**: Re-records switch; idempotent

### `session.next.moved`
- **Producer**: Session mover
- **Payload** (`SessionNextMoved`): `{ timestamp: number; sessionID: string; location: LocationRef; subdirectory?: string }`
- **Consumer**: TUI, session manager
- **Ordering**: When session directory changes
- **Replay**: Re-records move; idempotent

### `session.next.prompted`
- **Producer**: Prompt delivery system
- **Payload** (`SessionNextPrompted`): `{ timestamp: number; sessionID: string; messageID: string; prompt: Prompt; delivery: "steer" | "queue" }`
- **Consumer**: TUI prompt, session runtime
- **Ordering**: Before prompt is sent to model
- **Replay**: Re-delivers prompt; `delivery` mode must be preserved

### `session.next.prompt.admitted`
- **Producer**: Session admission controller
- **Payload** (`SessionNextPromptAdmitted`): `{ timestamp: number; sessionID: string; messageID: string; prompt: Prompt; delivery: "steer" | "queue" }`
- **Consumer**: Session runtime
- **Ordering**: After `prompted`; when prompt is admitted to the queue
- **Replay**: Re-admits prompt; idempotent

### `session.next.context.updated`
- **Producer**: Context manager
- **Payload** (`SessionNextContextUpdated`): `{ timestamp: number; sessionID: string; messageID: string; text: string }`
- **Consumer**: Context window manager
- **Ordering**: When context window changes
- **Replay**: Updates context; idempotent

### `session.next.synthetic`
- **Producer**: Synthetic message generator
- **Payload** (`SessionNextSynthetic`): `{ timestamp: number; sessionID: string; messageID: string; text: string }`
- **Consumer**: Message store, TUI
- **Ordering**: After compaction or auto-continue
- **Replay**: Re-generates synthetic message; idempotent

### `session.next.shell.started`
- **Producer**: Shell executor
- **Payload** (`SessionNextShellStarted`): `{ timestamp: number; sessionID: string; messageID: string; callID: string; command: string }`
- **Consumer**: Terminal, TUI
- **Ordering**: Before shell command execution
- **Replay**: Re-starts shell; may have side effects

### `session.next.shell.ended`
- **Producer**: Shell executor
- **Payload** (`SessionNextShellEnded`): `{ timestamp: number; sessionID: string; messageID: string; callID: string; output: string }`
- **Consumer**: Terminal, TUI
- **Ordering**: After `shell.started`
- **Replay**: Re-records output; idempotent

### `session.next.step.started`
- **Producer**: Step executor
- **Payload** (`SessionNextStepStarted`): `{ timestamp: number; sessionID: string; assistantMessageID: string; agent: string; model: ModelRef; snapshot?: string }`
- **Consumer**: TUI step indicator, progress bar
- **Ordering**: Before any text/reasoning/tool parts in the step
- **Replay**: Re-starts step; idempotent

### `session.next.step.ended`
- **Producer**: Step executor
- **Payload** (`SessionNextStepEnded`): `{ timestamp: number; sessionID: string; assistantMessageID: string; finish: string; cost: number; tokens: {...}; snapshot?: string; files?: Array<string> }`
- **Consumer**: TUI, cost tracker, step indicator
- **Ordering**: After all parts in the step complete
- **Replay**: Re-records step end; idempotent

### `session.next.step.failed`
- **Producer**: Step executor
- **Payload** (`SessionNextStepFailed`): `{ timestamp: number; sessionID: string; assistantMessageID: string; error: SessionErrorUnknown }`
- **Consumer**: Error handler, TUI
- **Ordering**: When step fails; may trigger retry
- **Replay**: Re-records failure; may trigger retry logic
- **Failure mode**: Error may be unrecoverable; session may enter error state

### `session.next.text.started`
- **Producer**: Text streamer
- **Payload** (`SessionNextTextStarted`): `{ timestamp: number; sessionID: string; assistantMessageID: string; textID: string }`
- **Consumer**: Text renderer
- **Ordering**: Before text deltas for a new text part
- **Replay**: Re-starts text stream; idempotent

### `session.next.text.delta`
- **Producer**: Text streamer
- **Payload** (`SessionNextTextDelta`): `{ timestamp: number; sessionID: string; assistantMessageID: string; textID: string; delta: string }`
- **Consumer**: Text renderer (streaming)
- **Ordering**: Sequential within a text part; MUST be ordered
- **Replay**: Deltas must be applied in order; loss causes truncation
- **Failure mode**: Lost delta = truncated text; may need re-sync

### `session.next.text.ended`
- **Producer**: Text streamer
- **Payload** (`SessionNextTextEnded`): `{ timestamp: number; sessionID: string; assistantMessageID: string; textID: string; text: string }`
- **Consumer**: Text renderer, message store
- **Ordering**: After all deltas for a text part
- **Replay**: Re-records text; idempotent

### `session.next.reasoning.started`
- **Producer**: Reasoning streamer
- **Payload** (`SessionNextReasoningStarted`): `{ timestamp: number; sessionID: string; assistantMessageID: string; reasoningID: string; providerMetadata?: LlmProviderMetadata }`
- **Consumer**: Reasoning display
- **Ordering**: Before reasoning deltas
- **Replay**: Re-starts reasoning; idempotent

### `session.next.reasoning.delta`
- **Producer**: Reasoning streamer
- **Payload** (`SessionNextReasoningDelta`): `{ timestamp: number; sessionID: string; assistantMessageID: string; reasoningID: string; delta: string }`
- **Consumer**: Reasoning display (streaming)
- **Ordering**: Sequential; MUST be ordered
- **Replay**: Deltas must be applied in order
- **Failure mode**: Lost delta = truncated reasoning

### `session.next.reasoning.ended`
- **Producer**: Reasoning streamer
- **Payload** (`SessionNextReasoningEnded`): `{ timestamp: number; sessionID: string; assistantMessageID: string; reasoningID: string; text: string; providerMetadata?: LlmProviderMetadata }`
- **Consumer**: Reasoning display, message store
- **Ordering**: After all reasoning deltas
- **Replay**: Re-records reasoning; idempotent

### `session.next.tool.input.started`
- **Producer**: Tool input streamer
- **Payload** (`SessionNextToolInputStarted`): `{ timestamp: number; sessionID: string; assistantMessageID: string; callID: string; name: string }`
- **Consumer**: Tool display
- **Ordering**: Before tool input deltas
- **Replay**: Re-starts tool input; idempotent

### `session.next.tool.input.delta`
- **Producer**: Tool input streamer
- **Payload** (`SessionNextToolInputDelta`): `{ timestamp: number; sessionID: string; assistantMessageID: string; callID: string; delta: string }`
- **Consumer**: Tool input display (streaming)
- **Ordering**: Sequential
- **Replay**: Deltas must be ordered

### `session.next.tool.input.ended`
- **Producer**: Tool input streamer
- **Payload** (`SessionNextToolInputEnded`): `{ timestamp: number; sessionID: string; assistantMessageID: string; callID: string; text: string }`
- **Consumer**: Tool input display
- **Ordering**: After all input deltas
- **Replay**: Re-records ended input; idempotent

### `session.next.tool.called`
- **Producer**: Tool executor
- **Payload** (`SessionNextToolCalled`): `{ timestamp: number; sessionID: string; assistantMessageID: string; callID: string; tool: string; input: { [key: string]: unknown }; provider: { executed: boolean; metadata?: LlmProviderMetadata } }`
- **Consumer**: Tool display, execution engine
- **Ordering**: Before tool progress/success/failed
- **Replay**: Re-calls tool; may have side effects (idempotency depends on tool)

### `session.next.tool.progress`
- **Producer**: Tool executor
- **Payload** (`SessionNextToolProgress`): `{ timestamp: number; sessionID: string; assistantMessageID: string; callID: string; structured: { [key: string]: unknown }; content: Array<LlmToolContent> }`
- **Consumer**: Tool progress display
- **Ordering**: During tool execution; may be multiple
- **Replay**: Progress events are informational; may be dropped on replay

### `session.next.tool.success`
- **Producer**: Tool executor
- **Payload** (`SessionNextToolSuccess`): `{ timestamp: number; sessionID: string; assistantMessageID: string; callID: string; structured: { [key: string]: unknown }; content: Array<LlmToolContent>; outputPaths?: Array<string>; result?: unknown; provider: { executed: boolean; metadata?: LlmProviderMetadata } }`
- **Consumer**: Tool display, message builder
- **Ordering**: After `tool.called`; before next tool or step end
- **Replay**: Re-records success; idempotent for read-only tools

### `session.next.tool.failed`
- **Producer**: Tool executor
- **Payload** (`SessionNextToolFailed`): `{ timestamp: number; sessionID: string; assistantMessageID: string; callID: string; error: SessionErrorUnknown; result?: unknown; provider: { executed: boolean; metadata?: LlmProviderMetadata } }`
- **Consumer**: Error handler, tool display
- **Ordering**: After `tool.called`; may trigger retry
- **Replay**: Re-records failure; may trigger retry
- **Failure mode**: May cascade to step failure

### `session.next.retried`
- **Producer**: Retry handler
- **Payload** (`SessionNextRetried`): `{ timestamp: number; sessionID: string; attempt: number; error: SessionNextRetryError }`
- **Consumer**: Retry logic, TUI
- **Ordering**: After a failed attempt; before retry
- **Replay**: Re-executes retry; may have different outcome
- **Failure mode**: Max retries exceeded → session error

### `session.next.compaction.started`
- **Producer**: Compaction service
- **Payload** (`SessionNextCompactionStarted`): `{ timestamp: number; sessionID: string; messageID: string; reason: "auto" | "manual" }`
- **Consumer**: TUI, session manager
- **Ordering**: Before compaction
- **Replay**: Re-starts compaction; may have side effects

### `session.next.compaction.delta`
- **Producer**: Compaction service
- **Payload** (`SessionNextCompactionDelta`): `{ timestamp: number; sessionID: string; messageID: string; text: string }`
- **Consumer**: Compaction progress display
- **Ordering**: During compaction; sequential
- **Replay**: Deltas must be ordered

### `session.next.compaction.ended`
- **Producer**: Compaction service
- **Payload** (`SessionNextCompactionEnded`): `{ timestamp: number; sessionID: string; messageID: string; reason: "auto" | "manual"; text: string; recent: string }`
- **Consumer**: TUI, message store
- **Ordering**: After compaction completes
- **Replay**: Re-records end; idempotent

### `session.next.revert.staged`
- **Producer**: Revert service
- **Payload** (`SessionNextRevertStaged`): `{ timestamp: number; sessionID: string; revert: RevertState }`
- **Consumer**: TUI, session manager
- **Ordering**: Before revert is committed
- **Replay**: Re-stages revert; idempotent

### `session.next.revert.cleared`
- **Producer**: Revert service
- **Payload** (`SessionNextRevertCleared`): `{ timestamp: number; sessionID: string }`
- **Consumer**: TUI, session manager
- **Ordering**: After revert is discarded
- **Replay**: Re-clears; idempotent

### `session.next.revert.committed`
- **Producer**: Revert service
- **Payload** (`SessionNextRevertCommitted`): `{ timestamp: number; sessionID: string; messageID: string }`
- **Consumer**: TUI, session manager, file system
- **Ordering**: After revert is applied
- **Replay**: Re-applies revert; may have side effects

---

## 4. Tool events (v1)

### `tool.updated` (via `message.part.updated`)
- Tool state changes are emitted as `message.part.updated` with `part.type === "tool"`
- `ToolPart.state` transitions: `pending → running → completed | error`

---

## 5. Permission events

### `permission.asked` (v1)
- **Producer**: Permission system
- **Payload** (`EventPermissionUpdated`): `{ type: "permission.updated"; properties: Permission }`
- **Consumer**: Permission handler, TUI
- **Ordering**: Before tool/command execution requiring permission
- **Replay**: Re-asks; must be handled by user
- **Failure mode**: If no response, request may time out

### `permission.replied` (v1)
- **Producer**: User input
- **Payload** (`EventPermissionReplied`): `{ sessionID: string; permissionID: string; response: string }`
- **Consumer**: Permission system, session runtime
- **Ordering**: After `permission.asked`
- **Replay**: Re-submits response; idempotent

### `permission.v2.asked`
- **Producer**: Permission system
- **Payload** (`EventPermissionV2Asked`): `{ id: string; sessionID: string; action: string; resources: Array<string>; save?: Array<string>; metadata?: { [key: string]: unknown }; source?: PermissionV2Source }`
- **Consumer**: Permission handler, TUI
- **Ordering**: Before tool/command execution
- **Replay**: Re-asks

### `permission.v2.replied`
- **Producer**: User input
- **Payload** (`EventPermissionV2Replied`): `{ sessionID: string; requestID: string; reply: PermissionV2Reply }`
- **Consumer**: Permission system
- **Ordering**: After `permission.v2.asked`
- **Replay**: Re-submits reply; idempotent
- **Reply values**: `"once" | "always" | "reject"`

---

## 6. MCP events

### `mcp.tools.changed`
- **Producer**: MCP client
- **Payload** (`EventMcpToolsChanged`): `{ server: string }`
- **Consumer**: Tool registry, TUI sidebar
- **Ordering**: After MCP server connection change
- **Replay**: Re-fetches tools; idempotent

### `mcp.browser.open.failed`
- **Producer**: MCP browser handler
- **Payload** (`EventMcpBrowserOpenFailed`): `{ mcpName: string; url: string }`
- **Consumer**: Error handler, TUI
- **Ordering**: When browser-based MCP fails to open
- **Replay**: May trigger fallback to non-browser connection

### `server.connected`
- **Producer**: Server connection manager
- **Payload** (`EventServerConnected`): `{ [key: string]: unknown }`
- **Consumer**: All subscribers
- **Ordering**: After successful server connection
- **Replay**: Re-emits; idempotent

---

## 7. LSP events

### `lsp.updated`
- **Producer**: LSP client
- **Payload** (`EventLspUpdated`): `{ [key: string]: unknown }`
- **Consumer**: TUI sidebar, LSP status
- **Ordering**: When LSP server state changes
- **Replay**: Re-fetches status; idempotent

### `lsp.client.diagnostics`
- **Producer**: LSP client
- **Payload** (`EventLspClientDiagnostics`): `{ serverID: string; path: string }`
- **Consumer**: Editor diagnostics, TUI
- **Ordering**: After diagnostics are received
- **Replay**: Re-fetches diagnostics; idempotent

---

## 8. File/Git events

### `file.edited`
- **Producer**: File watcher / editor
- **Payload** (`EventFileEdited`): `{ file: string }`
- **Consumer**: File index, TUI
- **Ordering**: After file modification
- **Replay**: Re-indexes file; idempotent

### `file.watcher.updated`
- **Producer**: File watcher
- **Payload** (`EventFileWatcherUpdated`): `{ file: string; event: "add" | "change" | "unlink" }`
- **Consumer**: File index, session context
- **Ordering**: When file system changes detected
- **Replay**: Re-watches; idempotent

### `vcs.branch.updated`
- **Producer**: Git service
- **Payload** (`EventVcsBranchUpdated`): `{ branch?: string }`
- **Consumer**: TUI, Git status
- **Ordering**: After branch switch
- **Replay**: Re-reads branch; idempotent

### `session.diff`
- **Producer**: Diff engine
- **Payload** (`EventSessionDiff`): `{ sessionID: string; diff: Array<FileDiff> }`
- **Consumer**: TUI diff view
- **Ordering**: After file changes within session
- **Replay**: Re-computes diff; idempotent

---

## 9. Todo event

### `todo.updated`
- **Producer**: Todo manager
- **Payload** (`EventTodoUpdated`): `{ sessionID: string; todos: Array<Todo> }`
- **Consumer**: TUI sidebar, session manager
- **Ordering**: After todo list changes
- **Replay**: Re-reads todos; idempotent

---

## 10. Command event

### `command.executed`
- **Producer**: Command executor
- **Payload** (`EventCommandExecuted`): `{ name: string; sessionID: string; arguments: string; messageID: string }`
- **Consumer**: Command history, TUI
- **Ordering**: After command execution
- **Replay**: Re-executes; may have side effects

---

## 11. TUI events

### `tui.prompt.append`
- **Producer**: TUI input
- **Payload** (`EventTuiPromptAppend`): `{ text: string }`
- **Consumer**: TUI prompt input
- **Ordering**: User types in prompt
- **Replay**: Re-appends text; idempotent

### `tui.command.execute`
- **Producer**: TUI keybinding handler
- **Payload** (`EventTuiCommandExecute`): `{ command: string }`
- **Consumer**: Command executor
- **Ordering**: User triggers command
- **Replay**: Re-executes command; may have side effects

### `tui.toast.show`
- **Producer**: TUI notification system
- **Payload** (`EventTuiToastShow`): `{ title?: string; message: string; variant: "info" | "success" | "warning" | "error"; duration?: number }`
- **Consumer**: TUI toast renderer
- **Ordering**: When notification should appear
- **Replay**: Re-shows toast; idempotent

### `tui.session.select`
- **Producer**: TUI session selector
- **Payload** (`EventTuiSessionSelect`): `{ sessionID: string }`
- **Consumer**: TUI router, session manager
- **Ordering**: When user selects a session
- **Replay**: Re-selects session; idempotent

---

## 12. Question events

### `question.asked`
- **Producer**: Question system
- **Payload** (`EventQuestionAsked`): `{ id: string; sessionID: string; questions: Array<QuestionInfo>; tool?: QuestionTool }`
- **Consumer**: Question UI, session runtime
- **Ordering**: Before user response
- **Replay**: Re-asks; must be handled by user

### `question.replied`
- **Producer**: User input
- **Payload** (`EventQuestionReplied`): `{ sessionID: string; requestID: string; answers: Array<QuestionAnswer> }`
- **Consumer**: Question system, session runtime
- **Ordering**: After `question.asked`
- **Replay**: Re-submits answers; idempotent

### `question.rejected`
- **Producer**: User input
- **Payload** (`EventQuestionRejected`): `{ sessionID: string; requestID: string }`
- **Consumer**: Question system
- **Ordering**: After `question.asked`; user declines
- **Replay**: Re-rejects; idempotent

### `question.v2.asked` / `question.v2.replied` / `question.v2.rejected`
- Same semantics with v2-specific payloads including `QuestionV2Info`, `QuestionV2Answer`, `QuestionV2Tool`

---

## 13. Workspace events

### `workspace.ready`
- **Producer**: Workspace manager
- **Payload** (`EventWorkspaceReady`): `{ name: string }`
- **Consumer**: TUI, session manager
- **Ordering**: After workspace initialization
- **Replay**: Re-marks ready; idempotent

### `workspace.failed`
- **Producer**: Workspace manager
- **Payload** (`EventWorkspaceFailed`): `{ message: string }`
- **Consumer**: Error handler, TUI
- **Ordering**: When workspace initialization fails
- **Replay**: Re-emits; may trigger recovery

### `workspace.status`
- **Producer**: Workspace manager
- **Payload** (`EventWorkspaceStatus`): `{ workspaceID: string; status: "connected" | "connecting" | "disconnected" | "error" }`
- **Consumer**: TUI, workspace manager
- **Ordering**: State transitions
- **Replay**: Latest status wins

### `worktree.ready` / `worktree.failed`
- Same pattern as workspace events for worktree-specific operations

---

## 14. Pty events

### `pty.created` / `pty.updated` / `pty.exited` / `pty.deleted`
- Terminal session lifecycle events
- Full payloads in `types.gen.d.ts` (lines 571-595)

---

## 15. Sync events (v2 durable)

All v2 events have corresponding `SyncEvent*` types with:
- `type: "sync"`
- `id: string`
- `syncEvent: { type: string; id: string; seq: number; aggregateID: string; data: {...} }`

**Ordering guarantee**: `seq` is monotonically increasing per aggregate; events must be applied in `seq` order within an aggregate.

**Replay behavior**: Sync events are the source of truth for replay; they contain complete state snapshots and deltas.

---

## 16. Error and installation events

### `session.error` — already documented above

### `installation.updated` / `installation.update-available`
- Application installation state changes

### `plugin.added`
- `{ id: string }` — new plugin registered

### `project.updated`
- `{ id: string; worktree: string; vcs?: ProjectVcs; name?: string; icon?: ProjectIcon; commands?: ProjectCommands; time: ProjectTime; sandboxes: Array<string> }`

### `global.disposed`
- Server instance disposal

---

## Event ordering summary

```
session.created → session.status(busy) → [request events] → session.status(idle)
  → session.next.* → [compaction] → session.compacted
  → [permission] → [tool] → [message.part.*] → session.error | session.status(idle)
```

**Global ordering**: Within a session, events are ordered by timestamp. Across sessions, ordering is not guaranteed.

**Sync ordering**: Within an aggregate, `seq` determines order. Cross-aggregate ordering is causal.

---

## Event replay semantics

| Replay type | Behavior | Risk |
|-------------|----------|------|
| Idempotent replay | Safe to replay; produces same state | Low |
| State-replacement replay | Replaces current state; may lose interim state | Medium |
| Side-effect replay | May trigger actions (shell, tool calls, file writes) | High |
| Delta-replay | Must be ordered; skipping causes corruption | Critical |

---

## Event failure modes

| Failure | Consequence | Recovery |
|---------|-------------|----------|
| Lost delta event | Truncated text/reasoning | Re-sync from snapshot |
| Out-of-order event | State corruption | Replay from last known good state |
| Malformed event | Parsing error; may crash handler | Skip event; emit `session.error` |
| Duplicate event | State inconsistency | Deduplicate by `id` |
| Stale event | Overwrites newer state | Discard if `seq` < current |
