# Component Map — Carte des Composants UI Command Center

> Carte exhaustive des composants UI du Command Center EurinHash OpenCode.
> Chaque composant est construit à partir des design tokens `TuiThemeCurrent` et réutilise les primitives `@opentui`.
> Layout target : WORKSPACE | CHAT/SESSION | AGENT | ACTIVITY | COMMAND
> Règle fondamentale : `UNKNOWN != ZERO`

---

## 1. Architecture des Composants

### 1.1 Principes de Construction

- Chaque composant lit ses couleurs depuis `theme.current` (`TuiThemeCurrent`)
- Aucun style hardcodé dans la logique métier
- Chaque composant est composé de : **glyphe + libellé + couleur + métrique**
- `thinkingOpacity` est appliqué par le composant quand le contexte le justifie (état THINKING)
- Les composants sont indépendants du thème (dark/light)
- `UNKNOWN != ZERO` : jamais afficher `0` ou `""` pour une donnée inconnue

### 1.2 Composition de Base

```
[GLYCHE] LIBELLÉ · DONNÉES · MÉTRIQUES
```

Les composants ne contiennent jamais de couleur arbitraire — uniquement les tokens `TuiThemeCurrent`.

---

## 2. Composants par Zone de Layout

### 2.1 WORKSPACE

| Composant | Propriétés | Source | Rôle |
|-----------|-----------|--------|------|
| `Sidebar` | `items[]`, `collapsed`, `width` | `TuiState.vcs`, `TuiState.session.diff()`, `TuiState.session.todo()` | Navigation latérale fichiers/git/todos |
| `StatusIndicator` | `status`, `label`, `metric?`, `thinkingOpacity?` | `SessionStatus`, `WorkspaceInfo` | Affiche le statut workspace |
| `Badge` | `prefix`, `label`, `color` | `VcsFileStatus`, `WorkspaceInfo` | Marqueur textuel de statut VCS |
| `Table` | `columns[]`, `rows[]`, `width` | `session.diff(id)` → `TuiSidebarFileItem[]` | Tableau des fichiers modifiés |
| `KeyValueList` | `items[]` | `WorkspaceInfo` | Liste clé-valeur des infos projet |
| `Panel` | `title`, `children`, `width` | `TuiState` | Conteneur de contenu avec bordures |
| `Alert` | `level`, `title`, `message` | `EventWorkspaceStatus`, `EventWorkspaceFailed` | Alerte workspace |

**Propriétés détaillées — Sidebar :**

| Propriété | Type | Description |
|-----------|------|-------------|
| `items` | `SidebarItem[]` | Liste des items (sessions, agents, MCP, LSP) |
| `collapsed` | `boolean` | Si true, sidebar réduite (icônes uniquement) |
| `width` | `number` | Largeur en colonnes (20–35) |
| `activeId` | `string` | ID de l'item sélectionné |

**Propriétés détaillées — StatusIndicator :**

| Propriété | Type | Description |
|-----------|------|-------------|
| `status` | `Status enum` | L'un des 11 statuts (ACTIVE, IDLE, THINKING, etc.) |
| `label` | `string` | Nom de l'agent/service |
| `metric` | `string` | Métrique optionnelle |
| `thinkingOpacity` | `number` | Appliqué si THINKING |

---

### 2.2 CHAT / SESSION

