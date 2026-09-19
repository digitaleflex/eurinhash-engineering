# Information Hierarchy — Hiérarchie Visuelle des Informations

> **Source**: Docs design/architecture/state existants, Prompt `02-ux-information-architecture.md`
> **Date**: 2026-09-19
> **Règle**: « Keep high-frequency information stable to avoid visual noise. »

---

## 1. Principe de la Hiérarchie

L'information dans le HashCode Terminal est organisée en **4 niveaux hiérarchiques** basés sur la fréquence, la criticité et la permanence :

```
┌─────────────────────────────────────────────────────────────────┐
│               NIVEAU 0 — PERMANENT (toujours visible)           │
│  Prompt COMMAND | Statut global | Mode de densité                 │
│  ████████████████████████████████████████████████████████████████ │
│  Fréquence: 100% | Permanence: 100% | Taille: fixe              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           NIVEAU 1 — STATIQUE (stable pendant une session)      │
│  WORKSPACE | AGENT status | Mode actif | Theme                  │
│  ████████████████████████████████████████████████████████████████ │
│  Fréquence: 5-10% | Permanence: Élevée | Taille: sidebar        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│         NIVEAU 2 — DYNAMIQUE (change pendant l'exécution)       │
│  CHAT/SESSION stream | Tool progress | Reasoning | Part status   │
│  ████████████████████████████████████████████████████████████████ │
│  Fréquence: 50-100% | Permanence: Faible | Taille: centrale     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│          NIVEAU 3 — ÉPHÉMÈRE (apparaît et disparaît)           │
│  ACTIVITY events | Toast | Alerts | Permission prompts          │
│  ████████████████████████████████████████████████████████████████ │
│  Fréquence: 100% pendant l'event | Permanence: Très faible       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Niveau 0 — Permanent

### 2.1 Composants permanents

| Composant | Zone | Contenu | Règle |
|-----------|------|---------|-------|
| `Prompt` | COMMAND | `›` ou `$` + input | Toujours visible en bas |
| `StatusBar` | Global | Workers status | Toujours visible en haut ou bas |
| `DensityIndicator` | Global | Mode actuel | Toujours visible |
| `ThemeIndicator` | Global | Mode dark/light | Visible quand changé |

### 2.2 Contenu du StatusBar

```
[● ACTIVE] eurinhash [◐ THINKING] novita [✗ ERROR] google [○ IDLE] pollinations  ·  STANDARD  ·  120 cols
```

**Ordre des workers dans le StatusBar** : `eurinhash` (agent) → workers par score EV décroissant.

**Règle** : Le StatusBar est le seul composant visible à TOUS les niveaux de densité et à TOUTES les largeurs.

### 2.3 Règles du Niveau 0

1. **JAMAIS** masquer le prompt — c'est l'ancrage visuel
2. **JAMAIS** changer le statut du StatusBar pendant un stream — le StatusBar est stable
3. **TOUJOURS** afficher le mode de densité actuel
4. `thinkingOpacity` s'applique uniquement aux éléments THINKING du StatusBar
5. `UNKNOWN` reste `UNKNOWN` dans le StatusBar — jamais remplacé par `IDLE`

---

## 3. Niveau 1 — Statique

### 3.1 Composants statiques

| Composant | Zone | Contenu | Fréquence de changement |
|-----------|------|---------|------------------------|
| `Sidebar` | WORKSPACE | Sessions, todos, fichiers | Lors d'un changement de session |
| `AgentStatus` | AGENT | Liste des workers | Lors d'un routing/switch |
| `ModelIndicator` | CHAT/SESSION | Modèle actif | Lors d'un switch de modèle |
| `VcsInfo` | WORKSPACE | Branche git | Lors d'un `EventVcsBranchUpdated` |
| `LspStatus` | TOOLS | Statut LSP | Lors d'un `EventLspUpdated` |
| `McpStatus` | TOOLS | Statut MCP | Lors d'un `EventMcpToolsChanged` |

### 3.2 Priorité de rendu

```
1. WORKSPACE sidebar (toujours affiché si largeur ≥ 20 cols)
2. AGENT status bar (toujours affiché)
3. Model/Agent indicator dans CHAT (toujours affiché)
4. VCS/Branch info (toujours affiché si disponible)
5. LSP/MCP status (contextuel, sidebar)
```

### 3.3 Règles du Niveau 1

1. **Stable** : Ces éléments ne changent pas pendant un stream de réponse
2. **Lisibles** : Toujours affichés avec leur texte complet (pas seulement le glyphe)
3. **Pas de surcharge** : Un seul item par zone peut être « actif » à la fois
4. `UNKNOWN` reste `UNKNOWN` : `[? UNKNOWN]` si le statut n'a jamais été déterminé
5. **Séparateurs** : `borderSubtle` entre les items, `border` pour la bordure de la sidebar

---

## 4. Niveau 2 — Dynamique

### 4.1 Composants dynamiques

| Composant | Zone | Contenu | Fréquence de changement |
|-----------|------|---------|------------------------|
| `TextBlock` | CHAT/SESSION | Texte streaming | Delta par delta |
| `ThinkingBlock` | CHAT/SESSION | Reasoning streaming | Delta par delta |
| `ToolCallCard` | CHAT/SESSION | Outil en cours | State change |
| `ProgressBar` | CHAT/SESSION | Progression | Step by step |
| `DiffView` | CHAT/SESSION | Diff de code | À la fin d'un step |
| `StepBoundary` | CHAT/SESSION | Début/fin de step | À chaque step |
| `CostDisplay` | CHAT/SESSION | Coût accumulé | À chaque step finish |
| `TokenDisplay` | CHAT/SESSION | Tokens comptés | À chaque step finish |

### 4.2 Priorité de rendu dans CHAT/SESSION

```
1. Texte streaming (le plus prioritaire — c'est la réponse)
2. Reasoning streaming (secondaire — mais visible)
3. Tool progress (contextuel — affiché pendant l'exécution)
4. Step boundaries (structure — affiché au début/fin)
5. Cost/Token display (métriques — en fin de step)
```

### 4.3 Flux de mise à jour

```
session.next.text.started
  → session.next.text.delta × N
  → session.next.text.ended

session.next.reasoning.started
  → session.next.reasoning.delta × N
  → session.next.reasoning.ended

session.next.tool.called
  → session.next.tool.progress × N
  → session.next.tool.success | session.next.tool.failed

session.next.step.started
  → [text/reasoning/tool events]
  → session.next.step.ended | session.next.step.failed
```

### 4.4 Règles du Niveau 2

1. **Le stream est roi** : Pendant un stream, CHAT occupe >80% de la surface
2. **Deltas uniquement** : Seuls les deltas sont affichés — pas de re-render complet
3. **`thinkingOpacity` appliqué** : Le reasoning est rendu avec `thinkingOpacity`
4. **Pas de flicker** : Les mises à jour sont incrémentales, pas de re-render complet
5. **`UNKNOWN != ZERO`** : `cost: undefined` ≠ `cost: 0`. Les tokens non comptés restent `undefined`
6. **Stabilité** : Les métriques de haut niveau (coût, tokens) sont affichées de manière stable pendant le stream

---

## 5. Niveau 3 — Éphémère

### 5.1 Composants éphémères

| Composant | Zone | Contenu | Durée d'affichage |
|-----------|------|---------|-------------------|
| `Toast` | Global | Notification | Courte (info/success) / Moyenne (warning) / Longue (error) |
| `Alert` | Zone concernée | Alerte contextuelle | Jusqu'à confirmation utilisateur |
| `Dialog` | Global | Confirmation/sélection/saisie | Jusqu'à réponse utilisateur |
| `PermissionPrompt` | COMMAND | Demande de permission | Jusqu'à `ask`/`deny`/`allow` |
| `QuestionPrompt` | COMMAND | Question utilisateur | Jusqu'à réponse |
| `ActivityEvent` | ACTIVITY | Événement individuel | Jusqu'à défilement |
| `Spinner` | CHAT/SESSION | Chargement | Pendant l'exécution |

### 5.2 Priorité d'affichage des Alerts

```
1. Error alert — affiché immédiatement, bloque l'action suivante
2. Permission alert — affiché immédiatement, attend la réponse
3. Warning alert — affiché, ne bloque pas
4. Info alert — affiché, le plus discret
5. Toast — éphémère, auto-dismiss
```

### 5.3 Règles du Niveau 3

1. **Un alert à la fois** : Un seul alert/blockage actif à la fois
2. **Focus forcé** : Un alert de niveau error ou permission force le focus vers COMMAND
3. **`thinkingOpacity` ne s'applique pas** : Les alertes sont toujours pleinement opaques
4. **Raison obligatoire** : Chaque alerte DOIT avoir une raison claire
5. **`UNKNOWN` reste `UNKNOWN`** : Une alerte avec donnée inconnue affiche `[? UNKNOWN]` — jamais `[○ IDLE]` ou autre
6. **Pas de superposition** : Les toasts ne se superposent pas — le dernier remplace le précédent

---

## 6. Hiérarchie Visuelle par Composant

### 6.1 Grille de priorité

| Composant | Niveau | Priorité | Opacité | Largeur min |
|-----------|--------|----------|---------|-------------|
| `Prompt` | 0 | 1 | 100% | 20 cols |
| `StatusBar` | 0 | 2 | 100% | 60 cols |
| `Sidebar` | 1 | 3 | 100% | 20 cols |
| `AgentStatus` | 1 | 4 | 100% | 40 cols |
| `TextBlock` | 2 | 5 | 100% | 60 cols |
| `ThinkingBlock` | 2 | 6 | `thinkingOpacity` | 40 cols |
| `ToolCallCard` | 2 | 7 | 100% | 60 cols |
| `ProgressBar` | 2 | 8 | 100% | 20 cols |
| `DiffView` | 2 | 9 | 100% | 60 cols |
| `Toast` | 3 | 10 | 100% | 40 cols |
| `Alert` | 3 | 11 | 100% | 40 cols |
| `Dialog` | 3 | 12 | 100% | 50 cols |
| `ActivityEvent` | 3 | 13 | 100% | 40 cols |
| `Spinner` | 3 | 14 | 100% | 20 cols |

### 6.2 Surface d'affichage par zone

| Zone | Surface MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|------|-----------------|----------|-----------|----------|-------|
| WORKSPACE | 0% (masquée) | 20% | 15% | 10% | 10% |
| CHAT/SESSION | 80% | 60% | 70% | 30% | 30% |
| AGENT | 5% (status bar) | 15% | 10% | 20% | 15% |
| ACTIVITY | 0% | 5% | 5% | 30% | 30% |
| COMMAND | 15% | 20% | 15% | 10% | 15% |
| OBSERVABILITY | 0% | overlay | overlay | overlay | overlay |
| TOOLS | 0% | overlay | overlay | overlay | overlay |

---

## 7. Hiérarchie Typographique

### 7.1 Niveaux de typographie

| Niveau | Token | Usage | Zone |
|--------|-------|-------|------|
| `display` | `bold` + `primary` + `space-lg` | Titres de section | WORKSPACE headers |
| `heading` | `bold` + `text` | Titres de composant | AGENT, ACTIVITY |
| `body` | `regular` + `text` | Contenu principal | CHAT/SESSION |
| `label` | `bold` + `textMuted` | Labels, clés | KEYVALUEList |
| `mono` | `font-mono` + `textMuted` | Données techniques | Tous les composants |
| `caption` | `italic` + `textMuted` | Timestamps, métadonnées | ACTIVITY |

### 7.2 Règles typographiques

1. **JAMAIS** mix `mono` et `proportional` sur la même ligne sans séparation
2. **TOUJOURS** `mono` pour les nombres, IDs, pourcentages, coûts
3. **`caption`** pour les timestamps et les informations de métadonnées
4. **`label`** en `bold` pour les clés de données (ex: `Nom: valeur`)
5. `thinkingOpacity` s'applique UNIQUEMENT aux éléments en état THINKING
6. **`UNKNOWN`** est toujours affiché en `textMuted` avec le glyphe `?`

---

## 8. Hiérarchie des Données par Zone

### 8.1 WORKSPACE — Données par priorité

```
1. Répertoire projet (path.directory) — toujours visible
2. Branche git (vcs.branch) — si disponible
3. Fichiers modifiés (session.diff) — si disponibles
4. Todos (session.todo) — si disponibles
5. Statut workspace (EventWorkspaceStatus) — si error/disconnected
```

### 8.2 CHAT/SESSION — Données par priorité

```
1. Message user actuel — toujours visible
2. Response streaming (text/reasoning) — le plus prioritaire
3. Tool progress — contextuellement prioritaire
4. Step boundaries — structurel
5. Cost/tokens — métriques, affichées en fin de step
6. Error messages — affichées immédiatement si présentes
```

### 8.3 AGENT — Données par priorité

```
1. Worker actif — toujours visible
2. Modèle sélectionné — toujours visible
3. Liste des workers avec statut — visible dans le status bar
4. Latence/throughput — observabilité, overlay
5. Coût par worker — métrique dérivée
```

### 8.4 ACTIVITY — Données par priorité

```
1. Événements error — toujours affichés en premier
2. Événements permission/question — bloquants
3. Événements session.status — structurants
4. Événements tool — contextuels
5. Événements file/vcs — informatifs
6. Événements sync — background, faible priorité
```

### 8.5 COMMAND — Données par priorité

```
1. Input utilisateur — toujours visible
2. Mode (normal/shell) — indiqué par l'indicateur
3. Historique des commandes — accessible via ↑/↓
4. Complétion automatique — contextuelle
5. Keybindings — accessibles via help
```

---

## 9. Règles de Non-Fabrication dans la Hiérarchie

| Règle | Description | Exemple |
|-------|-------------|---------|
| `UNKNOWN != ZERO` | Jamais afficher 0 pour une donnée inconnue | `cost: undefined` → `N/A` pas `$0.00` |
| `UNKNOWN != IDLE` | Un statut inconnu n'est pas idle | `[? UNKNOWN]` pas `[○ IDLE]` |
| `N/A != 0` | N/A est N/A, jamais 0 | Limite inconnue → `N/A` pas `0` |
| `undefined != ""` | Pas de champ undefined remplacé par empty string | `branch: undefined` pas `branch: ""` |
| `0 est valide` | 0 comme valeur réelle est correct | `tokens.input: 0` est correct |

---

## 10. Interaction entre Niveaux et Modes

### 10.1 Règles par mode

| Règle | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|-------|---------|----------|-----------|----------|-------|
| Niveau 0 visible | Oui | Oui | Oui | Oui | Oui |
| Niveau 1 visible | Partiel | Oui | Oui | Oui | Oui |
| Niveau 2 visible | Oui | Oui | Oui | Oui | Oui |
| Niveau 3 visible | Contextuel | Oui | Contextuel | Oui | Oui |
| `thinkingOpacity` | Oui | Oui | Oui | Oui | Oui |
| Données brutes | Non | Non | Oui | Oui | Oui |
| Métriques détaillées | Non | Partiel | Oui | Oui | Oui |

### 10.2 Règle de la non-fabrication par niveau

- **Niveau 0** : Aucune donnée fabriquée — le prompt et le StatusBar reflètent l'état réel
- **Niveau 1** : Les états statiques ne sont jamais fabriqués — `undefined` reste `undefined`
- **Niveau 2** : Les streams affichent uniquement les deltas reçus — jamais de contenu généré
- **Niveau 3** : Les alertes ont toujours une raison — jamais d'alerte sans cause
- **TOUS les niveaux** : `UNKNOWN` reste `UNKNOWN`, `N/A` reste `N/A`, `null` reste `null`

---

## 11. Références

- `docs/design/design-tokens.md` — Tokens de conception et hiérarchie typographique
- `docs/design/status-language.md` — Vocabulaire de statut et `UNKNOWN != ZERO`
- `docs/design/components.md` — Composants et leur hiérarchie de rendu
- `docs/design/density.md` — Niveaux de densité et leur impact sur la hiérarchie
- `docs/design/terminal-compatibility.md` — Compatibilité par largeur
- `docs/architecture/architecture-overview.md` — Architecture système et couches
- `docs/architecture/state-contracts.md` — `TuiState` et ses sélecteurs
- `docs/state/domain-state.md` — États de domaine (12 types)
- `docs/state/presentation-state.md` — Présentation state et adapters
- `docs/state/unknown-data.md` — Protocole `UNKNOWN != ZERO` détaillé
- Prompt `02-ux-information-architecture.md` — Source du mandat
