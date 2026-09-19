# Observability Performance — Rendering Performance

> **Source**: `docs/design/density.md`, `docs/architecture/presentation-adapters.md`, `docs/architecture/state-contracts.md`
> **Rule**: Prefer memoized/derived selectors, event-driven updates and bounded rendering. Do not poll aggressively if the runtime already emits events.

---

## 1. Performance Principles

### 1.1 Core Rules

1. **No polling**: The runtime emits events — use them. Never poll for state changes.
2. **Event-driven updates**: Subscribe to `TuiEventBus` via `api.event.on()`. Updates happen only when events fire.
3. **Memoized selectors**: Derived values (tokens, cost, duration, percentage) are computed once and cached until source data changes.
4. **Bounded rendering**: Each event triggers a bounded number of DOM operations.
5. **No flicker**: Continuously changing values must not cause terminal flicker.

### 1.2 Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Event → re-render latency | < 16ms | Per frame at 60fps |
| Selector computation | < 1ms | Per selector |
| Full observability update | < 50ms | All domains |
| Memory per session | < 1MB | View-model per session |
| Event subscription overhead | Negligible | Per subscription |

---

## 2. Event-Driven Architecture

### 2.1 Subscription Pattern

```typescript
// Subscribe ONCE at plugin initialization
const unsubscribe = api.event.on("session.status", (event) => {
    // Recalculate only SESSION domain
    recalculateSession(event.sessionID);
});

// Cleanup on dispose
api.lifecycle.onDispose(() => {
    unsubscribe();
});
```

### 2.2 Event-to-View-Model Mapping

| Event | View-Model Recalculation | Domains Affected |
|-------|--------------------------|------------------|
| `session.status` | Recalculate `SessionStatus` | SESSION, ACTIVITY |
| `session.next.model.switched` | Recalculate model info | MODEL |
| `session.next.context.updated` | Recalculate context usage | CONTEXT |
| `session.next.step.ended` | Recalculate tokens, cost, performance | TOKENS, COST, PERFORMANCE |
| `session.next.tool.called` | Recalculate current tool | TOOLS |
| `session.next.tool.progress` | Update tool progress | TOOLS |
| `session.next.tool.success` | Update tool result | TOOLS |
| `session.next.tool.failed` | Update tool error | TOOLS |
| `message.part.updated` | Recalculate tokens from parts | TOKENS, CONTEXT |
| `permission.asked/replied` | Recalculate permission state | SECURITY |
| `lsp.updated` | Recalculate LSP status | TOOLS |
| `mcp.tools.changed` | Recalculate MCP status | TOOLS |
| `session.idle` | Update session phase | SESSION, ACTIVITY |
| `session.compacted` | Update compaction state | SESSION |

### 2.3 No-Polling Guarantee

All observability data is obtained via:
- `api.state.*` — Read-only selectors (called at render time, not polled)
- `api.event.on()` — Event subscriptions (push-based)
- `api.lifecycle.onDispose()` — Cleanup

**Never**: `setInterval()`, `setTimeout()` for state polling, periodic refresh loops.

---

## 3. Memoized Selectors

### 3.1 Selector Composition

All observability view-model properties are derived from `TuiState` via memoized selectors.

```typescript
// Pattern 1: Direct selector
const sessionStatus = (state: TuiState, sessionID: string) =>
    state.session.status(sessionID);  // Memoized by TuiState

// Pattern 2: Derived selector
const contextPercentage = (state: TuiState, sessionID: string, messageID: string) => {
    const stepFinish = findStepFinish(api.state.part(messageID));
    const model = api.state.session.get(sessionID)?.model;
    if (!stepFinish || !model?.limit?.context) return undefined;
    return (sumTokens(stepFinish.tokens) / model.limit.context) * 100;
};

// Pattern 3: Composed selector
const sessionOverview = (state: TuiState, sessionID: string) => ({
    session: state.session.get(sessionID),
    status: state.session.status(sessionID),
    messages: state.session.messages(sessionID),
    // All derived from TuiState, never stored separately
});
```

### 3.2 Selector Cache Rules

1. Selectors return new objects/arrays each call (immutability)
2. The TUI framework handles memoization at the component level
3. View-model calculations are recomputed only when relevant event data changes
4. No redundant calculations across domains

