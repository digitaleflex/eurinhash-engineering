# Risk Register — Registre des risques

## Vue d'ensemble

Ce registre identifie les risques techniques, opérationnels et de gouvernance pour le projet EurinHash OpenCode. Chaque risque est classé par sévérité et avec des atténuations connues.

## Risques critiques

### R1 : Dépendance aux modèles FREE avec quotas stricts

| Attribut | Valeur |
|----------|--------|
| Sévérité | CRITIQUE |
| Probabilité | ÉLEVÉE |
| Impact | Toute l'application peut devenir inopérante |

**Description** : Tous les modèles sont en FREE tier avec des quotas stricts (gemini-2.5-flash : 20 req/jour, openrouter : ~3 req/jour). Si tous les workers échouent, le système ne peut plus traiter les tâches.

**Atténuation** :
- `route.py` sélectionne automatiquement le meilleur worker
- `--next` fallback sur échec
- `quota.py` surveille les quotas
- `free-probe.py` rafraîchit le statut
- `pollinations` comme backup ultime sans clé

**Résiduel** : Moyen — le système peut fonctionner sur les workers intégrés (0 quota) en dernier recours.

### R2 : Bug Windows fsync sur better-compact

| Attribut | Valeur |
|----------|--------|
| Sévérité | HAUTE |
| Probabilité | CONFIRMÉE |
| Impact | La compaction échoue sur Windows |

**Description** : better-compact échoue avec EPERM sur Windows à cause du fsync. Le package npm a été patché pour contourner le problème.

**Atténuation** :
- Patché dans le cache npm (`~/.config/opencode/scripts/patch-better-compact.py`)
- auto-compact.ts en backup

**Résiduel** : Moyen — si le patch npm est mis à jour, le bug peut réapparaître.

### R3 : DB PostgreSQL inaccessible (P1000 auth)

| Attribut | Valeur |
|----------|--------|
| Sévérité | HAUTE |
| Probabilité | CONFIRMÉE |
| Impact | Le MCP postgres échoue, certaines features de persistence |

**Description** : Le serveur PostgreSQL échoue au boot avec P1000 auth. Le MCP postgres est désactivé (`"enabled": false`).

**Atténuation** :
- MCP postgres désactivé dans la config
- Prisma peut fonctionner avec une autre DB
- `opencode-mem` pour la mémoire persistante

**Résiduel** : Moyen — la DB n'est pas nécessaire pour le TUI, mais pour certaines features web.

### R4 : Fichiers .env exposés dans le repo

| Attribut | Valeur |
|----------|--------|
| Sévérité | HAUTE |
| Probabilité | CONFIRMÉE |
| Impact | Fuite de clés API |

**Description** : Le repo contient `.env`, `.env.development`, `.env.local`, `.env.backup-2026-09-07` dans la racine. `.gitignore` est présent mais certains fichiers existent.

**Atténuation** :
- `envsitter-guard` plugin pour la protection
- `.gitignore` pour exclure du versioning
- Les clés API OpenCode sont dans `~/.config/opencode/` (fichiers séparés)

**Résiduel** : Élevé — vérifier que `.gitignore` couvre tous les fichiers .env.

## Risques majeurs

### R5 : Migration TUI incertaine (@opencode-ai/tui)

| Attribut | Valeur |
|----------|--------|
| Sévérité | MAJEUR |
| Probabilité | MOYENNE |
| Impact | Les plugins TUI devront être mis à jour |

**Description** : Le TUI est en cours d'extraction vers `@opencode-ai/tui`. Les packages `@opentui/*` pourraient être renommés ou modifiés.

**Atténuation** :
- Les types `.d.ts` sont stables pour l'instant
- `TuiPluginApi` est le contrat stable
- Les plugins peuvent être mis à jour quand l'extraction est finale

**Résiduel** : Moyen — les plugins locaux (auto-compact, guard, audit-logger) sont des plugins serveur, pas TUI, donc moins affectés.

### R6 : Plugins externes non maintenus

| Attribut | Valeur |
|----------|--------|
| Sévérité | MAJEUR |
| Probabilité | ÉLEVÉE |
| Impact | Fonctionnalités perdues si les packages disparaissent |

**Description** : Certains plugins npm (`@bluelovers/opencode-arise`, `better-compact`, etc.) sont des packages externes qui peuvent être retirés ou mis à jour sans préavis.

**Atténuation** :
- Les plugins locaux (.ts) sont sous contrôle du projet
- `opencode.jsonc` peut être modifié pour retirer des plugins
- better-compact a un patch local

**Résiduel** : Moyen — la dépendance aux packages externes est réduite au minimum.

### R7 : Agent aurinhash.md obsolète

| Attribut | Valeur |
|----------|--------|
| Sévérité | MOYENNE |
| Probabilité | MOYENNE |
| Impact | Le superviseur peut proposer des workers retirés |

