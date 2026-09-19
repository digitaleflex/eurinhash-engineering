# Design Tokens — HashCode Terminal Design System

> Système de tokens pour l'interface terminale d'EurinHash OpenCode.
> Basé sur les primitives `TuiThemeCurrent` d'OpenCode (`@opencode-ai/plugin`).
> Terminal-native, pas de déclinaison web UI.

---

## 1. Couleurs (RGBA)

Toutes les couleurs sont exprimées en RGBA via le type `RGBA` d'`@opentui/core`.
Chaque token est une valeur `{ r: 0–255, g: 0–255, b: 0–255, a: 0–1 }`.

### 1.1 Couleurs sémantiques primaires

| Token | Rôle | Usage terminal |
|---|---|---|
| `primary` | Couleur principale de marque | En-têtes, accentuation CLI, prompts actifs |
| `secondary` | Couleur secondaire | Sous-titres, informations de rang 2 |
| `accent` | Accent d'interaction | Focus, sélection, éléments cliquables |
| `text` | Texte principal | Contenu lisible par défaut |
| `textMuted` | Texte secondaire | Placeholders, labels inactifs, métadonnées |

### 1.2 Couleurs de statut

| Token | Rôle | Usage terminal |
|---|---|---|
| `success` | Succès | Validation, completion, `✓` |
| `warning` | Avertissement | Condamnation douce, `⚠` |
| `error` | Erreur | Échec, rupture, `✗` |
| `info` | Information | Contexte neutre, notifications |

### 1.3 Couleurs de fond

| Token | Rôle | Usage terminal |
|---|---|---|
| `background` | Fond global | Fond du terminal / viewport |
| `backgroundPanel` | Fond de panneau | Boîtes de dialogue, fiches |
| `backgroundElement` | Fond d'élément | Ligne de liste, carte interactive |
| `backgroundMenu` | Fond de menu | Dropdowns, selections, palettes |

### 1.4 Couleurs de bordure

| Token | Rôle | Usage terminal |
|---|---|---|
| `border` | Bordure standard | Séparateurs, frames, lignes |
| `borderActive` | Bordure active | Focus actuel, élément sélectionné |
| `borderSubtle` | Bordure subtile | Délimitations légères, groupes |

### 1.5 Couleurs de texte sélectionné

| Token | Rôle |
|---|---|
| `selectedListItemText` | Texte d'élément de liste sélectionné |

### 1.6 Couleurs de diff

| Token | Rôle |
|---|---|
| `diffAdded` | Texte ajouté dans un diff |
| `diffRemoved` | Texte supprimé dans un diff |
| `diffContext` | Ligne de contexte dans un diff |
| `diffHunkHeader` | En-tête de hunk |
| `diffHighlightAdded` | Surlignage ajout |
| `diffHighlightRemoved` | Surlignage retrait |
| `diffAddedBg` | Fond ligne ajoutée |
| `diffRemovedBg` | Fond ligne retirée |
| `diffContextBg` | Fond ligne contexte |
| `diffLineNumber` | Numéro de ligne diff |
| `diffAddedLineNumberBg` | Fond numéro ligne ajoutée |
| `diffRemovedLineNumberBg` | Fond numéro ligne retirée |

### 1.7 Couleurs Markdown

| Token | Rôle |
|---|---|
| `markdownText` | Texte markdown courant |
| `markdownHeading` | Titres markdown |
| `markdownLink` | Liens markdown |
| `markdownLinkText` | Texte de lien markdown |
| `markdownCode` | Code inline |
| `markdownBlockQuote` | Bloc citation |
| `markdownEmph` | Emphasis (italique) |
| `markdownStrong` | Fort emphasis (gras) |
| `markdownHorizontalRule` | Règle horizontale |
| `markdownListItem` | Élément de liste |
| `markdownListEnumeration` | Numéro de liste |
| `markdownImage` | Image markdown |
| `markdownImageText` | Texte alternatif image |
| `markdownCodeBlock` | Bloc de code |

