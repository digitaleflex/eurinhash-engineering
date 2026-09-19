# Navigation Model — Modèle de Navigation entre Zones

> **Source**: Docs architecture/design/state existants, Prompt `02-ux-information-architecture.md`
> **Date**: 2026-09-19
> **Règle**: « Preserve keyboard-first interaction » — la navigation est toujours au clavier d'abord

---

## 1. Vue d'ensemble

La navigation entre les 5 zones du HashCode Terminal est gérée par un système de **routage** basé sur des clés de raccourci (`keymap`). Le TUI utilise `TuiRouteCurrent` pour suivre la position actuelle dans l'application.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        NAVIGATION ROUTE                              │
│                                                                      │
│  ┌──────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────┐ │
│  │HOME  │───▶│WORKSPACE │───▶│ CHAT/    │───▶│ AGENT    │───▶│ACTIVITY│ │
│  │      │    │          │    │ SESSION  │    │          │    │      │ │
│  └──────┘    └──────────┘    └──────────┘    └──────────┘    └──────┘ │
│     │                        │                        │             │
│     │                        ▼                        ▼             │
│     │                  ┌──────────┐    ┌──────────┐               │
│     │                  │ COMMAND  │    │TOOLS     │               │
│     │                  │  (bas)   │    │(context) │               │
│     │                  └──────────┘    └──────────┘               │
│     │                                                               │
│     └──── OBSERVABILITY (overlay, accessible depuis n'importe où)    │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  Raccourcis: Tab | Enter | Esc | / | Ctrl+...                 │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Types de Navigation

### 2.1 Navigation Globale (zones principales)

| Raccourci | Action | De → Vers |
|-----------|--------|-----------|
| `Tab` | Focus suivant | WORKSPACE → CHAT → AGENT → ACTIVITY → COMMAND |
| `Shift+Tab` | Focus précédent | COMMAND → ACTIVITY → AGENT → CHAT → WORKSPACE |
| `Ctrl+W` | Focus WORKSPACE | N'importe où → WORKSPACE |
| `Ctrl+C` | Focus CHAT/SESSION | N'importe où → CHAT/SESSION |
| `Ctrl+A` | Focus AGENT | N'importe où → AGENT |
| `Ctrl+E` | Focus ACTIVITY | N'importe où → ACTIVITY |
| `Ctrl+I` | Focus COMMAND | N'importe où → COMMAND |
| `Alt+1..5` | Zone directe | `Alt+1`=WORKSPACE, `Alt+2`=CHAT, `Alt+3`=AGENT, `Alt+4`=ACTIVITY, `Alt+5`=COMMAND |

### 2.2 Navigation Contextuelle (au sein d'une zone)

#### WORKSPACE
| Raccourci | Action |
|-----------|--------|
| `↑` / `↓` | Naviguer entre sessions dans la sidebar |
| `Enter` | Ouvrir une session → CHAT/SESSION |
| `Ctrl+N` | Nouvelle session |
| `Ctrl+Shift+T` | Nouveau todo |
| `F2` | Renommer session |
| `Del` | Supprimer session |

#### CHAT/SESSION
| Raccourci | Action |
|-----------|--------|
| `↑` / `↓` | Naviguer dans l'historique des messages |
| `PgUp` / `PgDn` | Scroll dans le contexte |
| `Ctrl+←` / `Ctrl+→` | Saute entre parts (text, reasoning, tool) |
| `Ctrl+Space` | Activer/désactiver le mode verbose |
| `Ctrl+R` | Re-play d'une part |
| `Escape` | Annuler l'action en cours → retour au prompt |

#### AGENT
| Raccourci | Action |
|-----------|--------|
| `↑` / `↓` | Naviguer entre workers dans le status bar |
| `Enter` | Détailler un worker |
| `R` | Re-router le prochain request vers un autre worker |
| `Ctrl+R` | Rafraîchir le statut des workers |
| `Space` | Activer/désactiver un worker |

#### ACTIVITY
| Raccourci | Action |
|-----------|--------|
| `↑` / `↓` | Naviguer dans le flux d'événements |
| `F` | Filtrer par type d'événement |
| `C` | Copier l'événement dans le presse-papiers |
| `E` | Explorer le détail d'un événement |
| `Escape` | Fermer le panneau d'activity |

#### COMMAND
| Raccourci | Action |
|-----------|--------|
| `Ctrl+K` | Command palette |
| `/` | Début de commande |
| `↑` / `↓` | Historique des commandes |
| `Tab` | Complétion automatique |
| `Enter` | Soumettre la commande |
| `Escape` | Annuler la saisie |
| `Ctrl+D` | Changer le niveau de densité |

---

## 3. Structure de Routage

### 3.1 `TuiRouteCurrent`

```typescript
type TuiRouteCurrent =
  | { name: "home" }
  | { name: "session"; params: { sessionID: string; prompt?: unknown } }
  | { name: string; params?: Record<string, unknown> };
```

**Routes** :
| Route | Nom | Params | Description |
|-------|-----|--------|-------------|
| `/` | `home` | — | Page d'accueil, vue d'ensemble |
| `/session/:sessionID` | `session` | `sessionID`, `prompt?` | Conversation active |
| `/workspace` | `workspace` | — | Gestion des projets |
| `/agent` | `agent` | — | Orchestration workers |
| `/activity` | `activity` | — | Flux d'événements |
| `/command` | `command` | — | Ligne de commande |
| `/settings` | `settings` | — | Configuration (plugin) |

### 3.2 Navigation entre zones via `api.router`

Le TuiPlugin utilise `api.router` pour naviguer :

```typescript
// Naviguer vers une session spécifique
api.router.navigate("session", { sessionID: "abc", prompt: "hello" });

// Retourner à la home
api.router.navigate("home");

// Ouvrir une commande
api.router.navigate("command", { command: "/density STANDARD" });
```

---

## 4. Hiérarchie de Navigation par Mode

### 4.1 MINIMAL

| Priorité | Zone | Raccourci |
|----------|------|-----------|
| 1 | CHAT/SESSION | Toujours visible, focus par défaut |
| 2 | COMMAND | En bas, accessible |
| 3 | WORKSPACE | Masquée sauf hamburger |
| 4 | AGENT | Status bar réduit |
| 5 | ACTIVITY | Non affiché |

**Navigation** : `Tab` ne fait que CHAT ↔ COMMAND. WORKSPACE et AGENT sont accessibles via `Alt+1` et `Alt+3`. ACTIVITY est inaccessible.

### 4.2 STANDARD (défaut)

| Priorité | Zone | Raccourci |
|----------|------|-----------|
| 1 | CHAT/SESSION | Focus par défaut, surface maximale |
| 2 | WORKSPACE | Sidebar gauche, `Ctrl+W` |
| 3 | AGENT | Sidebar gauche, `Ctrl+A` |
| 4 | COMMAND | En bas, `Ctrl+I` |
| 5 | ACTIVITY | Déroulant, `Ctrl+E` |

**Navigation** : `Tab` cycle WORKSPACE → CHAT → AGENT → ACTIVITY → COMMAND. Toutes les zones sont accessibles.

### 4.3 DEVELOPER

| Priorité | Zone | Raccourci |
|----------|------|-----------|
| 1 | CHAT/SESSION | Focus par défaut |
| 2 | COMMAND | Accessible |
| 3 | WORKSPACE | Sidebar réduite |
| 4 | AGENT | Status bar compact |
| 5 | ACTIVITY | Contextuel |

**Navigation** : `Tab` cycle CHAT → COMMAND → WORKSPACE → AGENT. ACTIVITY est contextuel (apparaît quand un événement se produit).

### 4.4 OBSERVER

| Priorité | Zone | Raccourci |
|----------|------|-----------|
| 1 | ACTIVITY | Focus par défaut, surface maximale |
| 2 | CHAT/SESSION | Sidebar réduite |
| 3 | AGENT | Status bar détaillé |
| 4 | WORKSPACE | Contextuel |
| 5 | COMMAND | Accessible |

**Navigation** : `Tab` cycle ACTIVITY → AGENT → CHAT → WORKSPACE → COMMAND. ACTIVITY est la zone principale.

### 4.5 DEBUG

| Priorité | Zone | Raccourci |
|----------|------|-----------|
| 1 | ACTIVITY | Flux complet d'événements |
| 2 | CHAT/SESSION | Données brutes |
| 3 | AGENT | Données brutes |
| 4 | WORKSPACE | Données brutes |
| 5 | COMMAND | Données brutes |

**Navigation** : Toutes les zones affichent des données brutes. `Tab` cycle dans l'ordre alphabétique. Pas de masquage.

---

## 5. Navigation par Contexte d'Event

### 5.1 Navigation automatique déclenchée par des événements

| Événement | Navigation déclenchée |
|-----------|----------------------|
| `permission.asked` | → COMMAND (focus sur la réponse) |
| `question.asked` | → COMMAND (focus sur la réponse) |
| `session.status` → `{ type: "busy" }` | → CHAT/SESSION (focus sur le stream) |
| `session.status` → `{ type: "idle" }` | → COMMAND (focus sur le prompt) |
| `EventSessionNextToolCalled` | → ACTIVITY (afficher l'outil) |
| `EventMcpToolsChanged` | → TOOLS (afficher les outils MCP) |
| `EventWorkspaceFailed` | → WORKSPACE (afficher l'erreur) |
| `EventSessionError` | → CHAT/SESSION (afficher l'erreur) |

### 5.2 Navigation par contexte de focus

Quand l'utilisateur est en mode THINKING (agent en cours de raisonnement), la navigation se modifie :

- `Tab` ne change pas de zone — le stream est prioritaire
- `Escape` interrupte le stream → retour au prompt
- `Ctrl+C` force l'arrêt → retour au prompt

---

## 6. Gestion du Focus

### 6.1 Focus States par Composant

| Composant | Focus actif | Focus passif | Focus perdu |
|-----------|------------|--------------|-------------|
| `Sidebar` | `borderActive` + `accent` | `borderSubtle` + `textMuted` | `border` + `text` |
| `Prompt` | `borderActive` + `accent` | `border` + `text` | `border` + `textMuted` |
| `Table` | `backgroundElement` + `selectedListItemText` | `background` + `text` | `background` + `textMuted` |
| `Dialog` | `borderActive` + `backgroundMenu` | `border` + `backgroundPanel` | — |
| `CommandSurface` | `borderActive` + `accent` | `border` + `backgroundMenu` | `border` + `backgroundMenu` |

### 6.2 Règles de Focus

1. **Un seul focus à la fois** — le focus est exclusif entre les zones
2. **Le focus suit l'action** — quand un event se produit, le focus se déplace vers la zone concernée
3. **Escape libère le focus** — retour au zone précédente
4. **Le focus est mémorisé** — quand on revient dans une zone, on retrouve la position précédente
5. **Keyboard-first** — la navigation se fait toujours au clavier ; la souris est secondaire

---

## 7. Back Navigation

### 7.1 `Escape` comme retour universel

| Contexte | Action de `Escape` |
|----------|-------------------|
| Dans COMMAND | Annuler la saisie → retour au prompt vide |
| Dans un Dialog | Fermer le dialog → retour à la zone précédente |
| Dans un stream THINKING | Interrupt → retour au prompt |
| Dans ACTIVITY | Fermer le panneau → retour à la zone précédente |
| Dans WORKSPACE | Fermer la sidebar → retour à CHAT |
| Dans tout contexte | `Esc` = retour à la zone de base |

### 7.2 Pile de navigation

```
[HOME] → [WORKSPACE] → [SESSION:abc] → [COMMAND:/density STANDARD]
  ↑                                           │
  └───────────────────────────────────────────┘
  (Escape revient à SESSION:abc)
```

La pile est gérée par `api.router` et est limitée à 10 entrées pour éviter la surcharge mémoire.

---

## 8. Navigation par Mode — Tableau Récapitulatif

| Zone | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|------|---------|----------|-----------|----------|-------|
| WORKSPACE | Masquée (hamburger) | Sidebar | Sidebar réduite | Contextuel | Données brutes |
| CHAT/SESSION | Maximal | Maximal | Maximal | Réduite | Données brutes |
| AGENT | Status bar | Sidebar | Status bar | Status bar détaillé | Données brutes |
| ACTIVITY | Inaccessible | Déroulant | Contextuel | Maximal | Maximal |
| COMMAND | Accessible | Accessible | Accessible | Accessible | Données brutes |
| OBSERVABILITY | Overlay minimal | Overlay | Overlay | Overlay détaillé | Données brutes |
| TOOLS | Contextuel | Contextuel | Contextuel | Contextuel | Contextuel |

---

## 9. Règles de Navigation

### 9.1 Règles Fondamentales

1. **Keyboard-first** : La navigation est toujours au clavier d'abord
2. **Un seul focus** : Un seul élément a le focus à un moment donné
3. **Focus suit l'action** : Le focus se déplace vers la zone affectée par un événement
4. **Escape = retour** : `Escape` est universellement le retour
5. **Tab = cycle** : `Tab` cycle dans l'ordre de priorité du mode courant
6. **Pas de horizontal scroll** : La navigation ne nécessite jamais de scroll horizontal
7. **Pas de perte de données** : La navigation ne fait jamais perdre du contenu

### 9.2 Règles par Mode

| Règle | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|-------|---------|----------|-----------|----------|-------|
| Sidebar visible | Non | Oui | Oui | Contextuel | Oui |
| ACTIVITY accessible | Non | Oui | Contextuel | Oui | Oui |
| OBSERVABILITY visible | Non | Overlay | Overlay | Overlay | Données brutes |
| TOOLS contextuel | Oui | Oui | Oui | Oui | Oui |

### 9.3 Règles de Transition

- Les transitions entre zones DOIVENT être fluides (pas de flicker)
- La transition ne doit PAS perdre le contenu de la zone précédente
- Le focus DOIT être clairement indiqué dans la zone d'arrivée
- `thinkingOpacity` ne change PAS pendant la navigation
- `UNKNOWN` reste `UNKNOWN` pendant la navigation

---

## 10. Références

- `docs/architecture/architecture-overview.md` — `TuiRouteCurrent` et `api.router`
- `docs/design/components.md` — Composants de navigation (Sidebar, Menu, Breadcrumb)
- `docs/design/terminal-compatibility.md` — Adaptation par largeur de la navigation
- `docs/design/density.md` — Navigation par niveau de densité
- `docs/state/presentation-state.md` — `TuiRouteCurrent`, `TuiDialogStack`
- `docs/state/event-catalog.md` — Événements déclenchant la navigation
- Prompt `02-ux-information-architecture.md` — Source du mandat
