# Responsive Behavior — Adaptation 80/100/120/160 Colonnes

> **Source**: `docs/design/terminal-compatibility.md`, `docs/design/density.md`, Prompt `02-ux-information-architecture.md`
> **Date**: 2026-09-19
> **Règle**: « Design for narrow terminals first, then wider terminals. »

---

## 1. Grille de Compatibilité

| Largeur | Nom | Usage | Niveau de densité par défaut |
|---------|-----|-------|------------------------------|
| **80** | Classic | Terminaux historiques, SSH, pipelines | `MINIMAL` |
| **100** | Standard | Terminaux modernes, IDE intégré | `STANDARD` |
| **120** | Large | Terminaux wides, moniteurs larges | `OBSERVER` |
| **160** | Ultra-wide | Moniteurs 4K+, terminaux spéciaux | `OBSERVER` (ou `STANDARD`) |

### 1.1 Détection automatique

```
Au démarrage → Mesurer la largeur du terminal
→ Choisir le niveau de densité recommandé
→ L'utilisateur peut toujours surcharger via /density
Sur resize → Recalculer le layout → Redessiner tous les composants
→ Pas de flicker → Pas de perte de données
```

---

## 2. Adaptation par Zone et Largeur

### 2.1 WORKSPACE

| Élément | 80 cols | 100 cols | 120 cols | 160 cols |
|---------|---------|----------|----------|----------|
| Sidebar | Icônes uniquement, hover pour labels | Labels tronqués | Tous les items | Tous les items + détails |
| Fichiers modifiés | Liste clé-valeur | Table compacte | Table complète | Table large |
| Todos | Liste clé-valeur | Liste clé-valeur | Table | Table + métadonnées |
| Branch info | `[main]` (tronqué si besoin) | `[main]` | `[main]` | `[main] · origin/main` |
| Workspace status | `[●]` / `[✗]` | `[● ACTIVE]` | `[● ACTIVE] · connected` | `[● ACTIVE] · connected · /path` |

### 2.2 CHAT/SESSION

| Élément | 80 cols | 100 cols | 120 cols | 160 cols |
|---------|---------|----------|----------|----------|
| Zone de texte | 60 cols de contenu | 75 cols | 90 cols | 125 cols |
| Message user | Wrap automatique | Wrap automatique | Wrap automatique | Wrap automatique |
| Reasoning | Texte seul, `thinkingOpacity` | Texte seul + indicateur | Texte + barre de progression | Texte + barre + tokens |
| Tool progress | `[▶]` + nom | `[▶] nom` | `[▶] nom · input` | `[▶] nom · input · output` |
| DiffView | JSON lines | Liste clé-valeur | Table compacte | Table complète |
| Cost display | `$X.XX` | `$X.XX · N tokens` | `$X.XX · N tokens · %` | `$X.XX · N tokens · % · time` |
| Token display | `38%` | `38% · 100K` | `38% · 100K/262K` | `38% · 100K/262K · $0.00` |
| Progress bar | Texte seul `38%` | Barre courte | Barre standard | Barre longue |
| Prompt | 60 cols | 75 cols | 90 cols | 125 cols |

### 2.3 AGENT

| Élément | 80 cols | 100 cols | 120 cols | 160 cols |
|---------|---------|----------|----------|----------|
| Status bar | `[●]` `[◐]` `[✗]` `[○]` | `[● A]` `[◐ T]` `[✗ E]` | `[● ACTIVE]` `[◐ THINKING]` | `[● ACTIVE]` `[◐ THINKING]` + score EV |
| Worker detail | Nom + statut uniquement | Nom + statut + temps | Nom + statut + temps + latence | Tout : nom, statut, temps, latence, throughput, coût |
| Model indicator | `model.id` | `provider/model` | `provider/model · variant` | `provider/model · variant · capabilities` |
| EV score | Non affiché | Non affiché | `[38%]` | `[38%] · route: codestral` |
| Latence | Non affiché | Non affiché | `2.3s` | `2.3s · p99=4.1s` |
| Throughput | Non affiché | Non affiché | Non affiché | `100K tok/s` |

