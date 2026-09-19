# Activity Center — Centre d'Activité avec Filtrage

> Flux chronologique des événements système avec filtrage multi-dimensionnel.
> Layout target : ACTIVITY
> Règle fondamentale : `UNKNOWN != ZERO`

---

## 1. Vue d'Ensemble

L'Activity Center est le flux chronologique de tous les événements système. Il fournit une vue unifiée de ce qui se passe en arrière-plan dans le terminal EurinHash OpenCode.

### 1.1 Événements Couverts

L'Activity Center agrège les événements de toutes les catégories :

| Catégorie | Count | Priorité affichage |
|-----------|-------|-------------------|
| Request/stream | 30+ | High — pendant l'exécution |
| Session lifecycle | 5 | High |
| Tool | 6 | Medium — contextuel |
| Permission | 4 | High — avant action |
| MCP | 3 | Medium |
| LSP | 2 | Medium |
| File/Git | 5 | Low-Medium |
| Question | 5 | High — interaction utilisateur |
| Workspace | 5 | Medium |
| Pty | 4 | Low |
| TUI | 6 | Medium |
| Error | 1 | Critical |
| Sync | 16+ | Low (background) |

### 1.2 Source des Événements

Tous les événements proviennent du `TuiEventBus` via `api.event.on()`. Le catalogue complet est documenté dans `docs/state/event-catalog.md`.

```typescript
// Abonnement aux événements Activity Center
api.event.on("session.status", (event) => addEvent(event));
api.event.on("message.part.updated", (event) => addEvent(event));
api.event.on("session.next.tool.called", (event) => addEvent(event));
api.event.on("permission.asked", (event) => addEvent(event));
api.event.on("mcp.tools.changed", (event) => addEvent(event));
api.event.on("lsp.updated", (event) => addEvent(event));
api.event.on("file.edited", (event) => addEvent(event));
api.event.on("vcs.branch.updated", (event) => addEvent(event));
api.event.on("workspace.ready", (event) => addEvent(event));
api.event.on("workspace.failed", (event) => addEvent(event));
api.event.on("session.error", (event) => addEvent(event));
// ... et tous les autres types d'événements
```

---

## 2. Architecture de l'Activity Center

### 2.1 Composants

| Composant | Rôle | Source |
|-----------|------|--------|
| `EventLog` | Flux chronologique principal | `TuiEventBus` |
| `EventItem` | Item individuel d'événement | `Event` type |
| `FilterBar` | Barre de filtrage multi-dimensionnelle | `EventCatalog` |
| `Caption` | Timestamp et métadonnées | `Event.time` |
| `Separator` | Séparation temporelle | — |
| `Badge` | Type d'événement | `Event.type` |
| `Alert` | Alerte critique | `EventSessionError` |
| `SelectionList` | Liste sélectionnable filtrée | `Event[]` filtered |

### 2.2 Flux de Données

```
TuiEventBus (api.event.on)
    │
    ▼
EventLog (collecte tous les événements)
    │
    ├── FilterBar (filtre par catégorie, agent, sévérité)
    │       │
    │       ▼
    │   Filtered Events
    │       │
    │       ▼
    │   EventItem[] (rendu individuel)
    │       │
    │       ├── Badge (type/catégorie)
    │       ├── Caption (timestamp)
    │       ├── Separator (séparation temporelle)
    │       └── Alert (si erreur/warning)
    │
    └── SelectionList (si mode sélection activé)
```

### 2.3 Règles de NON-Fabrication

| Cas | Affichage correct | Affichage interdit |
|-----|-------------------|-------------------|
| `EventSessionError.error: undefined` | Absent — pas de détail | `""` |
| `EventSessionError.sessionID: undefined` | Global error | `""` |
| `EventMessagePartUpdated.delta: undefined` | Full update | `""` |
| `EventLspUpdated.properties: {}` | Vide (objet valide) | `undefined` |

---

## 3. Filtrage

### 3.1 Dimensions de Filtrage

Le Activity Center supporte le filtrage par 6 dimensions :

| Dimension | Valeurs | Source |
|-----------|---------|--------|
| **Session** | Tous les `sessionID` actifs | `EventSession*` |
| **Agent** | `eurinhash`, `worker-codestral`, `worker-groq`, etc. | `EventSessionNextAgentSwitched`, worker status |
| **Tool** | Tous les noms d'outils | `EventSessionNextTool*` |
| **MCP** | Tous les serveurs MCP | `EventMcp*`, `TuiState.mcp()` |
| **LSP** | Tous les serveurs LSP | `EventLsp*`, `TuiState.lsp()` |
| **Severity** | `info`, `success`, `warning`, `error` | Dérivé du type d'événement |

