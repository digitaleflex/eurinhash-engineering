# Information Architecture — Phase 02

> **Source**: Prompt `02-ux-information-architecture.md`, docs architecture/design/state existants
> **Date**: 2026-09-19
> **Version OpenCode**: 1.18.31
> **Règle fondamentale**: `UNKNOWN != ZERO` — jamais confondre absence de donnée avec valeur nulle

---

## 1. Vue d'ensemble

L'architecture informationnelle du HashCode Terminal organise l'information en **5 zones** distinctes. Chaque zone a un périmètre sémantique, un niveau de priorité visuelle, et des règles d'affichage propres.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         HASHCODE TERMINAL                                    │
│                                                                              │
│  ┌──────────┐  ┌──────────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │ WORKSPACE │  │ CHAT/SESSION │  │  AGENT   │  │ ACTIVITY │  │ COMMAND  │  │
│  │          │  │              │  │          │  │          │  │          │  │
│  │ Projet   │  │ Conversation │  │ Workers  │  │ Événements│ │ Entrée   │  │
│  │ Fichiers │  │ Messages     │  │ Status   │  │ Activité │  │ Commandes│  │
│  │ Git      │  │ Parts        │  │ Routing  │  │ Logs     │  │          │  │
│  └──────────┘  └──────────────┘  └──────────┘  └──────────┘  └──────────┘  │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                         OBSERVABILITY (overlay)                        │   │
│  │  Métriques temps réel | Latence | Throughput | Coût | Risk/Sensibility │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                          TOOLS (contextuel)                            │   │
│  │  MCP | LSP | Permissions | Questions | Todo                          │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Les 5 Zones — Définition

### 2.1 WORKSPACE

**Sémantique** : Le projet en cours, son contexte filesystem et VCS.

**Contenu** :
| Donnée | Source | Présentation |
|--------|--------|-------------|
| Répertoire projet | `path.directory` | Chemin absolu, `mono` |
| Branche Git | `vcs.branch` | Nom de branche ou `null` |
| Fichiers modifiés | `session.diff(id)` | `TuiSidebarFileItem[]` |
| Todos | `session.todo(id)` | `TuiSidebarTodoItem[]` |
| Workspace ID | `Workspace.id` | Identifiant unique |
| Statut workspace | `EventWorkspaceStatus` | `connected \| connecting \| disconnected \| error` |

**Priorité visuelle** : Haute — toujours visible dans la sidebar.

**Largeur minimale** : 20 colonnes (icônes seul) ; 80+ pour labels complets.

**États** :
- `connected` : `[● ACTIVE]`
- `connecting` : `[◐ THINKING]`
- `disconnected` : `[⊘ DISCONNECTED]`
- `error` : `[✗ ERROR]`
- Aucun workspace : `[? UNKNOWN]`

**Règle UNKNOWN** : `branch: null` signifie « pas de git » — ne jamais afficher `""`. `Workspace.branch: null` ≠ `Workspace.branch: ""`.

---

### 2.2 CHAT / SESSION

**Sémantique** : La conversation centrale — l'échange utilisateur ↔ agent. C'est le cœur de l'expérience.

**Contenu** :
| Donnée | Source | Présentation |
|--------|--------|-------------|
| Messages | `session.messages(id)` | `Message[]` — User/Assistant |
| Parts | `part(messageID)` | `Part[]` — text, reasoning, tool, step |
| Statut session | `session.status(id)` | `SessionStatus` : idle/busy/retry |
| Modèle actif | `Session.model` | `ModelRef` |
| Agent actif | `Session.agent` | Nom de l'agent |
| Coût | `Session.cost` (v2) | `$X.XX` ou `N/A` |
| Tokens | `Session.tokens` (v2) | `{input, output, reasoning, cache}` |
| Erreur | `EventSessionError` | Message d'erreur |

**Priorité visuelle** : **Maximum** — occupe le centre du terminal, la plus grande surface d'écran.

**Largeur minimale** : 60 colonnes pour le prompt ; 80+ pour le rendu complet des parts.

**Flux de vie** :
```
session.created → session.status(busy) → [message.part.* stream] → session.status(idle)
  → [compaction si nécessaire] → session.compacted
```

**États de chaque message/part** :
- `text.started` → `text.delta` → `text.ended`
- `reasoning.started` → `reasoning.delta` → `reasoning.ended`
- `tool.called` → `tool.progress` → `tool.success` | `tool.failed`
- `step.started` → [parts] → `step.ended` | `step.failed`
- `compaction.started` → `compaction.delta` → `compaction.ended`

