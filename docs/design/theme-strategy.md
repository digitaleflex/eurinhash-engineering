# Theme Strategy — Stratégie Thème Dark/Light

> Stratégie de thème pour EurinHash OpenCode terminal-native.
> Basée sur le type `TuiTheme` (`mode: () => "dark" | "light"`).

---

## 1. Architecture du Thème

### 1.1 Type `TuiTheme`

Le système de thème OpenCode expose :

```typescript
interface TuiTheme {
  readonly current: TuiThemeCurrent;  // Tous les tokens RGBA
  readonly selected: string;          // Nom du thème actuel
  has(name: string): boolean;         // Vérifier un thème
  set(name: string): boolean;         // Activer un thème
  install(jsonPath: string): Promise<void>;  // Installer un thème
  mode(): "dark" | "light";           // Mode actuel
  readonly ready: boolean;            // Thème chargé
}
```

### 1.2 `TuiThemeCurrent`

Le thème courant expose 60+ tokens RGBA :
- **Couleurs sémantiques** : `primary`, `secondary`, `accent`, `error`, `warning`, `success`, `info`
- **Texte** : `text`, `textMuted`, `selectedListItemText`
- **Fond** : `background`, `backgroundPanel`, `backgroundElement`, `backgroundMenu`
- **Bordures** : `border`, `borderActive`, `borderSubtle`
- **Diff** : 13 tokens pour les diffs
- **Markdown** : 14 tokens pour le rendu markdown
- **Syntaxe** : 9 tokens pour la coloration syntaxique
- **Opacité** : `thinkingOpacity` (nombre)

---

## 2. Mode Dark (Défaut)

Le mode dark est le **mode par défaut** pour le terminal EurinHash.

### 2.1 Justification

- Les terminaux sont historiquement sombres (black background, green/amber text)
- Le dark réduit la fatigue oculaire en session prolongée
- Les couleurs d'accent sont plus saturées sur fond sombre
- `thinkingOpacity` est plus visible sur fond sombre

### 2.2 Palette Dark

| Token | Rôle | Valeur typique (référence) |
|---|---|---|
| `background` | Fond global | Noir profond |
| `backgroundPanel` | Fond panneau | Noir légèrement clair |
| `backgroundElement` | Fond élément | Noir moyen |
| `backgroundMenu` | Fond menu | Noir avec bordure |
| `text` | Texte principal | Blanc/Gris clair |
| `textMuted` | Texte secondaire | Gris moyen |
| `primary` | Couleur principale | Accent vif |
| `secondary` | Couleur secondaire | Accent moyen |
| `accent` | Accent | Couleur d'interaction |
| `error` | Erreur | Rouge/Corail |
| `warning` | Avertissement | Ambre |
| `success` | Succès | Vert |
| `info` | Information | Cyan/Bleu |
| `border` | Bordure | Gris foncé |
| `borderActive` | Bordure active | Accent |
| `borderSubtle` | Bordure subtile | Gris très foncé |

### 2.3 Règles Dark

- Le `background` est TOUJOURS plus sombre que `backgroundPanel`
- Le `text` est TOUJOURS plus clair que `textMuted`
- Les couleurs de statut (`error`, `warning`, `success`, `info`) sont saturées sur fond sombre
- `thinkingOpacity` est appliqué aux éléments `secondary` en état THINKING
- Les bordures `borderSubtle` sont quasi-invisibles sur fond sombre

---

## 3. Mode Light

Le mode light est disponible pour les utilisateurs qui préfèrent un terminal clair.

### 3.1 Justification

- Certains utilisateurs préfèrent le light pour la lisibilité en plein jour
- Les écrans externes haute luminosité bénéficient du mode light
- Compatibilité avec certains terminaux qui imposent le light

### 3.2 Palette Light

| Token | Rôle | Valeur typique (référence) |
|---|---|---|
| `background` | Fond global | Blanc/Gris très clair |
| `backgroundPanel` | Fond panneau | Blanc/Gris clair |
| `backgroundElement` | Fond élément | Gris très clair |
| `backgroundMenu` | Fond menu | Blanc |
| `text` | Texte principal | Noir/Gris foncé |
| `textMuted` | Texte secondaire | Gris moyen |
| `primary` | Couleur principale | Accent foncé mais visible |
| `secondary` | Couleur secondaire | Accent moyen |
| `accent` | Accent | Couleur d'interaction |
| `error` | Erreur | Rouge foncé |
| `warning` | Avertissement | Ambre foncé |
| `success` | Succès | Vert foncé |
| `info` | Information | Bleu foncé |
| `border` | Bordure | Gris moyen |
| `borderActive` | Bordure active | Accent |
| `borderSubtle` | Bordure subtile | Gris très clair |

### 3.3 Règles Light

- Le `background` est TOUJOURS plus clair que `backgroundPanel`
- Le `text` est TOUJOURS plus foncé que `textMuted`
- Les couleurs de statut sont moins saturées sur fond clair
- `thinkingOpacity` fonctionne de la même manière que sur dark
- Les bordures `borderSubtle` sont visibles mais subtiles sur fond clair
- Le `diffAdded` et `diffRemoved` inversent leurs rôles relatifs entre dark et light

---

## 4. Détection et Sélection

### 4.1 Détection Automatique

Le système détecte automatiquement le mode préféré :

1. **Variable d'environnement** : `TERM_COLOR_MODE` ou `NO_COLOR` pour désactiver
2. **Préférence utilisateur** : Dans la configuration OpenCode (`theme.mode`)
3. **Système d'exploitation** : `prefers-color-scheme` si disponible
4. **Manuel** : Commande `/theme dark` ou `/theme light`

### 4.2 Priorité de Détection

