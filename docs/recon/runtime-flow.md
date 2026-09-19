# Runtime Flow — Flux d'exécution

## Architecture runtime

OpenCode est une application **dual-Process** :
1. **Serveur** (`@opencode-ai/sdk/dist/server.js`) — orchestration backend
2. **Client/TUI** (`@opencode-ai/sdk/dist/client.js`) — interface terminal
3. **Communication** — WebSocket/Server-Sent Events via `OpencodeClient`

## Flux principal d'une requête utilisateur

```
Utilisateur (TUI)
    │
    ▼
[TuiPromptProps.onSubmit] → submit()
    │
    ▼
[TuiEventBus] → émet un "tui.command.execute" ou envoie via OpencodeClient
    │
    ▼
[OpencodeClient] → POST /session/{id}/message (WebSocket/SSE)
    │
    ▼
[Server] → crée UserMessage, route vers agent/model
    │
    ▼
[Provider] → appel LLM (via @ai-sdk/* adapters)
    │
    ▼
[AssistantMessage] avec Part[] (text, tool, reasoning, etc.)
    │
    ▼
[EventBus] → émet les événements (message.part.updated, etc.)
    │
    ▼
[TuiEventBus.on("message.part.updated")] → met à jour le rendu
    │
    ▼
TUI affiche le résultat
```

## Flux de création de session

```
TUI → OpencodeClient.createSession({ title? })
    │
    ▼
Server → POST /session → crée Session
    │
    ▼
Server → émet "session.created" (EventSessionCreated)
    │
    ▼
TUI → reçoit l'événement, met à jour sidebar
```

## Flux de tool execution

```
Agent → décide d'exécuter un tool
    │
    ▼
[tool.execute.before] hooks (plugins)
    │   - guard.ts : vérifie commandes destructrices
    │   - audit-logger.ts : log l'outil
    │
    ▼
[Server] → exécute le tool (bash, edit, glob, grep, etc.)
    │
    ▼
[tool.execute.after] hooks (plugins)
    │   - guard.ts : redacte secrets dans output
    │   - audit-logger.ts : log le résultat
    │
    ▼
ToolPart.update → state: pending → running → completed/error
    │
    ▼
Event : "message.part.updated" avec ToolState
```

## Flux de compaction

```
Contexte approche de la limite
    │
    ▼
[experimental.session.compacting] hook
    │   - auto-compact.ts : injecte directive de préservation
    │
    ▼
Serveur compresse la session
    │
    ▼
[experimental.compaction.autocontinue] hook
    │   - auto-compact.ts : output.enabled = true
    │
    ▼
Serveur ajoute message synthétique "continue"
    │
    ▼
Agent reprend là où il s'était arrêté
```

## Flux d'événements (Event → TUI)

```
Server → EventStream (SSE/WebSocket)
    │
    ├── message.updated → TUI met à jour le message
    ├── message.part.updated → TUI met à jour la part (delta)
    ├── message.part.removed → TUI retire la part
    ├── session.status → TUI met à jour le status (idle/busy/retry)
    ├── session.created → TUI ajoute à la sidebar
    ├── session.deleted → TUI retire de la sidebar
    ├── permission.updated → TUI affiche la demande de permission
    ├── permission.replied → TUI traite la réponse
    ├── todo.updated → TUI met à jour les todos
    ├── command.executed → TUI confirme la commande
    ├── file.edited → TUI rafraîchit le fichier
    ├── file.watcher.updated → TUI met à jour le watcher
    ├── lsp.client.diagnostics → TUI affiche les diagnostics
    ├── pty.created/updated/exited/deleted → TUI met à jour le terminal
    ├── session.compacted → TUI indique la compaction
    ├── session.idle → TUI indique l'idle
    ├── tui.prompt.append → TUI ajoute du texte au prompt
    ├── tui.toast.show → TUI affiche un toast
    └── server.connected → TUI confirme la connexion
```

## Flux de routage des workers (EURINHASH)

```
Tâche reçue
    │
    ▼
Classifie le type : code | general | quick | long | heavy | review
    │
    ▼
route.py <type> calcule EV = qualité × disponibilité / coût
    │   - qualité : baseline ajustée par Bayes (route-usage.json)
    │   - disponibilité : 1.0 si ok, 0.2 si rate_limited, 0 si error
    │   - coût : latence / 1000 + 1
    │
    ▼
Retourne le worker meilleur EV
    │
    ▼
Si le worker échoue (429, timeout, auth) :
    │
    ▼
route.py <type> --next → worker suivant avec le progress accumulé
    │
    ▼
route.py record <worker> success|fail → feedback loop Bayes
```

## Flux de probe des workers

```
free-probe.py (lancé par route.py si free-models.json > 30min)
    │
    ├── worker-google : POST gemini-2.5-flash:generateContent
    ├── worker-zhipu : POST /chat/completions (glm-4.7-flash)
    ├── worker-codestral : POST /chat/completions (poolside/laguna-s-2.1:free)
    ├── worker-groq : POST /chat/completions (qwen/qwen3.8-27b)
    ├── worker-novita : POST /chat/completions (inclusionai/ling-3.0-flash-sante)
    ├── worker-pollinations : POST /openai (openai, anonyme)
    ├── worker-ollama : POST localhost:11434/v1/chat/completions
    └── worker-cloudflare : POST Cloudflare AI (@cf/zai-org/glm-4.7-flash)
    │
    ▼
Écrit free-models.json { updated, models, latencies_ms }
```

## Flux d'initialisation

```
opencode.jsonc lu
    │
    ├── Plugins chargés (tous les 9 plugins listés)
    ├── Provider configuré (zhipu, google, groq, openrouter, huggingface, novita, pollinations, ollama)
    ├── MCP servers initialisés (cloudflare, cloudflare-docs, postgres, agentdeals)
    ├── Keybinds chargés
    ├── Agent config (explorer, librarian, fixer, councillor, designer, oracle)
    └── TUI démarre avec OpencodeClient connecté
```

## Fait vs Hypothèse

| Fait confirmé | Hypothèse |
|---------------|-----------|
| Le serveur et le client communiquent via OpencodeClient | Le transport exact (WebSocket vs SSE) est dans le source server.js non inspecté |
| Les événements sont typés et nommés (Event["type"]) | La fréquence exacte des événements pendant une exécution n'est pas mesurée |
| La compaction utilise les hooks expérimentaux | Le comportement exact de la compaction (quand elle se déclenche) est dans le source |
| Les hooks tool.execute.before/after sont synchrones avec l'exécution | La latence ajoutée par les hooks n'est pas mesurée |
| Le routage EV est un système Python externe | La communication entre le TUI et route.py n'est pas documentée dans les types |

## Goulots d'étranglement identifiés

1. **free-probe.py** : fait 1 micro-appel par worker (~7 appels), peut prendre 10-30s
2. **route.py** : charge free-models.json + route-usage.json à chaque appel
3. **audit-logger.ts** : écriture asynchrone dans un fichier JSONL, rotation quotidienne
4. **opencode.jsonc** : configuration très volumineuse (461 lignes), chargée entièrement au démarrage
5. **better-compact** : problème Windows fsync (EPERM) → nécessite un patch npm
