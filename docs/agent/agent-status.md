# Agent Status — Statut de l'Agent

> **Source** : `@opencode-ai/plugin/dist/tui.d.ts`, `@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`, `@opencode-ai/sdk/dist/gen/types.gen.d.ts`
> **Règle** : `UNKNOWN != ZERO` — le statut est toujours une valeur déterminée ; jamais une valeur fabriquée.

---

## 1. Vue d'ensemble

Le statut de l'agent est la représentation la plus visible de l'état courant dans HashCode Terminal. Il indique à l'utilisateur ce que l'agent est en train de faire, ce qu'il attend, et s'il y a des problèmes nécessitant une attention.

Le statut est dérivé de `TuiState` et des événements système. Il ne jamais être une source de vérité indépendante.

---

## 2. Vocabulaire de statut

### 2.1 Statut de l'agent (11 valeurs)

| Statut | Glyphe | Couleur | Badge | Sémantique |
|--------|--------|---------|-------|------------|
| ACTIVE | `●` | `accent` | `[● ACTIVE]` | Opérationnel, prêt |
| IDLE | `○` | `textMuted` | `[○ IDLE]` | Inactif, pas de tâche |
| THINKING | `◐` | `secondary` + `thinkingOpacity` | `[◐ THINKING]` | En raisonnement |
| RUNNING | `▶` | `primary` | `[▶ RUNNING]` | En exécution |
| WAITING | `⌛` | `warning` | `[⌛ WAITING]` | En attente d'entrée externe |
| SUCCESS | `✓` | `success` | `[✓ SUCCESS]` | Réussi |
| WARNING | `⚠` | `warning` | `[⚠ WARNING]` | Condition non-idéale |
| ERROR | `✗` | `error` | `[✗ ERROR]` | Échec |
| CANCELLED | `⊘` | `textMuted` | `[⊘ CANCELLED]` | Annulé |
| UNKNOWN | `?` | `textMuted` | `[? UNKNOWN]` | Indéterminé |
| DISCONNECTED | `⊘`/`✗` | `error` | `[⊘ DISCONNECTED]` | Lien perdu |

### 2.2 Règles fondamentales

1. **Double signal** : Chaque statut est lisible sans dépendre de la couleur seule (texte + glyphe)
2. **`UNKNOWN != ZERO`** : `UNKNOWN` ne se confond jamais avec `ZERO`, `IDLE`, ou tout autre statut
3. **`CANCELLED != IDLE`** : Cancelled = action interrompue ; Idle = pas encore commencé
4. **`DISCONNECTED` est un sous-ensemble d'`ERROR`** mais a sa propre signalétique
5. **`thinkingOpacity` s'applique UNIQUEMENT à THINKING**

---

## 3. Statut par type d'entité

### 3.1 Agent principal (eurinhash)

**Source** : `TuiState.session.status(sessionID)` + `SessionStatus`

```
[● ACTIVE]  eurinhash-agent  ·  38% · 100K / 262K · $0.00
```

**Champs affichés** :
| Champ | Source | UNKNOWN |
|-------|--------|---------|
| Nom | `Session` (agent) | Jamais absent pour un agent configuré |
| Pourcentage | `StepFinishPart.tokens` / `limit.context` | `N/A` quand la limite est inconnue |
| Coût | `Session.cost` | `N/A` quand pas calculé (undefined → N/A) |
| Dernière action | Dernier événement de la session | — |

**Règle `UNKNOWN != ZERO`** :
- `cost: undefined` → affiche `N/A` (pas `N/A` ne veut pas dire "0")
- `cost: 0` → affiche `$0.00` (coût nul confirmé)
- `100K / 262K · 38%` quand la limite est connue
- `100K · 38%` quand la limite est inconnue

### 3.2 Workers (sub-agents)

**Source** : `free-models.json` + `scripts/free-probe.py` + `EventSessionNextAgentSwitched`

```
[◐ THINKING]  worker-novita  ·  2.3s · Attente réponse
[✗ ERROR]  worker-google  ·  Connection refused · N/A
[○ IDLE]  worker-pollinations  ·  Initializing
[⊘ DISCONNECTED]  worker-codestral  ·  Reconnection in 5s
```

