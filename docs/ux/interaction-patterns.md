# Interaction Patterns — Patterns d'Interaction

> **Source**: Docs design/architecture/state existants, Prompt `02-ux-information-architecture.md`
> **Date**: 2026-09-19
> **Règle**: « Preserve keyboard-first interaction. »

---

## 1. Vue d'ensemble

Les patterns d'interaction du HashCode Terminal sont conçus pour un **workflow au clavier d'abord** avec une validation souris/click en secondaire. Chaque pattern définit comment l'utilisateur interagit avec les 5 zones et les composants.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PATTERNS D'INTERACTION                            │
│                                                                      │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐            │
│  │ NAVIGATION  │    │  SELECTION  │    │  COMPLETION │            │
│  │             │    │             │    │             │            │
│  │ Tab / Arrow │    │ Enter /     │    │ Tab /       │            │
│  │ Ctrl+1..5   │    │ Space /     │    │ Ctrl+Space  │            │
│  │ Escape      │    │ Click       │    │ /           │            │
│  └─────────────┘    └─────────────┘    └─────────────┘            │
│                                                                      │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐            │
│  │ CONFIRMATION│    │  FEEDBACK   │    │  UNDO/REDO  │            │
│  │             │    │             │    │             │            │
│  │ Enter /     │    │ Toast /     │    │ Ctrl+Z      │            │
│  │ Y / N /     │    │ Alert /     │    │ Ctrl+Shift+Z│            │
│  │ Escape      │    │ Progress    │    │             │            │
│  └─────────────┘    └─────────────┘    └─────────────┘            │
│                                                                      │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐            │
│  │ STREAM      │    │  PERMISSION │    │  SEARCH     │            │
│  │             │    │             │    │             │            │
│  │ Ctrl+C      │    │ ask/deny/   │    │ Ctrl+F /    │            │
│  │ Escape      │    │ allow       │    │ /search     │            │
│  │ Space       │    │             │    │ Arrow keys  │            │
│  └─────────────┘    └─────────────┘    └─────────────┘            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Pattern de Navigation

### 2.1 Tab Navigation

**Déclencheur** : `Tab` / `Shift+Tab`

**Comportement** :
- `Tab` : Déplace le focus vers la zone suivante dans l'ordre de priorité du mode
- `Shift+Tab` : Déplace le focus vers la zone précédente
- Le focus est EXCLUSIF — un seul élément a le focus à la fois
- Le focus suit l'ordre : `WORKSPACE → CHAT → AGENT → ACTIVITY → COMMAND`

**Règles** :
1. Le focus DOIT être visuellement distinct (`borderActive` + `accent`)
2. `thinkingOpacity` ne change PAS pendant la navigation
3. La navigation NE DOIT PAS perdre de données
4. Si la zone cible est masquée (MINIMAL), le focus saute à la zone suivante visible

### 2.2 Direct Zone Access

**Déclencheur** : `Alt+1` à `Alt+5`

**Comportement** :
- `Alt+1` : WORKSPACE
- `Alt+2` : CHAT/SESSION
- `Alt+3` : AGENT
- `Alt+4` : ACTIVITY
- `Alt+5` : COMMAND

**Règles** :
1. Fonctionne à TOUS les niveaux de densité
2. Si la zone est masquée, afficher un toast `[⚠ WARNING] Zone not available in MINIMAL mode`
3. Le focus se déplace immédiatement

### 2.3 Keyboard Shortcuts

**Déclencheur** : `Ctrl+W/C/A/E/I`

**Comportement** :
- `Ctrl+W` : Focus WORKSPACE
- `Ctrl+C` : Focus CHAT/SESSION
- `Ctrl+A` : Focus AGENT
- `Ctrl+E` : Focus ACTIVITY
- `Ctrl+I` : Focus COMMAND

**Règles** :
1. Fonctionne depuis N'IMPORTE QUELLE zone
2. Pas de conflit avec les raccourcis système du terminal
3. Les raccourcis sont affichables via `KeybindDisplay`

---

## 3. Pattern de Sélection

### 3.1 List Selection

**Déclencheur** : `↑` / `↓` + `Enter`

**Comportement** :
1. `↑` / `↓` : Déplacer la sélection dans une liste
2. `Enter` : Confirmer la sélection
3. `Escape` : Annuler la sélection
4. `Tab` : Sortir de la liste

**Utilisation** :
- Sélection de session dans WORKSPACE
- Sélection de worker dans AGENT
- Sélection d'événement dans ACTIVITY
- Sélection de fichier dans la sidebar

**Règles** :
1. L'élément sélectionné est mis en évidence (`backgroundElement` + `selectedListItemText`)
2. La sélection est cyclique (le dernier item → le premier et vice versa)
3. `Enter` confirme, `Escape` annule — jamais de confirmation implicite
4. `UNKNOWN` items sont sélectionnables et affichés comme `[? UNKNOWN]`

