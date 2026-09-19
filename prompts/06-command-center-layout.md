# 06 — Command Center Layout

## Mission
Transformer l'IA EurinHash en layout terminal concret et responsive.

## Prompt
Conçois puis spécifie le Command Center :

```text
┌──────────────┬─────────────────────────────┬──────────────┐
│ WORKSPACE    │ CHAT / SESSION              │ AGENT        │
├──────────────┴─────────────────────────────┴──────────────┤
│ ACTIVITY / EVENTS / TOOL EXECUTION                         │
├───────────────────────────────────────────────────────────┤
│ COMMAND / INPUT                                            │
└───────────────────────────────────────────────────────────┘
```

Le chat reste le centre de gravité. Workspace, Agent et Activity enrichissent la conversation sans l'étouffer.

Définis les variantes `MINIMAL`, `STANDARD`, `DEVELOPER`, `OBSERVER`, `DEBUG`, ainsi que les seuils de largeur, collapse, overlay, wrapping, scrolling et priorité.

Réutilise les primitives TUI existantes lorsque possible. Ne crée pas de second système de layout parallèle si l'existant peut être étendu proprement.

### Livrables
- `docs/course/06-command-center-layout.md`
- wireframes textuels ;
- layout contracts ;
- responsive matrix ;
- component placement map ;
- stratégie de migration depuis le layout actuel.

## Gate
Le layout doit rester utilisable sur petit terminal et devenir diagnostique sur grand terminal sans modifier le modèle mental principal.