**Règle UNKNOWN** : `cost: undefined` ≠ `cost: 0`. `tokens: undefined` ≠ `tokens: {input: 0}`. `session.get(id)` retourne `undefined` si inexistant — jamais un objet session vide.

---

### 2.3 AGENT

**Sémantique** : L'orchestration des workers EurinHash — routing, statut des workers, modèle sélectionné.

**Contenu** :
| Donnée | Source | Présentation |
|--------|--------|-------------|
| Workers disponibles | `free-models.json` | Liste des workers + statut |
| Worker actif | `agent/eurinhash.md` | Agent en cours d'exécution |
| Modèle sélectionné | `ModelRef` | Provider + model ID |
| Routing EV | `scripts/route.py` | Score de route |
| Latence worker | Calculé | `timestamp` diff |
| Throughput | Calculé | `tokens / elapsed` |
| Coût par worker | Dérivé | `$X.XX` |
| Risque/Sensibilité | À définir | `guard.ts` / eurinhash |

**Priorité visuelle** : Moyenne-haute — visible dans la sidebar ou le status bar.

**Largeur minimale** : 40 colonnes pour le status bar ; 80+ pour le détail complet.

**Workers et leur statut** :
| Worker | Modèle | Statut possible |
|--------|--------|----------------|
| `worker-codestral` | `laguna-s-2.1:free` | ACTIVE, IDLE, THINKING, ERROR, DISCONNECTED, UNKNOWN |
| `worker-groq` | `qwen3.8-27b` | ACTIVE, IDLE, THINKING, ERROR, DISCONNECTED, UNKNOWN |
| `worker-novita` | `ling-3.0-flash-sante` | ACTIVE, IDLE, THINKING, ERROR, DISCONNECTED, UNKNOWN |
| `worker-zhipu` | `glm-4.7-flash` | ACTIVE, IDLE, THINKING, ERROR, DISCONNECTED, UNKNOWN |
| `worker-google` | `gemini-2.5-flash` | ACTIVE, IDLE, THINKING, ERROR, DISCONNECTED, UNKNOWN |
| `worker-pollinations` | `openai` | ACTIVE, IDLE, THINKING, ERROR, DISCONNECTED, UNKNOWN |

**Règle de fallback** : `codestral → groq → novita → zhipu → google → ollama → mammouth(payant)`

**Règle UNKNOWN** : Worker dont le statut n'a jamais été interrogé = `[? UNKNOWN]`. Jamais `[○ IDLE]` par défaut. `UNKNOWN != ZERO` — un worker sans donnée n'est pas un worker inactif.

---

### 2.4 ACTIVITY / EVENTS

**Sémantique** : Le flux chronologique des événements système — ce qui se passe en arrière-plan.

**Contenu** :
| Donnée | Source | Présentation |
|--------|--------|-------------|
| Événements session | `EventSession*` | Création, statut, compactage |
| Événements message | `EventMessage*` | Mise à jour, suppression |
| Événements outils | `EventSessionNextTool*` | Appel, succès, échec |
| Événements permission | `EventPermission*` | Demande, réponse |
| Événements MCP | `EventMcp*` | Outils changés, navigateur |
| Événements LSP | `EventLsp*` | Diagnostics, mise à jour |
| Événements fichier | `EventFile*` | Édité, watcher |
| Événements Git | `EventVcsBranchUpdated` | Branche changée |
| Événements workspace | `EventWorkspace*` | Ready, failed, status |
| Événements pty | `EventPty*` | Processus terminal |
| Événements question | `EventQuestion*` | Questions utilisateur |
| Événements installation | `EventInstallation*` | Mises à jour |
| Événements commande | `EventCommandExecuted` | Commandes exécutées |
| Événements sync | `SyncEvent*` | Events durables v2 |

**Priorité visuelle** : Moyenne — visible dans un panneau déroulant ou une sidebar dédiée.

**Largeur minimale** : 40 colonnes pour la liste ; 80+ pour le détail complet.

**Classification** :
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

**Règle UNKNOWN** : `EventSessionError.error: undefined` ≠ pas d'erreur. Un événement sans payload d'erreur signifie que l'erreur n'a pas de détail — pas qu'il n'y a pas d'erreur.

---

### 2.5 COMMAND

**Sémantique** : L'entrée utilisateur — la ligne de commande et les commandes du système.

