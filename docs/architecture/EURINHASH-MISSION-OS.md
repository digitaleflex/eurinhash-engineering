# EURINHASH — Mission Operating Model

> Version 1.0 — 2026-09-19
>
> **FR/EN reference architecture**

## FR — Vision

EurinHash n'est pas un simple AI gateway, un routeur de modèles ou un clone d'agent de code. Il s'agit d'une couche d'orchestration IA orientée mission qui transforme une intention en résultat vérifié et produisant des preuves.

### Problème fondamental

Les systèmes IA savent de mieux en mieux produire une réponse ou exécuter une tâche, mais ils ne garantissent pas suffisamment :

- que les ressources choisies sont suffisantes et économiquement adaptées ;
- que l'état réel d'un provider, modèle, compte, gateway ou outil correspond à sa configuration ;
- que les erreurs sont correctement classifiées et récupérées ;
- que le travail demandé est réellement terminé ;
- que la déclaration « terminé » est soutenue par des preuves techniques.

### Promesse

**Transformer une intention en résultat vérifié.**

`INTENT → MISSION → CONTRACT → CONTEXT → TASKS → CAPABILITIES → RESOURCES → EXECUTION → VALIDATION → VERIFICATION → PROOF`

## EN — Vision

EurinHash is not merely an AI gateway, model router, or coding-agent clone. It is a mission-oriented AI orchestration layer that turns intent into a verified, evidence-backed outcome.

### Fundamental problem

AI systems increasingly generate answers and execute tasks, but they do not sufficiently guarantee that:

- selected resources are sufficient and economically appropriate;
- runtime reality matches configured provider/model/account/gateway/tool state;
- failures are correctly classified and recovered;
- the requested work is actually complete;
- a “done” claim is backed by technical evidence.

### Promise

**Turn intent into verified outcomes.**

`INTENT → MISSION → CONTRACT → CONTEXT → TASKS → CAPABILITIES → RESOURCES → EXECUTION → VALIDATION → VERIFICATION → PROOF`

---

## 1. Architecture

```text
                         EURINHASH
                             │
                       INTENT ENGINE
                             │
                       MISSION ENGINE
                             │
                      CONTRACT ENGINE
                             │
                       CONTEXT ENGINE
                             │
                         TASK ENGINE
                             │
                  CAPABILITY ANALYZER
                             │
                     RESOURCE PLANNER
                             │
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼
      MODEL ROUTER       AGENT ROUTER       TOOL ROUTER
          │                  │                  │
          └──────────────────┼──────────────────┘
                             ▼
                    MODEL ECONOMY ENGINE
                             │
                    EXECUTION ENGINE
                             │
                    EVENT / STATE ENGINE
                             │
                    VALIDATION ENGINE
                             │
                    MISSION VERIFIER
                             │
                    EVIDENCE / PROOF
```

## 2. Core domain objects

- **Mission**: unit of user intent and acceptance.
- **Contract**: objective, constraints, deliverables, acceptance criteria, budgets and risk.
- **Context**: relevant knowledge required to execute the mission.
- **Task**: executable unit in a dependency graph.
- **Capability**: requirement such as coding, reasoning, tool use, context length, modality.
- **Model**: AI model identity and capabilities.
- **Provider**: service exposing one or more models.
- **Account**: credential/quota identity for a provider.
- **Gateway**: transport/routing substrate such as OmniRoute.
- **Agent**: execution role with authority, tools and resource policy.
- **Tool**: executable capability with permission/risk constraints.
- **Budget**: money, tokens, time, agent calls, risk and human attention.
- **Event**: immutable observation emitted by the runtime/control plane.
- **State**: deterministic projection of observed events.
- **Validation**: technical checks actually executed.
- **Evidence**: concrete outputs of validation and execution.
- **Verification**: decision that acceptance criteria are satisfied by evidence.
- **Proof**: durable explanation of why the mission is considered verified.

## 3. Truth model

These distinctions are mandatory:

