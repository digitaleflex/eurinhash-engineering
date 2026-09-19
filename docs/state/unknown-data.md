# Unknown Data — Treatment Protocol: `UNKNOWN != ZERO`

> Source: All `.d.ts` type files
> Rule: When a value is absent, it is `undefined`. It is NEVER `0`, `""`, `false`, or an empty array as a substitute.

---

## Core principle

In the OpenCode type system, optional fields are marked with `?` in TypeScript. The absence of a value is represented by `undefined`, not by any sentinel value.

**This is a design decision, not an implementation detail.** Every state field, every event payload, and every configuration option follows this rule.

### Why `UNKNOWN != ZERO` matters

| Scenario | Correct | Wrong | Why |
|----------|---------|-------|-----|
| Session has no parent | `parentID: undefined` | `parentID: ""` | `""` could be a valid ID |
| Tool has no output yet | `output: undefined` | `output: ""` | `""` is a valid output; `undefined` means not yet produced |
| LSP has no branch | `branch: undefined` | `branch: ""` | `""` could mean a branch named "" |
| Compaction hasn't started | `time.compacting: undefined` | `time.compacting: 0` | `0` would mean started at epoch |
| Cost not yet computed | `cost: undefined` | `cost: 0` | `0` means free; `undefined` means not calculated |
| Error not present | `error: undefined` | `error: ""` | `""` is a valid error message |
| Tokens not yet counted | `tokens.input: 0` | — | `0` IS correct here — tokens counted as 0 |

**Critical distinction**: `0` as a count is valid when the count is genuinely zero. `undefined` as a value means "not yet known" or "not applicable."

---

## Field-by-field UNKNOWN treatment

### Session fields

| Field | Type | When absent | Never |
|-------|------|-------------|-------|
| `parentID` | `string \| undefined` | Root session | `""` |
| `summary` | `Summary \| undefined` | No summary yet | `{ additions: 0, deletions: 0, files: 0 }` |
| `share?.url` | `string \| undefined` | Not shared | `""` |
| `time.compacting` | `number \| undefined` | Not compacting | `0` |
| `time.archived` | `number \| undefined` | v2, not archived | `0` |
| `revert` | `Revert \| undefined` | No revert in progress | `{ messageID: "", partID: "" }` |
| `cost` (v2) | `number \| undefined` | Not computed | `0` |
| `tokens` (v2) | `Tokens \| undefined` | Not computed | `{ input: 0, ... }` |
| `metadata` (v2) | `{ [key: string]: unknown } \| undefined` | No metadata | `{}` |
| `permission` (v2) | `PermissionRuleset \| undefined` | No custom rules | `[]` (empty array is valid) |
| `agent` (v2) | `string \| undefined` | No agent specified | `""` |
| `model` (v2) | `ModelRef \| undefined` | No model set | `{ id: "", providerID: "" }` |

### Message fields

| Field | Type | When absent | Never |
|-------|------|-------------|-------|
| `AssistantMessage.error` | `ErrorType \| undefined` | No error | `""` or fake error object |
| `AssistantMessage.time.completed` | `number \| undefined` | Not completed | `0` |
| `AssistantMessage.finish` | `string \| undefined` | Not finished | `""` |
| `UserMessage.summary` | `Summary \| undefined` | No summary | `{ title: "", body: "", diffs: [] }` |
| `UserMessage.system` | `string \| undefined` | No system prompt | `""` |
| `Part.metadata` | `{ [key: string]: unknown } \| undefined` | No metadata | `{}` (empty object is valid but distinct) |
| `TextPart.synthetic` | `boolean \| undefined` | Not synthetic | `false` (false means explicitly not synthetic) |
| `TextPart.ignored` | `boolean \| undefined` | Not ignored | `false` |
| `TextPart.time.end` | `number \| undefined` | Not ended | `0` |
| `ReasoningPart.time.end` | `number \| undefined` | Not ended | `0` |

### Tool fields

| Field | Type | When absent | Never |
|-------|------|-------------|-------|
| `ToolStateRunning.title` | `string \| undefined` | No title | `""` |
| `ToolStateRunning.metadata` | `{ [key: string]: unknown } \| undefined` | No metadata | `{}` |
| `ToolStateCompleted.attachments` | `FilePart[] \| undefined` | No attachments | `[]` |
| `ToolStateCompleted.compacted` | `number \| undefined` | Not compacted | `0` |
| `ToolStateError.metadata` | `{ [key: string]: unknown } \| undefined` | No metadata | `{}` |
| `ToolPart.metadata` | `{ [key: string]: unknown } \| undefined` | No metadata | `{}` |

### MCP fields