### 2.4 ACTIVITY

| Élément | 80 cols | 100 cols | 120 cols | 160 cols |
|---------|---------|----------|----------|----------|
| Event list | JSON lines | Liste clé-valeur | Table compacte | Table complète |
| Event detail | Type uniquement | Type + timestamp | Type + timestamp + payload | Type + timestamp + payload + meta |
| Timestamp | `12:00` | `12:00:00` | `2026-09-19 12:00:00` | `2026-09-19T12:00:00.000Z` |
| Filter | Texte seul | Texte + icône | Texte + icône + count | Texte + icône + count + details |

### 2.5 COMMAND

| Élément | 80 cols | 100 cols | 120 cols | 160 cols |
|---------|---------|----------|----------|----------|
| Prompt | `› _` | `› _` | `› _` | `› _` |
| Command palette | Plein largeur | Plein largeur | 80% centrée | 70% centrée |
| Keybind display | `Ctrl+D → density` | `Ctrl+D → Change density` | `Ctrl+D → Change density level` | `Ctrl+D → Change density level (MINIMAL/STANDARD/...)` |
| Dialog | Plein largeur | Plein largeur | Taille contenu, max 70 cols | Taille contenu, max 80 cols |
| Toast | Texte seul | Texte + variant | Texte + variant + durée | Texte + variant + durée + timestamp |

---

## 3. Adaptation par Composant

### 3.1 Table

| Largeur | Format |
|---------|--------|
| ≥ 160 | Table complète avec bordures, headers, toutes les colonnes |
| 120–159 | Table complète, colonnes principales |
| 100–119 | Table compacte, headers réduits |
| 80–99 | Liste clé-valeur |
| < 80 | JSON lines |

**Exemple (≥ 120 cols)** :
```
┌─────────────┬──────────┬──────────┐
│ Name        │ Status   │ Progress │
├─────────────┼──────────┼──────────┤
│ eurinhash   │ ACTIVE   │ 38%      │
│ novita      │ THINKING │ 23%      │
│ google      │ ERROR    │ N/A      │
└─────────────┴──────────┴──────────┘
```

**Exemple (80–99 cols)** :
```
Name:     eurinhash
Status:   ACTIVE
Progress: 38%

Name:     novita
Status:   THINKING
Progress: 23%
```

### 3.2 Panel

| Largeur | Format |
|---------|--------|
| ≥ 120 | Coins + bordures complètes + padding `space-md` |
| 100–119 | Bordures simples + padding `space-sm` |
| 80–99 | Bordure supérieure uniquement |
| < 80 | Ligne de séparation `─` |

**Exemple (≥ 120 cols)** :
```
╭──────────────────────────────────────────╮
│  Information Title                         │
│                                          │
│  Content inside the panel.               │
╰──────────────────────────────────────────╯
```

### 3.3 ProgressBar

| Largeur | Format |
|---------|--------|
| ≥ 120 | `[████████░░] 38% · 100K / 262K` |
| 100–119 | `[████░░░░░░] 38%` |
| 80–99 | `38% · 100K/262K` (texte seul) |
| < 80 | `38%` (texte seul) |

### 3.4 Alert

| Largeur | Format |
|---------|--------|
| ≥ 120 | Boîte complète avec borders |
| 100–119 | Bordure + texte |
| 80–99 | Texte préfixé par le glyphe |
| < 80 | Texte seul avec `[⚠]` prefix |

### 3.5 Badge

| Largeur | Format |
|---------|--------|
| ≥ 120 | `[● ACTIVE]` |
| 100–119 | `[● ACT]` |
| 80–99 | `[● A]` ou `●` |
| < 80 | `●` (glyphe seul) |

---

## 4. Règles de Non-Fabrication par Largeur

### 4.1 `UNKNOWN` reste `UNKNOWN` à toute largeur

