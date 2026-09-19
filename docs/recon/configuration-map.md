# Configuration Map — Surface de configuration

## Vue d'ensemble

La configuration OpenCode est gérée via `opencode.jsonc` (JSON avec commentaires) situé à `~/.config/opencode/opencode.jsonc`. La configuration est chargée au démarrage et peut être mise à jour via l'API `Config` (hook `config` dans les plugins).

## Fichier de configuration principal

**Chemin** : `~/.config/opencode/opencode.jsonc`

**Structure** : see `docs/recon/configuration-map.md` for full schema (extracted from opencode.jsonc)

### Sections de la config opencode.jsonc

| Section | Type | Rôle |
|---------|------|------|
| `$schema` | string | Référence au schéma JSON |
| `shell` | string | Shell par défaut ("bash") |
| `instructions` | string[] | Instructions morph tools |
| `plugin` | string[] | Liste des plugins |
| `permission` | object | Permissions globales (edit, read, bash, etc.) |
| `default_agent` | string | Agent par défaut ("eurinhash") |
| `model` | string | Modèle par défaut ("opencode/deepseek-v4-flash-free") |
| `small_model` | string | Petit modèle ("opencode/mimo-v2.5-free") |
| `subagent_depth` | number | Profondeur des sous-agents (3) |
| `compaction` | object | Configuration de la compaction |
| `tool_output` | object | Limites de sortie d'outils |
| `experimental` | object | Features expérimentales |
| `formatter` | false | Formatter désactivé |
| `lsp` | true | LSP activé |
| `agent` | object | Configuration des agents |
| `mcp` | object | Configuration des serveurs MCP |
| `provider` | object | Configuration des providers |

### Compaction config

```json
"compaction": {
    "auto": true,
    "prune": true,
    "tail_turns": 10,
    "preserve_recent_tokens": 30000,
    "reserved": 20000
}
```

### Permission config (precedence: last rule wins)

```json
"permission": {
    "skill": "allow", "read": "allow", "edit": "allow",
    "glob": "allow", "grep": "allow", "list": "allow", "todowrite": "allow",
    "bash": { "*": "allow", "rm *": "ask", "rm -rf *": "deny", ... }
}
```

### Agent config

```json
"agent": {
    "explorer": { "model": "opencode/mimo-v2.5-free" },
    "librarian": { "model": "opencode/mimo-v2.5-free" },
    "fixer": { "model": "openrouter/poolside/laguna-s-2.1:free" },
    "councillor": { "model": "openrouter/poolside/laguna-s-2.1:free" },
    "designer": { "model": "novita/inclusionai/ling-3.0-flash-sante" },
    "oracle": { "model": "zhipu/glm-4.7-flash" }
}
```

## Fichiers de configuration additionnels

