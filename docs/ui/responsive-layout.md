# Responsive Layout — Compatibilité 80/100/120/160 Colonnes

> Stratégie de compatibilité terminale pour le Command Center EurinHash OpenCode.
> Largeurs supportées : 80, 100, 120, 160 colonnes.
> Layout target : WORKSPACE | CHAT/SESSION | AGENT | ACTIVITY | COMMAND
> Règle fondamentale : `UNKNOWN != ZERO`

---

## 1. Largeurs Standards

| Largeur | Nom | Usage | Support |
|---------|-----|-------|---------|
| 80 | Classic | Terminaux historiques, SSH, pipelines | Universel |
| 100 | Standard | Terminaux modernes, IDE intégré | Quasi-universel |
| 120 | Large | Terminaux wides, moniteurs larges | Courant |
| 160 | Ultra-wide | Moniteurs 4K+, terminaux spéciaux | Limité |

---

## 2. Grille de Compatibilité

### 2.1 Layout par Largeur

| Élément | 80 cols | 100 cols | 120 cols | 160 cols |
|---------|---------|----------|----------|----------|
| Sidebar | Réduite (20 cols) | Standard (25 cols) | Standard (30 cols) | Standard (35 cols) |
| Zone principale | 60 cols | 75 cols | 90 cols | 125 cols |
| Prompt | 60 cols | 75 cols | 90 cols | 125 cols |
| Table | Colonnes minimales | Table compacte | Table complète | Table large |
| Barre de progression | Texte seul | Barre courte | Barre standard | Barre longue |
| Panneaux | Plein largeur | Plein largeur | Possibilité de split | Split multi-colonnes |
| Borders | Simple | Standard | Double | Triple |
| Activity Center | Compact | Compact | Détaillé | Détaillé + métriques |

### 2.2 Règles de Largeur Minimale

| Élément | Largeur min | Comportement en dessous |
|---------|-------------|------------------------|
| Sidebar | 20 cols | Masquée, hamburger menu |
| Table | 40 cols | Colonnes cachées, scroll horizontal |
| Barre de progression | 20 cols | Texte seul `38%` |
| Panneau | 30 cols | Réduit à l'essentiel |
| Alerte | 40 cols | Compacte, texte seul |
| Dialog | 50 cols | Plein largeur si plus petit |
| Commande | 60 cols | Wrap automatique |
| StatusIndicator | 40 cols | Label tronqué puis glyphe seul |
| EventLog | 40 cols | Format compact, timestamps minimaux |
| FilterBar | 40 cols | Icônes uniquement, hover pour labels |
| ProgressBar | 20 cols | Texte seul `38%` |
| AgentStatusBar | 40 cols | Worker name tronqué |

---

## 3. Adaptation Dynamique

### 3.1 Détection de Largeur

```
La largeur du terminal est détectée au démarrage et lors du resize.
Le layout s'adapte automatiquement sans rechargement.
```

### 3.2 Seuil de Transition

| De | Vers | Action |
|-----|------|--------|
| < 80 | MINIMAL | Sidebar masquée, layout réduit |
| 80–99 | DEVELOPER | Pas de lignes vides, padding réduit |
| 100–119 | STANDARD | Layout standard |
| 120–159 | OBSERVER | Layout large, métriques détaillées |
| ≥ 160 | OBSERVER/STANDARD | Layout étendu (choix utilisateur) |

### 3.3 Règles d'Adaptation

1. **Ne jamais** dépasser la largeur du terminal — pas de wrap forcé sur les métriques
2. **Toujours** prévoir un fallback pour 80 colonnes
3. **Jamais** de horizontal scroll dans le terminal — tout doit tenir sur une ligne
4. **TOUJOURS** `wrap` pour le texte long (pas de ligne infinie)
5. `thinkingOpacity` ne change pas avec la largeur du terminal
6. Les statuts restent lisibles après resize
7. `UNKNOWN` reste `UNKNOWN` après resize
8. `N/A` reste `N/A` après resize
9. `$0.00` reste `$0.00` après resize

---

## 4. Adaptation par Composant

### 4.1 Sidebar

