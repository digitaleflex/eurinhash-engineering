# Upstream Migration — Contraintes de migration upstream

> **Source** : `ARCHITECTURE_FINDINGS.md`, `ARCHITECTURE_FINDINGS.md` §10 (Risques & Dépendances)
> **Principe** : Préserver la compatibilité upstream autant que possible. Aucune modification du binaire OpenCode.

---

## 1. Contexte

OpenCode est un binaire compilé (`.exe`) version **1.18.31**. L'analyse a été faite via les types TypeScript `.d.ts` et l'API plugin. Le projet eurinhash-opencode ne modifie **jamais** le source ou le binaire OpenCode.

### 1.1 État actuel

| Aspect | Valeur |
|--------|--------|
| Version OpenCode | 1.18.31 |
| Plugin API version | 1.18.30 |
| Écart version | Patch (1.18.30 → 1.18.31) — Compatible |
| Source | Compilé (.exe) — Pas de source TS |
| Framework TUI | SolidJS via @opentui/* |
| Plugins | TuiPlugin (côté TUI) + Plugin (côté server) |

---

## 2. Contraintes de migration

### 2.1 Compatibilité de version

**Contrainte** : Les plugins doivent être compatibles avec la version du binaire OpenCode.

| Écart de version | Impact | Mitigation |
|-----------------|--------|------------|
| Patch (1.18.30 → 1.18.31) | ✅ Compatible | Aucune action requise |
| Mineur (1.18.x → 1.19.x) | ⚠️ Risque de breaking changes | Tester les types `.d.ts`, vérifier les hooks |
| Majeur (1.x → 2.x) | ❌ Risque élevé | Relecture complète de l'API plugin |

**Règle** : Les plugins doivent spécifier la version minimale d'OpenCode qu'ils supportent.

### 2.2 API Plugin stable vs expérimentale

| Catégorie | Stability | Migration |
|-----------|-----------|-----------|
| `TuiPluginApi` (state, event, ui, slots) | **Stable** | Non concernée |
| `PluginInput` / `Hooks` | **Stable** | Non concernée |
| `tool()` definition | **Stable** | Non concernée |
| `api.command` (legacy) | **Deprecated** | Utiliser `api.keymap.registerLayer()` |
| `experimental.*` hooks | **Instable** | Suivre les changements upstream |
| `@opentui/*` packages | **Stable** | Non concernée |

### 2.3 Dépendances `@opentui/*`

**Contrainte** : `@opentui` n'est pas dans le `node_modules` standard — il est fourni par le binaire OpenCode. Les types sont disponibles via `@opencode-ai/plugin/dist/tui.d.ts`.

| Package | Source | Risque |
|---------|--------|--------|
| `@opentui/core` | Fourni par OpenCode | Dépend de la version du binaire |
| `@opentui/keymap` | Fourni par OpenCode | Dépend de la version du binaire |
| `@opentui/solid` | Fourni par OpenCode | Dépend de la version du binaire |

**Règle** : Les types `@opentui` doivent être importés via `@opencode-ai/plugin/dist/tui.d.ts`, jamais directement depuis `@opentui/*`.

---

## 3. Points de rupture potentiels

### 3.1 `TuiPluginApi` évolutions

| Champ | Status | Risque migration |
|-------|--------|-----------------|
| `api.command` | **@deprecated** — supprimé en v2 | Faible (déjà migré vers keymap) |
| `api.state` | Stable | Risque si `SessionStatus` évolue |
| `api.event` | Stable | Risque si `Event` union s'élargit |
| `api.slots` | Stable | Risque si `TuiHostSlotMap` évolue |
| `api.lifecycle` | Stable | Risque si `TuiLifecycle` change |
| `api.plugins` | Stable | Risque si `TuiPluginInstallOptions` change |

### 3.2 `Hooks` interface évolutions

| Hook | Status | Risque migration |
|------|--------|-----------------|
| `event` | Stable | Faible |
| `tool.execute.before/after` | Stable | Faible |
| `permission.ask` | Stable | Faible |
| `chat.message` | Stable | Faible |
| `experimental.*` | **Instable** | Élevé — peut changer entre versions |

### 3.3 Types SDK évolutions

| Type | Status | Risque migration |
|------|--------|-----------------|
| `Event` union | Croissant | Moyen — nouveaux types ajoutés |
| `Session` | Stable | Faible |
| `Part` union | Croissant | Moyen — nouveaux types de parts |
| `Message` | Stable | Faible |
| `Model` / `Provider` | Stable | Faible |

---

## 4. Stratégie de migration

### 4.1 Approche : Compatibilité ascendante

```
Règle 1 : Ne jamais modifier le binaire OpenCode
Règle 2 : Toujours utiliser l'API publique @opencode-ai/plugin
Règle 3 : Importer les types via les .d.ts, jamais directement
Règle 4 : Gérer les deprecated API avec des guards
Règle 5 : Tester sur chaque mise à jour mineure d'OpenCode
```

### 4.2 Gestion des deprecated

```typescript
// ✅ CORRECT : Utiliser l'API moderne
api.keymap.registerLayer(bindings);

// ❌ INTERDIT : Utiliser l'API deprecated
api.command.register(() => commands);

// Guard pour la compatibilité
if (api.command) {
  // Fallback pour les anciennes versions
  api.command.register(() => commands);
} else {
  api.keymap.registerLayer(bindings);
}
```

### 4.3 Gestion des versions de types

```typescript
// Vérifier la version d'OpenCode
const version = api.app.version; // "1.18.31"

// Adapter selon la version
if (version.startsWith("1.18")) {
  // Logique pour v1.18.x
} else if (version.startsWith("1.19")) {
  // Logique pour v1.19.x
}
```

### 4.4 Migration des hooks expérimentaux

Les hooks `experimental.*` sont la plus grande source de risque :

| Hook | Risque | Stratégie |
|------|--------|-----------|
| `experimental.provider.small_model` | Moyen | Surveiller les changements de signature |
| `experimental.session.compacting` | Moyen | Surveiller les changements de signature |
| `experimental.compaction.autocontinue` | Moyen | Surveiller les changements de signature |
| `experimental.chat.messages.transform` | Élevé | Peut être renommé ou supprimé |
| `experimental.chat.system.transform` | Élevé | Peut être renommé ou supprimé |
| `experimental.text.complete` | Moyen | Peut être intégré au core |

**Règle** : Chaque hook expérimental doit avoir un commentaire documentant la version d'OpenCode où il a été introduit et les changements connus.

---

## 5. Risques identifiés

### 5.1 Source compilé

| Risque | Impact | Mitigation |
|--------|--------|------------|
| Pas de source TS directe | Impossible de debugger le core | Utiliser les types `.d.ts` et le plugin API |
| Binaire figé | Les bugs du core ne sont pas corrigés par nous | Signaler via les canaux officiels |
| Écart version plugin/binaire | Incompatibilité potentielle | Tester à chaque mise à jour |

### 5.2 `@opentui` non inspectable

| Risque | Impact | Mitigation |
|--------|--------|------------|
| Framework TUI non inspectable | Impossible de modifier le rendu | Utiliser les types `.d.ts` fournis |
| `@opentui` pas dans `node_modules` | Dépendance opaque | Importer via `@opencode-ai/plugin/dist/tui.d.ts` |

### 5.3 Données manquantes dans le SDK

| Risque | Impact | Mitigation |
|--------|--------|------------|
| Latence non disponible | Pas de métrique de performance | Calcul via timestamps |
| Queue non disponible | Pas de métrique de file d'attente | Compteur de messages/parts |
| Risque/Sensibilité non dans SDK | Pas de source native | Définir dans `guard.ts` ou eurinhash |

---

## 6. Plan de migration

### 6.1 Phase 1 — Stabilisation (immédiat)

- [ ] Vérifier la compatibilité avec OpenCode 1.18.31
- [ ] Tester tous les types `.d.ts` contre le binaire actuel
- [ ] Valider que `TuiPluginApi` fonctionne comme attendu
- [ ] Confirmer que les hooks `experimental.*` sont stables

### 6.2 Phase 2 — Adaptation mineure (à chaque mise à jour)

- [ ] Tester sur OpenCode 1.19.x
- [ ] Vérifier les nouveaux types dans `Event` union
- [ ] Vérifier les nouveaux hooks `experimental.*`
- [ ] Mettre à jour les `.d.ts` si nécessaire
- [ ] Migrer les deprecated APIs (`api.command` → `api.keymap`)

### 6.3 Phase 3 — Migration majeure (si v2.x)

- [ ] Relecture complète de l'API plugin
- [ ] Vérification de `TuiPluginApi` v2
- [ ] Mise à jour des types SDK v2
- [ ] Migration des hooks expérimentaux
- [ ] Test complet du système de slots

---

## 7. Rollback strategy

### 7.1 Rollback plugin

Si une nouvelle version d'OpenCode casse la compatibilité :

1. **Désactiver le plugin** : `api.plugins.deactivate("eurinhash")`
2. **Revenir à la version précédente** : `api.plugins.install("eurinhash@version-précédente")`
3. **Activer le plugin** : `api.plugins.activate("eurinhash")`

### 7.2 Rollback TUI

Si le rendu TUI est cassé :

1. **Supprimer les slots** : Ne plus enregistrer de `TuiSlotPlugin`
2. **Supprimer les événements** : Ne plus souscrire aux `Event` types
3. **Redémarrer le TUI** : `createOpencodeTui()` relance avec les plugins par défaut

### 7.3 Rollback complet

Si OpenCode lui-même a un problème :

1. **Recharger la version précédente du binaire**
2. **Vérifier la compatibilité des types `.d.ts`**
3. **Tester le plugin dans l'ancien environnement**
4. **Reprendre le travail avec la version stable**

---

## 8. Coût de migration

| Type de migration | Coût estimé | Risque |
|-------------------|-------------|--------|
| Patch version (1.18.x → 1.18.y) | Faible | Faible |
| Mineure (1.18 → 1.19) | Moyen | Moyen |
| Majeure (1.x → 2.x) | Élevé | Élevé |
| Changement `@opentui` | Moyen | Moyen |
| Ajout de type `Event` | Faible | Faible |
| Changement hook expérimental | Moyen | Moyen |

---

## 9. Validation de la migration

```
CHECKLIST — Upstream Migration Validation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
☐ Aucune modification du binaire OpenCode
☐ Tous les types importés via @opencode-ai/plugin/dist/*.d.ts
☐ Pas d'import direct depuis @opentui/*
☐ Les deprecated APIs (api.command) sont migrées vers keymap
☐ Les hooks expérimentaux sont documentés avec leur version
☐ La compatibilité est testée à chaque mise à jour OpenCode
☐ Un plan de rollback est défini pour chaque version
☐ Les TuiPlugin utilisent api.lifecycle.onDispose pour le cleanup
☐ Les plugins server utilisent l'interface Hooks publique
☐ Les types TuiState sont readonly dans tous les cas
☐ Les nouveaux types Event sont testés dès leur apparition
☐ Les changements de signature des hooks sont documentés
```

---

## 10. Références

- `ARCHITECTURE_FINDINGS.md` §10 — Risques & Dépendances
- `ARCHITECTURE_FINDINGS.md` §11 — Stratégie d'implémentation recommandée
- `plugin/dist/tui.d.ts` — TuiPluginApi types (dépréciations notées)
- `plugin/dist/index.d.ts` — Hooks interface (hooks expérimentaux)
- `plugin/dist/tool.d.ts` — ToolDefinition types
- `sdk/dist/v2/gen/types.gen.d.ts` — Event union et types
- `sdk/dist/v2/server.d.ts` — createOpencodeServer, createOpencodeTui
