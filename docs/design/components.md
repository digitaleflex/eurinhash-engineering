# Components — Composants Terminaux Réutilisables

> Composants UI terminaux pour EurinHash OpenCode.
> Tous les composants sont construits à partir des design tokens et du statut vocabulary.
> Terminal-native, pas de web UI.

---

## 1. Architecture des Composants

### 1.1 Principes

- Chaque composant lit ses couleurs depuis `theme.current`
- Aucun style hardcodé dans la logique métier
- Chaque composant est composé de : statut, glyphe, texte, métriques
- `thinkingOpacity` est appliqué par le composant quand le contexte le justifie
- Les composants sont indépendants du thème (dark/light)

### 1.2 Composition de Base

Tout composant est construit avec :

```
[GLYCHE] LIBELLÉ · DONNÉES · MÉTRIQUES
```

Les composants ne doivent jamais contenir de couleur arbitraire — uniquement les tokens `TuiThemeCurrent`.

---

## 2. Composants de Statut

### 2.1 StatusIndicator

Affiche le statut courant d'un agent ou service.

**Construit avec :**
- Glyphe statut + libellé + couleur `TuiThemeCurrent`
- `thinkingOpacity` appliqué si THINKING

**Exemple :**
```
[● ACTIVE] eurinhash-agent · 38%
[◐ THINKING] worker-novita · 2.3s
[✗ ERROR] worker-google · Connection refused
```

**Propriétés :**
| Propriété | Type | Description |
|---|---|---|
| `status` | Status enum | L'un des 11 statuts |
| `label` | string | Nom de l'agent/service |
| `metric` | string | Métrique optionnelle |
| `thinkingOpacity` | number | Appliqué si THINKING |

---

### 2.2 StatusBar

Barre de statut globale affichée en bas ou en haut du terminal.

**Construit avec :**
- Liste de `StatusIndicator`
- Fond `backgroundPanel` + bordure `border`

**Exemple :**
```
[● ACTIVE] eurinhash · [◐ THINKING] novita · [✗ ERROR] google · [○ IDLE] pollinations
```

**Propriétés :**
| Propriété | Type | Description |
|---|---|---|
| `items` | StatusIndicator[] | Liste des statuts |
| `position` | "top" \| "bottom" | Position de la barre |
| `compact` | boolean | Mode dense si true |

---

## 3. Composants de Données

### 3.1 MetricDisplay

Affiche une métrique avec format standardisé.

**Construit avec :**
- `mono` pour la valeur
- `textMuted` pour le label
- `space-sm` entre label et valeur

**Formats validés :**
- `38%` — pourcentage
- `100K / 262K · 38%` — utilisation avec limite
- `100K · 38%` — limite inconnue
- `$0.00` — coût nul confirmé
- `N/A` — coût inconnu

**Règle :** `UNKNOWN != ZERO` — jamais afficher `0` pour une donnée inconnue

**Exemple :**
```
Coût : $0.00
Utilisation : 100K / 262K · 38%
Limite : N/A
```

---

### 3.2 ProgressBar

Barre de progression avec format terminal.

**Construit avec :**
- `primary` pour le segment rempli
- `backgroundElement` pour le segment vide
- `text` pour le pourcentage

**Formats par largeur :**
- ≥ 120 cols : `[████████░░] 38%`
- 80–119 cols : `38%` (texte seul)

**Exemple :**
```
[████████░░░░] 38%
[████░░░░░░░░] 38%
38%
```

---

### 3.3 Table

Table de données terminale.

**Construit avec :**
- `border` pour les séparateurs
- `text` pour le contenu
- `textMuted` pour les headers
- `backgroundElement` pour la ligne sélectionnée

**Adaptation par largeur :**
| Largeur | Format |
|---|---|
| ≥ 120 | Table complète |
| 100–119 | Table compacte |
| 80–99 | Liste clé-valeur |
| < 80 | JSON lines |

