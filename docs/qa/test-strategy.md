# Test Strategy — Stratégie de Tests QA

> **Projet**: EurinHash OpenCode v1.18.31
> **Phase**: Phase 10 — QA, Performance & Regression
> **Date**: 2026-09-19
> **Règle**: `UNKNOWN != ZERO` — chaque cas de test vérifie le traitement des données manquantes.

---

## 1. Approche de Test

### 1.1 Pyramid Structure

| Niveau | Type | Outil | Portée |
|--------|------|-------|--------|
| **Unit** | Tests unitaires | `node --test tests/unit.test.cjs` | Fonctions, hooks, adapters |
| **Integration** | Tests d'intégration | `node --test tests/integration.test.cjs` | Flux session, event contracts |
| **E2E** | Tests end-to-end | `playwright test` | UI, terminal rendering, flows |
| **Performance** | Benchmarks | Scripts custom | Latency, throughput, memory |
| **Regression** | Comparaison | Baseline diff | Comportement avant/après |

### 1.2 Principes Fondamentaux

1. **`UNKNOWN != ZERO`** : Chaque test vérifie que les données absentes sont `undefined`, jamais `0`, `""`, `false` ou `[]` comme substitut
2. **Événement-driven** : Tous les tests de rendu vérifient que les mises à jour sont pilotées par des événements, jamais par polling
3. **État dérivé** : Les calculs (tokens, coût, pourcentage) sont vérifiés comme dérivations de `TuiState`, jamais comme valeurs stockées
4. **Terminal compatibility** : Chaque test UI est exécuté à 80, 100, 120 et 160 colonnes

### 1.3 Outils et Commandes

```bash
# Type checking
npm run typecheck          # tsc --noEmit

# Linting
npm run lint               # eslint .

# Unit tests
npm run test:unit          # node --test tests/unit.test.cjs

# Integration tests
npm run test:integration   # node --test tests/integration.test.cjs

# All tests
npm run test:all           # Tous les tests unitaires + integration

# E2E tests
npm run test:e2e           # playwright test

# Validation complète
npm run validate           # typecheck + lint + test:unit

# Profiling
npm run test:profiling     # node --test tests/profiling.test.cjs
```

---

## 2. Domaines de Test

### 2.1 Session Lifecycle

| Cas | Description | Preuve | Commande |
|-----|-------------|--------|----------|
| Session active | `SessionStatus = { type: "busy" }` | `api.state.session.status()` | `node --test tests/unit.test.cjs` |
| Session terminée | `SessionStatus = { type: "idle" }` | `session.idle` event | `node --test tests/unit.test.cjs` |
| Session erreur | `EventSessionError` émis | `api.state.session.status()` → error | `node --test tests/unit.test.cjs` |
| Session retry | `SessionStatus = { type: "retry" }` | `session.next.retried` | `node --test tests/unit.test.cjs` |

### 2.2 Context Usage

| Pourcentage | État attendu | Métrique | Source |
|-------------|-------------|----------|--------|
| 0% | `M-CTX-004 = "NORMAL"` | `contextUsed = 0` | `StepFinishPart.tokens` |
| 38% | `M-CTX-004 = "ATTENTION"` | `percentage < 50` | `derived: used / limit * 100` |
| 75% | `M-CTX-004 = "HIGH"` | `percentage >= 75` | `derived: used / limit * 100` |
| 95% | `M-CTX-004 = "CRITICAL"` | `percentage >= 90` | `derived: used / limit * 100` |
| 100%+ | `M-CTX-004 = "LIMIT"` | `percentage >= 100` | `derived: used / limit * 100` |
| Limit inconnu | `M-CTX-003 = undefined` | `limit = 0` ou absent | `Model.limit.context` |

### 2.3 MCP Integration

| MCP Status | État | Outils disponibles | Test |
|------------|------|-------------------|------|
| 0/5 connectés | `McpStatusDisabled` | Aucun | `api.state.mcp()` → vide |
| 2/5 connectés | Mixte `connected` + `disabled` + `failed` | 2 actifs | `api.state.mcp()` → filtrer |
| 5/5 connectés | Tous `McpStatusConnected` | Tous | `api.state.mcp()` → pleine liste |
| `needs_auth` | `McpStatusNeedsAuth` | Bloqués | `api.state.mcp()` → status |
| `failed` | `McpStatusFailed` | Erreur | `api.state.mcp()` → error |

### 2.4 LSP State

| État | `LspStatus.status` | Comportement | Test |
|------|-------------------|--------------|------|
| Absent | Tableau vide | Pas de LSP configuré | `api.state.lsp()` → `[]` |
| Actif | `"connected"` | Diagnostics disponibles | `api.state.lsp()` → status |
| Erreur | `"error"` | Pas de diagnostics | `api.state.lsp()` → status |

### 2.5 Coût et Tokens

| Scénario | `cost` | `tokens` | Règle |
|----------|--------|----------|-------|
| Coût nul | `0` | `{ input: 0, ... }` | `0` est VALIDE pour un compteur |
| Coût non nul | `> 0` | `{ input: N, ... }` | Valeur confirmée |
| Pas encore calculé | `undefined` | `undefined` | `UNKNOWN != ZERO` |
| Tokens en K | `tokens.input < 1000` | Comptage standard | `0` valide |
| Tokens en M | `tokens.input >= 1000000` | Comptage standard | `0` valide |

