# Observability Presentation — Compact and Detailed Views

> **Source**: `docs/design/density.md`, `docs/design/components.md`, `docs/design/status-language.md`, `docs/architecture/presentation-adapters.md`
> **Rule**: Support drill-down from summary metric to evidence. Avoid continuously changing values that cause terminal flicker.

---

## 1. View Architecture

The observability layer provides two view families: **compact** (single-line summaries) and **detailed** (multi-line drill-downs). Both adapt to 5 density modes.

```
┌─────────────────────────────────────────────────────────────┐
│                    VIEW ARCHITECTURE                           │
│                                                               │
│  COMPACT VIEW                                          │
│  ├── StatusBar (app_bottom slot)                         │
│  │   ├── [● ACTIVE] eurinhash · 38%                     │
│  │   ├── [◐ THINKING] novita · 2.3s                     │
│  │   └── [✗ ERROR] google · N/A                         │
│  │                                                        │
│  └── StatusCompact component                            │
│       └── Single-line status indicators                  │
│                                                               │
│  DETAILED VIEW                                         │
│  ├── StatusDetailed (sidebar_footer slot)                │
│  │   ├── Full session info                              │
│  │   ├── Token breakdown                                │
│  │   ├── Cost breakdown                                 │
│  │   └── Tool list                                      │
│  │                                                        │
│  └── StatusDebug (app slot)                              │
│       └── All domains, raw data, DEBUG mode              │
│                                                               │
│  DRILL-DOWN                                              │
│  ├── Click StatusCompact → StatusDetailed                │
│  └── Click StatusDetailed → StatusDebug                  │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Compact View

### 2.1 StatusBar (Level 1 — app_bottom)

The primary compact observability display. One line per agent/service.

**Format**: `[GLYPH STATUS] label · metric1 · metric2 · metric3`

**Example**:
```
[● ACTIVE] eurinhash-agent · 38% · 100K / 262K · $0.00
[◐ THINKING] worker-novita · 2.3s · Attente réponse
[✗ ERROR] worker-google · Connection refused · N/A
[○ IDLE] worker-pollinations
```

**Components**:
- `StatusIndicator` — glyph + status + label + optional metric
- `MetricDisplay` — formatted value in `mono`
- `StatusBar` — collection of `StatusIndicator` items

**Density adaptation**:

| Mode | Format |
|------|--------|
| MINIMAL | One status per line, 2 empty lines between |
| STANDARD | One status per line, 1 empty line between |
| DEVELOPER | All statuses on one line, separated by `·` |
| OBSERVER | All statuses on one line, full metrics |
| DEBUG | Single line, all data raw |

### 2.2 Compact Session Summary

```
[● ACTIVE] session-id · 38% · $0.00 · 2.3s
```

| Field | Source | Density |
|-------|--------|---------|
| Status glyph + label | `SessionStatus` | All modes |
| Session identifier | `Session.id` | All modes |
| Context percentage | `M-CTX-003` | STANDARD+ |
| Cost | `M-CST-001` | STANDARD+ |
| Duration | `M-PRF-002` | STANDARD+ |

### 2.3 Compact Tool Status

```
[▶ RUNNING] tool-name · input.args · 1.2s
[✓ SUCCESS] tool-name · 0.8s · output.size
[✗ ERROR] tool-name · error.message · N/A
[○ PENDING] tool-name · queued
```

### 2.4 Compact MCP/LSP Status

```
MCP: [● connected] server1 · [○ disabled] server2 · [✗ failed] server3
LSP: [● connected] eslint · [○ error] pyright
```

---

## 3. Detailed View

### 3.1 StatusDetailed (sidebar_footer slot)

Drill-down from compact view. Shows full domain data for a session.

**Format**: Key-value list with sections.

```
╭─ SESSION: my-session ──────────────────────────────╮
│ ID:        abc-123                                   │
│ Status:    [● ACTIVE]                               │
│ Created:   2026-09-19 12:00:00                      │
│ Duration:  2m 33s                                   │
│ Directory: /workspace/project                       │
│ Agent:     eurinhash-agent                          │
│ Model:     anthropic/claude-sonnet-4 · variant: fast│
│ Model status: [● ACTIVE]                            │
│ Provider:  anthropic · key: ***                     │
╰──────────────────────────────────────────────────────╯

╭─ CONTEXT ──────────────────────────────────────────╮
│ Used:     100.4K tokens                             │
│ Limit:    262K tokens                               │
│ Usage:    [████████░░] 38%                          │
│ Status:   [● NORMAL]                                │
╰──────────────────────────────────────────────────────╯

