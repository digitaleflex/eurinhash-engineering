# TUI Map — Architecture Terminal User Interface

## Vue d'ensemble

Le TUI OpenCode est construit sur le framework **@opentui** (SolidJS-based). Il est en cours d'extraction vers `@opencode-ai/tui` (upstream migration). Le TUI communique avec le serveur OpenCode via `OpencodeClient` et reçoit les événements en temps réel via un EventBus.

## Stack TUI

| Composant | Package | Rôle |
|-----------|---------|------|
| Moteur rendu | `@opentui/core` | CliRenderer, Renderable, SlotMode, RGBA |
| Keybindings | `@opentui/keymap` | Keymap<Renderable, KeyEvent>, Binding |
| Keybindings extras | `@opentui/keymap/extras` | BindingConfig, SequenceBindingLike, stringifyKeyStroke |
| Rendu JSX | `@opentui/solid` | JSX, SolidPlugin |
| Plugin TUI | `@opencode-ai/plugin/dist/tui.d.ts` | Contrat plugin TUI |

## Entrées du TUI

### Routes (`TuiRouteDefinition`)
- `home` — Page d'accueil
- `session` — Page de session (params: sessionID, prompt?)
- Routes dynamiques (`name: string, params?: Record<string, unknown>`)

### Slots (`TuiSlotMap`)
Slots principaux injectés dans le layout :

| Slot | Paramètres | Rôle |
|------|------------|------|
| `app` | — | Container racine |
| `app_bottom` | — | Barre du bas |
| `home_logo` | — | Logo accueil |
| `home_prompt` | `ref?: TuiPromptRef` | Zone de saisie accueil |
| `home_prompt_right` | — | Zone droite accueil |
| `home_bottom` | — | Zone bas accueil |
| `home_footer` | — | Pied de page accueil |
| `session_prompt` | `session_id`, `visible?`, `disabled?`, `on_submit?`, `ref?` | Zone saisie session |
| `session_prompt_right` | `session_id` | Zone droite session |
| `sidebar_title` | `session_id`, `title`, `share_url?` | Titre sidebar |
| `sidebar_content` | `session_id` | Contenu sidebar |
| `sidebar_footer` | `session_id` | Pied sidebar |

### Dialogs (`TuiDialog*Props`)
- `TuiDialogProps` — Dialog générique (size: medium/large/xlarge)
- `TuiDialogAlertProps` — Alert (title, message, onConfirm?)
- `TuiDialogConfirmProps` — Confirmation (title, message, onConfirm?, onCancel?)
- `TuiDialogPromptProps` — Prompt (title, description?, placeholder?, value?, busy?)
- `TuiDialogSelectProps<Value>` — Select (title, options, flat?, onMove?, onFilter?)

### Prompt (`TuiPromptProps`, `TuiPromptRef`)
- `TuiPromptInfo` — `{ input, mode?, parts[] }`
- `TuiPromptRef` — `{ focused, current, set(), reset(), blur(), focus(), submit() }`

## Composants UI (`TuiPluginApi.ui`)

| Composant | Signature | Rôle |
|-----------|-----------|------|
| `Dialog` | `(props: TuiDialogProps) => JSX.Element` | Dialog générique |
| `DialogAlert` | `(props: TuiDialogAlertProps) => JSX.Element` | Alert |
| `DialogConfirm` | `(props: TuiDialogConfirmProps) => JSX.Element` | Confirmation |
| `DialogPrompt` | `(props: TuiDialogPromptProps) => JSX.Element` | Prompt |
| `DialogSelect` | `<Value>(props) => JSX.Element` | Sélection |
| `Slot` | `<Name>(props) => JSX.Element \| null` | Slot custom |
| `Prompt` | `(props: TuiPromptProps) => JSX.Element` | Zone de saisie |
| `toast` | `(input: TuiToast) => void` | Notification toast |
| `dialog` | `TuiDialogStack` | Accès au stack de dialogs |

## Keybindings

