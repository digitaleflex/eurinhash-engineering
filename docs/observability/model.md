# Observability Model — EurinHash OpenCode

> **Source**: `@opencode-ai/sdk/dist/gen/types.gen.d.ts`, `@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`, `@opencode-ai/plugin/dist/index.d.ts`, `@opencode-ai/plugin/dist/tui.d.ts`
> **Rule**: `UNKNOWN != ZERO` — never fabricate a value; if a field is absent, it is `undefined`, not `0`, `""`, or `false`.
> **Design**: 5 modes — MINIMAL, STANDARD, DEVELOPER, OBSERVER, DEBUG (see `docs/design/density.md`).

---

## 1. Canonical Domains

The observability model captures exactly 9 domains. Each domain corresponds to data actually available from the runtime. No extrapolation, no estimation.

### 1.1 Domain Map

| Domain | Abbreviation | Source Types | Primary Events |
|--------|-------------|-------------|----------------|
| SESSION | `SES` | `Session`, `SessionStatus` | `session.status`, `session.created`, `session.updated`, `session.idle`, `session.compacted` |
| MODEL | `MDL` | `Model`, `Provider`, `ModelRef` | `session.next.model.switched`, `session.next.agent.switched` |
| CONTEXT | `CTX` | `Model.limit`, `Session.tokens` | `session.next.context.updated` |
| TOKENS | `TOK` | `StepFinishPart.tokens`, `AssistantMessage.tokens` | `session.next.step.ended`, `message.part.updated` |
| COST | `CST` | `StepFinishPart.cost`, `Session.cost` | `session.next.step.ended` |
| ACTIVITY | `ACT` | `SessionStatus`, `Part` union | `session.status`, `message.part.updated`, `session.next.*` |
| TOOLS | `TOO` | `ToolPart`, `ToolState`, `McpStatus` | `session.next.tool.called`, `.progress`, `.success`, `.failed` |
| PERFORMANCE | `PRF` | `time.start`, `time.end`, derived | Computed from timestamps in events |
| SECURITY | `SEC` | `PermissionRequest`, `McpStatus`, `Provider.key` | `permission.asked`, `permission.replied`, `mcp.*` |

### 1.2 Domain Relationships

```
SESSION ──contains──► MODEL
  │                    │
  ├──has──► CONTEXT   └──provides──► TOKENS
  │                    │
  ├──has──► TOKENS    └──provides──► COST
  │                    │
  ├──has──► COST      └──provides──► ACTIVITY
  │                    │
  ├──has──► ACTIVITY  └──contains──► TOOLS
  │                    │
  ├──has──► TOOLS     └──derives──► PERFORMANCE
  │                    │
  ├──has──► PERFORMANCE    └──derives──► SECURITY
  │                    │
  └──has──► SECURITY
```

---

## 2. Domain Definitions

### 2.1 SESSION

**Origin**: `Session` (v1 `types.gen.d.ts:465`, v2 `v2/gen/types.gen.d.ts:64`)

| Field | Type | Semantics | UNKNOWN |
|-------|------|-----------|---------|
| `id` | `string` | Unique session identifier | Never absent for live sessions |
| `projectID` | `string` | Project owner | Never absent |
| `directory` | `string` | Workspace path | Never absent |
| `parentID` | `string \| undefined` | Parent session for child sessions | `undefined` = root — NOT `""` |
| `title` | `string` | Display title | Never absent |
| `version` | `string` | Schema version | Never absent |
| `time.created` | `number` | Unix ms timestamp | `undefined` if not set — NOT `0` |
| `time.updated` | `number` | Unix ms timestamp | `undefined` if not updated |
| `time.compacting` | `number \| undefined` | When compaction started | `undefined` when not compacting |
| `time.archived` | `number \| undefined` | v2 — when archived | `undefined` when active |
| `status` | `SessionStatus` | `idle` / `busy` / `retry` | Derived from `session.status` event |
| `cost` (v2) | `number \| undefined` | Accumulated cost | `undefined` until first token — NOT `0` |
| `tokens` (v2) | `Tokens \| undefined` | Token counts | `undefined` until first token |
| `agent` (v2) | `string \| undefined` | Agent name | `undefined` if not set |
| `model` (v2) | `ModelRef \| undefined` | Current model | `undefined` if not set |

**Derived states**:
- `idle`: `{ type: "idle" }` — no active request
- `busy`: `{ type: "busy" }` — request in progress
- `retry`: `{ type: "retry"; attempt: number; message: string; next: number }` — retry scheduled

### 2.2 MODEL