| Composant | Propriétés | Source | Rôle |
|-----------|-----------|--------|------|
| `MessageList` | `messages[]`, `sessionID` | `session.messages(id)` | Liste des messages de conversation |
| `TextBlock` | `text`, `delta?`, `synthetic?` | `TextPart` | Rendu du texte streaming |
| `ThinkingBlock` | `text`, `time` | `ReasoningPart` | Rendu du reasoning avec `thinkingOpacity` |
| `ToolCallCard` | `tool`, `state`, `input`, `output?` | `ToolPart` | Carte d'appel d'outil |
| `ProgressBar` | `percent`, `limit?`, `tokens?` | `StepFinishPart.tokens` | Barre de progression |
| `DiffView` | `diffs: FileDiff[]` | `session.diff(id)` | Affichage de diff |
| `MarkdownView` | `content` | `TextPart` (markdown) | Rendu markdown en terminal |
| `SyntaxView` | `code`, `language` | `TextPart` | Coloration syntaxique |
| `Spinner` | `label?`, `state` | `ToolStateRunning` | Indicateur de chargement |
| `Alert` | `level`, `title`, `message` | `EventSessionError`, `EventSessionStatus` | Alerte d'erreur/warning |
| `Toast` | `title?`, `message`, `variant`, `duration?` | `EventTuiToastShow` | Notification éphémère |
| `Dialog` | `title`, `message`, `options?`, `size` | `TuiDialogStack` | Dialogue modale |
| `Prompt` | `input`, `mode?`, `parts` | `TuiPromptInfo` | Zone de saisie utilisateur |
| `SelectionList` | `items[]`, `selectedIndex` | `PermissionRequest[]`, `QuestionRequest[]` | Liste sélectionnable |
| `Caption` | `text`, `timestamp` | `Part.time` | Texte de métadonnée |
| `Separator` | `type` | — | Séparateur horizontal/vertical |
| `Badge` | `prefix`, `label`, `color` | `SessionStatus` | Badge de statut session |

**Propriétés détaillées — MessageList :**

| Propriété | Type | Description |
|-----------|------|-------------|
| `messages` | `Message[]` | Messages de la session |
| `sessionID` | `string` | ID de la session |
| `scrollPosition` | `number` | Position de scroll actuelle |
| `showMetadata` | `boolean` | Afficher les métadonnées |

**Propriétés détaillées — ToolCallCard :**

| Propriété | Type | Description |
|-----------|------|-------------|
| `tool` | `string` | Nom de l'outil |
| `state` | `ToolState` | `pending` \| `running` \| `completed` \| `error` |
| `input` | `{ [key: string]: unknown }` | Entrée de l'outil |
| `output` | `string` | Sortie (uniquement en `completed`) |
| `error` | `string` | Message d'erreur (uniquement en `error`) |
| `time.start` | `number` | Timestamp de début |
| `time.end` | `number \| undefined` | Timestamp de fin |

**Propriétés détaillées — ProgressBar :**

| Propriété | Type | Description |
|-----------|------|-------------|
| `percent` | `number` | Pourcentage (0–100) |
| `limit` | `string` | Limite contexte (ex: "262K") |
| `tokens` | `Tokens` | `{ input, output, reasoning, cache }` |
| `cost` | `number \| undefined` | Coût accumulé |

---

### 2.3 AGENT

| Composant | Propriétés | Source | Rôle |
|-----------|-----------|--------|------|
| `AgentStatusBar` | `workers[]`, `activeAgent` | `TuiState`, `free-models.json` | Barre de statut des workers |
| `StatusIndicator` | `status`, `label`, `metric?` | Worker status | Indicateur de statut worker |
| `MetricDisplay` | `label`, `value`, `format` | Calculé (tokens/elapsed) | Métrique worker |
| `Table` | `columns[]`, `rows[]` | `free-models.json` | Tableau des workers |
| `Alert` | `level`, `title`, `message` | Worker error | Alerte worker |
| `Spinner` | `label?`, `state` | Worker state | Indicateur de chargement worker |
| `ModelSelector` | `models[]`, `selected` | `TuiState.provider`, `Session.model` | Sélecteur de modèle |

**Propriétés détaillées — AgentStatusBar :**

| Propriété | Type | Description |
|-----------|------|-------------|
| `workers` | `WorkerStatus[]` | Liste des workers avec statut |
| `activeAgent` | `string` | Agent actuellement actif |
| `position` | `"top" \| "bottom"` | Position de la barre |
| `compact` | `boolean` | Mode dense si true |

**Propriétés détaillées — MetricDisplay :**

| Propriété | Type | Description |
|-----------|------|-------------|
| `label` | `string` | Label de la métrique |
| `value` | `string` | Valeur formatée |
| `format` | `"percent" \| "tokens" \| "cost" \| "duration" \| "latency"` | Type de format |
| `unknown` | `boolean` | Si true, affiche `N/A` (jamais `0`) |

**Règle UNKNOWN :** Quand un worker n'a jamais été interrogé, afficher `[? UNKNOWN]` — jamais `[○ IDLE]`.

---

### 2.4 ACTIVITY

