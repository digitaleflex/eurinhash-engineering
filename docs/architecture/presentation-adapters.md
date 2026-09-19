# Presentation Adapters — Adaptateurs de présentation

> **Source** : `ARCHITECTURE_FINDINGS.md`, `plugin/dist/tui.d.ts`, `plugin/dist/index.d.ts`
> **Principe** : Les adaptateurs présentent les données du SDK dans les slots TUI sans dupliquer l'état de domaine.

---

## 1. Architecture des adaptateurs

```
┌─────────────────────────────────────────────────────────────┐
│                  TUI SLOT SYSTEM                            │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐ │
│  │  TuiSlotPlugin<Slots>                                 │ │
│  │  (SolidPlugin<TuiSlotMap, TuiSlotContext>)            │ │
│  │                                                       │ │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐              │ │
│  │  │ Slot    │  │ Slot    │  │ Slot    │              │ │
│  │  │ app     │  │ sidebar │  │ session │              │ │
│  │  │ _bottom │  │ _content│  │ _prompt │              │ │
│  │  └────┬────┘  └────┬────┘  └────┬────┘              │ │
│  │       │            │            │                     │ │
│  │       ▼            ▼            ▼                     │ │
│  │  ┌──────────────────────────────────────────────┐    │ │
│  │  │  Adapter Components (SolidJS)                │    │ │
│  │  │                                              │    │ │
│  │  │  StatusCompact ← TuiState.session.status()   │    │ │
│  │  │  StatusDetailed ← TuiState.session.get()     │    │ │
│  │  │  StatusDebug ← TuiState + Calculs            │    │ │
│  │  │  ContextMeter ← TuiState.part() + StepFinish │    │ │
│  │  │  McpStatusList ← TuiState.mcp()              │    │ │
│  │  │  LspStatusList ← TuiState.lsp()              │    │ │
│  │  │  PerformancePanel ← Calculs (tokens/elapsed) │    │ │
│  │  │  SecurityPanel → guard.ts / eurinhash        │    │ │
│  │  └──────────────────────────────────────────────┘    │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐ │
│  │  Adapter Utilities                                     │ │
│  │                                                        │ │
│  │  formatTokens(n) → "100.4K", "1.2M"                  │ │
│  │  formatDuration(ms) → "22s", "01m 43s"               │ │
│  │  formatCost(n) → "$0.00", "$1.42"                    │ │
│  │  getContextStatus(pct) → "NORMAL" | "ATTENTION" ...   │ │
│  │  getMcpLabel(status) → "CONNECTED" | "DISABLED" ...   │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Types d'adaptateurs

### 2.1 Adaptateur de session (`StatusCompact`, `StatusDetailed`, `StatusDebug`)

**Source** : `TuiState.session.get(id)`, `TuiState.session.status(id)`

```typescript
// Adaptateur : lecture TuiState → données affichées
function adaptSessionStatus(sessionID: string): SessionDisplay {
  const session = api.state.session.get(sessionID);
  const status = api.state.session.status(sessionID);
  return {
    id: session?.id ?? "",
    title: session?.title ?? "",
    directory: session?.directory ?? "",
    status: status?.type ?? "unknown",
    created: session?.time.created ?? 0,
    updated: session?.time.updated ?? 0,
    // Calculs dérivés
    elapsed: session ? session.time.updated - session.time.created : 0,
  };
}
```

**Contrat** :
- **Input** : `sessionID: string`
- **Output** : `SessionDisplay` (view-model)
- **Source** : `TuiState.session.get()`, `TuiState.session.status()`
- **Owner** : TuiPlugin
- **Lifecycle** : Lecture au rendu, recalcul sur `EventSessionStatus`

### 2.2 Adaptateur de tokens (`ContextMeter`, `PerformancePanel`)

**Source** : `TuiState.part(messageID)` + `StepFinishPart.tokens`

```typescript
// Adaptateur : lecture des Parts → calcul tokens/coût
function adaptTokens(messageID: string): TokenDisplay {
  const parts = api.state.part(messageID);
  const stepFinish = parts.find(p => p.type === "step-finish") as StepFinishPart | undefined;
  const textParts = parts.filter(p => p.type === "text") as TextPart[];
  return {
    inputTokens: stepFinish?.tokens.input ?? 0,
    outputTokens: stepFinish?.tokens.output ?? 0,
    reasoningTokens: stepFinish?.tokens.reasoning ?? 0,
    cacheRead: stepFinish?.tokens.cache.read ?? 0,
    cacheWrite: stepFinish?.tokens.cache.write ?? 0,
    cost: stepFinish?.cost ?? 0,
    contextUsed: textParts.reduce((acc, p) => acc + p.text.length, 0),
    // Calculs
    throughput: stepFinish ? calculateThroughput(stepFinish) : 0,
  };
}
```

**Contrat** :
- **Input** : `messageID: string`
- **Output** : `TokenDisplay` (view-model)
- **Source** : `TuiState.part()`, `StepFinishPart`
- **Owner** : TuiPlugin
- **Lifecycle** : Lecture au rendu, recalcul sur `EventMessagePartUpdated`

### 2.3 Adaptateur MCP (`McpStatusList`)

**Source** : `TuiState.mcp()`

```typescript
// Adaptateur : lecture TuiState.mcp() → liste status MCP
function adaptMcpStatus(): McpDisplay[] {
  const mcps = api.state.mcp();
  return mcps.map(mcp => ({
    name: mcp.name,
    status: mcp.status,
    error: mcp.error,
    label: getMcpLabel(mcp.status), // "CONNECTED" | "DISABLED" | etc.
  }));
}
```

**Contrat** :
- **Input** : Aucun
- **Output** : `McpDisplay[]`
- **Source** : `TuiState.mcp()`
- **Owner** : TuiPlugin
- **Lifecycle** : Lecture au rendu, recalcul sur `EventMcpToolsChanged`

### 2.4 Adaptateur LSP (`LspStatusList`)

**Source** : `TuiState.lsp()`

```typescript
// Adaptateur : lecture TuiState.lsp() → liste status LSP
function adaptLspStatus(): LspDisplay[] {
  const lsps = api.state.lsp();
  return lsps.map(lsp => ({
    id: lsp.id,
    root: lsp.root,
    status: lsp.status,
    label: lsp.status === "connected" ? "CONNECTED" : "ERROR",
  }));
}
```

**Contrat** :
- **Input** : Aucun
- **Output** : `LspDisplay[]`
- **Source** : `TuiState.lsp()`
- **Owner** : TuiPlugin
- **Lifecycle** : Lecture au rendu, recalcul sur `EventLspUpdated`

### 2.5 Adaptateur de thème (`ThemeAdapter`)

**Source** : `api.theme.current`, `api.theme.mode()`

```typescript
// Adaptateur : lecture thème → couleurs pour composants
function adaptTheme(): ThemeColors {
  const theme = api.theme.current;
  return {
    primary: theme.primary,
    error: theme.error,
    warning: theme.warning,
    success: theme.success,
    text: theme.text,
    background: theme.background,
    // Mapping RGBA → couleurs pour les composants
  };
}
```

**Contrat** :
- **Input** : Aucun
- **Output** : `ThemeColors`
- **Source** : `api.theme.current`
- **Owner** : TuiPlugin
- **Lifecycle** : Lecture au rendu

---

## 3. Slots d'enregistrement

### 3.1 Slots disponibles (`TuiHostSlotMap`)

| Slot | Contexte | Usage EurinHash |
|------|----------|-----------------|
| `app` | `{}` | Overlay debug (niveau 8) |
| `app_bottom` | `{}` | Status bar compact (niveau 1) |
| `home_logo` | `{}` | Logo |
| `home_prompt` | `{ ref?: TuiPromptRef }` | Prompt home |
| `home_prompt_right` | `{}` | Slot droit home |
| `home_bottom` | `{}` | Footer home |
| `home_footer` | `{}` | Status bar home |
| `sidebar_title` | `{ session_id, title, share_url? }` | Titre sidebar |
| `sidebar_content` | `{ session_id }` | Contenu sidebar |
| `sidebar_footer` | `{ session_id }` | Infos session (niveau 2-3) |
| `session_prompt` | `{ session_id, visible?, disabled?, on_submit? }` | Prompt session |
| `session_prompt_right` | `{ session_id }` | Slot droit prompt |

### 3.2 Enregistrement d'un adaptateur

```typescript
const plugin: TuiPlugin = async (api, options, meta) => {
  // Enregistrer l'adaptateur dans le slot app_bottom
  api.slots.register({
    id: "eurinhash-status-compact",
    slots: {
      app_bottom: () => <StatusCompact />,
    },
  });

  // Enregistrer l'adaptateur détaillé dans sidebar_footer
  api.slots.register({
    id: "eurinhash-status-detailed",
    slots: {
      sidebar_footer: ({ session_id }) => <StatusDetailed sessionID={session_id} />,
    },
  });
};
```

---

## 4. Formatage des adaptateurs

### 4.1 Utilities

```typescript
function formatTokens(n: number): string {
  if (n >= 1_000_000) return `${(n / 1_000_000).toFixed(1)}M`;
  if (n >= 1_000) return `${(n / 1_000).toFixed(1)}K`;
  return n.toString();
}

