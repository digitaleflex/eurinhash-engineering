# Agent Lifecycle — Cycle de Vie de l'Agent

> **Source** : `@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`, `@opencode-ai/sdk/dist/gen/types.gen.d.ts`, `@opencode-ai/plugin/dist/index.d.ts`
> **Règle** : `UNKNOWN != ZERO` — chaque état est fondé sur une preuve runtime ; aucun état inventé pour l'animation.

---

## 1. Vue d'ensemble du cycle de vie

Le cycle de vie de l'agent modélise le parcours visible d'une requête depuis sa réception jusqu'à sa complétion ou son échec. Le modèle d'état fondamental est :

```
IDLE → THINKING → TOOL CALL → WAITING → RESPONDING → COMPLETED
                              ↘                    ↗
                               ERROR ←─────────────┘
                               ↓
                           CANCELLED
```

Chaque état est associé à une **preuve runtime** — un événement, un type de part, ou un changement de `SessionStatus`. Aucun état n'est purement visuel.

---

## 2. États détaillés

### 2.1 IDLE

**Sémantique** : Aucune requête en cours. L'agent est disponible mais ne produit rien.

**Preuve runtime** :
- `SessionStatus = { type: "idle" }`
- Émis via `session.idle` ou après complétion d'une requête sans action ultérieure
- `TuiState.session.status(sessionID)` retourne `{ type: "idle" }`

**Indicateur UI** : `[○ IDLE]` (glyphe `○`, couleur `textMuted`)

**Transitions valides** :
| Événement | Vers | Condition |
|-----------|------|-----------|
| Message utilisateur | THINKING | Message accepté par le runtime |
| Compaction | IDLE | Démarre et revient à idle |
| Suppression session | terminal | Session supprimée |

**Champ pertinent** : `Session.time.updated` indique le dernier timestamp d'activité.

---

### 2.2 THINKING

**Sémantique** : L'agent est en cours de raisonnement interne (chain-of-thought). Le modèle génère des tokens de raisonnement.

**Preuve runtime** :
- `session.next.reasoning.started` → `session.next.reasoning.delta*` → `session.next.reasoning.ended`
- `ReasoningPart` dans le flux de parties du message
- `SessionStatus` reste `{ type: "busy" }` (THINKING est un sous-état de busy)

**Indicateur UI** : `[◐ THINKING]` (glyphe `◐`, couleur `secondary` avec `thinkingOpacity`)

**Transitions valides** :
| Événement | Vers | Condition |
|-----------|------|-----------|
| Raisonnement terminé | TOOL CALL ou RESPONDING | `reasoning.ended` émis |
| Erreur | ERROR | `session.error` |
| Annulation | CANCELLED | `MessageAbortedError` |

**Champs pertinents** :
| Champ | Type | Signification | UNKNOWN |
|-------|------|---------------|---------|
| `reasoning.text` | `string` | Contenu du raisonnement | `undefined` si pas encore commencé |
| `reasoning.providerMetadata` | `LlmProviderMetadata \| undefined` | Métadonnées fournisseur | `undefined` |

**Règle `UNKNOWN != ZERO`** : `reasoning.text` est `undefined` avant le début — jamais `""`. `time.end` est `undefined` tant que le raisonnement n'est pas terminé.

---

### 2.3 TOOL CALL

**Sémantique** : L'agent a invoqué un outil. L'exécution est en cours ou en attente de permission.

**Preuve runtime** :
- `session.next.tool.called` — l'outil a été appelé
- `session.next.tool.input.started` → `session.next.tool.input.delta*` → `session.next.tool.input.ended`
- `ToolPart` avec `state.status: "pending"` ou `"running"`
- Si permission requise : `permission.asked` ou `permission.v2.asked`

**Indicateur UI** : `[▶ RUNNING]` ou `[⌛ WAITING]` selon que l'exécution est active ou en attente de permission

**Transitions valides** :
| Événement | Vers | Condition |
|-----------|------|-----------|
| `tool.execute.before` hook autorisé | TOOL CALL → WAITING (si permission) ou RESPONDING (si exécution) | Hook retourne allow |
| `tool.execute.before` hook refusé | TOOL CALL → IDLE | Hook retourne deny |
| Permission accordée | TOOL CALL → RESPONDING | `permission.replied` (allow) |
| Permission refusée | TOOL CALL → IDLE | `permission.replied` (deny/reject) |
| Timeout outil | ERROR | `ToolStateError` |
| Annulation | CANCELLED | `MessageAbortedError` |

