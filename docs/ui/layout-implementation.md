# Layout Implementation — Implémentation des Layouts Command Center

> Implémentation concrète de l'architecture de layout du Command Center EurinHash OpenCode.
> Layout target : WORKSPACE | CHAT/SESSION | AGENT | ACTIVITY | COMMAND
> Basé sur le TUI Framework `@opentui` (SolidJS-based) et les slots `TuiHostSlotMap`.
> Règle fondamentale : `UNKNOWN != ZERO`

---

## 1. Architecture du Layout

### 1.1 Structure Target

```
┌──────────────┬─────────────────────────────┬──────────────┐
│ WORKSPACE    │ CHAT / SESSION              │ AGENT        │
│ files / git  │ conversation / activity     │ state/model   │
├──────────────┴─────────────────────────────┴──────────────┤
│ ACTIVITY / EVENTS / TOOL EXECUTION                         │
├───────────────────────────────────────────────────────────┤
│ COMMAND / INPUT                                            │
└───────────────────────────────────────────────────────────┘
```

### 1.2 Mapping vers les Slots TUI

| Zone Layout | Slot TUI | Composant | Propriétés |
|-------------|----------|-----------|------------|
| WORKSPACE | `sidebar_content` | `Sidebar` | `items`, `collapsed`, `width` |
| WORKSPACE | `sidebar_title` | `SidebarTitle` | `session_id`, `title` |
| WORKSPACE | `sidebar_footer` | `SidebarFooter` | `session_id` |
| CHAT/SESSION | `app` (central) | `MessageList` + `Prompt` | `messages`, `sessionID` |
| CHAT/SESSION | `session_prompt` | `Prompt` | `input`, `mode`, `parts` |
| CHAT/SESSION | `session_prompt_right` | `AgentStatusBar` | `workers`, `activeAgent` |
| AGENT | `sidebar_content` (bas) | `AgentStatusBar` | `workers[]` |
| ACTIVITY | `app_bottom` | `EventLog` | `events`, `filter` |
| COMMAND | `session_prompt` | `Prompt` | `input`, `mode` |
| COMMAND | `app_bottom` | `CommandSurface` | `commands[]` |

### 1.3 Hiérarchie de Rendu

```
TuiPluginApi
  ├── api.slots.register(plugin)
  │     └── TuiSlotPlugin<Slots>
  │           └── SolidPlugin<TuiSlotMap, TuiSlotContext>
  │                 └── JSX.Element (composants)
  │                       ├── Workspace (sidebar_content)
  │                       │     ├── Sidebar
  │                       │     ├── StatusIndicator
  │                       │     ├── Table
  │                       │     └── KeyValueList
  │                       ├── Chat (app central)
  │                       │     ├── MessageList
  │                       │     ├── TextBlock
  │                       │     ├── ThinkingBlock
  │                       │     ├── ToolCallCard
  │                       │     ├── ProgressBar
  │                       │     ├── DiffView
  │                       │     └── MarkdownView
  │                       ├── Agent (sidebar bas)
  │                       │     ├── AgentStatusBar
  │                       │     ├── MetricDisplay
  │                       │     └── ModelSelector
  │                       ├── Activity (app_bottom)
  │                       │     ├── EventLog
  │                       │     ├── FilterBar
  │                       │     └── Caption
  │                       └── Command (session_prompt)
  │                             ├── Prompt
  │                             ├── CommandSurface
  │                             └── KeybindDisplay
  └── api.event.on(type, handler)
        └── Mise à jour des composants
              └── Recalcul via TuiState
```

---

## 2. Implémentation par Zone

### 2.1 WORKSPACE Layout

**Position** : Sidebar gauche
**Slot** : `sidebar_content`, `sidebar_title`, `sidebar_footer`
**Largeur** : 20–35 colonnes

```
┌──────────────────────┐
│ [●] eurinhash        │ ← sidebar_title
├──────────────────────┤
│ 📁 src/              │
│ 📄 main.tsx          │
│ 📁 tests/            │
│ ─────────────────    │
│ [✓] main branch      │ ← StatusIndicator
├──────────────────────┤
│ 📊 3 files modified  │ ← Table
│ 📋 2 todos           │ ← KeyValueList
├──────────────────────┤
│ [●] MCP: agentdeals  │
│ [✗] MCP: postgres    │
│ [✓] LSP: typescript  │
└──────────────────────┘
```

**Implémentation** :

