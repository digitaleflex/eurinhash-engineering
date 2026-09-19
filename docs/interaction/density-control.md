# Density Control — Ajustement de la Densité

> Mécanisme de contrôle de la densité du terminal HashCode.
> La densité ajuste la quantité d'information par ligne et l'espacement vertical.
> **Règle fondamentale** : `UNKNOWN != ZERO` · `Layout changes must not alter domain behavior`

---

## 1. Vue d'ensemble

La densité est un paramètre de présentation qui contrôle :
- Le nombre de lignes vides entre les sections
- Le padding horizontal
- La longueur de ligne
- La typographie utilisée
- Le niveau de détail des composants

La densité **ne modifie jamais** la logique métier. Elle change uniquement la représentation visuelle des données.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CONTRÔLE DE DENSITÉ                               │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  Commande        │  /density MINIMAL|STANDARD|...            │  │
│  │  Raccourci       │  Ctrl+D → Sélecteur de densité           │  │
│  │  Config          │  "density": "STANDARD" dans opencode.jsonc │  │
│  │  Auto            │  Détection par largeur de terminal        │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                        │
│  Niveaux : MINIMAL → STANDARD → DEVELOPER → OBSERVER → DEBUG       │
│  Par défaut : STANDARD                                               │
│  Règle : UNKNOWN != ZERO                                             │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Les 5 Niveaux de Densité

### 2.1 Tableau Comparatif

| Aspect | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|--------|---------|----------|-----------|----------|-------|
| Densité | Très sparse | Équilibré | Dense | Dense+ | Ultra-dense |
| Lignes vides | 2 | 1 | 0 | 0 | 0 |
| Padding | `space-lg` (8) | `space-md` (4) | `space-sm` (2) | `space-xs` (1) | 0 |
| Longueur ligne | 120+ | 100 | 80 | 160 | Illimité |
| Typographie | `display`/`body` | `heading`/`body` | `body`/`mono` | `mono` | `mono` |
| Couleurs | Toutes | Toutes | Réduites | Réduites | Monochrome |
| Bordures | `border` | `border` | `borderSubtle` | `borderSubtle` | Aucune |
| Usage | Monitoring | Quotidien | Débogage | Surveillance | Diagnostic |
| Par défaut | Non | **OUI** | Non | Non | Non |

### 2.2 Exemples Visuels par Niveau

#### MINIMAL (120+ cols, 2 lignes vides)
```
[● ACTIVE]  eurinhash-agent  ·  38%

[◐ THINKING]  worker-novita  ·  2.3s

[✓ SUCCESS]  build-completed  ·  2.3s
```

#### STANDARD (100 cols, 1 ligne vide)
```
[● ACTIVE]  eurinhash-agent  ·  38%
[◐ THINKING]  worker-novita  ·  2.3s
[✓ SUCCESS]  build-completed  ·  2.3s
```

#### DEVELOPER (80 cols, 0 ligne vide)
```
[● ACTIVE] eurinhash-agent · 38% · [◐ THINKING] worker-novita · 2.3s · [✓ SUCCESS] build-completed · 2.3s
```

#### OBSERVER (160 cols, 0 ligne vide)
```
[● ACTIVE] eurinhash-agent · 38% · [◐ THINKING] worker-novita · 2.3s · [✓ SUCCESS] build-completed · 2.3s · [✗ ERROR] worker-google · Connection refused · N/A
```

#### DEBUG (Illimité, 0 padding)
```
2026-09-19T12:00:00.000Z [● ACTIVE] eurinhash-agent pid=12345 mem=45MB cpu=12% · 38% · 100K/262K · $0.00 · worker-novita pid=67890 status=THINKING latency=2300ms
```

---

## 3. Mécanismes de Changement de Densité

### 3.1 Via Commande

```
/density MINIMAL      → Passer en mode MINIMAL
/density STANDARD     → Passer en mode STANDARD
/density DEVELOPER    → Passer en mode DEVELOPER
/density OBSERVER     → Passer en mode OBSERVER
/density DEBUG        → Passer en mode DEBUG
/density              → Afficher le niveau actuel
```

**Comportement** :
1. La commande est traitée dans la zone COMMAND
2. Le terminal se reconfigure visuellement
3. Aucune donnée de domaine n'est modifiée
4. Un toast confirme le changement : `[✓ SUCCESS] Density changed to STANDARD`
5. Si la largeur du terminal est insuffisante, un toast d'avertissement est affiché

### 3.2 Via Raccourci Clavier

| Raccourci | Action |
|-----------|--------|
| `Ctrl+D` | Ouvrir le sélecteur de densité |
| `Ctrl+Alt+1` | Passer en MINIMAL |
| `Ctrl+Alt+2` | Passer en STANDARD |
| `Ctrl+Alt+3` | Passer en DEVELOPER |
| `Ctrl+Alt+4` | Passer en OBSERVER |
| `Ctrl+Alt+5` | Passer en DEBUG |