### `KeybindsConfig` (dans `Config`)
Toutes les keybindings sont définies dans `@opencode-ai/sdk/dist/gen/types.gen.d.ts` :
- `leader`, `app_exit`, `editor_open`, `theme_list`, `sidebar_toggle`, `scrollbar_toggle`, `username_toggle`, `status_view`, `session_export`, `session_new`, `session_list`, `session_timeline`, `session_share`, `session_unshare`, `session_interrupt`, `session_compact`
- Navigation messages : `messages_page_up/down`, `messages_line_up/down`, `messages_half_page_up/down`, `messages_first/last`, `messages_next/previous`, `messages_last_user`, `messages_copy`, `messages_undo/redo`, `messages_toggle_conceal`
- Tools : `tool_details`, `model_list`, `model_cycle_recent`, `command_list`, `agent_list`, `agent_cycle`
- Input : `input_clear`, `input_forward_delete`, `input_paste`, `input_submit`, `input_newline`, `history_previous/next`
- Session enfants : `session_child_cycle`, `session_child_cycle_reverse`
- Terminal : `terminal_suspend`, `terminal_title_toggle`

### Mode API (`TuiModeApi`)
- `current(): string` — mode actuel
- `push(mode: string): () => void` — pousse un mode (retourne fonction de pop)

## Thème (`TuiTheme`, `TuiThemeCurrent`)

Le thème est un système RGBA complet avec :
- **Couleurs de base** : primary, secondary, accent, error, warning, success, info, text, textMuted, background, backgroundPanel, backgroundElement, backgroundMenu, border, borderActive, borderSubtle
- **Diff** : diffAdded, diffRemoved, diffContext, diffHunkHeader, diffHighlightAdded/Removed, diffAddedBg, diffRemovedBg, diffContextBg, diffLineNumber
- **Markdown** : markdownText, markdownHeading, markdownLink, markdownCode, markdownBlockQuote, markdownEmph, markdownStrong, markdownHorizontalRule, markdownListItem, markdownListEnumeration, markdownImage, markdownCodeBlock
- **Syntaxe** : syntaxComment, syntaxKeyword, syntaxFunction, syntaxVariable, syntaxString, syntaxNumber, syntaxType, syntaxOperator, syntaxPunctuation
- **Autre** : thinkingOpacity

Méthodes : `current`, `selected`, `has(name)`, `set(name)`, `install(jsonPath)`, `mode()` (dark/light), `ready`

## Attention (`TuiAttention`)

Système de notifications et sons :
- `notify(input)` — envoie une notification avec son optionnel
- `soundboard` — gestion des packs de sons
- `TuiAttentionSoundNames` : ["default", "question", "permission", "error", "done", "subagent_done"]
- `TuiAttentionWhen` : "always" | "focused" | "blurred"
- `TuiAttentionNotifyResult` : { ok, notification, sound, skipped? }

## KV Store (`TuiKV`)

- `get<Value>(key, fallback?)` — lecture persistante
- `set(key, value)` — écriture persistante
- `ready` — état de disponibilité

## Plugins TUI (`TuiPluginApi.plugins`)

- `list()` — liste des plugins installés
- `activate(id)` — active un plugin
- `deactivate(id)` — désactive un plugin
- `add(spec)` — ajoute un plugin
- `install(spec, options?)` — installe un plugin

## Fait vs Hypothèse

| Fait confirmé | Hypothèse |
|---------------|-----------|
| Le TUI utilise @opentui (Solid-based) avec des types strictement définis | Les composants UI exacts sont dans le source @opentui non inspecté |
| Les slots sont définis dans TuiHostSlotMap avec 16 slots fixes | Le nombre exact de slots custom possibles est limité uniquement par la mémoire |
| Le système de keybindings est complet et configurable | Les keybindings par défaut exactes ne sont pas dans les .d.ts |
| Le TUI supporte les dialogs, prompts, selects, toasts | Le comportement exact des animations/transitions n'est pas documenté dans les types |
| `TuiCommand` et `TuiCommandApi` sont deprecated (remplacés par keymap) | La date exacte de suppression v2 n'est pas connue |

## Migration @opencode-ai/tui

Le TUI est en cours d'extraction vers `@opencode-ai/tui`. Implications :
1. `@opentui/*` packages pourraient être renommés ou absorbés
2. `TuiPluginApi` sera le contrat stable
3. `TuiPluginModule` passera de `{ server, tui? }` à `{ tui }` uniquement
4. Les `@opentui/solid` imports dans tui.d.ts pourraient changer
5. Les types `@opencode-ai/plugin/dist/tui.d.ts` pourraient être déplacés
