# 07 — Agent & Activity UX

## Mission
Faire comprendre à l'utilisateur ce que l'agent fait, sans exposer du bruit inutile.

## Prompt
Conçois l'expérience Agent et Activity du Command Center.

Définis les représentations pour :
- agent actif ;
- phase de raisonnement visible ;
- tool call ;
- tool result ;
- réponse ;
- attente ;
- erreur ;
- annulation ;
- session idle ;
- MCP ;
- LSP ;
- Git ;
- événements système.

Construis un Activity Center chronologique avec filtres quand les données existent : `ALL`, `AGENT`, `TOOLS`, `MCP`, `LSP`, `GIT`, `ERRORS`.

Chaque entrée doit expliquer au minimum : heure, catégorie, action, résultat ou état. Évite la duplication entre Activity, Chat et Status.

### Livrables
- `docs/course/07-agent-activity-ux.md`
- agent state presentation ;
- activity event taxonomy ;
- filtering model ;
- verbosity rules par mode ;
- exemples de flux complets.

## Gate
Un utilisateur doit pouvoir distinguer clairement état de session, état de requête et état d'outil.