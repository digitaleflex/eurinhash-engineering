# Permission States — États de Permission

> **Source** : `@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`, `@opencode-ai/sdk/dist/gen/types.gen.d.ts`, `@opencode-ai/plugin/dist/index.d.ts`
> **Règle** : `UNKNOWN != ZERO` — les permissions sont soit accordées, soit refusées, soit demandées. Aucun état intermédiaire fictif.

---

## 1. Vue d'ensemble

Le système de permission contrôle l'accès aux actions sensibles que l'agent peut entreprendre : exécution d'outils, écriture de fichiers, accès à des données externes, et toute opération nécessitant une approbation explicite de l'utilisateur.

Le cycle de permission suit un modèle strict : **demande → réponse → résolution**. Chaque permission a un état déterminé par la réponse de l'utilisateur.

---

## 2. Modèle de permission

### 2.1 v1 `Permission`

```typescript
type Permission = {
    id: string;
    type: string;
    pattern?: string | Array<string>;
    sessionID: string;
    messageID: string;
    callID?: string;
    title: string;
    metadata: { [key: string]: unknown };
    time: { created: number };
};
```

### 2.2 v2 `PermissionRequest`

```typescript
type PermissionRequest = {
    id: string;
    sessionID: string;
    permission: string;
    patterns: Array<string>;
    metadata: { [key: string]: unknown };
    always: Array<string>;
    tool?: { messageID: string; callID: string };
};
```

### 2.3 v2 `PermissionV2Asked`

```typescript
type PermissionV2Asked = {
    id: string;
    sessionID: string;
    action: string;
    resources: Array<string>;
    save?: Array<string>;
    metadata?: { [key: string]: unknown };
    source?: PermissionV2Source;
};
```

---

## 3. États de permission

### 3.1 Machine à états

```
asked ──reply "once"──► resolved (one-time allow)
  │                          (l'outil s'exécute une fois)
asked ──reply "always"──► resolved + rule_saved
  │                          (l'outil s'exécute ; règle sauvegardée)
asked ──reply "reject"──► rejected
  │                          (l'outil est bloqué)
asked ──timeout──► expired
```

### 3.2 Réponses de permission (v2)

| Réponse | Signification | Conséquence |
|---------|---------------|-------------|
| `"once"` | Autoriser une seule fois | L'outil s'exécute ; règle non sauvegardée |
| `"always"` | Autoriser toujours | L'outil s'exécute ; règle sauvegardée dans `always` |
| `"reject"` | Refuser | L'outil est bloqué ; `permission.rejected` émis |

### 3.3 Réponses de permission (v1)

| Réponse | Signification | Conséquence |
|---------|---------------|-------------|
| `"ask"` | Permission demandée | L'outil attend la décision |
| `"deny"` | Refus | L'outil est bloqué |
| `"allow"` | Autorisation | L'outil s'exécute |

---

## 4. État par dimension

### 4.1 Statut de la permission

| État | Valeur | Sens | UNKNOWN |
|------|--------|------|---------|
| En attente | `"ask"` | Permission demandée, awaiting user response | Jamais absent quand une permission est active |
| Accordée | `"allow"` | Permission accordée | — |
| Refusée | `"deny"` | Permission refusée | — |
| Expirée | `timeout` | Pas de réponse dans le délai | — |

**Règle** : `status` doit toujours être l'un des trois valeurs valides (`"ask"`, `"allow"`, `"deny"`). Il ne peut jamais être `UNKNOWN` — si la permission n'existe pas, le champ `permission` est simplement `undefined` sur le session.

### 4.2 Champ `always`

```typescript
always: Array<string>;  // Règles à mémoriser
```

| Valeur | Sens | UNKNOWN |
|--------|------|---------|
| `[]` | Aucune règle persistante | `[]` est une valeur valide — pas `undefined` |
| `["pattern1", ...]` | Règles à sauvegarder | Jamais `undefined` quand la réponse est `"always"` |
| `undefined` | Non applicable | `undefined` quand ce n'est pas une réponse `"always"` |

**Règle `UNKNOWN != ZERO`** : `always: []` signifie "pas de règle persistante" — c'est distinct de `always: undefined` qui signifie "la réponse n'était pas 'always'". Ne jamais confondre un tableau vide avec une absence de valeur.

### 4.3 Champ `patterns`

```typescript
patterns: Array<string>;  // Motifs correspondants
```

| Valeur | Sens | UNKNOWN |
|--------|------|---------|
| `[]` | Aucun motif spécifique | `[]` est valide |
| `["/edit", ...]` | Motifs de permission | Jamais `undefined` |

### 4.4 Champ `tool` (v2)

```typescript
tool?: { messageID: string; callID: string };
```

| Valeur | Sens | UNKNOWN |
|--------|------|---------|
| Présent | Permission originaire d'un appel d'outil | — |
| `undefined` | Permission non liée à un outil | `undefined` est correct |

### 4.5 Champ `source` (v2)

```typescript
source?: PermissionV2Source;
```

| Valeur | Sens | UNKNOWN |
|--------|------|---------|
| Valeur enum | Origine de la demande | — |
| `undefined` | Pas issu d'un outil | `undefined` est correct |

---

## 5. Hooks de permission

### 5.1 `permission.ask`

```typescript
"permission.ask"?: (
    input: Permission,
    output: { status: "ask" | "deny" | "allow" }
) => Promise<void>;
```

**Sémantique** :
- Le hook est invoqué avant l'exécution d'une action sensible
- Le hook peut déterminer le statut (`ask`, `deny`, `allow`)
- Si `deny` → l'outil est bloqué, pas d'erreur émise à la couche message
- Si `allow` → l'outil s'exécute immédiatement
- Si `ask` → l'utilisateur est sollicité