| Field | Type | When absent | Never |
|-------|------|-------------|-------|
| `McpStatusFailed.error` | `string` | Present in failed state | `""` — error is required when status is "failed" |
| `McpStatusNeedsClientRegistration.error` | `string` | Present in this state | `""` — error is required |
| `McpLocalConfig.environment` | `{ [key: string]: string } \| undefined` | No env vars | `{}` |
| `McpRemoteConfig.headers` | `{ [key: string]: string } \| undefined` | No headers | `{}` |
| `McpRemoteConfig.oauth` | `McpOAuthConfig \| false \| undefined` | No OAuth | `null` |

### LSP fields

| Field | Type | When absent | Never |
|-------|------|-------------|-------|
| `LspStatus.name` | `string` | Never absent | `""` |
| `LspStatus.root` | `string` | Never absent | `""` |
| `LspConfig.initialization` | `{ [key: string]: unknown } \| undefined` | No initialization | `{}` |
| `LspConfig.env` | `{ [key: string]: string } \| undefined` | No env vars | `{}` |

### Permission fields

| Field | Type | When absent | Never |
|-------|------|-------------|-------|
| `Permission.pattern` | `string \| Array<string> \| undefined` | No pattern | `[]` or `""` |
| `Permission.callID` | `string \| undefined` | Not tool-based | `""` |
| `Permission.metadata` | `{ [key: string]: unknown }` | Empty metadata | `undefined` — field always exists |
| `PermissionV2Reply` | `"once" \| "always" \| "reject"` | Never absent | `""` |
| `PermissionV2Asked.save` | `Array<string> \| undefined` | No save rules | `[]` |
| `PermissionV2Asked.source` | `PermissionV2Source \| undefined` | Not from tool | `""` |
| `PermissionV2Asked.metadata` | `{ [key: string]: unknown } \| undefined` | No metadata | `{}` |

### Git/VCS fields

| Field | Type | When absent | Never |
|-------|------|-------------|-------|
| `VcsInfo.branch` | `string \| undefined` | No branch | `""` |
| `VcsInfo.default_branch` | `string \| undefined` | No default branch | `""` |
| `VcsBranchUpdated.branch` | `string \| undefined` | Branch removed | `""` |
| `FileDiff.before` | `string` | New file | `""` — `before` is always present |
| `FileDiff.after` | `string` | Deleted file | `""` — `after` is always present |

### Model fields

| Field | Type | When absent | Never |
|-------|------|-------------|-------|
| `Model.family` | `string \| undefined` | No family info | `""` |
| `Model.variants` | `{ [key: string]: { [key: string]: unknown } } \| undefined` | No variants | `{}` |
| `ModelRef.variant` | `string \| undefined` | Default variant | `""` |
| `Session.model` | `ModelRef \| undefined` | No model set | `{ id: "", providerID: "" }` |
| `Session.agent` | `string \| undefined` | No agent | `""` |
| `Session.cost` | `number \| undefined` | Not computed | `0` |
| `Session.tokens` | `Tokens \| undefined` | Not computed | `{ input: 0, ... }` |

### Workspace fields

| Field | Type | When absent | Never |
|-------|------|-------------|-------|
| `Workspace.branch` | `string \| null` | Not a git repo | `""` — `null` means no git |
| `Workspace.directory` | `string \| null` | Not set | `""` |
| `Workspace.extra` | `unknown \| null` | No extra data | `{}` |
| `WorkspaceInfo.branch` | `string \| null` | No git branch | `""` |
| `WorkspaceInfo.directory` | `string \| null` | Not set | `""` |
| `Project.vcs` | `ProjectVcs \| undefined` | No VCS | `""` |
| `Path.home` | `string` | Always present | Never absent |
| `Path.state` | `string` | Always present | Never absent |

### TUI fields

| Field | Type | When absent | Never |
|-------|------|-------------|-------|
| `TuiState.vcs` | `{ branch?: string; default_branch?: string } \| undefined` | No VCS | `{ branch: "" }` |
| `TuiState.session.get(id)` | `Session \| undefined` | No session | Fake session object |
| `TuiState.session.status(id)` | `SessionStatus \| undefined` | Status unknown | `{ type: "idle" }` |
| `TuiPromptInfo.mode` | `"normal" \| "shell" \| undefined` | Not specified | `"normal"` |
| `TuiToast.title` | `string \| undefined` | No title | `""` |
| `TuiThemeCurrent` fields | `RGBA` | Always present | `undefined` |
| `TuiPluginEntry.version` | `string \| undefined` | No version | `""` |
| `TuiPluginEntry.modified` | `number \| undefined` | Not modified | `0` |

### Event fields

