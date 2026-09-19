# Observability Privacy — Data Privacy

> **Source**: `docs/architecture/architecture-overview.md`, `docs/architecture/state-contracts.md`, `docs/architecture/tui-sdk-boundary.md`, `docs/architecture/upstream-migration.md`
> **Rule**: Observability data is derived from runtime state. No personal data is collected beyond what the runtime provides.

---

## 1. Privacy Principles

### 1.1 Data Minimization

The observability layer collects only data that is:
1. **Available from the runtime** via `TuiState` or SDK events
2. **Necessary for observability** (session status, model, tokens, cost, tools, performance, security)
3. **Already exposed by the runtime** — never accessed through internal APIs or direct memory inspection

### 1.2 Data Boundaries

| Data Domain | What is collected | What is NOT collected |
|-------------|-------------------|----------------------|
| SESSION | Session ID, status, duration, timestamps | User identity, session content |
| MODEL | Provider, model ID, variant, capabilities | API keys, provider credentials |
| CONTEXT | Token usage, context percentage | Prompt content, user input text |
| TOKENS | Token counts per dimension | Input/output text |
| COST | Cost value, currency | Billing information, payment methods |
| ACTIVITY | Phase, current tool, step | Tool arguments/responses |
| TOOLS | Tool name, state, duration | Tool input/output content |
| PERFORMANCE | Latency, duration, throughput | User behavior patterns |
| SECURITY | Permission status, MCP status | Permission details, user decisions |

### 1.3 Provider Key Protection

- `Provider.key` is **never** displayed or logged
- `Provider.key` is `string | undefined` — `undefined` when not set
- API keys are handled by the runtime; the observability layer only knows whether auth exists (`M-SEC-003`)
- Key presence is indicated by `Provider.key !== undefined`, never by displaying the key value

---

## 2. Data Classification

### 2.1 Classification Levels

| Level | Data | Handling | Display |
|-------|------|----------|---------|
| **Public** | Session status, model ID, token counts, cost, duration | Displayed in all modes | Full display |
| **Internal** | Tool names, tool states, MCP/LSP status | Displayed in all modes | Full display |
| **Confidential** | Permission details, provider auth status | Displayed as indicators only | Status indicator, never detail |
| **Secret** | Provider API keys, tool arguments/responses | Never collected by observability | NEVER displayed |

### 2.2 Data Flow

```
Runtime (OpenCode core)
  │
  ├── Exposes via TuiState ──► TuiPlugin (eurinhash)
  │                              │
  │                              ├── Observability view-model (derived)
  │                              │      │
  │                              │      ▼
  │                              │   TUI rendering (Terminal)
  │                              │
  │                              └── Event subscriptions (api.event.on)
  │
  ├── Exposes via Events ──► TuiEventBus ──► Handlers
  │
  └── NEVER exposes:
      ├── Provider API keys
      ├── Tool input/output content
      ├── User prompt text
      ├── Message content (unless explicitly in TuiState)
      └── Permission decisions (only status is observable)
```

---

## 3. TUI/SDK Boundary Privacy

### 3.1 Boundary Rules

From `docs/architecture/tui-sdk-boundary.md`:

1. **The TUI does not reach into backend internals** — all data comes through `TuiState` or `OpencodeClient`
2. **`TuiState` is read-only** — no data can be written back
3. **All types come from SDK `.d.ts`** — no internal types accessed
4. **No direct memory access** — nothing is read outside the SDK boundary

### 3.2 What TuiState Exposes vs. What It Doesn't

| Available via TuiState | NOT available via TuiState |
|------------------------|---------------------------|
| `session.get(id)` | Prompt text |
| `session.status(id)` | Tool arguments/responses |
| `session.messages(id)` | Full message content (only metadata) |
| `session.permission(id)` | Permission decision rationale |
| `part(messageID)` | Part text content (only type + metadata) |
| `provider` | Provider API keys |
| `mcp()` | MCP server internal state |
| `lsp()` | LSP diagnostic details (only status) |
| `config` | Full configuration with secrets |
| `path` | File contents |
| `vcs` | Git diff content |