| Fichier | Chemin | Rôle |
|---------|--------|------|
| free-models.json | `~/.config/opencode/free-models.json` | Statut live des workers + latences |
| route-usage.json | `~/.config/opencode/route-usage.json` | Historique succès/échec (feedback Bayes) |
| mode.json | `~/.config/opencode/mode.json` | Mode FREE/PRO |
| .openrouter-key | `~/.config/opencode/.openrouter-key` | Clé API OpenRouter |
| .gemini-key | `~/.config/opencode/.gemini-key` | Clé API Google Gemini |
| .zhipu-key | `~/.config/opencode/.zhipu-key` | Clé API Z.AI |
| .groq-key | `~/.config/opencode/.groq-key` | Clé API Groq |
| .novita-key | `~/.config/opencode/.novita-key` | Clé API Novita |
| .hf-key | `~/.config/opencode/.hf-key` | Clé API Hugging Face |
| .cloudflare-key | `~/.config/opencode/.cloudflare-key` | Clé API Cloudflare |
| .cloudflare-account | `~/.config/opencode/.cloudflare-account` | Account ID Cloudflare |
| audit-YYYY-MM-DD.jsonl | `~/.config/opencode/logs/` | Logs d'audit |
| agent/eurinhash.md | `~/.config/opencode/agent/eurinhash.md` | Prompt du superviseur |
| scripts/*.py | `~/.config/opencode/scripts/` | Scripts de gouvernance |

## Configuration TUI (tui.json / tui.jsonc)

La configuration TUI est intégrée dans `Config.tui` :

```typescript
interface TuiConfigView {
    $schema?: string;
    theme?: string;
    plugin?: Record<string, boolean>;
    tui?: { scroll_speed?, scroll_acceleration?, diff_style? };
    leader_timeout: number;
    attention: TuiAttentionConfigView;
    keybinds: TuiBindingLookupView;
}
```

## Configuration du SDK (SdkConfig)

Le `SdkConfig` est le type de configuration du SDK, étendu par le `Config` du plugin :

```typescript
type Config = Omit<SDKConfig, "plugin"> & {
    plugin?: Array<string | [string, PluginOptions]>;
};
```

Le `SDKConfig` contient : `model`, `small_model`, `theme`, `keybinds`, `agents`, `providers`, `mcp`, `lsp`, `formatter`, `permission`, `tools`, `experimental`, etc.

## Variables d'environnement

| Variable | Rôle |
|----------|------|
| `EURINHASH_MODE` | Mode FREE ou PRO |
| `NO_COLOR` | Désactive les couleurs ANSI dans quota.py |

## Providers configurés

### Statut actuel (free-models.json)

| Worker | Modèle | Statut | Latence |
|--------|--------|--------|---------|
| worker-google | gemini-2.5-flash | **ok** | 1183ms |
| worker-groq | qwen/qwen3.8-27b | **ok** | 245ms |
| worker-novita | ling-3.0-flash-sante | **ok** | 1223ms |
| worker-pollinations | openai | **ok** | 630ms |
| worker-codestral | poolside/laguna-s-2.1:free | **rate_limited** | 1400ms |
| worker-zhipu | glm-4.7-flash | **error** | 31073ms |
| worker-ollama | devstral | **error** | 4077ms |
| worker-cloudflare | — | **skipped** | 0ms |

### Stratégie de routage

```
EV(worker) = qualité × disponibilité / coût
- qualité : baseline ajustée par Bayes
- disponibilité : 1.0 (ok), 0.2 (rate_limited), 0 (error)
- coût : latence / 1000 + 1
```

## MCP Servers configurés

| Server | Type | URL | Enabled |
|--------|------|-----|---------|
| cloudflare | remote | https://mcp.cloudflare.com/mcp | false |
| cloudflare-docs | remote | https://docs.mcp.cloudflare.com/mcp | false |
| postgres | local | npx @database-mcp/postgres | false |
| agentdeals | remote | https://agentdeals-production.up.railway.app/mcp | true |

## Configuration Prisma (base de données)

Le repo hashcode_reboot utilise Prisma pour la persistance :
- `prisma/schema.prisma` — schéma de la base
- `prisma/migrations/` — migrations
- La DB est PostgreSQL (P1000 auth actuellement inaccessible)

## Configuration Next.js

- `next.config.ts` — configuration Next.js
- `tsconfig.json` — TypeScript strict, path alias `@/*`
- `tailwind.config.ts` — Tailwind CSS 4
- `components.json` — shadcn/ui configuration

## Fait vs Hypothèse

| Fait confirmé | Hypothèse |
|---------------|-----------|
| La config est en JSONC (commentaires autorisés) | Le mécanisme de validation exact est dans le SDK |
| Les clés API sont stockées dans des fichiers séparés | La sécurité des fichiers .key dépend des permissions filesystem |
| Les permissions utilisent la règle "last match wins" | Le parser exact des patterns bash est dans le SDK |
| Les providers sont configurés avec npm adapters | Les npm adapters sont installés dans le runtime |
| Le MCP agentdeals est le seul activé | Les autres MCP peuvent être activés selon les besoins |
| La DB Prisma est inaccessible (P1000) | Le comportement quand la DB est inaccessible est géré par le code |

## Points de configuration critiques

1. **`permission.bash`** : la configuration de sécurité la plus critique (garde-fous destructeurs)
2. **`provider`** : détermine quels modèles sont disponibles et leurs configurations
3. **`agent`** : configuration des agents par rôle (modèles forcés)
4. **`compaction`** : contrôle la gestion du contexte (auto, prune, reserved)
5. **`plugin`** : liste des plugins actifs, l'ordre peut importer
6. **`experimental`** : features en cours de développement (batch_tool, continue_loop_on_deny)