- `Sidebar` lit `TuiState.vcs`, `TuiState.session.diff()`, `TuiState.session.todo()`, `TuiState.mcp()`, `TuiState.lsp()`
- `StatusIndicator` construit avec `border`, `backgroundPanel`, `text`, `textMuted`
- `Table` adapte ses colonnes par largeur (≥120 : complète, 100–119 : compacte, 80–99 : liste clé-valeur, <80 : JSON lines)
- `Panel` construit avec `border`, `backgroundPanel`, `corner` characters (`┌┐└┘`), `space-md` padding

**Règles de largeur** :

| Largeur | Sidebar | Table | Panels |
|---------|---------|-------|--------|
| ≥ 120 | Pleine (30+ cols) | Table complète | Coins + bordures complètes |
| 100–119 | Standard (25 cols) | Table compacte | Bordures simples |
| 80–99 | Icônes uniquement | Liste clé-valeur | Bordure supérieure |
| < 80 | Masquée | JSON lines | Ligne de séparation |

---

### 2.2 CHAT / SESSION Layout

**Position** : Zone centrale (la plus grande surface)
**Slot** : `app`, `session_prompt`, `session_prompt_right`
**Largeur** : 60+ colonnes

```
┌──────────────────────────────────────────────────────────────┐
│ [◐ THINKING] worker-novita · 2.3s                            │ ← StatusCompact
├──────────────────────────────────────────────────────────────┤
│ [● ACTIVE]  eurinhash-agent · 38% · 100K/262K · $0.00       │ ← Caption
│                                                            │
│ User: Comment je...                                        │ ← TextBlock
│                                                            │
│ [◐ THINKING] Analy...                                      │ ← ThinkingBlock
│                                                            │
│ ▶ RUNNING: tool: bash                                      │ ← ToolCallCard
│   Input: ls -la                                            │
│   Output: file.txt                                           │
│                                                            │
│ [████████░░] 38% · 100K/262K                               │ ← ProgressBar
│                                                            │
│ ┌──────────────────────────────────────────────────────┐   │
│ │ + line added (diffAdded)                             │   │ ← DiffView
│ │ - line removed (diffRemoved)                         │   │
│ └──────────────────────────────────────────────────────┘   │
├──────────────────────────────────────────────────────────────┤
│ › _ /density STANDARD                                        │ ← Prompt
└──────────────────────────────────────────────────────────────┘
```

**Implémentation** :

- `MessageList` lit `TuiState.session.messages(id)` et `TuiState.part(messageID)`
- `TextBlock` rend les `TextPart` avec `text` token
- `ThinkingBlock` rend les `ReasoningPart` avec `thinkingOpacity` appliqué
- `ToolCallCard` rend les `ToolPart` avec `borderActive` pour le focus
- `ProgressBar` construit avec `primary` pour le segment rempli, `backgroundElement` pour le vide
- `DiffView` utilise `diffAdded`, `diffRemoved`, `diffContext`, `diffHunkHeader` tokens
- `Prompt` construit avec `primary` pour le curseur, `text` pour la saisie

**Règles de rendu** :

- `thinkingOpacity` est appliqué aux `ThinkingBlock` et aux éléments `secondary` en état THINKING
- Les deltas sont rendus incrémentalement via `message.part.delta`
- `EventMessagePartUpdated` déclenche les mises à jour sans full-tree rerender
- Chaque partie (text, reasoning, tool) est rendue séparément

---

### 2.3 AGENT Layout

**Position** : Sidebar bas ou barre de statut
**Slot** : `sidebar_footer`, `app_bottom`
**Largeur** : 40+ colonnes

```
┌──────────────────────────────────────────────────────────────┐
│ [● ACTIVE] worker-codestral · 1183ms │ [● ACTIVE] worker-groq │
│ [◐ THINKING] worker-novita · 1223ms  │ [✗ ERROR] worker-zhipu │
│ [● ACTIVE] worker-pollinations · 630ms│ [? UNKNOWN] worker-ollama│
└──────────────────────────────────────────────────────────────┘
```

**Implémentation** :

- `AgentStatusBar` lit `TuiState.mcp()`, `TuiState.lsp()`, et `free-models.json`
- `StatusIndicator` construit pour chaque worker avec le statut approprié
- `MetricDisplay` formaté via `formatTokens()`, `formatDuration()`, `formatCost()`
- `ModelSelector` lit `TuiState.provider`, `Session.model`