| Règle | Description |
|-------|-------------|
| `UNKNOWN` | Ne JAMAIS changer en `0` ou `N/A` si la largeur change |
| `N/A` | Reste `N/A` quelle que soit la largeur |
| `$0.00` | Reste `$0.00` si le coût est connu et nul |
| `100K · 38%` | Quand la limite est inconnue, afficher ainsi — jamais compléter |
| `38%` | Pourcentage simple, valide à toute largeur |

### 4.2 Ajustements de format autorisés (mais pas de fabrication)

| Situation | Format 80 cols | Format 120+ cols | Interdit |
|-----------|---------------|-----------------|----------|
| Coût inconnu | `N/A` | `N/A` | `0` ou `$0.00` |
| Limite inconnue | `100K · 38%` | `100K / 262K · 38%` | `100K · 38%` quand la limite est connue |
| Statut inconnu | `[?]` ou `[? U]` | `[? UNKNOWN]` | `[○ IDLE]` ou tout autre statut |
| Branch absente | `null` ou pas affiché | `null` | `""` |

### 4.3 Règle de la largeur minimale d'affichage

- Un statut ne DOIT JAMAIS être tronqué au point de perdre son sens
- `[● ACTIVE]` → si trop court → `[● ACT]` → si encore trop court → `●`
- `[✗ ERROR]` → si trop court → `[✗ ERR]` → si encore trop court → `✗`
- Le fallback textuel est TOUJOURS disponible
- `UNKNOWN` reste lisible à toute largeur

---

## 5. Gestion du Resize

### 5.1 Détection de Resize

```
Sur événement resize du terminal :
1. Mesurer la nouvelle largeur
2. Recalculer le layout (mode + largeur)
3. Redessiner tous les composants
4. Vérifier qu'aucun texte n'est tronqué de manière abusive
5. Vérifier que UNKNOWN != ZERO est respecté
6. Vérifier que thinkingOpacity est conservé
```

### 5.2 Seuil de Resize Critique

| Transition | Action |
|------------|--------|
| > 120 → < 120 | Basculer de OBSERVER à STANDARD |
| 120 → 100 | Réduire les tables, tronquer les labels |
| 100 → 80 | Masquer la sidebar, réduire le padding, réduire les métriques |
| < 80 | Mode MINIMAL, sidebar cachée, tout en compact |
| < 80 → ≥ 80 | Rétablir progressivement le layout |

### 5.3 Règles de Resize

1. **Pas de flicker** — le resize DOIT être fluide
2. **Pas de perte de données** — le contenu ne doit pas disparaître
3. **Rétrogradation progressive** — passer de 160 à 80 cols en étapes
4. **`thinkingOpacity` ne change pas** lors d'un resize
5. Les statuts restent lisibles après resize
6. `UNKNOWN` reste `UNKNOWN` après resize
7. Les métriques sont en `mono` et ne changent pas de format
8. `thinkingOpacity` ne crée pas de contenu supplémentaire

### 5.4 Commandes de Largeur

| Commande | Action |
|----------|--------|
| `/width 80` | Forcer 80 colonnes |
| `/width 100` | Forcer 100 colonnes |
| `/width 120` | Forcer 120 colonnes |
| `/width 160` | Forcer 160 colonnes |
| `/width auto` | Détection automatique |
| `/terminal info` | Largeur, hauteur, type, couleurs supportées |
| `/terminal test` | Test de rendu de tous les glyphes |

---

## 6. Compatibilité Terminale

### 6.1 Types de Terminaux Supportés

| Terminal | Largeurs | Couleurs | Unicode | Notes |
|----------|----------|----------|---------|-------|
| iTerm2 | 80–160+ | 256/Truecolor | Oui | macOS |
| Terminal.app | 80–160+ | 256/Truecolor | Oui | macOS |
| Windows Terminal | 80–160+ | 256/Truecolor | Oui | Windows |
| Konsole | 80–160+ | 256/Truecolor | Oui | Linux |
| GNOME Terminal | 80–160+ | 256/Truecolor | Oui | Linux |
| Alacritty | 80–160+ | Truecolor | Oui | Multi-plateforme |
| Kitty | 80–160+ | Truecolor | Oui | Multi-plateforme |
| WezTerm | 80–160+ | Truecolor | Oui | Multi-plateforme |
| VS Code Terminal | 80–160+ | Truecolor | Oui | Intégré |
| Tmux | 80–160+ | 256/Truecolor | Limité | Wrapper |
| Screen | 80–120 | 256 | Limité | Legacy |