| Composant | Propriétés | Source | Rôle |
|-----------|-----------|--------|------|
| `EventLog` | `events[]`, `filter`, `groupBy` | `Event*` (tous types) | Flux chronologique des événements |
| `EventItem` | `event`, `type`, `timestamp`, `severity` | `EventCatalog` | Item individuel d'événement |
| `FilterBar` | `filters[]`, `activeFilter` | `EventCatalog` | Barre de filtrage |
| `Caption` | `text`, `timestamp` | `Event.time` | Timestamp d'événement |
| `Separator` | `type` | — | Séparation temporelle |
| `Badge` | `prefix`, `label`, `color` | `Event.type` | Badge de type d'événement |
| `Toast` | `title?`, `message`, `variant` | `EventTuiToastShow` | Notification d'activité |
| `Alert` | `level`, `title`, `message` | `EventSessionError`, etc. | Alerte critique |
| `SelectionList` | `items[]` | `Event` filtered | Liste sélectionnable d'événements |

**Propriétés détaillées — EventLog :**

| Propriété | Type | Description |
|-----------|------|-------------|
| `events` | `Event[]` | Liste des événements (tous types) |
| `filter` | `EventFilter` | Filtre actif (session, agent, tool, MCP, LSP, severity) |
| `groupBy` | `"session" \| "agent" \| "tool" \| "type" \| "severity"` | Groupement des événements |
| `timeRange` | `{ start, end }` | Plage temporelle |
| `maxItems` | `number` | Nombre max d'items affichés |

**Propriétés détaillées — FilterBar :**

| Propriété | Type | Description |
|-----------|------|-------------|
| `filters` | `FilterOption[]` | Options de filtrage disponibles |
| `activeFilter` | `string` | ID du filtre actif |
| `categories` | `string[]` | Catégories : session, agent, tool, MCP, LSP, severity |

**Propriétés détaillées — EventItem :**

| Propriété | Type | Description |
|-----------|------|-------------|
| `event` | `Event` | L'événement complet |
| `type` | `string` | Type d'événement (ex: `tool.called`) |
| `timestamp` | `number` | Timestamp Unix ms |
| `severity` | `info \| success \| warning \| error` | Niveau de sévérité |
| `category` | `string` | Catégorie de l'événement |
| `sessionID?` | `string` | ID de session (si applicable) |
| `agent?` | `string` | Agent concerné |

**Filtrage par catégorie :**

| Catégorie | Événements | Count |
|-----------|-----------|-------|
| Session lifecycle | `session.created`, `session.updated`, `session.deleted`, `session.status`, `session.idle`, `session.compacted`, `session.error` | 7 |
| Message lifecycle | `message.updated`, `message.removed`, `message.part.updated`, `message.part.removed`, `message.part.delta` | 5 |
| Request/stream | `session.next.*` (30+ événements) | 30+ |
| Tool | `session.next.tool.called`, `.progress`, `.success`, `.failed`, `.input.started`, `.input.ended` | 6 |
| Permission | `permission.asked`, `permission.replied`, `permission.v2.asked`, `permission.v2.replied` | 4 |
| MCP | `mcp.tools.changed`, `mcp.browser.open.failed`, `server.connected` | 3 |
| LSP | `lsp.updated`, `lsp.client.diagnostics` | 2 |
| File/Git | `file.edited`, `file.watcher.updated`, `vcs.branch.updated`, `session.diff`, `session.compacted` | 5 |
| Question | `question.asked`, `question.replied`, `question.rejected`, `question.v2.*` | 5 |
| Workspace | `workspace.ready`, `workspace.failed`, `workspace.status`, `worktree.ready`, `worktree.failed` | 5 |
| Pty | `pty.created`, `pty.updated`, `pty.exited`, `pty.deleted` | 4 |
| TUI | `tui.prompt.append`, `tui.command.execute`, `tui.toast.show`, `tui.session.select` | 4 |
| Error | `session.error` | 1 |
| Sync | `sync.*` (v2 durable) | 16+ |

---

### 2.5 COMMAND

