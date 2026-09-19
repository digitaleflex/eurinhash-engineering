# Observability Semantics — Value Semantics

> **Source**: `docs/state/unknown-data.md`, `docs/design/status-language.md`, `docs/design/components.md`
> **Rule**: `UNKNOWN != ZERO`. Every value has precise semantics. Never fabricate.

---

## 1. Core Semantic Rules

### 1.1 `UNKNOWN != ZERO`

| Scenario | Correct (UNKNOWN) | Wrong (ZERO) | Why |
|----------|-------------------|--------------|-----|
| No cost computed | `N/A` | `$0.00` | `0` means free; `N/A` means not calculated |
| No model selected | `undefined` | `{ id: "", providerID: "" }` | Empty model implies a model exists |
| Not compacting | `undefined` | `0` | `0` means started at epoch |
| Session has no parent | `undefined` | `""` | `""` could be a valid ID |
| No tool output yet | `undefined` | `""` | `""` is a valid output |
| Tokens not counted | `undefined` | `0` | Only if genuinely zero — but `0` is valid for tokens already counted |

### 1.2 When `0` IS Correct

| Field | `0` means |
|-------|-----------|
| `tokens.input` | Zero input tokens (message with no input) |
| `tokens.output` | Zero output tokens |
| `tokens.reasoning` | Zero reasoning tokens |
| `additions` / `deletions` | No lines changed |
| `cost` | Zero cost (only if genuinely free) |
| `attempt` (retry) | Zero retries attempted |

### 1.3 When `""` IS Correct

| Field | `""` means |
|-------|------------|
| `FileDiff.before` | Empty file (new file) |
| `FileDiff.after` | Empty file (deleted file) |
| `error` in `ToolStateError` | Empty error string (edge case) |
| `text` in `TextPart` | Empty text |

---

## 2. Domain-Specific Semantics

### 2.1 SESSION Semantics

| Value | Meaning | UNKNOWN |
|-------|---------|---------|
| `status: { type: "idle" }` | No active request | Never default for unknown |
| `status: { type: "busy" }` | Request in progress | Never default |
| `status: { type: "retry" }` | Retry scheduled | Never default |
| `parentID: undefined` | Root session | NOT `""` |
| `cost: undefined` | Not computed | NOT `$0.00` |
| `tokens: undefined` | Not computed | NOT `{ input: 0, ... }` |
| `time.compacting: undefined` | Not compacting | NOT `0` |

### 2.2 MODEL Semantics

| Value | Meaning | UNKNOWN |
|-------|---------|---------|
| `status: "active"` | Ready for use | — |
| `status: "beta"` | Testing phase | — |
| `status: "alpha"` | Experimental | — |
| `status: "deprecated"` | No longer recommended | — |
| `limit.context: 0` | Unbounded or unknown | NOT a valid limit |
| `variant: undefined` | Default variant | NOT "no variant" |
| `capabilities.x: false` | Feature not supported | NOT unknown — `false` is definitive |

### 2.3 CONTEXT Semantics

| Value | Meaning | UNKNOWN |
|-------|---------|---------|
| `percentage: undefined` | Limit unknown or 0 | NOT `0%` |
| `status: "NORMAL"` | Below 50% usage | — |
| `status: "ATTENTION"` | ≥50% usage | — |
| `status: "HIGH"` | ≥75% usage | — |
| `status: "CRITICAL"` | ≥90% usage | — |
| `status: "LIMIT"` | ≥100% usage | — |

### 2.4 TOKENS Semantics

| Value | Meaning | UNKNOWN |
|-------|---------|---------|
| `tokens.input: 0` | No input tokens | — |
| `tokens.output: 0` | No output tokens | — |
| `tokens.reasoning: 0` | No reasoning | — |
| `tokens.cache.read: 0` | No cache reads | — |
| `tokens.cache.write: 0` | No cache writes | — |
| `tokens: undefined` | Not computed yet | NOT `{ input: 0, output: 0, ... }` |

