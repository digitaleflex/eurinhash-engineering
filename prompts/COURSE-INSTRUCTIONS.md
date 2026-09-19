# Course Instructions — EurinHash UI/UX

## Objectif
Cette suite sert à piloter un travail réel sur l'UI/UX d'EurinHash. Les prompts sont destinés à être exécutés dans l'ordre sur le repository concerné.

## Repositories

- Documentation et méthode : `digitaleflex/eurinhash-engineering`
- TUI custom : `digitaleflex/eurinhash-opencode`
- Configuration : `digitaleflex/opencode-config`

## Avant chaque session

1. Lire le prompt demandé.
2. Lire les livrables des prompts précédents.
3. Vérifier l'état réel du repository et de la branche.
4. Vérifier les changements déjà présents avant d'en créer de nouveaux.
5. Distinguer faits, hypothèses et décisions.

## Pendant la session

Le participant doit toujours répondre à quatre questions :

1. **Quel problème UX résout-on ?**
2. **Quelle est la source de vérité ?**
3. **Dans quelle couche implémente-t-on ?**
4. **Comment vérifie-t-on que l'expérience est meilleure sans casser l'existant ?**

## Après la session

Produire :
- les décisions prises ;
- les fichiers touchés ;
- les tests exécutés ;
- les problèmes découverts ;
- les inconnues restantes ;
- le prochain gate.

## Règles absolues

- EurinHash first, upstream later.
- Configuration avant customisation lorsqu'elle suffit.
- Ne pas réimplémenter le moteur OpenCode pour une raison visuelle.
- Ne pas inventer de données runtime.
- `UNKNOWN != ZERO`.
- Ne pas exposer de secrets dans logs, activity ou telemetry.
- Préserver chat, sessions, streaming, tools, permissions, MCP, LSP, commandes et keybindings existants.
- Une fonctionnalité n'est pas terminée tant qu'elle n'a pas été vérifiée dans le terminal réel.

## Progression

```text
00 Recon
 ↓
01 Anatomy
 ↓
02 Information Architecture
 ↓
03 Design System
 ↓
04 Interaction Model
 ↓
05 State & Data UX
 ↓
06 Command Center Layout
 ↓
07 Agent & Activity UX
 ↓
08 Config vs Custom UI
 ↓
09 Implementation
 ↓
10 UX QA & Performance
 ↓
11 Polish & Product Readiness
```

## Format conseillé pour le cours

Pour chaque module :

```text
THÉORIE / CONTEXTE
        ↓
PROMPT
        ↓
TRAVAIL SUR LE REPOSITORY
        ↓
LIVRABLE
        ↓
REVUE
        ↓
GATE
        ↓
MODULE SUIVANT
```

Le cours doit rester pratique : chaque module doit produire un artefact ou une amélioration directement exploitable dans le projet EurinHash.