---

## 4. Plugin Privacy

### 4.1 Plugin Data Access

The eurinhash plugin (`TuiPlugin`) accesses only:
- `api.state` (read-only)
- `api.event` (subscribe to events)
- `api.slots` (register rendering slots)
- `api.theme` (read theme)
- `api.kv` (key-value store)
- `api.client` (SDK client)

### 4.2 Plugin Data Restrictions

The plugin **never**:
1. Reads `Provider.key` directly
2. Accesses tool input/output content
3. Accesses message/prompt text
4. Accesses permission decision content
5. Accesses internal server state beyond the SDK boundary
6. Sends data to external services
7. Stores personal data in `TuiKV`

### 4.3 Hooks Privacy

Server-side hooks (`Hooks` interface) have access to more data, but:
1. Hooks are server-side, not TUI-side
2. Hook implementations (e.g., `guard.ts`) are separate from the observability layer
3. Observability does not use hook data directly
4. Hook data is not passed to the TUI

---

## 5. Event Privacy

### 5.1 Event Payload Classification

| Event | Payload Data | Privacy Level |
|-------|-------------|---------------|
| `session.status` | `sessionID`, `status` | Public |
| `session.created` | `info: Session` | Public (metadata only) |
| `session.next.model.switched` | `model: ModelRef` | Public |
| `session.next.step.ended` | `cost`, `tokens` | Public |
| `session.next.tool.called` | `tool`, `input` metadata | Internal (no content) |
| `session.next.tool.success` | `structured`, `content` | Internal (metadata only) |
| `session.next.tool.failed` | `error` | Internal |
| `permission.asked` | `action`, `resources`, `patterns` | Confidential |
| `permission.replied` | `reply` ("once"\|"always"\|"reject") | Confidential |
| `lsp.client.diagnostics` | `serverID`, `path` | Internal |
| `mcp.tools.changed` | `server` | Internal |

### 5.2 Event Handling Privacy Rules

1. **Observability handlers never log full event payloads** — only extract needed fields
2. **Event data is not stored persistently** — only used for current rendering
3. **No event data is sent externally** — all processing is local to the TUI
4. **Error messages are sanitized** — `SessionErrorUnknown` is `{ type: "unknown"; message: string }`; the message may contain provider details but is not logged

---

## 6. KV Store Privacy

### 6.1 `TuiKV` Usage

```typescript
type TuiKV = {
    get: <Value = unknown>(key: string, fallback?: Value) => Value;
    set: (key: string, value: unknown) => void;
    readonly ready: boolean;
};
```

### 6.2 KV Privacy Rules

1. **Never store** provider keys, tokens, cost values in KV
2. **Never store** user-specific data in KV
3. **Only store** UI preferences, theme settings, density mode
4. **Keys must be namespaced** — e.g., `eurinhash:density`, `eurinhash:theme`
5. **Fallback values** are provided via `get(key, fallback)` — `undefined` means not set

---

## 7. Serialization Privacy

### 7.1 JSON Serialization

When serializing observability data for display:

1. **Never serialize** `Provider.key` — even if `undefined`, it must not appear
2. **Never serialize** tool input/output content
3. **Never serialize** message/prompt text
4. **Serialize only** derived metrics (tokens, cost, duration, status)
5. **`undefined` fields are omitted** from JSON — never replaced with `null` or `""`

### 7.2 Example: Safe Session Serialization

```json
// ✅ CORRECT — only observability-relevant fields
{
    "id": "abc-123",
    "status": "busy",
    "model": { "id": "claude-sonnet-4", "providerID": "anthropic" },
    "tokens": { "input": 45200, "output": 52100, "reasoning": 3100 },
    "cost": 0.00,
    "duration": 153000
}

// ❌ FORBIDDEN — includes secret or content data
{
    "id": "abc-123",
    "providerKey": "sk-...",           // NEVER include API keys
    "prompt": "Please refactor...",    // NEVER include prompt text
    "toolOutput": "result..."          // NEVER include tool output
}
```

