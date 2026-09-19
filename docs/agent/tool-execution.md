# Tool Execution — Exécution des Outils

> **Source** : `@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`, `@opencode-ai/sdk/dist/gen/types.gen.d.ts`, `@opencode-ai/plugin/dist/index.d.ts`
> **Règle** : `UNKNOWN != ZERO` — chaque champ de l'outil suit le protocole `undefined` pour absent, jamais `0`, `""` ou `false`.

---

## 1. Vue d'ensemble

L'exécution d'outil est le mécanisme par lequel l'agent interagit avec l'environnement externe — système de fichiers, commandes shell, serveurs MCP, LSP, et tout autre service. Chaque exécution traverse un cycle de vie prévisible et exposé à l'utilisateur.

### 1.1 Types d'outils

| Catégorie | Origine | Exemples |
|-----------|---------|----------|
| Outils natifs | `tool()` definition | `edit`, `write`, `bash`, `grep`, `read` |
| Outils MCP | Serveur MCP distant | Selon la configuration du serveur |
| Outils LSP | Serveur LSP | Diagnostics, formatting, code actions |
| Outils shell | `BunShell` / `pty` | Commandes terminal |

---

## 2. Cycle de vie de l'exécution d'outil

### 2.1 Machine à états

```
pending ──execute──► running ──success──► completed
  │                      │
  │                      └──failure──► error
  │
  └──cancel──► error
```

#### États détaillés

| État | `status` | Preuve runtime | Description |
|------|----------|----------------|-------------|
| **Pending** | `"pending"` | `ToolStatePending` | Outil appelé, input connu, pas encore exécuté |
| **Running** | `"running"` | `ToolStateRunning` | Outil en cours d'exécution |
| **Completed** | `"completed"` | `ToolStateCompleted` | Outil terminé avec succès |
| **Error** | `"error"` | `ToolStateError` | Outil a échoué |

### 2.2 Transitions et événements

| De | À | Événement | Condition |
|----|---|-----------|-----------|
| pending | running | `tool.execute.before` hook | Hook retourne allow |
| pending | pending | `tool.execute.before` hook | Hook retourne deny |
| running | running | `session.next.tool.progress` | Mise à jour de progression |
| running | completed | `session.next.tool.success` | Outil réussi |
| running | error | `session.next.tool.failed` | Outil échoué |
| running | error | Timeout | Temps d'exécution dépassé |
| error | running | Retry | Nouvelle tentative |
| pending | error | Annulation | `MessageAbortedError` |
| completed | — | terminal | État final |
| error | terminal (sauf retry) | — | État final |

---

## 3. Structure des données par état

### 3.1 v1 `ToolPart`

```typescript
type ToolPart = {
    id: string;
    sessionID: string;
    messageID: string;
    type: "tool";
    callID: string;
    tool: string;
    state: ToolState;
    metadata?: { [key: string]: unknown };
};
```

### 3.2 v2 `SessionMessageToolState`

**Pending** :
```typescript
type SessionMessageToolStatePending = {
    status: "pending";
    input: string;  // Raw input string
};
```

**Running** :
```typescript
type SessionMessageToolStateRunning = {
    status: "running";
    input: { [key: string]: unknown };
    structured: { [key: string]: unknown };
    content: Array<LlmToolContent>;
};
```

**Completed** :
```typescript
type SessionMessageToolStateCompleted = {
    status: "completed";
    input: { [key: string]: unknown };
    attachments?: Array<PromptFileAttachment>;
    content: Array<LlmToolContent>;
    outputPaths?: Array<string>;
    structured: { [key: string]: unknown };
    result?: unknown;
};
```

**Error** :
```typescript
type SessionMessageToolStateError = {
    status: "error";
    input: { [key: string]: unknown };
    content: Array<LlmToolContent>;
    structured: { [key: string]: unknown };
    error: SessionErrorUnknown;
    result?: unknown;
};
```

---

## 4. Hooks d'exécution

### 4.1 `tool.execute.before`

Exécuté avant l'exécution de l'outil. Permet la validation, la modification des arguments et le logging.

```typescript
"tool.execute.before"?: (
    input: { tool: string; sessionID: string; callID: string; },
    output: { args: any; }
) => Promise<void>;
```

**Sémantique** :
- Si le hook retourne sans erreur → l'outil passe à `running`
- Si le hook lance une erreur → l'outil reste `pending` ou passe à `error`
- Peut modifier `output.args` pour transformer les arguments

### 4.2 `tool.execute.after`

Exécuté après l'exécution de l'outil. Permet le post-traitement, l'audit et la validation du résultat.

