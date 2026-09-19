# Observability Metrics — Every Metric with Authoritative Source

> **Source**: `@opencode-ai/sdk/dist/gen/types.gen.d.ts`, `@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`, `@opencode-ai/plugin/dist/index.d.ts`, `@opencode-ai/plugin/dist/tui.d.ts`
> **Rule**: Every metric has a single authoritative source file/symbol/API field. `UNKNOWN != ZERO`.

---

## Metric Index

| ID | Metric | Domain | Authoritative Source | Type |
|----|--------|--------|---------------------|------|
| M-SES-001 | Session ID | SESSION | `Session.id` | string |
| M-SES-002 | Session Status | SESSION | `SessionStatus` via `api.state.session.status(id)` | SessionStatus |
| M-SES-003 | Session Duration | SESSION | `Session.time.created`, `Session.time.updated` | number (ms) |
| M-SES-004 | Session Created At | SESSION | `Session.time.created` | number (unix ms) |
| M-MDL-001 | Model Provider | MODEL | `Session.model.providerID` | string |
| M-MDL-002 | Model ID | MODEL | `Session.model.id` | string |
| M-MDL-003 | Model Variant | MODEL | `Session.model.variant` | string \| undefined |
| M-MDL-004 | Model Status | MODEL | `Model.status` | "alpha"\|"beta"\|"deprecated"\|"active" |
| M-MDL-005 | Model Cost | MODEL | `Model.cost` | { input, output, cache } |
| M-MDL-006 | Model Context Limit | MODEL | `Model.limit.context` | number |
| M-CTX-001 | Context Used | CONTEXT | `StepFinishPart.tokens` sum | number |
| M-CTX-002 | Context Limit | CONTEXT | `Model.limit.context` | number |
| M-CTX-003 | Context Percentage | CONTEXT | Derived: used / limit * 100 | number \| undefined |
| M-CTX-004 | Context Status | CONTEXT | Derived from percentage | "NORMAL"\|"ATTENTION"\|"HIGH"\|"CRITICAL"\|"LIMIT" |
| M-TOK-001 | Input Tokens | TOKENS | `StepFinishPart.tokens.input` | number |
| M-TOK-002 | Output Tokens | TOKENS | `StepFinishPart.tokens.output` | number |
| M-TOK-003 | Reasoning Tokens | TOKENS | `StepFinishPart.tokens.reasoning` | number |
| M-TOK-004 | Cache Read Tokens | TOKENS | `StepFinishPart.tokens.cache.read` | number |
| M-TOK-005 | Cache Write Tokens | TOKENS | `StepFinishPart.tokens.cache.write` | number |
| M-TOK-006 | Total Tokens | TOKENS | Derived: input + output + reasoning | number |
| M-CST-001 | Step Cost | COST | `StepFinishPart.cost` | number \| undefined |
| M-CST-002 | Session Cost | COST | `Session.cost` (v2) | number \| undefined |
| M-CST-003 | Cost Currency | COST | Derived | "USD" \| "N/A" |
| M-ACT-001 | Current Phase | ACTIVITY | `SessionStatus.type` | "idle"\|"busy"\|"retry" |
| M-ACT-002 | Current Tool | ACTIVITY | `ToolPart.tool`, `ToolPart.state.status` | string \| undefined |
| M-ACT-003 | Current Step | ACTIVITY | `StepFinishPart.reason`, `StepFinishPart.finish` | string \| undefined |
| M-TOO-001 | Tool Call ID | TOOLS | `ToolPart.callID` | string |
| M-TOO-002 | Tool Name | TOOLS | `ToolPart.tool` | string |
| M-TOO-003 | Tool State | TOOLS | `ToolPart.state.status` | ToolState |
| M-TOO-004 | Tool Duration | TOOLS | `ToolPart.state.time.end - ToolPart.state.time.start` | number \| undefined |
| M-TOO-005 | MCP Server Status | TOOLS | `McpStatus.status` | McpStatus |
| M-TOO-006 | MCP Server Error | TOOLS | `McpStatusFailed.error` | string \| undefined |
| M-TOO-007 | LSP Server Status | TOOLS | `LspStatus.status` | "connected"\|"error" |
| M-PRF-001 | Latency | PERFORMANCE | `time.end - time.start` | number \| undefined |
| M-PRF-002 | Duration | PERFORMANCE | `time.updated - time.created` | number \| undefined |
| M-PRF-003 | Throughput | PERFORMANCE | Derived: tokens / elapsed | number \| undefined |
| M-PRF-004 | Queue Depth | PERFORMANCE | Derived from pending messages | number \| undefined |
| M-SEC-001 | Permission Status | SECURITY | `PermissionRequest.status` | "ask"\|"deny"\|"allow" |
| M-SEC-002 | Permission Action | SECURITY | `PermissionRequest.permission` | string |
| M-SEC-003 | Provider Auth | SECURITY | `Provider.key` | string \| undefined |
| M-SEC-004 | MCP Auth Required | SECURITY | `McpStatus.status === "needs_auth"` | boolean |
| M-SEC-005 | Risk Level | SECURITY | Derived from Permission patterns | string \| undefined |