**Champs pertinents** :
| Champ | Type | Signification | UNKNOWN |
|-------|------|---------------|---------|
| `ToolPart.callID` | `string` | Identifiant unique de l'appel | Jamais absent pour un ToolPart |
| `ToolPart.tool` | `string` | Nom de l'outil | Jamais absent |
| `ToolPart.state.status` | `"pending" \| "running" \| "completed" \| "error"` | État de l'outil | Toujours une valeur valide |
| `ToolPart.state.input` | `{ [key: string]: unknown }` | Arguments | Présent dans tous les sous-états |
| `ToolPart.state.title` | `string \| undefined` | Titre de l'action | `undefined` si non défini |

**Sensibilité des données** : Les arguments de l'outil peuvent contenir des données sensibles (chemins de fichiers, clés API). La représentation UI doit :
- Masquer ou tronquer les valeurs potentiellement sensibles dans `input`
- Ne jamais afficher de secrets en clair dans le terminal
- Respecter `UNKNOWN != ZERO` : `input` est toujours présent (peut être `{}`), jamais absent

---

### 2.4 WAITING

**Sémantique** : L'agent est en attente d'une entrée externe — approbation utilisateur, réponse d'un serveur MCP, authentification, ou résultat d'un outil.

**Preuve runtime** :
- `permission.asked` / `permission.v2.asked` en attente de réponse
- `McpStatusNeedsAuth` ou `McpStatusNeedsClientRegistration`
- `McpStatus` = `"needs_auth"` ou `"needs_client_registration"`
- `session.next.tool.input.started` pendant le streaming d'entrée d'outil

**Indicateur UI** : `[⌛ WAITING]` (glyphe `⌛`, couleur `warning`)

**Transitions valides** :
| Événement | Vers | Condition |
|-----------|------|-----------|
| Permission accordée (`"once"`) | RESPONDING | `permission.replied` avec `"once"` |
| Permission accordée (`"always"`) | RESPONDING | `permission.replied` avec `"always"` + règle sauvegardée |
| Permission rejetée | IDLE | `permission.replied` avec `"reject"` |
| MCP authentifié | RESPONDING | `McpStatus` → `"connected"` |
| MCP échec | ERROR | `McpStatus` → `"failed"` |
| Timeout | ERROR | Permission expirée |

**Règle `UNKNOWN != ZERO`** : `PermissionV2Asked.save` est `undefined` si aucune règle persistante — jamais `[]`. `PermissionV2Asked.source` est `undefined` si pas issu d'un outil.

---

### 2.5 RESPONDING

**Sémantique** : L'agent génère une réponse textuelle ou de raisonnement après avoir exécuté des outils ou reçu une permission.

**Preuve runtime** :
- `session.next.text.started` → `session.next.text.delta*` → `session.next.text.ended`
- `session.next.reasoning.started` → `session.next.reasoning.delta*` → `session.next.reasoning.ended`
- `TextPart` et/ou `ReasoningPart` dans le flux
- `SessionStatus = { type: "busy" }`

**Indicateur UI** : `[▶ RUNNING]` (glyphe `▶`, couleur `primary`) pour le streaming actif

**Transitions valides** :
| Événement | Vers | Condition |
|-----------|------|-----------|
| Texte terminé | COMPLETED ou TOOL CALL | `text.ended` émis, prochaine étape |
| Raisonnement terminé | TOOL CALL ou COMPLETED | `reasoning.ended` émis |
| Étape terminée | COMPLETED | `step.ended` émis |
| Erreur | ERROR | `session.error` ou `step.failed` |
| Annulation | CANCELLED | `MessageAbortedError` |

**Champs pertinents** :
| Champ | Type | Signification | UNKNOWN |
|-------|------|---------------|---------|
| `TextPart.text` | `string` | Contenu textuel (peut être partiel) | Jamais absent pour TextPart |
| `TextPart.synthetic` | `boolean \| undefined` | Texte synthétique | `undefined` si non applicable |
| `TextPart.time.end` | `number \| undefined` | Timestamp de fin | `undefined` tant que non terminé |
| `AssistantMessage.error` | `ErrorType \| undefined` | Erreur de réponse | `undefined` si pas d'erreur |
| `AssistantMessage.finish` | `string \| undefined` | Raison de fin | `undefined` si pas terminé |

**Règle `UNKNOWN != ZERO`** : `TextPart.time.end` est `undefined` jusqu'à la complétion — jamais `0`. `AssistantMessage.time.completed` est `undefined` tant que la réponse n'est pas terminée.

---

### 2.6 COMPLETED

**Sémantique** : L'agent a terminé sa requête avec succès. La réponse est complète.

**Preuve runtime** :
- `session.next.step.ended` avec `finish` reason
- `session.status` → `{ type: "idle" }`
- `session.idle` émis
- `SessionNextStepEnded` contient `cost`, `tokens`, `finish`

