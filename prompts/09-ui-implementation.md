# 09 — UI Implementation

## Mission
Transformer les décisions UX validées en une expérience EurinHash réelle, robuste et agréable dans `digitaleflex/eurinhash-opencode`.

## Prompt
Implémente les décisions des prompts 00–08 dans `digitaleflex/eurinhash-opencode`.

Travaille par **verticales UX complètes**, pas par accumulation de composants isolés :
1. shell et layout ;
2. workspace ;
3. chat/session ;
4. agent experience ;
5. activity center ;
6. context/observability ;
7. modes et responsive ;
8. personnalisation et command palette.

Pour chaque verticale :
- partir du comportement utilisateur cible ;
- identifier les primitives existantes ;
- identifier la source de vérité runtime ;
- choisir configuration (`opencode-config`) ou custom code (`eurinhash-opencode`) ;
- implémenter le minimum architectural nécessaire ;
- préserver les flux OpenCode fondamentaux ;
- gérer explicitement loading, empty, unknown, error et success ;
- vérifier focus, clavier, souris, scrolling et terminal étroit ;
- ajouter tests adaptés ;
- typecheck/build/tests ;
- documenter le résultat et les limitations.

## Règle d'architecture
Ne pas reconstruire le moteur OpenCode dans l'UI. Utiliser ses données et événements via les frontières existantes. Si une donnée manque, documenter le manque avant de créer un mécanisme parallèle.

## Règle UX
Chaque modification doit répondre à une friction utilisateur identifiable. Un composant n'est pas une fonctionnalité : il doit améliorer un parcours, une compréhension ou une capacité d'action.

## Qualité EurinHash
L'implémentation doit produire une interface :
- lisible à distance ;
- dense sans être confuse ;
- terminal-native ;
- keyboard-first ;
- responsive ;
- stable pendant le streaming ;
- tolérante aux données inconnues ;
- sans télémétrie fictive ;
- sans fuite de secrets.

## Livrables
- code fonctionnel ;
- tests unitaires/intégration/rendu selon les possibilités du repo ;
- `docs/course/09-ui-implementation.md` ;
- journal des décisions ;
- captures/snapshots ou preuves de rendu lorsque possible ;
- limites connues.

## Gate
Après chaque verticale, l'application reste lançable et les flux fondamentaux — session, chat, streaming, modèles, agents, outils, permissions, commandes et navigation — continuent à fonctionner.