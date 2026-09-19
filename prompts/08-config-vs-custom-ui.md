# 08 — Configuration vs Custom UI

## Mission
Décider systématiquement où implémenter chaque amélioration.

## Prompt
Pour chaque exigence UX issue des prompts précédents, décide entre :

1. `opencode-config` ;
2. plugin/extension TUI ;
3. `eurinhash-opencode` — UI/presentation ;
4. adapter/selector de données ;
5. backend/SDK uniquement si aucune couche de présentation ne suffit.

Utilise cette grille :

| Question | Décision |
|---|---|
| Peut-on le faire proprement par configuration ? | config |
| Est-ce une extension isolée d'un slot existant ? | plugin |
| Est-ce un nouveau composant/layout/comportement UX ? | custom TUI |
| Manque-t-il une donnée fiable ? | adapter/API |
| Le moteur doit-il réellement changer ? | dernier recours |

Pour chaque décision, documente justification, fichier cible, risques, tests et rollback.

Ne optimise pas pour une future contribution upstream. L'objectif est la qualité du produit EurinHash.

### Livrables
- `docs/course/08-config-vs-custom-ui.md`
- decision matrix ;
- implementation boundaries ;
- dependency rules ;
- backlog priorisé des changements.

## Gate
Aucune modification de code ne doit être commencée sans savoir pourquoi elle appartient à cette couche.