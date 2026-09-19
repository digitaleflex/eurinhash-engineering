# Activity Feed — Flux d'Activité Chronologique

> **Source** : `@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`, `@opencode-ai/sdk/dist/gen/types.gen.d.ts`, `@opencode-ai/plugin/dist/index.d.ts`
> **Règle** : `UNKNOWN != ZERO` — chaque événement a un type et un timestamp ; les champs absents sont `undefined`.
> **Conception** : Activity center avec filtrage par session/agent/tool/MCP/LSP/severity.

---

## 1. Concept

L'Activity Feed est le flux chronologique central de toutes les activités du système. Il agrège les événements de tous les domaines (session, message, outil, permission, MCP, LSP, fichier, git, workspace, question, commande) en une vue unique et filtrable.

### 1.1 Source de vérité

L'Activity Feed est dérivé de :
1. **`Event` union** du SDK — tous les événements système
2. **`TuiEventBus`** — souscription en temps réel aux événements
3. **`TuiState`** — lecture des états actuels pour enrichissement

Le flux n'est **jamais** une source de vérité indépendante — il est toujours dérivé des événements et de l'état de domaine.

---

## 2. Taxonomie des événements

### 2.1 Par domaine

| Domaine | Événements | Exemples |
|---------|------------|----------|
| Session lifecycle | 5 | `session.created`, `session.updated`, `session.status`, `session.idle`, `session.error` |
| Message lifecycle | 4 | `message.updated`, `message.part.updated`, `message.part.removed` |
| Request/stream | 30+ | `session.next.*` (v2) — text, reasoning, tool, step, compaction, shell |
| Tool | 6 | `tool.called`, `tool.progress`, `tool.success`, `tool.failed`, `tool.input.started`, `tool.input.ended` |
| Permission | 4 | `permission.asked`, `permission.replied`, `permission.v2.asked`, `permission.v2.replied` |
| MCP | 3 | `mcp.tools.changed`, `mcp.browser.open.failed`, `server.connected` |
| LSP | 2 | `lsp.updated`, `lsp.client.diagnostics` |
| File/Git | 5 | `file.edited`, `file.watcher.updated`, `vcs.branch.updated`, `session.diff`, `session.compacted` |
| Todo | 1 | `todo.updated` |
| Command | 1 | `command.executed` |
| TUI | 6 | `tui.prompt.append`, `tui.command.execute`, `tui.toast.show`, `tui.session.select`, etc. |
| Workspace | 5 | `workspace.ready`, `workspace.failed`, `workspace.status`, `worktree.ready`, `worktree.failed` |
| Question | 5 | `question.asked`, `question.replied`, `question.rejected`, `question.v2.*` |
| Pty | 4 | `pty.created`, `pty.updated`, `pty.exited`, `pty.deleted` |
| Installation | 2 | `installation.updated`, `installation.update-available` |
| Plugin | 1 | `plugin.added` |
| Error | 1 | `session.error` |
| Sync | 16+ | `sync.*` (v2 durable events) |

### 2.2 Par sévérité

| Sévérité | Couleur | Glyphe | Événements associés |
|----------|---------|--------|---------------------|
| **Info** | `info` | `ℹ` | `session.created`, `session.updated`, `tool.input.started`, `workspace.ready` |
| **Success** | `success` | `✓` | `tool.success`, `step.ended`, `message.part.updated` (complétion) |
| **Warning** | `warning` | `⚠` | `permission.asked`, `workspace.status` (connecting), `tool.progress` |
| **Error** | `error` | `✗` | `session.error`, `tool.failed`, `permission.v2.rejected`, `workspace.failed`, `mcp.browser.open.failed` |
| **Critical** | `error` (bold) | `✗` | `session.error` avec `ProviderAuthError`, `McpStatusFailed`, `pty.exited` (non-zero) |

---

## 3. Structure d'un événement dans le flux

### 3.1 Enregistrement de l'activity feed

Chaque événement est transformé en un enregistrement d'activité :

```typescript
interface ActivityEntry {
    id: string;                    // ID unique de l'événement
    timestamp: number;             // Unix ms
    type: string;                  // Type d'événement (discriminant)
    domain: ActivityDomain;        // Domaine catégoriel
    severity: ActivitySeverity;    // Sévérité
    sessionID: string | undefined; // Session concernée (undefined si global)
    agent: string | undefined;     // Agent concerné (undefined si non pertinent)
    tool: string | undefined;      // Outil concerné (undefined si non pertinent)
    mcp: string | undefined;       // Serveur MCP (undefined si non pertinent)
    lsp: string | undefined;       // Serveur LSP (undefined si non pertinent)
    message: string;               // Texte descriptif court
    details?: { [key: string]: unknown }; // Données supplémentaires
}
```

