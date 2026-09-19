# 06 — Command Center Layout

## Mission
Transformer l'architecture UX validée en un **Command Center terminal de nouvelle génération**, utilisable au quotidien et capable de devenir un poste de contrôle complet lorsque l'utilisateur le demande.

## Prompt
Conçois puis spécifie le layout cible d'EurinHash :

```text
┌──────────────┬─────────────────────────────┬──────────────┐
│ WORKSPACE    │ CHAT / SESSION              │ AGENT        │
│              │                             │              │
│ files / git  │ conversation                │ state/model  │
│ project      │ reasoning + results         │ context/tools │
├──────────────┴─────────────────────────────┴──────────────┤
│ ACTIVITY / EVENTS / TOOL EXECUTION                         │
├───────────────────────────────────────────────────────────┤
│ COMMAND / INPUT                                            │
└───────────────────────────────────────────────────────────┘
```

Définis précisément :
- zones persistantes ;
- zones contextuelles ;
- overlays ;
- navigation ;
- focus ;
- collapse/expand ;
- scroll ;
- commandes ;
- raccourcis ;
- responsive ;
- densité ;
- transitions d'état.

Conçois les modes `MINIMAL`, `STANDARD`, `DEVELOPER`, `OBSERVER`, `DEBUG` comme **niveaux d'information**, pas comme cinq produits différents.

Le système doit savoir réduire l'interface lorsque l'espace manque et augmenter la visibilité lorsque l'espace et le besoin de diagnostic le permettent.

## Principes produit
1. Le chat est le centre de gravité.
2. Le workspace montre le contexte de travail.
3. L'agent montre ce qu'il fait et dans quel état il se trouve.
4. Activity montre ce qui s'est réellement produit.
5. Context/observability expliquent les limites et performances sans envahir l'écran.
6. Une information n'est affichée qu'une fois comme source principale.
7. Aucun chiffre sans signification.
8. Aucun état ambigu.
9. Aucun dashboard permanent obligatoire.
10. Le terminal reste rapide et calme.

## Livrables
- `docs/course/06-command-center-layout.md`
- wireframes textuels ;
- layout contracts ;
- responsive matrix ;
- mode matrix ;
- focus/navigation model ;
- component placement map ;
- stratégie de migration depuis le layout actuel.

## Gate
Le layout doit rester excellent sur petit terminal, confortable en usage quotidien et devenir un véritable poste de diagnostic sur grand terminal sans casser le modèle mental principal.