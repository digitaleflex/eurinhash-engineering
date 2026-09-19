# Status Language — Vocabulaire de Statut

> Vocabulaire sémantique des états pour EurinHash OpenCode.
> Chaque état a une sémantique visuelle textuelle ET colorée.
> `UNKNOWN != ZERO` — jamais confondre une absence de donnée avec une valeur nulle.

---

## 1. Lexique des Statuts

### 1.1 ACTIVE

- **Sémantique** : Opérationnel, prêt, en service
- **Couleur** : `accent`
- **Glyphe** : `●` (point plein)
- **Badge** : `[● ACTIVE]`
- **Usage** : Agent connecté, session active, service joignable
- **Exemple** : `● ACTIVE · EURINHASH v0.13.0`

### 1.2 IDLE

- **Sémantique** : Inactif, en attente, pas de tâche en cours
- **Couleur** : `textMuted`
- **Glyphe** : `○` (point vide)
- **Badge** : `[○ IDLE]`
- **Usage** : Agent sans tâche, session ouverte mais inactive
- **Exemple** : `○ IDLE · Aucune tâche en cours`

### 1.3 THINKING

- **Sémantique** : En réflexion, en cours de raisonnement interne
- **Couleur** : `secondary` avec `thinkingOpacity` appliqué
- **Glyphe** : `◐` (demi-cercle)
- **Badge** : `[◐ THINKING]`
- **Usage** : Agent en cours de traitement cognitif
- **Exemple** : `◐ THINKING · Analyse de la demande...`
- **Note** : `thinkingOpacity` modifie l'opacité visuelle — jamais de changement de couleur

### 1.4 RUNNING

- **Sémantique** : En exécution active, tâche en cours
- **Couleur** : `primary`
- **Glyphe** : `▶` (triangle droit)
- **Badge** : `[▶ RUNNING]`
- **Usage** : Agent exécutant une action, processus actif
- **Exemple** : `▶ RUNNING · Agent: code-review en cours`

### 1.5 WAITING

- **Sémantique** : En attente d'une entrée externe, d'une réponse, d'une ressource
- **Couleur** : `warning`
- **Glyphe** : `⌛` (horloge)
- **Badge** : `[⌛ WAITING]`
- **Usage** : Attente d'approbation, de permission, de réponse API
- **Exemple** : `[⌛ WAITING] · Permission requise par l'utilisateur`

### 1.6 SUCCESS

- **Sémantique** : Réussi, complet, opération terminée
- **Couleur** : `success`
- **Glyphe** : `✓` (checkmark)
- **Badge** : `[✓ SUCCESS]`
- **Usage** : Tâche complétée avec succès
- **Exemple** : `[✓ SUCCESS] · Build terminé en 2.3s`

### 1.7 WARNING

