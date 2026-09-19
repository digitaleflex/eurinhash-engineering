# Command Palette — Palette de Commandes

> Palette de commandes keyboard-first pour le HashCode Terminal.
> Coexiste avec le système de commandes/keymap existant d'OpenCode.
> **Règle fondamentale** : `Commands must have predictable focus behavior`

---

## 1. Vue d'ensemble

La command palette est l'interface centrale pour la navigation et la configuration du terminal. Elle est accessible au clavier et permet de rechercher, exécuter et configurer toutes les commandes du système.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COMMAND PALETTE                                    │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  Déclenchement     │  Ctrl+K  ou  /                           │  │
│  │  Navigation        │  ↑↓ pour naviguer dans la liste        │  │
│  │  Sélection         │  Enter pour exécuter                     │  │
│  │  Annulation        │  Escape pour fermer                      │  │
│  │  Complétion        │  Tab pour compléter                      │  │
│  │  Recherche         │  Tapez pour filtrer                      │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                        │
│  Coexistence : Ne remplace PAS le système de commandes existant      │
│  Focus : Prévisible et cohérent                                        │
│  Escape : Toujours cohérent                                          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Déclenchement

### 2.1 Raccourcis d'Ouverture

| Raccourci | Action | Contexte |
|-----------|--------|----------|
| `Ctrl+K` | Ouvrir la command palette | TOUT contexte |
| `/` | Début de commande | Dans COMMAND zone |
| `Ctrl+D` | Ouvrir le sélecteur de densité | TOUT contexte |

**Règles** :
1. `Ctrl+K` fonctionne depuis **N'IMPORTE QUELLE** zone
2. `/` n'est actif que dans la zone COMMAND
3. `Ctrl+D` ouvre un sous-menu dédié à la densité
4. `Ctrl+K` et `/` NE DOIVENT PAS entrer en conflit

### 2.2 Fermeture

| Raccourci | Action |
|-----------|--------|
| `Escape` | Fermer la palette → retour à la zone précédente |
| `Ctrl+C` | Fermer la palette → interruption si en cours |
| `Click` | Fermer la palette (souris secondaire) |

**Règles** :
1. `Escape` est le retour universel — il ferme la palette ET revient à la zone précédente
2. La pile de navigation est préservée — `Escape` revient exactement à la position précédente
3. `Ctrl+C` ferme la palette et interrompt si un stream est en cours
4. La fermeture NE perMET PAS de perte de données

---

## 3. Navigation dans la Palette

### 3.1 Navigation au Clavier

| Touche | Action |
|--------|--------|
| `↑` | Item précédent |
| `↓` | Item suivant |
| `Enter` | Exécuter / Confirmer l'item sélectionné |
| `Tab` | Complétion automatique de la commande en cours |
| `Shift+Tab` | Complétion inverse |
| `Escape` | Fermer la palette |
| `Ctrl+C` | Fermer + interruption |

### 3.2 Recherche et Filtrage

1. Taper un texte filtre la liste des commandes
2. La recherche est **incrémentale** — les résultats se mettent à jour à chaque caractère
3. `↑` / `↓` naviguent entre les résultats filtrés
4. `Enter` exécute la commande sélectionnée
5. `Escape` annule la recherche (sans exécuter)

**Comportement de filtrage** :
- La recherche filtre par nom de commande ET par description
- Les commandes partielles sont affichées
- `UNKNOWN` dans les résultats de recherche → `[? UNKNOWN]` — jamais `0` ou vide
- Si aucun résultat : afficher `[? UNKNOWN] · No matching commands`

### 3.3 Complétion Automatique

| Déclencheur | Comportement |
|-------------|-------------|
| `Tab` dans la palette | Complétion automatique de la commande en cours |
| `Shift+Tab` | Complétion inverse |
| Plusieurs correspondances | Afficher un `SelectList` des options |
| `Enter` sur la complétion | Sélectionner l'option |
| `Escape` | Annuler la complétion |

**Exemples** :
```
› /den<Tab> → /density STANDARD
› /ter<Tab> → /terminal info
› /th<Tab> → /theme dark (avec SelectList : dark, light)
```

---

## 4. Structure de la Palette

### 4.1 Catégories de Commandes

```
COMMAND PALETTE
├── 📁 Navigation
│   ├── /focus workspace
│   ├── /focus chat
│   ├── /focus agent
│   ├── /focus activity
│   ├── /focus command
│   ├── /density [MINIMAL|STANDARD|...]
│   └── /theme [dark|light]
├── 📁 Session
│   ├── /new session
│   ├── /delete session [id]
│   ├── /rename session [id]
│   ├── /share session [id]
│   └── /compact [id]
├── 📁 Agent
│   ├── /reroute [worker]
│   ├── /refresh
│   └── /toggle [worker]
├── 📁 Activity
│   ├── /filter [type]
│   ├── /clear
│   └── /export
├── 📁 Configuration
│   ├── /keybindings list
│   ├── /keybindings set [key] [action]
│   ├── /preferences
│   └── /settings
├── 📁 Terminal
│   ├── /terminal info
│   ├── /terminal size
│   └── /terminal reset
├── 📁 Aide
│   ├── /help
│   ├── /shortcuts
│   └── /about
└── 📁 Système
    ├── /undo
    ├── /redo
    ├── /clear history
    └── /exit
```

