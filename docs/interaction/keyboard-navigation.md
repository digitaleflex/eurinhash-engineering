# Keyboard Navigation — Navigation et Raccourcis Clavier

> **Source**: Docs ux/navigation-model, ux/interaction-patterns, design/components, ui/layout-modes
> **Date**: 2026-09-19
> **Règles fondamentales** : Keyboard-first · `Escape` = retour universel · `UNKNOWN != ZERO` · `TAB` = cycle

---

## 1. Vue d'ensemble

Le HashCode Terminal est conçu pour un **workflow au clavier d'abord**. La souris est secondaire. TOUS les patterns d'interaction sont accessibles au clavier, sans exception.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    NAVIGATION CLAVIER                                │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  CYCLE DE FOCUS          │  ACCÈS DIRECT                    │  │
│  │  Tab / Shift+Tab         │  Alt+1..5 / Ctrl+W/C/A/E/I      │  │
│  │  WORKSPACE→CHAT→AGENT    │                                │  │
│  │  →ACTIVITY→COMMAND      │                                │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  COMMANDES RAPIDES       │  NAVIGATION CONTEXTUELLE         │  │
│  │  Ctrl+K / /              │  ↑↓ / Enter / Escape             │  │
│  │  Ctrl+D (densité)        │  Ctrl+C (interrupt)              │  │
│  └──────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Règles Fondamentales

### 2.1 Règles de Navigation

1. **Keyboard-first** : TOUS les patterns sont accessibles au clavier
2. **Un seul focus** : Un seul élément a le focus à un moment donné
3. **Focus suit l'action** : Le focus se déplace vers la zone affectée
4. **`Escape` = retour** : `Escape` est universellement le retour
5. **`Tab` = cycle** : `Tab` cycle dans l'ordre de priorité du mode courant
6. **`Shift+Tab` = cycle inverse** : Retour vers la zone précédente
7. **`UNKNOWN != ZERO`** : Les raccourcis ne fabriquent jamais de données
8. **`thinkingOpacity` ne change PAS pendant la navigation** : Le focus ne modifie pas l'opacité de réflexion
9. **Pas de perte de données** : La navigation ne fait jamais perdre du contenu
10. **Pas de conflit** : Les raccourcis ne doivent pas entrer en conflit avec les raccourcis du terminal

### 2.2 Hiérarchie de Priorité des Raccourcis

Lorsqu'un raccourci est pressé, la priorité de résolution est :

```
1. Commande en cours (prompt actif)
2. Dialog/Confirmation en cours
3. Pattern de sélection actif
4. Navigation globale (zones)
5. Pattern de stream (Ctrl+C, Ctrl+Space)
6. Pattern de search (Ctrl+F, Ctrl+K)
7. Pattern de undo/redo (Ctrl+Z)
```

---

## 3. Raccourcis Globaux (Tous Modes)

### 3.1 Navigation entre Zones

| Raccourci | Action | De → Vers | Mode |
|-----------|--------|-----------|------|
| `Tab` | Focus suivant | WORKSPACE → CHAT → AGENT → ACTIVITY → COMMAND | Tous |
| `Shift+Tab` | Focus précédent | COMMAND → ACTIVITY → AGENT → CHAT → WORKSPACE | Tous |
| `Ctrl+W` | Focus WORKSPACE | N'importe où → WORKSPACE | STANDARD, DEVELOPER |
| `Ctrl+C` | Focus CHAT/SESSION | N'importe où → CHAT/SESSION | STANDARD, DEVELOPER |
| `Ctrl+A` | Focus AGENT | N'importe où → AGENT | STANDARD, DEVELOPER |
| `Ctrl+E` | Focus ACTIVITY | N'importe où → ACTIVITY | STANDARD, DEVELOPER |
| `Ctrl+I` | Focus COMMAND | N'importe où → COMMAND | Tous |
| `Alt+1` | Focus WORKSPACE | Direct | Tous |
| `Alt+2` | Focus CHAT/SESSION | Direct | Tous |
| `Alt+3` | Focus AGENT | Direct | Tous |
| `Alt+4` | Focus ACTIVITY | Direct | STANDARD, DEVELOPER, OBSERVER |
| `Alt+5` | Focus COMMAND | Direct | Tous |

### 3.2 Navigation Contextuelle (au sein d'une zone)

#### WORKSPACE

