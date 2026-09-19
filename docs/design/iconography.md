# Iconography — Stratégie Icônes/Glyphes avec Fallback

> Système d'icônes pour le terminal natif EurinHash OpenCode.
> Pas de web SVG — uniquement glyphes terminaux (Unicode box-drawing, symbols, fonts monospace).

---

## 1. Principes Fondamentaux

### 1.1 Terminal-native

Les icônes terminales sont des **glyphes Unicode** rendus directement par le terminal, pas des images ni des SVG. Chaque glyphe DOIT :

- Être présent dans les fontes monospace standard (Menlo, Consolas, DejaVu Sans Mono, Powerline, Nerd Font)
- Avoir un **fallback textuel** pour les terminaux qui ne le supportent pas
- Ne jamais être le seul indicateur de sens

### 1.2 Règle du double signal

Chaque icône DOIT être accompagnée d'un texte ou d'une structure qui transmet le même sens si le glyphe est absent.

```
✓ SUCCESS  (icône + texte)
SUCCESS    (texte seul — fallback)
```

### 1.3 Règle de la non-dépendance chromatique

La couleur n'est JAMAIS le seul canal de transmission du sens d'une icône.

---

## 2. Taxonomie des Glyphes

### 2.1 Statut (11 glyphes)

| Statut | Glyphe Unicode | Fallback textuel | Catégorie |
|---|---|---|---|
| ACTIVE | `●` (U+25CF) | `[ACTIVE]` | Rond plein |
| IDLE | `○` (U+25CB) | `[IDLE]` | Rond vide |
| THINKING | `◐` (U+25D0) | `[THINKING]` | Demi-cercle |
| RUNNING | `▶` (U+25B6) | `[RUNNING]` | Triangle |
| WAITING | `⌛` (U+231B) | `[WAITING]` | Horloge |
| SUCCESS | `✓` (U+2713) | `[SUCCESS]` | Checkmark |
| WARNING | `⚠` (U+26A0) | `[WARNING]` | Triangle alerte |
| ERROR | `✗` (U+2717) | `[ERROR]` | Croix |
| CANCELLED | `⊘` (U+2298) | `[CANCELLED]` | Cercle barré |
| UNKNOWN | `?` (U+003F) | `[UNKNOWN]` | Point d'interrogation |
| DISCONNECTED | `⊘` ou `✗` | `[DISCONNECTED]` | Cercle barré / Croix |

### 2.2 Navigation (16 glyphes)

| Fonction | Glyphe Unicode | Fallback | Description |
|---|---|---|---|
| Retour | `◂` (U+25C4) | `← BACK` | Navigation arrière |
| Avant | `▸` (U+25B8) | `→ FWD` | Navigation avant |
| Haut | `▲` (U+25B2) | `↑ UP` | Monter |
| Bas | `▼` (U+25BC) | `↓ DOWN` | Descendre |
| Menu | `☰` (U+2630) | `[MENU]` | Ouvrir menu |
| Plus | `＋` (U+FF0B) | `[+]` | Ajouter |
| Moins | `－` (U+FF0D) | `[-]` | Réduire |
| Fermer | `✕` (U+2715) | `[X]` | Fermer |
| Recherche | `🔍` ou `⌕` | `[SEARCH]` | Chercher |
| Home | `⌂` (U+2302) | `[HOME]` | Accueil |
| Settings | `⚙` (U+2699) | `[SETTINGS]` | Configuration |
| Help | `?` (U+003F) | `[HELP]` | Aide |

### 2.3 Action (12 glyphes)

| Fonction | Glyphe Unicode | Fallback | Description |
|---|---|---|---|
| Exécuter | `▶` (U+25B6) | `[RUN]` | Lancer |
| Arrêter | `⏹` (U+23F9) | `[STOP]` | Arrêter |
| Pause | `⏸` (U+23F8) | `[PAUSE]` | Mettre en pause |
| Redémarrer | `⟳` (U+1F503) | `[RESTART]` | Recommencer |
| Copier | `⎘` (U+2398) | `[COPY]` | Copier |
| Coller | `⎗` (U+2397) | `[PASTE]` | Coller |
| Supprimer | `🗑` ou `✕` | `[DELETE]` | Supprimer |
| Télécharger | `↓` (U+2193) | `[DOWNLOAD]` | Télécharger |
| Partager | `⤴` (U+21A4) | `[SHARE]` | Partager |
| Export | `⤵` (U+21A5) | `[EXPORT]` | Exporter |
| Importer | `⤴` (U+21A4) | `[IMPORT]` | Importer |
| Refresh | `⟳` ou `↻` | `[REFRESH]` | Rafraîchir |

