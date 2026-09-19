# Density — Niveaux de Densité Terminale

> Trois niveaux de densité pour le terminal EurinHash OpenCode.
> La densité contrôle la quantité d'information par ligne et l'espacement vertical.

---

## 1. Vue d'Ensemble

| Niveau | Densité | Lignes vides | Padding | Usage recommandé |
|---|---|---|---|---|
| `MINIMAL` | Très sparse | 2 lignes entre sections | `space-lg` (8 cols) | Sessions longues, monitoring continu |
| `STANDARD` | Équilibré | 1 ligne entre sections | `space-md` (4 cols) | Usage quotidien, développement |
| `DEVELOPER` | Dense | 0 ligne entre sections | `space-sm` (2 cols) | Débogage, inspection rapide |
| `OBSERVER` | Dense+ | 0 ligne entre sections | `space-xs` (1 col) | Surveillance, logs continus |
| `DEBUG` | Ultra-dense | 0 ligne, données brutes | 0 | Diagnostic, traces complètes |

---

## 2. Description par Niveau

### 2.1 MINIMAL

- **Densité** : Très sparse
- **Lignes vides** : 2 entre chaque bloc
- **Padding** : `space-lg` (8 cols)
- **Typographie** : `display` pour les titres, `body` pour le contenu
- **Longueur de ligne** : 120+ colonnes
- **Usage** : Sessions longues, monitoring continu, revue de projets

```
[● ACTIVE]  eurinhash-agent  ·  38%

[◐ THINKING]  worker-novita  ·  2.3s

[✓ SUCCESS]  build-completed  ·  2.3s
```

### 2.2 STANDARD

- **Densité** : Équilibré
- **Lignes vides** : 1 entre chaque bloc
- **Padding** : `space-md` (4 cols)
- **Typographie** : `heading` pour les titres, `body` pour le contenu
- **Longueur de ligne** : 100 colonnes
- **Usage** : Usage quotidien, développement, sessions normales

```
[● ACTIVE]  eurinhash-agent  ·  38%
[◐ THINKING]  worker-novita  ·  2.3s
[✓ SUCCESS]  build-completed  ·  2.3s
```

### 2.3 DEVELOPER

- **Densité** : Dense
- **Lignes vides** : 0 entre les éléments
- **Padding** : `space-sm` (2 cols)
- **Typographie** : `body` pour tout, `mono` pour les données
- **Longueur de ligne** : 80 colonnes
- **Usage** : Débogage, inspection rapide, revue de code

```
[● ACTIVE] eurinhash-agent · 38% · [◐ THINKING] worker-novita · 2.3s · [✓ SUCCESS] build-completed · 2.3s
```

### 2.4 OBSERVER

- **Densité** : Dense+
- **Lignes vides** : 0 entre les éléments
- **Padding** : `space-xs` (1 col)
- **Typographie** : `mono` pour tout le contenu
- **Longueur de ligne** : 160 colonnes
- **Usage** : Surveillance de logs, flux continus, métriques temps réel

```
[● ACTIVE] eurinhash-agent · 38% · [◐ THINKING] worker-novita · 2.3s · [✓ SUCCESS] build-completed · 2.3s · [✗ ERROR] worker-google · Connection refused · N/A
```

### 2.5 DEBUG

- **Densité** : Ultra-dense
- **Lignes vides** : 0
- **Padding** : 0
- **Typographie** : `mono` uniquement, données brutes
- **Longueur de ligne** : Illimitée (wrap automatique)
- **Usage** : Diagnostic, traces complètes, logs bruts

```
2026-09-19T12:00:00.000Z [● ACTIVE] eurinhash-agent pid=12345 mem=45MB cpu=12% · 38% · 100K/262K · $0.00 · worker-novita pid=67890 status=THINKING latency=2300ms · worker-google pid=11111 status=ERROR err=Connection refused
```

---

## 3. Tableau Comparatif

| Aspect | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|---|---|---|---|---|---|
| Lignes vides | 2 | 1 | 0 | 0 | 0 |
| Padding horizontal | 8 cols | 4 cols | 2 cols | 1 col | 0 |
| Titres | `display` | `heading` | — | — | — |
| Contenu | `body` | `body` | `body` | `mono` | `mono` |
| Largeur ligne | 120+ | 100 | 80 | 160 | Illimité |
| Métriques | `mono` | `mono` | `mono` | `mono` | `mono` |
| Glyphes statut | Plein | Plein | Réduit | Réduit | Texte |
| Couleurs | Toutes | Toutes | Réduites | Réduites | Monochrome |
| Timestamps | `caption` | `caption` | `mono` | `mono` | Complet |
| Borders | `border` | `border` | `borderSubtle` | `borderSubtle` | Aucune |
| Segregations | `rule-strong` | `rule` | `rule-thin` | Aucune | Aucune |