### 3.2 Range Selection

**Déclencheur** : `Shift+↑` / `Shift+↓`

**Comportement** :
- Sélection d'une plage d'éléments consécutifs
- Utilisé pour copier plusieurs sessions, événements, etc.

### 3.3 Context Menu

**Déclencheur** : `Right-click` ou `Shift+F10`

**Comportement** :
1. Affiche un `Menu` avec les actions contextuelles
2. Navigation au clavier dans le menu (`↑` / `↓` + `Enter`)
3. `Escape` ferme le menu

**Actions contextuelles par zone** :
| Zone | Actions |
|------|---------|
| WORKSPACE | New session, Delete, Rename, Share |
| CHAT/SESSION | Interrupt, Compact, Copy, Export |
| AGENT | Reroute, Refresh, Toggle, Detail |
| ACTIVITY | Filter, Copy, Expand, Export |
| COMMAND | History, Clear, Save |

---

## 4. Pattern de Complétion

### 4.1 Command Completion

**Déclencheur** : `Tab` dans COMMAND

**Comportement** :
1. `Tab` : Complétion automatique de la commande en cours
2. `Shift+Tab` : Complétion inversée
3. Si plusieurs correspondances : afficher un `SelectList` des options
4. `Enter` : Sélectionner l'option

**Exemple** :
```
› /den<Tab> → /density STANDARD
› /ter<Tab> → /terminal info
```

### 4.2 File Path Completion

**Déclencheur** : `Tab` après un chemin partiel

**Comportement** :
1. Complète le chemin vers le fichier/dossier correspondant
2. Si plusieurs correspondances : affiche toutes les options
3. `Escape` : Annule la complétion

### 4.3 Model/Provider Completion

**Déclencheur** : `Tab` dans la sélection de modèle

**Comportement** :
1. Complète avec le provider ou modèle suivant
2. Affiche les informations du modèle (capabilities, coût)
3. `UNKNOWN` models sont affichés comme `[? UNKNOWN]`

---

## 5. Pattern de Confirmation

### 5.1 Dialog Confirmation

**Déclencheur** : `Enter` sur une action destructrice

**Comportement** :
1. Affiche un `Dialog` de type `Confirm`
2. Options : `Yes` / `No` / `Cancel`
3. `Enter` = Yes, `N` = No, `Escape` = Cancel
4. Le focus est sur le bouton `Yes` par défaut
5. `thinkingOpacity` ne s'applique PAS aux dialogs

**Exemple** :
```
╭─ CONFIRM ──────────────────────────────────╮
│ Delete session "abc"?                        │
│                                            │
│ [Yes] [No] [Cancel]                          │
╰──────────────────────────────────────────────╯
```

### 5.2 Permission Confirmation

**Déclencheur** : `EventPermissionV2Asked`

**Comportement** :
1. Affiche un `Dialog` de type `Permission`
2. Options : `ask` / `deny` / `allow`
3. `1` = allow once, `2` = allow always, `3` = reject
4. Le focus est sur l'option par défaut
5. Timeout → deny automatique

**Règle** : Chaque permission DOIT avoir une raison claire affichée dans le dialog.

### 5.3 Question Confirmation

**Déclencheur** : `EventQuestionV2Asked`

**Comportement** :
1. Affiche un `Dialog` de type `Prompt`
2. Champ de saisie pour la réponse
3. `Enter` = soumettre, `Escape` = rejeter
4. `Tab` entre les champs (si plusieurs questions)

---

## 6. Pattern de Feedback

### 6.1 Toast

**Déclencheur** : Événement système (`EventTuiToastShow2`, `EventSessionError`, etc.)

**Comportement** :
1. Affiche un `Toast` en bas ou en haut du terminal
2. Durée : courte (info/success) / moyenne (warning) / longue (error)
3. Auto-dismiss après la durée
4. `Escape` ou `Click` pour dismiss immédiatement
5. Les toasts sont empilés — le dernier est visible, les précédents sont dans le scrollback

**Niveaux** :
| Niveau | Couleur | Durée | Glyphe |
|--------|---------|-------|--------|
| info | `info` | Courte | `ℹ` |
| success | `success` | Courte | `✓` |
| warning | `warning` | Moyenne | `⚠` |
| error | `error` | Longue | `✗` |

**Règles** :
1. `UNKNOWN` dans un toast → `[? UNKNOWN]` — jamais `0` ou `IDLE`
2. Chaque toast DOIT avoir une raison
3. Pas de superposition de toasts — le dernier remplace le précédent
4. Les toasts n'interrompent pas le stream CHAT

