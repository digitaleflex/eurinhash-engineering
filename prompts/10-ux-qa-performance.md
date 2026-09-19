# 10 — UX QA & Performance

## Mission
Tester l'expérience réelle, pas seulement la compilation.

## Prompt
Audite `eurinhash-opencode` après implémentation.

Teste au minimum :
- démarrage ;
- ouverture/reprise de session ;
- streaming ;
- prompt ;
- historique ;
- changement agent/modèle ;
- tools ;
- permissions ;
- MCP ;
- LSP ;
- Git ;
- dialogs ;
- keybindings ;
- layouts et modes.

Teste plusieurs tailles de terminal et recherche : clipping, wrapping, overflow, focus perdu, scroll cassé, panneaux impossibles à fermer, textes illisibles, événements rapides, historiques volumineux.

Mesure lorsque possible : startup, first render, event-to-render, streaming responsiveness, mémoire et coût des rerenders.

Vérifie également la confidentialité : aucune credential, clé API, prompt privé, contenu sensible de fichier ou argument d'outil sensible ne doit être exposé dans logs/activity/telemetry.

### Livrables
- `docs/course/10-ux-qa-performance.md`
- test matrix ;
- defects ;
- performance findings ;
- accessibility findings ;
- regression report ;
- corrections recommandées.

## Gate
Aucune fonctionnalité UI n'est considérée terminée uniquement parce que le build passe.