**Exemple (largeur ≥ 120) :**
```
┌─────────────┬──────────┬──────────┐
│ Name        │ Status   │ Progress │
├─────────────┼──────────┼──────────┤
│ eurinhash   │ ACTIVE   │ 38%      │
│ novita      │ THINKING │ 23%      │
│ google      │ ERROR    │ N/A      │
└─────────────┴──────────┴──────────┘
```

---

### 3.4 KeyValueList

Liste clé-valeur compacte.

**Construit avec :**
- `label` pour la clé
- `text` pour la valeur
- `space-sm` entre clé et valeur

**Exemple :**
```
Nom:        eurinhash
Statut:     ACTIVE
Progression: 38%
Coût:       $0.00
```

---

## 4. Composants de Contrôle

### 4.1 Panel

Panneau de contenu avec bordures et padding.

**Construit avec :**
- `border` pour la bordure
- `backgroundPanel` pour le fond
- `corner` characters (`┌┐└┘`)
- `space-md` pour le padding

**Adaptation par largeur :**
| Largeur | Bordure |
|---|---|
| ≥ 120 | Coins + bordures complètes |
| 100–119 | Bordures simples |
| 80–99 | Bordure supérieure |
| < 80 | Ligne de séparation |

**Exemple :**
```
╭──────────────────────────────────────────╮
│  Information Title                         │
│                                          │
│  Content inside the panel.               │
│  Using textMuted for secondary info.     │
╰──────────────────────────────────────────╯
```

---

### 4.2 Alert

Boîte d'alerte avec statut visuel.

**Construit avec :**
- Glyphe de statut + `border` + couleur de statut
- `backgroundPanel` pour le fond

**Niveaux :**
| Niveau | Glyphe | Couleur |
|---|---|---|
| Info | `ℹ` | `info` |
| Success | `✓` | `success` |
| Warning | `⚠` | `warning` |
| Error | `✗` | `error` |

**Exemple :**
```
╭─ ⚠ WARNING ─────────────────────────╮
│ Approchant la limite de 262K tokens  │
╰──────────────────────────────────────╯
```

---

### 4.3 Dialog

Dialogue modale pour confirmation, sélection, saisie.

**Construit avec :**
- `borderActive` pour le focus
- `backgroundMenu` pour le fond
- `selectedListItemText` pour l'élément sélectionné

**Types :**
| Type | Usage |
|---|---|
| Confirm | Validation d'action |
| Alert | Information importante |
| Prompt | Saisie utilisateur |
| Select | Choix parmi des options |

**Propriétés :**
| Propriété | Type | Description |
|---|---|---|
| `title` | string | Titre du dialogue |
| `message` | string | Message ou description |
| `onConfirm` | function | Action de confirmation |
| `onCancel` | function | Action d'annulation |
| `options` | SelectOption[] | Options de sélection |
| `size` | medium/large/xlarge | Taille du dialogue |

---

### 4.4 Prompt

Zone de saisie de commande.

**Construit avec :**
- `primary` pour le curseur/indicateur
- `text` pour la saisie
- `textMuted` pour le placeholder
- `accent` pour le focus

**Modes :**
| Mode | Indicateur | Usage |
|---|---|---|
| Normal | `›` | Commande standard |
| Shell | `$` | Commande shell |

**Exemple :**
```
› _ /density STANDARD
$ _ ls -la
```

---

## 5. Composants de Navigation

### 5.1 Sidebar

Navigation latérale pour les sessions, agents, MCP, LSP.

**Construit avec :**
- `borderSubtle` pour les séparations
- `backgroundElement` pour l'item sélectionné
- `textMuted` pour les items inactifs
- `selectedListItemText` pour l'item sélectionné

**Adaptation par largeur :**
| Largeur | Affichage |
|---|---|
| ≥ 120 | Tous les items avec labels |
| 100–119 | Labels tronqués |
| 80–99 | Icônes uniquement |
| < 80 | Masquée (hamburger) |

---

### 5.2 Menu

Menu dropdown pour les options et configurations.

**Construit avec :**
- `backgroundMenu` pour le fond
- `border` pour la bordure
- `selectedListItemText` pour la sélection