### 6.2 Alert

**Déclencheur** : Condition critique (erreur, warning, seuil dépassé)

**Comportement** :
1. Affiche un `Alert` dans la zone concernée
2. L'alerte est bloquante si error/permission
3. L'alerte est non-bloquante si warning/info
4. `Escape` ferme l'alerte (non-bloquante)
5. L'utilisateur DOIT répondre pour les alertes bloquantes

**Exemple** :
```
╭─ ⚠ WARNING ─────────────────────────╮
│ Approchant la limite de 262K tokens  │
╰──────────────────────────────────────╯
```

### 6.3 Progress

**Déclencheur** : Opération en cours (stream, tool call, compaction)

**Comportement** :
1. Affiche un `ProgressBar` ou `Spinner`
2. `thinkingOpacity` appliqué au reasoning
3. Les métriques sont mises à jour en temps réel
4. `Escape` n'interrompt PAS le progress — c'est `Ctrl+C` ou `Escape` dans le stream

**Règles** :
1. Le progress est affiché dans CHAT/SESSION (zone prioritaire)
2. `UNKNOWN` dans le progress → `N/A` pour les métriques inconnues
3. Le progress ne peut PAS être fabriqué — afficher uniquement les données reçues

---

## 7. Pattern de Stream Control

### 7.1 Interrupt Stream

**Déclencheur** : `Ctrl+C` ou `Escape` (pendant un stream)

**Comportement** :
1. `Ctrl+C` : Arrête immédiatement le stream, envoie un signal d'interruption
2. `Escape` : Annule l'action en cours, retourne au prompt
3. Le stream s'arrête proprement — les données partielles sont conservées
4. Un toast `[⚠ WARNING] Stream interrupted` est affiché

**Règles** :
1. `Ctrl+C` est PLUS PUISSANT que `Escape` — il force l'arrêt
2. `Escape` annule l'action en cours mais ne force pas toujours l'arrêt
3. Après interruption, le focus revient au COMMAND
4. Les données partielles restent visibles dans CHAT/SESSION
5. `thinkingOpacity` disparaît après l'interruption

### 7.2 Pause Stream

**Déclencheur** : `Ctrl+Space` (pendant un stream)

**Comportement** :
1. Met le stream en pause (buffering)
2. L'utilisateur peut lire le contenu déjà streaming
3. `Ctrl+Space` à nouveau reprend le stream
4. `Escape` annule la pause et retourne au prompt

**Règles** :
1. La pause ne coupe PAS le stream — elle le bufferise
2. Le buffer est limité à 1000 lignes pour éviter la surcharge mémoire
3. `thinkingOpacity` est appliqué au contenu bufferisé
4. `UNKNOWN` reste `UNKNOWN` pendant la pause

### 7.3 Re-play

**Déclencheur** : `Ctrl+R` (sur une partie)

**Comportement** :
1. Re-exécute la partie sélectionnée
2. Affiche un `Spinner` pendant la re-exécution
3. Le stream est remplacé par le nouveau contenu

---

## 8. Pattern de Permission

### 8.1 Permission Ask

**Déclencheur** : `EventPermissionV2Asked`

**Comportement** :
1. Le focus se déplace vers COMMAND
2. Un `Dialog` de type `Permission` s'affiche
3. Options : `ask` (once), `allow` (always), `reject` (permanently)
4. `Enter` = confirmer l'option sélectionnée
5. `Escape` = deny (timeout)

**Règles** :
1. Chaque permission DOIT avoir une raison claire
2. Le timeout entraîne un `deny` automatique
3. `UNKNOWN` dans la permission → `[? UNKNOWN]` — jamais `allow` par défaut
4. Les permissions sont affichées dans la zone COMMAND (focus forcé)

### 8.2 Permission Reply

**Déclencheur** : Réponse de l'utilisateur

**Comportement** :
1. `ask` → autorise une seule fois
2. `allow` → autorise et mémorise
3. `deny`/`reject` → refuse
4. Le focus revient à la zone précédente
5. Un `Toast` confirme la réponse

---

## 9. Pattern de Search

### 9.1 Search dans le Stream

**Déclencheur** : `Ctrl+F`

**Comportement** :
1. Affiche une barre de search en haut du terminal
2. `↑` / `↓` : Navigue entre les résultats
3. `Enter` : Saute au résultat suivant
4. `Escape` : Ferme le search
5. Le texte search est en `mono`

### 9.2 Search dans l'Historique

**Déclencheur** : `Ctrl+K` + texte

**Comportement** :
1. Affiche les commandes précédentes correspondant au texte
2. `↑` / `↓` : Navigue entre les résultats
3. `Enter` : Exécute la commande sélectionnée
4. `Escape` : Ferme le search

---