| Largeur | Comportement |
|---------|-------------|
| ≥ 120 | Pleine largeur, tous les items visibles |
| 100–119 | Largeur réduite, labels tronqués |
| 80–99 | Icones uniquement, hover pour labels |
| < 80 | Masquée, hamburger pour navigation |

**Implémentation** :

```typescript
function getSidebarWidth(terminalWidth: number): number {
    if (terminalWidth >= 120) return 30;
    if (terminalWidth >= 100) return 25;
    if (terminalWidth >= 80) return 20;
    return 0; // Masquée
}
```

**Règle UNKNOWN** : Quand la sidebar est masquée (< 80), les informations workspace sont toujours accessibles via le hamburger menu. `branch: null` reste `null` — jamais `""`.

---

### 4.2 Table

| Largeur | Comportement |
|---------|-------------|
| ≥ 160 | Toutes les colonnes visibles |
| 120–159 | Colonnes principales visibles |
| 100–119 | Colonnes essentielles seulement |
| 80–99 | Une colonne par ligne (liste) |
| < 80 | Format JSON lines |

**Implémentation** :

```typescript
function getTableFormat(width: number): TableFormat {
    if (width >= 160) return "full";
    if (width >= 120) return "main-columns";
    if (width >= 100) return "essential";
    if (width >= 80) return "list";
    return "json-lines";
}
```

**Règle UNKNOWN** : `additions: 0` est un vrai zéro (0 lignes ajoutées). `branch: undefined` est absent — jamais `""`.

---

### 4.3 Barre de Progression

| Largeur | Format |
|---------|--------|
| ≥ 120 | `[████████░░] 38% · 100K / 262K` |
| 100–119 | `[████░░░░] 38%` |
| 80–99 | `38% · 100K/262K` (texte seul) |
| < 80 | `38%` (texte seul) |

**Implémentation** :

```typescript
function getProgressBarFormat(width: number): ProgressBarFormat {
    if (width >= 120) return "full";
    if (width >= 100) return "short";
    if (width >= 80) return "text-only";
    return "percent-only";
}
```

**Règle UNKNOWN** : Quand la limite est inconnue, afficher `100K · 38%` — jamais compléter avec une limite fabriquée. Quand le coût n'est pas calculé, afficher `N/A` — jamais `$0.00`.

---

### 4.4 Panneaux

| Largeur | Comportement |
|---------|-------------|
| ≥ 120 | Bordures complètes avec corners |
| 100–119 | Bordures simples |
| 80–99 | Bordure supérieure uniquement |
| < 80 | Ligne de séparation `─` |

**Implémentation** :

```typescript
function getPanelBorder(width: number): PanelBorder {
    if (width >= 120) return { corners: true, borders: "full" };
    if (width >= 100) return { corners: false, borders: "simple" };
    if (width >= 80) return { corners: false, borders: "top-only" };
    return { corners: false, borders: "separator" };
}
```

**Règle UNKNOWN** : Les bordures changent mais les données restent identiques. `UNKNOWN != ZERO` s'applique au contenu, pas à la bordure.

---

### 4.5 Alertes

| Largeur | Comportement |
|---------|-------------|
| ≥ 120 | Boîte complète avec borders |
| 100–119 | Bordure + texte |
| 80–99 | Texte préfixé par le glyphe |
| < 80 | Texte seul avec `[⚠]` prefix |

**Implémentation** :

```typescript
function getAlertFormat(width: number): AlertFormat {
    if (width >= 120) return "full-box";
    if (width >= 100) return "border-text";
    if (width >= 80) return "prefixed";
    return "text-only";
}
```

---

### 4.6 Dialogs

| Largeur | Comportement |
|---------|-------------|
| ≥ 120 | Taille selon contenu, max 80 cols |
| 100–119 | Taille selon contenu, max 70 cols |
| 80–99 | Plein largeur, padding minimal |
| < 80 | Plein largeur, sans padding |

---

### 4.7 StatusBar (Agent)

| Largeur | Comportement |
|---------|-------------|
| ≥ 160 | Tous les workers affichés avec métriques complètes |
| ≥ 120 | Workers affichés avec métriques |
| ≥ 100 | Workers affichés, noms complets |
| ≥ 80 | Workers affichés, noms tronqués |
| < 80 | Statut réduit à `[●]` `[◐]` `[✗]` |

