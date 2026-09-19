# EurinHash Engineering — Prompt Course

Suite de prompts séquentiels pour analyser, concevoir et implémenter l'expérience **UI/UX du TUI EurinHash**, principalement dans `digitaleflex/eurinhash-opencode`, avec `digitaleflex/opencode-config` comme couche de configuration.

## Principe

- **EurinHash first.** Le produit et son expérience utilisateur sont la priorité.
- OpenCode est le socle technique et la référence d'architecture, pas la roadmap produit.
- Utiliser `opencode-config` lorsque le besoin relève de la configuration.
- Modifier `eurinhash-opencode` lorsque le besoin relève de l'UI, de l'interaction, du layout ou de la présentation d'état.
- Ne pas modifier inutilement le moteur/backend.
- Ne jamais inventer de télémétrie ou de données runtime.
- `UNKNOWN != ZERO`.

## Parcours du cours

| # | Prompt | Objectif | Sortie principale |
|---|---|---|---|
| 00 | [Recon UI/UX](./00-recon-ui-ux.md) | Comprendre l'existant | Audit UI/UX |
| 01 | [Anatomie de l'interface](./01-ui-anatomy.md) | Cartographier écrans et composants | UI map |
| 02 | [Information Architecture](./02-information-architecture.md) | Organiser l'information | IA + navigation |
| 03 | [Design System](./03-design-system.md) | Définir le langage visuel | Design tokens |
| 04 | [Interaction Model](./04-interaction-model.md) | Définir les comportements | Interaction spec |
| 05 | [State & Data UX](./05-state-data-ux.md) | Relier UI et état réel | State presentation model |
| 06 | [Command Center Layout](./06-command-center-layout.md) | Concevoir le layout cible | Layout spec |
| 07 | [Agent & Activity UX](./07-agent-activity-ux.md) | Rendre l'agent lisible | Agent/activity spec |
| 08 | [Configuration vs Custom UI](./08-config-vs-custom-ui.md) | Choisir où implémenter | Decision matrix |
| 09 | [Implementation](./09-ui-implementation.md) | Implémenter progressivement | Code + tests |
| 10 | [UX QA & Performance](./10-ux-qa-performance.md) | Valider qualité et performance | QA report |
| 11 | [Polish & Product Readiness](./11-polish-product-readiness.md) | Finaliser l'expérience | Release-ready UI |

## Règle d'exécution

Ne pas sauter directement à l'implémentation. Chaque prompt doit produire des preuves réutilisables par le suivant.

```text
RECON → ANATOMY → IA → DESIGN → INTERACTION → STATE
      → LAYOUT → AGENT/ACTIVITY → DECISION → BUILD → QA → POLISH
```

## Contexte de travail

Repositories concernés :

- `digitaleflex/eurinhash-engineering` — architecture, décisions, prompts et documentation.
- `digitaleflex/eurinhash-opencode` — implémentation du TUI custom.
- `digitaleflex/opencode-config` — configuration, thème, agents, commandes et extensions configurables.

Les documents déjà présents sous `docs/recon/`, `docs/ux/`, `docs/design/`, `docs/ui/`, `docs/state/` et `docs/observability/` constituent une base de travail et doivent être vérifiés avant de recréer une information existante.