**Description** : `agent/eurinhash.md` mentionne `worker-sambanova`, `worker-cerebras`, `worker-cohere`, `worker-together`, `worker-deepseek`, `worker-zenmux` comme retirés mais aussi `worker-sambanova` dans le WORKERS dict de route.py.

**Atténuation** :
- `route.py` WORKERS dict ne contient que les workers actifs
- `quota.py` WORKERS list ne contient que les workers actifs
- Le free-models.json reflète le statut actuel

**Résiduel** : Faible — la config actuelle fonctionne mais le prompt EURINHASH doit être mis à jour.

### R8 : Performance de free-probe.py

| Attribut | Valeur |
|----------|--------|
| Sévérité | MOYENNE |
| Probabilité | ÉLEVÉE |
| Impact | Lenteur du routage |

**Description** : `free-probe.py` fait 7+ appels HTTP séquentiels avec des sleeps de 1.5s entre chaque. Le probe complet peut prendre 15-20 secondes.

**Atténuation** :
- Le probe n'est lancé que si `free-models.json` > 30min
- `route.py` peut fonctionner avec le cache
- Les workers intégrés (0 quota) ne nécessitent pas de probe

**Résiduel** : Moyen — le routage initial peut être lent.

## Risques mineurs

### R9 : Clés API stockées dans des fichiers .key

| Attribut | Valeur |
|----------|--------|
| Sévérité | FAIBLE |
| Probabilité | MOYENNE |
| Impact | Fuite si les fichiers sont lus |

**Atténuation** : Fichiers dans `~/.config/opencode/` (pas dans le repo), permissions filesystem.

### R10 : Pas de tests pour les plugins locaux

| Attribut | Valeur |
|----------|--------|
| Sévérité | FAIBLE |
| Probabilité | ÉLEVÉE |
| Impact | Bugs non détectés |

**Atténuation** : Les plugins sont simples (hooks courts), fonctionnement testé en production.

### R11 : opencode.jsonc très volumineux (461 lignes)

| Attribut | Valeur |
|----------|--------|
| Sévérité | FAIBLE |
| Probabilité | FAIBLE |
| Impact | Lecture difficile, modification risquée |

**Atténuation** : Commentaires détaillés, structure par sections.

### R12 : Dépendance au runtime Bun

| Attribut | Valeur |
|----------|--------|
| Sévérité | FAIBLE |
| Probabilité | MOYENNE |
| Impact | `BunShell` utilisé dans PluginInput |

**Description** : `PluginInput.$` est de type `BunShell`, ce qui suggère une dépendance à Bun. Si le runtime change, cela pourrait casser.

**Atténuation** : Le runtime OpenCode peut supporter plusieurs runtimes.

### R13 : Mode PRO non configuré

| Attribut | Valeur |
|----------|--------|
| Sévérité | FAIBLE |
| Probabilité | FAIBLE |
| Impact | Si le mode PRO est activé, la dépense est non contrôlée |

**Atténuation** : `quota.py` peut détecter le mode PRO et afficher le plafond. `mode.json` contrôle le mode actuel.

## Fait vs Hypothèse

| Fait confirmé | Hypothèse |
|---------------|-----------|
| Les risques R1, R2, R3, R4 sont confirmés par la config et les logs | Les impacts exacts des risques nécessitent des tests de charge |
| better-compact a un bug Windows confirmé | Le patch est suffisant pour tous les cas |
| La DB est inaccessible (P1000) | Le comportement exact quand la DB est indisponible est dans le code |
| Les workers FREE ont des quotas stricts | Les quotas exacts changent selon les providers |
| Les plugins externes peuvent être retirés | Les mainteneurs ne supprimeront pas forcément les packages |

## Matrice de risque résumée

| # | Risque | Sévérité | Probabilité | Résiduel |
|---|--------|----------|-------------|----------|
| R1 | Dépendance modèles FREE | CRITIQUE | ÉLEVÉE | Moyen |
| R2 | Bug Windows better-compact | HAUTE | CONFIRMÉE | Moyen |
| R3 | DB PostgreSQL inaccessible | HAUTE | CONFIRMÉE | Moyen |
| R4 | Fichiers .env exposés | HAUTE | CONFIRMÉE | Élevé |
| R5 | Migration TUI incertaine | MAJEUR | MOYENNE | Moyen |
| R6 | Plugins externes non maintenus | MAJEUR | ÉLEVÉE | Moyen |
| R7 | Agent eurinhash.md obsolète | MOYENNE | MOYENNE | Faible |
| R8 | Performance free-probe.py | MOYENNE | ÉLEVÉE | Moyen |
| R9 | Clés API dans fichiers .key | FAIBLE | MOYENNE | Faible |
| R10 | Pas de tests plugins | FAIBLE | ÉLEVÉE | Faible |
| R11 | Config volumineuse | FAIBLE | FAIBLE | Faible |
| R12 | Dépendance Bun | FAIBLE | MOYENNE | Faible |
| R13 | Mode PRO non configuré | FAIBLE | FAIBLE | Faible |