| Field | Type | When absent | Never |
|-------|------|-------------|-------|
| `EventSessionError.sessionID` | `string \| undefined` | Global error | `""` |
| `EventSessionError.error` | `ErrorType \| undefined` | No error detail | `""` or fake error |
| `EventMessagePartUpdated.delta` | `string \| undefined` | Full update, no delta | `""` — delta absent means full update |
| `EventPermissionUpdated.properties` | `Permission` | Always present | — |
| `EventLspUpdated.properties` | `{ [key: string]: unknown }` | Always present (empty) | `undefined` |
| `EventServerConnected.properties` | `{ [key: string]: unknown }` | Always present (empty) | `undefined` |
| `EventVcsBranchUpdated.branch` | `string \| undefined` | Branch removed | `""` |

### Config fields

| Field | Type | When absent | Never |
|-------|------|-------------|-------|
| `Config.model` | `string \| undefined` | No default model | `""` |
| `Config.small_model` | `string \| undefined` | No small model | `""` |
| `Config.username` | `string \| undefined` | No custom username | `""` |
| `Config.enterprise.url` | `string \| undefined` | No enterprise | `""` |
| `Provider.key` | `string \| undefined` | No API key | `""` |
| `Provider.env` | `Array<string>` | Always present (may be empty) | `[]` is valid |
| `Provider.models` | `{ [key: string]: Model }` | Always present (may be empty) | `{}` is valid |
| `McpLocalConfig.environment` | `{ [key: string]: string } \| undefined` | No env vars | `{}` |
| `McpOAuthConfig.clientId` | `string \| undefined` | Dynamic registration | `""` |
| `McpOAuthConfig.clientSecret` | `string \| undefined` | Dynamic registration | `""` |
| `McpOAuthConfig.scope` | `string \| undefined` | No scope | `""` |

---

## `undefined` vs `0` vs `""` vs `false` vs `[]`

### When `0` IS correct

| Field | Value `0` means |
|-------|----------------|
| `tokens.input` | Zero input tokens (message with no input) |
| `tokens.output` | Zero output tokens |
| `tokens.reasoning` | Zero reasoning tokens |
| `additions` | No lines added |
| `deletions` | No lines deleted |
| `cost` | Zero cost (only if genuinely free) |
| `time.start` / `time.end` | Epoch — only if actually at timestamp 0 |
| `attempt` (in retry) | Zero retries attempted |

### When `""` IS correct

| Field | Value `""` means |
|-------|-----------------|
| `FileDiff.before` | Empty file (new file) |
| `FileDiff.after` | Empty file (deleted file) |
| `error` in `ToolStateError` | Empty error string (edge case) |
| `text` in `TextPart` | Empty text |

### When `undefined` is correct

| Field | `undefined` means |
|-------|-------------------|
| `parentID` | Root session |
| `time.compacting` | Not compacting |
| `cost` | Not computed yet |
| `tokens` | Not computed yet |
| `error` | No error occurred |
| `finish` | Not finished |
| `summary` | No summary available |
| `metadata` | No metadata attached |
| `model` | No model selected |
| `agent` | No agent configured |
| `branch` | No branch (not a git repo) |
| `revert` | No revert in progress |
| `status` (TuiState) | Status unknown |
| `session.get(id)` | Session does not exist |

---

## Serialization implications

When serializing state to JSON or transmitting over the wire:

1. `undefined` fields are typically omitted from JSON objects (TypeScript `?` → JSON absent)
2. `0`, `""`, `false`, and `[]` are always serialized
3. **Never replace `undefined` with `0` or `""` before serialization** — this corrupts the type semantics
4. When deserializing, missing fields should be `undefined`, not default values

### Example: Session JSON

```json
// Correct serialization
{ "id": "abc", "title": "My Session", "parentID": undefined }
// Becomes in JSON: { "id": "abc", "title": "My Session" }
// parentID is absent — this is correct

// INCORRECT serialization
{ "id": "abc", "title": "My Session", "parentID": "" }
// parentID is "" — this implies a session with empty-string ID as parent

// INCORRECT serialization
{ "id": "abc", "title": "My Session", "parentID": null }
// null is not a valid value for string | undefined
```

---

## Validation rules

1. **All `?` fields**: Accept `undefined` and the field's type
2. **All non-`?` fields**: Must always have a value; never `undefined`
3. **Discriminated unions**: Always validate the discriminant; never assume a variant
4. **`undefined` in arrays**: `Array<string>` can contain `undefined` only if `Array<string | undefined>`
5. **Optional objects**: `{ [key: string]: unknown } | undefined` means the entire object may be absent

---

## Testing guidelines

For each state field, test:
1. Present with valid value
2. Present with `0` / `""` / `false` (valid zero values)
3. Absent (`undefined`)
4. Invalid type (should fail validation)

**Never test** `0` as a substitute for `undefined` — these are fundamentally different states.