- **Sémantique** : Avertissement, condition non-idéale mais fonctionnelle
- **Couleur** : `warning`
- **Glyphe** : `⚠` (triangle d'alerte)
- **Badge** : `[⚠ WARNING]`
- **Usage** : Configuration suboptimale, near-limit, comportement attendu avec réserves
- **Exemple** : `[⚠ WARNING] · Approchant la limite de 262K tokens`

### 1.8 ERROR

- **Sémantique** : Échec, rupture, condition non-récupérable
- **Couleur** : `error`
- **Glyphe** : `✗` (croix)
- **Badge** : `[✗ ERROR]`
- **Usage** : Opération échouée, erreur critique
- **Exemple** : `[✗ ERROR] · Connection refused to MCP endpoint`

### 1.9 CANCELLED

- **Sémantique** : Annulé par l'utilisateur ou le système, non terminé
- **Couleur** : `textMuted`
- **Glyphe** : `⊘` (cercle barré)
- **Badge** : `[⊘ CANCELLED]`
- **Usage** : Tâche interrompue avant completion
- **Exemple** : `[⊘ CANCELLED] · Build annulé par l'utilisateur`

### 1.10 UNKNOWN

- **Sémantique** : Indéterminé, information non disponible
- **Couleur** : `textMuted`
- **Glyphe** : `?` (point d'interrogation)
- **Badge** : `[? UNKNOWN]`
- **Usage** : Donnée absente, statut non déterminable, pas encore initialisé
- **Exemple** : `[? UNKNOWN] · Coût de l'agent non calculé`
- **Règle** : `UNKNOWN != ZERO` — ne jamais afficher `0` ou vide pour UNKNOWN

### 1.11 DISCONNECTED

- **Sémantique** : Déconnecté, lien perdu, endpoint injoignable
- **Couleur** : `error`
- **Glyphe** : `⊘` (cercle barré variant) ou `✗`
- **Badge** : `[⊘ DISCONNECTED]`
- **Usage** : Agent ou service perdu la connexion
- **Exemple** : `[⊘ DISCONNECTED] · Worker novita injoignable`

---

## 2. Matrice de Sémantique

| Statut | Couleur | Glyphe | Opacité spéciale | Texte complémentaire |
|---|---|---|---|---|
| ACTIVE | accent | ● | — | Rien |
| IDLE | textMuted | ○ | — | Rien |
| THINKING | secondary | ◐ | thinkingOpacity | Points de suspension |
| RUNNING | primary | ▶ | — | Rien |
| WAITING | warning | ⌛ | — | Temps d'attente estimé |
| SUCCESS | success | ✓ | — | Durée, résultat |
| WARNING | warning | ⚠ | — | Détails de la condition |
| ERROR | error | ✗ | — | Message d'erreur |
| CANCELLED | textMuted | ⊘ | — | Raison de l'annulation |
| UNKNOWN | textMuted | ? | — | Rien (jamais de valeur fabriquée) |
| DISCONNECTED | error | ⊘ | — | Endpoint / service |

---

## 3. Règles de Combinaison

### 3.1 Règle du double signal

Chaque statut DOIT être lisible sans dépendre de la couleur seule :
- **Texte** : le libellé (ACTIVE, ERROR, etc.) est TOUJOURS présent
- **Glyphe** : le symbole visuel est TOUJOURS présent
- **Couleur** : la couleur est UN SIGNALS SUPPLÉMENTAIRE, pas le seul

### 3.2 Règle de la non-confusion

- `UNKNOWN` ne doit JAMAIS se confondre avec `ZERO` ou `IDLE`
- `CANCELLED` ne doit JAMAIS se confondre avec `IDLE` (cancelled = action interrompue, idle = pas encore commencé)
- `DISCONNECTED` est un sous-ensemble de `ERROR` mais a sa propre signalétique (perte de connexion vs erreur fonctionnelle)

### 3.3 Règle de la télémetry

| Cas | Format |
|---|---|
| Pourcentage simple | `38%` |
| Utilisation avec limite | `100K / 262K · 38%` |
| Limite inconnue | `100K · 38%` |
| Coût nul confirmé | `$0.00` |
| Coût inconnu | `N/A` |

---

## 4. Exemples Composites

### 4.1 Ligne de statut d'agent

```
[● ACTIVE]  eurinhash-agent  ·  38% · 100K / 262K · $0.00
```

### 4.2 Ligne de statut de worker

```
[◐ THINKING]  worker-novita  ·  2.3s · Attente réponse
```

### 4.3 Ligne d'erreur

```
[✗ ERROR]  worker-google  ·  Connection refused · N/A
```

### 4.4 Ligne de worker déconnecté

```
[⊘ DISCONNECTED]  worker-codestral  ·  Reconnection in 5s
```

### 4.5 Ligne de statut inconnu

```
[? UNKNOWN]  worker-pollinations  ·  Initializing
```

---

## 5. États Composites

Certains contextes combinent plusieurs statuts :

| Contexte | Affichage |
|---|---|
| Agent actif + tâche en cours | `[● ACTIVE] [▶ RUNNING] AgentName` |
| Agent actif + en attente | `[● ACTIVE] [⌛ WAITING] AgentName` |
| Agent inactif + erreur | `[○ IDLE] [✗ ERROR] AgentName` |
| Agent pensant + erreur | `[◐ THINKING] [✗ ERROR] AgentName` |

---

## 6. Vocabulaire Interdit

Les termes suivants ne sont PAS des statuts valides dans ce système :
- `NONE` → utiliser `IDLE`
- `NULL` → utiliser `UNKNOWN`
- `DONE` → utiliser `SUCCESS`
- `BUSY` → utiliser `RUNNING` ou `THINKING` selon le contexte
- `OFF` → utiliser `DISCONNECTED` ou `IDLE` selon le contexte
- `N/A` comme statut → `N/A` est réservé aux valeurs télémetry uniquement

---

## 7. Traduction et Localisation

Le vocabulaire de statut est en anglais (construit dans le code), mais peut être affiché en français via la couche de présentation :

| Statut | FR (presentation) | EN (code) |
|---|---|---|
| ACTIVE | Actif | ACTIVE |
| IDLE | Inactif | IDLE |
| THINKING | Réflexion | THINKING |
| RUNNING | Exécution | RUNNING |
| WAITING | En attente | WAITING |
| SUCCESS | Succès | SUCCESS |
| WARNING | Avertissement | WARNING |
| ERROR | Erreur | ERROR |
| CANCELLED | Annulé | CANCELLED |
| UNKNOWN | Inconnu | UNKNOWN |
| DISCONNECTED | Déconnecté | DISCONNECTED |

---

## 8. Règles de Non-Fabrication

1. **JAMAIS** afficher `0` pour UNKNOWN — utiliser `N/A` ou `?`
2. **JAMAIS** afficher une valeur inventée pour un statut non-déterminé
3. **TOUJOURS** afficher le texte du statut même si la couleur est omise
4. **JAMAIS** utiliser la couleur seule pour distinguer des statuts
5. `thinkingOpacity` s'applique UNIQUEMENT à THINKING, jamais à d'autres statuts
6. Un statut `UNKNOWN` doit rester `UNKNOWN` même après des tentatives de résolution échouées — ne pas le transformer en `IDLE` par défaut
