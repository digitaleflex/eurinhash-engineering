# Failure Semantics — Error Handling & Recovery

> Source: `@opencode-ai/sdk/dist/gen/types.gen.d.ts`, `@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`
> Rule: Every error type has a specific meaning. Never conflate different failure modes.

---

## Error type taxonomy

### Provider errors (v1 `types.gen.d.ts`)

| Error type | `name` field | Semantics | Recoverable? |
|------------|-------------|-----------|-------------|
| `ProviderAuthError` | `"ProviderAuthError"` | Authentication failure | Yes — re-authenticate |
| `UnknownError` | `"UnknownError"` | Unclassified error | Depends on message |
| `MessageOutputLengthError` | `"MessageOutputLengthError"` | Output exceeded limit | Yes — retry with shorter prompt |
| `MessageAbortedError` | `"MessageAbortedError"` | Request was aborted | Yes — resubmit |
| `ApiError` | `"APIError"` | API-level error | Depends on `isRetryable` |

### v2 extended error types (`v2/gen/types.gen.d.ts`)

| Error type | `name` field | Semantics | Recoverable? |
|------------|-------------|-----------|-------------|
| `StructuredOutputError` | `"StructuredOutputError"` | Failed structured output | Yes — retry with different format |
| `ContextOverflowError` | `"ContextOverflowError"` | Context window exceeded | Yes — compact or reduce context |
| `ContentFilterError` | `"ContentFilterError"` | Content filtered by provider | Maybe — modify input |
| `SessionBusyError` | `SessionBusyError` | Session already busy | Yes — wait or retry |
| `MoveSessionError` | `"MoveSessionError"` | Session move failed | Maybe |

### Bad request / not found errors

| Error type | `name` field | Semantics |
|------------|-------------|-----------|
| `BadRequestError` | `"BadRequest"` | Invalid request parameters |
| `NotFoundError` | `"NotFoundError"` | Resource not found |

### v2 effect errors (`v2/gen/types.gen.d.ts`)

| Error type | `_tag` | Semantics |
|------------|--------|-----------|
| `EffectHttpApiErrorBadRequest` | `"BadRequest"` | HTTP 400 |
| `EffectHttpApiErrorInternalServerError` | `"InternalServerError"` | HTTP 500 |
| `EffectHttpApiErrorForbidden` | `"Forbidden"` | HTTP 403 |
| `InvalidRequestError` | `"InvalidRequestError"` | Structured validation error |
| `UnauthorizedError` | `"UnauthorizedError"` | Missing/invalid auth |
| `ForbiddenError` | `"ForbiddenError"` | Permission denied |
| `ConflictError` | `"ConflictError"` | Resource conflict |
| `ServiceUnavailableError` | `"ServiceUnavailableError"` | Service down |
| `ProviderAuthError1` | `"BadRequest" \| ...` | Provider auth errors |
| `McpServerNotFoundError` | `"McpServerNotFoundError"` | MCP server not found |
| `McpUnsupportedOAuthError` | — | Unsupported OAuth |
| `WorktreeError` | `"WorktreeNotGitError" \| ...` | Worktree operation failed |
| `ProjectNotFoundError` | `"ProjectNotFoundError"` | Project not found |
| `PtyNotFoundError` | `"PtyNotFoundError"` | Pty not found |
| `PtyForbiddenError` | `"PtyForbiddenError"` | Pty access denied |
| `QuestionNotFoundError` | `"QuestionNotFoundError"` | Question not found |
| `PermissionNotFoundError` | `"PermissionNotFoundError"` | Permission not found |
| `SessionNotFoundError` | `"SessionNotFoundError"` | Session not found |
| `MessageNotFoundError` | `"MessageNotFoundError"` | Message not found |
| `ProviderNotFoundError` | `"ProviderNotFoundError"` | Provider not found |
| `InvalidCursorError` | `"InvalidCursorError"` | Pagination cursor invalid |

---

## Session error semantics

### `EventSessionError` payload analysis

```typescript
type EventSessionError = {
    type: "session.error";
    properties: {
        sessionID?: string;
        error?: ProviderAuthError | UnknownError | MessageOutputLengthError | MessageAbortedError | ApiError;
    };
};
```