### 2.4 Indicateurs (8 glyphes)

| Fonction | Glyphe Unicode | Fallback | Description |
|---|---|---|---|
| Chargement | `⟳` ou `◌` | `[LOADING]` | En cours |
| Progression | `▓░` | `[PROGRESS]` | Barre de progression |
| Alerte | `⚡` ou `!` | `[ALERT]` | Alerte |
| Info | `ℹ` (U+2139) | `[INFO]` | Information |
| Confirmation | `✓` | `[OK]` | Confirmé |
| Attention | `‼` (U+203C) | `[CAUTION]` | Double attention |
| Nouveau | `●` | `[NEW]` | Nouveau élément |
| Mis à jour | `↻` | `[UPDATED]` | Mis à jour |

### 2.5 Connectivité (6 glyphes)

| Fonction | Glyphe Unicode | Fallback | Description |
|---|---|---|---|
| Connecté | `●` (U+25CF) | `[CONNECTED]` | En ligne |
| Déconnecté | `○` ou `⊘` | `[DISCONNECTED]` | Hors ligne |
| Réseau | `◉` (U+25C9) | `[NETWORK]` | Réseau actif |
| WiFi | `▦` | `[WiFi]` | WiFi |
| Port | `⏚` (U+231A) | `[PORT]` | Port |
| API | `⟡` ou `⚡` | `[API]` | Endpoint API |

### 2.6 Format (6 glyphes)

| Fonction | Glyphe Unicode | Fallback | Description |
|---|---|---|---|
| Code | `{'` ou `</>` | `[CODE]` | Code |
| Terminal | `▮` | `[TERM]` | Terminal |
| Fichier | `📄` ou `▓` | `[FILE]` | Fichier |
| Dossier | `▓` ou `📁` | `[DIR]` | Dossier |
| Secret | `🔒` ou `⊗` | `[SECRET]` | Donnée sensible |
| Lock | `🔒` ou `⊗` | `[LOCK]` | Verrouillé |

---

## 3. Stratégie de Fallback

### 3.1 Niveaux de Fallback

```
Niveau 1 : Glyphe Unicode complet   → ● ACTIVE
Niveau 2 : Glyphe simplifié         → * ACTIVE
Niveau 3 : Texte seul (crochet)     → [ACTIVE]
Niveau 4 : Texte seul (sans crochets) → ACTIVE
```

### 3.2 Détection de support

Le détecteur de terminal vérifie :

1. **Nerd Font** disponible → tous les glyphes Nerd Font
2. **Fonte monospace Unicode** → glyphes standards Unicode
3. **ASCII uniquement** → fallback Niveau 3 (`[TEXT]`)
4. **Terminal basique** → fallback Niveau 4 (`TEXT`)

### 3.3 Règles de Fallback

- Le fallback DOIT TOUJOURS préserver le sens
- `[TEXT]` est le minimum universel — toujours disponible
- Ne jamais afficher un espace vide ou un caractère `?` sans contexte
- Les glyphes doubles (ex: `✓✓`) ne sont pas des fallbacks — c'est une erreur

---

## 4. Système de Composition

### 4.1 Format Composite

```
[GLYCHE] STATUT · DESCRIPTION · MÉTRIQUE
```

Exemples :

```
[● ACTIVE]  eurinhash-agent  ·  38% · 100K / 262K · $0.00
[◐ THINKING]  worker-novita  ·  2.3s · Attente réponse
[✗ ERROR]  worker-google  ·  Connection refused · N/A
```

### 4.2 Règles de Composition

- Un composant ne DOIT jamais avoir plus d'un glyphe statut par ligne
- Les glyphes de navigation sont TOUJOURS accompagnés de texte
- Les glyphes d'action sont TOUJOURS accompagnés de leur label
- Les métriques sont TOUJOURS en `mono`

---

## 5. Palette de Glyphes par Contexte

### 5.1 Sidebar

| Élément | Glyphe | Couleur |
|---|---|---|
| Session active | `●` | `accent` |
| Session inactive | `○` | `textMuted` |
| Erreur session | `✗` | `error` |
| Agent connecté | `●` | `success` |
| Agent déconnecté | `⊘` | `error` |
| MCP actif | `◆` | `success` |
| MCP erreur | `✗` | `error` |
| LSP actif | `✓` | `success` |
| LSP erreur | `✗` | `error` |

