# Preferences — Préférences Utilisateur

> Système de préférences utilisateur pour le HashCode Terminal.
> Gestion des préférences persistantes sans fuite dans la logique de domaine.
> **Règle fondamentale** : `User preferences must not leak into domain logic`

---

## 1. Vue d'ensemble

Les préférences utilisateur sont des choix de présentation et de comportement qui persistent entre les sessions. Elles contrôlent l'apparence et le comportement du terminal **sans modifier** la logique métier.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PRÉFÉRENCES UTILISATEUR                           │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  CATÉGORIE         │  EXEMPLES                              │  │
│  │                    │                                        │  │
│  │  Apparence         │  theme, density, sidebar visibility     │  │
│  │  Interaction       │  keybindings, focus behavior            │  │
│  │  Feedback          │  notifications, attention behavior      │  │
│  │  Télémetry         │  telemetry visibility, activity verbosity│  │
│  │  Commandes         │  command aliases                         │  │
│  │  UI Persistante    │  layout state, last zone, scroll pos    │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                        │
│  Stockage : ~/.config/opencode/opencode.jsonc                        │
│  Règle : Les préférences ne fuient PAS dans le domain logic          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Catégories de Préférences

### 2.1 Apparence (Appearance)

Contrôle l'apparence visuelle du terminal.

| Préférence | Type | Valeurs | Défaut | Description |
|------------|------|---------|--------|-------------|
| `theme` | string | `"dark"` \| `"light"` | `"dark"` | Mode de couleur |
| `density` | enum | `MINIMAL` \| `STANDARD` \| `DEVELOPER` \| `OBSERVER` \| `DEBUG` | `STANDARD` | Niveau de densité |
| `sidebarVisible` | boolean | `true` \| `false` | `true` | Visibilité de la sidebar |
| `sidebarWidth` | number | 20–35 | 25 | Largeur de la sidebar en colonnes |
| `fontSize` | number | 10–20 | 12 | Taille de la police |
| `fontFamily` | string | Nom de la police | `Mono` | Police du terminal |
| `showTimestamps` | boolean | `true` \| `false` | `true` | Affichage des timestamps |
| `showMetrics` | boolean | `true` \| `false` | `true` | Affichage des métriques |

**Règles** :
1. `theme` est lu via `theme.current` — jamais hardcodé
2. `density` NE change PAS le domain logic
3. `sidebarVisible` est une préférence de présentation uniquement
4. Les valeurs `UNKNOWN` dans les préférences → échec sûr → valeur par défaut

### 2.2 Interaction

Contrôle le comportement d'interaction.

| Préférence | Type | Valeurs | Défaut | Description |
|------------|------|---------|--------|-------------|
| `keybindings` | object | Mapping touche → action | Voir keymap | Personnalisation des raccourcis |
| `focusBehavior` | enum | `"strict"` \| `"loose"` | `"strict"` | Mode de gestion du focus |
| `autoComplete` | boolean | `true` \| `false` | `true` | Activation de la complétion auto |
| `tabCycle` | enum | `"zones"` \| `"items"` | `"zones"` | Comportement de Tab |
| `escapeBehavior` | enum | `"back"` \| `"cancel"` | `"back"` | Comportement de Escape |
| `undoLimit` | number | 1–50 | 20 | Nombre d'actions undo mémorisées |

**Règles** :
1. Les keybindings personnalisés NE DOIVENT PAS entrer en conflit avec les raccourcis système
2. `focusBehavior` strict = un seul focus ; loose = focus multiple limité
3. `escapeBehavior` `"back"` = retour zone précédente ; `"cancel"` = annulation uniquement
4. `tabCycle` `"zones"` = cycle entre zones ; `"items"` = cycle dans les items de la zone

### 2.3 Feedback

Contrôle les mécanismes de feedback.

| Préférence | Type | Valeurs | Défaut | Description |
|------------|------|---------|--------|-------------|
| `notificationsEnabled` | boolean | `true` \| `false` | `true` | Activation des notifications |
| `toastDuration` | enum | `"short"` \| `"medium"` \| `"long"` | `"medium"` | Durée par défaut des toasts |
| `soundEnabled` | boolean | `true` \| `false` | `false` | Feedback sonore |
| `attentionBehavior` | enum | `"toast"` \| `"alert"` \| `"both"` | `"toast"` | Comportement d'attention |
| `showProgress` | boolean | `true` \| `false` | `true` | Affichage des progress bars |
| `animateTransitions` | boolean | `true` \| `false` | `false` | Animations de transition |

**Règles** :
1. Les notifications DOIVENT toujours avoir une raison
2. `attentionBehavior` `"alert"` = alertes bloquantes ; `"toast"` = notifications éphémères
3. `animateTransitions` `false` est recommandé pour les terminaux lents
4. `UNKNOWN` dans le feedback → `[? UNKNOWN]` avec raison — jamais silencieux