**Champs affichés** :
| Champ | Source | UNKNOWN |
|-------|--------|---------|
| Nom du worker | `free-models.json` | Jamais absent |
| Statut | `free-models.json` | Toujours une valeur valide |
| Temps | `EventSessionNextStepStarted.timestamp` | `N/A` si non démarré |
| Model | `Session.model.id` | `N/A` si pas de modèle |

**Règle `UNKNOWN != ZERO`** :
- Un worker sans statut connu affiche `[? UNKNOWN]` — jamais `[○ IDLE]` ou `[0%]`
- Le temps de réponse `2.3s` est réel ; s'il n'est pas disponible, c'est `N/A` pas `0s`

### 3.3 Outils

**Source** : `ToolPart.state`, `SessionMessageToolState*`

| État outil | Statut agent | Affichage |
|------------|-------------|-----------|
| `pending` | WAITING | `[⌛ WAITING] edit · src/utils.ts` |
| `running` | RUNNING | `[▶ RUNNING] edit · 1.2s` |
| `completed` | SUCCESS | `[✓ SUCCESS] edit · 0.8s · 3 changes` |
| `error` | ERROR | `[✗ ERROR] edit · Exit code 1` |

### 3.4 Serveurs MCP

**Source** : `TuiState.mcp()` → `TuiSidebarMcpItem`

| `McpStatus["status"]` | Affichage |
|------------------------|-----------|
| `"connected"` | `[● ACTIVE] filesystem` |
| `"disabled"` | `[○ IDLE] filesystem` |
| `"failed"` | `[✗ ERROR] filesystem · Connection refused` |
| `"needs_auth"` | `[⌛ WAITING] filesystem · Auth required` |
| `"needs_client_registration"` | `[? UNKNOWN] filesystem · Registration needed` |

**Règle `UNKNOWN != ZERO`** :
- `"needs_client_registration"` est un état explicite — pas `UNKNOWN`
- `"disabled"` est un état explicite — pas `IDLE`
- `"connected"` signifie opérationnel — pas `"idle"`

### 3.5 Serveurs LSP

**Source** : `TuiState.lsp()` → `TuiSidebarLspItem`

| `LspStatus.status` | Affichage |
|--------------------|-----------|
| `"connected"` | `[✓ SUCCESS] typescript · diagnostics OK` |
| `"error"` | `[✗ ERROR] typescript · Server crashed` |

**Règle `UNKNOWN != ZERO`** :
- `status: "error"` est un état d'erreur explicite — pas `"not connected"` ou `"idle"`
- `status: "connected"` est opérationnel — pas `"idle"`
- Il n'y a pas de statut `UNKNOWN` pour LSP — soit connecté, soit en erreur

---

## 4. Composant `StatusIndicator`

### 4.1 Propriétés

| Propriété | Type | Description |
|-----------|------|-------------|
| `status` | `Status enum` | L'un des 11 statuts |
| `label` | `string` | Nom de l'agent/service |
| `metric` | `string` | Métrique optionnelle |
| `thinkingOpacity` | `number` | Appliqué si THINKING |

### 4.2 Rendu

```
[● ACTIVE]  eurinhash-agent  ·  38% · 100K / 262K · $0.00
```

**Construit avec** :
- Glyphe statut + libellé + couleur `TuiThemeCurrent`
- `thinkingOpacity` appliqué si THINKING
- `MetricDisplay` pour les métriques

### 4.3 Fallback

Si le terminal ne supporte pas les glyphes Unicode :
```
[ACTIVE]  eurinhash-agent  ·  38% · 100K / 262K · $0.00
```

Niveau de fallback : `●` → `*` → `[ACTIVE]` → `ACTIVE`

---

## 5. Composant `StatusBar`

### 5.1 Layout global

Barre de statut affichée en haut ou en bas du terminal :

```
[● ACTIVE] eurinhash · [◐ THINKING] novita · [✗ ERROR] google · [○ IDLE] pollinations
```

**Propriétés** :
| Propriété | Type | Description |
|-----------|------|-------------|
| `items` | `StatusIndicator[]` | Liste des statuts |
| `position` | `"top" \| "bottom"` | Position |
| `compact` | `boolean` | Mode dense |