function formatDuration(ms: number): string {
  const seconds = Math.floor(ms / 1000);
  if (seconds < 60) return `${seconds}s`;
  const minutes = Math.floor(seconds / 60);
  const secs = seconds % 60;
  return `${String(minutes).padStart(2, "0")}m ${String(secs).padStart(2, "0")}s`;
}

function formatCost(n: number): string {
  return `$${n.toFixed(2)}`;
}

function getContextStatus(pct: number): "NORMAL" | "ATTENTION" | "HIGH" | "CRITICAL" | "LIMIT" {
  if (pct >= 100) return "LIMIT";
  if (pct >= 90) return "CRITICAL";
  if (pct >= 75) return "HIGH";
  if (pct >= 50) return "ATTENTION";
  return "NORMAL";
}

function getMcpLabel(status: string): string {
  const labels: Record<string, string> = {
    connected: "CONNECTED",
    disabled: "DISABLED",
    failed: "FAILED",
    needs_auth: "NEEDS_AUTH",
    needs_client_registration: "NEEDS_REGISTRATION",
  };
  return labels[status] ?? status.toUpperCase();
}
```

### 4.2 Contrat des utilities

- **Input** : Valeurs numériques (tokens, durée, coût, pourcentage)
- **Output** : Chaînes formatées pour l'affichage
- **Owner** : TuiPlugin
- **Lifecycle** : Calculées au rendu
- **Failure** : Valeurs par défaut, pas d'erreur

---

## 5. Hiérarchie de rendu

```
TuiPluginApi
  ├── api.slots.register(plugin)
  │     └── TuiSlotPlugin<Slots>
  │           └── SolidPlugin<TuiSlotMap, TuiSlotContext>
  │                 └── JSX.Element (composants)
  │                       └── StatusCompact / StatusDetailed / StatusDebug
  │                             └── Adaptateurs → TuiState
  │                                   └── formatTokens, formatDuration, etc.
  │
  ├── api.event.on(type, handler)
  │     └── Mise à jour du view-model
  │           └── Recalcul des composants
  │
  ├── api.ui.Slot(props)
  │     └── Rendu d'un slot enregistré
  │
  └── api.renderer (CliRenderer)
        └── Rendu terminal final