### 6.2 Fallback par Fonctionnalité

| Fonctionnalité non supportée | Fallback |
|------------------------------|----------|
| Unicode avancé | ASCII + crochets `[...]` |
| Truecolor | ANSI 256 colors |
| Box-drawing | ASCII `| - +` |
| Nerd Font | Texte seul ou ASCII |
| Mouse support | Navigation clavier |
| Split panes | Full screen alterné |

---

## 7. Matrice de Compatibilité Résumée

| Composant | 80 cols | 100 cols | 120 cols | 160 cols |
|-----------|---------|----------|----------|----------|
| Sidebar | Icones + hover | Réduite | Standard | Standard |
| Table | Liste clé-valeur | Table compacte | Table complète | Table large |
| Progress bar | Texte seul | Barre courte | Barre standard | Barre longue |
| Panel | Ligne de séparation | Bordure simple | Bordure complète | Bordure double |
| Alert | Texte prefixé | Bordure + texte | Boîte complète | Boîte complète |
| Dialog | Plein largeur | Plein largeur | Taille contenu | Taille contenu |
| Badge | `[● A]` | `[● ACT]` | `[● ACTIVE]` | `[● ACTIVE]` |
| Métriques | `38%` | `38% · 100K` | `38% · 100K/262K` | `38% · 100K/262K · $0.00` |
| Status | Texte seul | Glyphe + texte | Glyphe + texte | Glyphe + texte complet |

---

## 8. Règles Fondamentales de Responsive

1. **JAMAIS** supposer une largeur de terminal — TOUJOURS détecter
2. **TOUJOURS** avoir un fallback pour 80 colonnes
3. **JAMAIS** de ligne qui dépasse la largeur du terminal sans wrap
4. **TOUJOURS** `wrap` pour le texte, jamais `overflow`
5. `thinkingOpacity` est **indépendant** de la largeur du terminal
6. Les glyphes de statut sont TOUJOURS lisibles à 80 colonnes
7. `UNKNOWN` ne change JAMAIS de valeur en fonction de la largeur
8. `N/A` reste `N/A` à toute largeur
9. Les métriques sont en `mono` et ne changent pas de format
10. **JAMAIS** de données fabriquées pour remplir l'espace dans un niveau de densité

---

## 9. Interaction entre Responsive et Densité

| Largeur | Densité recommandée | Notes |
|---------|---------------------|-------|
| < 80 | `MINIMAL` | Sidebar masquée, layout réduit |
| 80–99 | `DEVELOPER` | Pas de lignes vides, padding réduit |
| 100–119 | `STANDARD` | Layout standard, 1 ligne entre sections |
| 120–159 | `OBSERVER` | Layout large, métriques détaillées |
| ≥ 160 | `OBSERVER` ou `STANDARD` | Choix utilisateur |

### 9.1 Règles de transition

- La transition entre niveaux DOIT être progressive
- Pas de saut brutal de MINIMAL à DEBUG
- Le niveau par défaut est `STANDARD`
- L'utilisateur peut changer le niveau via `/density` ou une config
- La largeur du terminal DOIT être vérifiée AVANT de choisir le niveau
- `UNKNOWN` et `N/A` sont affichés de manière identique à TOUS les niveaux

---

## 10. Références

- `docs/design/terminal-compatibility.md` — Source primaire de compatibilité terminale
- `docs/design/density.md` — Niveaux de densité et leur interaction avec la largeur
- `docs/design/components.md` — Adaptation par largeur des composants
- `docs/design/design-tokens.md` — Tokens et leur application par largeur
- `docs/architecture/architecture-overview.md` — Architecture et rendu TUI
- Prompt `02-ux-information-architecture.md` — Source du mandat