### 2.6 Terminal Width

| Largeur | Mode | Comportement attendu |
|---------|------|---------------------|
| 80 colonnes | MINIMAL | Sidebar masquée, texte seul pour progress |
| 100 colonnes | STANDARD | Layout standard, table compacte |
| 120 colonnes | OBSERVER | Métriques détaillées, table complète |
| 160 colonnes | OBSERVER/STANDARD | Layout étendu |

### 2.7 Données Manquantes

| Donnée | Comportement attendu | Règle |
|--------|---------------------|-------|
| Session inexistante | `session.get(id)` → `undefined` | Jamais d'objet factice |
| Permission absente | `session.permission(id)` → `[]` | `[]` est valide (pas `undefined`) |
| Tool en cours | `ToolPart.state.status = "pending"` | Preuve runtime |
| Tool actif | `ToolPart.state.status = "running"` | Preuve runtime |
| Tool terminé | `ToolPart.state.status = "completed"` | Preuve runtime |
| Tool en erreur | `ToolPart.state.status = "error"` | Preuve runtime |

### 2.8 Tool Call Active

| État | `ToolState` | Événement | Test |
|------|-------------|-----------|------|
| Pending | `"pending"` | `tool.execute.before` hook | Hook retourne allow/deny |
| Running | `"running"` | `tool.progress` | Progression mise à jour |
| Success | `"completed"` | `tool.success` | Résultat affiché |
| Failed | `"error"` | `tool.failed` | Erreur affichée |
| Active | `"running"` + MCP | `mcp.tools.changed` | Outils MCP dynamiques |

---

## 3. Stratégie par Phase

### 3.1 Phase 1 — Type Safety

- `npm run typecheck` → Zéro erreur de type
- Vérification que tous les champs optionnels sont `?` dans les types
- Validation `UNKNOWN != ZERO` sur tous les champs optionnels

### 3.2 Phase 2 — Lint & Format

- `npm run lint` → Zéro warning/escalade
- Vérification des conventions de nommage
- Vérification des imports

### 3.3 Phase 3 — Unit Tests

- `npm run test:unit` → Tous les tests passent
- Couverture des 8 cas de test par métrique (session, context, MCP, LSP, coût, tokens, terminal, données manquantes)
- Test de la machine à états (session, tool, MCP, LSP, permission)

### 3.4 Phase 4 — Integration Tests

- `npm run test:integration` → Tous les tests passent
- Flux complet : session → chat → tool → completion
- Tests de replay event-driven

### 3.5 Phase 5 — E2E Tests

- `npm run test:e2e` → Tous les tests playwright passent
- Tests sur 4 largeurs de terminal (80, 100, 120, 160)
- Tests de navigation, permissions, tool execution

### 3.6 Phase 6 — Performance & Regression

- Benchmarks de startup time, first render, event-to-render latency
- Comparaison comportement avant/après
- Memory growth sur sessions longues

---

## 4. Critères d'Acceptation

### 4.1 Release Gate

| Critère | Seulement quand | Preuve |
|---------|----------------|--------|
| `npm run typecheck` passe | Toujours | Sortie vide d'erreurs |
| `npm run lint` passe | Toujours | Sortie vide d'erreurs |
| `npm run test:unit` passe | Toujours | Tous les tests OK |
| `npm run test:integration` passe | Toujours | Tous les tests OK |
| `npm run test:e2e` passe | Toujours | Tous les tests OK |
| `npm run build` passe | Toujours | Build sans erreur |
| Benchmarks dans les seuils | Toujours | Latency < 16ms |
| `UNKNOWN != ZERO` vérifié | Toujours | Tous les champs optionnels testés |
| Documentation QA à jour | Toujours | Tous les fichiers `docs/qa/` présents |

### 4.2 Interdiction

- ❌ Marquer release-ready si le release gate n'est pas entièrement evidencé
- ❌ Ignorer les données manquantes sous prétexte que "c'est rare"
- ❌ Remplacer `undefined` par `0` ou `""` dans les tests
- ❌ Tester uniquement le chemin heureux (happy path)
- ❌ Omettre les tests de terminal étroit (80 colonnes)

---

## 5. Références

- `docs/architecture/architecture-overview.md` — Architecture système
- `docs/architecture/event-contracts.md` — Contrats d'événements
- `docs/architecture/state-contracts.md` — Contrats d'état
- `docs/state/state-machine.md` — Machines à états
- `docs/state/unknown-data.md` — Protocole `UNKNOWN != ZERO`
- `docs/agent/agent-lifecycle.md` — Cycle de vie de l'agent
- `docs/agent/tool-execution.md` — Exécution des outils
- `docs/observability/metrics.md` — Métriques et sources autoritatives
- `docs/observability/model.md` — Modèle d'observabilité
- `docs/design/terminal-compatibility.md` — Compatibilité terminale
- `package.json` — Scripts et configuration du projet