```typescript
"tool.execute.after"?: (
    input: { tool: string; sessionID: string; callID: string; args: any; },
    output: { title: string; output: string; metadata: any; }
) => Promise<void>;
```

**Sémantique** :
- Toujours exécuté après la complétion ou l'erreur
- Peut enrichir `title`, `output` et `metadata`
- Ne bloque pas l'état final (la transition est déjà déterminée)

---

## 5. Présentation de l'outil dans l'UI

### 5.1 Composant `ToolCallCard`

Chaque exécution d'outil est affichée comme un `ToolCallCard` dans le flux de messages :

```
┌─ [▶ RUNNING] edit · fichier.ts ──────────────────────┐
│  Arguments :                                          │
│    path: src/utils.ts                                 │
│    old_string: "foo"                                  │
│    new_string: "bar"                                  │
│                                                       │
│  Durée : 1.2s · 100%                                  │
└─────────────────────────────────────────────────────────┘
```

**Composants constitutifs** :
| Élément | Source | Composant UI |
|---------|--------|-------------|
| Nom de l'outil | `ToolPart.tool` | `Badge` avec `[▶ RUNNING]` |
| Arguments | `ToolPart.state.input` | `KeyValueList` (avec protection données sensibles) |
| Titre | `ToolStateRunning.title` | `text` |
| Progression | `session.next.tool.progress` | `ProgressBar` |
| Durée | `ToolStateCompleted.time.start/end` | `MetricDisplay` |
| Résultat | `ToolStateCompleted.output` | `MarkdownView` ou `SyntaxView` |
| Erreur | `ToolStateError.error` | `Alert` niveau `error` |

### 5.2 Protection des données sensibles

Les arguments de l'outil peuvent contenir :
- Chemins de fichiers
- Contenu de fichiers
- Clés API ou tokens
- Données utilisateurs

**Règles d'affichage** :
1. Les chemins de fichiers sont affichés tels quels (pas de secrets)
2. Le contenu de fichiers est tronqué ou masqué si suspecté de contenir des secrets
3. Les valeurs de clés/credentials sont remplacées par `[SECRET]` (glyphe `🔒`)
4. `metadata` est `{ [key: string]: unknown } | undefined` — `undefined` quand absent, jamais `{}` comme substitut

### 5.3 ToolPart aux différents états

| État | Affichage | Glyphe | Composant |
|------|-----------|--------|-----------|
| Pending | Icône horloge + nom de l'outil | `⌛` | `Spinner` + `Badge` |
| Running | Spinning + titre + progression | `⟳` | `ProgressBar` |
| Completed | Checkmark + résultat | `✓` | `MarkdownView` |
| Error | Croix + message d'erreur | `✗` | `Alert` (niveau `error`) |

---

## 6. Progression et durée

### 6.1 Progression

La progression est communiquée via :
- `session.next.tool.progress` : événements multiples pendant l'exécution
- `structured` et `content` dans `ToolStateRunning` : données partielles
- `ProgressBar` : affichage visuel de la progression

**Règle** : Les événements de progression sont informatifs et peuvent être perdus au replay sans corruption d'état.

### 6.2 Durée

| Moment | Champ | Type | Semantics |
|--------|-------|------|-----------|
| Début | `time.start` | `number` | Timestamp unix ms |
| Fin | `time.end` | `number \| undefined` | `undefined` tant que non terminé |
| Durée | calculée | `number` | `time.end - time.start` |

**Règle `UNKNOWN != ZERO`** :
- `time.end` est `undefined` pendant l'exécution — jamais `0`
- La durée est affichée uniquement quand `time.end` est présent
- `ToolStateCompleted.compacted?: number` est `undefined` si pas compacté

---

## 7. Résultat et statut

### 7.1 Succès

**Preuve** : `session.next.tool.success` émis

**Champs du résultat** :
| Champ | Type | UNKNOWN |
|-------|------|---------|
| `result` | `unknown \| undefined` | `undefined` quand pas de résultat structuré |
| `output` | `string \| undefined` | `undefined` en erreur ; présent en `completed` |
| `outputPaths` | `Array<string> \| undefined` | `undefined` si aucun |
| `attachments` | `Array<PromptFileAttachment> \| undefined` | `undefined` si aucune |

**Affichage** : Le `output` est rendu via `MarkdownView` ou `SyntaxView` selon le type de contenu.

### 7.2 Échec

**Preuve** : `session.next.tool.failed` émis