**Règles** :
1. `Ctrl+D` ouvre un `Dialog` de type `Select` avec les 5 niveaux
2. `Ctrl+Alt+1..5` changent directement le niveau
3. Le changement est immédiat et visuel
4. `Escape` ferme le sélecteur sans changer le niveau

### 3.3 Via Configuration

Le niveau de densité est configurable dans `opencode.jsonc` :

```jsonc
{
    "density": "STANDARD"
}
```

**Valeurs valides** : `MINIMAL` | `STANDARD` | `DEVELOPER` | `OBSERVER` | `DEBUG`

**Règles de configuration** :
1. `STANDARD` est le **défaut** si non spécifié
2. La configuration est lue au démarrage
3. Les changements en runtime sont persistés via la persistence existante
4. `UNKNOWN` dans la configuration → échec sûr → `STANDARD` par défaut

### 3.4 Détection Automatique

Le système peut détecter automatiquement le niveau recommandé selon la largeur du terminal :

| Largeur du terminal | Niveau recommandé |
|---------------------|-------------------|
| < 80 | `MINIMAL` |
| 80–99 | `DEVELOPER` |
| 100–119 | `STANDARD` |
| 120–159 | `OBSERVER` |
| ≥ 160 | `STANDARD` (ou `OBSERVER` si préféré) |

**Règles** :
1. La détection automatique utilise la largeur du terminal
2. L'utilisateur peut outrider la détection automatique
3. La détection ne modifie pas la configuration de l'utilisateur
4. `UNKNOWN` dans la largeur du terminal → `STANDARD` par défaut
5. La détection automatique est une suggestion, pas une contrainte

---

## 4. Interaction Densité et Composants

### 4.1 Badges de Statut

| Niveau | Format |
|--------|--------|
| MINIMAL | `[● ACTIVE]` (plein, sur ligne dédiée) |
| STANDARD | `[● ACTIVE]` (inline) |
| DEVELOPER | `●` (glyphe seul, texte dans la même ligne) |
| OBSERVER | `●` (glyphe seul, tout compact) |
| DEBUG | `ACTIVE` (texte seul) |

**Règle `UNKNOWN != ZERO`** : `[? UNKNOWN]` est affiché à tout niveau — jamais `[○ IDLE]` ou `0`.

### 4.2 Barres de Progression

| Niveau | Format |
|--------|--------|
| MINIMAL | `[████░░░░] 38%` (sur ligne dédiée) |
| STANDARD | `[████░░░░] 38%` (inline) |
| DEVELOPER | `38%` (texte seul) |
| OBSERVER | `38% · 100K/262K` (texte seul) |
| DEBUG | `0.38` (ratio brut) |

### 4.3 Tables

| Niveau | Format |
|--------|--------|
| MINIMAL | Table complète avec bordures, headers |
| STANDARD | Table compacte, headers réduits |
| DEVELOPER | Liste clé-valeur |
| OBSERVER | Données tabulaires serrées |
| DEBUG | CSV / JSON lines |

### 4.4 Panneaux

| Niveau | Format |
|--------|--------|
| MINIMAL | Panneau boité avec corners et padding |
| STANDARD | Panneau avec borders, padding standard |
| DEVELOPER | Panneau avec borderSubtle, padding réduit |
| OBSERVER | Ligne de séparation seulement |
| DEBUG | Pas de panneau — données brutes |

### 4.5 Alerts

| Niveau | Format |
|--------|--------|
| MINIMAL | Boîte complète avec borders |
| STANDARD | Bordure + texte |
| DEVELOPER | Texte préfixé par le glyphe |
| OBSERVER | Texte préfixé par le glyphe |
| DEBUG | Non applicable |

### 4.6 Sidebar

| Niveau | Affichage |
|--------|-----------|
| MINIMAL | Pleine largeur, tous les items |
| STANDARD | Labels tronqués |
| DEVELOPER | Icones uniquement, hover pour labels |
| OBSERVER | Icones uniquement |
| DEBUG | Non applicable |

---

## 5. Règles de Non-Fabrication par Densité

### 5.1 Règle de l'information minimale par ligne

- Chaque ligne DOIT contenir au minimum : statut + nom + métrique principale
- Ne jamais avoir une ligne avec uniquement une couleur sans contenu textuel
- `thinkingOpacity` s'applique à tous les niveaux de densité

### 5.2 Règle de la largeur de colonne

| Largeur | Niveau | Usage |
|---------|--------|-------|
| 80 | Développeur minimum | Terminal classique |
| 100 | Standard recommandé | Usage quotidien |
| 120 | Large recommandé | Monitoring |
| 160 | Ultra-large | Observer, métriques |

### 5.3 Règles de Non-Fabrication