### 1.8 Couleurs Syntaxe (tokenisation)

| Token | Rôle |
|---|---|
| `syntaxComment` | Commentaire |
| `syntaxKeyword` | Mot-clé |
| `syntaxFunction` | Nom de fonction |
| `syntaxVariable` | Variable |
| `syntaxString` | Chaîne de caractères |
| `syntaxNumber` | Nombre littéral |
| `syntaxType` | Nom de type |
| `syntaxOperator` | Opérateur |
| `syntaxPunctuation` | Ponctuation |

### 1.9 Opacité de réflexion

| Token | Type | Rôle |
|---|---|---|
| `thinkingOpacity` | `number` | Opacité appliquée aux éléments en état THINKING (réflexion) |

---

## 2. Spacing

Le terminal utilise une grille basée sur la largeur de caractère (monospace).
Tous les espacements sont exprimés en unités de colonne (`col`) ou en lignes (`row`).

### 2.1 Échelle de spacing

| Token | Valeur | Usage |
|---|---|---|
| `space-xs` | 1 col | Espacement intra-caractère minimal |
| `space-sm` | 2 cols | Entre éléments proches (label/valeur) |
| `space-md` | 4 cols | Entre sections, padding de panneau |
| `space-lg` | 8 cols | Entre blocs majeurs |
| `space-xl` | 12 cols | Segmentation de haut niveau |
| `space-2xl` | 16 cols | Séparation de page |

### 2.2 Règles de spacing terminal

- **1 col** = largeur d'un caractère monospace dans la police courante
- Pas de pixels : le terminal rend tout en cellules caractères
- `space-sm` (2 cols) minimum pour séparer visuellement deux éléments sur une même ligne
- `space-md` (4 cols) minimum pour un padding de panneau

---

## 3. Typographie

La typographie terminale est contrainte par les capacités du terminal :
pas de fontes variables, pas de styles dynamiques, pas de transformations CSS.

### 3.1 Hiérarchie

| Niveau | Construire avec | Usage |
|---|---|---|
| `display` | `bold` + `primary` + 2× spacing avant | Titres de section, headers de panneau |
| `heading` | `bold` + `text` | Titres de composant |
| `body` | `regular` + `text` | Contenu principal |
| `label` | `bold` + `textMuted` | Labels, champs, clés |
| `mono` | `font-mono` + `textMuted` | Code, identifiants, tokens, nombres |
| `caption` | `italic` + `textMuted` | Notes, métadonnées, timestamps |

### 3.2 Poids de police disponibles

| Poids | Support terminal |
|---|---|
| `regular` | La plupart des fontes monospace |
| `bold` | Universellement supporté |
| `italic` | Supporté par la plupart des fontes |

### 3.3 Familles de fontes

| Famille | Usage | Fallback |
|---|---|---|
| `mono` | Code, tokens, nombres, identifiants | `Menlo`, `Consolas`, `Courier New`, `Liberation Mono` |
| `proportional` | Texte courant, labels | `Inter`, `Roboto`, `Segoe UI`, `DejaVu Sans` |

### 3.4 Règles typographiques

- JAMAIS de mix `mono` et `proportional` sur la même ligne sans séparation claire
- `mono` pour toute donnée technique : IDs, pourcentages, coûts, codes
- `caption` pour les timestamps et les informations de métadonnées
- `label` en `bold` pour les clés de données (ex: `Nom: valeur`)

---

## 4. Séparateurs

| Type | Construire avec | Usage |
|---|---|---|
| `rule` | `border` + `─` (box-drawing) | Séparation horizontale entre sections |
| `rule-thin` | `borderSubtle` + `┄` | Séparation légère |
| `rule-strong` | `borderActive` + `═` | Séparation forte, début/fin de bloc |
| `rule-vertical` | `borderSubtle` + `│` | Séparation verticale (colonnes) |
| `corner` | `border` + `┌┐└┘` | Coins de panneau |