```
1. Configuration utilisateur explicite (theme.mode)
2. Variable d'environnement
3. Préférence système OS
4. Default: DARK
```

### 4.3 Règle de Non-Override

- Si l'utilisateur a explicitement choisi un mode, NE PAS changer automatiquement
- Le mode automatique ne s'active que si l'utilisateur n'a pas configuré de préférence
- `TuiTheme.ready` DOIT être `true` avant d'appliquer un thème

---

## 5. Changement de Thème en Runtime

### 5.1 API de Changement

```typescript
// Vérifier le mode actuel
const currentMode = theme.mode(); // "dark" | "light"

// Changer de thème
theme.set("my-custom-theme");

// Installer un nouveau thème
await theme.install("/path/to/theme.json");

// Vérifier la disponibilité
theme.has("another-theme");
```

### 5.2 Comportement au Changement

1. TOUS les tokens sont rechargés via `theme.current`
2. Les composants UI se mettent à jour automatiquement
3. `thinkingOpacity` reste inchangé (c'est un nombre, pas une couleur)
4. Les bordures et backgrounds changent de teinte
5. La syntaxe et le markdown changent de couleurs
6. `thinkingOpacity` s'applique de manière identique dans les deux modes

### 5.3 Règles de Transition

- Le changement de thème NE DOIT PAS entraîner de perte de données
- Les statuts (`ACTIVE`, `ERROR`, etc.) restent lisibles dans les deux modes
- Les glyphes ne changent PAS lors d'un changement de thème
- La densité (`MINIMAL`, `STANDARD`, etc.) est indépendante du thème
- `UNKNOWN` et `N/A` restent affichés de la même manière dans les deux modes

---

## 6. Thèmes Personnalisés

### 6.1 Installation de Thèmes

Les thèmes personnalisés sont installés via `theme.install(jsonPath)`.
Le fichier JSON doit contenir :

```json
{
  "name": "my-theme",
  "mode": "dark",
  "tokens": {
    "primary": "rgba(0, 200, 100, 1)",
    "background": "rgba(10, 10, 10, 1)",
    "...": "..."
  }
}
```

### 6.2 Règles de Thèmes Personnalisés

1. UNIQUEMENT les tokens définis dans `TuiThemeCurrent` peuvent être surchargés
2. NE JAMAIS inventer de nouveaux tokens de couleur
3. `thinkingOpacity` reste un `number` — pas de couleur
4. Le thème DOIT définir TOUS les tokens requis — pas de valeurs manquantes
5. Un thème personnalisé DOIT être validé avant installation

---

## 7. Accessibilité Thématique

### 7.1 Contraste

| Paire | Ratio minimum |
|---|---|
| `text` / `background` | 4.5:1 |
| `textMuted` / `background` | 3:1 |
| `primary` / `background` | 3:1 |
| `error` / `background` | 4.5:1 |
| `success` / `background` | 3:1 |

### 7.2 Mode Sans Couleur

Pour les utilisateurs malvoyants ou les terminaux monochromes :

- `borderSubtle` → `dim` attribute
- `borderActive` → `bold + underline`
- `error` → `bold + underline`
- `success` → `underline`
- `warning` → `bold + dim`
- `info` → `reverse-video`
- `thinkingOpacity` → `dim` attribute

### 7.3 Règles d'Accessibilité

1. Le texte est TOUJOURS lisible sans couleur
2. Les statuts sont TOUJOURS identifiables par le glyphe + texte
3. `UNKNOWN` reste `?` dans tous les modes
4. `N/A` reste `N/A` dans tous les modes
5. Le mode sans couleur est activable via configuration

---

## 8. Règles de Thème

1. **DARK** est le mode par défaut — jamais changer sans consentement explicite
2. **JAMAIS** hardcoder des valeurs de couleur — utiliser `theme.current`
3. **TOUJOURS** vérifier `theme.ready` avant d'appliquer un thème
4. **JAMAIS** mélanger des tokens de deux modes dans la même ligne
5. `thinkingOpacity` est **indépendant** du thème (c'est un nombre)
6. Les statuts visuels (glyphes + texte) sont **indépendants** du thème
7. `UNKNOWN != ZERO` dans tous les modes de thème
8. Les métriques télémetry (`38%`, `100K / 262K`, `$0.00`, `N/A`) sont en `mono` et ne changent pas de couleur de fond

---

## 9. Intégration avec `TuiThemeCurrent`

### 9.1 Mapping des Tokens

Chaque composant UI lit directement depuis `theme.current` :

```
Background: theme.current.background
Text:       theme.current.text
Status:     theme.current[success|warning|error|info]
Border:     theme.current[border|borderActive|borderSubtle]
```

### 9.2 Règle de Lecture Dynamique

- TOUS les composants lisent les couleurs à partir de `theme.current`
- JAMAIS de valeur hardcodée dans le code UI
- `TuiThemeCurrent` est `readonly` — les composants ne doivent pas tenter de le modifier
- `has()`, `set()`, `install()` sont les seules méthodes de mutation de thème

### 9.3 Vérification de la Disponibilité

Avant d'utiliser un token :

```typescript
if (theme.has("custom-token")) {
  const color = theme.current.customToken;
} else {
  // Fallback vers token standard
}
```

---

## 10. Résumé des Décisions

| Décision | Valeur |
|---|---|
| Mode par défaut | DARK |
| Changement de mode | Manuel ou détection OS |
| Transition de thème | Instantanée (pas d'animation terminale) |
| Accessibilité | Mode sans couleur disponible |
| Personnalisation | Via `theme.install(jsonPath)` |
| Lecture des couleurs | Dynamique via `theme.current` |
| `thinkingOpacity` | Indépendant du thème |
| `UNKNOWN` | Ne change jamais, quel que soit le thème |