### 5.2 Adaptation par largeur

| Largeur | Affichage |
|---------|-----------|
| ≥ 120 cols | Barre complète avec tous les items et métriques |
| 100–119 cols | Items avec labels tronqués |
| 80–99 cols | Icônes et labels courts |
| < 80 cols | Masquée (hamburger menu) |

---

## 6. État composite de l'agent

### 6.1 Combinaisons de statuts

| Contexte | Affichage |
|----------|-----------|
| Agent actif + tâche en cours | `[● ACTIVE] [▶ RUNNING] AgentName` |
| Agent actif + en attente | `[● ACTIVE] [⌛ WAITING] AgentName` |
| Agent inactif + erreur | `[○ IDLE] [✗ ERROR] AgentName` |
| Agent pensant + erreur | `[◐ THINKING] [✗ ERROR] AgentName` |
| Agent déconnecté | `[⊘ DISCONNECTED] AgentName` |

### 6.2 Règles de combinaison

1. **Maximum un statut de base** (ACTIVE, IDLE, RUNNING, WAITING, etc.)
2. **Un statut de résultat** peut s'ajouter (SUCCESS, ERROR, WARNING)
3. `thinkingOpacity` ne s'applique qu'au statut THINKING, pas aux autres
4. Les glyphes ne doivent jamais se chevaucher — un seul glyphe statut par ligne

---

## 7. Dérivation du statut depuis TuiState

### 7.1 Algorithme de dérivation

```typescript
function deriveAgentStatus(state: TuiState, sessionID: string): AgentStatus {
    const sessionStatus = state.session.status(sessionID);
    const messages = state.session.messages(sessionID);
    const permissions = state.session.permission(sessionID);
    const mcpItems = state.mcp();
    const lspItems = state.lsp();

    // 1. Vérifier les erreurs en premier
    if (hasError(sessionStatus, messages)) return { status: "ERROR", ... };

    // 2. Vérifier les permissions en attente
    if (permissions.length > 0) return { status: "WAITING", ... };

    // 3. Vérifier MCP/LSP déconnectés
    if (hasDisconnectedMCP(mcpItems)) return { status: "DISCONNECTED", ... };

    // 4. Dériver depuis SessionStatus
    switch (sessionStatus?.type) {
        case "idle": return { status: "IDLE", ... };
        case "busy":
            // Analyser le dernier message pour le sous-statut
            return deriveBusyStatus(messages);
        case "retry": return { status: "WARNING", ... };
        default: return { status: "UNKNOWN", ... };
    }
}
```

### 7.2 Dérivation des sous-statuts (busy)

Quand `SessionStatus = { type: "busy" }`, le sous-statut est dérivé du dernier message :

| Dernier message | Sous-statut | Badge |
|-----------------|-------------|-------|
| `StepStartPart` | RUNNING | `[▶ RUNNING]` |
| `TextPart` (streaming) | RUNNING | `[▶ RUNNING]` |
| `ReasoningPart` (streaming) | THINKING | `[◐ THINKING]` |
| `ToolPart` (pending) | WAITING | `[⌛ WAITING]` |
| `ToolPart` (running) | RUNNING | `[▶ RUNNING]` |
| `ToolPart` (completed) | SUCCESS | `[✓ SUCCESS]` |
| `ToolPart` (error) | ERROR | `[✗ ERROR]` |

---

## 8. Panel d'agent (agent-panel)

### 8.1 Identité de l'agent

| Champ | Source | UNKNOWN |
|-------|--------|---------|
| Nom | `Session.agent` | `undefined` si pas configuré |
| Modèle | `Session.model.id` | `undefined` si pas configuré |
| Provider | `Session.model.providerID` | `undefined` si pas configuré |
| Variant | `Session.model.variant` | `undefined` si variant par défaut |

### 8.2 Tâche en cours

| Champ | Source | UNKNOWN |
|-------|--------|---------|
| Requête courante | Dernier `UserMessage` | `undefined` si aucun message |
| Phase courante | `StepFinishPart.reason` ou dernier événement | `undefined` si aucune phase |
| Temps écoulé | `time.start` → `Date.now()` | `N/A` si pas démarré |
| Prochaine action attendue | Dérivé de l'état en cours | `undefined` si non déterminable |