**Critical distinction**: `0` means the count is genuinely zero. `undefined` means the count hasn't been computed yet.

### 2.5 COST Semantics

| Value | Meaning | UNKNOWN |
|-------|---------|---------|
| `cost: 0.00` | Genuinely free tier | — |
| `cost: undefined` | Not computed | NOT `$0.00` as placeholder |
| `currency: "USD"` | Cost in USD | — |
| `currency: "N/A"` | Provider doesn't expose cost | NOT `""` |

### 2.6 ACTIVITY Semantics

| Value | Meaning | UNKNOWN |
|-------|---------|---------|
| `phase: "idle"` | No active request | — |
| `phase: "busy"` | Request in progress | — |
| `phase: "retry"` | Retry scheduled | — |
| `currentTool: undefined` | No tool executing | NOT `"none"` |
| `currentStep: undefined` | No step info | NOT `""` |

### 2.7 TOOLS Semantics

| Value | Meaning | UNKNOWN |
|-------|---------|---------|
| `ToolState.status: "pending"` | Queued, input known | — |
| `ToolState.status: "running"` | Executing | — |
| `ToolState.status: "completed"` | Finished successfully | — |
| `ToolState.status: "error"` | Failed | — |
| `McpStatus.status: "connected"` | MCP server reachable | — |
| `McpStatus.status: "disabled"` | Explicitly disabled | — |
| `McpStatus.status: "failed"` | Connection failed | — |
| `McpStatus.status: "needs_auth"` | Auth required | — |
| `McpStatus.status: "needs_client_registration"` | Registration needed | — |
| `LspStatus.status: "connected"` | LSP operational | — |
| `LspStatus.status: "error"` | LSP error | — |
| `error: undefined` | No error | NOT `""` |

### 2.8 PERFORMANCE Semantics

| Value | Meaning | UNKNOWN |
|-------|---------|---------|
| `latency: undefined` | Timestamps unavailable | NOT `0` |
| `duration: undefined` | Timestamps unavailable | NOT `0` |
| `throughput: undefined` | Missing tokens or duration | NOT `0` |
| `queueDepth: undefined` | Queue state not tracked | NOT `0` |

### 2.9 SECURITY Semantics

| Value | Meaning | UNKNOWN |
|-------|---------|---------|
| `permission: "ask"` | Requires user approval | — |
| `permission: "allow"` | Approved | — |
| `permission: "deny"` | Rejected | — |
| `Provider.key: undefined` | No API key | NOT `""` |
| `McpStatus.status: "needs_auth"` | Auth required | — |
| `riskLevel: undefined` | Risk not supported by runtime | NOT `"low"` |

---

## 3. Status Vocabulary

### 3.1 All Statuses

| Status | Glyph | Color | Semantic |
|--------|-------|-------|----------|
| ACTIVE | `●` | `accent` | Operational, ready |
| IDLE | `○` | `textMuted` | Inactive, waiting |
| THINKING | `◐` | `secondary` + `thinkingOpacity` | Reasoning internally |
| RUNNING | `▶` | `primary` | Executing action |
| WAITING | `⌛` | `warning` | Waiting for external input |
| SUCCESS | `✓` | `success` | Completed successfully |
| WARNING | `⚠` | `warning` | Sub-optimal but functional |
| ERROR | `✗` | `error` | Failed, critical |
| CANCELLED | `⊘` | `textMuted` | Interrupted before completion |
| UNKNOWN | `?` | `textMuted` | Indeterminate, data unavailable |
| DISCONNECTED | `⊘` | `error` | Connection lost |

### 3.2 Status Rules

1. **Double signal**: Every status must be readable via text + glyph, not color alone
2. **`UNKNOWN` ≠ `ZERO`**: Never display `0` or empty for UNKNOWN
3. **`UNKNOWN` ≠ `IDLE`**: UNKNOWN means data unavailable; IDLE means no task
4. **`CANCELLED` ≠ `IDLE`**: Cancelled = interrupted; Idle = not started
5. **`thinkingOpacity`** applies ONLY to THINKING, never to other statuses