| Composant | Propriétés | Source | Rôle |
|-----------|-----------|--------|------|
| `Prompt` | `input`, `mode?`, `parts`, `ref?` | `TuiPromptInfo`, `TuiPromptRef` | Zone de saisie de commande |
| `CommandSurface` | `commands[]`, `title` | `KeybindsConfig` | Palette de commandes |
| `KeybindDisplay` | `keybinds[]` | `KeybindsConfig` | Affichage des raccourcis |
| `Dialog` | `title`, `message`, `options?`, `size`, `onConfirm?`, `onCancel?` | `TuiDialogStack` | Dialogue modale |
| `Toast` | `title?`, `message`, `variant`, `duration?` | `EventTuiToastShow` | Notification éphémère |
| `Alert` | `level`, `title`, `message` | `EventSessionError` | Alerte |
| `Badge` | `prefix`, `label`, `color` | `CommandState` | Badge de statut commande |
| `SelectionList` | `items[]` | `CommandOptions` | Liste sélectionnable |
| `Separator` | `type` | — | Séparateur |
| `Caption` | `text`, `timestamp` | — | Texte de métadonnée |

**Propriétés détaillées — Prompt :**

| Propriété | Type | Description |
|-----------|------|-------------|
| `input` | `string` | Texte saisi par l'utilisateur |
| `mode` | `"normal" \| "shell" \| undefined` | Mode du prompt |
| `parts` | `FilePart \| AgentPart \| TextPart[]` | Parts attachées |
| `ref` | `TuiPromptRef` | Référence pour focus/submit |
| `placeholder` | `string` | Texte de placeholder |
| `disabled` | `boolean` | Si le prompt est désactivé |

**Propriétés détaillées — CommandSurface :**

| Propriété | Type | Description |
|-----------|------|-------------|
| `commands` | `CommandItem[]` | Liste des commandes disponibles |
| `title` | `string` | Titre de la palette |
| `filter` | `string` | Filtre de recherche |
| `selectedIndex` | `number` | Index de l'élément sélectionné |

**Propriétés détaillées — KeybindDisplay :**

| Propriété | Type | Description |
|-----------|------|-------------|
| `keybinds` | `KeybindItem[]` | Liste des raccourcis |
| `category` | `string` | Catégorie (navigation, session, agent, model, input, terminal) |
| `showDescriptions` | `boolean` | Afficher les descriptions |

---

## 3. Composants Réutilisés (TUI Primitives)

### 3.1 Primitives @opentui

| Composant | Package | Usage dans Command Center |
|-----------|---------|--------------------------|
| `CliRenderer` | `@opentui/core` | Rendu terminal principal |
| `Renderable` | `@opentui/core` | Base de tous les éléments rendus |
| `SlotMode` | `@opentui/core` | Gestion des slots (app, sidebar, session) |
| `RGBA` | `@opentui/core` | Couleurs RGBA pour tous les tokens |
| `KeyEvent` | `@opentui/core` | Événements clavier |
| `Keymap` | `@opentui/keymap` | Gestion des raccourcis |
| `Binding` | `@opentui/keymap` | Liaison clé-action |
| `SolidPlugin` | `@opentui/solid` | Plugin SolidJS pour le TUI |
| `JSX.Element` | `@opentui/solid` | Composants JSX |

### 3.2 Adaptateurs TuiState

| Adaptateur | Source | Output | Usage |
|------------|--------|--------|-------|
| `StatusCompact` | `TuiState.session.status()` | `SessionDisplay` | Barre de statut compacte |
| `StatusDetailed` | `TuiState.session.get()` | `SessionDisplay` | Statut détaillé dans sidebar |
| `StatusDebug` | `TuiState` + calculs | `SessionDisplay` | Vue debug complète |
| `ContextMeter` | `TuiState.part()` + `StepFinishPart` | `TokenDisplay` | Compteur de contexte |
| `McpStatusList` | `TuiState.mcp()` | `McpDisplay[]` | Liste MCP sidebar |
| `LspStatusList` | `TuiState.lsp()` | `LspDisplay[]` | Liste LSP sidebar |
| `PerformancePanel` | Calculs (tokens/elapsed) | `PerformanceData` | Panneau métriques |

### 3.3 Utilities de Formatage