---

## Detailed Metric Specifications

### SESSION Domain (M-SES-*)

#### M-SES-001 — Session ID
- **Type**: `string`
- **Source**: `Session.id` (`types.gen.d.ts:465`, `v2/gen/types.gen.d.ts:64`)
- **Selector**: `api.state.session.get(sessionID)?.id`
- **UNKNOWN**: Never absent for live sessions

#### M-SES-002 — Session Status
- **Type**: `SessionStatus` = `{ type: "idle" } | { type: "busy" } | { type: "retry"; attempt, message, next }`
- **Source**: `SessionStatus` event / `api.state.session.status(sessionID)`
- **Selector**: `api.state.session.status(sessionID)`
- **Events**: `session.status`, `session.idle`
- **UNKNOWN**: Must be one of three discriminated variants; never fabricate

#### M-SES-003 — Session Duration
- **Type**: `number` (milliseconds)
- **Source**: `Session.time.created`, `Session.time.updated`
- **Calculation**: `time.updated - time.created`
- **UNKNOWN**: If either timestamp is `undefined` → duration is `undefined`

#### M-SES-004 — Session Created At
- **Type**: `number` (Unix ms)
- **Source**: `Session.time.created`
- **Selector**: `api.state.session.get(sessionID)?.time.created`
- **UNKNOWN**: `undefined` if not set — NOT `0`

### MODEL Domain (M-MDL-*)

#### M-MDL-001 — Model Provider
- **Type**: `string`
- **Source**: `Session.model.providerID` / `Model.providerID`
- **Selector**: `api.state.session.get(sessionID)?.model?.providerID`
- **UNKNOWN**: `undefined` if no model selected

#### M-MDL-002 — Model ID
- **Type**: `string`
- **Source**: `Session.model.id` / `Model.id`
- **Selector**: `api.state.session.get(sessionID)?.model?.id`
- **UNKNOWN**: `undefined` if no model selected

#### M-MDL-003 — Model Variant
- **Type**: `string \| undefined`
- **Source**: `Session.model.variant` / `ModelRef.variant`
- **Selector**: `api.state.session.get(sessionID)?.model?.variant`
- **UNKNOWN**: `undefined` = default variant — NOT "no variant selected"

#### M-MDL-004 — Model Status
- **Type**: `"alpha" | "beta" | "deprecated" | "active"`
- **Source**: `Model.status`
- **Selector**: `api.state.provider` → find model → `.status`
- **UNKNOWN**: Must be one of four values; never fabricate

#### M-MDL-005 — Model Cost (per-token)
- **Type**: `{ input: number, output: number, cache: { read: number, write: number } }`
- **Source**: `Model.cost`
- **Selector**: Provider model definition
- **UNKNOWN**: `0` is valid for free tiers; field always exists if model is defined

#### M-MDL-006 — Model Context Limit
- **Type**: `number`
- **Source**: `Model.limit.context`
- **Selector**: Provider model definition
- **UNKNOWN**: `0` means unbounded or unknown — NOT a valid limit number

### CONTEXT Domain (M-CTX-*)

#### M-CTX-001 — Context Used
- **Type**: `number` (tokens)
- **Source**: `StepFinishPart.tokens.input + StepFinishPart.tokens.output + StepFinishPart.tokens.reasoning`
- **Selector**: `api.state.part(messageID)` → find `StepFinishPart` → sum tokens
- **Events**: `session.next.step.ended`, `message.part.updated`

#### M-CTX-002 — Context Limit
- **Type**: `number`
- **Source**: `Model.limit.context`
- **Selector**: `api.state.provider` → model → `limit.context`
- **UNKNOWN**: `0` means unbounded/unknown