**Règles UNKNOWN pour les agents** :

| Cas | Affichage correct | Affichage interdit |
|-----|-------------------|-------------------|
| Worker non interrogé | `[? UNKNOWN]` | `[○ IDLE]` |
| Worker rate_limited | `[⚠ WARNING]` | `[● ACTIVE]` |
| Worker error | `[✗ ERROR]` | `[○ IDLE]` |
| Coût non calculé | `N/A` | `$0.00` |
| Latence inconnue | `N/A` | `0ms` |

**Fallback de routing** : `codestral → groq → novita → zhipu → google → ollama → mammouth(payant)`

---

### 2.4 ACTIVITY Layout

**Position** : Panneau déroulant sous la zone CHAT
**Slot** : `app_bottom`, `sidebar_footer`
**Largeur** : 40+ colonnes

```
┌──────────────────────────────────────────────────────────────┐
│ ACTIVITY                                        [Filter ▼]   │
├──────────────────────────────────────────────────────────────┤
│ [12:00:01] [tool] ▶ RUNNING tool: bash          │ [FILTERS] │
│ [12:00:02] [session] ● ACTIVE session: abc123    │           │
│ [12:00:03] [tool] ✗ ERROR tool: read          │           │
│ [12:00:04] [mcp] ◐ THINKING MCP: agentdeals    │           │
│ [12:00:05] [permission] ⌛ WAITING permission    │           │
│ [12:00:06] [lsp] ✓ SUCCESS lsp: typescript     │           │
└──────────────────────────────────────────────────────────────┘
```

**Implémentation** :

- `EventLog` lit tous les événements via `TuiEventBus.on()`
- `FilterBar` permet le filtrage par : session, agent, tool, MCP, LSP, severity
- `EventItem` construit avec `border`, `backgroundPanel`, `text`, `textMuted`
- `Caption` construit avec `textMuted` + `italic` + `mono` pour timestamps
- `Separator` construit avec `borderSubtle` pour la séparation temporelle
- `Badge` construit avec le glyphe et la couleur du statut

**Filtrage par catégorie** :

| Filtre | Valeurs | Source |
|--------|---------|--------|
| Session | Tous types `EventSession*` | `EventSession*` |
| Agent | `EventSessionNext*`, worker events | `EventSessionNext*` |
| Tool | `EventSessionNextTool*`, `tool.*` | `EventSessionNextTool*` |
| MCP | `EventMcp*`, `server.connected` | `EventMcp*` |
| LSP | `EventLsp*`, `lsp.client.diagnostics` | `EventLsp*` |
| Severity | `info`, `success`, `warning`, `error` | Dérivé du type d'événement |

**Groupement** :

| Groupement | Description |
|------------|-------------|
| `type` | Par catégorie d'événement |
| `session` | Par session ID |
| `agent` | Par agent/worker |
| `severity` | Par niveau de sévérité |
| `time` | Par plage temporelle |

---

### 2.5 COMMAND Layout

**Position** : En bas du terminal, toujours visible
**Slot** : `session_prompt`, `session_prompt_right`, `app_bottom`
**Largeur** : 20+ colonnes

```
┌──────────────────────────────────────────────────────────────┐
│ › _ /density STANDARD                                        │ ← Prompt (normal mode)
│ $ _ ls -la                                                   │ ← Prompt (shell mode) |
├──────────────────────────────────────────────────────────────┤
│ COMMANDS                                                     │ ← CommandSurface
│ ──────────────                                               │
│   /density STANDARD   Change density level                  │
│   /terminal info      Show terminal info                    │
│   /theme dark         Switch to dark mode                   │
│ ──────────────                                               │
│ Ctrl+D → Change density  / → Command palette               │ ← KeybindDisplay
└──────────────────────────────────────────────────────────────┘
```

**Implémentation** :

- `Prompt` construit avec `primary` pour le curseur `›` ou `$`, `text` pour la saisie, `textMuted` pour le placeholder
- `CommandSurface` construit avec `backgroundMenu` pour le fond, `border` pour les bordures, `accent` pour la commande active
- `KeybindDisplay` construit avec `mono` pour les touches, `textMuted` pour les descriptions
- `Dialog` construit avec `borderActive` pour le focus, `backgroundMenu` pour le fond

**Modes de prompt** :

| Mode | Indicateur | Usage | Couleur |
|------|-----------|-------|---------|
| Normal | `›` | Commande standard, agent | `primary` |
| Shell | `$` | Commande shell | `mono` |

