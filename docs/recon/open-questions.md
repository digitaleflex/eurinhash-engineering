# Open Questions — Questions ouvertes et inconnues

## Vue d'ensemble

Cette document liste les questions qui restent sans réponse après la reconnaissance. Elles sont classées par domaine et nécessitent une investigation supplémentaire.

## TUI et Frontend

### Q1 : Architecture exacte du TUI

**Question** : Comment le TUI est-il rendu exactement ? Le framework @opentui/core est-il basé sur Ink (React), SolidJS, ou un moteur propriétaire ?

**Pourquoi c'est important** : Cela détermine comment les plugins TUI peuvent interagir avec le rendu.

**Pour investiguer** : Lire le source de `@opentui/core` et `@opentui/solid`. Vérifier si `@opentui/solid` utilise SolidJS ou si c'est un wrapper.

### Q2 : Mécanisme de communication TUI ↔ Serveur

**Question** : Comment le TUI communique-t-il avec le serveur ? WebSocket, SSE, ou HTTP polling ?

**Pour investiguer** : Lire `@opencode-ai/sdk/dist/v2/server.d.ts` et `@opencode-ai/sdk/dist/v2/client.d.ts` pour le transport exact.

### Q3 : Emplacement des sources TUI

**Question** : Le code source du TUI est-il dans le repo `@opencode-ai/opencode` ou dans `@opencode-ai/tui` ? La migration vers `@opencode-ai/tui` est-elle terminée ?

**Pour investiguer** : Vérifier les repos GitHub @opencode-ai/opencode et @opencode-ai/tui.

### Q4 : Configuration tui.json / tui.jsonc

**Question** : Existe-t-il un fichier `tui.json` ou `tui.jsonc` séparé ? Les types `TuiConfigView` mentionnent des champs `tui`, mais la config principale est `opencode.jsonc`.

**Pour investiguer** : Chercher dans `~/.config/opencode/` et `~/.config/` pour des fichiers tui.json.

## Plugin et SDK

### Q5 : better-compact — code source et patch

**Question** : Le package `better-compact` est-il installé où ? Le patch est-il dans le cache npm ou dans un dépôt séparé ?

**Pour investiguer** : Chercher `better-compact` dans tous les node_modules globaux. Vérifier `~/.config/opencode/scripts/patch-better-compact.py`.

### Q6 : Plugins npm — code source

**Question** : Les packages `@bluelovers/opencode-arise`, `oh-my-opencode-slim`, `opencode-mem`, `envsitter-guard`, `opencode-plugin-preload-skills` sont-ils disponibles en source ?

**Pour investiguer** : Chercher les repos GitHub correspondants. Vérifier `~/.config/opencode/node_modules/`.

### Q7 : PluginManager interne

**Question** : Comment le runtime OpenCode charge-t-il les plugins ? Le mécanisme de résolution des chemins relatifs (comme `./plugin/guard.ts`) est-il documenté ?

**Pour investiguer** : Lire le source du serveur OpenCode pour le plugin loading.

### Q8 : V2 effect system — maturité

**Question** : Le système `v2/effect/` est-il utilisé en production ou est-ce un namespace en cours de développement ?

**Pour investiguer** : Vérifier si `v2/effect/` est exporté dans les index principaux ou si c'est interne.

## Runtime et Gouvernance

### Q9 : Communication TUI ↔ route.py

**Question** : Comment le TUI appelle-t-il `route.py` ? Via subprocess, IPC, ou le TUI exécute les commandes lui-même ?

**Pour investiguer** : Chercher dans le code source du TUI les appels à `python route.py`.

### Q10 : Mode PRO — transition FREE → PRO

**Question** : Comment le système gère-t-il la transition du mode FREE au mode PRO ? Le fichier `mode.json` est-il créé automatiquement ?

**Pour investiguer** : Lire le source pour la gestion du mode. Vérifier si `mode.json` existe déjà.

### Q11 : Prisma DB — état de la base

**Question** : La DB Prisma est-elle configurée pour PostgreSQL local ou remote ? Pourquoi l'auth P1000 échoue-t-elle ?

**Pour investiguer** : Lire `prisma/schema.prisma` et vérifier la configuration DB.

### Q12 : Persistence des sessions