**Règle UNKNOWN** : Un worker non interrogé affiche `[? UNKNOWN]` — jamais `[○ IDLE]`. Quand la largeur force le tronquage, le statut reste `[?]` ou `[●]`, jamais `[○]` pour UNKNOWN.

---

### 4.8 Activity Center

| Largeur | Comportement |
|---------|-------------|
| ≥ 160 | Événements complets, timestamps détaillés, filtres développés |
| ≥ 120 | Événements complets, timestamps standards, filtres par catégorie |
| ≥ 100 | Événements avec timestamps, filtres réduits |
| ≥ 80 | Événements compact, timestamps minimaux |
| < 80 | Événements texte seul, séparation par type |

---

### 4.9 Command (Prompt)

| Largeur | Comportement |
|---------|-------------|
| ≥ 120 | Prompt complet avec tous les éléments |
| ≥ 100 | Prompt complet |
| ≥ 80 | Prompt avec wrap automatique |
| < 80 | Prompt minimal, mode indication |

**Règle UNKNOWN** : `TuiPromptInfo.mode: undefined` ≠ `"normal"`. Le mode non spécifié ne change jamais en `"normal"` suite à un resize.

---

## 5. Contenu par Largeur

### 5.1 Métriques Télémetry

| Largeur | Format |
|---------|--------|
| ≥ 120 | `100K / 262K · 38% · $0.00` |
| 100–119 | `100K · 38% · $0.00` |
| 80–99 | `38% · $0.00` |
| < 80 | `38%` |

### 5.2 Règles de Non-Fabrication par Largeur

| Règle | Description |
|-------|-------------|
| `UNKNOWN` | Ne JAMAIS changer en `0` ou `N/A` si la largeur change |
| `N/A` | Reste `N/A` quelle que soit la largeur |
| `$0.00` | Reste `$0.00` si le coût est connu et nul |
| `100K · 38%` | Quand la limite est inconnue, afficher ainsi — jamais compléter |
| `38%` | Pourcentage simple, valide à toute largeur |
| `[● ACTIVE]` | Si trop court → `[● ACT]` → si encore trop court → `●` |
| `[✗ ERROR]` | Si trop court → `[✗ ERR]` → si encore trop court → `✗` |

### 5.3 Règle de la Largeur Minimale d'Affichage

- Un statut ne DOIT JAMAIS être tronqué au point de perdre son sens
- `[● ACTIVE]` → si trop court → `[● ACT]` → si encore trop court → `●`
- `[✗ ERROR]` → si trop court → `[✗ ERR]` → si encore trop court → `✗`
- Le fallback textuel est TOUJOURS disponible
- `UNKNOWN` utilise `?` — jamais un glyphe mystère inventé
- `ZERO` ne reçoit jamais le glyphe `⊘` — c'est réservé à `UNKNOWN`/`CANCELLED`

---

## 6. Gestion du Resize

### 6.1 Détection de Resize

```
Sur événement resize du terminal :
1. Mesurer la nouvelle largeur
2. Recalculer le layout
3. Redessiner tous les composants
4. Vérifier qu'aucun texte n'est tronqué de manière abusive
```

### 6.2 Règles de Resize

1. **Pas de flicker** — le resize DOIT être fluide
2. **Pas de perte de données** — le contenu ne doit pas disparaître
3. **Rétrogradation progressive** — passer de 160 à 80 cols en étapes
4. `thinkingOpacity` ne change pas lors d'un resize
5. Les statuts restent lisibles après resize
6. `UNKNOWN` reste `UNKNOWN` après resize
7. `N/A` reste `N/A` après resize
8. `$0.00` reste `$0.00` après resize
9. `100K · 38%` reste après resize quand la limite est inconnue
10. Les métriques sont en `mono` et ne changent pas de format

### 6.3 Seuil de Resize Critique

| Transition | Action |
|------------|--------|
| > 120 → < 120 | Basculer de OBSERVER à STANDARD |
| 120 → 100 | Réduire les tables |
| 100 → 80 | Masquer la sidebar, réduire le padding |
| < 80 | Mode MINIMAL, sidebar cachée |

