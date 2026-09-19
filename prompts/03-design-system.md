# 03 — EurinHash Design System

## Mission
Définir un langage visuel terminal cohérent avant de multiplier les composants.

## Prompt
Conçois le Design System EurinHash à partir de l'IA et des contraintes du terminal.

Définis :
- palette sémantique ;
- niveaux de contraste ;
- typographie et monospace ;
- spacing ;
- borders/dividers ;
- icônes et symboles ;
- badges ;
- états ;
- progress indicators ;
- panels ;
- tables ;
- dialogs ;
- prompts ;
- empty/loading/error states.

Les couleurs doivent être sémantiques et ne jamais être l'unique moyen de transmettre un état. Les états agent doivent rester lisibles en terminal monochrome.

Définis également les règles pour :
`ACTIVE`, `THINKING`, `TOOL CALL`, `RESPONDING`, `WAITING`, `COMPLETED`, `ERROR`, `CANCELLED`.

### Livrables
- `docs/course/03-design-system.md`
- design tokens ;
- component visual rules ;
- state vocabulary ;
- terminal accessibility rules ;
- exemples de rendu textuel.

## Gate
Tout nouveau composant UI doit pouvoir réutiliser les tokens et primitives définis ici.