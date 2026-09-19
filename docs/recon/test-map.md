# Test Map — Tests et Vérification

## Vue d'ensemble

Le projet hashcode_reboot est une application Next.js avec une suite de tests couvrant les aspects applicatifs (auth, events, profiling, matching, workshop). Le runtime OpenCode lui-même est un outil CLI/TUI dont les tests ne sont pas dans ce repo mais dans le repo upstream @opencode-ai/opencode.

## Tests du repo hashcode_reboot

### Tests unitaires et fonctionnels (Node.js test runner)

Exécutés via `npm run test` ou `npm run test:all` :

| Test | Fichier | Rôle |
|------|---------|------|
| **unit** | `tests/unit.test.cjs` | Tests unitaires généraux |
| **magic-link** | `tests/magic-link.test.cjs` | Test des liens magiques (auth) |
| **event-validation** | `tests/event-validation.test.cjs` | Validation des événements |
| **events-gate** | `tests/events-gate.test.cjs` | Gates de contrôle des événements |
| **profiling** | `tests/profiling.test.cjs` | Tests de performance |
| **account-rgpd** | `tests/account-rgpd.test.cjs` | Tests conformité RGPD (suppression compte) |
| **matching** | `tests/matching.test.cjs` | Tests de matching ateliers |
| **workshop-validation** | `tests/workshop-validation.test.cjs` | Validation des ateliers |
| **workshop-progression** | `tests/workshop-progression.test.cjs` | Progression des ateliers |
| **workshop-quiz** | `tests/workshop-quiz.test.cjs` | Quiz des ateliers |
| **workshop-emails** | `tests/workshop-emails.test.cjs` | Emails des ateliers |
| **integration** | `tests/integration.test.cjs` | Tests d'intégration |

### Commandes de test

```bash
npm test              # Tous les tests unitaires
npm run test:unit     # Alias pour test
npm run test:magic-link
npm run test:events
npm run test:profiling
npm run test:integration
npm run test:all      # Tous les tests (unit + integration)
npm run test:e2e      # Playwright e2e
```

### Configuration Playwright (e2e)

`playwright.config.ts` — Configuration des tests e2e

## Tests du runtime OpenCode

### Tests @opencode-ai/sdk

Les tests du SDK OpenCode se trouvent dans le dépôt upstream `@opencode-ai/opencode`. Le package `@opencode-ai/sdk` génère des types TypeScript depuis une spécification OpenAPI dans `dist/gen/`.

### Tests @opencode-ai/plugin

Le package `@opencode-ai/plugin` contient des exemples dans `dist/example.d.ts` et `dist/example-workspace.d.ts`. Le code source des tests n'est pas disponible localement.

## Scripts de vérification

### Scripts de gouvernance EURINHASH

| Script | Usage | Type |
|--------|-------|------|
| `route.py` | Routage EV des workers | Python 3 |
| `quota.py` | Dashboard quotas FREE | Python 3 |
| `free-probe.py` | Probe des workers FREE | Python 3 |

### Scripts du repo hashcode_reboot

| Script | Usage |
|--------|-------|
| `scripts/collect-email-metrics.ts` | Collecte métriques email |
| `scripts/seed-email-templates.ts` | Seed templates email |
| `scripts/seed-github-program.ts` | Seed programme GitHub |
| `scripts/seed-github-workshop.ts` | Seed atelier GitHub |
| `scripts/copy-standalone.mjs` | Copy standalone build |
| `scripts/import-blacklist-from-soft-deleted.mjs` | Import blacklist |
| `scripts/import-direct.mjs` | Import direct |
| `scripts/import-old-members.mjs` | Import anciens membres |
| `scripts/test-brevo-api.js` | Test API Brevo |
| `scripts/test-collector.ts` | Test collector |
| `scripts/test-email-services.mjs` | Test services email |

## Vérification de type et lint

### hashcode_reboot

```bash
npm run typecheck   # tsc --noEmit
npm run lint        # eslint .
npm run validate    # typecheck + lint + test:unit
```

### Configuration

- **TypeScript** : strict mode, ES2017 target, React JSX
- **ESLint** : `eslint.config.mjs` (ESLint 9 flat config)
- **Tailwind** : `@tailwindcss/postcss` (v4)
- **Prisma** : `npm run db:generate` / `npm run db:migrate`

## Tests manquants / non couverts

### TUI (non testé dans ce repo)
- Aucun test unitaire pour les composants TUI (@opentui)
- Aucun test d'intégration pour le TUI
- Pas de test de performance pour le rendu TUI
- Pas de test d'accessibilité terminal

### Plugins locaux (non testés)
- `auto-compact.ts` : pas de test
- `guard.ts` : pas de test
- `audit-logger.ts` : pas de test

### Route.py (non testé)
- Pas de tests unitaires pour route.py
- Pas de tests pour quota.py
- Pas de tests pour free-probe.py

### Points de test critiques pour la migration TUI
- TuiPluginApi — pas de test
- TuiEventBus — pas de test
- TuiState synchronisation — pas de test
- TuiSlot system — pas de test

## Fait vs Hypothèse

| Fait confirmé | Hypothèse |
|---------------|-----------|
| Les tests hashcode_reboot utilisent Node.js test runner | Les tests e2e Playwright existent mais les spécifications ne sont pas dans le repo |
| La commande `npm run test:all` combine unit + integration | Les tests sont en .cjs (CommonJS) pour compatibilité |
| Les plugins locaux n'ont pas de tests | Les tests du SDK upstream sont dans le repo @opencode-ai/opencode |
| Les scripts Python de gouvernance n'ont pas de tests | La couverture est assurée par le fonctionnement quotidien |
| Les scripts hashcode_reboot sont pour les emails/workshops | Les scripts ne sont pas directement liés au runtime OpenCode |

## Couverture de test estimée

| Domaine | Couverture |
|---------|-----------|
| Next.js app (auth, workshops) | Bonne (tests unit + e2e) |
| Event system (hashcode) | Moyenne (event-validation + events-gate) |
| Runtime OpenCode TUI | Nulle (pas dans ce repo) |
| Plugins locaux | Nulle |
| Scripts Python de gouvernance | Nulle |
| SDK @opencode-ai | Inconnue (upstream) |