**Contenu** :
| Donnée | Source | Présentation |
|--------|--------|-------------|
| Input utilisateur | `TuiPromptInfo.input` | Texte saisi |
| Mode prompt | `TuiPromptInfo.mode` | `normal` \| `shell` |
| Historique | `history_previous` / `history_next` | Commandes précédentes |
| Commandes système | `/density`, `/terminal`, `/theme`, etc. | CommandSurface |
| Keybindings | `KeybindsConfig` | Affichage des raccourcis |
| File complet | `input_paste` | Contenu du presse-papiers |
| Soumission | `input_submit` | Envoi de la commande |

**Priorité visuelle** : **Fixe en bas** — toujours visible, ancrage visuel.

**Largeur minimale** : 20 colonnes ; 60+ pour le texte complet sans wrap.

**Modes** :
| Mode | Indicateur | Usage |
|------|-----------|-------|
| Normal | `›` | Commande standard, agent |
| Shell | `$` | Commande shell, exécution système |

**Composants** :
- `Prompt` : Zone de saisie
- `CommandSurface` : Palette de commandes
- `KeybindDisplay` : Raccourcis clavier
- `Dialog` : Confirmation, sélection, saisie
- `Toast` : Notification éphémère

**Règle UNKNOWN** : `TuiPromptInfo.mode: undefined` ≠ `"normal"`. Quand le mode n'est pas spécifié, il est `undefined` — jamais implicitement `"normal"`.

---

## 3. Modes de Densité

Chaque zone se comporte différemment selon les 5 modes de densité. Le mode est configurable via `/density` et détecté automatiquement selon la largeur du terminal.

### 3.1 Les 5 Modes

| Mode | Densité | Lignes vides | Padding | Largeur ligne | Usage |
|------|---------|-------------|---------|---------------|-------|
| `MINIMAL` | Très sparse | 2 | `space-lg` (8 cols) | 120+ | Sessions longues, monitoring |
| `STANDARD` | Équilibré | 1 | `space-md` (4 cols) | 100 | Usage quotidien (défaut) |
| `DEVELOPER` | Dense | 0 | `space-sm` (2 cols) | 80 | Débogage, inspection rapide |
| `OBSERVER` | Dense+ | 0 | `space-xs` (1 col) | 160 | Surveillance, métriques temps réel |
| `DEBUG` | Ultra-dense | 0 | 0 | Illimité | Diagnostic, traces complètes |

### 3.2 Comportement des Zones par Mode

#### WORKSPACE

| Mode | Affichage |
|------|-----------|
| MINIMAL | Sidebar masquée, hamburger uniquement |
| STANDARD | Sidebar complète avec labels |
| DEVELOPER | Sidebar réduite, labels tronqués |
| OBSERVER | Sidebar complète + métriques détaillées |
| DEBUG | Données brutes (JSON lines) |

#### CHAT/SESSION

| Mode | Affichage |
|------|-----------|
| MINIMAL | Messages uniquement, pas de métadonnées |
| STANDARD | Messages + indicateur de modèle/agent |
| DEVELOPER | Messages + toutes les métadonnées |
| OBSERVER | Messages + métriques temps réel |
| DEBUG | Données brutes de toutes les parts |

#### AGENT

| Mode | Affichage |
|------|-----------|
| MINIMAL | Status bar uniquement (`●`/`◐`/`✗`/`○`) |
| STANDARD | Status bar + noms des workers |
| DEVELOPER | Status bar compact avec temps |
| OBSERVER | Status bar détaillé avec latence/throughput |
| DEBUG | Toutes les données brutes des workers |

#### ACTIVITY

| Mode | Affichage |
|------|-----------|
| MINIMAL | Non affiché |
| STANDARD | Déroulant, événements clés uniquement |
| DEVELOPER | Contextuel, affiché quand un événement se produit |
| OBSERVER | Flux complet d'événements, surface maximale |
| DEBUG | Flux complet + données brutes de chaque événement |

#### COMMAND

| Mode | Affichage |
|------|-----------|
| MINIMAL | Prompt minimal, pas de complétion |
| STANDARD | Prompt complet, complétion automatique |
| DEVELOPER | Prompt + keybindings affichés |
| OBSERVER | Prompt + métriques contextuelles |
| DEBUG | Données brutes du prompt |

### 3.3 Règles de Mode