### 3.2 `ActivityDomain`

```typescript
type ActivityDomain =
    | "session" | "message" | "request" | "tool"
    | "permission" | "mcp" | "lsp" | "file" | "git"
    | "workspace" | "todo" | "command" | "tui"
    | "question" | "pty" | "installation" | "plugin" | "sync" | "error";
```

### 3.3 `ActivitySeverity`

```typescript
type ActivitySeverity = "info" | "success" | "warning" | "error" | "critical";
```

**Règle `UNKNOWN != ZERO`** : `severity` est toujours une valeur valide. Un événement sans sévérité assignée n'existe pas dans le flux — la sévérité est toujours déterminée par le type d'événement.

---

## 4. Filtrage

L'Activity Center supporte le filtrage par six dimensions :

### 4.1 Par session

**Source** : `sessionID` dans l'entrée d'activité

**Comportement** :
- Afficher uniquement les événements d'une session spécifique
- `sessionID: undefined` est filtré (événements globaux)
- `sessionID: "abc"` affiche les événements de la session "abc"

**UI** : `SelectionList` des sessions via `TuiState.session.count()` + `TuiState.session.get(id)`

### 4.2 Par agent

**Source** : `agent` dans l'entrée d'activité

**Comportement** :
- Afficher uniquement les événements liés à un agent spécifique
- `agent: undefined` est filtré (événements non liés à un agent)
- Les événements `session.next.agent.switched` identifient les changements d'agent

**UI** : `SelectionList` des agents actifs

### 4.3 Par outil

**Source** : `tool` dans l'entrée d'activité

**Comportement** :
- Afficher uniquement les événements liés à un outil spécifique
- `tool: undefined` est filtré (événements non liés à un outil)
- Les événements `tool.called`, `tool.progress`, `tool.success`, `tool.failed` ont `tool` renseigné

**UI** : `SelectionList` des outils utilisés dans la session active

### 4.4 Par MCP

**Source** : `mcp` dans l'entrée d'activité

**Comportement** :
- Afficher uniquement les événements liés à un serveur MCP
- `mcp: undefined` est filtré
- Les événements `mcp.tools.changed`, `mcp.browser.open.failed` ont `mcp` renseigné

**UI** : `TuiState.mcp()` pour la liste des serveurs MCP

### 4.5 Par LSP

**Source** : `lsp` dans l'entrée d'activité

**Comportement** :
- Afficher uniquement les événements liés à un serveur LSP
- `lsp: undefined` est filtré
- Les événements `lsp.updated`, `lsp.client.diagnostics` ont `lsp` renseigné

**UI** : `TuiState.lsp()` pour la liste des serveurs LSP

### 4.6 Par sévérité

**Source** : `severity` dans l'entrée d'activité

**Comportement** :
- Afficher uniquement les événements d'une sévérité donnée
- Combinable avec tous les autres filtres
- Sélection multiple de sévérités possible

**UI** : `CheckboxGroup` avec les 5 niveaux de sévérité

### 4.7 Combinatoire de filtres

Les filtres sont combinés avec un **ET logique** :

```
session=abc AND agent=eurinhash AND tool=edit AND severity=error
```

Affiche uniquement les événements de la session "abc", de l'agent "eurinhash", pour l'outil "edit", avec sévérité "error".

**Règle** : Si un filtre est appliqué mais que le champ est `undefined` dans un événement, cet événement est exclu du résultat — `undefined` ne matche pas "tout".

---

## 5. Affichage du flux d'activité

### 5.1 Composant `ActivityList`

Le flux est affiché comme une liste chronologique :

