# 00 — Recon UI/UX

## Mission
Auditer l'interface actuelle d'OpenCode utilisée comme base d'EurinHash avant toute modification.

## Prompt
Tu es **Lead UI/UX Engineer + Terminal UX Architect**. Analyse `digitaleflex/eurinhash-opencode` et, si nécessaire, `digitaleflex/opencode-config`.

Ne code rien. Cartographie l'expérience réelle : écrans, routes TUI, composants, layouts, sidebar, prompt, dialogs, status, thèmes, keybindings, plugins/slots, états, événements, données consommées, persistance et contraintes terminal.

Pour chaque zone, identifie :
1. ce qui existe réellement ;
2. la source de vérité runtime ;
3. ce qui est configurable dans `opencode-config` ;
4. ce qui nécessite une modification de `eurinhash-opencode` ;
5. les frictions UX ;
6. les dépendances et risques.

Compare les observations avec les documents existants dans `docs/recon/`, `docs/ux/`, `docs/design/`, `docs/ui/`, `docs/state/` et `docs/observability/`. Signale toute divergence au lieu de la corriger silencieusement.

### Contraintes
- Aucun code.
- Aucun comportement inventé.
- Ne pas confondre état UI et état backend.
- `UNKNOWN != ZERO`.
- Ne pas proposer d'upstream PR.

### Livrables
- `docs/course/00-ui-ux-recon.md`
- inventaire des écrans et composants ;
- matrice config vs custom code ;
- liste des frictions UX ;
- questions ouvertes ;
- `RECON UI/UX STATUS` avec faits, hypothèses et inconnues.

## Gate
Impossible de passer au prompt 01 tant que l'interface actuelle n'est pas suffisamment cartographiée.