| Raccourci | Action |
|-----------|--------|
| `↑` / `↓` | Naviguer entre sessions |
| `Enter` | Ouvrir une session → CHAT/SESSION |
| `Ctrl+N` | Nouvelle session |
| `Ctrl+Shift+T` | Nouveau todo |
| `F2` | Renommer la session |
| `Del` | Supprimer la session (avec confirmation) |
| `Ctrl+S` | Partager la session |
| `Ctrl+R` | Rafraîchir le statut |

#### CHAT / SESSION

| Raccourci | Action |
|-----------|--------|
| `↑` / `↓` | Naviguer dans l'historique des messages |
| `PgUp` / `PgDn` | Scroll dans le contexte |
| `Ctrl+←` / `Ctrl+→` | Sauté entre parts (text, reasoning, tool) |
| `Ctrl+Space` | Activer/désactiver le mode verbose |
| `Ctrl+R` | Re-play d'une part |
| `Escape` | Annuler l'action en cours → retour au prompt |
| `Ctrl+C` | Arrêter le stream |
| `Ctrl+M` | Déclencher la compaction |
| `Ctrl+E` | Exporter la conversation |

#### AGENT

| Raccourci | Action |
|-----------|--------|
| `↑` / `↓` | Naviguer entre workers |
| `Enter` | Détailler un worker |
| `R` | Re-router le prochain request |
| `Ctrl+R` | Rafraîchir le statut |
| `Space` | Activer/désactiver un worker |

#### ACTIVITY

| Raccourci | Action |
|-----------|--------|
| `↑` / `↓` | Naviguer dans le flux d'événements |
| `F` | Filtrer par type d'événement |
| `C` | Copier l'événement |
| `E` | Explorer le détail d'un événement |
| `Ctrl+L` | Effacer le flux |
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
| `Ctrl+U` | Effacer l'input |
| `Ctrl+V` | Coller le contenu |
| `Ctrl+D` | Changer le niveau de densité |

---

## 4. Raccourcis par Mode de Layout

| Raccourci | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|-----------|---------|----------|-----------|----------|-------|
| `Tab` | CHAT ↔ COMMAND uniquement | WORKSPACE→CHAT→AGENT→ACTIVITY→COMMAND | CHAT→COMMAND→WORKSPACE→AGENT | ACTIVITY→AGENT→CHAT→WORKSPACE→COMMAND | Ordre alphabétique |
| `Ctrl+W/C/A/E/I` | Non disponible (sauf CHAT/COMMAND) | Disponible | Disponible | Disponible (contextuel) | Non disponible |
| `Alt+1..5` | PARTIEL (zones masquées) | Disponible | Disponible | Disponible | Disponible |
| `Ctrl+D` | Disponible | Disponible | Disponible | Disponible | Disponible |
| `Ctrl+K` | Disponible | Disponible | Disponible | Disponible | Disponible |
| `Ctrl+F` | Disponible | Disponible | Disponible | Disponible | Disponible |

### 4.1 MINIMAL — Raccourcis Réduits

En mode MINIMAL, seules les zones CHAT/SESSION et COMMAND sont accessibles :

| Raccourci | Action | Note |
|-----------|--------|------|
| `Tab` | CHAT ↔ COMMAND | Cycle limité |
| `Alt+2` | Focus CHAT/SESSION | Zone principale |
| `Alt+5` | Focus COMMAND | Accessible |
| `Ctrl+I` | Focus COMMAND | Toujours |
| `Ctrl+K` | Command palette | Toujours |
| `↑`/`↓` | Navigation dans CHAT ou COMMAND | Selon le focus |

Si l'utilisateur tente `Alt+1`, `Alt+3` ou `Alt+4` :
> `[⚠ WARNING] Zone not available in MINIMAL mode`

### 4.2 STANDARD — Raccourcis Complets

Tous les raccourcis sont disponibles. C'est le mode par défaut.

### 4.3 DEVELOPER — Raccourcis Complets (Sidebar Réduite)

Tous les raccourcis sont disponibles mais la sidebar est réduite. `Alt+1` et `Ctrl+W` fonctionnent mais la sidebar affiche uniquement les icônes.

### 4.4 OBSERVER — Raccourcis Contextuels

`ACTIVITY` est la zone principale. `Tab` commence par ACTIVITY. Les raccourcis WORKSPACE et CHAT sont disponibles mais les zones sont moins mises en avant.

### 4.5 DEBUG — Raccourcis Toutes Zones

Toutes les zones affichent des données brutes. `Tab` cycle dans l'ordre alphabétique (ACTIVITY, AGENT, CHAT, COMMAND, WORKSPACE). Pas de masquage.

---

## 5. Raccourcis de Stream Control