### 6.4 Flux de Resize

```
Resize event
    │
    ▼
Mesurer la nouvelle largeur
    │
    ▼
Vérifier le seuil de transition
    │
    ├── Largeur inchangée → Rien
    ├── Largeur augmentée → Mise à niveau progressive
    └── Largeur diminuée → Rétrogradation progressive
            │
            ▼
    Recalculer le layout
            │
            ▼
    Redessiner les composants
            │
            ▼
    Vérifier : aucun texte tronqué abusivement
            │
            ▼
    Vérifier : UNKNOWN reste UNKNOWN
            │
            ▼
    Vérifier : N/A reste N/A
            │
            ▼
    Vérifier : $0.00 reste $0.00
            │
            ▼
    Rendu final
```

---

## 7. Compatibilité Terminale

### 7.1 Types de Terminaux Supportés

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

### 7.2 Fonctionnalités par Terminal

| Fonctionnalité | 80+ | 100+ | 120+ | 160+ |
|----------------|-----|------|------|------|
| Couleurs ANSI | ✓ | ✓ | ✓ | ✓ |
| Truecolor | ✓ | ✓ | ✓ | ✓ |
| Unicode basique | ✓ | ✓ | ✓ | ✓ |
| Unicode avancé | Partiel | ✓ | ✓ | ✓ |
| Box-drawing | ✓ | ✓ | ✓ | ✓ |
| Icônes Nerd Font | Partiel | ✓ | ✓ | ✓ |
| Split panes | ✓ | ✓ | ✓ | ✓ |
| Scrollback | Standard | Standard | Grand | Illimité |
| Mouse support | ✓ | ✓ | ✓ | ✓ |
| Bracketed paste | ✓ | ✓ | ✓ | ✓ |
| Alt screen buffer | ✓ | ✓ | ✓ | ✓ |

### 7.3 Fallback par Terminal

| Fonctionnalité non supportée | Fallback |
|------------------------------|----------|
| Unicode avancé | ASCII + crochets `[...]` |
| Truecolor | ANSI 256 colors |
| Box-drawing | ASCII `| - +` |
| Nerd Font | Texte seul ou ASCII |
| Mouse support | Navigation clavier |
| Split panes | Full screen alterné |

---

## 8. Commandes de Compatibilité

### 8.1 Commandes de Diagnostic

| Commande | Résultat |
|----------|----------|
| `/terminal info` | Largeur, hauteur, type, couleurs supportées |
| `/terminal test` | Test de rendu de tous les glyphes |
| `/terminal fallback` | Mode fallback activé |
| `/terminal detect` | Détection automatique du terminal |

### 8.2 Commandes de Largeur

| Commande | Action |
|----------|--------|
| `/width 80` | Forcer 80 colonnes |
| `/width 100` | Forcer 100 colonnes |
| `/width 120` | Forcer 120 colonnes |
| `/width 160` | Forcer 160 colonnes |
| `/width auto` | Détection automatique |

### 8.3 Commandes de Density

| Commande | Action |
|----------|--------|
| `/density` | Afficher le niveau actuel |
| `/density MINIMAL` | Passer en MINIMAL |
| `/density STANDARD` | Passer en STANDARD |
| `/density DEVELOPER` | Passer en DEVELOPER |
| `/density OBSERVER` | Passer en OBSERVER |
| `/density DEBUG` | Passer en DEBUG |

---

## 9. Règles de Compatibilité

### 9.1 Règles Fondamentales

1. **JAMAIS** supposer une largeur de terminal — TOUJOURS détecter
2. **TOUJOURS** avoir un fallback pour 80 colonnes
3. **JAMAIS** de ligne qui dépasse la largeur du terminal sans wrap
4. **TOUJOURS** `wrap` pour le texte, jamais `overflow`
5. `thinkingOpacity` est **indépendant** de la largeur du terminal
6. Les glyphes de statut sont TOUJOURS lisibles à 80 colonnes
7. `UNKNOWN != ZERO` à toutes les largeurs
8. `N/A` reste `N/A` à toutes les largeurs
9. `$0.00` reste `$0.00` à toutes les largeurs
10. Les métriques sont en `mono` à toutes les largeurs
11. `100K · 38%` reste à toutes les largeurs quand la limite est inconnue