#### M-CTX-003 — Context Percentage
- **Type**: `number \| undefined`
- **Source**: Derived: `contextUsed / contextLimit * 100`
- **Calculation**: Only when both used and limit are known and limit > 0
- **UNKNOWN**: `undefined` if limit is `0` or unknown — NOT `0%`

#### M-CTX-004 — Context Status
- **Type**: `"NORMAL" | "ATTENTION" | "HIGH" | "CRITICAL" | "LIMIT"`
- **Source**: Derived from percentage (see `presentation-adapters.md` `getContextStatus()`)
- **Thresholds**: ≥100% LIMIT, ≥90% CRITICAL, ≥75% HIGH, ≥50% ATTENTION, <50% NORMAL

### TOKENS Domain (M-TOK-*)

#### M-TOK-001 through M-TOK-006 — Token Metrics
- **Source**: `StepFinishPart.tokens` — the authoritative source of truth
- **Selector**: `api.state.part(messageID)` → find `StepFinishPart` → `.tokens`
- **Events**: `session.next.step.ended`
- **Rule**: `0` is valid for any token dimension (genuinely zero). `undefined` means not yet counted.
- **Source code**: `v2/gen/types.gen.d.ts` — `StepFinishPart` has `tokens: { input, output, reasoning, cache: { read, write } }`

### COST Domain (M-CST-*)

#### M-CST-001 — Step Cost
- **Type**: `number \| undefined`
- **Source**: `StepFinishPart.cost`
- **Selector**: `api.state.part(messageID)` → find `StepFinishPart` → `.cost`
- **Events**: `session.next.step.ended`
- **UNKNOWN**: `undefined` until step completes — NOT `0` as placeholder

#### M-CST-002 — Session Cost
- **Type**: `number \| undefined` (v2 only)
- **Source**: `Session.cost`
- **Selector**: `api.state.session.get(sessionID)?.cost`
- **UNKNOWN**: `undefined` until first token; NOT `0` as placeholder

#### M-CST-003 — Cost Currency
- **Type**: `"USD" | "N/A"`
- **Source**: Derived — all OpenCode costs are in USD
- **Rule**: If provider does not expose cost, show `N/A` rather than `$0.00`

### ACTIVITY Domain (M-ACT-*)

#### M-ACT-001 — Current Phase
- **Type**: `"idle" | "busy" | "retry"`
- **Source**: `SessionStatus` via `api.state.session.status(sessionID)`
- **Events**: `session.status`, `session.idle`

#### M-ACT-002 — Current Tool
- **Type**: `string \| undefined`
- **Source**: `ToolPart.tool` when `ToolPart.state.status === "running"`
- **Selector**: `api.state.part(messageID)` → find running `ToolPart`
- **UNKNOWN**: `undefined` when no tool is currently executing

#### M-ACT-003 — Current Step
- **Type**: `string \| undefined`
- **Source**: `StepFinishPart.reason`, `StepFinishPart.finish`
- **Selector**: `api.state.part(messageID)` → find `StepFinishPart`

### TOOLS Domain (M-TOO-*)

#### M-TOO-001 through M-TOO-004 — Tool Metrics
- **Source**: `ToolPart` (v1) / `SessionMessageToolState*` (v2)
- **Selector**: `api.state.part(messageID)` → find `ToolPart`
- **Events**: `session.next.tool.called`, `.progress`, `.success`, `.failed`
- **ToolState**: `pending → running → completed | error`

#### M-TOO-005 — MCP Server Status
- **Type**: `McpStatus` (discriminated union of 5 states)
- **Source**: `McpStatus` via `api.state.mcp()`
- **Selector**: `api.state.mcp()` → `.status`
- **States**: `connected | disabled | failed | needs_auth | needs_client_registration`

#### M-TOO-006 — MCP Server Error
- **Type**: `string \| undefined`
- **Source**: `McpStatusFailed.error`
- **Selector**: `api.state.mcp()` → filter `status === "failed"` → `.error`
- **UNKNOWN**: `undefined` in all states except `failed` and `needs_client_registration`

#### M-TOO-007 — LSP Server Status
- **Type**: `"connected" | "error"`
- **Source**: `LspStatus.status` via `api.state.lsp()`
- **Selector**: `api.state.lsp()` → `.status`

### PERFORMANCE Domain (M-PRF-*)

#### M-PRF-001 — Latency
- **Type**: `number \| undefined` (ms)
- **Source**: `time.start`, `time.end` in `Part` types
- **Calculation**: `time.end - time.start`
- **UNKNOWN**: If either timestamp is `undefined` → `undefined`