```text
CONFIGURED ≠ DISCOVERED ≠ AVAILABLE ≠ AUTHORIZED
≠ HEALTHY ≠ CAPABLE ≠ ELIGIBLE ≠ ACTIVE ≠ USED ≠ VERIFIED

REQUESTED ≠ RUNNING ≠ COMPLETED ≠ VALIDATED ≠ VERIFIED

UNKNOWN ≠ ZERO
SKIPPED ≠ PASSED
```

The system must never fabricate telemetry, quota, cost, capability, progress or success.

## 4. Resource decision model

The router does not ask “what is the best model?”. It asks:

> What is the least costly sufficient resource that satisfies task, context, reliability, security, quota and latency requirements?

Candidate pipeline:

`DISCOVER → FILTER → ELIGIBILITY → SCORE → SELECT → EXPLAIN → EXECUTE → LEARN`

Decision dimensions may include task fit, context fit, tool support, health, quota, reliability, latency, cost and policy constraints. Scores are not truth; observed runtime evidence remains authoritative.

## 5. Economy model

Effective mission cost is broader than API price:

`financial cost + wasted tokens + retries + failed execution + latency + context reconstruction + human intervention + verification cost`

Budgets may include:

- financial budget;
- token budget;
- duration budget;
- agent-call budget;
- risk budget;
- human-attention budget.

## 6. Resilience

Failures must be classified before recovery:

`429 quota/rate | 401 auth | 403 policy | 404 model | 502 upstream | 503 unavailable | 504 timeout | STREAM_STALLED | TOOL_FAILURE | INVALID_OUTPUT`

Recovery actions can include retry, cooldown, circuit breaker, fallback, reroute or abort. Fallback is not a blind provider loop.

## 7. Mission verification

A model claiming “done” is not sufficient.

```text
MISSION CONTRACT
      +
EXECUTION EVIDENCE
      +
VALIDATION RESULTS
      +
ACCEPTANCE CRITERIA
      ↓
MISSION VERIFIER
      ↓
VERIFIED / UNVERIFIED / FAILED / BLOCKED
```

The verifier must identify each criterion individually and retain evidence references.

## 8. Multi-model review

Multiple models are used only when the expected quality/risk reduction justifies their resource cost. Typical roles may include planner, implementer, reviewer, security reviewer and verifier. This is policy-driven, not a default requirement.

## 9. System boundaries

| System | Authority |
|---|---|
| `eurinhash-engineering` | product intent, domain, architecture, UX, contracts, acceptance |
| `opencode-config` | orchestration, routing, agents, policies, control plane, workflows |
| `eurinhash-opencode` | terminal/product experience and runtime-facing UI |
| OpenCode | execution substrate |
| models.dev | external model catalog knowledge |
| OmniRoute | optional gateway/transport substrate |

Rule: **Engineering defines. Config orchestrates. Product executes/presents. Validation proves.**

## 10. Product north star

EurinHash should optimize for **verified mission outcomes**, not provider count, model count, agent count, or dashboard complexity.

The competitive question is not:

> “How many models can we connect?”

It is:

> “Can we accomplish a mission reliably, economically, safely, explainably, and prove that it is complete?”

## 11. Non-goals

- Do not become a provider-count race.
- Do not blindly fork upstream OpenCode.
- Do not make OmniRoute the architectural authority.
- Do not treat free models as inherently reliable or cheap.
- Do not use UI state as runtime truth.
- Do not declare success without evidence.
- Do not invent missing backend telemetry.

## 12. Roadmap themes

1. Model intelligence and capability matching.
2. Intelligent routing and economy.
3. Gateway/provider/account abstraction.
4. Resilient streaming and failover.
5. Mission contracts and budgets.
6. Mission verification and evidence.
7. Explainable decisions.
8. Multi-model review where justified.
9. Replay and evaluation.
10. Learning from observed mission outcomes.

---

# Appendix — Research lessons

The design was informed by observed ecosystem failure modes: quota opacity, fallback loops, stale provider catalogs, UI/runtime divergence, streaming stalls, client-version integration drift, policy restrictions on free tiers, tool-call failures and configuration/schema mismatches. These are treated as engineering problem classes, not as claims that any single implementation is universally deficient.
