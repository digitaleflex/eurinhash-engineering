# 01 — Anatomie de l'interface

## Mission
Transformer la reconnaissance en **carte produit de l'interface**, en séparant clairement structure, responsabilité et expérience utilisateur.

## Prompt
À partir du recon validé et du code réel, construis l'anatomie complète du TUI EurinHash.

Découpe l'expérience en : shell/application, navigation, workspace, chat/session, agent, activity, context/observability, prompt/input, tools, MCP, LSP, Git, dialogs, command palette, status, notifications, overlays et personnalisation.

Pour chaque élément documente :
- responsabilité utilisateur ;
- données et source de vérité ;
- états ;
- événements ;
- interactions clavier/souris ;
- focus et navigation ;
- contraintes de largeur ;
- fréquence d'apparition ;
- priorité informationnelle ;
- dépendances ;
- primitives existantes ;
- amélioration EurinHash proposée.

Construis également une **carte des responsabilités** : aucun composant ne doit devenir propriétaire d'un état métier qu'il ne contrôle pas.

Ne redessine pas encore. Nous cherchons d'abord à comprendre le système et le modèle mental utilisateur.

## Vision cible
L'interface doit progressivement devenir un **Command Center terminal-native** : l'utilisateur sait où il travaille, ce que fait l'agent, ce qui vient de se produire et ce qu'il peut faire ensuite, sans perdre le fil du chat.

## Livrables
- `docs/course/01-ui-anatomy.md`
- screen map ;
- component map ;
- responsibility map ;
- interaction ownership map ;
- data/source map ;
- composants à réutiliser/créer ;
- premières opportunités UX.

## Gate
Chaque élément visible du futur Command Center doit avoir une responsabilité, une source de données et un propriétaire d'interaction identifiés.