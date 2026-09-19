# Layout Modes — Modes de Layout MINIMAL, STANDARD, DEVELOPER, OBSERVER, DEBUG

> Les 5 modes de layout du Command Center EurinHash OpenCode.
> Layout changes must not alter domain behavior.
> Règle fondamentale : `UNKNOWN != ZERO`

---

## 1. Vue d'Ensemble

### 1.1 Les 5 Modes

| Mode | Densité | Lignes vides | Padding | Longueur ligne | Usage |
|------|---------|-------------|---------|----------------|-------|
| `MINIMAL` | Très sparse | 2 lignes entre sections | `space-lg` (8 cols) | 120+ cols | Sessions longues, monitoring continu |
| `STANDARD` | Équilibré | 1 ligne entre sections | `space-md` (4 cols) | 100 cols | Usage quotidien, développement |
| `DEVELOPER` | Dense | 0 ligne entre sections | `space-sm` (2 cols) | 80 cols | Débogage, inspection rapide |
| `OBSERVER` | Dense+ | 0 ligne entre sections | `space-xs` (1 col) | 160 cols | Surveillance, logs continus |
| `DEBUG` | Ultra-dense | 0 ligne, données brutes | 0 | Illimitée (wrap auto) | Diagnostic, traces complètes |

### 1.2 Règles Fondamentales

1. **Layout changes must not alter domain behavior** — le mode ne modifie jamais la logique métier
2. Le niveau par défaut est `STANDARD`
3. La transition entre niveaux DOIT être progressive
4. `UNKNOWN` reste `UNKNOWN` à tout niveau de densité
5. `N/A` reste `N/A` à tout niveau de densité
6. Les métriques sont TOUJOURS en `mono`
7. `thinkingOpacity` est disponible à TOUS les niveaux
8. `UNKNOWN != ZERO` — jamais de données fabriquées

---

## 2. Description Détaillée par Mode

### 2.1 MINIMAL

**Densité** : Très sparse
**Lignes vides** : 2 entre chaque bloc
**Padding** : `space-lg` (8 cols)
**Typographie** : `display` pour les titres, `body` pour le contenu
**Longueur de ligne** : 120+ colonnes
**Usage** : Sessions longues, monitoring continu, revue de projets

```
[● ACTIVE]  eurinhash-agent  ·  38%

[◐ THINKING]  worker-novita  ·  2.3s

[✓ SUCCESS]  build-completed  ·  2.3s
```

**Caractéristiques** :
- Chaque section a des lignes vides pour la respiration visuelle
- Les titres utilisent `display` (bold + primary + 2× spacing avant)
- Les bordures sont `border` complètes avec coins `┌┐└┘`
- Les métriques sont toujours affichées en entier
- Les tables sont complètes avec tous les headers

**Composants adaptés** :

| Composant | Format MINIMAL |
|-----------|---------------|
| Panel | Panneau boité avec corners et padding |
| Table | Table complète avec bordures, headers |
| ProgressBar | `[████░░░░] 38%` sur ligne dédiée |
| Badge | `[● ACTIVE]` plein, sur ligne dédiée |
| StatusBar | Plein avec tous les items |
| Alert | Boîte complète avec borders |
| Dialog | Taille selon contenu, max 80 cols |
| Sidebar | Pleine largeur, tous les items visibles |

---

### 2.2 STANDARD

**Densité** : Équilibré
**Lignes vides** : 1 entre chaque bloc
**Padding** : `space-md` (4 cols)
**Typographie** : `heading` pour les titres, `body` pour le contenu
**Longueur de ligne** : 100 colonnes
**Usage** : Usage quotidien, développement, sessions normales

```
[● ACTIVE]  eurinhash-agent  ·  38%
[◐ THINKING]  worker-novita  ·  2.3s
[✓ SUCCESS]  build-completed  ·  2.3s
```

**Caractéristiques** :
- Une ligne vide entre les blocs
- Les titres utilisent `heading` (bold + text)
- Les bordures sont `border` standard
- Les métriques sont affichées en ligne
- C'est le **mode par défaut**

**Composants adaptés** :

| Composant | Format STANDARD |
|-----------|----------------|
| Panel | Panneau avec borders, padding standard |
| Table | Table compacte, headers réduits |
| ProgressBar | `[████░░░░] 38%` inline |
| Badge | `[● ACTIVE]` inline |
| StatusBar | Compacte |
| Alert | Bordure + texte |
| Dialog | Taille selon contenu, max 70 cols |
| Sidebar | Largeur réduite, labels tronqués |