### 3.3 Derivation vs. Storage

| Approach | Rule |
|----------|------|
| **Derived** | Tokens, cost, duration, percentage — calculated from `StepFinishPart` at render time |
| **Stored** | Never store derived values separately — always derive from `TuiState` |
| **Source of truth** | `StepFinishPart.tokens`, `StepFinishPart.cost` — these are the authoritative values |

---

## 4. Bounded Rendering

### 4.1 Render Bounds

| Scenario | Max DOM Operations | Time Budget |
|----------|-------------------|-------------|
| Single event update | Number of affected components | < 16ms |
| Batch event update (same loop) | All affected components, once | < 50ms |
| Full re-render (mode switch) | All observability components | < 100ms |
| Status bar update | 1 per indicator | < 5ms |

### 4.2 Update Batching

Multiple events in the same event loop are batched into a single re-render:

```typescript
// All events processed in one tick
api.event.on("session.next.step.ended", (event) => {
    batchUpdate(() => {
        recalculateTokens(event);
        recalculateCost(event);
        recalculatePerformance(event);
    });
});
```

### 4.3 Flicker Prevention

| Technique | Implementation |
|-----------|---------------|
| Memoized components | Only re-render when props change |
| Stable keys | Component keys based on IDs, not indices |
| Batched updates | All state changes in one render pass |
| Throttled display | Rapidly changing values (e.g., tokens) update at fixed intervals, not per event |
| Layout stability | Component positions don't shift on value changes |

---

## 5. Mode-Specific Performance

### 5.1 Rendering Cost by Mode

| Mode | Components Rendered | Update Frequency | Performance Cost |
|------|-------------------|-----------------|-----------------|
| MINIMAL | StatusBar only | On event | Lowest |
| STANDARD | StatusBar + StatusDetailed (on drill-down) | On event | Low |
| DEVELOPER | All inline components | On event | Medium |
| OBSERVER | All components, max metrics | On event + throttled | Medium-High |
| DEBUG | All components, raw data | On event + potentially high | Highest |

### 5.2 Optimization Per Mode

| Mode | Optimization |
|------|-------------|
| MINIMAL | Single `StatusBar` subscription; minimal selectors |
| STANDARD | Lazy-loaded `StatusDetailed`; only subscribe to active session events |
| DEVELOPER | Inline rendering; fewer subscriptions, more computation per event |
| OBSERVER | Throttled updates for rapidly changing values (tokens, latency) |
| DEBUG | Raw data display; potentially higher update frequency accepted |

### 5.3 Density Transition Rules

- Transitions between modes are progressive, never abrupt
- Mode change triggers a single re-render, not gradual updates
- Component selection changes, not position shifts
- `thinkingOpacity` is applied regardless of mode with zero performance cost

---

## 6. Adapter Performance

### 6.1 Adapter Computation Cost

| Adapter | Source | Computation | Cost |
|---------|--------|-------------|------|
| `adaptSessionStatus` | `TuiState.session.get()`, `.status()` | Simple property read | O(1) |
| `adaptTokens` | `TuiState.part()`, `StepFinishPart` | Find part + sum tokens | O(n) where n = parts count |
| `adaptMcpStatus` | `TuiState.mcp()` | Map over MCP list | O(m) where m = MCP count |
| `adaptLspStatus` | `TuiState.lsp()` | Map over LSP list | O(l) where l = LSP count |
| `adaptTheme` | `api.theme.current` | Direct read | O(1) |
| `getContextStatus` | Percentage | Comparison chain | O(1) |
| `formatTokens` | Number | Division + formatting | O(1) |
| `formatDuration` | Number | Division + padding | O(1) |
| `formatCost` | Number | String formatting | O(1) |

### 6.2 Adapter Caching Strategy

1. Adapters are pure functions — no side effects
2. Results are memoized at the component level
3. Re-computation happens only when source event data changes
4. Formatting utilities are stateless and re-entrant

### 6.3 Event → Adapter → Component Flow

```
Event fires
  │
  ▼
Batch update triggered
  │
  ▼
Relevant adapters recalculated (only those affected by the event)
  │
  ▼
Components receive new props (via memoized selectors)
  │
  ▼
Only changed components re-render (SolidJS fine-grained reactivity)
  │
  ▼
Terminal output updated (bounded)
```