**Indicateur UI** : `[✓ SUCCESS]` (glyphe `✓`, couleur `success`)

**Transitions valides** :
| Événement | Vers | Condition |
|-----------|------|-----------|
| Nouvelle requête utilisateur | IDLE → THINKING | Cycle recommence |
| Compaction | IDLE → IDLE (compacting) | `session.compacted` |

**Champs pertinents** :
| Champ | Type | Signification | UNKNOWN |
|-------|------|---------------|---------|
| `StepFinishPart.cost` | `number` | Coût de la requête | `0` est une valeur valide (coût nul) |
| `StepFinishPart.tokens` | `Tokens` | Compteurs de tokens | Toujours présent même si `0` |
| `StepFinishPart.reason` | `string` | Raison de complétion | Jamais absent |
| `Session.status` | `SessionStatus` | `{ type: "idle" }` | Toujours un variant valide |

**Règle `UNKNOWN != ZERO`** : `cost: 0` signifie un coût nul confirmé (gratuit) — c'est une valeur valide. `cost: undefined` signifie "pas encore calculé".

---

### 2.7 ERROR

**Sémantique** : Une erreur s'est produite pendant le cycle de vie de l'agent. L'opération courante est interrompue.

**Preuve runtime** :
- `session.error` émis avec `error: ProviderAuthError \| UnknownError \| MessageOutputLengthError \| MessageAbortedError \| ApiError \| ...`
- `session.next.step.failed` avec `error: SessionErrorUnknown`
- `ToolStateError` avec `status: "error"`

**Indicateur UI** : `[✗ ERROR]` (glyphe `✗`, couleur `error`)

**Champs pertinents** :
| Champ | Type | Signification | UNKNOWN |
|-------|------|---------------|---------|
| `EventSessionError.error` | `ErrorType \| undefined` | Détail de l'erreur | `undefined` si pas de détail |
| `EventSessionError.sessionID` | `string \| undefined` | Session concernée | `undefined` si erreur globale |
| `ToolStateError.error` | `string` | Message d'erreur de l'outil | Présent dans `error` state |
| `ToolStateError.time.end` | `number` | Timestamp d'échec | Jamais absent en error state |

**Mapping erreur → état** :
| Type d'erreur | État après | Action de récupération |
|---------------|------------|----------------------|
| `ProviderAuthError` | IDLE + indicateur erreur | Ré-authentification |
| `MessageAbortedError` | IDLE | Re-soumission |
| `MessageOutputLengthError` | IDLE | Prompt plus court |
| `ApiError` (retryable) | RETRY | Backoff exponentiel |
| `ApiError` (non retryable) | IDLE + erreur | Vérifier le statut API |
| `ContextOverflowError` | IDLE | Compacter le contexte |
| `ToolStateError` | IDLE ou RETRY | Dépend de l'outil |
| `StructuredOutputError` | IDLE | Ajuster le format |
| `McpStatusFailed` | ERROR (contenu dans le tool) | Vérifier le serveur MCP |

**Règle `UNKNOWN != ZERO`** : `error` est `undefined` quand aucune erreur n'est présente — jamais `""`. Un `ToolStateError.error` avec `""` est un cas limite valide mais distinct de `undefined`.

---

### 2.8 CANCELLED

**Sémantique** : L'utilisateur ou le système a annulé une opération en cours. Ce n'est pas une erreur — c'est une interruption intentionnelle.

**Preuve runtime** :
- `MessageAbortedError` émis (c'est une erreur d'annulation, pas une erreur fonctionnelle)
- `session.status` → `{ type: "idle" }` après l'annulation
- Tous les outils en cours doivent être interrompus

**Indicateur UI** : `[⊘ CANCELLED]` (glyphe `⊘`, couleur `textMuted`)

**Transitions valides** :
| Événement | Vers | Condition |
|-----------|------|-----------|
| Annulation initiée | CANCELLED | `session.interrupt` command |
| Après nettoyage | IDLE | Toutes les opérations arrêtées |

**Garanties d'annulation** :
1. **Outil** : Les outils en cours sont interrompus ; `ToolStateError` émis
2. **Stream** : Les streams en cours terminent ; plus de deltas
3. **Session** : Retourne à `idle`
4. **Pas d'état partiel** : L'annulation ne laisse aucun état à moitié terminé

**Règle `UNKNOWN != ZERO`** : `MessageAbortedError` est distinct des erreurs — c'est une intention, pas un dysfonctionnement.

---

## 3. Diagramme complet des transitions