### 4.2 Format d'Affichage des Commandes

```
COMMAND PALETTE
─────────────────────
  /density STANDARD    Change density level
  /theme dark          Switch to dark mode
  /focus workspace     Focus workspace zone
  /new session         Create a new session
─────────────────────
  ↑↓ Navigate  Enter Execute  Tab Complete  Escape Close
```

---

## 5. Focus Behavior

### 5.1 Règles de Focus Prédictibles

1. **La command palette TOUJOURS le focus** — quand elle s'ouvre, le focus est sur le champ de recherche
2. **Le focus est visible** — `borderActive` + `accent` sur le champ de recherche
3. **Le focus reste dans la palette** pendant la navigation
4. **`Enter` exécute** — le focus reste dans la palette (ou ferme après exécution)
5. **`Escape` retourne** — le focus revient à la zone qui était active AVANT l'ouverture
6. **Le focus est mémorisé** — quand on rouvre la palette, on retrouve la position précédente

### 5.2 Focus après Exécution

| Type de commande | Focus après exécution |
|------------------|-----------------------|
| Navigation (`/focus workspace`) | Zone cible (WORKSPACE) |
| Configuration (`/density STANDARD`) | Zone précédente + toast confirmant |
| Session (`/new session`) | CHAT/SESSION avec la nouvelle session |
| Agent (`/reroute worker`) | Zone AGENT |
| Activity (`/clear`) | Zone ACTIVITY |
| Aide (`/help`) | Palette reste ouverte sur l'aide |
| Système (`/undo`) | Zone précédente + toast confirmant |

### 5.3 Focus et Commandes Dangereuses

Les commandes destructrices (`/delete`, `/compact`, `/reset`) suivent les règles de permission :

1. `Enter` sur une commande destructive → `Dialog` de type `Confirm`
2. Le focus est sur le bouton `Yes` par défaut
3. `Enter` = Yes, `N` = No, `Escape` = Cancel
4. Le focus revient à la zone précédente après la réponse
5. `dangerous actions require existing permission/confirmation semantics`

---

## 6. Interaction avec le Système Existant

### 6.1 Coexistence avec OpenCode Commands

La command palette **coexiste** avec le système de commandes d'OpenCode :

| Aspect | Command Palette | OpenCode Native |
|--------|----------------|-----------------|
| Déclenchement | `Ctrl+K` | `/`, `Ctrl+K` |
| Navigation | `↑↓` + `Enter` | `↑↓` + `Enter` |
| Complétion | `Tab` | `Tab` |
| Annulation | `Escape` | `Escape` |
| Stockage | `opencode.jsonc` | Plugin system |

**Règles** :
1. `Ctrl+K` OUVRE la palette — il ne remplace PAS la commande existante
2. `/` dans COMMAND zone ouvre la saisie de commande native
3. La palette EST un SUR-ensemble des commandes — elle ne duplique PAS
4. Les commandes natives restent accessibles directement
5. La palette ajoute la **recherche** et le **filtre** — c'est une couche d'abstraction

### 6.2 Coexistence avec Keymap

Le système de keymap existant (`KeybindsConfig`) reste fonctionnel :

1. `Ctrl+K` est le raccourci de la palette — il est enregistré dans le keymap
2. Les keymaps personnalisés de l'utilisateur sont respectés
3. `Ctrl+K` ne DOIT PAS être retiré du keymap existant
4. La palette utilise les même bindings que le keymap pour la navigation interne
5. `UNKNOWN` dans le keymap → `[? UNKNOWN]` — jamais `0`

### 6.3 Commandes Disponibles dans la Palette

Toutes les commandes du système sont accessibles via la palette :

| Catégorie | Commandes |
|-----------|-----------|
| Layout | `/density`, `/terminal`, `/theme` |
| Session | `/new`, `/delete`, `/rename`, `/share`, `/compact` |
| Agent | `/reroute`, `/refresh`, `/toggle` |
| Activity | `/filter`, `/clear`, `/export` |
| Configuration | `/keybindings`, `/preferences`, `/settings` |
| Navigation | `/focus workspace/chat/agent/activity/command` |
| Interaction | `/undo`, `/redo`, `/history`, `/clear` |
| Système | `/help`, `/shortcuts`, `/about`, `/exit` |

---

## 7. Commandes Système par Catégorie

### 7.1 Commandes de Layout

| Commande | Description | Focus après |
|----------|-------------|-------------|
| `/density MINIMAL` | Passer en MINIMAL | Zone précédente + toast |
| `/density STANDARD` | Passer en STANDARD | Zone précédente + toast |
| `/density DEVELOPER` | Passer en DEVELOPER | Zone précédente + toast |
| `/density OBSERVER` | Passer en OBSERVER | Zone précédente + toast |
| `/density DEBUG` | Passer en DEBUG | Zone précédente + toast |
| `/density` | Afficher le niveau actuel | Palette reste ouverte |