---

## 7. Memory Management

### 7.1 Memory Budget

| Component | Memory Limit | Cleanup |
|-----------|-------------|---------|
| View-model per session | < 1MB | On session delete |
| Event subscriptions | Per subscription | `unsubscribe()` on `onDispose` |
| Component state | Per component | Cleanup via SolidJS lifecycle |
| Cached selectors | Per selector | Invalidated on event |

### 7.2 Cleanup Protocol

```typescript
// In TuiPlugin entry point
const plugin: TuiPlugin = async (api, options, meta) => {
    const unsub1 = api.event.on("session.status", handler1);
    const unsub2 = api.event.on("message.part.updated", handler2);
    const unsub3 = api.event.on("lsp.updated", handler3);

    api.lifecycle.onDispose(() => {
        unsub1();
        unsub2();
        unsub3();
        // All cached selectors invalidated automatically
    });
};
```

### 7.3 Memory Leak Prevention

1. Every `api.event.on()` must have a corresponding `unsubscribe()` in `onDispose`
2. No closures that capture `TuiState` references indefinitely
3. View-models are recomputed, not accumulated
4. No global caches that grow unbounded

---

## 8. No-Polling Enforcement

### 8.1 Forbidden Patterns

```typescript
// ❌ FORBIDDEN: Polling for state
setInterval(() => {
    const status = api.state.session.status(sessionID);
}, 1000);

// ❌ FORBIDDEN: Periodic refresh loop
setTimeout(() => {
    refreshAllMetrics();
}, 5000);

// ❌ FORBIDDEN: Manual state polling
const interval = setInterval(refreshMetrics, 100);
```

### 8.2 Required Patterns

```typescript
// ✅ REQUIRED: Event-driven subscription
api.event.on("session.status", (event) => {
    updateStatus(event.status);
});

// ✅ REQUIRED: Read-only selectors at render time
const tokens = api.state.part(messageID)?.find(p => p.type === "step-finish")?.tokens;

// ✅ REQUIRED: Cleanup
api.lifecycle.onDispose(() => { unsubscribe(); });
```

### 8.3 Exception: `scripts/free-probe.py`

The only polling mechanism in the system is `scripts/free-probe.py`, which refreshes worker status every 30 seconds. This is acceptable because:
1. It runs outside the TUI rendering path
2. It refreshes worker availability, not observability metrics
3. It operates at a low frequency (30s interval)
4. It feeds `free-models.json`, which is read by the routing layer, not the observability layer

---

## 9. Performance Validation

### 9.1 Validation Checklist

```
CHECKLIST — Performance Validation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
☐ No setInterval/setTimeout for state polling
☐ All updates via api.event.on()
☐ All subscriptions cleaned in api.lifecycle.onDispose()
☐ Selectors are memoized (re-computed only on relevant events)
☐ Derived values (tokens, cost, duration) computed from StepFinishPart
☐ No duplicate state storage (view-model derived from TuiState)
☐ Batch updates for multiple events in same loop
☐ Flicker-free: memoized components + stable keys
☐ Memory bounded: < 1MB per session for view-model
☐ Event → re-render latency < 16ms
☐ Full observability update < 50ms
☐ Cleanup on plugin dispose verified
☐ free-probe.py runs outside TUI rendering path
```

### 9.2 Performance Regression Detection

| Signal | Detection | Action |
|--------|-----------|--------|
| Increasing render time | Measure event → re-render latency | Check selector complexity |
| Memory growth | Profile heap per session | Check for missing cleanup |
| Terminal flicker | Visual inspection | Check memoization and batching |
| High CPU | Profile event handler cost | Check adapter computation |
| Stale data | Verify event → view-model mapping | Check subscription correctness |

---

## 10. References

- `docs/architecture/presentation-adapters.md` — Adapter architecture
- `docs/architecture/state-contracts.md` — State contract rules
- `docs/design/density.md` — Density modes and performance characteristics
- `docs/architecture/plugin-presentation-model.md` — Plugin lifecycle and cleanup
- `docs/architecture/tui-sdk-boundary.md` — TUI/SDK boundary
- `docs/architecture/event-contracts.md` — Event types and contracts
- `docs/state/event-catalog.md` — Full event catalog