---

### 2.3 DEVELOPER

**Densité** : Dense
**Lignes vides** : 0 entre les éléments
**Padding** : `space-sm` (2 cols)
**Typographie** : `body` pour tout, `mono` pour les données
**Longueur de ligne** : 80 colonnes
**Usage** : Débogage, inspection rapide, revue de code

```
[● ACTIVE] eurinhash-agent · 38% · [◐ THINKING] worker-novita · 2.3s · [✓ SUCCESS] build-completed · 2.3s
```

**Caractéristiques** :
- Pas de lignes vides entre les éléments
- Padding réduit à 2 colonnes
- Tout est en `body` sauf les données en `mono`
- Les badges sont réduits (glyphe seul dans la même ligne)
- Les bordures sont `borderSubtle`
- Fonctionne sur 80 colonnes minimum

**Composants adaptés** :

| Composant | Format DEVELOPER |
|-----------|-----------------|
| Panel | Panneau avec borderSubtle, padding réduit |
| Table | Liste clé-valeur |
| ProgressBar | `38%` (texte seul) |
| Badge | `●` (glyphe seul, texte dans la même ligne) |
| StatusBar | Compacte extrême |
| Alert | Texte préfixé par le glyphe |
| Dialog | Plein largeur, padding minimal |
| Sidebar | Icones uniquement, hover pour labels |

---

### 2.4 OBSERVER

**Densité** : Dense+
**Lignes vides** : 0 entre les éléments
**Padding** : `space-xs` (1 col)
**Typographie** : `mono` pour tout le contenu
**Longueur de ligne** : 160 colonnes
**Usage** : Surveillance de logs, flux continus, métriques temps réel

```
[● ACTIVE] eurinhash-agent · 38% · [◐ THINKING] worker-novita · 2.3s · [✓ SUCCESS] build-completed · 2.3s · [✗ ERROR] worker-google · Connection refused · N/A
```

**Caractéristiques** :
- 0 ligne vide entre les éléments
- Padding d'1 colonne seulement
- Tout en `mono`
- Toutes les métriques sont affichées
- Les bordures sont `borderSubtle` ou absentes
- Nécessite minimum 120 colonnes

**Composants adaptés** :

| Composant | Format OBSERVER |
|-----------|----------------|
| Panel | Ligne de séparation seulement |
| Table | Données tabulaires serrées |
| ProgressBar | `38% · 100K/262K` (texte seul) |
| Badge | `●` (glyphe seul, tout compact) |
| StatusBar | Ultra-compacte |
| Alert | Texte préfixé par le glyphe |
| Dialog | Taille contenu |
| Sidebar | Standard |

---

### 2.5 DEBUG

**Densité** : Ultra-dense
**Lignes vides** : 0
**Padding** : 0
**Typographie** : `mono` uniquement, données brutes
**Longueur de ligne** : Illimitée (wrap automatique)
**Usage** : Diagnostic, traces complètes, logs bruts

```
2026-09-19T12:00:00.000Z [● ACTIVE] eurinhash-agent pid=12345 mem=45MB cpu=12% · 38% · 100K/262K · $0.00 · worker-novita pid=67890 status=THINKING latency=2300ms · worker-google pid=11111 status=ERROR err=Connection refused
```

**Caractéristiques** :
- 0 ligne vide, 0 padding
- Données brutes en `mono`
- Pas de bordures
- Pas de panneau — données brutes
- Nécessite 160+ colonnes ou wrap automatique
- NE DOIT PAS être le niveau par défaut — réservé au diagnostic

**Composants adaptés** :

| Composant | Format DEBUG |
|-----------|-------------|
| Panel | Pas de panneau — données brutes |
| Table | CSV / JSON lines |
| ProgressBar | `0.38` (ratio brut) |
| Badge | `ACTIVE` (texte seul) |
| StatusBar | Non applicable |
| Alert | Non applicable |
| Dialog | Non applicable |
| Sidebar | Non applicable |

---

## 3. Tableau Comparatif Complet

### 3.1 Densité et Spacing

| Aspect | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|--------|---------|----------|-----------|----------|-------|
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