```
                    ┌─────────────────────────────────────────────────────┐
                    │                   CYCLE PRINCIPAL                    │
                    └─────────────────────────────────────────────────────┘

  ┌─────────┐    user message     ┌──────────┐    step.started    ┌───────────┐
  │  IDLE   │ ──────────────────► │ THINKING │ ─────────────────► │ TOOL CALL │
  │ {type:  │                    │ {type:   │                   │           │
  │ "idle"} │                    │ "busy"}  │                   │  WAITING  │
  └─────────┘                    └──────────┘                   │ (si perms)│
        ▲                                                           └─────┬─────┘
        │                                                                    │
        │         ┌──────────────────────────────────────────────────────────┘
        │         │
        │    tool.success    ┌───────────┐    reasoning/text    ┌───────────┐
        │    or tool.failed  │ RESPONDING│ ───────────────────► │ COMPLETED │
        │    + step.ended    │ {type:    │                      │ {type:    │
        └───────────────────►│ "busy"}   │                      │ "idle"}   │
                             └───────────┘                      └───────────┘
                                   │                                     ▲
                     step.failed   │                                     │
                                   ▼                                     │
                             ┌─────────┐                                │
                             │  ERROR  │                                │
                             │         │                                │
                             └─────────┘                                │
                                   │                                     │
                                   ▼                                     │
                             ┌───────────┐                               │
                             │CANCELLED  │ ◄────────────────────────────┘
                             │           │    user interrupt
                             └───────────┘

  ┌─────────┐    retry         ┌──────────┐
  │  RETRY  │ ─────────────────►│ THINKING │
  │ {type:  │   next reached   │ {type:   │
  │ "retry"}│                  │ "busy"}  │
  └─────────┘                  └──────────┘
```

---

## 4. Preuves runtime par état

| État | Preuve primaire | Événement | Type Part/Session | Champ discriminant |
|------|----------------|-----------|-------------------|---------------------|
| IDLE | `SessionStatus` | `session.idle` | — | `{ type: "idle" }` |
| THINKING | `ReasoningPart` | `reasoning.started/delta/ended` | `ReasoningPart` | `type: "reasoning"` |
| TOOL CALL | `ToolPart` | `tool.called` | `ToolPart` | `type: "tool"` |
| WAITING | `PermissionRequest` | `permission.asked` | — | `status: "ask"` |
| RESPONDING | `TextPart` | `text.started/delta/ended` | `TextPart` | `type: "text"` |
| COMPLETED | `StepFinishPart` | `step.ended` | `StepFinishPart` | `type: "step-finish"` |
| ERROR | `EventSessionError` | `session.error` | — | `error` présent |
| CANCELLED | `MessageAbortedError` | `session.interrupt` | — | `name: "MessageAbortedError"` |
| RETRY | `RetryPart` | `session.next.retried` | `RetryPart` | `type: "retry"` |

---

## 5. Règle `UNKNOWN != ZERO` dans le cycle de vie

| Moment | Valeur correcte | Valeur incorrecte | Pourquoi |
|--------|----------------|-------------------|----------|
| Pas de raisonnement commencé | `reasoning.text: undefined` | `reasoning.text: ""` | `""` est du texte valide |
| Pas de coût calculé | `cost: undefined` | `cost: 0` | `0` veut dire "gratuit" |
| Pas de tokens comptés | `tokens.input: 0` | — | `0` est valide pour un compteur |
| Pas d'erreur | `error: undefined` | `error: ""` | `""` est un message d'erreur valide |
| Pas de fin de stream | `time.end: undefined` | `time.end: 0` | `0` veut dire timestamp epoch |
| Pas de permission | `permission: undefined` | `permission: []` | `[]` est un ruleset valide |
| Pas de titre d'outil | `title: undefined` | `title: ""` | `""` est un titre valide |
| Pas de modèle | `model: undefined` | `model: {id: ""}` | Objet vide vs absent |

---

## 6. Multi-agent readiness

Le cycle de vie est conçu pour supporter des sous-agents sans modification de l'architecture UI :

- **Agent principal** : Affiche le cycle de vie complet avec `SessionStatus`
- **Sous-agent** : Affiche son propre cycle de vie avec `AgentPart` (type `"agent"`) dans le flux de parties
- **Exécution d'outil** : Est un sous-cycle au sein du cycle de l'agent, distingué par `ToolPart`

La distinction est fondamentale :
- **Agent** = orchestre le cycle de vie complet (IDLE → THINKING → ...)
- **Sous-agent** = cycle d'agent encapsulé dans un `AgentPart`
- **Outil** = exécution d'une fonction au sein d'une étape de l'agent

Chaque niveau a son propre état machine et sa propre preuve runtime, mais tous suivent le même pattern visuel avec les mêmes composants `StatusIndicator` et `Badge`.