| Fonction | Input | Output | Usage |
|----------|-------|--------|-------|
| `formatTokens(n)` | `number` | `"100.4K"`, `"1.2M"` | Affichage tokens |
| `formatDuration(ms)` | `number` | `"22s"`, `"01m 43s"` | Affichage durée |
| `formatCost(n)` | `number` | `"$0.00"`, `"$1.42"` | Affichage coût |
| `getContextStatus(pct)` | `number` | `"NORMAL" \| "ATTENTION" \| "HIGH" \| "CRITICAL" \| "LIMIT"` | Statut contexte |
| `getMcpLabel(status)` | `string` | `"CONNECTED" \| "DISABLED" \| ...` | Label MCP |

---

## 4. Propriétés Communes à Tous les Composants

### 4.1 Propriétés de Thème

Tous les composants reçoivent leur couleur via `theme.current` :

| Token | Rôle | Usage |
|-------|------|-------|
| `primary` | Couleur principale | En-têtes, accentuation |
| `secondary` | Couleur secondaire | Sous-titres |
| `accent` | Accent d'interaction | Focus, sélection |
| `text` | Texte principal | Contenu lisible |
| `textMuted` | Texte secondaire | Labels inactifs |
| `error` | Erreur | Échec, `✗` |
| `warning` | Avertissement | `⚠` |
| `success` | Succès | `✓` |
| `info` | Information | `ℹ` |
| `background` | Fond global | Terminal background |
| `backgroundPanel` | Fond de panneau | Boîtes |
| `backgroundElement` | Fond d'élément | Lignes sélectionnées |
| `backgroundMenu` | Fond de menu | Dropdowns |
| `border` | Bordure standard | Séparateurs |
| `borderActive` | Bordure active | Focus |
| `borderSubtle` | Bordure subtile | Délimitations légères |
| `thinkingOpacity` | Opacité | Éléments THINKING |

### 4.2 Propriétés de Responsive

| Propriété | Type | Description |
|-----------|------|-------------|
| `width` | `number` | Largeur du terminal en colonnes |
| `minWidth` | `number` | Largeur minimale du composant |
| `collapsed` | `boolean` | Si le panneau est réduit |
| `layoutMode` | `LayoutMode` | Mode de layout courant |

### 4.3 Règles de Non-Fabrication

| Règle | Application |
|-------|------------|
| `UNKNOWN != ZERO` | `undefined` ≠ `0`, `""` ≠ absent |
| `N/A` reste `N/A` | Jamais transformé |
| `$0.00` reste `$0.00` | Jamais supprimé |
| `100K · 38%` reste | Jamais complété artificiellement |
| Les métriques sont en `mono` | Toujours |
| `thinkingOpacity` s'applique uniquement à THINKING | Jamais à d'autres statuts |
| Chaque composant a un fallback pour 80 colonnes | Toujours |

---

## 5. Matrice de Réutilisation / Remplacement

| Composant existant | Action | Raison |
|-------------------|--------|--------|
| `StatusIndicator` | **Réutilisé** | Déjà défini dans design-tokens |
| `ProgressBar` | **Réutilisé** | Adaptation largeur via docs |
| `Table` | **Réutilisé** | Déjà adapté par largeur |
| `Panel` | **Réutilisé** | Déjà défini avec coins/bordures |
| `Alert` | **Réutilisé** | Déjà défini avec niveaux |
| `Dialog` | **Réutilisé** | Déjà défini dans TuiPluginApi |
| `Prompt` | **Réutilisé** | Déjà défini dans TuiPluginApi |
| `Sidebar` | **Réutilisé** | Adapté pour Command Center |
| `Toast` | **Réutilisé** | Déjà défini dans TuiPluginApi |
| `Spinner` | **Réutilisé** | Déjà défini dans design-tokens |
| `Badge` | **Réutilisé** | Déjà défini dans design-tokens |
| `Separator` | **Réutilisé** | Déjà défini dans design-tokens |
| `Caption` | **Réutilisé** | Déjà défini dans design-tokens |
| `CommandSurface` | **Réutilisé** | Déjà défini dans design-tokens |
| `KeybindDisplay` | **Réutilisé** | Déjà défini dans design-tokens |
| `EventLog` | **Nouveau** | Spécifique à l'Activity Center |
| `FilterBar` | **Nouveau** | Spécifique au filtrage d'activité |
| `AgentStatusBar` | **Nouveau** | Agrégation des workers |
| `MetricDisplay` | **Nouveau** | Formatage spécifique worker |
| `ModelSelector` | **Nouveau** | Sélection de modèle |