1. **`STANDARD` est le défaut** pour tous les utilisateurs
2. `DEVELOPER` et `OBSERVER` ne DOIVENT être accessibles qu'aux utilisateurs avertis
3. `DEBUG` NE DOIT PAS être le niveau par défaut — réservé au diagnostic
4. La détection automatique utilise la largeur du terminal
5. Le changement de mode est configurable via `/density`
6. `thinkingOpacity` est disponible à TOUS les modes
7. `UNKNOWN` et `N/A` sont affichés de manière identique à TOUS les modes
8. JAMAIS de données fabriquées pour remplir l'espace dans un mode de densité

---

## 4. Zones Additionnelles (Overlay)

### 3.1 OBSERVABILITY

**Sémantique** : Métriques temps réel — ne jamais dominer la conversation.

**Contenu** : Latence, queue, throughput, coût cumulé, risk/sensibilité, utilisation tokens.

**Règle** : `Do not let observability dominate the coding conversation.` (Règle 6 du prompt)

**Données manquantes** : Latence, queue, throughput, risk/sensibilité ne sont PAS dans `TuiState`. Elles doivent être calculées ou exposées via le serveur API.

### 3.2 TOOLS

**Sémantique** : Contexte des outils — MCP, LSP, permissions, questions, todo.

**Contenu** : Statut MCP (`TuiSidebarMcpItem[]`), statut LSP (`TuiSidebarLspItem[]`), permissions en attente, questions en attente, todo list.

**Règle** : Affichage contextuel — n'apparaît que quand pertinent (permission demandée, outil en cours, etc.).

---

## 4. Règles transversales entre zones

### 4.1 `UNKNOWN != ZERO` dans chaque zone

| Zone | Cas UNKNOWN | Affichage correct | Affichage interdit |
|------|------------|-------------------|-------------------|
| WORKSPACE | `branch: null` | `null` (pas de git) | `""` |
| CHAT | `cost: undefined` | `N/A` | `$0.00` |
| CHAT | `tokens: undefined` | `N/A` | `{input: 0}` |
| AGENT | Worker non interrogé | `[? UNKNOWN]` | `[○ IDLE]` |
| ACTIVITY | `error: undefined` dans Event | Absent — pas de détail | `""` |
| COMMAND | `mode: undefined` | Non spécifié | `"normal"` |

### 4.2 Priorité visuelle globale

```
CHAT/SESSION  ████████████████████████████████████████████████████  MAXIMUM
WORKSPACE     ████████████████████                                HAUTE
AGENT         ██████████████                                      MOYENNE-HAUTE
ACTIVITY      ██████████                                          MOYENNE
COMMAND       ████████████████████████████████████████              FIXE (bas)
OBSERVABILITY ████                                                LOW (overlay)
TOOLS         ████                                                CONTEXTUEL
```

### 4.3 Règle de la conversation centrale

> « Keep the conversation central. Do not let observability dominate the coding conversation. »

- CHAT/SESSION occupe toujours la plus grande surface
- OBSERVABILITY est un overlay — jamais la zone par défaut
- WORKSPACE et AGENT sont dans la sidebar
- COMMAND est en bas, fixe
- ACTIVITY est déroulant/contextuel

### 4.4 Règle de la stabilité

> « Keep high-frequency information stable to avoid visual noise. »

- Le statut CHAT/SESSION est stable pendant le stream
- Le statut WORKSPACE ne change que lors d'un changement de projet
- Le statut AGENT change lors du routing
- ACTIVITY est le seul flux dynamique fréquent
- COMMAND est fixe et immuable pendant l'édition

### 4.5 Règle de la raison

> « Every alert has a reason. »

Chaque alerte dans chaque zone DOIT avoir une explication :
- `[✗ ERROR] worker-google` → `Connection refused`
- `[⚠ WARNING]` → `Approchant la limite de 262K tokens`
- `[? UNKNOWN]` → `Coût de l'agent non calculé`

---

## 5. Matrice des États par Zone