### 3.2 Implémentation du Filtrage

```typescript
interface EventFilter {
    sessionID?: string;        // Filtrer par session
    agent?: string;            // Filtrer par agent
    tool?: string;             // Filtrer par nom d'outil
    mcp?: string;              // Filtrer par serveur MCP
    lsp?: string;              // Filtrer par serveur LSP
    severity?: string[];       // Filtrer par sévérité
    category?: string;         // Filtrer par catégorie
    timeRange?: { start: number; end: number };  // Plage temporelle
    search?: string;           // Recherche textuelle
}
```

### 3.3 Logique de Filtrage

```typescript
function filterEvents(events: Event[], filter: EventFilter): Event[] {
    return events.filter(event => {
        // Session filter
        if (filter.sessionID && !matchesSession(event, filter.sessionID)) return false;
        
        // Agent filter
        if (filter.agent && !matchesAgent(event, filter.agent)) return false;
        
        // Tool filter
        if (filter.tool && !matchesTool(event, filter.tool)) return false;
        
        // MCP filter
        if (filter.mcp && !matchesMcp(event, filter.mcp)) return false;
        
        // LSP filter
        if (filter.lsp && !matchesLsp(event, filter.lsp)) return false;
        
        // Severity filter
        if (filter.severity && !filter.severity.includes(getSeverity(event))) return false;
        
        // Category filter
        if (filter.category && getCategory(event) !== filter.category) return false;
        
        // Time range filter
        if (filter.timeRange && (event.time < filter.timeRange.start || event.time > filter.timeRange.end)) return false;
        
        // Search text filter
        if (filter.search && !matchesSearch(event, filter.search)) return false;
        
        return true;
    });
}
```

### 3.4 Groupement

| Groupement | Description | Utilisation |
|------------|-------------|-------------|
| `type` | Par catégorie d'événement | Vue par type |
| `session` | Par session ID | Débogage de session |
| `agent` | Par agent/worker | Monitoring worker |
| `severity` | Par niveau de sévérité | Vue errors d'abord |
| `time` | Par plage temporelle | Analyse temporelle |

### 3.5 Règles de Filtrage

1. Le filtrage ne modifie jamais les données source — c'est une vue dérivée
2. `UNKNOWN` reste `UNKNOWN` après filtrage — jamais filtré comme `IDLE`
3. Un événement sans payload d'erreur (`error: undefined`) ne signifie pas "pas d'erreur"
4. Les événements `sync.*` (v2 durable) sont toujours disponibles mais peuvent être regroupés

---

## 4. Affichage des Événements

### 4.1 Format d'Affichage par Type

| Type d'événement | Format d'affichage | Glyphe | Couleur |
|-----------------|-------------------|--------|---------|
| `session.created` | `[✓ SUCCESS] Session created: {name}` | `✓` | `success` |
| `session.status(busy)` | `[▶ RUNNING] Session busy` | `▶` | `primary` |
| `session.status(idle)` | `[○ IDLE] Session idle` | `○` | `textMuted` |
| `session.error` | `[✗ ERROR] {message}` | `✗` | `error` |
| `message.part.delta` | `[▓] {text}` | `▓` | `text` |
| `tool.called` | `[▶ RUNNING] {tool} - {input}` | `▶` | `primary` |
| `tool.success` | `[✓ SUCCESS] {tool} completed` | `✓` | `success` |
| `tool.failed` | `[✗ ERROR] {tool} - {error}` | `✗` | `error` |
| `permission.asked` | `[⌛ WAITING] Permission: {action}` | `⌛` | `warning` |
| `permission.replied` | `[✓ SUCCESS] Permission {response}` | `✓` | `success` |
| `mcp.tools.changed` | `[◐ THINKING] MCP: {server}` | `◐` | `secondary` |
| `lsp.updated` | `[● ACTIVE] LSP: {server}` | `●` | `accent` |
| `file.edited` | `[▸] {file}` | `▸` | `text` |
| `vcs.branch.updated` | `[▸] branch: {branch}` | `▸` | `text` |
| `workspace.ready` | `[✓ SUCCESS] Workspace: {name}` | `✓` | `success` |
| `workspace.failed` | `[✗ ERROR] {message}` | `✗` | `error` |
| `question.asked` | `[⚠ WARNING] Question: {text}` | `⚠` | `warning` |
| `pty.exited` | `[○ IDLE] Pty exited: {id}` | `○` | `textMuted` |
| `session.compacted` | `[⚠ WARNING] Session compacted` | `⚠` | `warning` |
| `command.executed` | `[▸] {command}` | `▸` | `text` |
| `sync.*` | `[◐] {syncType} seq={seq}` | `◐` | `textMuted` |

