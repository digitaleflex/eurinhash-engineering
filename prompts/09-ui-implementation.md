# 09 — UI Implementation

## Mission
Implémenter le design validé dans `eurinhash-opencode` avec des changements petits, vérifiables et réversibles.

## Prompt
Implémente les décisions des prompts 00–08 dans `digitaleflex/eurinhash-opencode`.

Travaille par verticales fonctionnelles :
1. shell/layout ;
2. workspace ;
3. chat/session ;
4. agent panel ;
5. activity center ;
6. context/observability ;
7. modes et responsive ;
8. personnalisation.

Pour chaque verticale :
- identifier les primitives existantes ;
- créer ou modifier le minimum nécessaire ;
- brancher uniquement des données runtime autorisées ;
- conserver permissions, commandes, keybindings et flux de session ;
- ajouter tests ;
- vérifier typecheck/build/tests ;
- documenter les décisions.

Évite les gros refactors non nécessaires et tout second système de vérité.

### Livrables
- code fonctionnel ;
- tests unitaires/intégration/rendu selon les possibilités du repo ;
- `docs/course/09-ui-implementation.md` ;
- journal des changements ;
- limites connues.

## Gate
Après chaque verticale, l'application doit rester lançable et les flux fondamentaux d'OpenCode doivent continuer à fonctionner.