```
┌───────────────────────────────────────────────────────────────┐
│  FILTRES : [Session ▼] [Agent ▼] [Tool ▼] [MCP ▼] [LSP ▼] [Sev ▼] │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  12:00:01  ℹ  [● ACTIVE] session.created · main              │
│  12:00:03  ℹ  [▶ RUNNING] tool.edit · src/utils.ts            │
│  12:00:05  ⚠  [⌛ WAITING] permission.edit · src/**/*.ts       │
│  12:00:08  ✓  [✓ SUCCESS] tool.edit · 1.2s · 3 changes         │
│  12:00:10  ℹ  [◐ THINKING] agent eurinhash · reasoning...      │
│  12:00:15  ✗  [✗ ERROR] tool.bash · Exit code 1               │
│  12:00:16  ⚠  [⌛ WAITING] mcp.filesystem · needs_auth         │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

### 5.2 Composants constitutifs

| Élément | Composant | Données |
|---------|-----------|---------|
| Timestamp | `Caption` | `time` en format `HH:mm:ss` |
| Glyphe de sévérité | `Badge` | Basé sur `severity` |
| Statut | `StatusIndicator` | Basé sur le type d'événement |
| Description | `text` | `message` de l'entrée |
| Métriques | `MetricDisplay` | Durée, nombre de changements, etc. |
| Détails | `Panel` ou `Dialog` | `details` quand l'utilisateur clique |

### 5.3 Vue par sévérité

Quand un seul filtre de sévérité est appliqué :

```
[✗ ERROR]
────────────────────────────────────────────
12:00:15  tool.bash · Exit code 1
12:00:18  session.error · ProviderAuthError
12:00:20  mcp.filesystem · Connection refused
────────────────────────────────────────────
3 erreurs
```

### 5.4 Vue par session

Quand un seul filtre de session est appliqué :

```
[Session: main-session]
────────────────────────────────────────────
12:00:01  session.created
12:00:03  tool.edit · src/utils.ts
12:00:05  permission.asked · edit
12:00:08  tool.success · 1.2s
12:00:10  reasoning · Analyse...
────────────────────────────────────────────
5 événements · 1.2s d'exécution
```

---

## 6. Ordre chronologique et garanties

### 6.1 Ordonnancement

- **Au sein d'une session** : Les événements sont ordonnés par timestamp
- **Entre sessions** : L'ordre n'est pas garanti
- **Sync events (v2)** : Le `seq` détermine l'ordre dans un aggregate
- **Delta events** : Doivent être appliqués dans l'ordre — `session.next.text.delta` etc.

### 6.2 Replay et Activity Feed

| Type de replay | Impact sur le feed | Risque |
|----------------|--------------------|--------|
| Idempotent | Re-affiche le même événement | Faible |
| State-replacement | Remplace l'état, le feed se met à jour | Moyen |
| Side-effect | Peut ré-afficher des événements avec effets | Élevé |
| Delta-replay | Doit être ordonné, skipping = corruption | Critique |

### 6.3 Déduplication

Les événements sont dédupliqués par `id`. En cas de duplicate dans le flux :
- L'événement le plus récent (par timestamp) l'emporte
- Les doublons sont silencieusement ignorés dans le feed

---

## 7. Event sourcing dans l'Activity Feed

### 7.1 Construction d'une entrée à partir d'un événement brut

```typescript
function createActivityEntry(event: Event): ActivityEntry {
    const domain = classifyDomain(event.type);
    const severity = classifySeverity(event.type);
    const sessionID = extractSessionID(event);
    const agent = extractAgent(event);
    const tool = extractTool(event);
    const mcp = extractMCP(event);
    const lsp = extractLSP(event);
    const message = formatMessage(event);

    return {
        id: event.type + ":" + extractID(event),
        timestamp: extractTimestamp(event),
        type: event.type,
        domain, severity,
        sessionID, agent, tool, mcp, lsp,
        message,
        details: extractDetails(event)
    };
}
```

### 7.2 Extraction des dimensions de filtrage

| Champ source | Extraction | Exemple |
|-------------|------------|---------|
| `EventSessionCreated.info.id` | `sessionID` | `"session-abc"` |
| `EventSessionNextToolCalled.tool` | `tool` | `"edit"` |
| `EventMcpToolsChanged.server` | `mcp` | `"filesystem"` |
| `EventLspUpdated.serverID` | `lsp` | `"typescript"` |
| `EventSessionNextAgentSwitched.agent` | `agent` | `"eurinhash"` |
| `EventPermissionV2Asked.action` | `tool` (dérivé) | `"edit"` |

### 7.3 Champs `undefined` dans l'extraction

Quand un champ ne peut pas être extrait :
- `sessionID` = `undefined` pour les événements globaux (`installation.updated`, `plugin.added`)
- `agent` = `undefined` pour les événements non liés à un agent
- `tool` = `undefined` pour les événements non liés à un outil
- `mcp` = `undefined` pour les événements non liés à MCP
- `lsp` = `undefined` pour les événements non liés à LSP

**Règle `UNKNOWN != ZERO`** : Un champ `undefined` signifie "non applicable" — jamais une valeur vide. Un filtre sur cet exclut l'événement.

---

## 8. Règles de non-fabrication

### 8.1 `UNKNOWN != ZERO` dans l'Activity Feed

| Champ | Quand absent | Valeur correcte | Valeur incorrecte |
|-------|-------------|-----------------|-------------------|
| `sessionID` | Événement global | `undefined` | `""` |
| `agent` | Non lié à un agent | `undefined` | `"unknown"` |
| `tool` | Non lié à un outil | `undefined` | `"none"` |
| `mcp` | Non lié à MCP | `undefined` | `"none"` |
| `lsp` | Non lié à LSP | `undefined` | `"none"` |
| `details` | Pas de détails | `undefined` | `{}` (bien que `{}` soit parfois correct) |

### 8.2 Sévérité

| Cas | Sévérité | Pourquoi |
|-----|----------|----------|
| `session.created` | `info` | Événement d'initialisation |
| `tool.success` | `success` | Opération réussie |
| `permission.asked` | `warning` | Action nécessitant attention |
| `session.error` | `error` | Échec |
| `session.error` + `ProviderAuthError` | `critical` | Échec d'authentification |
| `mcp.browser.open.failed` | `error` | Échec spécifique |
| `tool.failed` | `error` | Échec de l'outil |

Un événement **doit** toujours avoir une sévérité assignée. `UNKNOWN` comme sévérité est interdit.

---

## 9. Multi-agent et Activity Feed

L'Activity Feed est conçu pour afficher les activités de plusieurs agents simultanément :

- **Agent principal** : Activités dans le flux principal
- **Sous-agent** : Activités incluses dans le flux avec identification `agent`
- **Agents distants** : Activées des workers EURINHASH peuvent apparaître avec leur `agent` identifier

### 9.1 Regroupement par agent

Quand un agent spécifique est filtré :
```
[Agent: eurinhash]
────────────────────────────────────────────
12:00:01  [● ACTIVE] session.created
12:00:03  [▶ RUNNING] tool.edit · src/utils.ts
12:00:10  [◐ THINKING] reasoning · Analyse...
────────────────────────────────────────────
```

### 9.2 Distinction agent vs outil vs sous-agent

| Entité | Champ | Affichage |
|--------|-------|-----------|
| Agent principal | `agent` | Identifié par nom |
| Sous-agent | `agent` + `AgentPart` | Préfixé avec `[SUBAGENT]` |
| Exécution d'outil | `tool` | Affiché comme action |
| MCP | `mcp` | Préfixé avec `[MCP]` |
| LSP | `lsp` | Préfixé avec `[LSP]` |

---

## 10. Performance et pagination

### 10.1 Volume

Un agent actif peut générer des centaines d'événements par minute. Le feed doit gérer :
- **Pagination** : Charger par lots (pages)
- **Échantillonnage** : Les événements de progression (`tool.progress`) peuvent être échantillonnés
- **Filtrage serveur** : Les filtres doivent être appliqués côté serveur quand possible

### 10.2 Stratégie de rétention

| Type d'événement | Rétention | Raison |
|-------------------|-----------|--------|
| `session.created` | Durée de vie de la session | Audit |
| `tool.success/failed` | Durée de vie de la session | Débogage |
| `tool.progress` | Court terme (quelques minutes) | Informatif uniquement |
| `message.part.delta` | Court terme | Delta peut être recalculé |
| `session.error` | Permanent | Diagnostic |
| `session.idle` | Durée de vie de la session | Contexte |

---

## 11. Règles de non-fabrication dans l'Activity Feed

1. **JAMAIS** afficher `0` ou `""` pour un champ absent — utiliser `undefined` et ne pas afficher
2. **JAMAIS** fabriquer un nom d'agent ou de tool — utiliser uniquement les valeurs des événements
3. **JAMAIS** afficher une sévérité `UNKNOWN` — chaque événement a une sévérité déterminée
4. **TOUJOURS** afficher le timestamp dès qu'il est disponible
5. **JAMAIS** confondre `sessionID: undefined` (événement global) avec `sessionID: ""` (ID vide)