### 7.2 Commandes de Thème

| Commande | Description | Focus après |
|----------|-------------|-------------|
| `/theme dark` | Activer le mode dark | Zone précédente + toast |
| `/theme light` | Activer le mode light | Zone précédente + toast |
| `/theme` | Afficher le thème actuel | Palette reste ouverte |

### 7.3 Commandes de Session

| Commande | Description | Focus après |
|----------|-------------|-------------|
| `/new session` | Créer une nouvelle session | CHAT/SESSION |
| `/delete session [id]` | Supprimer une session (confirmation) | Zone précédente |
| `/rename session [id]` | Renommer une session | WORKSPACE |
| `/share session [id]` | Partager une session | WORKSPACE |
| `/compact [id]` | Compacter une session (confirmation) | CHAT/SESSION |

### 7.4 Commandes de Navigation

| Commande | Description | Focus après |
|----------|-------------|-------------|
| `/focus workspace` | Focus WORKSPACE | WORKSPACE |
| `/focus chat` | Focus CHAT/SESSION | CHAT/SESSION |
| `/focus agent` | Focus AGENT | AGENT |
| `/focus activity` | Focus ACTIVITY | ACTIVITY |
| `/focus command` | Focus COMMAND | COMMAND |

### 7.5 Commandes d'Interaction

| Commande | Description | Focus après |
|----------|-------------|-------------|
| `/undo` | Annuler la dernière action | Zone précédente + toast |
| `/redo` | Rétablir la dernière action annulée | Zone précédente + toast |
| `/history` | Afficher l'historique des commandes | Palette (dans l'historique) |
| `/clear` | Effacer le flux d'activity | ACTIVITY |

### 7.6 Commandes de Configuration

| Commande | Description | Focus après |
|----------|-------------|-------------|
| `/keybindings list` | Afficher tous les raccourcis | Palette |
| `/keybindings set [key] [action]` | Définir un raccourci | Palette |
| `/preferences` | Afficher les préférences | Palette |
| `/settings` | Ouvrir les paramètres | Settings panel |

---

## 8. Patterns de Commandes

### 8.1 Pattern de Complétion

```
› /den<Tab> → /density STANDARD
› /ter<Tab> → /terminal info
› /th<Tab> → /theme dark (SelectList : dark, light)
› /new<Tab> → /new session
```

### 8.2 Pattern de Confirmation

```
› /delete session abc
╭─ CONFIRM ──────────────────────────────────╮
│ Delete session "abc"?                         │
│                                               │
│ [Yes] [No] [Cancel]                           │
╰──────────────────────────────────────────────╯
```

### 8.3 Pattern de Sélection

```
› /theme
╭─ SELECT THEME ─────────────────────────────╮
│ dark   (default)                              │
│ light                                         │
│                                               │
│ [Select] [Cancel]                             │
╰──────────────────────────────────────────────╯
```

### 8.4 Pattern de Filtrage

```
› /d<Tab>
  /density MINIMAL    Change density level
  /density STANDARD   Change density level
  /density DEVELOPER  Change density level
  /density OBSERVER   Change density level
  /density DEBUG      Change density level
```

---

## 9. Règles Fondamentales de la Command Palette

1. **Commands must have predictable focus behavior** — le focus est toujours prévisible
2. **`Escape` est cohérent** — retour à la zone précédente, jamais de comportement inattendu
3. **Long-running operations must visibly indicate state** — les commandes longues affichent un progress
4. **Dangerous actions require confirmation** — `Confirm` dialog pour les commandes destructrices
5. **Les préférences ne fuient PAS dans le domain logic** — les commandes de configuration changent la présentation uniquement
6. **`UNKNOWN != ZERO`** — dans les résultats de recherche et les confirmations
7. **La palette coexiste** avec le système de commandes existant — elle ne le remplace pas
8. **`Tab` = complétion** — jamais de navigation hors de la palette
9. **La recherche est incrémentale** — les résultats se mettent à jour à chaque caractère
10. **Le focus est mémorisé** — on retrouve sa position après réouverture

---

## 10. Références

- `docs/ux/interaction-patterns.md` — Patterns de sélection et confirmation
- `docs/ux/navigation-model.md` — Navigation et focus behavior
- `docs/design/components.md` — `CommandSurface`, `Dialog`, `SelectionList`
- `docs/design/components.md` — `Prompt` et `CommandSurface`
- `docs/ui/component-map.md` — Composants de la zone COMMAND
- `docs/design/keybind-display.md` — Affichage des raccourcis
- `docs/state/presentation-state.md` — `TuiState` et `TuiDialogStack`
- `docs/architecture/architecture-overview.md` — `api.router` et navigation
- Prompt `08-interaction-personalization.md` — Source du mandat Phase 08