**Règle UNKNOWN pour le mode** : `TuiPromptInfo.mode: undefined` ≠ `"normal"`. Quand le mode n'est pas spécifié, il est `undefined` — jamais implicitement `"normal"`.

---

## 3. Implémentation des Slots

### 3.1 Enregistrement des Slots

```typescript
const plugin: TuiPlugin = async (api, options, meta) => {
  // WORKSPACE - Sidebar content
  api.slots.register({
    id: "eurinhash-workspace",
    slots: {
      sidebar_content: ({ session_id }) => <WorkspaceSession sessionID={session_id} />,
      sidebar_title: ({ session_id, title, share_url }) => <SidebarTitle ... />,
      sidebar_footer: ({ session_id }) => <SidebarFooter sessionID={session_id} />,
    },
  });

  // CHAT/SESSION - Main area
  api.slots.register({
    id: "eurinhash-chat",
    slots: {
      app: () => <ChatCenter />,
      session_prompt: ({ session_id, visible, disabled, on_submit, ref }) => (
        <Prompt sessionID={session_id} ref={ref} ... />
      ),
      session_prompt_right: ({ session_id }) => <AgentStatusBar sessionID={session_id} />,
    },
  });

  // ACTIVITY - Bottom panel
  api.slots.register({
    id: "eurinhash-activity",
    slots: {
      app_bottom: () => <EventLog />,
    },
  });

  // COMMAND - Prompt area
  api.slots.register({
    id: "eurinhash-command",
    slots: {
      session_prompt: ({ session_id, visible, disabled, on_submit, ref }) => (
        <CommandPrompt sessionID={session_id} ref={ref} ... />
      ),
    },
  });
};
```

### 3.2 Adaptateurs par Zone

| Zone | Adaptateur | Source | Output |
|------|-----------|--------|--------|
| WORKSPACE | `StatusCompact`, `StatusDetailed` | `TuiState.session.status()`, `TuiState.session.get()` | `SessionDisplay` |
| CHAT/SESSION | `ContextMeter`, `PerformancePanel` | `TuiState.part()`, `StepFinishPart` | `TokenDisplay` |
| AGENT | `McpStatusList`, `LspStatusList` | `TuiState.mcp()`, `TuiState.lsp()` | `McpDisplay[]`, `LspDisplay[]` |
| ACTIVITY | `EventLog` | `TuiEventBus.on()` | `Event[]` |
| COMMAND | `Prompt` | `TuiPromptInfo` | `DisplayMessage[]` |

---

## 4. Gestion du Resize

### 4.1 Détection de Resize

```
Sur événement resize du terminal :
1. Mesurer la nouvelle largeur
2. Recalculer le layout
3. Redessiner tous les composants
4. Vérifier qu'aucun texte n'est tronqué de manière abusive
```

### 4.2 Seuil de Transition

| Largeur | Niveau | Action |
|---------|--------|--------|
| < 80 | MINIMAL | Sidebar masquée, layout réduit |
| 80–99 | DEVELOPER | Pas de lignes vides, padding réduit |
| 100–119 | STANDARD | Layout standard |
| 120–159 | OBSERVER | Layout large, métriques détaillées |
| ≥ 160 | OBSERVER/STANDARD | Layout étendu (choix utilisateur) |

### 4.3 Règles de Resize

1. **Pas de flicker** — le resize DOIT être fluide
2. **Pas de perte de données** — le contenu ne doit pas disparaître
3. **Rétrogradation progressive** — passer de 160 à 80 cols en étapes
4. `thinkingOpacity` ne change pas lors d'un resize
5. Les statuts restent lisibles après resize
6. `UNKNOWN` reste `UNKNOWN` après resize

---

## 5. Composants Réutilisés vs Remplacés

### 5.1 Réutilisés