**v2 extended**: error union includes `StructuredOutputError`, `ContextOverflowError`, `ContentFilterError`

### Error → State mapping

| Error type | Session state after error | Recovery action |
|------------|--------------------------|-----------------|
| `ProviderAuthError` | idle (with error flag) | Re-authenticate provider |
| `MessageAbortedError` | idle | Resubmit message |
| `MessageOutputLengthError` | idle | Shorten prompt |
| `ApiError` (isRetryable) | retry | Retry with backoff |
| `ApiError` (not retryable) | idle (with error) | Check API status |
| `StructuredOutputError` | idle | Adjust output format |
| `ContextOverflowError` | idle | Compact context |
| `ContentFilterError` | idle | Modify input |
| `SessionBusyError` | busy | Wait or force |
| `MoveSessionError` | idle | Check directory permissions |

### `session.next.step.failed` semantics

```typescript
type SessionNextStepFailed = {
    type: "session.next.step.failed";
    data: {
        timestamp: number;
        sessionID: string;
        assistantMessageID: string;
        error: SessionErrorUnknown;  // { type: "unknown"; message: string }
    };
};
```

**Recovery**: Step failure may trigger `session.next.retried` if retryable, or `session.error` if not.

---

## Tool error semantics

### `ToolStateError` structure

```typescript
type ToolStateError = {
    status: "error";
    input: { [key: string]: unknown };
    error: string;           // Human-readable error message
    metadata?: { [key: string]: unknown };
    time: { start: number; end: number };
};
```

### v2 `SessionMessageToolStateError`

```typescript
type SessionMessageToolStateError = {
    status: "error";
    input: { [key: string]: unknown };
    content: Array<LlmToolContent>;
    structured: { [key: string]: unknown };
    error: SessionErrorUnknown;  // { type: "unknown"; message: string }
    result?: unknown;
};
```

### Tool error → Recovery mapping

| Tool error context | Recovery |
|--------------------|----------|
| Tool not found | Check tool definition; may be MCP tool |
| Tool execution error | Retry with same input (idempotent tools only) |
| Tool timeout | Increase timeout or cancel |
| Tool permission denied | Permission state must be resolved |
| Tool input validation error | Fix input and retry |
| MCP tool connection error | Check MCP server state |

---

## Permission failure semantics

### Permission denied flow

```
permission.asked → permission.replied (deny) → tool blocked
  → session continues without tool execution
```

### v2 permission failure

| Reply | Meaning | Consequence |
|-------|---------|-------------|
| `"once"` | Allow one time | Tool executes once; rule not saved |
| `"always"` | Allow always | Tool executes; rule saved in `always` |
| `"reject"` | Deny | Tool blocked; `permission.rejected` emitted |

### `PermissionV2Ruleset` evaluation

```typescript
type PermissionV2Rule = {
    action: string;
    resource: string;
    effect: PermissionV2Effect;  // "allow" | "deny" | "ask"
};
```

**Evaluation order**: Rules are evaluated in order. First match wins. `deny` takes precedence over `ask`.

---

## MCP failure semantics

### `McpStatusFailed`

```typescript
type McpStatusFailed = {
    status: "failed";
    error: string;  // Specific error message
};
```

**Recovery**: Reconnect or restart MCP server. Error message provides diagnostics.

### `McpStatusNeedsAuth`

```typescript
type McpStatusNeedsAuth = {
    status: "needs_auth";
};
```

**Recovery**: Trigger OAuth flow or provide API key. `McpOAuthConfig` provides the auth configuration.

### `McpStatusNeedsClientRegistration`

```typescript
type McpStatusNeedsClientRegistration = {
    status: "needs_client_registration";
    error: string;
};
```

**Recovery**: Complete dynamic client registration (RFC 7591). Error provides registration details.

### MCP browser failure

```typescript
type EventMcpBrowserOpenFailed = {
    type: "mcp.browser.open.failed";
    properties: { mcpName: string; url: string };
};
```

**Recovery**: Fall back to non-browser connection or prompt user to open manually.

---

## LSP failure semantics

### `LspStatus` error

```typescript
type LspStatus = {
    id: string;
    name: string;
    root: string;
    status: "connected" | "error";
};
```