### 4.2 Séparation Temporelle

Les événements sont séparés temporellement par des `Separator` :

- Même minute : pas de séparation
- Même heure, minute différente : `─── HH:MM ───`
- Même jour, heure différente : `─── HH:MM ───`
- Jours différents : `─── MMMM DD, YYYY ───`

### 4.3 Sévérité et Couleurs

| Sévérité | Glyphe | Couleur | Background |
|----------|--------|---------|------------|
| `critical` | `✗` | `error` | `backgroundPanel` |
| `error` | `✗` | `error` | `backgroundPanel` |
| `warning` | `⚠` | `warning` | `backgroundPanel` |
| `info` | `ℹ` | `info` | `backgroundPanel` |
| `success` | `✓` | `success` | `backgroundPanel` |

---

## 5. Filtrage par Sévérité

### 5.1 Dérivation de la Sévérité

La sévérité est dérivée automatiquement du type d'événement :

| Catégorie | Sévérité par défaut | Événements |
|-----------|-------------------|------------|
| Error | `critical` | `session.error`, `tool.failed`, `workspace.failed` |
| Permission | `warning` | `permission.asked`, `permission.replied` |
| Tool | `info` | `tool.called`, `tool.progress`, `tool.success` |
| Session | `info` | `session.created`, `session.updated`, `session.status` |
| Message | `info` | `message.updated`, `message.part.updated` |
| MCP | `info` | `mcp.tools.changed`, `server.connected` |
| LSP | `info` | `lsp.updated`, `lsp.client.diagnostics` |
| File/Git | `low` | `file.edited`, `vcs.branch.updated` |
| Question | `warning` | `question.asked`, `question.replied` |
| Sync | `low` | `sync.*` |

### 5.2 Règles de Sévérité

1. `session.error` → `critical` ou `error` selon le type d'erreur
2. `tool.failed` → `error` si le tool a échoué, `warning` si échec non critique
3. `permission.asked` → `warning` car bloque l'exécution
4. Les événements `sync.*` sont toujours `low` car en arrière-plan
5. `UNKNOWN` ne change pas de sévérité — reste `info` par défaut

---

## 6. Filtrage par Agent

### 6.1 Agents Disponibles

| Agent | Modèle | Statut possible |
|-------|--------|-----------------|
| `worker-codestral` | `laguna-s-2.1:free` | ACTIVE, IDLE, THINKING, ERROR, DISCONNECTED, UNKNOWN |
| `worker-groq` | `qwen3.8-27b` | ACTIVE, IDLE, THINKING, ERROR, DISCONNECTED, UNKNOWN |
| `worker-novita` | `ling-3.0-flash-sante` | ACTIVE, IDLE, THINKING, ERROR, DISCONNECTED, UNKNOWN |
| `worker-zhipu` | `glm-4.7-flash` | ACTIVE, IDLE, THINKING, ERROR, DISCONNECTED, UNKNOWN |
| `worker-google` | `gemini-2.5-flash` | ACTIVE, IDLE, THINKING, ERROR, DISCONNECTED, UNKNOWN |
| `worker-pollinations` | `openai` | ACTIVE, IDLE, THINKING, ERROR, DISCONNECTED, UNKNOWN |

### 6.2 Filtrage Agent

```typescript
function matchesAgent(event: Event, agent: string): boolean {
    // Vérifier si l'événement est lié à l'agent
    switch (event.type) {
        case "session.next.agent.switched":
            return event.data.agent === agent;
        case "session.next.tool.called":
            return event.data.provider?.executed && isAgentTool(event, agent);
        case "session.next.model.switched":
            return isAgentModel(event, agent);
        default:
            return false;
    }
}
```

### 6.3 Règle UNKNOWN pour les Agents

| Cas | Affichage correct | Affichage interdit |
|-----|-------------------|-------------------|
| Worker non interrogé | `[? UNKNOWN]` | `[○ IDLE]` |
| Worker rate_limited | `[⚠ WARNING]` | `[● ACTIVE]` |
| Worker error | `[✗ ERROR]` | `[○ IDLE]` |
| Agent non spécifié | `[? UNKNOWN]` | `[○ IDLE]` |
| Latence inconnue | `N/A` | `0ms` |

---

## 7. Filtrage par MCP et LSP

### 7.1 Filtrage MCP