### 9.2 Règles de Non-Fabrication par Largeur

| Règle | Description |
|-------|-------------|
| `UNKNOWN` | Ne pas changer en `0` ou `IDLE` quand on réduit la largeur |
| `N/A` | Ne jamais remplacer par `0` ou `-` quand on réduit la largeur |
| `$0.00` | Ne jamais supprimer quand on réduit la largeur |
| `38%` | Pourcentage toujours affiché, quel que soit le format |
| `100K · 38%` | Quand la limite est inconnue, afficher ainsi — jamais compléter |

### 9.3 Règles de Contenu

1. **Ne jamais** tronquer un statut au point de perdre son identité
2. **Toujours** maintenir le texte du statut lisible
3. **JAMAIS** de données fabriquées pour remplir l'espace
4. Les métriques sont en `mono` et ne changent pas de format
5. `thinkingOpacity` ne crée pas de contenu supplémentaire
6. `UNKNOWN` reste `UNKNOWN` après resize
7. Le fallback textuel est TOUJOURS disponible

---

## 10. Matrice de Compatibilité Résumée

### 10.1 Composants

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
| AgentStatusBar | `[●]` `[✗]` | Nom + `[●]` | Nom complet + `[●]` | Nom complet + métriques |
| EventLog | Compact | Texte + timestamps | Texte + timestamps | Texte + timestamps + filtres |

### 10.2 UNKNOWN par Largeur

| Composant | 80 cols | 100 cols | 120 cols | 160 cols |
|-----------|---------|----------|----------|----------|
| `UNKNOWN` | `?` | `?` | `?` | `?` |
| `N/A` | `N/A` | `N/A` | `N/A` | `N/A` |
| `$0.00` | `$0.00` | `$0.00` | `$0.00` | `$0.00` |
| `100K · 38%` | `100K · 38%` | `100K · 38%` | `100K · 38%` | `100K · 38%` |

`UNKNOWN` ne change JAMAIS de format avec la largeur du terminal.

---

## 11. Checklist de Déploiement

Avant de déployer sur un environnement terminal, vérifier :

- [ ] Rendu correct à 80 colonnes
- [ ] Rendu correct à 100 colonnes
- [ ] Rendu correct à 120 colonnes
- [ ] Rendu correct à 160 colonnes
- [ ] `UNKNOWN` affiché correctement à toutes les largeurs
- [ ] `N/A` affiché correctement à toutes les largeurs
- [ ] `thinkingOpacity` visible à toutes les largeurs
- [ ] Pas de ligne dépassant la largeur du terminal
- [ ] Tous les statuts lisibles à 80 colonnes
- [ ] Fallback ASCII fonctionnel
- [ ] `ZERO` ≠ `UNKNOWN` vérifié
- [ ] `$0.00` affiché correctement (pas supprimé)
- [ ] `100K · 38%` affiché correctement quand la limite est inconnue
- [ ] Les métriques sont en `mono`
- [ ] Les glyphes sont suivis d'un espace et du texte
- [ ] Resize fluide sans flicker
- [ ] Pas de perte de données lors du resize
- [ ] `UNKNOWN` reste `UNKNOWN` après resize
- [ ] `N/A` reste `N/A` après resize
- [ ] Mouse support préservé
- [ ] Keybindings préservées

---

## 12. Résumé

| Largeur | Nom | Layout | Sidebar | Métriques | Usage |
|---------|-----|--------|---------|-----------|-------|
| 80 | Classic | MINIMAL | Masquée | `38%` | Terminaux historiques |
| 100 | Standard | STANDARD | Réduite | `38% · 100K` | Usage quotidien |
| 120 | Large | OBSERVER | Standard | `38% · 100K/262K` | Monitoring |
| 160 | Ultra-wide | OBSERVER | Standard | `38% · 100K/262K · $0.00` | Métriques détaillées |

**Layout target** : WORKSPACE | CHAT/SESSION | AGENT | ACTIVITY | COMMAND
**Règle fondamentale** : `UNKNOWN != ZERO` — à toutes les largeurs