**Question** : Les sessions sont-elles persistées sur le disque ou en mémoire uniquement ? Que se passe-t-il si le serveur redémarre ?

**Pour investiguer** : Chercher dans le code source du serveur la persistence des sessions.

## Sécurité et Confidentialité

### Q13 : redactDeep dans guard.ts

**Question** : `redactDeep` importé de `../src/core/secret-redactor` — ce module existe-t-il dans le repo hashcode_reboot ou est-il externe ?

**Pour investiguer** : Chercher `src/core/secret-redactor` dans le repo ou dans les node_modules.

### Q14 : Fichiers .env dans le repo

**Question** : Les fichiers `.env`, `.env.backup-2026-09-07`, `.env.development`, `.env.local` sont-ils vraiment dans la racine du repo et pas dans `.gitignore` ?

**Pour investiguer** : Vérifier `.gitignore` et le statut git.

### Q15 : Audit logger — `agent.invoked` hook

**Question** : `agent.invoked` est-il un hook réel ou un hook fictif ? Le commentateur dit "not in current SDK version" — quand sera-t-il supporté ?

**Pour investiguer** : Vérifier le SDK upstream pour la présence de `agent.invoked` ou `agent.*` hooks.

## Tests et Qualité

### Q16 : Tests du SDK OpenCode

**Question** : Où sont les tests du package @opencode-ai/sdk et @opencode-ai/plugin ?

**Pour investiguer** : Chercher dans le repo @opencode-ai/opencode.

### Q17 : Tests de performance du TUI

**Question** : Existe-t-il des benchmarks pour le rendu TUI ? Des tests de performance pour le streaming des messages ?

**Pour investiguer** : Chercher des benchmarks dans le repo upstream.

### Q18 : Accessibilité terminal

**Question** : Le TUI est-il compatible avec les lecteurs d'écran et les terminaux alternatifs ?

**Pour investiguer** : Chercher des tests d'accessibilité dans le code source.

## Migration et Upstream

### Q19 : Statut de la migration @opencode-ai/tui

**Question** : L'extraction du TUI vers `@opencode-ai/tui` est-elle terminée ? Les packages @opentui/* sont-ils toujours nécessaires ?

**Pour investiguer** : Vérifier le repo @opencode-ai/tui et le statut des issues upstream.

### Q20 : SDK v2 — API stable ou instable

**Question** : Le SDK v2 (`@opencode-ai/sdk/dist/v2/`) est-il l'API recommandée ou le v1 (`@opencode-ai/sdk/dist/`) est-il encore utilisé ?

**Pour investiguer** : Vérifier quel SDK est importé dans le code actuel.

### Q21 : opencode.jsonc — format et validation

**Question** : Le format `.jsonc` est-il supporté nativement par le runtime ou y a-t-il un parser custom ?

**Pour investiguer** : Chercher le parser de configuration dans le source.

### Q22 : better-compact — alternative locale

**Question** : auto-compact.ts peut-il remplacer entièrement better-compact ou sont-ils complémentaires ?

**Pour investiguer** : Comparer les fonctionnalités des deux plugins.

## Priorité des questions

| Priorité | Questions | Impact |
|----------|-----------|--------|
| **P0** | Q1, Q2, Q5, Q7, Q13 | Dépendance critique pour le fonctionnement |
| **P1** | Q3, Q6, Q9, Q19, Q20 | Migration et maintenance |
| **P2** | Q4, Q10, Q11, Q12, Q22 | Configuration et persistance |
| **P3** | Q14, Q15, Q16, Q17, Q18 | Sécurité et qualité |
| **P4** | Q21 | Format et validation |

## Fait vs Hypothèse

| Fait confirmé | Hypothèse |
|---------------|-----------|
| Les questions Q5-Q15 sont des zones d'ombre légitimes | Les réponses à Q1-Q4 et Q19-Q21 nécessitent l'accès aux repos upstream |
| Les plugins externes n'ont pas leur source localement | Les packages npm peuvent être inspectés via npm registry |
| Le SDK v2 existe et est utilisé | Le v1 pourrait encore être la version de production |
| La migration TUI est en cours | L'extraction pourrait être plus avancée que ce que les types suggèrent |