| État | WORKSPACE | CHAT/SESSION | AGENT | ACTIVITY | COMMAND |
|------|-----------|-------------|-------|----------|---------|
| **initial/loading** | `[◐ THINKING]` workspace | `[◐ THINKING]` prompt | `[◐ THINKING]` worker | — | `[◐ THINKING]` input |
| **idle** | `[○ IDLE]` | `{ type: "idle" }` | `[○ IDLE]` | — | Prompt vide |
| **active** | `[● ACTIVE]` | `{ type: "busy" }` | `[● ACTIVE]` | Flux events | `[▶ RUNNING]` |
| **thinking** | — | `[◐ THINKING]` | `[◐ THINKING]` | — | — |
| **tool call** | — | `[▶ RUNNING]` tool | `[▶ RUNNING]` | `[▶ RUNNING]` tool event | — |
| **waiting for permission** | — | `[⌛ WAITING]` | `[⌛ WAITING]` | `[⌛ WAITING]` permission | — |
| **responding/streaming** | — | Text/Reasoning stream | — | Delta events | — |
| **completed** | `[✓ SUCCESS]` | `{ type: "idle" }` | `[✓ SUCCESS]` | `[✓ SUCCESS]` | — |
| **error** | `[✗ ERROR]` | `[✗ ERROR]` | `[✗ ERROR]` | `[✗ ERROR]` | — |
| **cancelled** | `[⊘ CANCELLED]` | `[⊘ CANCELLED]` | `[⊘ CANCELLED]` | `[⊘ CANCELLED]` | — |
| **unavailable** | `[⊘ DISCONNECTED]` | — | `[⊘ DISCONNECTED]` | — | — |
| **empty** | `[○ IDLE]` no files | Messages empty | `[○ IDLE]` no workers | Events empty | Prompt empty |
| **degraded/partial** | `[⚠ WARNING]` partial | `[⚠ WARNING]` partial | `[⚠ WARNING]` partial | `[⚠ WARNING]` partial | — |

---

## 6. Inventaire des Composants par Zone

### WORKSPACE
- `Sidebar` — navigation latérale
- `StatusIndicator` — statut workspace
- `Table` — fichiers modifiés
- `KeyValueList` — infos projet
- `Badge` — statut VCS

### CHAT/SESSION
- `MarkdownView` — rendu message
- `TextBlock` — texte streaming
- `ThinkingBlock` — reasoning
- `ToolCallCard` — appel outil
- `ProgressBar` — progression
- `DiffView` — diff
- `Alert` — erreur/warning
- `Spinner` — chargement

### AGENT
- `StatusBar` — barre de statut workers
- `StatusIndicator` — statut worker
- `MetricDisplay` — métriques worker
- `Table` — tableau workers
- `Alert` — alerte worker

### ACTIVITY
- `EventLog` — flux d'événements
- `Toast` — notification
- `Caption` — timestamp
- `Separator` — séparation temporelle
- `Badge` — type d'événement

### COMMAND
- `Prompt` — zone de saisie
- `CommandSurface` — palette
- `KeybindDisplay` — raccourcis
- `Dialog` — confirmation/sélection
- `Toast` — feedback
- `Alert` — alerte commande

### OBSERVABILITY (overlay)
- `MetricDisplay` — métriques
- `ProgressBar` — utilisation
- `Table` — tableau métriques
- `Panel` — panneau métriques
- `Alert` — alerte seuil

### TOOLS (contextuel)
- `SelectionList` — sélection MCP/LSP
- `Alert` — permission/question
- `StatusBar` — statut outils
- `Dialog` — confirmation permission

---

## 7. Priorité d'Implémentation

### Phase 1 — Fondation (priorité critique)
1. **CHAT/SESSION** — La conversation est le centre
2. **COMMAND** — L'entrée utilisateur est l'ancrage
3. **WORKSPACE** — Le contexte du projet

### Phase 2 — Orchestration (priorité haute)
4. **AGENT** — Le routing des workers
5. **ACTIVITY** — Le flux d'événements

### Phase 3 — Superposition (priorité moyenne)
6. **OBSERVABILITY** — Overlay métriques
7. **TOOLS** — Contexte des outils

### Phase 4 — Polish (priorité normale)
8. Toutes les combinaisons de modes
9. States complexes (degraded, partial data)
10. Accessibilité complète

---

## 8. Références

- `docs/architecture/architecture-overview.md` — Architecture système
- `docs/architecture/state-contracts.md` — Contrats d'état `TuiState`
- `docs/architecture/event-contracts.md` — Contrats d'événements
- `docs/design/design-tokens.md` — Tokens de design
- `docs/design/status-language.md` — Vocabulaire de statut
- `docs/design/components.md` — Composants terminaux
- `docs/design/density.md` — Niveaux de densité
- `docs/design/terminal-compatibility.md` — Compatibilité terminale
- `docs/state/domain-state.md` — États de domaine (12 types)
- `docs/state/event-catalog.md` — Catalogue d'événements
- `docs/state/presentation-state.md` — État de présentation
- `docs/state/unknown-data.md` — Protocole `UNKNOWN != ZERO`
- Prompt `02-ux-information-architecture.md` — Source du mandat