| Serveur | Status possible | Filtre |
|---------|----------------|--------|
| `cloudflare` | `connected`, `disabled`, `failed`, `needs_auth`, `needs_client_registration` | `mcp:cloudflare` |
| `cloudflare-docs` | Idem | `mcp:cloudflare-docs` |
| `postgres` | Idem | `mcp:postgres` |
| `agentdeals` | Idem | `mcp:agentdeals` |

```typescript
function matchesMcp(event: Event, mcpServer: string): boolean {
    if (event.type === "mcp.tools.changed") {
        return event.properties.server === mcpServer;
    }
    if (event.type === "mcp.browser.open.failed") {
        return event.properties.mcpName === mcpServer;
    }
    if (event.type === "server.connected") {
        // Vérifier si le serveur connecté correspond
        return isServerConnected(event, mcpServer);
    }
    return false;
}
```

### 7.2 Filtrage LSP

| Serveur | Status | Filtre |
|---------|--------|--------|
| TypeScript LSP | `connected` ou `error` | `lsp:typescript` |
| Autres LSP | Idem | `lsp:{name}` |

```typescript
function matchesLsp(event: Event, lspServer: string): boolean {
    if (event.type === "lsp.updated") {
        return isLspEvent(event, lspServer);
    }
    if (event.type === "lsp.client.diagnostics") {
        return event.properties.serverID === lspServer;
    }
    return false;
}
```

---

## 8. Filtrage par Session

### 8.1 Filtrage par Session

```typescript
function matchesSession(event: Event, sessionID: string): boolean {
    // Les événements avec sessionID dans le payload
    if ("sessionID" in event) {
        return event.sessionID === sessionID;
    }
    // Les événements avec sessionID imbriqué
    if (event.data?.sessionID) {
        return event.data.sessionID === sessionID;
    }
    return false;
}
```

### 8.2 Cas Particuliers

| Cas | Comportement |
|-----|-------------|
| `sessionID` absent | L'événement est global (ex: `server.connected`) |
| `sessionID` est `undefined` | Session globale — jamais filtré par une session spécifique |
| Session supprimée | Les événements historiques restent affichés |

---

## 9. Filtrage par Outil (Tool)

### 9.1 Filtrage par Nom d'Outil

```typescript
function matchesTool(event: Event, toolName: string): boolean {
    if (event.type === "session.next.tool.called") {
        return event.data.tool === toolName;
    }
    if (event.type === "session.next.tool.progress") {
        return isToolEvent(event, toolName);
    }
    if (event.type === "session.next.tool.success") {
        return isToolEvent(event, toolName);
    }
    if (event.type === "session.next.tool.failed") {
        return isToolEvent(event, toolName);
    }
    if (event.type === "session.next.tool.input.started") {
        return event.data.name === toolName;
    }
    if (event.type === "session.next.tool.input.delta") {
        return isToolEvent(event, toolName);
    }
    if (event.type === "session.next.tool.input.ended") {
        return isToolEvent(event, toolName);
    }
    return false;
}
```

### 9.2 Types d'Outils

| Type | Description | Exemple |
|------|-------------|---------|
| `bash` | Commande shell | `ls -la`, `git status` |
| `read` | Lecture de fichier | `read file.ts` |
| `write` | Écriture de fichier | `write file.ts` |
| `edit` | Modification de fichier | `edit file.ts` |
| MCP | Outil MCP | `agentdeals:search` |
| LSP | Outil LSP | `typescript:diagnostics` |

---

## 10. Gestion du Temps

### 10.1 Timestamps

| Format | Condition | Exemple |
|--------|-----------|---------|
| `HH:MM:SS` | Même jour | `12:00:01` |
| `MMM DD HH:MM` | Même semaine | `Sep 19 12:00` |
| `MMM DD YYYY HH:MM` | Plus vieux | `Sep 19 2026 12:00` |
| `MMM DD YYYY` | Très vieux | `Sep 19 2026` |

### 10.2 Règles de Temps

1. `time.start` est toujours avant `time.end`
2. `time.updated` reflète la dernière modification
3. Les timestamps sont en `mono` (monospace)
4. `thinkingOpacity` ne s'applique pas aux timestamps
5. `UNKNOWN` ne s'applique pas aux timestamps — ils sont toujours présents pour les événements

---

## 11. Filtrage par Sévérité (Détail)

### 11.1 Sévérité Critique

| Événement | Contexte | Action |
|-----------|----------|--------|
| `session.error` | Erreur de session | Afficher `✗ ERROR` avec le message |
| `tool.failed` | Outil en erreur | Afficher `✗ ERROR` avec l'erreur |
| `workspace.failed` | Echec workspace | Afficher `✗ ERROR` avec le message |
| `session.next.step.failed` | Échec d'étape | Afficher `✗ ERROR` avec l'erreur |