#### M-PRF-002 — Duration
- **Type**: `number \| undefined` (ms)
- **Source**: `time.created`, `time.updated` in `Session` / `Message`
- **Calculation**: `time.updated - time.created`
- **UNKNOWN**: If either timestamp is `undefined` → `undefined`

#### M-PRF-003 — Throughput
- **Type**: `number \| undefined` (tokens/second)
- **Source**: Derived from `M-TOK-*` and `M-PRF-002`
- **Calculation**: `totalTokens / (duration / 1000)`
- **UNKNOWN**: If duration or tokens are `undefined` → `undefined`

#### M-PRF-004 — Queue Depth
- **Type**: `number \| undefined`
- **Source**: Derived from pending messages
- **Calculation**: Count of messages not yet processed
- **UNKNOWN**: If message queue state is not tracked → `undefined`

### SECURITY Domain (M-SEC-*)

#### M-SEC-001 — Permission Status
- **Type**: `"ask" | "deny" | "allow"`
- **Source**: `PermissionRequest.status` via `api.state.session.permission(sessionID)`
- **Events**: `permission.asked`, `permission.replied`, `permission.v2.asked`, `permission.v2.replied`

#### M-SEC-002 — Permission Action
- **Type**: `string`
- **Source**: `PermissionRequest.permission` / `PermissionV2Asked.action`
- **Selector**: `api.state.session.permission(sessionID)`

#### M-SEC-003 — Provider Auth
- **Type**: `string \| undefined`
- **Source**: `Provider.key`
- **Selector**: `api.state.provider` → `.key`
- **UNKNOWN**: `undefined` when no API key configured — NOT `""`

#### M-SEC-004 — MCP Auth Required
- **Type**: `boolean`
- **Source**: `McpStatus.status === "needs_auth"`
- **Selector**: `api.state.mcp()` → check status

#### M-SEC-005 — Risk Level
- **Type**: `string \| undefined`
- **Source**: Derived from `PermissionRequest.patterns` and `McpStatus`
- **UNKNOWN**: If risk/sensitivity not supported by runtime → `undefined` (NOT `0`)

---

## Metric Sources Summary

All authoritative sources are in the SDK type definitions:

| Source File | Types Provided |
|-------------|---------------|
| `@opencode-ai/sdk/dist/gen/types.gen.d.ts` | v1 Event, Session, Message, Part, Model, Provider, Permission, McpStatus, LspStatus |
| `@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts` | v2 Event, Session, SessionMessage, Model, Provider, PermissionRequest, McpStatus, LspStatus, StepFinishPart |
| `@opencode-ai/plugin/dist/index.d.ts` | Plugin, Hooks, ToolDefinition, Config, Provider |
| `@opencode-ai/plugin/dist/tui.d.ts` | TuiState, TuiPluginApi, TuiEventBus, TuiTheme, TuiSidebarMcpItem, TuiSidebarLspItem |

---

## Event-to-Metric Mapping

| Event | Metrics Updated | Domain |
|-------|----------------|--------|
| `session.status` | M-SES-002, M-ACT-001 | SESSION, ACTIVITY |
| `session.created` | M-SES-001, M-SES-004 | SESSION |
| `session.updated` | M-SES-001, M-SES-003 | SESSION |
| `session.idle` | M-ACT-001 | ACTIVITY |
| `session.next.model.switched` | M-MDL-001, M-MDL-002, M-MDL-003 | MODEL |
| `session.next.context.updated` | M-CTX-001 | CONTEXT |
| `session.next.step.ended` | M-TOK-001..006, M-CST-001, M-CTX-001, M-PRF-001, M-PRF-002 | TOKENS, COST, CONTEXT, PERFORMANCE |
| `session.next.tool.called` | M-TOO-001, M-TOO-002, M-TOO-003 | TOOLS |
| `session.next.tool.progress` | M-TOO-003 | TOOLS |
| `session.next.tool.success` | M-TOO-003, M-TOO-004 | TOOLS |
| `session.next.tool.failed` | M-TOO-003, M-TOO-004 | TOOLS |
| `permission.asked` | M-SEC-001, M-SEC-002 | SECURITY |
| `permission.replied` | M-SEC-001 | SECURITY |
| `lsp.updated` | M-TOO-007 | TOOLS |
| `mcp.tools.changed` | M-TOO-005, M-TOO-006 | TOOLS |
| `message.part.updated` | M-CTX-001, M-TOK-* | CONTEXT, TOKENS |
