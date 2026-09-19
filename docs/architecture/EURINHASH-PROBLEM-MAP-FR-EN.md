# EURINHASH — Problem Map & Solution Framework

> FR/EN — v1.0 — 2026-09-19

## FR — Carte des problèmes

### A. Intelligence
- sélection d'un modèle sans compréhension fine de la tâche ;
- capacité réelle différente de la configuration ;
- routage opaque ;
- absence de distinction entre catalogue et état live.

**Réponse :** Model Intelligence + Capability Matching + Explainable Router.

### B. Économie
- tokens inutiles ;
- retries coûteux ;
- modèles puissants utilisés pour des tâches simples ;
- quotas gratuits mal connus ;
- coût de l'échec ignoré.

**Réponse :** Model Economy + budgets + contexte optimisé.

### C. Fiabilité
- fallback après mauvaise classification ;
- providers indisponibles ;
- streams bloqués ;
- timeouts longs ;
- cycles de retry.

**Réponse :** Failure Classification + Circuit Breaker + Streaming State Machine + Resilience Engine.

### D. Exécution agentique
- agents trop autonomes ;
- permissions trop larges ;
- contexte excessif ;
- absence de checkpoints ;
- état difficile à reprendre.

**Réponse :** Agent Control + Permission Gate + Task Graph + Checkpoints.

### E. Vérification
- « done » confondu avec « verified » ;
- tests non exécutés ;
- critères d'acceptation vagues ;
- absence de preuve durable.

**Réponse :** Mission Contract + Validation Engine + Mission Verifier + Evidence/Proof.

### F. UX
- chat comme centre unique ;
- états difficiles à comprendre ;
- métriques sans contexte ;
- divergences UI/runtime ;
- manque d'explication des décisions.

**Réponse :** Mission-centered Command Center + canonical state projection + explainability.

### G. Intégrations
- changements de versions ;
- schémas de configuration divergents ;
- IDs provider incohérents ;
- dépendance à un gateway particulier.

**Réponse :** Gateway Abstraction + explicit adapters + contract validation.

### H. Sécurité
- secrets dans logs ;
- mauvaise cible de repository ;
- working tree inattendu ;
- outils trop permissifs ;
- replay/forgery d'événements.

**Réponse :** explicit execution boundaries + authorization gate + sanitization + adversarial tests.

---

## EN — Problem map

### Intelligence
Model selection can ignore task requirements and live availability. Catalog knowledge must be separated from runtime truth.

**Response:** Model Intelligence, Capability Matching and Explainable Routing.

### Economics
Token waste, unnecessary retries, overpowered models, uncertain free quotas and hidden failure costs reduce efficiency.

**Response:** Model Economy, budgets and context optimization.

### Reliability
Fallback can misclassify failures; providers can disappear; streams can stall; long requests can time out; retries can loop.

**Response:** failure classification, circuit breakers, streaming state machines and resilience policies.

### Agent execution
Agents need explicit authority, scoped permissions, checkpoints and resumability.

**Response:** Agent Control, Permission Gate, Task Graph and checkpoints.

### Verification
“Done” is not proof. Tests, acceptance criteria and evidence must be explicit.

**Response:** Mission Contract, Validation Engine, Mission Verifier and Evidence/Proof.

### UX
The chat should not be the only mental model. The Command Center should expose mission state, decisions, execution and proof without duplicating runtime truth.

### Integrations and security
Provider/gateway abstraction, contract validation, explicit repository boundaries, secret hygiene and adversarial testing are mandatory foundations.

---

# North-star question / Question centrale

**FR:** « Comment accomplir une mission réelle avec les ressources suffisantes, au coût et au risque maîtrisés, puis prouver qu'elle est terminée ? »

**EN:** “How can we accomplish a real mission with sufficient resources, controlled cost and risk, and then prove that it is complete?”