### 5.2 Prompt

| Élément | Glyphe | Couleur |
|---|---|---|
| Mode normal | `›` | `primary` |
| Mode shell | `$` | `mono` |
| Envoi | `▶` | `primary` |
| Annulation | `⊘` | `textMuted` |
| Focus | `❯` | `accent` |

### 5.3 Output / Resultat

| Élément | Glyphe | Couleur |
|---|---|---|
| Succès | `✓` | `success` |
| Échec | `✗` | `error` |
| Avertissement | `⚠` | `warning` |
| Information | `ℹ` | `info` |
| Diff ajouté | `+` | `success` |
| Diff retiré | `−` | `error` |
| Diff contexte | ` ` | `textMuted` |
| Code block | `{'` | `mono` |

### 5.4 Dialog / Confirm

| Élément | Glyphe | Couleur |
|---|---|---|
| Confirmation | `✓` | `success` |
| Annulation | `✕` | `textMuted` |
| Danger | `⚠` | `error` |
| Information | `ℹ` | `info` |
| Select | `›` | `accent` |

---

## 6. Règles d'Icônes

### 6.1 Règles de Non-Fabrication

1. **JAMAIS** créer un glyphe qui n'a pas de correspondance dans Unicode ou une fonte standard
2. **JAMAIS** utiliser un emoji coloré (🟢, 🔴) — les terminaux ne les rendent pas de manière fiable
3. **TOUJOURS** fournir un fallback textuel pour chaque glyphe
4. **JAMAIS** supposer que le terminal supporte les emojis
5. `UNKNOWN` utilise `?` — jamais un glyphe mystère inventé

### 6.2 Règles de Rendu

1. La largeur d'un glyphe = 1 colonne de caractère (monospace)
2. Les glyphes doubles (ex: `⟲`) = 1 colonne si la fonte le supporte, sinon fallback
3. `thinkingOpacity` ne s'applique PAS aux glyphes — uniquement au texte/background
4. Un glyphe de statut DOIT être suivi d'un espace puis du libellé

### 6.3 Règles de Cohérence

1. Utiliser TOUJOURS le même glyphe pour le même statut
2. Ne jamais alterner entre `✓` et `✔` pour SUCCESS
3. Ne jamais alterner entre `✗` et `✕` pour ERROR
4. Les glyphes sont en `regular` ou `bold` selon la hiérarchie, jamais en `italic`

---

## 7. Fallback pour Terminaux Limités

### 7.1 Terminal sans support Unicode

Pour les terminaux qui ne supportent que l'ASCII (ex: certains vieux terminaux, pipelines SSH):

| Glyphe | Fallback ASCII |
|---|---|
| `●` | `*` |
| `○` | `o` |
| `✓` | `OK` |
| `✗` | `ERR` |
| `⚠` | `!` |
| `▶` | `>` |
| `⊘` | `X` |
| `◐` | `..` |
| `⌛` | `W` |
| `?` | `?` |

### 7.2 Terminal sans support coloré

Pour les terminaux monochromes :

- Utiliser les attributs ANSI standards : `bold`, `dim`, `underline`, `reverse`
- `borderSubtle` → `dim`
- `borderActive` → `bold + underline`
- `error` → `bold`
- `success` → `underline`
- `warning` → `bold + dim`

---

## 8. Vérification des Glyphes

Avant d'ajouter un glyphe au système, vérifier :

1. [ ] Le glyphe est dans Unicode standard ou Nerd Font
2. [ ] Le fallback textuel est défini et préserve le sens
3. [ ] Le glyphe fait 1 colonne de large
4. [ ] Le glyphe n'est pas un emoji coloré
5. [ ] Le glyphe n'est pas confondant avec un autre dans le même contexte
6. [ ] `UNKNOWN` ne reçoit jamais un nouveau glyphe — reste `?`
7. [ ] `ZERO` ne reçoit jamais le glyphe `⊘` — c'est réservé à `UNKNOWN`/`CANCELLED`

---

## 9. Extensibilité

Pour ajouter un nouveau glyphe :

1. Vérifier la disponibilité dans les fontes standards
2. Définir le fallback textuel
3. Ajouter à la section correspondante (Statut, Navigation, Action, etc.)
4. Mettre à jour la matrice de fallback si nécessaire
5. **JAMAIS** ajouter un glyphe sans fallback textuel

Le système de glyphes est figé par version — les changements nécessitent une mise à jour du design system documentée.
