# Layout Modes Configuration — Configuration des 5 Modes de Layout

> Les 5 modes de layout du HashCode Terminal : MINIMAL, STANDARD, DEVELOPER, OBSERVER, DEBUG.
> **Règle fondamentale** : `Layout changes must not alter domain behavior`
> **Règle fondamentale** : `UNKNOWN != ZERO`
> **Mode par défaut** : `STANDARD`

---

## 1. Vue d'ensemble

Le layout du terminal est organisé en **5 modes** qui contrôlent la densité, l'espacement, la visibilité des zones et le format des composants. Le changement de mode ne modifie **jamais** la logique métier.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    LES 5 MODES DE LAYOUT                                 │
│                                                                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │ MINIMAL  │  │ STANDARD │  │ DEVELOPER│  │ OBSERVER │  │  DEBUG   │ │
│  │          │  │          │  │          │  │          │  │          │ │
│  │ Very     │  │ Balanced │  │ Dense    │  │ Dense+   │  │ Ultra    │ │
│  │ Sparse   │  │ (default)│  │          │  │          │  │ Dense    │ │
│  │          │  │          │  │          │  │          │  │          │ │
│  │ 120+cols │  │ 100 cols │  │ 80 cols  │  │ 160 cols │  │ Unlimited │ │
│  │ Padding: │  │ Padding: │  │ Padding: │  │ Padding: │  │ Padding:  │ │
│  │ space-lg │  │ space-md │  │ space-sm │  │ space-xs │  │ 0         │ │
│  │ Empty:2  │  │ Empty:1  │  │ Empty:0  │  │ Empty:0  │  │ Empty:0   │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘ │
│                                                                            │
│  Règle : Layout changes MUST NOT alter domain behavior                    │
│  Règle : UNKNOWN != ZERO                                                  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Les 5 Modes — Description Détaillée

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
- Sidebar pleine largeur, tous les items visibles

**Zones accessibles** :
- CHAT/SESSION : Toujours visible, focus par défaut
- COMMAND : En bas, accessible
- WORKSPACE : Masquée sauf hamburger
- AGENT : Status bar réduit
- ACTIVITY : Non affiché

### 2.2 STANDARD (Défaut)

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
- C'est le **mode par défaut** pour tous les utilisateurs
- Sidebar complète avec labels

**Zones accessibles** :
- Toutes les zones sont accessibles
- `Tab` cycle WORKSPACE → CHAT → AGENT → ACTIVITY → COMMAND
- `Ctrl+W/C/A/E/I` fonctionnent pour toutes les zones

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
- Sidebar réduite, labels tronqués

**Zones accessibles** :
- `Tab` cycle CHAT → COMMAND → WORKSPACE → AGENT
- ACTIVITY est contextuel (apparaît quand un événement se produit)

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
- ACTIVITY est la zone principale (focus par défaut)

**Zones accessibles** :
- `Tab` cycle ACTIVITY → AGENT → CHAT → WORKSPACE → COMMAND
- ACTIVITY est la zone avec la plus grande surface

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
- **NE DOIT PAS** être le niveau par défaut — réservé au diagnostic

**Zones accessibles** :
- Toutes les zones affichent des données brutes
- `Tab` cycle dans l'ordre alphabétique
- Pas de masquage

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

### 3.2 Badges de Statut par Mode

| Niveau | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|--------|---------|----------|-----------|----------|-------|
| ACTIVE | `[● ACTIVE]` | `[● ACTIVE]` | `●` | `●` | `ACTIVE` |
| THINKING | `[◐ THINKING]` | `[◐ THINKING]` | `◐` | `◐` | `THINKING` |
| ERROR | `[✗ ERROR]` | `[✗ ERROR]` | `✗` | `✗` | `ERROR` |
| UNKNOWN | `[? UNKNOWN]` | `[? UNKNOWN]` | `?` | `?` | `UNKNOWN` |

### 3.3 Barres de Progression par Mode

| Niveau | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|--------|---------|----------|-----------|----------|-------|
| Format | `[████░░░░] 38%` (ligne dédiée) | `[████░░░░] 38%` (inline) | `38%` (texte seul) | `38% · 100K/262K` (texte seul) | `0.38` (ratio brut) |

### 3.4 Tables par Mode

| Niveau | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|--------|---------|----------|-----------|----------|-------|
| Format | Table complète avec bordures | Table compacte, headers réduits | Liste clé-valeur | Données tabulaires serrées | CSV / JSON lines |

### 3.5 Panneaux par Mode

| Niveau | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|--------|---------|----------|-----------|----------|-------|
| Format | Panneau boité avec corners | Panneau avec borders, padding standard | Panneau avec borderSubtle, padding réduit | Ligne de séparation seulement | Pas de panneau — données brutes |

### 3.6 Alerts par Mode

| Niveau | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|--------|---------|----------|-----------|----------|-------|
| Format | Boîte complète avec borders | Bordure + texte | Texte préfixé par le glyphe | Texte préfixé par le glyphe | Non applicable |

### 3.7 Dialogs par Mode

| Niveau | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|--------|---------|----------|-----------|----------|-------|
| Format | Taille selon contenu, max 80 cols | Taille selon contenu, max 70 cols | Plein largeur, padding minimal | Taille contenu | Non applicable |

### 3.8 Sidebar par Mode

| Niveau | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|--------|---------|----------|-----------|----------|-------|
| Affichage | Pleine largeur | Largeur réduite, labels tronqués | Icones uniquement, hover pour labels | Icones uniquement | Non applicable |

---

## 4. Configuration du Mode

### 4.1 Via Configuration

Le niveau de densité est configurable dans `opencode.jsonc` :

