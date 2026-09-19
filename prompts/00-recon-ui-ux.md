# 00 — Reconnaissance produit UI/UX

## Mission
Comprendre l'expérience terminal actuelle avant toute modification et établir la baseline EurinHash.

## Prompt
Tu es **Principal UI/UX Engineer, Terminal UX Architect et Product Investigator**. Audite le TUI réel de `digitaleflex/eurinhash-opencode` et la couche `digitaleflex/opencode-config`.

Travaille comme un enquêteur produit : inspecte le code réel, exécute l'application si possible, observe les écrans et parcours, puis confronte tes observations aux documents d'ingénierie existants.

Cartographie au minimum : shell, home, session, chat, message list, prompt, sidebar, workspace, agent/status, activity, tools, MCP, LSP, Git, model/provider, dialogs, command palette, notifications, overlays, thèmes, keybindings, scrolling, focus et responsive terminal.

Pour chaque zone, établis :
1. comportement actuel ;
2. source de vérité runtime ;
3. événements qui l'alimentent ;
4. données réellement disponibles ;
5. configuration possible via `opencode-config` ;
6. modification nécessaire dans `eurinhash-opencode` ;
7. friction utilisateur ;
8. opportunité EurinHash ;
9. risque technique ;
10. test permettant de prouver le comportement.

Compare avec `docs/recon/`, `docs/ux/`, `docs/design/`, `docs/ui/`, `docs/state/` et `docs/observability/`. Toute divergence doit être signalée.

## Vision EurinHash
L'objectif est de construire un **terminal de commande et de contrôle pour agents de développement** : puissant, lisible, rapide, terminal-native et centré sur le travail.

Le chat reste le centre de gravité. Workspace, Agent, Activity, Context, Tools et Observability doivent augmenter la compréhension sans transformer le terminal en dashboard surchargé.

## Contraintes
- Aucun code dans ce module.
- Aucun comportement inventé.
- `UNKNOWN != ZERO`.
- Ne jamais exposer de secrets, prompts privés, sorties sensibles ou credentials dans les artefacts.
- Ne pas chercher à préparer une contribution upstream.

## Livrables
- `docs/course/00-ui-ux-recon.md`
- baseline UX actuelle ;
- inventaire écrans/composants ;
- matrice configuration vs custom UI ;
- matrice données/runtime ;
- frictions UX ;
- opportunités classées par impact et risque ;
- questions ouvertes ;
- `RECON UI/UX STATUS`.

## Gate
Aucune conception ne démarre tant que nous ne savons pas précisément ce que l'utilisateur voit aujourd'hui, ce qui l'alimente et où se situe chaque possibilité d'amélioration.