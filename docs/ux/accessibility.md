# Accessibility — Accessibilité Terminal

> **Source**: Docs design/architecture/state existants, Prompt `02-ux-information-architecture.md`
> **Date**: 2026-09-19
> **Règle**: « Every number needs context. Every state needs semantic meaning. Every alert has a reason. »

---

## 1. Vue d'ensemble

L'accessibilité du HashCode Terminal garantit que l'interface est utilisable par TOUS les utilisateurs, quelles que soient leurs capacités, dans TOUS les environnements terminaux. Le terminal est nativement accessible grâce à :

- Le texte comme vecteur principal (pas de dépendance à la couleur seule)
- Le clavier comme interface primaire (pas de dépendance à la souris)
- La structure sémantique des statuts (glyphe + texte + couleur)
- Les fallback pour tous les terminaux

```
┌─────────────────────────────────────────────────────────────────────┐
│                   ACCESSIBILITÉ TERMINALE                            │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  1. PERÇO VISUEL     │  Glyphe + Texte + Couleur             │   │
│  │                     │  JAMAIS couleur seule                   │   │
│  ├──────────────────────────────────────────────────────────────┤   │
│  │  2. PERÇO CLAVIER    │  Tous les patterns accessibles        │   │
│  │                     │  Keyboard-first, toujours               │   │
│  ├──────────────────────────────────────────────────────────────┤   │
│  │  3. PERÇO SÉMANTIQUE │  Chaque état a un sens                │   │
│  │                     │  UNKNOWN != ZERO, jamais de fabrique    │   │
│  ├──────────────────────────────────────────────────────────────┤   │
│  │  4. PERÇO TECHNIQUE  │  Fallback pour tous les terminaux     │   │
│  │                     │  80+ cols, ANSI 256, ASCII              │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Perception Visuelle

### 2.1 Double Signal — JAMAIS la couleur seule

Chaque statut DOIT être lisible sans dépendre de la couleur seule :

**Règle du double signal** :
- **Texte** : le libellé (ACTIVE, ERROR, etc.) est TOUJOURS présent
- **Glyphe** : le symbole visuel (●, ✗, ◐, etc.) est TOUJOURS présent
- **Couleur** : la couleur est UN SIGNAL SUPPLÉMENTAIRE, pas le seul

**Exemples** :
```
[● ACTIVE]  eurinhash-agent  ·  38%        ← Lisible sans couleur
[✗ ERROR]  worker-google  ·  Connection     ← Lisible sans couleur
[◐ THINKING]  worker-novita  ·  2.3s        ← Lisible sans couleur
[? UNKNOWN]  worker-pollinations            ← Lisible sans couleur
```

**Pour les daltoniens** :
| Statut | Glyphe | Texte | Couleur | Indépendant de la couleur ? |
|--------|--------|-------|---------|---------------------------|
| ACTIVE | `●` | `ACTIVE` | `accent` | ✅ Oui |
| IDLE | `○` | `IDLE` | `textMuted` | ✅ Oui |
| THINKING | `◐` | `THINKING` | `secondary` + opacity | ✅ Oui |
| RUNNING | `▶` | `RUNNING` | `primary` | ✅ Oui |
| WAITING | `⌛` | `WAITING` | `warning` | ✅ Oui |
| SUCCESS | `✓` | `SUCCESS` | `success` | ✅ Oui |
| WARNING | `⚠` | `WARNING` | `warning` | ✅ Oui |
| ERROR | `✗` | `ERROR` | `error` | ✅ Oui |
| CANCELLED | `⊘` | `CANCELLED` | `textMuted` | ✅ Oui |
| UNKNOWN | `?` | `UNKNOWN` | `textMuted` | ✅ Oui |
| DISCONNECTED | `⊘` | `DISCONNECTED` | `error` | ✅ Oui |

### 2.2 Contraste et Couleurs

**Règles** :
1. **JAMAIS** de couleur seule comme indicateur de statut
2. **TOUJOURS** un glyphe + un texte en complément de la couleur
3. `thinkingOpacity` est appliqué UNIQUEMENT aux éléments THINKING
4. Les couleurs sont lues dynamiquement via `theme.current` — jamais hardcodées
5. Les tokens de couleur sont `RGBA` — pas de dépendance à un canal spécifique

### 2.3 Glyphes Fallback

Pour les terminaux ne supportant pas les box-drawing ou Unicode avancé :

| Glyphe original | Fallback ASCII | Contexte |
|----------------|----------------|----------|
| `┌┐└┘` | `+-` | Coins de panneau |
| `─` | `-` | Bordure horizontale |
| `│` | `|` | Bordure verticale |
| `┄` | `:` | Séparation subtile |
| `═` | `=` | Séparation forte |
| `●` | `*` | Point actif |
| `○` | `o` | Point inactif |
| `◐` | `@` | Point thinking |
| `▶` | `>` | Triangle droit |
| `⌛` | `!` | Horloge |
| `✗` | `X` | Croix |
| `✓` | `V` | Checkmark |
| `⚠` | `!` | Triangle alerte |
| `⊘` | `O` | Cercle barré |
| `?` | `?` | Point d'interrogation |

---

## 3. Perception Clavier

### 3.1 Keyboard-First — TOUS les patterns sont accessibles

**Règle fondamentale** : La navigation et l'interaction sont TOUJOURS au clavier d'abord. La souris est secondaire.

**Raccourcis essentiels** :
| Raccourci | Action | Accessibilité |
|-----------|--------|---------------|
| `Tab` | Focus suivant | ✅ Essentiel |
| `Shift+Tab` | Focus précédent | ✅ Essentiel |
| `Enter` | Confirmer/Sélectionner | ✅ Essentiel |
| `Escape` | Annuler/Retour | ✅ Essentiel |
| `↑`/`↓` | Navigation dans une liste | ✅ Essentiel |
| `Ctrl+C` | Interrompre/Arrêter | ✅ Essentiel |
| `Ctrl+K` | Command palette | ✅ Essentiel |
| `/` | Début de commande | ✅ Essentiel |

### 3.2 Focus Indicator

**Règles** :
1. Le focus est TOUJOURS visible via `borderActive` + `accent`
2. Le focus ne dépend JAMAIS de la couleur seule — le `borderActive` est visible même en monochrome
3. Le focus est clairement différent du hover (`borderSubtle` + `textMuted`)
4. Le focus perdu est indiqué par `border` + `text` sans modification
5. `thinkingOpacity` ne change PAS le focus indicator

### 3.3 Navigation sans Souris

Tous les composants sont accessibles au clavier :
- `Sidebar` : `↑`/`↓` pour naviguer, `Enter` pour sélectionner
- `Table` : `↑`/`↓` pour les lignes, `←`/`→` pour les colonnes
- `Prompt` : Saisie directe, `↑`/`↓` pour l'historique
- `Dialog` : `Tab` entre les boutons, `Enter` pour confirmer, `Escape` pour annuler
- `CommandSurface` : Saisie directe, `Tab` pour la complétion
- `SelectionList` : `↑`/`↓` pour la sélection, `Enter` pour confirmer

---

## 4. Perception Sémantique

### 4.1 Chaque État a un Sens Sémantique

**Règle** : « Every state needs semantic meaning. »

| État | Sens | Glyphe | Texte | Couleur | Sémantique claire ? |
|------|------|--------|-------|---------|---------------------|
| `ACTIVE` | Opérationnel | `●` | `ACTIVE` | `accent` | ✅ |
| `IDLE` | Inactif | `○` | `IDLE` | `textMuted` | ✅ |
| `THINKING` | En réflexion | `◐` | `THINKING` | `secondary` + opacity | ✅ |
| `RUNNING` | En exécution | `▶` | `RUNNING` | `primary` | ✅ |
| `WAITING` | En attente | `⌛` | `WAITING` | `warning` | ✅ |
| `SUCCESS` | Réussi | `✓` | `SUCCESS` | `success` | ✅ |
| `WARNING` | Avertissement | `⚠` | `WARNING` | `warning` | ✅ |
| `ERROR` | Échec | `✗` | `ERROR` | `error` | ✅ |
| `CANCELLED` | Annulé | `⊘` | `CANCELLED` | `textMuted` | ✅ |
| `UNKNOWN` | Indéterminé | `?` | `UNKNOWN` | `textMuted` | ✅ |
| `DISCONNECTED` | Déconnecté | `⊘` | `DISCONNECTED` | `error` | ✅ |

### 4.2 `UNKNOWN != ZERO` — Sémantique de l'Absence

**Règle fondamentale** : `UNKNOWN` ne doit JAMAIS se confondre avec `ZERO` ou `IDLE`.

| Cas | Incorrect | Correct | Pourquoi |
|-----|-----------|---------|----------|
| Coût non calculé | `$0.00` | `N/A` | `0` signifie gratuit, `N/A` signifie inconnu |
| Token non compté | `0` | `undefined` | `0` est un compte réel, `undefined` est inconnu |
| Branch absente | `""` | `null` | `""` est une branche vide, `null` est pas de git |
| Session inexistante | `{ id: "" }` | `undefined` | Un objet vide n'est pas une session |
| Worker non interrogé | `[○ IDLE]` | `[? UNKNOWN]` | Inconnu ≠ Inactif |

### 4.3 Contexte pour les Nombres

**Règle** : « Every number needs context. »

| Nombre | Contexte | Affichage |
|--------|----------|-----------|
| `38` | Pourcentage | `38%` |
| `100` | Tokens | `100K` |
| `0` | Coût connu et nul | `$0.00` |
| `0` | Lignes ajoutées | `0 additions` |
| `0` | Tokens d'input | `tokens.input: 0` (valide) |
| Inconnu | Coût | `N/A` |
| Inconnu | Limite | `N/A` |
| Inconnu | Statut worker | `[? UNKNOWN]` |

### 4.4 Chaque Alerte a une Raison

**Règle** : « Every alert has a reason. »

```
╭─ ⚠ WARNING ─────────────────────────╮
│ Approchant la limite de 262K tokens  │
╰──────────────────────────────────────╯
```

```
╭─ ✗ ERROR ───────────────────────────╮
│ Connection refused to MCP endpoint   │
╰──────────────────────────────────────╯
```

```
╭─ ? UNKNOWN ─────────────────────────╮
│ Coût de l'agent non calculé          │
╰──────────────────────────────────────╯
```

---

## 5. Compatibilité Technique

### 5.1 Terminaux Minimum Supportés

| Terminal | Largeur min | Couleurs min | Unicode min | Notes |
|----------|-------------|--------------|-------------|-------|
| Screen | 80 cols | ANSI 256 | Basique | Legacy |
| Tmux | 80 cols | ANSI 256 | Basique | Wrapper |
| GNOME Terminal | 80 cols | ANSI 256 | Basique | Linux |
| Konsole | 80 cols | ANSI 256 | Basique | Linux |
| Windows Terminal | 80 cols | ANSI 256 | Basique | Windows |
| VS Code Terminal | 80 cols | ANSI 256 | Basique | Intégré |
| iTerm2 | 80 cols | ANSI 256 | Oui | macOS |
| Terminal.app | 80 cols | ANSI 256 | Oui | macOS |
| Alacritty | 80 cols | Truecolor | Oui | Multi-plateforme |
| Kitty | 80 cols | Truecolor | Oui | Multi-plateforme |
| WezTerm | 80 cols | Truecolor | Oui | Multi-plateforme |

### 5.2 Fallback par Niveau de Support

| Fonctionnalité | Support complet | Fallback |
|----------------|----------------|----------|
| Box-drawing | ✅ | ASCII `| - +` |
| Truecolor | ✅ | ANSI 256 colors |
| Unicode avancé | ✅ | ASCII + crochets |
| Nerd Font icons | ✅ | Texte seul |
| Mouse support | ✅ | Navigation clavier |
| Split panes | ✅ | Full screen alterné |
| Bracketed paste | ✅ | Input standard |
| Alt screen buffer | ✅ | Standard screen |

### 5.3 Largeur Minimale

| Composant | Largeur min | Comportement en dessous |
|-----------|-------------|------------------------|
| Prompt | 20 cols | Texte seul, wrap |
| StatusBar | 60 cols | Compact, glyphes uniquement |
| Sidebar | 20 cols | Icônes uniquement, hover pour labels |
| Table | 40 cols | Liste clé-valeur, puis JSON lines |
| Alert | 40 cols | Texte préfixé par le glyphe |
| Dialog | 50 cols | Plein largeur |
| DiffView | 60 cols | JSON lines |
| ProgressBar | 20 cols | Texte seul `38%` |
| Panel | 30 cols | Réduit à l'essentiel |
| CommandSurface | 40 cols | Format compact |

---

## 6. Patterns d'Accessibilité Spécifiques

### 6.1 Screen Reader Simulation

Bien que les terminaux n'aient pas de screen reader natif, l'architecture garantit que le contenu est lisible de manière séquentielle :

1. **Ordre logique** : WORKSPACE → CHAT → AGENT → ACTIVITY → COMMAND
2. **Hiérarchie sémantique** : Les titres sont `display`/`heading`, le contenu est `body`, les métadonnées sont `caption`
3. **Texte alternatif** : Chaque glyphe a un texte associé
4. **`UNKNOWN` est explicite** : `[? UNKNOWN] · Coût non calculé` — pas juste `?`

### 6.2 High Contrast Mode

**Pour les terminaux en mode high contrast** :

| Adaptation | Description |
|------------|-------------|
| Glyphes agrandis | `●` devient `●●` ou `**` |
| Texte prioritaire | Le texte est PLUS grand que le glyphe |
| Pas d'opacité | `thinkingOpacity` est désactivé en high contrast |
| Bordures épaisses | `borderActive` est plus visible |
| `UNKNOWN` explicite | `[? UNKNOWN]` toujours affiché en texte complet |

### 6.3 Motion Reduction

**Pour les terminaux qui ne supportent pas les animations** :

| Adaptation | Description |
|------------|-------------|
| `thinkingOpacity` statique | Pas de variation d'opacité — état fixe |
| Pas de flicker | Les mises à jour sont instantanées |
| Pas de transition | Le resize est immédiat |
| Pas de spinner animé | `◐` fixe au lieu d'animation |
| Pas de progress bar animé | Texte statique `38%` |

### 6.4 Terminal Emulator Accessibility Settings

**Recommandations pour l'utilisateur** :

| Paramètre | Recommandation | Raison |
|-----------|---------------|--------|
| Font size | ≥ 12pt | Lisibilité |
| Font family | `Mono` ou `Consolas` | Support box-drawing |
| Cursor shape | Block | Visibilité du focus |
| Bell | On | Feedback sonore |
| Scrollback | ≥ 10000 lignes | Historique des events |
| Color scheme | Dark (défaut) | Meilleur contraste |
| Cursor color | `accent` | Visibilité du focus |

---

## 7. Accessibilité par Mode

### 7.1 Accessibilité par Niveau de Densité

| Niveau | Accessibilité | Notes |
|--------|---------------|-------|
| `MINIMAL` | ✅ Haute | Texte grand, espaces larges, facile à lire |
| `STANDARD` | ✅ Haute | Équilibré, par défaut |
| `DEVELOPER` | ⚠️ Moyenne | Dense, mais toujours lisible |
| `OBSERVER` | ⚠️ Moyenne | Très dense, nécessite 120+ cols |
| `DEBUG` | ❌ Basse | Données brutes, pas adapté à la lecture |

### 7.2 Accessibilité par Largeur

| Largeur | Accessibilité | Notes |
|---------|---------------|-------|
| 80 cols | ✅ Haute | Fallback complet, tous les composants accessibles |
| 100 cols | ✅ Haute | Layout standard, tous les composants visibles |
| 120 cols | ✅ Haute | Layout étendu, métriques détaillées |
| 160 cols | ✅ Haute | Layout complet, tous les détails visibles |

### 7.3 Règles de Non-Réduction de l'Accessibilité

1. **JAMAIS** réduire l'accessibilité quand on passe à un mode plus dense
2. **TOUJOURS** maintenir le texte des statuts lisible à 80 colonnes
3. **JAMAIS** de données fabriquées pour remplir l'espace
4. **TOUJOURS** `UNKNOWN` reste lisible à toute largeur et densité
5. **JAMAIS** cacher une information critique pour des raisons de place
6. `thinkingOpacity` ne réduit PAS l'accessibilité — le texte reste lisible

---

## 8. Règles d'Accessibilité Fondamentales

### 8.1 Règles Textuelles

1. **Chaque glyphe a un texte associé** — `[●]` → `[● ACTIVE]` → `[● ACTIVE] eurinhash-agent`
2. **`UNKNOWN` est toujours explicite** — `[? UNKNOWN] · Raison`
3. **Les nombres ont toujours du contexte** — `38%` (pas juste `38`), `100K` (pas juste `100`)
4. **Les erreurs ont toujours une raison** — `[✗ ERROR] · Connection refused`
5. **JAMAIS de couleur seule** — le double signal est obligatoire

### 8.2 Règles Clavier

1. **Tous les patterns sont accessibles au clavier** — aucune fonctionnalité nécessite la souris
2. **`Escape` est universel** — retour, annulation, fermeture
3. **Le focus est visible** — `borderActive` + `accent` à tout moment
4. **Tabulation logique** — l'ordre de tabulation suit la hiérarchie visuelle
5. **Raccourcis affichables** — `KeybindDisplay` rend les raccourcis accessibles

### 8.3 Règles Techniques

1. **Fallback ASCII pour tous les terminaux** — aucune dépendance à l'Unicode avancé
2. **ANSI 256 colors minimum** — Truecolor est un bonus, pas une exigence
3. **80 colonnes est la largeur minimale** — tous les patterns fonctionnent à 80 cols
4. **Pas de dépendance au terminal émulé** — le comportement est le même dans tous les terminaux
5. **`thinkingOpacity` est indépendant** — ne dépend pas du terminal ou de la largeur

### 8.4 Règles `UNKNOWN != ZERO`

1. **`UNKNOWN` reste `UNKNOWN`** à toute largeur, densité, et mode
2. **`N/A` reste `N/A`** — jamais transformé en `0` ou `-`
3. **`$0.00` reste `$0.00`** — le coût connu et nul est affiché tel quel
4. **`100K · 38%` reste `100K · 38%`** quand la limite est inconnue — jamais complété artificiellement
5. **`undefined` est `undefined`** — jamais remplacé par `0`, `""`, ou `false`

---

## 9. Test d'Accessibilité

### 9.1 Checklist de Test

```
CHECKLIST — Accessibilité Terminal
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
☐ Chaque statut est lisible sans couleur (glyphe + texte)
☐ Le focus est visible à tout moment (borderActive + accent)
☐ TOUS les patterns sont accessibles au clavier
☐ Escape fonctionne comme retour universel
☐ UNKNOWN est affiché comme [? UNKNOWN] à toute largeur
☐ N/A n'est jamais remplacé par 0 ou -
☐ $0.00 n'est jamais supprimé quand le coût est connu et nul
☐ 100K · 38% n'est jamais complété quand la limite est inconnue
☐ Le terminal fonctionne à 80 colonnes
☐ Le fallback ASCII fonctionne (box-drawing → | - +)
☐ Le mode MINIMAL est lisible à 80 colonnes
☐ thinkingOpacity ne perturbe pas la lisibilité
☐ Les toasts ont toujours une raison
☐ Les alerts ont toujours une raison
☐ Les dialogues ont toujours une raison
☐ Aucun raccourci ne conflit avec les raccourcis du terminal
☐ L'ordre de tabulation suit la hiérarchie visuelle
☐ Le screen reader simulation est valide (ordre logique)
☐ Le high contrast mode est considéré
☐ Le motion reduction est considéré
```

### 9.2 Test par Largeur

| Test | 80 cols | 100 cols | 120 cols | 160 cols |
|------|---------|----------|----------|----------|
| Statuts lisibles | ✅ | ✅ | ✅ | ✅ |
| Tables lisibles | Liste | Compacte | Complète | Complète |
| Métriques affichées | `38%` | `38% · 100K` | `38% · 100K/262K` | Complet |
| Progress bar | Texte | Barre courte | Barre standard | Barre longue |
| Sidebar visible | Icônes | Réduite | Standard | Standard |
| Panels | Ligne sép. | Bordure simple | Bordure complète | Bordure double |
| Dialogs | Plein largeur | Plein largeur | Taille contenu | Taille contenu |
| `UNKNOWN` lisible | ✅ | ✅ | ✅ | ✅ |
| Focus visible | ✅ | ✅ | ✅ | ✅ |
| Keyboard-first | ✅ | ✅ | ✅ | ✅ |

### 9.3 Test par Mode

| Test | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|------|---------|----------|-----------|----------|-------|
| Accessibilité | ✅ Haute | ✅ Haute | ⚠️ Moyenne | ⚠️ Moyenne | ❌ Basse |
| Texte lisible | ✅ | ✅ | ✅ | ✅ | ✅ |
| Glyphes lisibles | ✅ | ✅ | ✅ | ✅ | ✅ (texte) |
| Métriques contextuelles | ✅ | ✅ | ✅ | ✅ | ✅ |
| `UNKNOWN != ZERO` | ✅ | ✅ | ✅ | ✅ | ✅ |
| Keyboard-first | ✅ | ✅ | ✅ | ✅ | ✅ |
| Focus visible | ✅ | ✅ | ✅ | ✅ | ✅ |
| Fallback ASCII | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## 10. Références

- `docs/design/status-language.md` — Vocabulaire de statut et `UNKNOWN != ZERO`
- `docs/design/design-tokens.md` — Tokens de couleur et focus
- `docs/design/components.md` — Composants et leur accessibilité
- `docs/design/terminal-compatibility.md` — Compatibilité terminale et fallback
- `docs/design/density.md` — Accessibilité par niveau de densité
- `docs/architecture/architecture-overview.md` — Architecture système
- `docs/architecture/state-contracts.md` — `TuiState` et accessibilité
- `docs/state/domain-state.md` — États de domaine
- `docs/state/unknown-data.md` — Protocole `UNKNOWN != ZERO` détaillé
- `docs/state/presentation-state.md` — Présentation state
- Prompt `02-ux-information-architecture.md` — Source du mandat
