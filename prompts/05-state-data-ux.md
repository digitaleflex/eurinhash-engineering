# 05 — State & Data UX

## Mission
Relier chaque élément visible à une source de vérité réelle sans dupliquer le moteur OpenCode.

## Prompt
À partir des reconstructions précédentes, construis le modèle de présentation d'état pour EurinHash.

Sépare explicitement :
- SESSION STATE ;
- REQUEST STATE ;
- MODEL STATE ;
- TOOL STATE ;
- MCP SERVER STATE ;
- MCP MODEL ACCESS STATE ;
- LSP STATE ;
- STREAM STATE ;
- UI STATE ;
- PREFERENCE STATE.

Pour chaque métrique ou état, indique source, fraîcheur, transformation UI, fallback et condition d'inconnu.

Règle absolue : `UNKNOWN != ZERO`.

Exemples : coût connu nul → `$0.00`; coût inconnu → `N/A`. Limite de contexte connue → `100K / 262K · 38%`; limite inconnue → ne pas l'inventer.

Aucune clé API, credential, prompt utilisateur, sortie sensible d'outil ou secret ne doit devenir une donnée d'observabilité.

### Livrables
- `docs/course/05-state-data-ux.md`
- state ownership matrix ;
- telemetry presentation rules ;
- unknown/degraded states ;
- event-to-UI mapping ;
- privacy rules.

## Gate
Chaque donnée affichée doit avoir une source identifiable et un comportement documenté lorsqu'elle manque.