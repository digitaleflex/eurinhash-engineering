# 02 — Information Architecture

## Mission
Concevoir une hiérarchie d'information qui garde le chat central tout en rendant le contexte de travail immédiatement lisible.

## Prompt
À partir des prompts 00–01 et des docs UX existantes, conçois l'Information Architecture du Command Center EurinHash.

Le modèle cible doit organiser :
- Workspace ;
- Chat/Session ;
- Agent ;
- Activity ;
- Context ;
- Tools ;
- MCP ;
- LSP ;
- Git ;
- Model/Provider ;
- Session/history.

Définis pour chaque information : priorité, fréquence de consultation, permanence, niveau de détail, emplacement, mode d'accès et comportement quand l'espace manque.

Conserve cinq modes : `MINIMAL`, `STANDARD`, `DEVELOPER`, `OBSERVER`, `DEBUG`.

Définis aussi les règles de responsive terminal : quoi disparaît, quoi se compacte, quoi devient overlay et quoi reste toujours visible.

### Livrables
- `docs/course/02-information-architecture.md`
- hiérarchie de l'information ;
- navigation model ;
- mode matrix ;
- responsive behavior ;
- règles de priorité et de densité.

## Gate
Aucune nouvelle information ne doit apparaître sans propriétaire UX et sans justification de sa priorité.