### 5.2 Evaluation des règles

```typescript
type PermissionV2Rule = {
    action: string;
    resource: string;
    effect: PermissionV2Effect;  // "allow" | "deny" | "ask"
};
```

**Ordre d'évaluation** :
1. Les règles sont évaluées dans l'ordre
2. Le premier match l'emporte
3. `deny` prend toujours le dessus sur `ask`
4. Si aucune règle ne match, le comportement par défaut est `ask`

---

## 6. Flux de permission

### 6.1 Flux v1

```
permission.asked → [permission.replied] → { status: "allow" | "deny" }
                                                  ↓
                                          tool exécuté OU bloqué
```

### 6.2 Flux v2

```
permission.v2.asked → [permission.v2.replied] → { reply: "once" | "always" | "reject" }
                                                        ↓
                                        tool exécuté (once/always) OU bloqué (reject)
                                                        ↓
                                        "always" → règle sauvegardée dans `always`
```

### 6.3 Flux avec timeout

```
permission.v2.asked → timeout → expired
                              ↓
                        Outil bloqué (équivalent à deny)
```

---

## 7. Affichage dans l'UI

### 7.1 Composant de permission

Quand une permission est demandée, l'UI affiche :

```
╭─ ⚠ PERMISSION REQUIRED ───────────────────────────╮
│                                                      │
│  Action:    edit                                      │
│  Pattern:   src/**/*.ts                               │
│  Resource:  /home/user/project/src/utils.ts           │
│                                                      │
│  [✓ Allow once]  [✓ Allow always]  [✗ Reject]       │
╰──────────────────────────────────────────────────────────╯
```

**Composants** :
| Élément | Source | Composant UI |
|---------|--------|-------------|
| Action | `PermissionV2Asked.action` | `text` |
| Ressources | `resources: Array<string>` | `List` ou `KeyValueList` |
| Règles sauvegardées | `always: Array<string>` | `Badge` vert pour chaque règle |
| Boutons de réponse | `permission.replied` | `Dialog` avec options |
| Timeout | Durée d'attente | `ProgressBar` + `⌛ WAITING` |

### 7.2 Indicateur de permission en attente

Quand une permission est en attente, le statut global de la session est :

```
[● ACTIVE] [⌛ WAITING] agent-name · Permission requise
```

**Composants** :
- `StatusIndicator` avec `[⌛ WAITING]`
- `Alert` avec les détails de la permission
- `Dialog` pour la réponse utilisateur

---

## 8. Permission et erreur

### 8.1 Permission refusée

Une permission refusée **ne génère pas d'erreur** au niveau message :

```
permission.asked → permission.replied (reject) → tool blocked
  → session continues without tool execution
```

**Différence avec une erreur** :
- Permission refusée = l'outil est bloqué, la session continue
- Erreur d'outil = l'outil a échoué, potentiellement `session.error`

### 8.2 Permission expirée

Si la permission expire (timeout), le comportement est équivalent à un refus :
- L'outil est bloqué
- La session continue
- `permission.expired` est traité comme un deny implicite

---

## 9. Règles de non-fabrication

### 9.1 `UNKNOWN != ZERO` dans les permissions

| Champ | Quand absent | Valeur correcte | Valeur incorrecte |
|-------|-------------|-----------------|-------------------|
| `Permission.pattern` | Pas de motif spécifique | `[]` ou `""` selon le type | Ne jamais `undefined` si le champ existe |
| `Permission.callID` | Pas lié à un outil | `undefined` | `""` |
| `Permission.metadata` | Toujours présent | `{}` si vide | Ne jamais `undefined` — le champ existe |
| `PermissionV2Asked.save` | Pas de règle persistante | `undefined` | `[]` |
| `PermissionV2Asked.source` | Pas issu d'un outil | `undefined` | `""` |
| `PermissionV2Reply` | Jamais absent quand demandé | `"once" \| "always" \| "reject"` | `""` |
| `always` | Pas de règle sauvegardée | `[]` (si réponse "always") | Ne jamais `undefined` si "always" |

### 9.2 Règles composites

| Situation | Affichage |
|-----------|-----------|
| Permission active, pas de réponse | `[⌛ WAITING]` + `Alert` |
| Permission accordée | `[✓ SUCCESS]` + confirmation |
| Permission refusée | `[⊘ CANCELLED]` + info que l'outil est bloqué |
| Permission expirée | `[○ IDLE]` + notification |
| Aucune permission requise | Ne pas afficher l'indicateur |

---

## 10. Multi-agent et permissions

Les permissions sont contextuelles par session :

- **Agent principal** : Les permissions sont affichées dans le contexte du flux principal
- **Sous-agent** : Les permissions d'un sous-agent sont rattachées à l'agent principal via `sessionID` et `messageID`
- **Outils MCP** : Les permissions pour les outils MCP peuvent inclure `source: PermissionV2Source` pour identifier l'origine

**Règle** : `PermissionV2Source` est `undefined` quand la permission n'est pas originaire d'un outil MCP spécifique.

---

## 11. Résumé des états de permission

| État | Glyphe | Couleur | Badge | Quand |
|------|--------|---------|-------|-------|
| En attente | `⌛` | `warning` | `[⌛ WAITING]` | `permission.asked` émis |
| Accordée | `✓` | `success` | `[✓ SUCCESS]` | `permission.replied` avec `"allow"` ou `"once"` |
| Refusée | `⊘` | `textMuted` | `[⊘ CANCELLED]` | `permission.replied` avec `"reject"` ou `"deny"` |
| Expirée | `○` | `textMuted` | `[○ IDLE]` | Timeout sans réponse |
