# 01 — Anatomie de l'interface

## Mission
Transformer le résultat du recon en carte précise de l'interface.

## Prompt
À partir de `00-ui-ux-recon.md` et du code réel, construis l'anatomie complète du TUI EurinHash.

Découpe l'interface en :
- shell/application ;
- navigation ;
- workspace ;
- chat/session ;
- agent ;
- activity ;
- prompt/input ;
- dialogs ;
- status ;
- notifications ;
- overlays ;
- configuration/personalization.

Pour chaque composant, documente responsabilité, données d'entrée, état, événements, interactions clavier/souris, contraintes de largeur et relation avec les autres composants.

Ne redessine pas encore l'interface. L'objectif est de comprendre **qui affiche quoi, pourquoi et à partir de quelle source**.

### Livrables
- `docs/course/01-ui-anatomy.md`
- component map ;
- screen map ;
- interaction ownership map ;
- dépendances entre composants ;
- composants à réutiliser ;
- composants à créer uniquement si nécessaire.

## Gate
Chaque élément visible du futur Command Center doit avoir un emplacement architectural identifié.