### 11.2 Sévérité Error

| Événement | Contexte | Action |
|-----------|----------|--------|
| `mcp.browser.open.failed` | Navigateur MCP échoué | Afficher `✗ ERROR` |
| `permission.replied` (reject) | Permission refusée | Afficher `✗ ERROR` |
| `tool.execute.after` avec erreur | Hook erreur | Afficher `✗ ERROR` |

### 11.3 Sévérité Warning

| Événement | Contexte | Action |
|-----------|----------|--------|
| `permission.asked` | Permission demandée | Afficher `⚠ WARNING` |
| `session.compacted` | Compaction | Afficher `⚠ WARNING` |
| `session.next.retried` | Retry | Afficher `⚠ WARNING` |
| `tool.progress` | Progression outil | Afficher `⚠ WARNING` si long |

### 11.4 Sévérité Info

| Événement | Contexte | Action |
|-----------|----------|--------|
| `session.created` | Nouvelle session | Afficher `ℹ` |
| `message.part.delta` | Streaming | Afficher `▓` |
| `file.edited` | Fichier modifié | Afficher `▸` |
| `command.executed` | Commande | Afficher `▸` |

### 11.5 Sévérité Success

| Événement | Contexte | Action |
|-----------|----------|--------|
| `tool.success` | Outil réussi | Afficher `✓ SUCCESS` |
| `session.status(idle)` | Session idle | Afficher `[○ IDLE]` |
| `workspace.ready` | Workspace prêt | Afficher `✓ SUCCESS` |
| `session.next.compaction.ended` | Compaction terminée | Afficher `✓ SUCCESS` |

---

## 12. Performance de l'Activity Center

### 12.1 Stratégie de Rendu

1. **Virtualisation** : Seulement les événements visibles sont rendus
2. **Incrémental** : Les nouveaux événements sont ajoutés sans re-rendu complet
3. **Filtrage côté client** : Le filtrage ne nécessite pas de re-synchronisation avec le serveur
4. **Groupement dynamique** : Le groupement est calculé à la volée

### 12.2 Limites

| Paramètre | Valeur | Description |
|-----------|--------|-------------|
| `maxItems` | 1000 | Nombre max d'événements en mémoire |
| `pageSize` | 50 | Nombre d'événements par page |
| `timeRange` | 24h | Fenêtre temporelle par défaut |
| `updateInterval` | 100ms | Intervalle de rafraîchissement |

### 12.3 Règles de Performance

1. `message.part.delta` → rendu incrémental uniquement (pas de re-rendu complet)
2. `TuiEventBus.on()` → abonnement ciblé, pas de polling
3. Cleanup dans `onDispose` → pas de fuite mémoire
4. `TuiState` est `readonly` → pas de mutation accidentelle

---

## 13. Règles de NON-Fabrication dans l'Activity Center

| Règle | Application |
|-------|------------|
| `UNKNOWN != ZERO` | `error: undefined` ≠ pas d'erreur |
| `N/A` reste `N/A` | Jamais remplacé par `0` ou `-` |
| `$0.00` reste `$0.00` | Jamais supprimé |
| `100K · 38%` reste | Jamais complété artificiellement |
| Les métriques sont en `mono` | Toujours |
| `thinkingOpacity` s'applique uniquement à THINKING | Jamais à d'autres statuts |
| Un statut ne doit jamais être tronqué au point de perdre son sens | `[● ACTIVE]` → `[● ACT]` → `●` |

---

## 14. Résumé

| Dimension | Composant | Source | Min width |
|-----------|-----------|--------|-----------|
| Session | `EventLog` + `FilterBar` | `EventSession*` | 40 cols |
| Agent | `EventLog` + `FilterBar` | `EventSessionNext*`, `free-models.json` | 40 cols |
| Tool | `EventLog` + `FilterBar` | `EventSessionNextTool*` | 40 cols |
| MCP | `EventLog` + `FilterBar` | `EventMcp*`, `TuiState.mcp()` | 40 cols |
| LSP | `EventLog` + `FilterBar` | `EventLsp*`, `TuiState.lsp()` | 40 cols |
| Sévérité | `EventLog` + `FilterBar` | Dérivé du type d'événement | 40 cols |

**Layout** : `app_bottom` — Activity Center en panneau déroulant sous CHAT/SESSION
**Filtrage** : 6 dimensions (session, agent, tool, MCP, LSP, severity)
**Règle fondamentale** : `UNKNOWN != ZERO`