## 10. Pattern de Undo/Redo

### 10.1 Undo

**Déclencheur** : `Ctrl+Z`

**Comportement** :
1. Annule la dernière action
2. Fonctionne pour : suppressions, modifications de session, changements de mode
3. Un `Toast` confirme l'undo
4. `Ctrl+Shift+Z` = Redo

**Règles** :
1. L'undo est limité à 20 actions pour éviter la surcharge mémoire
2. Les actions de stream NE SONT PAS annulables par undo
3. `UNKNOWN` dans l'état après undo → reste `UNKNOWN`

---

## 11. Patterns par Zone

### 11.1 WORKSPACE Patterns

| Pattern | Déclencheur | Comportement |
|---------|-------------|--------------|
| Select session | `↑`/`↓` + `Enter` | Ouvre la session dans CHAT |
| New session | `Ctrl+N` | Crée une nouvelle session |
| Delete session | `Del` + confirmation | Supprime la session |
| Rename | `F2` | Renomme la session |
| Share | `Ctrl+S` | Partage la session |
| Refresh | `Ctrl+R` | Rafraîchit le statut workspace |

### 11.2 CHAT/SESSION Patterns

| Pattern | Déclencheur | Comportement |
|---------|-------------|--------------|
| Scroll messages | `↑`/`↓` | Scroll dans l'historique |
| Skip to part | `Ctrl+←`/`Ctrl+→` | Sauté entre parts |
| Interrupt | `Ctrl+C` | Arrête le stream |
| Pause | `Ctrl+Space` | Met en pause le stream |
| Copy | `Ctrl+C` | Copie le texte sélectionné |
| Compact | `Ctrl+M` | Déclenche la compaction |
| Export | `Ctrl+E` | Exporte la conversation |

### 11.3 AGENT Patterns

| Pattern | Déclencheur | Comportement |
|---------|-------------|--------------|
| Select worker | `↑`/`↓` + `Enter` | Détail du worker |
| Reroute | `R` | Re-router le prochain request |
| Refresh | `Ctrl+R` | Rafraîchit les statuts |
| Toggle worker | `Space` | Active/désactive un worker |
| Detail | `Enter` | Affiche le détail du worker |

### 11.4 ACTIVITY Patterns

| Pattern | Déclencheur | Comportement |
|---------|-------------|--------------|
| Scroll events | `↑`/`↓` | Navigate dans le flux |
| Filter | `F` | Filtre par type |
| Copy event | `C` | Copie l'événement |
| Expand | `Enter` | Affiche le détail |
| Clear | `Ctrl+L` | Efface le flux |

### 11.5 COMMAND Patterns

| Pattern | Déclencheur | Comportement |
|---------|-------------|--------------|
| Command palette | `Ctrl+K` ou `/` | Ouvre la palette |
| History | `↑`/`↓` | Navigue dans l'historique |
| Complete | `Tab` | Complétion automatique |
| Submit | `Enter` | Exécute la commande |
| Cancel | `Escape` | Annule la saisie |
| Clear | `Ctrl+U` | Efface l'input |
| Paste | `Ctrl+V` | Colle le contenu |

---

## 12. Règles Fondamentales des Patterns

1. **Keyboard-first** : TOUS les patterns sont accessibles au clavier
2. **Escape = retour** : `Escape` est universellement le retour
3. **Un seul focus** : Un seul élément a le focus à un moment donné
4. **Focus suit l'action** : Le focus se déplace vers la zone affectée
5. **Pas de fabrication** : Aucun pattern ne crée de données — afficher uniquement ce qui existe
6. **`UNKNOWN != ZERO`** : Les patterns de feedback respectent la règle UNKNOWN != ZERO
7. **`thinkingOpacity`** : S'applique uniquement aux éléments THINKING, jamais aux autres patterns
8. **Confirmation pour les actions destructrices** : Delete, compact, reset nécessitent une confirmation
9. **Toast pour le feedback** : Feedback court = toast, feedback long = alert
10. **Pas de conflit** : Les raccourcis ne doivent pas entrer en conflit avec les raccourcis du terminal

---

## 13. Références

- `docs/design/components.md` — Composants de contrôle (Dialog, Prompt, SelectionList)
- `docs/design/design-tokens.md` — Tokens de focus et de sélection
- `docs/design/status-language.md` — Vocabulaire de statut dans les feedback
- `docs/design/density.md` — Patterns par niveau de densité
- `docs/architecture/architecture-overview.md` — Architecture et keymap
- `docs/architecture/event-contracts.md` — Événements déclenchant les patterns
- `docs/state/state-machine.md` — Machine à états et transitions
- `docs/state/presentation-state.md` — TuiState et selectors
- Prompt `02-ux-information-architecture.md` — Source du mandat
