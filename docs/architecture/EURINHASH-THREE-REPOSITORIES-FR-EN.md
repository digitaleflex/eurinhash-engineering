# EURINHASH — Three-Repository Architecture

> FR/EN — v1.0 — 2026-09-19

## FR

EurinHash est réparti sur trois dépôts avec des responsabilités non interchangeables.

```text
eurinhash-engineering
       │ defines
       ▼
opencode-config
       │ orchestrates
       ▼
eurinhash-opencode
       │ implements/presents
       ▼
validated mission
```

### 1. Engineering — `eurinhash-engineering`

Autorité : vision produit, domaine, architecture, UX, contrats, règles d'ingénierie, critères d'acceptation, recherche et décisions.

### 2. Control Plane — `opencode-config`

Autorité : orchestration, agents, routing, model intelligence, budgets, permissions, workflows, événements, état, validation et synchronisation cross-repository.

### 3. Product — `eurinhash-opencode`

Autorité : expérience terminal, Command Center, chat/mission, workspace, panels, navigation, focus, interaction et intégration avec le runtime.

### Règle de séparation

Une décision produit ne doit pas vivre uniquement dans le code. Un problème de configuration ne doit pas être résolu par un hack UI. Un problème UI ne doit pas devenir une modification du runtime sans capacité manquante explicitement documentée.

### External substrates

- OpenCode : runtime d'exécution.
- models.dev : connaissance catalogue des modèles.
- OmniRoute : gateway/transport optionnel.

Aucun de ces systèmes externes ne devient la source de vérité d'EurinHash.

## EN

EurinHash is split across three repositories with non-interchangeable responsibilities.

- `eurinhash-engineering`: product/domain/architecture/UX/contracts/acceptance authority.
- `opencode-config`: orchestration/control-plane authority.
- `eurinhash-opencode`: terminal/product experience and runtime-facing integration.

External substrates such as OpenCode, models.dev and OmniRoute may be reused through explicit boundaries, but none becomes EurinHash's architectural source of truth.

### Separation rule

A product decision must not live only in code. A configuration problem must not be solved by a UI hack. A UI problem must not modify the runtime unless a documented capability gap requires it.
