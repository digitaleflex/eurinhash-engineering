# 04 — Interaction Model

## Mission
Définir comment l'utilisateur navigue, agit, interrompt et récupère sans friction.

## Prompt
Conçois le modèle d'interaction du TUI EurinHash.

Analyse et spécifie :
- focus ;
- Tab/Shift+Tab ;
- Enter/Escape ;
- navigation clavier ;
- souris ;
- command palette ;
- raccourcis ;
- historique du prompt ;
- changement de session ;
- changement d'agent/modèle ;
- ouverture/fermeture des panneaux ;
- interruptions d'agent ;
- actions longues ;
- confirmations dangereuses ;
- erreurs et récupération ;
- notifications.

Le modèle doit coexister avec les commandes et keybindings existants d'OpenCode. Ne remplace pas un mécanisme fonctionnel sans nécessité.

### Livrables
- `docs/course/04-interaction-model.md`
- interaction state machine ;
- keymap ;
- focus model ;
- command palette rules ;
- interruption/recovery flows ;
- accessibility checklist.

## Gate
Toutes les actions critiques doivent avoir un chemin clavier clair et un état de récupération défini.