---

### 5.3 Breadcrumb

Fil d'Ariane pour la navigation dans les résultats.

**Construit avec :**
- `textMuted` pour les segments
- `borderSubtle` comme séparateur `›`
- `text` pour le segment courant

---

## 6. Composants de Feedback

### 6.1 Toast

Notification éphémère en bas ou en haut du terminal.

**Construit avec :**
- Couleur de statut (`success`, `warning`, `error`, `info`)
- `backgroundPanel` pour le fond
- `border` pour la bordure

**Types :**
| Type | Couleur | Durée |
|---|---|---|
| info | `info` | Courte |
| success | `success` | Courte |
| warning | `warning` | Moyenne |
| error | `error` | Longue |

---

### 6.2 Spinner

Indicateur de chargement.

**Construit avec :**
- `primary` pour la rotation
- `textMuted` pour le label

**Glyphs :**
| État | Glyphe |
|---|---|
| Chargement | `⟳` ou `◌` |
| Chargement THINKING | `◐` avec `thinkingOpacity` |
| Succès | `✓` |
| Échec | `✗` |

---

### 6.3 SelectionList

Liste sélectionnable avec focus.

**Construit avec :**
- `backgroundElement` + `selectedListItemText` pour la sélection
- `borderActive` pour le focus actuel
- `borderSubtle` pour les items non sélectionnés

---

## 7. Composants de Visualisation

### 7.1 DiffView

Affichage de diff avec couleurs dédiées.

**Construit avec :**
- `diffAdded` / `diffAddedBg` pour les ajouts
- `diffRemoved` / `diffRemovedBg` pour les retraits
- `diffContext` / `diffContextBg` pour le contexte
- `diffLineNumber` pour les numéros de ligne
- `diffHunkHeader` pour les en-têtes de hunk

**Exemple :**
```
+ line added (diffAdded)
- line removed (diffRemoved)
  line context (diffContext)
@@ hunk header (diffHunkHeader)
```

---

### 7.2 MarkdownView

Rendu de contenu markdown en terminal.

**Construit avec :**
- `markdownHeading` pour les titres
- `markdownText` pour le texte courant
- `markdownCode` pour le code inline
- `markdownCodeBlock` pour les blocs de code
- `markdownLink` pour les liens
- `markdownBlockQuote` pour les citations
- `markdownStrong` pour le gras
- `markdownEmph` pour l'italique

---

### 7.3 SyntaxView

Coloration syntaxique du code.

**Construit avec :**
- `syntaxComment` pour les commentaires
- `syntaxKeyword` pour les mots-clés
- `syntaxFunction` pour les noms de fonction
- `syntaxString` pour les chaînes
- `syntaxNumber` pour les nombres
- `syntaxType` pour les types
- `syntaxOperator` pour les opérateurs
- `syntaxPunctuation` pour la ponctuation
- `syntaxVariable` pour les variables

---

## 8. Composants d'Information

### 8.1 Badge

Marqueur textuel avec préfixe.

**Construit avec :**
- Glyphe statut + libellé
- Couleur du statut

**Exemples :**
```
[✓ SUCCESS] · [⚠ WARNING] · [✗ ERROR]
[● ACTIVE] · [○ IDLE] · [◐ THINKING]
```

---

### 8.2 Separator

Séparateur horizontal ou vertical.

**Construit avec :**
- `border` pour les séparateurs standards
- `borderSubtle` pour les séparateurs légers
- `borderActive` pour les séparateurs forts

**Types :**
| Type | Glyphe | Usage |
|---|---|---|
| Horizontal | `─` | Séparation entre sections |
| Vertical | `│` | Séparation entre colonnes |
| Fort | `═` | Début/fin de bloc |
| Léger | `┄` | Séparation subtile |

---

### 8.3 Caption

Texte de métadonnée ou timestamp.

**Construit avec :**
- `textMuted` + `italic`
- `mono` pour les timestamps et métriques

**Exemple :**
```
2026-09-19 12:00:00 · 38% · 100K / 262K
```

---