### 8.3 Permission requise

| Champ | Source | UNKNOWN |
|-------|--------|---------|
| Permission en attente | `TuiState.session.permission(sessionID)` | `[]` si aucune permission |
| Type de permission | `PermissionV2Asked.action` | `undefined` si pas de permission demandée |

**Règle** : Si aucune permission n'est en attente, le champ est `[]` ou absent — jamais `undefined` pour le tableau si le champ existe.

### 8.4 État d'erreur

| Champ | Source | UNKNOWN |
|-------|--------|---------|
| Type d'erreur | `EventSessionError.error.name` | `undefined` si pas d'erreur |
| Message d'erreur | `EventSessionError.error.message` | `undefined` si pas d'erreur |
| Récupérable | Dérivé du type d'erreur | `undefined` si pas d'erreur |

---

## 9. `UNKNOWN != ZERO` dans le statut

### 9.1 Cas critiques

| Situation | Valeur correcte | Valeur incorrecte | Pourquoi |
|-----------|-----------------|-------------------|----------|
| Agent sans statut | `[? UNKNOWN]` | `[○ IDLE]` ou `[0%]` | `UNKNOWN` est indéterminé, pas idle |
| Coût non calculé | `N/A` | `$0.00` ou `0` | `0` veut dire "gratuit" |
| Limite inconnue | `100K · 38%` | `100K / 0 · 38%` | `0` comme limite est invalide |
| Agent non configuré | `N/A` pour le modèle | `{id: ""}` | Objet vide vs absent |
| Pas de prochaine action | `undefined` ou `N/A` | `"idle"` ou `"waiting"` | Pas de prédiction fabriquée |
| Worker non sondé | `[? UNKNOWN]` | `[○ IDLE]` | Pas sondé ≠ inactif |
| MCP `needs_auth` | `[⌛ WAITING] · Auth required` | `[? UNKNOWN]` | État explicite, pas inconnu |
| LSP `error` | `[✗ ERROR] · Server crashed` | `[○ IDLE]` | Erreur ≠ inactif |

### 9.2 Règles de l'interface de statut

1. **JAMAIS** afficher `0` pour UNKNOWN — utiliser `N/A` ou `?`
2. **JAMAIS** afficher une valeur fabriquée pour un statut non-déterminé
3. **TOUJOURS** afficher le texte du statut même si la couleur est omise
4. **JAMAIS** utiliser la couleur seule pour distinguer des statuts
5. `thinkingOpacity` s'applique UNIQUEMENT à THINKING, jamais à d'autres statuts
6. Un statut `UNKNOWN` doit rester `UNKNOWN` même après des tentatives de résolution échouées

---

## 10. Multi-agent et statut

### 10.1 Agent principal vs sous-agent

| Entité | Afficher | Différenciation |
|--------|----------|-----------------|
| Agent principal (eurinhash) | `StatusIndicator` principal | `[● ACTIVE] eurinhash-agent` |
| Sous-agent | `StatusIndicator` secondaire | `[◐ THINKING] worker-novita` |
| Exécution d'outil | Status basé sur ToolPart | `[▶ RUNNING] edit · fichier.ts` |

### 10.2 Agent pool

Quand plusieurs workers sont actifs :
```
[● ACTIVE] eurinhash · [◐ THINKING] novita · [✗ ERROR] google · [○ IDLE] pollinations
```

Chaque worker est un `StatusIndicator` dans la `StatusBar`. Le worker principal est en premier.

### 10.3 Agent status par session

Quand plusieurs sessions sont ouvertes :
```
[● ACTIVE] session:main · [○ IDLE] session:secondary
```

La barre de statut peut afficher un résumé par session si `compact = false`.

---

## 11. Résumé

Le statut de l'agent est l'interface la plus visible entre le système et l'utilisateur. Il est toujours dérivé de `TuiState` et des événements système, jamais fabriqué. Les 11 statuts couvrent tous les cas possibles, et la règle `UNKNOWN != ZERO` garantit qu'aucune donnée absente n'est remplacée par une valeur zéro ou vide.