**Error state**: `status: "error"` means the LSP server encountered an error. The `id` and `root` identify which server failed.

**Recovery**: Restart LSP server. Check `lsp` configuration for command and extensions.

### `lsp.client.diagnostics` as error indicator

```typescript
type EventLspClientDiagnostics = {
    type: "lsp.client.diagnostics";
    properties: { serverID: string; path: string };
};
```

**Semantics**: Diagnostics are emitted even when status is "connected" — they indicate code-level issues, not server-level errors.

---

## Pty failure semantics

### Pty exit states

```typescript
type Pty = {
    id: string;
    title: string;
    command: string;
    args: Array<string>;
    cwd: string;
    status: "running" | "exited";
    pid: number;
    exitCode?: number;  // Only when status is "exited"
};
```

| `exitCode` | Meaning |
|------------|---------|
| `0` | Successful exit |
| Non-zero | Process error; code indicates error type |
| `undefined` | Status is "running", not exited |

### Pty errors

| Event | Semantics | Recovery |
|-------|-----------|----------|
| `pty.exited` | Process ended | Check `exitCode`; restart if needed |
| `pty.deleted` | Terminal removed | Cannot recover; create new pty |
| `PtyForbiddenError` | Access denied | Check permissions |
| `PtyNotFoundError` | Pty not found | Terminal may have been removed |

---

## Configuration error semantics

### `BadRequestError` details

```typescript
type BadRequestError = {
    name: "BadRequest";
    data: {
        message: string;
        kind?: "Params" | "Headers" | "Query" | "Body" | "Payload";
    };
};
```

**`kind` field**: Identifies which part of the request is invalid. This enables targeted fixes.

### Provider configuration errors

| Config field | Error | Semantics |
|-------------|-------|-----------|
| `provider.api` | BadRequest | Invalid API URL |
| `provider.env` | ProviderAuthError | Missing/invalid credentials |
| `provider.models` | BadRequest | Invalid model configuration |
| `provider.timeout` | BadRequest | Invalid timeout value |
| `mcp.command` | BadRequest | Invalid MCP command |
| `lsp.command` | BadRequest | Invalid LSP command |

---

## Retry semantics

### `RetryPart` structure

```typescript
type RetryPart = {
    id: string;
    sessionID: string;
    messageID: string;
    type: "retry";
    attempt: number;
    error: ApiError;
    time: { created: number };
};
```

### `SessionNextRetryError` (v2)

```typescript
type SessionNextRetryError = {
    message: string;
    statusCode?: number;
    isRetryable: boolean;
    responseHeaders?: { [key: string]: string };
    responseBody?: string;
    metadata?: { [key: string]: string };
};
```

### Retry policy

| Condition | Action |
|-----------|--------|
| `isRetryable: true` | Retry with exponential backoff |
| `isRetryable: false` | Do not retry; emit error |
| `statusCode` present | HTTP status for rate limiting decisions |
| `attempt` exceeds max | Stop retrying; emit `session.error` |
| `SessionNextRetried` emitted | Retry is in progress; update UI |

### v2 `chatMaxRetries` config

```typescript
experimental?: {
    chatMaxRetries?: number;  // Number of retries for chat completions
};
```

---

## Compaction failure semantics

### Compaction error flow

```
compaction.started → compaction.delta* → compaction.ended
                              ↓
                        error → session.error
```

### Compaction failure recovery

| Failure mode | Recovery |
|-------------|----------|
| Compaction timeout | Increase timeout; retry |
| Compaction error | Check error message; may need manual intervention |
| Compaction partial | May need to rollback using `revert` state |
| Auto-compaction fails | Disable auto-compaction; use manual |

### `RevertState` as recovery mechanism

```typescript
type RevertState = {
    messageID: string;
    partID?: string;
    snapshot?: string;
    diff?: string;
    files?: Array<FileDiff>;
};
```

**Recovery flow**: If compaction fails, use `RevertState` to restore the session to its pre-compaction state.

---

## Cancellation semantics

### `MessageAbortedError`

```typescript
type MessageAbortedError = {
    name: "MessageAbortedError";
    data: { message: string };
};
```

**Semantics**: The user or system aborted a message request. This is distinct from errors — it's an intentional cancellation.