╭─ TOKENS ──────────────────────────────────────────╮
│ Input:    45.2K                                   │
│ Output:   52.1K                                   │
│ Reasoning: 3.1K                                   │
│ Cache read: 0                                     │
│ Cache write: 0                                    │
│ Total:    100.4K                                  │
╰──────────────────────────────────────────────────────╯

╭─ COST ────────────────────────────────────────────╮
│ Step:     $0.00                                   │
│ Session:  $0.00                                   │
│ Currency: USD                                     │
╰──────────────────────────────────────────────────────╯

╭─ TOOLS ──────────────────────────────────────────╮
│ [▶ RUNNING] bash · ls -la · 0.5s                 │
│ [✓ SUCCESS] grep · pattern · 0.1s                │
│ [✗ ERROR]  npm · install · Connection refused    │
│ [○ PENDING] pip · install · queued               │
╰──────────────────────────────────────────────────────╯

╭─ PERFORMANCE ────────────────────────────────────╮
│ Latency:  230ms                                   │
│ Duration: 2m 33s                                  │
│ Throughput: 120 tokens/s                          │
│ Queue:    2 pending                               │
╰──────────────────────────────────────────────────────╯

╭─ SECURITY ───────────────────────────────────────╮
│ Permission: [○ IDLE] no pending                  │
│ Provider auth: [✓ present]                        │
│ MCP auth:  [○ 1 server needs auth]                │
╰──────────────────────────────────────────────────────╯
```

### 3.2 StatusDebug (app slot)

Maximum detail for diagnostic purposes. Only in DEBUG mode.

```
2026-09-19T12:00:00.000Z [● ACTIVE] session-id=abc-123 pid=12345 mem=45MB cpu=12%
  model=anthropic/claude-sonnet-4 provider=anthropic variant=fast status=ACTIVE
  tokens: input=45200 output=52100 reasoning=3100 cache={read:0,write:0}
  cost=$0.00 context=100400/262000=38% duration=2m33s latency=230ms
  tools: bash(running) grep(success:0.1s) npm(error: Connection refused) pip(pending)
  mcp: server1(connected) server2(disabled) server3(failed)
  lsp: eslint(connected) pyright(error)
  permission: idle
  provider_key=*** has_auth=true
```

---

## 4. Drill-Down Patterns

### 4.1 From Compact to Detailed

```
User clicks [● ACTIVE] eurinhash-agent in StatusBar
  │
  ▼
StatusBar → StatusCompact → StatusDetailed
  │
  ▼
Loads full session data via api.state.session.get(sessionID)
  │
  ▼
Renders all domain sections (context, tokens, cost, tools, performance, security)
```

### 4.2 From Detailed to Debug

```
User triggers debug mode (command /debug or density DEBUG)
  │
  ▼
StatusDetailed → StatusDebug
  │
  ▼
All domains rendered in raw format with timestamps
  │
  ▼
Raw event data appended for trace
```

### 4.3 Drill-Down Rules

1. **No data fabrication**: Drill-down shows only data available from `TuiState`
2. **Lazy loading**: Detailed view loads on demand, not pre-computed
3. **Event-driven**: Updates only when relevant events fire
4. **Memoized selectors**: Re-render only when source data changes
5. **No flicker**: Continuously changing values must not cause terminal flicker

---

## 5. Presentation Components

### 5.1 StatusIndicator

```
[GLYPH STATUS] label · metric
```

| Property | Type | Description |
|----------|------|-------------|
| `status` | Status enum | One of 11 statuses |
| `label` | `string` | Name of agent/service |
| `metric` | `string` | Optional metric value |
| `thinkingOpacity` | `number` | Applied if THINKING |

### 5.2 MetricDisplay

Standardized metric formatting per `docs/design/components.md`.

| Format | Condition |
|--------|-----------|
| `38%` | Simple percentage |
| `100K / 262K · 38%` | Usage with known limit |
| `100K · 38%` | Limit unknown |
| `$0.00` | Cost known and zero |
| `N/A` | Cost unknown |
| `2.3s` | Duration |
| `1.2M` / `100.4K` | Token count |

### 5.3 ProgressBar

```
[████████░░] 38%          (≥120 cols)
38%                     (80–119 cols)
0.38                    (DEBUG mode)
```

### 5.4 Table (for multi-item display)

| Width | Format |
|-------|--------|
| ≥120 | Table complete with borders |
| 100–119 | Table compact |
| 80–99 | Key-value list |
| <80 | JSON lines |

### 5.5 KeyValueList

```
Nom:        eurinhash
Statut:     [● ACTIVE]
Progression: 38%
Coût:       $0.00
```

---

## 6. Flicker Prevention

### 6.1 Rules

1. **Memoized selectors**: Values derived from `TuiState` are memoized; re-render only when source changes
2. **Event-driven updates**: Component updates only on relevant events, not on every render cycle
3. **Bounded rendering**: Number of DOM operations per event is bounded
4. **No polling**: If the runtime emits events, no polling is used
5. **Stable layout**: Component positions must not shift when values change
6. **Batch updates**: Multiple state changes in the same event loop are batched

### 6.2 Anti-Flicker Patterns

| Pattern | Implementation |
|---------|---------------|
| Stable metric display | `MetricDisplay` only re-renders when value changes |
| Status transitions | `StatusIndicator` animates via `thinkingOpacity`, not position |
| Progress bar | Width changes smoothly, not jump-rendered |
| Token count | Format changes (K→M) only at thresholds, not every update |
| Cost display | `$0.00` → `$0.01` update, no intermediate flashes |

---

## 7. Mode-Specific Presentation

### 7.1 MINIMAL

```
[● ACTIVE] eurinhash-agent  ·  38%