### 3.2 Composants par Mode

#### Badges de Statut

| Niveau | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|--------|---------|----------|-----------|----------|-------|
| ACTIVE | `[● ACTIVE]` | `[● ACTIVE]` | `●` | `●` | `ACTIVE` |
| THINKING | `[◐ THINKING]` | `[◐ THINKING]` | `◐` | `◐` | `THINKING` |
| ERROR | `[✗ ERROR]` | `[✗ ERROR]` | `✗` | `✗` | `ERROR` |
| UNKNOWN | `[? UNKNOWN]` | `[? UNKNOWN]` | `?` | `?` | `UNKNOWN` |

#### Barres de Progression

| Niveau | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|--------|---------|----------|-----------|----------|-------|
| Format | `[████░░░░] 38%` (ligne dédiée) | `[████░░░░] 38%` (inline) | `38%` (texte seul) | `38% · 100K/262K` (texte seul) | `0.38` (ratio brut) |

#### Tables

| Niveau | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|--------|---------|----------|-----------|----------|-------|
| Format | Table complète avec bordures | Table compacte, headers réduits | Liste clé-valeur | Données tabulaires serrées | CSV / JSON lines |

#### Panneaux

| Niveau | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|--------|---------|----------|-----------|----------|-------|
| Format | Panneau boité avec corners | Panneau avec borders, padding standard | Panneau avec borderSubtle, padding réduit | Ligne de séparation seulement | Pas de panneau — données brutes |

#### Alerts

| Niveau | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|--------|---------|----------|-----------|----------|-------|
| Format | Boîte complète avec borders | Bordure + texte | Texte préfixé par le glyphe | Texte préfixé par le glyphe | Non applicable |

#### Dialogs

| Niveau | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|--------|---------|----------|-----------|----------|-------|
| Format | Taille selon contenu, max 80 cols | Taille selon contenu, max 70 cols | Plein largeur, padding minimal | Taille contenu | Non applicable |

---

## 4. Transition entre Modes

### 4.1 Règles de Transition

1. La transition DOIT être progressive — pas de saut brutal de MINIMAL à DEBUG
2. Le niveau par défaut est `STANDARD`
3. L'utilisateur peut changer le niveau via une commande ou config
4. La détection automatique est disponible

### 4.2 Détection Automatique

| Largeur du terminal | Niveau recommandé |
|---------------------|-------------------|
| < 80 | `MINIMAL` |
| 80–99 | `DEVELOPER` |
| 100–119 | `STANDARD` |
| 120–159 | `OBSERVER` |
| ≥ 160 | `STANDARD` (ou `OBSERVER` si préféré) |

### 4.3 Commande de Changement de Mode

| Commande | Action |
|----------|--------|
| `/density MINIMAL` | Passer en mode MINIMAL |
| `/density STANDARD` | Passer en mode STANDARD |
| `/density DEVELOPER` | Passer en mode DEVELOPER |
| `/density OBSERVER` | Passer en mode OBSERVER |
| `/density DEBUG` | Passer en mode DEBUG |
| `/density` | Afficher le niveau actuel |

### 4.4 Raccourci Clavier

| Raccourci | Action |
|-----------|--------|
| `Ctrl+Alt+D` | Ouvrir le sélecteur de densité |
| `Ctrl+Alt+1` | MINIMAL |
| `Ctrl+Alt+2` | STANDARD |
| `Ctrl+Alt+3` | DEVELOPER |
| `Ctrl+Alt+4` | OBSERVER |
| `Ctrl+Alt+5` | DEBUG |

---

## 5. Configuration du Mode

### 5.1 Via Configuration

Le niveau de densité est configurable dans `opencode.jsonc` :

```jsonc
{
    "density": "STANDARD"  // MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG
}
```

### 5.2 Via TuiState

Le thème et la densité sont accessibles via `TuiState` :

```typescript
// Lire la densité actuelle
const density = api.tuiConfig.density; // "STANDARD"

// Changer la densité
api.ui.toast({ message: `Density changed to ${newDensity}` });
```

### 5.3 Règles de Configuration