**Origin**: `Model` (`v2/gen/types.gen.d.ts:1653`), `Provider` (`v2/gen/types.gen.d.ts:1733`), `ModelRef` (`v2/gen/types.gen.d.ts:2390`)

| Dimension | Field | Type | Semantics |
|-----------|-------|------|-----------|
| Selection | `model.id` + `providerID` + `variant?` | `ModelRef` | Current model; `variant` optional |
| Availability | `status` | `"alpha" \| "beta" \| "deprecated" \| "active"` | Never fabricate a status |
| Cost | `cost.input`, `cost.output`, `cost.cache` | `number` | Per-token pricing; `0` valid for free tiers |
| Limit | `limit.context` | `number` | Context window; `0` means unbounded/unknown |
| Capability | `capabilities` | `{ temperature, reasoning, ... }` | Boolean flags — `false` is real, not unknown |

### 2.3 CONTEXT

**Origin**: `Model.limit`, `Session.tokens`, `StepFinishPart`

| Dimension | Source | Field | Semantics |
|-----------|--------|-------|-----------|
| Used | `StepFinishPart` | `tokens.input + tokens.output + tokens.reasoning` | Tokens consumed in current step |
| Limit | `Model.limit.context` | `number` | Max context window; `0` = unknown/unbounded |
| Percentage | Derived | `used / limit * 100` | `undefined` if limit is `0` or unknown |
| Status | Derived | Context status | `NORMAL` / `ATTENTION` / `HIGH` / `CRITICAL` / `LIMIT` |

### 2.4 TOKENS

**Origin**: `StepFinishPart.tokens`, `AssistantMessage.tokens`, `Session.tokens` (v2)

| Dimension | Field | Type | Semantics |
|-----------|-------|------|-----------|
| Input | `tokens.input` | `number` | Input tokens; `0` is valid |
| Output | `tokens.output` | `number` | Output tokens; `0` is valid |
| Reasoning | `tokens.reasoning` | `number` | Reasoning tokens; `0` is valid |
| Cache Read | `tokens.cache.read` | `number` | Cache read tokens |
| Cache Write | `tokens.cache.write` | `number` | Cache write tokens |
| Total | Derived | `input + output + reasoning` | Sum of core dimensions |

### 2.5 COST

**Origin**: `StepFinishPart.cost`, `Session.cost` (v2), `Model.cost`

| Dimension | Field | Type | Semantics |
|-----------|-------|------|-----------|
| Value | `cost` | `number \| undefined` | Accumulated cost; `undefined` if not computed |
| Currency | Derived | `"USD"` | All costs in USD; `N/A` if provider doesn't expose |
| Known | `cost !== undefined` | `boolean` | Whether cost has been computed |

**Truth rule**: If a provider does not expose cost, show `N/A` rather than `$0.00`.

### 2.6 ACTIVITY

**Origin**: `SessionStatus`, `Part` union, `message.part.updated` events

| Dimension | Source | Field | Semantics |
|-----------|--------|-------|-----------|
| Current phase | `SessionStatus` | `type: "idle" \| "busy" \| "retry"` | Session-level activity |
| Current tool | `ToolPart` | `tool`, `state.status` | Active tool call if any |
| Current message | `Part` | `type` | Part type being processed |
| Step | `StepFinishPart` | `reason`, `finish` | Step completion info |

### 2.7 TOOLS

**Origin**: `ToolPart`, `ToolState`, `McpStatus`, `SessionMessageToolState*`

| Dimension | Source | Field | Semantics |
|-----------|--------|-------|-----------|
| Shell | `ToolPart` | `tool` | Shell tool calls |
| MCP | `McpStatus` | `status` | MCP server connection state |
| LSP | `LspStatus` | `status` | LSP server connection state |
| Tool classes | `ToolPart` | `type: "tool"` | All tool call types |

**Tool state transitions**: `pending → running → completed | error`

### 2.8 PERFORMANCE

**Origin**: Derived from `time.start`, `time.end` timestamps in `Part` types and events

| Dimension | Source | Calculation | Semantics |
|-----------|--------|-------------|-----------|
| Latency | `time.end - time.start` | Duration in ms | Per-step or per-part latency |
| Duration | `time.updated - time.created` | Duration in ms | Session or message duration |
| Throughput | `tokens / elapsed` | Tokens per second | When both tokens and elapsed are known |
| Queue | Derived from message count | Pending messages | When messages are queued |

**Truth rule**: Performance metrics are computed from timestamps. If timestamps are absent, the metric is `undefined` — NOT `0`.

### 2.9 SECURITY

**Origin**: `PermissionRequest`, `McpStatus`, `Provider.key`, `Session.permission`