### 2.4 Télémetry et Activité

Contrôle la visibilité des données de télémétrie et d'activité.

| Préférence | Type | Valeurs | Défaut | Description |
|------------|------|---------|--------|-------------|
| `telemetryVisible` | boolean | `true` \| `false` | `true` | Visibilité des métriques télémétrie |
| `activityVerbosity` | enum | `"minimal"` \| `"standard"` \| `"verbose"` | `"standard"` | Niveau de détail de l'activity |
| `showCost` | boolean | `true` \| `false` | `true` | Affichage des coûts |
| `showTokens` | boolean | `true` \| `false` | `true` | Affichage des tokens |
| `showLatency` | boolean | `true` \| `false` | `false` | Affichage de la latence |

**Règles** :
1. `telemetryVisible` `false` masque les métriques mais NE CHANGE PAS les données sous-jacentes
2. `activityVerbosity` `"minimal"` = événements clés ; `"verbose"` = tous les événements
3. Les données télémétrie sont toujours calculées même si masquées
4. `UNKNOWN` dans les métriques → `N/A` — jamais `0` ou fabriqué

### 2.5 Commandes et Alias

| Préférence | Type | Valeurs | Défaut | Description |
|------------|------|---------|--------|-------------|
| `commandAliases` | object | `{ alias: command }` | `{}` | Alias pour les commandes |
| `customCommands` | array | `["/cmd1", "/cmd2"]` | `[]` | Commandes personnalisées |
| `showKeybindHints` | boolean | `true` \| `false` | `true` | Affichage des hints de raccourcis |
| `commandHistorySize` | number | 10–100 | 50 | Nombre de commandes en historique |

**Règles** :
1. Les aliases NE DOIVENT PAS écraser les commandes système
2. `commandAliases` est stocké dans la configuration utilisateur
3. Les commandes personnalisées sont validées avant exécution
4. `UNKNOWN` dans un alias → échec sûr → alias ignoré

### 2.6 UI Persistante

| Préférence | Type | Valeurs | Défaut | Description |
|------------|------|---------|--------|-------------|
| `lastFocusedZone` | string | Nom de la zone | `"CHAT"` | Dernière zone avec le focus |
| `lastDensity` | enum | Niveau de densité | `STANDARD` | Dernière densité utilisée |
| `lastTheme` | string | Nom du thème | `"dark"` | Dernier thème utilisé |
| `scrollPosition` | object | `{ zone, position }` | `{}` | Positions de scroll par zone |
| `sidebarCollapsed` | boolean | `true` \| `false` | `false` | État replié de la sidebar |
| `dialogStack` | array | Historique des dialogs | `[]` | Pile de dialogs |

**Règles** :
1. `lastFocusedZone` est restauré au démarrage
2. `scrollPosition` est limité à 10 entrées pour éviter la surcharge mémoire
3. Les préférences persistantes NE DOIVENT PAS affecter le domain logic
4. Si une préférence est corrompue → échec sûr → valeur par défaut

---

## 3. Persistance des Préférences

### 3.1 Infrastructure de Persistence

Les préférences sont stockées dans `~/.config/opencode/opencode.jsonc` :

```jsonc
{
    "theme": "dark",
    "density": "STANDARD",
    "sidebarVisible": true,
    "telemetryVisible": true,
    "activityVerbosity": "standard",
    "commandAliases": {
        "ll": "/terminal info",
        "cls": "/clear"
    },
    "keybindings": {
        "custom": {}
    },
    "lastFocusedZone": "CHAT",
    "lastDensity": "STANDARD"
}
```

### 3.2 Règles de Persistence

1. **Identifier les facilities de persistence existantes** — étendre plutôt que créer un stockage parallèle
2. `opencode.jsonc` est la source primaire pour les préférences utilisateur
3. Les préférences sont lues au démarrage et mises en cache en mémoire
4. Les changements de préférences en runtime sont persistés immédiatement
5. Un backup automatique est créé avant chaque modification majeure
6. La persistence NE DOIT PAS contenir de données de domaine

### 3.3 Validation des Préférences

Toutes les préférences sont validées à la lecture :

| Cas | Comportement |
|-----|-------------|
| Valeur invalide | → Valeur par défaut + toast `[⚠ WARNING] Invalid preference: X → default` |
| Valeur `undefined` | → Valeur par défaut |
| Valeur `UNKNOWN` | → Échec sûr → Valeur par défaut |
| Clé corrompue | → Clé ignorée + warning |
| Fichier de config introuvable | → Configuration par défaut complète |

**Règle** : `Invalid configuration must fail safely and preserve usable defaults`