| Règle | Application |
|-------|------------|
| `UNKNOWN` | Ne JAMAIS changer en `0` ou `N/A` si la largeur change |
| `N/A` | Reste `N/A` quelle que soit la largeur |
| `$0.00` | Reste `$0.00` si le coût est connu et nul |
| `100K · 38%` | Quand la limite est inconnue, afficher ainsi — jamais compléter |
| `38%` | Pourcentage simple, valide à toute largeur |
| `thinkingOpacity` | Appliqué uniquement aux éléments THINKING à tout niveau |
| `UNKNOWN != ZERO` | `undefined` ≠ `0`, `""` ≠ absent |

**Exemples** :
- `cost: undefined` → affiche `N/A` (JAMAIS `$0.00`)
- `worker.status: undefined` → affiche `[? UNKNOWN]` (JAMAIS `[○ IDLE]`)
- `branch: null` → affiche `null` (JAMAIS `""`)

---

## 6. Accessibilité par Niveau de Densité

### 6.1 Accessibilité

| Niveau | Accessibilité | Notes |
|--------|---------------|-------|
| `MINIMAL` | ✅ Haute | Texte grand, espaces larges, facile à lire |
| `STANDARD` | ✅ Haute | Équilibré, par défaut |
| `DEVELOPER` | ⚠️ Moyenne | Dense, mais toujours lisible |
| `OBSERVER` | ⚠️ Moyenne | Très dense, nécessite 120+ cols |
| `DEBUG` | ❌ Basse | Données brutes, pas adapté à la lecture |

### 6.2 Mode Sans Couleur

Pour les utilisateurs malvoyants ou les terminaux monochromes :

| Élément | MINIMAL/STANDARD | DEVELOPER/OBSERVER | DEBUG |
|---------|-----------------|--------------------|-------|
| `borderSubtle` | `dim` attribute | `dim` attribute | Aucune bordure |
| `borderActive` | `bold + underline` | `bold + underline` | Aucune |
| `error` | `bold` | `bold` | `bold` |
| `success` | `underline` | `underline` | `underline` |
| `warning` | `bold + dim` | `bold + dim` | `bold` |
| `info` | `reverse-video` | `reverse-video` | `reverse-video` |
| `thinkingOpacity` | `dim` attribute | `dim` attribute | `dim` attribute |

### 6.3 Règles d'Accessibilité par Densité

1. **JAMAIS** réduire l'accessibilité quand on passe à un mode plus dense
2. **TOUJOURS** maintenir le texte des statuts lisible à 80 colonnes
3. **JAMAIS** de données fabriquées pour remplir l'espace
4. **TOUJOURS** `UNKNOWN` reste lisible à toute largeur et densité
5. **JAMAIS** cacher une information critique pour des raisons de place
6. `thinkingOpacity` ne réduit PAS l'accessibilité — le texte reste lisible

---

## 7. Interaction entre Densité et Thème

La densité est **indépendante** du thème :

1. Le thème (dark/light) change les couleurs
2. La densité change le layout et l'espacement
3. Les deux sont configurables séparément
4. `thinkingOpacity` est un `number` — indépendant du thème et de la densité
5. `UNKNOWN` et `N/A` restent affichés de la même manière dans tous les combinaisons thème/densité

---

## 8. Règles Fondamentales du Contrôle de Densité

1. **`STANDARD` est le défaut** pour tous les utilisateurs
2. `DEVELOPER` et `OBSERVER` ne DOIVENT être accessibles qu'aux utilisateurs avertis
3. `DEBUG` NE DOIT PAS être le niveau par défaut — réservé au diagnostic
4. `thinkingOpacity` est disponible à TOUS les niveaux
5. La largeur du terminal DOIT être vérifée AVANT de choisir le niveau
6. `UNKNOWN` et `N/A` sont affichés de manière identique à TOUS les niveaux de densité
7. JAMAIS de données fabriquées pour remplir l'espace dans un niveau de densité
8. Le niveau de densité est SOUVENT configuré par l'utilisateur, jamais imposé
9. **Layout changes must not alter domain behavior**
10. **`UNKNOWN != ZERO`** à tout niveau de densité
11. La densité est indépendante du thème
12. Les transitions entre niveaux DOIVENT être progressives

---

## 9. Références

- `docs/design/density.md` — Niveaux de densité détaillés
- `docs/ui/layout-modes.md` — Implementation des modes
- `docs/ux/navigation-model.md` — Navigation par niveau de densité
- `docs/design/components.md` — Composants adaptés par niveau
- `docs/design/theme-strategy.md` — Thème indépendant du niveau
- `docs/state/presentation-state.md` — `TuiState` et la densité
- `docs/state/unknown-data.md` — Protocole `UNKNOWN != ZERO`
- Prompt `08-interaction-personalization.md` — Source du mandat Phase 08