---

## 8. Privacy by Design

### 8.1 Architecture Decisions

| Decision | Rationale |
|----------|-----------|
| TuiState is read-only | Prevents accidental data leakage via write |
| Only SDK types accessed | No internal data structures exposed |
| Event-driven, not polling | No periodic scanning of all state |
| View-model derived, not stored | No persistent cache of sensitive data |
| No external data transmission | All processing is local to the TUI |
| `UNKNOWN` not `ZERO` | Prevents false confidence in data accuracy |

### 8.2 Data Retention

1. **Observability view-model**: Not retained between renders — recomputed on each event
2. **Event data**: Not stored — processed and discarded
3. **TuiKV**: Only UI preferences persisted — cleared on plugin deactivate
4. **No telemetry to external services**: All observability is local

### 8.3 User Control

Users can control observability through:
- `/density` command — adjust display density
- `/theme` command — switch theme
- Plugin deactivation — `api.plugins.deactivate("eurinhash")` stops all observability
- `api.lifecycle.onDispose()` — full cleanup on unload

---

## 9. Compliance with `UNKNOWN != ZERO` Privacy Implications

The `UNKNOWN != ZERO` rule has direct privacy implications:

| Scenario | UNKNOWN (correct) | ZERO (wrong) | Privacy Impact |
|----------|-------------------|--------------|----------------|
| Cost not computed | `N/A` | `$0.00` | Misleading: user thinks service is free |
| Tokens not counted | `N/A` | `0 tokens` | Misleading: user thinks no tokens used |
| Duration unknown | `N/A` | `0s` | Misleading: user thinks operation was instant |
| Model not set | `N/A` | `model: ""` | Misleading: implies a model exists |

**Rule**: Displaying `ZERO` for `UNKNOWN` data is a **privacy and accuracy violation**. It creates a false representation of the system state.

---

## 10. Privacy Validation

### 10.1 Validation Checklist

```
CHECKLIST — Privacy Validation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
☐ Provider API keys never displayed or logged
☐ Tool input/output content never accessed by observability
☐ Prompt/message text never accessed by observability
☐ Permission decisions never exposed in detail
☐ All data comes from TuiState or SDK events
☐ No data sent to external services
☐ No persistent storage of sensitive data
☐ TuiKV only stores UI preferences
☐ JSON serialization omits undefined fields (never null)
☐ UNKNOWN displayed as N/A or [? UNKNOWN], never as 0
☐ No polling of runtime state
☐ All event subscriptions cleaned in onDispose
☐ Plugin deactivation removes all observability data
☐ No internal server types accessed directly
☐ No memory inspection or direct access to runtime internals
```

### 10.2 Privacy Incident Response

| Signal | Detection | Action |
|--------|-----------|--------|
| Unexpected data in display | Code review of render components | Remove data field, verify source |
| Data sent externally | Network inspection | Block, remove transmission code |
| Persistent storage of secrets | KV inspection | Clear KV, add validation |
| API key exposure | Static analysis of observability code | Remove key access, use indicator only |

---

## 11. References

- `docs/architecture/architecture-overview.md` — System architecture and boundaries
- `docs/architecture/tui-sdk-boundary.md` — TUI/SDK boundary rules
- `docs/architecture/state-contracts.md` — State contract and readonly rules
- `docs/architecture/upstream-migration.md` — Migration constraints
- `docs/architecture/plugin-presentation-model.md` — Plugin architecture
- `docs/architecture/event-contracts.md` — Event contracts
- `docs/state/unknown-data.md` — `UNKNOWN != ZERO` protocol
- `docs/state/domain-state.md` — Domain state definitions
- `docs/design/design-tokens.md` — Design tokens and theme strategy