### Cancellation flow

```
User presses interrupt → session.interrupt command → MessageAbortedError
  → session.status → idle
  → All in-progress operations must stop
```

### Cancellation guarantees

1. **Tool cancellation**: Running tools must be interrupted; `ToolStateError` emitted
2. **Stream cancellation**: In-progress streams must terminate; no more deltas
3. **Session cancellation**: Session returns to `idle` state
4. **No partial state**: Cancellation must leave no half-finished state

---

## Error propagation model

```
Provider error → ApiError → Message error → Session error → TUI error display
MCP error → McpStatusFailed → Tool error → Message error → Session error
LSP error → LspStatus.error → Diagnostics → UI indicator
Permission error → Permission rejected → Tool blocked → No message error
Config error → BadRequest → Config validation → TUI error display
```

### Propagation rules

1. **Provider errors** propagate to message level, then session level
2. **MCP errors** are contained within the tool; may not reach session level
3. **Permission errors** block execution; no error is emitted to the message
4. **Config errors** prevent initialization; no session is created
5. **LSP errors** are informational; do not block session execution
6. **Pty errors** are contained within the terminal; do not affect session

---

## Failure detection heuristics

| Failure type | Detection signal | Time to detect |
|-------------|-----------------|----------------|
| Provider auth failure | `401`/`403` status | Immediate |
| API rate limit | `429` status + `isRetryable` | Immediate |
| Context overflow | `ContextOverflowError` | After response |
| Tool timeout | `ToolStateError` with timeout message | Configurable |
| MCP disconnection | `McpStatusFailed` | Connection timeout |
| LSP crash | `status: "error"` | Process exit |
| Pty exit | `pty.exited` event | Process exit |
| Session busy conflict | `SessionBusyError` | Immediate |

---

## Recovery strategies

### Automatic recovery

| Failure | Strategy | Condition |
|---------|----------|-----------|
| `ApiError` retryable | Retry with backoff | `isRetryable: true` |
| `MessageAbortedError` | Resubmit | User requests |
| MCP disconnection | Reconnect | `McpStatusFailed` with transient error |
| Pty crash | Restart | `pty.exited` with non-zero code |
| Context overflow | Compact | `ContextOverflowError` |

### Manual recovery

| Failure | Strategy | Condition |
|---------|----------|-----------|
| `ProviderAuthError` | Re-authenticate | Credential expired |
| `Permission rejected` | Modify request | User denied |
| MCP config error | Fix config | `McpStatusNeedsClientRegistration` |
| LSP config error | Fix config | `status: "error"` |
| `BadRequestError` | Fix request | `kind` indicates invalid field |
| Compaction failure | Manual rollback | `RevertState` available |

### Unrecoverable failures

| Failure | Reason | Action |
|---------|--------|--------|
| `ForbiddenError` | Permanent denial | Check permissions |
| `NotFoundError` | Resource deleted | Cannot recover |
| `ConflictError` | State mismatch | Resolve conflict |
| `SessionNotFoundError` | Session deleted | Cannot recover |
| `InvalidCursorError` | Pagination broken | Restart from beginning |

---

## Error type completeness check

Every error type in the system must be handled:

**v1 errors**: `ProviderAuthError`, `UnknownError`, `MessageOutputLengthError`, `MessageAbortedError`, `ApiError`, `BadRequestError`, `NotFoundError`

**v2 additional errors**: `StructuredOutputError`, `ContextOverflowError`, `ContentFilterError`, `SessionBusyError`, `MoveSessionError`, `InvalidRequestError`, `UnauthorizedError`, `ForbiddenError`, `ConflictError`, `ServiceUnavailableError`, `ProviderAuthError1`, `McpServerNotFoundError`, `McpUnsupportedOAuthError`, `WorktreeError`, `ProjectNotFoundError`, `PtyNotFoundError`, `PtyForbiddenError`, `QuestionNotFoundError`, `PermissionNotFoundError`, `SessionNotFoundError`, `MessageNotFoundError`, `ProviderNotFoundError`, `InvalidCursorError`, `SessionErrorUnknown`, `SessionNextRetryError`

**Never ignore an error type.** Every error type must have a documented handling strategy.
