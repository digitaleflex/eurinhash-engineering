# Repository Map — EurinHash OpenCode

## Vue d'ensemble

Le repo **digitaleflex/eurinhash-opencode** est une configuration/fork d'OpenCode destinée à évoluer vers **HashCode Terminal / Agent Command Center**. L'objectif est de produire un environnement de développement agentique entièrement FREE (0€), avec routage dynamique entre workers gratuits.

## Structure du repo principal (hashcode_reboot)

```
hashcode_reboot/
├── .agents/           # Configuration des agents OpenCode
├── .claude/           # Configuration Claude Code
├── .codex/            # Configuration Codex
├── .next/             # Build Next.js
├── .opencode/         # Config OpenCode locale (meilleur-compact, node_modules)
├── .vercel/           # Config Vercel
├── .vscode/           # Config VS Code
├── docs/              # Documentation ADR, plans, audit
├── node_modules/      # Dépendances npm
├── package/           # Package build
├── prisma/            # Schéma Prisma (Next.js + Prisma stack)
├── public/            # Assets statiques
├── scripts/           # Scripts utilitaires (seed, collect, copy-standalone)
├── src/               # Source Next.js (app/, components/, hooks/, lib/, middleware.ts)
├── tests/             # Tests unitaires et e2e (Playwright)
├── .env*              # Configuration secrets
├── bun.lock           # Bun lockfile
├── package.json       # Next.js 16 + React 19 + Prisma
├── next.config.ts     # Config Next.js
├── tailwind.config.ts # Config Tailwind CSS
├── tsconfig.json      # TypeScript config (ES2017, strict)
├── playwright.config.ts  # Config Playwright
├── eslint.config.mjs  # ESLint config
└── prisma/            # Prisma schema + migrations
```

## Architecture technique du repo

| Couche | Technologie | Rôle |
|--------|-------------|------|
| Framework | Next.js 16.1.1 | Application web principale |
| UI | React 19 + Tailwind CSS 4 + shadcn/ui | Interface utilisateur |
| Base de données | Prisma 6.11.1 + PostgreSQL | Persistance des données |
| Auth | Better Auth (via better-auth-best-practices skill) | Authentification |
| Cache | Upstash Redis + Ratelimit | Cache et rate limiting |
| Observabilité | Sentry + Vercel Speed Insights | Monitoring |
| Tests | Node.js test runner + Playwright | Tests unitaires et e2e |
| Build | Next.js build + copy-standalone.mjs | Build de production |

## Structure OpenCode (runtime)

```
~/.config/opencode/
├── opencode.jsonc          # Configuration canonique EURINHASH
├── free-models.json        # Statut live des workers FREE
├── agent/
│   └── eurinhash.md        # Superviseur EURINHASH (pipeline de routage)
├── plugin/
│   ├── auto-compact.ts     # Compaction intelligente automatique
│   ├── guard.ts            # Garde-fou commandes destructrices
│   └── audit-logger.ts     # Traçabilité décisions (JSONL)
├── scripts/
│   ├── route.py            # Routage EV (Espérance de Valeur)
│   ├── quota.py            # Dashboard quotas FREE
│   └── free-probe.py       # Probe workers FREE
├── .opencode/              # Config .opencode locale
└── node_modules/           # @opencode-ai/plugin, @opencode-ai/sdk
```

## Package boundaries

### Repo principal (hashcode_reboot)
- **nextjs_tailwind_shadcn_ts** v0.2.1 — Application web Next.js
- Prisma ORM pour la persistance
- Better Auth pour l'authentification
- Upstash pour Redis et rate limiting

### Runtime OpenCode (dépendances)
- `@opencode-ai/plugin` — Plugin SDK (dist/tui.d.ts, dist/index.d.ts)
- `@opencode-ai/sdk` — SDK client/serveur (dist/v2/, dist/gen/)
- `@opentui/core`, `@opentui/keymap`, `@opentui/solid` — Framework TUI

### Plugins installés (dans opencode.jsonc)
- `better-compact` — Pruning par étapes
- `opencode-mem` — Mémoire persistante
- `envsitter-guard` — Protection fichiers .env
- `oh-my-opencode-slim` — Optimisation agents
- `./plugin/guard.ts` — Garde-fou local
- `./plugin/audit-logger.ts` — Audit logger local
- `./plugin/auto-compact.ts` — Compaction auto locale
- `opencode-plugin-preload-skills` — Pré-charge skills
- `@bluelovers/opencode-arise` — Orchestrateur parallèle

## Fait vs Hypothèse

| Fait confirmé | Hypothèse |
|---------------|-----------|
| Next.js 16 + React 19 + Prisma 6 | La structure .opencode/ est un sous-projet séparé |
| Les scripts route.py/quota.py/free-probe.py sont des outils de gouvernance | better-compact est un plugin npm installé (chemin non trouvé dans node_modules local) |
| Les plugins .ts locaux sont compilés via @opencode-ai/plugin SDK | Le repo hashcode_reboot est le frontend web, pas le TUI |
| opencode.jsonc est la config principale | Les docs/recon/ seront utilisées comme base pour les prochains prompts |

## Obsolescence détectée

- `opencode-antigravity-multi-auth` : dépôt archivé par le propriétaire (2026-09-19)
- `worker-sambanova`, `worker-cerebras`, `worker-cohere` : clés 401 (retirés)
- `worker-together` : crédits épuisés
- `worker-deepseek` : pas de clé API
- `worker-zenmux` : PAYG, jamais routé auto
- `mistral/*` : "Payment Required" — retiré le 2026-09-12
- `google/gemini-2.5-flash` : rate_limited (20 req/jour) — abandonné le 2026-09-15
- `opencode-go/*` : indisponible le 2026-09-19 → bascule 100% FREE
- `muse-spark-*-contributor-free` et `nemotron-*-free` : interdits (entraînement modèle)

## Points d'entrée du TUI

Le TUI OpenCode est extrait vers `@opencode-ai/tui` (en cours de migration upstream). Les types TUI sont définis dans `@opencode-ai/plugin/dist/tui.d.ts`. Le TUI utilise `@opentui/core` (CliRenderer, KeyEvent, Renderable, SlotMode), `@opentui/keymap` (Binding, Keymap), et `@opentui/solid` (JSX, SolidPlugin).

Les routes TUI (`TuiRouteDefinition`) et les slots (`TuiSlotMap`, `TuiSlotPlugin`) constituent les points d'extension principaux du TUI.