**Champs d'erreur** :
| Champ | Type | Semantics |
|-------|------|-----------|
| `error` | `SessionErrorUnknown` | `{ type: "unknown"; message: string }` |
| `error` dans `ToolStateError` | `string` | Message d'erreur lisible |
| `result` | `unknown \| undefined` | Résultat partiel si disponible |

**Règle `UNKNOWN != ZERO`** : `error` dans `ToolStateError` est un `string` requis en état `error` — ce n'est jamais `undefined`. Mais si le message est vide, c'est `""` (valeur valide), pas `undefined`.

### 7.3 Raison d'échec

Les raisons d'échec sont catégorisées :

| Catégorie | Cause | Récupération |
|-----------|-------|--------------|
| Outil non trouvé | Définition manquante | Vérifier les définitions d'outils |
| Erreur d'exécution | Erreur runtime | Retentir (outils idempotents) |
| Timeout | Durée excessive | Augmenter le timeout ou annuler |
| Permission refusée | `permission.asked` rejeté | Résoudre la permission |
| Erreur de validation | Input invalide | Corriger l'input et retenter |
| MCP déconnecté | `McpStatusFailed` | Vérifier l'état du serveur MCP |

---

## 8. Outils MCP

### 8.1 Spécificité

Les outils MCP ont une couche supplémentaire de gestion d'état :

1. Le serveur MCP doit être `connected` (`McpStatus["status"] === "connected"`)
2. Les outils MCP sont dynamiques — listés via `EventMcpToolsChanged`
3. L'état du serveur affecte la disponibilité des outils

### 8.2 Interaction MCP → Tool

```
McpStatus.connected → outils MCP disponibles
McpStatus.disabled → outils MCP non disponibles (pas "unknown")
McpStatus.failed → outils MCP en erreur
McpStatus.needs_auth → outils MCP bloqués (pas "unknown")
```

**Règle `UNKNOWN != ZERO`** : Si `McpStatus` est `disabled`, les outils MCP ne sont simplement pas disponibles — ce n'est pas `UNKNOWN`. Si le serveur est `needs_auth`, l'accès est explicitement bloqué.

### 8.3 Affichage dans l'UI

Les outils MCP sont identifiés par leur nom de serveur :
```
[▶ RUNNING] MCP:filesystem · read-file · /home/user/project/index.ts
```

Le préfixe `MCP:` distingue les outils natifs des outils MCP.

---

## 9. Outils LSP

### 9.1 Spécificité

Les outils LSP sont invoqués par le serveur LSP et affichés comme des diagnostics :

- `EventLspClientDiagnostics` : diagnostics par chemin de fichier
- `LspStatus` : `{ status: "connected" | "error" }`

### 9.2 LSP → Tool mapping

| Événement LSP | Impact sur l'outil | Affichage |
|---------------|-------------------|-----------|
| `lsp.updated` | Outils LSP disponibles/mis à jour | `TuiSidebarLspItem` dans la sidebar |
| `lsp.client.diagnostics` | Résultats de l'analyse | `DiffView` ou indicateur dans le fichier |
| `LspStatus.error` | Outils LSP indisponibles | `[✗ ERROR]` dans la sidebar |

---

## 10. Règle `UNKNOWN != ZERO` dans les outils

| Champ | Quand absent | Valeur correcte | Valeur incorrecte |
|-------|-------------|-----------------|-------------------|
| `ToolStateRunning.title` | Pas de titre défini | `undefined` | `""` |
| `ToolStateCompleted.attachments` | Pas de pièces jointes | `undefined` | `[]` |
| `ToolStateCompleted.compacted` | Pas compacté | `undefined` | `0` |
| `ToolStateError.metadata` | Pas de métadonnées | `undefined` | `{}` |
| `ToolPart.metadata` | Pas de métadonnées | `undefined` | `{}` |
| `result` (completed) | Pas de résultat structuré | `undefined` | `null` |
| `output` (error) | Pas de sortie en erreur | Absent du type error | `""` |
| `structured` (pending) | Pas de structure | `{}` (valeur par défaut du type) | `undefined` — le champ existe |

---

## 11. Multi-agent et outils

Chaque agent (principal ou sous-agent) peut invoquer des outils de manière indépendante :

- **Agent principal** : Affiche tous ses outils dans le flux de message principal
- **Sous-agent** : Ses outils sont encapsulés dans un `AgentPart` avec `type: "agent"`
- **Exécution d'outil** : Est un sous-cycle distingué au sein du cycle de vie de l'agent

La distinction visuelle :
- Outils natifs : affichés directement dans le `ToolCallCard`
- Outils de sous-agent : affichés avec un indicateur de sous-agent
- Outils MCP : préfixés par le nom du serveur MCP