```

---

## 6. Règles des adaptateurs

1. **Pas de duplication d'état** : Les adaptateurs lisent `TuiState`, ne copient pas
2. **Calculs dérivés** : Tokens, durée, coût sont calculés à partir de `StepFinishPart`
3. **Synchronisation via événements** : `EventMessagePartUpdated` déclenche les recalculs
4. **Cleanup dans `onDispose`** : Les subscriptions événement doivent être nettoyées
5. **Type safety** : Tous les types viennent du SDK (`@opencode-ai/sdk/v2`)
6. **Thème appliqué** : Les couleurs viennent de `api.theme.current`

---

## 7. Validation des adaptateurs

```
CHECKLIST — Presentation Adapters Validation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
☐ Les adaptateurs lisent TuiState, ne copient pas les données
☐ Les calculs (tokens, coût, durée) sont dérivés de StepFinishPart
☐ Les subscriptions événement sont nettoyées dans onDispose
☐ Les types viennent du SDK (@opencode-ai/sdk/v2)
☐ Le thème est appliqué via api.theme.current
☐ Les slots sont enregistrés via api.slots.register()
☐ Les utilities de formatage sont centralisées
☐ Pas de dupliquer entre view-model et TuiState
☐ Les adaptateurs gèrent les cas undefined (données manquantes)
☐ Les labels MCP/LSP sont dérivés des statuts SDK
```