| Raccourci | Action | Contexte | Priorité |
|-----------|--------|----------|----------|
| `Ctrl+C` | Arrêter immédiatement le stream | Pendant un stream | **PLUS PUISSANT** |
| `Escape` | Annuler l'action en cours | Pendant un stream | Annule, ne force pas l'arrêt |
| `Ctrl+Space` | Pause/Reprise du stream | Pendant un stream | Toggle |
| `Ctrl+R` | Re-play d'une part | Après un stream | Remplace le stream |

**Règles** :
1. `Ctrl+C` est PLUS PUISSANT que `Escape` — il force l'arrêt
2. `Escape` annule l'action en cours mais ne force pas toujours l'arrêt
3. Après interruption, le focus revient au COMMAND
4. Les données partielles restent visibles dans CHAT/SESSION
5. `thinkingOpacity` disparaît après l'interruption
6. `Ctrl+Space` met le stream en pause (buffering, max 1000 lignes)
7. `Ctrl+Space` à nouveau reprend le stream

---

## 6. Raccourcis de Recherche et Historique

| Raccourci | Action | Description |
|-----------|--------|-------------|
| `Ctrl+F` | Search dans le stream | Barre de search en haut |
| `Ctrl+K` + texte | Search dans l'historique | Commandes précédentes |
| `↑` / `↓` dans le search | Naviger entre résultats | Résultats correspondants |
| `Enter` dans le search | Exécuter / Saute au résultat | Selon le type de search |
| `Escape` dans le search | Fermer le search | Retour à la zone précédente |

---

## 7. Raccourcis de Confirmation et Permission

### 7.1 Dialog Confirmation

| Raccourci | Action |
|-----------|--------|
| `Enter` | Yes / Confirmer |
| `N` | No |
| `Escape` | Cancel / Annuler |

Le focus est sur le bouton `Yes` par défaut.

### 7.2 Permission Confirmation

| Raccourci | Action |
|-----------|--------|
| `1` | Allow once |
| `2` | Allow always |
| `3` | Reject |
| `Escape` | Deny (timeout) |

Le focus est sur l'option par défaut. Timeout → `deny` automatique.

### 7.3 Question Confirmation

| Raccourci | Action |
|-----------|--------|
| `Enter` | Soumettre la réponse |
| `Escape` | Rejeter |
| `Tab` | Entre les champs (si plusieurs questions) |

---

## 8. Raccourcis Undo/Redo

| Raccourci | Action | Limite |
|-----------|--------|--------|
| `Ctrl+Z` | Undo | 20 actions maximum |
| `Ctrl+Shift+Z` | Redo | 20 actions maximum |

**Règles** :
1. L'undo fonctionne pour : suppressions, modifications de session, changements de mode
2. Les actions de stream NE SONT PAS annulables par undo
3. `UNKNOWN` dans l'état après undo → reste `UNKNOWN`

---

## 9. Gestion du Focus

### 9.1 Focus States par Composant

| Composant | Focus actif | Focus passif | Focus perdu |
|-----------|------------|--------------|-------------|
| `Sidebar` | `borderActive` + `accent` | `borderSubtle` + `textMuted` | `border` + `text` |
| `Prompt` | `borderActive` + `accent` | `border` + `text` | `border` + `textMuted` |
| `Table` | `backgroundElement` + `selectedListItemText` | `background` + `text` | `background` + `textMuted` |
| `Dialog` | `borderActive` + `backgroundMenu` | `border` + `backgroundPanel` | — |
| `CommandSurface` | `borderActive` + `accent` | `border` + `backgroundMenu` | `border` + `backgroundMenu` |

### 9.2 Règles de Focus

1. **Un seul focus à la fois** — le focus est exclusif entre les zones
2. **Le focus suit l'action** — quand un event se produit, le focus se déplace vers la zone concernée
3. **`Escape` libère le focus** — retour à la zone précédente
4. **Le focus est mémorisé** — quand on revient dans une zone, on retrouve la position précédente
5. **Keyboard-first** — la navigation se fait toujours au clavier
6. **`thinkingOpacity` ne change PAS le focus indicator** — le focus reste visible à tout moment

### 9.3 Focus par Mode

| Mode | Focus par défaut | Comportement de Tab |
|------|------------------|---------------------|
| MINIMAL | CHAT/SESSION | CHAT ↔ COMMAND |
| STANDARD | CHAT/SESSION | Cycle complet |
| DEVELOPER | CHAT/SESSION | Cycle réduit |
| OBSERVER | ACTIVITY | Commence par ACTIVITY |
| DEBUG | ACTIVITY | Ordre alphabétique |

---

## 10. Back Navigation