### 3.3 Forbidden Status Terms

| Forbidden | Use Instead |
|-----------|-------------|
| `NONE` | `IDLE` |
| `NULL` | `UNKNOWN` |
| `DONE` | `SUCCESS` |
| `BUSY` | `RUNNING` or `THINKING` |
| `OFF` | `DISCONNECTED` or `IDLE` |
| `N/A` as status | `N/A` is for telemetry values only |

---

## 4. Telemetry Format Semantics

| Format | Condition | Example |
|--------|-----------|---------|
| `38%` | Simple percentage | Context usage |
| `100K / 262K · 38%` | Usage with known limit | `used / limit · percentage` |
| `100K · 38%` | Limit unknown | `used · percentage` |
| `$0.00` | Cost known and zero | Confirmed free tier |
| `N/A` | Cost unknown | Provider doesn't expose cost |
| `2.3s` | Duration | Elapsed time |
| `22s`, `01m 43s` | Duration | Formatted time |
| `1.2M`, `100.4K` | Token count | Formatted tokens |

### 4.1 Format Rules

1. `UNKNOWN` stays `UNKNOWN` at every format level
2. `N/A` stays `N/A` — never transformed
3. `0` is displayed as `0` only when genuinely zero
4. All numbers in `mono` typography
5. Percentages are always `mono`
6. Costs are always `$X.XX` format or `N/A`

---

## 5. Presentation Invariants

| Presentation Field | When Unknown | NEVER |
|--------------------|--------------|-------|
| `session.get(id)` returns `undefined` | Show "No session" | Show empty session object |
| `session.status(id)` returns `undefined` | Show "Unknown" | Show `{ type: "idle" }` |
| `part(messageID)` returns `[]` | Show empty state | Show placeholder parts |
| `lsp()` returns `[]` | Show "No LSP" | Show fake LSP entry |
| `mcp()` returns `[]` | Show "No MCP" | Show fake MCP entry |
| `provider` returns `[]` | Show "No providers" | Show mock provider |
| `vcs` is `undefined` | Show "No VCS" | Show `{ branch: "" }` |

---

## 6. UNKNOWN Handling Protocol

### 6.1 Detection

1. Field is optional (`?` in TypeScript) → may be `undefined`
2. Field is part of a discriminated union → must match a variant
3. Field is derived → calculation may yield `undefined`

### 6.2 Presentation

| Context | Display |
|---------|---------|
| Cost not computed | `N/A` |
| Tokens not computed | `N/A` |
| Duration not available | `N/A` |
| Status indeterminate | `[? UNKNOWN]` |
| Provider not exposing data | `N/A` |
| Risk not supported | `N/A` |

### 6.3 NEVER Do This

- ❌ Display `$0.00` when cost is `undefined`
- ❌ Display `0%` when context percentage is `undefined`
- ❌ Display `0 tokens` when token count is `undefined`
- ❌ Display `"none"` when tool is `undefined`
- ❌ Display `{ id: "", providerID: "" }` when model is `undefined`
- ❌ Display `{ type: "idle" }` when status is `undefined`

---

## 7. Serialization Semantics

When serializing observability data:

1. `undefined` fields are omitted from JSON objects
2. `0`, `""`, `false` are always serialized
3. **Never replace `undefined` with `0` or `""` before serialization** — corrupts type semantics
4. When deserializing, missing fields should be `undefined`, not defaults

---

## 8. References

- `docs/state/unknown-data.md` — Full `UNKNOWN != ZERO` field-by-field treatment
- `docs/design/status-language.md` — Complete status vocabulary
- `docs/design/components.md` — Component display rules
- `docs/state/domain-state.md` — All domain type definitions
- `docs/architecture/state-contracts.md` — State contract rules