---

## 4. Règles de Densité

### 4.1 Règle de l'information minimale par ligne

- Chaque ligne DOIT contenir au minimum : statut + nom + métrique principale
- Ne jamais avoir une ligne avec uniquement une couleur sans contenu textuel
- `thinkingOpacity` s'applique à tous les niveaux de densité

### 4.2 Règle de la largeur de colonne

| Largeur | Niveau | Usage |
|---|---|---|
| 80 | Développeur minimum | Terminal classique |
| 100 | Standard recommandé | Usage quotidien |
| 120 | Large recommandé | Monitoring |
| 160 | Ultra-large | Observer, métriques |

### 4.3 Règle de la transition

- La transition entre niveaux DOIT être progressive
- Pas de saut brutal de MINIMAL à DEBUG
- Le niveau par défaut est `STANDARD`
- L'utilisateur peut changer le niveau via une commande ou config

### 4.4 Règle de la compatibilité

- `MINIMAL` est compatible avec TOUS les terminaux (y compris 80 cols)
- `OBSERVER` nécessite minimum 120 colonnes
- `DEBUG` nécessite 160+ colonnes ou wrap automatique
- `DEVELOPER` fonctionne sur 80 colonnes minimum

---

## 5. Configuration du Niveau

Le niveau de densité est configurable via :

- **Commande** : `/density MINIMAL|STANDARD|DEVELOPER|OBSERVER|DEBUG`
- **Config** : `density` dans la configuration OpenCode
- **Raccourci** : `Ctrl+Alt+D` puis sélection du niveau
- **Auto** : Détection automatique de la largeur du terminal

### 5.1 Détection Automatique

| Largeur terminal | Niveau recommandé |
|---|---|
| < 80 | `MINIMAL` |
| 80–99 | `DEVELOPER` |
| 100–119 | `STANDARD` |
| 120–159 | `OBSERVER` |
| ≥ 160 | `STANDARD` (ou `OBSERVER` si préféré) |

### 5.2 Règle de Non-Fabrication

- Ne jamais afficher des données qui ne sont pas disponibles dans le niveau de densité courant
- `UNKNOWN` reste `UNKNOWN` à tout niveau de densité
- `N/A` reste `N/A` à tout niveau de densité
- Les métriques partielles (ex: `100K · 38%` quand la limite est inconnue) sont affichées telles quelles — jamais complétées artificiellement

---

## 6. Interaction entre Densité et Composants

### 6.1 Barre de Progression

| Niveau | Format |
|---|---|
| MINIMAL | `[████░░░░] 38%` (sur ligne dédiée) |
| STANDARD | `[████░░░░] 38%` (inline) |
| DEVELOPER | `38%` (texte seul) |
| OBSERVER | `38% · 100K/262K` (texte seul) |
| DEBUG | `0.38` (ratio brut) |

### 6.2 Badges de Statut

| Niveau | Format |
|---|---|
| MINIMAL | `[● ACTIVE]` (plein, sur ligne dédiée) |
| STANDARD | `[● ACTIVE]` (inline) |
| DEVELOPER | `●` (glyphe seul, texte dans la même ligne) |
| OBSERVER | `●` (glyphe seul, tout compact) |
| DEBUG | `ACTIVE` (texte seul) |

### 6.3 Tables

| Niveau | Format |
|---|---|
| MINIMAL | Table complète avec bordures, headers |
| STANDARD | Table compacte, headers réduits |
| DEVELOPER | Liste clé-valeur |
| OBSERVER | Données tabulaires serrées |
| DEBUG | CSV / JSON lines |

### 6.4 Panneaux

| Niveau | Format |
|---|---|
| MINIMAL | Panneau boité avec corners et padding |
| STANDARD | Panneau avec borders, padding standard |
| DEVELOPER | Panneau avec borderSubtle, padding réduit |
| OBSERVER | Ligne de séparation seulement |
| DEBUG | Pas de panneau — données brutes |

---

## 7. Règles de Densité

1. Le niveau `STANDARD` est le **défaut** pour tous les utilisateurs
2. `DEVELOPER` et `OBSERVER` ne DOIVENT être accessibles qu'aux utilisateurs avertis
3. `DEBUG` NE DOIT PAS être le niveau par défaut — il est réservé au diagnostic
4. `thinkingOpacity` est disponible à TOUS les niveaux
5. La largeur du terminal DOIT être vérifiée AVANT de choisir le niveau
6. `UNKNOWN` et `N/A` sont affichés de manière identique à TOUS les niveaux de densité
7. JAMAIS de données fabriquées pour remplir l'espace dans un niveau de densité
8. Le niveau de densité est SOUVENT configuré par l'utilisateur, jamais imposé