| Composant | Source | Pourquoi réutilisé |
|-----------|--------|-------------------|
| `StatusIndicator` | `design/components.md` | Déjà défini, respecte le système de statuts |
| `ProgressBar` | `design/components.md` | Déjà adapté par largeur |
| `Table` | `design/components.md` | Déjà adapté par largeur |
| `Panel` | `design/components.md` | Déjà défini avec coins/bordures |
| `Alert` | `design/components.md` | Déjà défini avec niveaux |
| `Dialog` | `design/components.md` | Déjà défini dans TuiPluginApi |
| `Prompt` | `design/components.md` | Déjà défini dans TuiPluginApi |
| `Sidebar` | `design/components.md` | Adapté pour Command Center |
| `Toast` | `design/components.md` | Déjà défini dans TuiPluginApi |
| `Spinner` | `design/components.md` | Déjà défini |
| `Badge` | `design/components.md` | Déjà défini |
| `Separator` | `design/components.md` | Déjà défini |
| `Caption` | `design/components.md` | Déjà défini |
| `CommandSurface` | `design/components.md` | Déjà défini |
| `KeybindDisplay` | `design/components.md` | Déjà défini |

### 5.2 Remplacés

| Composant | Ancien | Nouveau | Raison |
|---------|--------|---------|--------|
| `EventLog` | — | Nouveau | Spécifique à l'Activity Center |
| `FilterBar` | — | Nouveau | Spécifique au filtrage d'activité |
| `AgentStatusBar` | `StatusBar` | Nouveau | Agrégation des workers avec métriques |
| `MetricDisplay` | `MetricDisplay` | Adapté | Formatage spécifique worker |
| `ModelSelector` | — | Nouveau | Sélection de modèle pour agents |

### 5.3 Enveloppés (Wrapped)

| Composant | Wrap | Raison |
|---------|------|--------|
| `Sidebar` | Wrap avec `collapsed` state | Permettre le pliage/dépliage |
| `Prompt` | Wrap avec `mode` toggle | Normal/Shell |
| `Table` | Wrap avec `filter` state | Filtrage d'activité |

---

## 6. Règles de Performance

### 6.1 Éviter les Full-Tree Rerenders

1. `message.part.updated` → mise à jour partielle uniquement (delta rendering)
2. `TuiEventBus.on(type, handler)` → abonnement ciblé
3. `api.state.part(messageID)` → lecture sélective
4. Cleanup dans `onDispose` → pas de fuite mémoire

### 6.2 Stratégie de Rendu Incremental

```
EventMessagePartDelta → TextRenderer (delta only)
EventMessagePartUpdated → PartCard (re-render part)
EventSessionStatus → StatusCompact (re-render status bar only)
EventMcpToolsChanged → McpStatusList (re-render MCP only)
EventLspUpdated → LspStatusList (re-render LSP only)
```

### 6.3 Mémoire et Cleanup

- `api.lifecycle.signal` → signal d'arrêt
- `api.lifecycle.onDispose(fn)` → callbacks de cleanup
- `api.event.on(type, handler)` → retourne une fonction d'unsubscribe
- `TuiState` est `readonly` → pas de mutation accidentelle

---

## 7. Validation du Layout

### 7.1 Checklist de Validation

```
CHECKLIST — Layout Implementation Validation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
☐ WORKSPACE visible dans sidebar_content
☐ CHAT/SESSION occupe la zone centrale maximale
☐ AGENT visible dans sidebar_footer ou app_bottom
☐ ACTIVITY déroulant sous la zone CHAT
☐ COMMAND fixe en bas (session_prompt)
☐ TOUS les composants lisent theme.current
☐ UNKNOWN != ZERO respecté dans TOUS les composants
☐ Pas de full-tree rerender — mises à jour incrémentales
☐ Fallback 80 colonnes pour TOUS les composants
☐ thinkingOpacity appliqué uniquement à THINKING
☐ Les métriques sont en mono
☐ Les glyphes ont un fallback textuel
☐ Mouse support préservé
☐ Keybindings préservées et découvrables
☐ Dialogs et command routing préservés
```

---

## 8. Résumé

| Zone | Slot | Composants | Largeur min | Priorité |
|------|------|-----------|-------------|----------|
| WORKSPACE | `sidebar_content` | Sidebar, StatusIndicator, Table, KeyValueList, Badge | 20 cols | HAUTE |
| CHAT/SESSION | `app` | MessageList, TextBlock, ThinkingBlock, ToolCallCard, ProgressBar, DiffView | 60 cols | MAXIMUM |
| AGENT | `sidebar_footer` | AgentStatusBar, MetricDisplay, ModelSelector | 40 cols | MOYENNE-HAUTE |
| ACTIVITY | `app_bottom` | EventLog, FilterBar, Caption, Badge | 40 cols | MOYENNE |
| COMMAND | `session_prompt` | Prompt, CommandSurface, KeybindDisplay | 20 cols | FIXE (bas) |