---

## 5. Badges

Les badges sont des marqueurs textuels préfixés par un caractère ou une couleur.
Ils ne doivent JAMAIS être le seul indicateur de statut.

### 5.1 Construction d'un badge

```
[PRÉFIXE] LIBELLÉ
```

Où `PRÉFIXE` est un glyphe ou abréviation de statut et `LIBELLÉ` est le texte.

### 5.2 Exemples

| Badge | Construire avec | Signification |
|---|---|---|
| `[● ACTIVE]` | `accent` + `ACTIVE` | Opérationnel |
| `[◐ THINKING]` | `secondary` + opacity | En réflexion |
| `[▶ RUNNING]` | `primary` + `RUNNING` | En cours d'exécution |
| `[○ IDLE]` | `textMuted` + `IDLE` | Inactif |
| `[! WARNING]` | `warning` + `WARNING` | Avertissement |
| `[✗ ERROR]` | `error` + `ERROR` | Échec |
| `[✓ SUCCESS]` | `success` + `SUCCESS` | Réussi |
| `[⊘ CANCELLED]` | `textMuted` + `CANCELLED` | Annulé |
| `[? UNKNOWN]` | `textMuted` + `UNKNOWN` | Indéterminé |
| `[⊘ DISCONNECTED]` | `error` + `DISCONNECTED` | Déconnecté |

---

## 6. Focus States

| État | Construire avec |
|---|---|
| Focus actif | `borderActive` + `accent` + `reverse-video` ou `bold` |
| Focus passif | `borderSubtle` + `textMuted` |
| Focus perdu | `border` + `text` sans modification |

---

## 7. Selection States

| État | Construire avec |
|---|---|
| Sélectionné | `backgroundElement` + `selectedListItemText` |
| Désélectionné | `background` + `text` |
| Partiellement sélectionné | `backgroundPanel` + `textMuted` |

---

## 8. Progress Bars

| Élément | Construire avec |
|---|---|
| Barre de progression | `primary` + `background` (fond) |
| Segment actif | `primary` caractère plein |
| Segment complet | `success` caractère plein |
| Segment échoué | `error` caractère plein |
| Segment warn | `warning` caractère plein |
| Fond de barre | `backgroundElement` ou `borderSubtle` |

Format terminal : `[████░░░░] 38%` ou `[████████] 100K / 262K · 38%`

---

## 9. Alerts

| Niveau | Construire avec | Glyphe |
|---|---|---|
| Info | `info` + `border` | `ℹ` |
| Success | `success` + `border` | `✓` |
| Warning | `warning` + `border` | `⚠` |
| Error | `error` + `border` | `✗` |

Structure d'une alerte :
```
╭─ [GLYCHE] TITRE ─────────────────────╮
│                                        │
│  Message d'alerte en texte normal       │
│                                        │
╰────────────────────────────────────────╯
```

---

## 10. Conventions Télémetry

Standardisation des représentations de données :

| Format | Condition |
|---|---|
| `38%` | Pourcentage simple |
| `100K / 262K · 38%` | Utilisation avec limite connue |
| `100K · 38%` | Limite inconnue |
| `$0.00` | Coût connu et nul |
| `N/A` | Coût inconnu |

---

## 11. Règles Géninales des Tokens

1. **NE JAMAIS** créer de nouvelles couleurs arbitrairement — utiliser uniquement les tokens `TuiThemeCurrent` existants
2. **TOUT** statut visuel DOIT avoir une structure texte/glyphe en complément de la couleur
3. **TOUJOURS** utiliser `mono` pour les nombres, IDs, pourcentages, coûts
4. **JAMAIS** de `UNKNOWN` == `ZERO` — les valeurs indéterminées restent `N/A` ou `?`
5. Les tokens sont lus dynamiquement via `theme.current` — jamais hardcodés
6. `thinkingOpacity` s'applique uniquement aux éléments visuels en état THINKING