[◐ THINKING]  worker-novita  ·  2.3s

[✓ SUCCESS]  build-completed  ·  2.3s
```

- 2 empty lines between sections
- `space-lg` padding
- `display` typography for titles
- Only status + label + primary metric

### 7.2 STANDARD (Default)

```
[● ACTIVE] eurinhash-agent  ·  38%
[◐ THINKING]  worker-novita  ·  2.3s
[✓ SUCCESS]  build-completed  ·  2.3s
```

- 1 empty line between sections
- `space-md` padding
- `heading` typography for titles
- Status + label + metrics

### 7.3 DEVELOPER

```
[● ACTIVE] eurinhash-agent · 38% · [◐ THINKING] worker-novita · 2.3s · [✓ SUCCESS] build-completed · 2.3s
```

- 0 empty lines
- `space-sm` padding
- `body` typography
- All metrics inline

### 7.4 OBSERVER

```
[● ACTIVE] eurinhash-agent · 38% · [◐ THINKING] worker-novita · 2.3s · [✓ SUCCESS] build-completed · 2.3s · [✗ ERROR] worker-google · Connection refused · N/A
```

- 0 empty lines
- `space-xs` padding
- `mono` typography
- Maximum metrics per line
- 160 column width

### 7.5 DEBUG

```
2026-09-19T12:00:00.000Z [● ACTIVE] eurinhash-agent pid=12345 mem=45MB cpu=12% · 38% · 100K/262K · $0.00 · worker-novita pid=67890 status=THINKING latency=2300ms
```

- 0 empty lines
- 0 padding
- `mono` only, raw data
- Timestamps, PIDs, memory, CPU
- Unlimited width with auto-wrap

---

## 8. Slot Registration

All observability views are registered via `api.slots.register()`:

```typescript
api.slots.register({
    id: "eurinhash-observability",
    slots: {
        app_bottom: () => <StatusBar />,                          // Compact (all modes)
        sidebar_footer: ({ session_id }) => <StatusDetailed sessionID={session_id} />,  // Detailed
        app: () => <StatusDebug />,                               // Debug mode only
    },
});
```

---

## 9. Presentation Validation Checklist

```
CHECKLIST — Presentation Validation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
☐ Compact view renders in app_bottom slot
☐ Detailed view renders in sidebar_footer slot
☐ Debug view renders in app slot (DEBUG mode only)
☐ All views derive from TuiState, never duplicate
☐ UNKNOWN values display as N/A or [? UNKNOWN]
☐ Zero values display as 0 when genuinely zero
☐ No flicker: memoized selectors + event-driven updates
☐ No polling: all updates via api.event.on()
☐ Density mode adapts format correctly
☐ Drill-down works: StatusBar → StatusDetailed → StatusDebug
☐ All metrics have authoritative source (see metrics.md)
☐ All values follow UNKNOWN != ZERO rule
☐ Colors come from theme.current, never hardcoded
☐ Every status has text + glyph (not color alone)
```

---

## 10. References

- `docs/design/density.md` — 5 density modes specification
- `docs/design/components.md` — Component library
- `docs/design/status-language.md` — Status vocabulary
- `docs/design/design-tokens.md` — Design tokens
- `docs/architecture/presentation-adapters.md` — Adapter patterns
- `docs/architecture/plugin-presentation-model.md` — Plugin presentation architecture
- `docs/state/presentation-state.md` — TuiState selectors
- `docs/architecture/tui-sdk-boundary.md` — TUI/SDK boundary