1. `STANDARD` est le **défaut** pour tous les utilisateurs
2. `DEVELOPER` et `OBSERVER` ne DOIVENT être accessibles qu'aux utilisateurs avertis
3. `DEBUG` NE DOIT PAS être le niveau par défaut — réservé au diagnostic
4. `thinkingOpacity` est disponible à TOUS les niveaux
5. La largeur du terminal DOIT être vérifiée AVANT de choisir le niveau
6. `UNKNOWN` et `N/A` sont affichés de manière identique à TOUS les niveaux de densité
7. JAMAIS de données fabriquées pour remplir l'espace dans un niveau de densité
8. Le niveau de densité est SOUVENT configuré par l'utilisateur, jamais imposé

---

## 6. Interaction entre Densité et Composants

### 6.1 Règles par Composant

| Composant | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|-----------|---------|----------|-----------|----------|-------|
| StatusIndicator | `[● ACTIVE]` sur ligne dédiée | `[● ACTIVE]` inline | `●` inline | `●` compact | `ACTIVE` texte seul |
| ProgressBar | `[████░░░░] 38%` ligne dédiée | `[████░░░░] 38%` inline | `38%` texte seul | `38% · 100K/262K` texte seul | `0.38` ratio brut |
| Table | Table complète | Table compacte | Liste clé-valeur | Table serrée | CSV / JSON lines |
| Panel | Coins + padding | Borders + padding | BorderSubtle + padding réduit | Ligne de séparation | Pas de panneau |
| Alert | Boîte complète | Bordure + texte | Texte préfixé | Texte préfixé | Non applicable |
| Sidebar | Tous les items | Labels tronqués | Icones uniquement | Icones uniquement | Non applicable |
| Dialog | Max 80 cols | Max 70 cols | Plein largeur | Taille contenu | Non applicable |
| Badge | `[● ACTIVE]` | `[● ACTIVE]` | `●` | `●` | `ACTIVE` |

### 6.2 Règles de Non-Fabrication par Mode

| Règle | Application |
|-------|------------|
| `UNKNOWN` | Ne JAMAIS changer en `0` ou `N/A` si la largeur change |
| `N/A` | Reste `N/A` quelle que soit la largeur |
| `$0.00` | Reste `$0.00` si le coût est connu et nul |
| `100K · 38%` | Quand la limite est inconnue, afficher ainsi — jamais compléter |
| `38%` | Pourcentage simple, valide à toute largeur |

---

## 7. Accessibilité par Mode

### 7.1 Mode Sans Couleur

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

### 7.2 Règles d'Accessibilité

1. Le texte est TOUJOURS lisible sans couleur
2. Les statuts sont TOUJOURS identifiables par le glyphe + texte
3. `UNKNOWN` reste `?` dans tous les modes
4. `N/A` reste `N/A` dans tous les modes
5. Le mode sans couleur est activable via configuration
6. La densité est indépendante du thème (dark/light)

---

## 8. Règles Fondamentales des Modes

1. Le niveau `STANDARD` est le **défaut** pour tous les utilisateurs
2. `DEVELOPER` et `OBSERVER` ne DOIVENT être accessibles qu'aux utilisateurs avertis
3. `DEBUG` NE DOIT PAS être le niveau par défaut — il est réservé au diagnostic
4. `thinkingOpacity` est disponible à TOUS les niveaux
5. La largeur du terminal DOIT être vérifiée AVANT de choisir le niveau
6. `UNKNOWN` et `N/A` sont affichés de manière identique à TOUS les niveaux de densité
7. JAMAIS de données fabriquées pour remplir l'espace dans un niveau de densité
8. Le niveau de densité est SOUVENT configuré par l'utilisateur, jamais imposé
9. **Layout changes must not alter domain behavior** — le mode ne modifie jamais la logique métier
10. `UNKNOWN != ZERO` à tout niveau de densité

---

## 9. Résumé

| Mode | Densité | Padding | Longueur | Usage | Par défaut |
|------|---------|---------|----------|-------|------------|
| `MINIMAL` | Très sparse | `space-lg` (8) | 120+ | Monitoring | Non |
| `STANDARD` | Équilibré | `space-md` (4) | 100 | Quotidien | **OUI** |
| `DEVELOPER` | Dense | `space-sm` (2) | 80 | Débogage | Non |
| `OBSERVER` | Dense+ | `space-xs` (1) | 160 | Surveillance | Non |
| `DEBUG` | Ultra-dense | 0 | Illimité | Diagnostic | Non |

**Règle fondamentale** : `Layout changes must not alter domain behavior`