---

## 6. Source des Composants

### 6.1 Fichiers Source

| Composant | Fichier Source | Package |
|-----------|---------------|---------|
| Tous les types UI | `@opencode-ai/plugin/dist/tui.d.ts` | `@opencode-ai/plugin` |
| Tous les types SDK | `@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts` | `@opencode-ai/sdk` |
| Types d'événements | `@opencode-ai/sdk/dist/gen/types.gen.d.ts` | `@opencode-ai/sdk` |
| Theme/tokens | `@opentui/core` | `@opentui/core` |
| Keybindings | `@opentui/keymap` | `@opentui/keymap` |
| Rendu SolidJS | `@opentui/solid` | `@opentui/solid` |
| Configuration | `~/.config/opencode/opencode.jsonc` | Config |
| Workers status | `~/.config/opencode/free-models.json` | Config |
| Agent routing | `~/.config/opencode/agent/eurinhash.md` | Agent |

### 6.2 Slots TUI Disponibles

| Slot | Paramètres | Usage dans Command Center |
|------|------------|--------------------------|
| `app` | — | Container racine |
| `app_bottom` | — | Status bar compact |
| `sidebar_title` | `{ session_id, title, share_url? }` | Titre sidebar |
| `sidebar_content` | `{ session_id }` | Contenu sidebar (WORKSPACE) |
| `sidebar_footer` | `{ session_id }` | Infos session |
| `session_prompt` | `{ session_id, visible?, disabled?, on_submit? }` | Prompt CHAT/COMMAND |
| `session_prompt_right` | `{ session_id }` | Zone droite prompt |

---

## 7. Règles de Composants

### 7.1 Règles Fondamentales

1. **JAMAIS** hardcoder des couleurs — utiliser `theme.current`
2. **TOUJOURS** fournir un fallback textuel pour chaque glyphe
3. **JAMAIS** un composant ne dépend de la couleur seule
4. `thinkingOpacity` s'applique UNIQUEMENT aux éléments THINKING
5. Chaque composant DOIT avoir un fallback pour 80 colonnes
6. `UNKNOWN != ZERO` dans tous les composants
7. Les métriques sont TOUJOURS en `mono`

### 7.2 Règles de Composition

1. Chaque composant est construit à partir des tokens existants
2. Pas de nouvelles couleurs — seulement les tokens `TuiThemeCurrent`
3. Les bordures utilisent `border`, `borderActive`, `borderSubtle`
4. Les fonds utilisent `background`, `backgroundPanel`, `backgroundElement`, `backgroundMenu`
5. Le texte utilise `text`, `textMuted`, `selectedListItemText`
6. Les accents utilisent `primary`, `secondary`, `accent`
7. Les statuts utilisent `error`, `warning`, `success`, `info`

### 7.3 Règles de Performance

1. Éviter les full-tree rerenders — utiliser `TuiEventBus.on()` pour les mises à jour incrémentales
2. `message.part.updated` → mise à jour partielle uniquement
3. Les composants doivent se réabonner aux événements pertinents uniquement
4. Cleanup dans `onDispose` — les subscriptions doivent être nettoyées
5. `TuiState` est `readonly` — les composants ne doivent jamais tenter de le modifier

---

## 8. Résumé

| Zone | Composants principaux | Min width |
|------|----------------------|-----------|
| WORKSPACE | Sidebar, StatusIndicator, Badge, Table, KeyValueList, Panel, Alert | 20 cols |
| CHAT/SESSION | MessageList, TextBlock, ThinkingBlock, ToolCallCard, ProgressBar, DiffView, MarkdownView, Prompt | 60 cols |
| AGENT | AgentStatusBar, StatusIndicator, MetricDisplay, Table, Alert, Spinner, ModelSelector | 40 cols |
| ACTIVITY | EventLog, EventItem, FilterBar, Caption, Separator, Badge, Alert | 40 cols |
| COMMAND | Prompt, CommandSurface, KeybindDisplay, Dialog, Toast, Alert, SelectionList | 20 cols |