```jsonc
{
    "density": "STANDARD"
}
```

**Valeurs valides** : `MINIMAL` | `STANDARD` | `DEVELOPER` | `OBSERVER` | `DEBUG`

### 4.2 Via TuiState

Le thème et la densité sont accessibles via `TuiState` :

```typescript
// Lire la densité actuelle
const density = api.tuiConfig.density; // "STANDARD"

// Changer la densité
api.ui.toast({ message: `Density changed to ${newDensity}` });
```

### 4.3 Règles de Configuration

1. `STANDARD` est le **défaut** pour tous les utilisateurs
2. `DEVELOPER` et `OBSERVER` ne DOIVENT être accessibles qu'aux utilisateurs avertis
3. `DEBUG` NE DOIT PAS être le niveau par défaut — réservé au diagnostic
4. `thinkingOpacity` est disponible à TOUS les niveaux
5. La largeur du terminal DOIT être vérifiée AVANT de choisir le niveau
6. `UNKNOWN` et `N/A` sont affichés de manière identique à TOUS les niveaux de densité
7. JAMAIS de données fabriquées pour remplir l'espace dans un niveau de densité
8. Le niveau de densité est SOUVENT configuré par l'utilisateur, jamais imposé
9. **Layout changes must not alter domain behavior** — le mode ne modifie jamais la logique métier
10. `UNKNOWN != ZERO` à tout niveau de densité

---

## 5. Détection Automatique

| Largeur du terminal | Niveau recommandé |
|---------------------|-------------------|
| < 80 | `MINIMAL` |
| 80–99 | `DEVELOPER` |
| 100–119 | `STANDARD` |
| 120–159 | `OBSERVER` |
| ≥ 160 | `STANDARD` (ou `OBSERVER` si préféré) |

**Règles** :
1. La détection automatique utilise la largeur du terminal
2. L'utilisateur peut outrider la détection automatique via la configuration
3. Le changement de niveau DOIT être progressif — pas de saut brutal
4. `UNKNOWN` reste `UNKNOWN` pendant la détection automatique

---

## 6. Transition entre Modes

### 6.1 Règles de Transition

1. La transition DOIT être progressive — pas de saut brutal de MINIMAL à DEBUG
2. Le niveau par défaut est `STANDARD`
3. L'utilisateur peut changer le niveau via une commande ou config
4. La détection automatique est disponible
5. Le focus est préservé pendant la transition
6. `thinkingOpacity` ne change PAS pendant la transition
7. `UNKNOWN` reste `UNKNOWN` pendant la transition
8. Aucune donnée n'est perdue pendant la transition

### 6.2 Comportement de Transition

```
MINIMAL → STANDARD : Ajout des lignes vides, padding passe de space-lg à space-md
STANDARD → DEVELOPER : Suppression des lignes vides, padding passe à space-sm
DEVELOPER → OBSERVER : Padding passe à space-xs, contenu passe en mono
OBSERVER → DEBUG : Suppression des bordures, données brutes
DEBUG → OBSERVER : Restauration des bordures, formatage des données
```

### 6.3 Sécurité de Transition

1. Si la largeur du terminal est insuffisante pour le mode cible, le mode est automatiquement ajusté
2. Un toast `[⚠ WARNING] Density adjusted for terminal width` est affiché si adaptation nécessaire
3. La transition ne modifie jamais les données de domaine
4. Les composants UI se mettent à jour sans rechargement complet

---

## 7. Navigation par Mode — Tableau Récapitulatif

| Zone | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|------|---------|----------|-----------|----------|-------|
| WORKSPACE | Masquée (hamburger) | Sidebar | Sidebar réduite | Contextuel | Données brutes |
| CHAT/SESSION | Maximal | Maximal | Maximal | Réduite | Données brutes |
| AGENT | Status bar | Sidebar | Status bar | Status bar détaillé | Données brutes |
| ACTIVITY | Inaccessible | Déroulant | Contextuel | Maximal | Maximal |
| COMMAND | Accessible | Accessible | Accessible | Accessible | Données brutes |
| OBSERVABILITY | Overlay minimal | Overlay | Overlay | Overlay détaillé | Données brutes |
| TOOLS | Contextuel | Contextuel | Contextuel | Contextuel | Contextuel |

### 7.1 Règles par Mode

| Règle | MINIMAL | STANDARD | DEVELOPER | OBSERVER | DEBUG |
|-------|---------|----------|-----------|----------|-------|
| Sidebar visible | Non | Oui | Oui | Contextuel | Oui |
| ACTIVITY accessible | Non | Oui | Contextuel | Oui | Oui |
| OBSERVABILITY visible | Non | Overlay | Overlay | Overlay | Données brutes |
| TOOLS contextuel | Oui | Oui | Oui | Oui | Oui |

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
9. **Layout changes must not alter domain behavior**
10. `UNKNOWN != ZERO` à tout niveau de densité
11. Le mode NE MODIFIE JAMAIS la logique métier
12. La transition entre niveaux DOIT être progressive

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
**Règle fondamentale** : `UNKNOWN != ZERO`

---

## 10. Références

- `docs/design/density.md` — Niveaux de densité détaillés
- `docs/ux/navigation-model.md` — Navigation par mode
- `docs/ui/layout-modes.md` — Implementation des modes
- `docs/design/components.md` — Composants adaptés par mode
- `docs/design/theme-strategy.md` — Thème indépendant du mode
- `docs/state/presentation-state.md` — `TuiState` et la densité
- `docs/design/design-tokens.md` — Tokens de spacing
- Prompt `08-interaction-personalization.md` — Source du mandat Phase 08