## 9. Composants de Commande

### 9.1 CommandSurface

Surface de commande pour les commandes utilisateur.

**Construit avec :**
- `backgroundMenu` pour le fond
- `border` pour les bordures
- `accent` pour le commande active

**Exemple :**
```
COMMANDS
──────────────
  /density STANDARD   Change density level
  /terminal info      Show terminal info
  /theme dark         Switch to dark mode
──────────────
```

---

### 9.2 KeybindDisplay

Affichage des raccourcis clavier.

**Construit avec :**
- `mono` pour les touches
- `textMuted` pour les descriptions
- `space-sm` entre touche et description

**Exemple :**
```
Ctrl+D  → Change density
/       → Command palette
Tab     → Next item
Enter   → Confirm
```

---

## 10. Règles de Composants

### 10.1 Règles Fondamentales

1. **JAMAIS** hardcoder des couleurs dans un composant — utiliser `theme.current`
2. **TOUJOURS** fournir un fallback textuel pour chaque glyphe
3. **JAMAIS** un composant ne doit dépendre de la couleur seule
4. `thinkingOpacity` est appliqué UNIQUEMENT aux éléments en état THINKING
5. Chaque composant DOIT avoir un fallback pour 80 colonnes
6. `UNKNOWN != ZERO` dans tous les composants
7. Les métriques sont TOUJOURS en `mono`

### 10.2 Règles de Composition

1. Chaque composant est construit à partir des tokens existants
2. Pas de nouvelles couleurs — seulement les tokens `TuiThemeCurrent`
3. Les bordures utilisent `border`, `borderActive`, `borderSubtle`
4. Les fonds utilisent `background`, `backgroundPanel`, `backgroundElement`, `backgroundMenu`
5. Le texte utilise `text`, `textMuted`, `selectedListItemText`
6. Les accents utilisent `primary`, `secondary`, `accent`
7. Les statuts utilisent `error`, `warning`, `success`, `info`

### 10.3 Règles de Non-Fabrication

1. **JAMAIS** créer une nouvelle couleur ou token
2. **JAMAIS** afficher une valeur fabriquée pour UNKNOWN
3. `N/A` est affiché tel quel — jamais transformé
4. `$0.00` est affiché tel quel — jamais supprimé
5. `100K · 38%` est affiché tel quel quand la limite est inconnue
6. Les statuts sont toujours affichés en texte + glyphe

---

## 11. Résumé des Composants

| Composant | Catégorie | Statut | Métrique | Largeur min |
|---|---|---|---|---|
| StatusIndicator | Statut | ✓ | ✓ | 40 cols |
| StatusBar | Statut | ✓ | ✓ | 60 cols |
| MetricDisplay | Donnée | — | ✓ | 20 cols |
| ProgressBar | Donnée | — | ✓ | 20 cols |
| Table | Donnée | ✓ | ✓ | 40 cols |
| KeyValueList | Donnée | — | ✓ | 20 cols |
| Panel | Contrôle | — | — | 30 cols |
| Alert | Contrôle | ✓ | — | 40 cols |
| Dialog | Contrôle | ✓ | — | 50 cols |
| Prompt | Contrôle | — | — | 20 cols |
| Sidebar | Navigation | ✓ | — | 20 cols |
| Menu | Navigation | ✓ | — | 20 cols |
| Breadcrumb | Navigation | — | — | 30 cols |
| Toast | Feedback | ✓ | — | 40 cols |
| Spinner | Feedback | — | — | 20 cols |
| SelectionList | Feedback | ✓ | — | 40 cols |
| DiffView | Visualisation | ✓ | ✓ | 60 cols |
| MarkdownView | Visualisation | — | — | 40 cols |
| SyntaxView | Visualisation | — | — | 40 cols |
| Badge | Information | ✓ | — | 20 cols |
| Separator | Information | — | — | 10 cols |
| Caption | Information | — | ✓ | 20 cols |
| CommandSurface | Commande | — | — | 40 cols |
| KeybindDisplay | Commande | — | — | 20 cols |