### 10.1 `Escape` comme Retour Universel

| Contexte | Action de `Escape` |
|----------|-------------------|
| Dans COMMAND | Annuler la saisie → retour au prompt vide |
| Dans un Dialog | Fermer le dialog → retour à la zone précédente |
| Dans un stream THINKING | Interrupt → retour au prompt |
| Dans ACTIVITY | Fermer le panneau → retour à la zone précédente |
| Dans WORKSPACE | Fermer la sidebar → retour à CHAT |
| Dans un Pattern de sélection | Annuler la sélection |
| Dans un Search | Fermer le search |
| Tout contexte | Retour à la zone de base |

### 10.2 Pile de Navigation

```
[HOME] → [WORKSPACE] → [SESSION:abc] → [COMMAND:/density STANDARD]
  ↑                                           │
  └───────────────────────────────────────────┘
  (Escape revient à SESSION:abc)
```

La pile est gérée par `api.router` et est limitée à **10 entrées** pour éviter la surcharge mémoire.

---

## 11. Conventions de Raccourcis

### 11.1 Convention de Nommage

| Symbole | Signification |
|---------|---------------|
| `+` | Touche simultanée |
| `,` | Séquence rapide (non simultanée) |
| `→` | Transition / navigation |
| `↔` | Cycle bidirectionnel |

### 11.2 Convention d'Affichage des Raccourcis

Tous les raccourcis sont affichables via `KeybindDisplay` :

```
Ctrl+D  → Change density
/       → Command palette
Tab     → Next item
Enter   → Confirm
```

### 11.3 Règles de Non-Conflit

1. Les raccourcis du terminal NE DOIVENT PAS entrer en conflit avec les raccourcis du système d'exploitation
2. `Ctrl+C` est réservé à l'interruption de stream dans le terminal
3. `Ctrl+Z` est réservé à l'undo dans le terminal
4. `Ctrl+V` est réservé au paste dans le terminal
5. `Ctrl+D` est réservé au changement de densité
6. `Ctrl+K` est réservé à la command palette
7. Les raccourcis `Alt+1..5` sont réservés à la navigation de zones

### 11.4 Raccourcis par Catégorie

| Catégorie | Préfixe | Exemples |
|-----------|---------|----------|
| Navigation | `Ctrl+W/C/A/E/I`, `Alt+1..5`, `Tab` | Focus zones |
| Stream | `Ctrl+C`, `Ctrl+Space`, `Ctrl+R` | Control stream |
| Commande | `Ctrl+K`, `/`, `Ctrl+D` | Command palette |
| Edition | `Ctrl+Z`, `Ctrl+Shift+Z`, `Ctrl+U` | Undo, clear |
| Search | `Ctrl+F`, `Ctrl+K` | Search |
| History | `↑`, `↓`, `Ctrl+K` | Historique |
| Confirmation | `Enter`, `Escape`, `N` | Dialogs |
| Permission | `1`, `2`, `3`, `Escape` | Permission |

---

## 12. Règles Fondamentales des Raccourcis

1. **Keyboard-first** : TOUS les patterns sont accessibles au clavier
2. **`Escape` est universel** : retour, annulation, fermeture
3. **`UNKNOWN != ZERO`** : Les raccourcis ne fabriquent jamais de données
4. **Le focus est visible** : `borderActive` + `accent` à tout moment
5. **Raccourcis affichables** : `KeybindDisplay` rend les raccourcis accessibles
6. **Pas de conflit** : Les raccourcis ne doivent pas entrer en conflit
7. **`thinkingOpacity` est indépendant** : ne dépend pas des raccourcis
8. **`N/A` reste `N/A`** : Dans les contextes où la donnée est inconnue

---

## 13. Références

- `docs/ux/navigation-model.md` — Modèle de navigation entre zones
- `docs/ux/interaction-patterns.md` — Patterns d'interaction complets
- `docs/design/components.md` — Composants et `KeybindDisplay`
- `docs/design/design-tokens.md` — Tokens de focus et de sélection
- `docs/architecture/architecture-overview.md` — Architecture et keymap
- `docs/architecture/event-contracts.md` — Événements déclenchant les patterns
- `docs/state/state-machine.md` — Machine à états et transitions
- `docs/state/presentation-state.md` — `TuiState` et selectors
- `docs/ui/layout-modes.md` — 5 modes de layout et leurs raccourcis
- `docs/design/density.md` — Niveaux de densité
- Prompt `02-ux-information-architecture.md` — Source du mandat
- Prompt `08-interaction-personalization.md` — Source du mandat Phase 08