| Dimension | Source | Field | Semantics |
|-----------|--------|-------|-----------|
| Risk level | `PermissionRequest` | `permission`, `patterns` | Permission scope |
| Sensitivity | `McpStatus` | `status: "needs_auth"` | Auth-required servers |
| Provider auth | `Provider.key` | `string \| undefined` | Whether credentials exist |
| Permission state | `PermissionRequest` | `status` | `ask` / `deny` / `allow` |

**Note**: Risk/sensitivity indicators are provider-dependent. If not supported, show `UNKNOWN`.

---

## 3. Observation Modes

All observability output adapts to 5 density modes (see `docs/design/density.md`).

| Mode | Density | Lines | Padding | Usage |
|------|---------|-------|---------|-------|
| `MINIMAL` | Very sparse | 2 empty lines between sections | `space-lg` (8 cols) | Long sessions, monitoring |
| `STANDARD` | Balanced | 1 empty line | `space-md` (4 cols) | Daily development (default) |
| `DEVELOPER` | Dense | 0 empty lines | `space-sm` (2 cols) | Debugging, inspection |
| `OBSERVER` | Dense+ | 0 empty lines | `space-xs` (1 col) | Monitoring, logs |
| `DEBUG` | Ultra-dense | 0 empty lines, raw data | 0 | Diagnostic, traces |

---

## 4. Truth Rules

1. **`UNKNOWN != ZERO`**: Missing fields are `undefined`. `0`, `""`, `false` are real values.
2. **Never estimate**: If a value is not available, it is not displayed as factual.
3. **Never convert missing telemetry into a fake default**: `N/A` stays `N/A`.
4. **Mark approximate/provider-dependent metrics**: Clearly labeled when applicable.
5. **If provider does not expose cost**: Show `N/A` rather than `$0.00`.
6. **Every metric has an authoritative source**: See `metrics.md`.

---

## 5. Event-Driven Architecture

The observability layer is entirely event-driven. No polling.

```
Runtime emits events
  │
  ▼
TuiEventBus.on(type, handler)   ← subscribe via api.event.on()
  │
  ▼
View-model recalculation        ← derived from TuiState + event payloads
  │
  ▼
Component re-render             ← bounded, memoized selectors
```

**Key events for observability**:
- `session.status` → SESSION domain update
- `session.next.model.switched` → MODEL domain update
- `session.next.context.updated` → CONTEXT domain update
- `session.next.step.ended` → TOKENS, COST domain update
- `session.next.tool.called/progress/success/failed` → TOOLS domain update
- `permission.asked/replied` → SECURITY domain update
- `lsp.updated`, `mcp.tools.changed` → TOOLS domain update

---

## 6. Presentation State Separation

The observability view-model (`AgentObservabilityState`) is derived from `TuiState` — never a copy.

| Principle | Rule |
|-----------|------|
| Read-only | `TuiState` is `readonly`; observables read, never write |
| Derived | All calculations (tokens, cost, duration) are derived from `StepFinishPart` |
| No duplication | View-model never duplicates domain state |
| Event-synchronized | `EventMessagePartUpdated` triggers recalculation |
| Cleanup | Subscriptions cleaned in `api.lifecycle.onDispose()` |

---

## 7. Missing Data Policy

Data NOT available in the runtime must be explicitly marked as unavailable:

| Data | Available in SDK? | Contournement |
|------|-------------------|---------------|
| Latency | ❌ | Calcul via `time.updated - time.created` |
| Queue depth | ❌ | Compteur de messages/parts |
| Throughput | ❌ | Calcul : tokens / elapsed |
| Risk/Sensitivity | ❌ | À définir dans `guard.ts` / eurinhash |
| Context limit | ⚠️ Via `Model.capabilities` | Calcul |
| Cost (provider-dependent) | ⚠️ Via `StepFinishPart.cost` | `N/A` if not computed |

**Rule**: Missing data is `UNKNOWN`. It is never `ZERO`, never `""`, never fabricated.

---

## 8. References

- `docs/state/domain-state.md` — All domain state types and fields
- `docs/state/event-catalog.md` — 25+ events with full semantics
- `docs/state/unknown-data.md` — `UNKNOWN != ZERO` treatment protocol
- `docs/architecture/event-contracts.md` — Event contracts by domain
- `docs/architecture/presentation-adapters.md` — Adapter patterns
- `docs/architecture/tui-sdk-boundary.md` — TUI/SDK boundary rules
- `docs/design/density.md` — 5 density modes
- `docs/design/status-language.md` — Status vocabulary
- `docs/design/components.md` — UI components