---

## 4. Séparation des Responsabilités

### 4.1 Préférences vs Domain Logic

| Aspect | Préférences | Domain Logic |
|--------|-------------|--------------|
| Stockage | `opencode.jsonc` | Base de données / State |
| Changement | Manuel utilisateur | Événements système |
| Persistance | Oui | Non (in-memory) |
| Impact | Présentation uniquement | Comportement métier |
| Validation | Schema utilisateur | Contraintes métier |
| `UNKNOWN` | Échec sûr → default | `[? UNKNOWN]` |

### 4.2 Règle de Non-Fuite

**Règle fondamentale** : `User preferences must not leak into domain logic`

- Les préférences contrôlent la PRÉSENTATION
- Le domain logic fonctionne avec les DONNÉES RÉELLES
- `density` change l'affichage, PAS les calculs de tokens/cost
- `telemetryVisible` masque l'affichage, PAS les données télémétrie
- `theme` change les couleurs, PAS la sémantique des statuts
- `activityVerbosity` filtre l'affichage, PAS le flux d'événements

### 4.3 Exemples de Non-Fuite

**CORRECT** :
```typescript
// La préférence de densité change l'affichage
const density = preferences.get("density"); // "STANDARD"
renderProgressBar(percent, density); // Format différent par densité
// Mais le calcul de percent reste le même
```

**INCORRECT** :
```typescript
// La préférence ne modifie PAS les données de domaine
const density = preferences.get("density");
const tokens = session.tokens; // Toujours les vraies données
// NE PAS : tokens = density === "MINIMAL" ? recalculate(tokens) : tokens;
```

---

## 5. Préférences par Mode

| Préférence | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|------------|---------|----------|-----------|----------|-------|
| `sidebarVisible` | `false` (hamburger) | `true` | `true` (réduit) | Contextuel | `true` |
| `telemetryVisible` | `false` | `true` | `true` | `true` | `true` |
| `activityVerbosity` | `"minimal"` | `"standard"` | `"verbose"` | `"verbose"` | `"verbose"` |
| `showCost` | `true` | `true` | `true` | `true` | `true` |
| `showMetrics` | `true` | `true` | `true` | `true` | `true` |
| `showTimestamps` | `true` | `true` | `true` | `true` | `true` |

---

## 6. Gestion des Préférences par Zone

### 6.1 Lecture des Préférences

```typescript
// Lire une préférence
const density = api.preferences.get("density"); // "STANDARD"

// Lire avec fallback
const theme = api.preferences.get("theme") ?? "dark";

// Vérifier si une préférence existe
const hasCustomKeybindings = api.preferences.has("keybindings");
```

### 6.2 Écriture des Préférences

```typescript
// Changer une préférence
api.preferences.set("density", "DEVELOPER");

// Avec validation
const result = api.preferences.set("density", "INVALID");
if (!result.valid) {
    api.ui.toast({ message: `[⚠ WARNING] Invalid density: ${result.error}` });
}
```

### 6.3 Écoute des Changements

```typescript
// Écouter un changement de préférence
api.preferences.onChange("density", (newValue, oldValue) => {
    api.ui.toast({ message: `Density changed from ${oldValue} to ${newValue}` });
});
```

---

## 7. Règles Fondamentales des Préférences

1. **`User preferences must not leak into domain logic`** — Les préférences contrôlent la présentation uniquement
2. **Les préférences sont persistantes** — elles survivent aux redémarrages
3. **Les préférences sont validées** — les valeurs invalides tombent sur des defaults
4. **`UNKNOWN != ZERO`** dans toutes les préférences
5. **`Invalid configuration must fail safely`** — échec sûr avec defaults utilisables
6. **La persistence existante est étendue** — pas de stockage parallèle sans justification
7. **Les préférences sont affichables** — `KeybindDisplay` et `CommandSurface` montrent les préférences actives
8. **Le focus est préservé** — les changements de préférences ne perdent pas la position de l'utilisateur
9. **`thinkingOpacity` est indépendant** — ne dépend pas des préférences
10. **Les données télémétrie sont toujours calculées** — le masquage est une affaire de présentation uniquement

---

## 8. Références

- `docs/design/design-tokens.md` — Tokens et préférences de thème
- `docs/design/theme-strategy.md` — Stratégie de thème
- `docs/state/presentation-state.md` — État de présentation
- `docs/state/unknown-data.md` — Protocole `UNKNOWN != ZERO`
- `docs/architecture/architecture-overview.md` — Architecture et persistence
- `docs/ui/layout-modes.md` — Modes de layout comme préférence
- `docs/design/density.md` — Densité comme préférence
- Prompt `08-interaction-personalization.md` — Source du mandat Phase 08
