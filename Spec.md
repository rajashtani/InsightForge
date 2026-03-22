# The Tripod Team: Scaling Software Engineering with SDD and AI Agents

## TL;DR

**Move beyond casual “vibe coding”** (loose natural-language prompting + trial-and-error with AI) toward **Specification-Driven Development (SDD)** — using structured, versioned Markdown specs as the central “contract” to guide AI agents reliably.

**Core proposal** — the **Tripod Team** (exactly 3 humans: Intent Leg (BA + Architect) for specs, AI Wrangler for execution, Oracle (SME) for truth and validation) + a simple `.sdd/` folder structure that separates foundations/rules, role-specific agent instructions, active feature specs, and archives completed work to prevent context poisoning.

**Result** — Lean, auditable, compliance-friendly agentic workflows that deliver traditional team throughput with green spec-derived tests + expert sign-off, especially suited for regulated and high-stakes domains.

It builds on 2025–2026 ecosystem momentum (GitHub Spec Kit, Tessl, Kiro, Augment Code) rather than claiming to invent SDD from scratch — just an opinionated, lightweight pattern emphasising adversarial agent roles (Architect/Developer/Reviewer), context hygiene via archiving, and minimal human ceremony for production-grade outcomes.

## The Tripod Team

*One Tripod delivers what a two-pizza team used to deliver, with an auditable spec trail.*

**What a Tripod is:**
A self-contained, three-person delivery unit built around AI-assisted development. Each leg owns a distinct phase of the spec-to-production lifecycle.

**What a Tripod is not:**
A traditional scrum team. There is no backlog grooming, no story points, no sprint ceremonies. The spec is the plan. Green tests are the progress metric.

**A Tripod is done when:**

- Specs are approved and version-controlled
- All BDD scenarios and tests derived from specs are green
- The Oracle has signed off output against real-world truth

```
            The Anchor
        (Intent Leg — BA/Architect)
              △
             /|\
            / | \
           /  |  \
          /   |   \
         ▽         ▽
   AI Wrangler      Oracle
  (Build Leg)    (Truth Leg — SME)
```

### Why Tripod Works: The Power of Three in Agentic Development

The Tripod is not arbitrary minimalism — it is the smallest stable unit that delivers full-cycle accountability, adversarial rigour, and traditional throughput while keeping ceremony near-zero. Three humans strike an optimal balance in the agentic era:

**Minimal viable separation of concerns without overlap waste.**
With only two people, roles blur: one person ends up owning both spec and truth, eliminating the critical adversarial check that catches logic gaps, compliance violations, or hallucinated shortcuts. Three creates clean ownership — Intent Leg defines the contract, Build Leg executes via agents, Truth Leg acts as oracle — mirroring proven patterns like red-team/blue-team or author/editor/reviewer workflows, but at extreme minimal scale.

**Cognitive diversity and error resistance without coordination tax.**
Research on group problem-solving (e.g., Laughlin’s studies on intellective tasks) shows groups of three often outperform both pairs and larger teams in verifiable-truth tasks: enough perspectives to surface edge cases, but too few for factionalism or diffusion of responsibility. In regulated domains (finance, healthcare, aerospace), where zero-loss tolerance is mandatory, this triad reliably catches what a solo expert or duo might miss — without the communication overhead that kicks in at five or more people.

**Leverages agentic amplification to match or exceed two-pizza throughput.**
Amazon’s “two-pizza” rule (5–8 people) aimed to minimise bureaucracy and maximise ownership for fast innovation. In 2025–2026 agentic workflows, frontier models and structured SDD hand most implementation and even design iteration to agents. This shifts the human bottleneck from writing code to steering, validating, and governing. A three-person pod delivers what a traditional 8–10 person scrum team used to, but with full traceability and green spec-tests.

**Human-in-the-loop remains meaningful and sustainable.**
Full autonomy is still experimental. AI-assisted coding tools can produce transient velocity gains while simultaneously introducing persistent increases in code complexity and static analysis warnings — exactly the kind of silent quality drift that the Truth Leg exists to catch. Three humans keep oversight distributed yet lightweight: no single point of failure, no burnout from one person juggling all gates, and easy scaling via parallel Tripods sharing common foundations.

In short: a two-person unit is too fragile — it lacks a genuine adversarial check. Four or more people reintroduce coordination overhead. Three is the sweet spot: stable, diverse, auditable, and maximally amplified by agents.

|Team Size|Pros                                                                                                    |Cons in Agentic / Regulated Context                                                                    |Fits Tripod Model?               |
|---------|--------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------|---------------------------------|
|1–2      |Ultra-fast decisions; minimal communication overhead                                                    |No real adversarial validation; single point of failure; hard to separate spec vs truth ownership      |No                               |
|3        |Clean separation of concerns (Intent / Build / Truth); built-in adversarial check; low coordination cost|Slightly more alignment needed than solo/duo; requires competent people in each leg                    |**Yes** (optimal)                |
|5–8      |More parallel bandwidth; can handle larger cross-cutting features                                       |Coordination tax returns; harder to maintain context hygiene; diffusion of responsibility risk         |Overkill for most scoped features|
|10+      |Can tackle very large systems and multiple services                                                     |Bureaucracy and ceremony re-emerge; significant process loss; compliance traceability becomes expensive|No                               |

## A Closer Look at Each Leg

### The Intent Leg: The Anchor

The Intent Leg owns the contract. The BA and Architect together are responsible for defining what the system is supposed to do, why it exists, and what failure looks like.

The BA and Architect are combined into one leg deliberately — not because they are the same person, but because spec quality requires both business translation and technical authority simultaneously. A BA without architectural grounding writes vague requirements an agent cannot act on deterministically. An Architect without BA discipline produces expertise that never gets structured into something shippable.

**What the Intent Leg owns:**

- `requirements.md` — the feature contract, including BDD-style Given/When/Then acceptance scenarios
- `design.md` — the architectural plan, drafted by the Architect Agent and approved by the Intent Leg before implementation begins
- The decision to consult the Oracle on domain-specific or compliance-sensitive requirements before the spec is locked

**On SME consultation:**
The Oracle (SME) is a stakeholder, not a permanent seat on the Intent Leg. The Intent Leg consults the Oracle on a need basis — typically when requirements touch regulated domain logic, edge cases that require deep field knowledge, or compliance constraints that cannot be inferred from `foundations/product.md` alone. This consultation happens before `requirements.md` is finalised, ensuring the Oracle’s knowledge is embedded in the spec rather than surfacing as a surprise during review.

**Agent on this leg:**

- **Architect Agent** — designs the solution architecture and validates `design.md` against `foundations/engineering.md`; prohibited from writing implementation code

### The Build Leg: AI Wrangler

The AI Wrangler is the most novel — and most load-bearing — human role in the Tripod. This is not a developer who occasionally uses an AI assistant. The Wrangler is a full-time agent orchestrator: closer to an air traffic controller than a coder.

The Wrangler’s responsibilities divide into two explicit categories:

**Context Stewardship** (structural, partially tool-assisted):

- Maintaining the `.sdd/` Specification Vault: keeping `foundations/` read-only, moving completed features to `archive/`, and ensuring `manifest.md` accurately reflects the current project state
- Preventing context poisoning by enforcing session boundaries — no agent carries state from a previous feature into a new one
- Monitoring `tasks.md` as a real-time progress heartbeat and escalating blockers to the Intent Leg before they become stalls

**Orchestration Judgment** (irreducibly human, high-stakes):

- Deciding when Architect Agent output is architecturally sound enough to hand off to the Developer Agent
- Recognising when a Revision Order from the Reviewer Agent reveals a spec ambiguity rather than an implementation failure, and routing it back to the Intent Leg instead of looping the Developer Agent indefinitely
- Detecting context drift mid-feature: when an agent’s outputs begin diverging from the spec in subtle ways that pass automated checks but violate intent

Context stewardship is mechanical work that better tooling will increasingly absorb. Orchestration judgment is irreducibly human — it requires understanding the intent behind the spec, not just its text. As the role matures, a skilled Wrangler should be spending the majority of their time on judgment calls, not on vault hygiene.

Inside the Build Leg, two AI agents run in a structured adversarial loop:

- **Developer Agent** — implements strictly against the architectural design and the approved spec; any off-spec deviation triggers an automatic Revision Order
- **Reviewer Agent** — validates unit tests derived from BDD scenarios, checks code quality, coverage thresholds, and linting standards; issues Revision Orders on failure; explicitly instructed to find failures, not confirm success

The Wrangler does not fix failures themselves. They route failures to the correct leg — back to the Intent Leg if the spec is the problem, back to the Developer Agent if the implementation is the problem. This separation keeps accountability clean.

**What the Reviewer Agent does and does not own:**
The Reviewer Agent is responsible for mechanical validation: unit test pass/fail, code coverage thresholds, static analysis, and linting. It operates on criteria derived directly from the BDD scenarios in `requirements.md`. It does not assess real-world correctness, domain logic validity, or business intent — those are the Oracle’s domain.

### The Truth Leg: Oracle

The Oracle is the SME and the human quality gate that no AI agent can replace. They bring field knowledge and domain depth — knowing which edge cases matter in production and when technically correct output is still wrong for the real world.

**What the Oracle owns:**

- Review and approval of BDD scenarios in `requirements.md` before the spec is locked, ensuring acceptance criteria reflect real-world validity and not just syntactic completeness
- Final adversarial review of implemented features against domain truth: does this actually behave correctly in the real world, not just against the tests?
- Security and compliance checks specific to the domain
- Issuance of Revision Orders when real-world validation fails, with specific references to the failed requirement or acceptance scenario

**The sign-off hierarchy:**
Reviewer Agent green is necessary but not sufficient. The Oracle signs off only after automated validation is clean and real-world correctness is confirmed. This separation of automated proof and human judgment is what makes the Tripod model suitable for regulated environments.

When the Oracle signs off, the feature folder moves from `specs/` to `archive/` — clearing the context window for the next feature.

## The Workflow: Planning, Implementation, Review

The Tripod operates across three phases, with explicit human checkpoints at each transition.

|Phase         |Owner     |Activity                                                                                                     |Artifact Produced                                  |
|--------------|----------|-------------------------------------------------------------------------------------------------------------|---------------------------------------------------|
|Planning      |Intent Leg|Analyse requirements with AI; consult Oracle if needed; write BDD acceptance scenarios; validate architecture|`requirements.md` (with BDD scenarios), `design.md`|
|Implementation|Build Leg |Orchestrate Developer Agent; Reviewer Agent validates unit tests and code quality; Wrangler routes failures  |`tasks.md` (live), production code                 |
|Review        |Truth Leg |Oracle validates domain logic and real-world correctness; issues Revision Orders or signs off                |Revision Orders or Oracle sign-off                 |


> **Shared ownership checkpoint:** `specs/[feature]/design.md` is drafted by the Wrangler’s Architect Agent, then approved by the BA/Architect before implementation begins. This is a deliberate governance gate — the Intent Leg confirms the proposed solution matches original intent before a single line of implementation code is written.

### Planning Phase

The Intent Leg analyses requirements alongside an AI agent to produce formal Markdown specifications. `requirements.md` contains Given/When/Then BDD acceptance scenarios that serve as the contract for both the Reviewer Agent (unit test derivation) and the Oracle (real-world sign-off). `design.md` is validated by the Architect Agent against `foundations/engineering.md`.

Before the spec is locked, the Intent Leg consults the Oracle if the feature touches regulated domain logic, compliance constraints, or field-specific edge cases. The Oracle’s input is embedded into the BDD scenarios at this stage — not discovered during review.

No implementation begins until both `requirements.md` and `design.md` are signed off by the Intent Leg.

### Implementation Phase

The Build Leg orchestrates the Developer Agent to generate production code strictly based on the finalised spec. The Developer Agent updates `tasks.md` continuously — giving the Wrangler a real-time heartbeat of what is complete, in progress, and blocked. No standup required.

The Reviewer Agent runs in parallel, validating unit tests derived from BDD scenarios, checking coverage thresholds, and enforcing code quality standards. When it finds failures, it issues Revision Orders to the Developer Agent with specific references to the failing test or standard.

The Wrangler monitors for two failure types: implementation failures (routed back to the Developer Agent) and spec ambiguities (routed back to the Intent Leg). Distinguishing these is the Wrangler’s most critical judgment call.

### Review Phase

The Oracle runs the final review: adversarial validation of domain logic, real-world correctness, security and compliance checks, and edge-case probing against the BDD scenarios they helped define. The Oracle does not fix failures — it issues Revision Orders with specific references to the failed acceptance scenario or engineering standard.

When the Oracle is satisfied, the feature folder moves from `specs/` to `archive/`. This is not just bookkeeping — it is the mechanism that prevents completed feature context from contaminating subsequent agent sessions.

## Specification Structure

The Specification Vault is the structural backbone of the Tripod workflow — a lightweight folder convention that keeps every agent’s context clean, purposeful, and auditable. Think of it as dividing work into specialised desks: the Architect Agent reads `foundations/` and `design.md`; it never needs to see `tasks.md` or raw code. This prevents context poisoning at the architectural level.

```
.sdd/
├── manifest.md                  ← The "front desk" for all agents and humans
├── foundations/                 ← Global rules (read-only for agents)
│   ├── product.md               ← The "What & Why" (business constraints, compliance)
│   └── engineering.md           ← The "How" (tech stack, patterns, standards)
├── roles/                       ← Agent persona instructions
│   ├── architect.md             ← "Validate design.md against engineering.md; no implementation code"
│   ├── developer.md             ← "Execute tasks.md strictly; never go off-spec"
│   └── reviewer.md              ← "Find failures; validate unit tests and code quality; issue Revision Orders"
├── specs/                       ← Active feature work
│   └── [feature]/
│       ├── requirements.md      ← The contract: BDD scenarios + acceptance criteria (owned by Intent Leg, reviewed by Oracle)
│       ├── design.md            ← The plan: architecture (drafted by Architect Agent, approved by Intent Leg)
│       └── tasks.md             ← Living progress document (updated by Developer Agent)
└── archive/                     ← Completed features (moved here after Oracle sign-off)
```

Four design decisions matter more than the folders themselves:

**`foundations/` is read-only for agents.** Only humans update global standards. This prevents an agent from silently learning a bad pattern from a previous feature and propagating it forward. The foundation version in `manifest.md` signals when agents must re-read this folder.

**`roles/` makes agent behaviour deterministic across sessions.** The Reviewer Agent is explicitly instructed to find failures and validate unit tests and code quality — not confirm success. The Architect Agent is prohibited from writing implementation code. Every session starts with the same ground rules regardless of what happened in the previous one.

**`archive/` solves context poisoning.** Completed specs are not deleted — they are moved. Leaving resolved requirements in the active context causes the AI to re-apply old constraints to new features. Moving to `archive/` is the formal signal that a feature is done and its context should not be loaded.

**`manifest.md` is the front desk.** Instead of an agent crawling the entire directory to understand project state, it reads the manifest first. This single file eliminates most “where am I in this project?” confusion at the start of every session.

## The manifest.md Schema

The manifest answers the three questions any agent or human asks at the start of a session: where are we, what is active, and what just happened. It is updated exclusively by the Wrangler at every phase transition. Agents read it — they never write it.

```markdown
# Project manifest

## Project
- **Name**: [project name]
- **Foundation version**: 1.0        ← bump when foundations/ changes
- **Last updated**: YYYY-MM-DD

## Current state
- **Phase**: Planning | Implementation | Review   ← exactly one at a time
- **Active spec**: specs/[feature-name]/
- **Status**: On track | Blocked | Awaiting sign-off

## Active agents
- **Architect Agent**: active | idle
- **Developer Agent**: active | idle
- **Reviewer Agent**: active | idle

## Last handoff
- **From**: Intent Leg
- **To**: Build Leg
- **Action**: requirements.md and design.md approved — implementation may begin
- **Date**: YYYY-MM-DD

## Blockers
- [None] or short description and who needs to resolve it

## Completed features
- archive/feature-name/   — Oracle sign-off YYYY-MM-DD
- archive/feature-name/   — Oracle sign-off YYYY-MM-DD
```

**Foundation version** is critical for agent reliability. If `engineering.md` or `product.md` changed since the last session, an agent operating on a cached understanding of the rules will silently apply stale constraints. A version bump signals that agents must re-read `foundations/` before proceeding.

**Exactly one phase at a time** is a hard constraint. The temptation is to track phase per feature, but that reintroduces the complexity the manifest exists to eliminate. One active spec, one phase — anything completed moves to `archive/`.

**Last handoff in plain language** rather than a status code. Agents orient faster from a sentence than from a state value, and it doubles as a lightweight human audit trail without requiring a separate changelog.

**Completed features listed inline** so the Architect Agent can see what patterns already exist in the codebase without loading stale spec files into context.

## Revision Orders

A Revision Order is the formal feedback artifact issued when a feature fails validation. It ensures failures are traceable, actionable, and routed to the correct leg.

**Who issues Revision Orders:**

- The **Reviewer Agent** issues Revision Orders for unit test failures, code coverage gaps, linting violations, and code quality standards. These are mechanical failures with deterministic criteria.
- The **Oracle** issues Revision Orders for domain logic failures, real-world correctness violations, security and compliance gaps, and BDD scenario failures that automated tests did not catch. These require human judgment.

**What a Revision Order contains:**

- Reference to the specific BDD scenario, acceptance criterion, or engineering standard that failed
- Description of the observed behaviour vs. expected behaviour
- Routing: is this an implementation failure (back to Developer Agent) or a spec ambiguity (back to Intent Leg)?

**The Wrangler’s routing responsibility:**
When a Revision Order arrives, the Wrangler determines whether the failure is an implementation problem or a spec problem. Implementation failures loop back to the Developer Agent. Spec ambiguities route to the Intent Leg for clarification before implementation resumes. This distinction is the most critical judgment call the Wrangler makes — getting it wrong either loops the Developer Agent indefinitely or masks a genuine spec gap.

## Scaling: Parallel Tripods

For larger systems, multiple Tripods operate in parallel, each owning a distinct bounded context or service boundary. Parallel Tripods share a common `foundations/` — the same `product.md` and `engineering.md` apply across all teams. Each Tripod maintains its own `specs/` and `archive/`.

Coordination between Tripods is handled at the `foundations/` level: shared engineering standards and product constraints are the integration contract. Cross-cutting concerns that affect multiple Tripods are resolved by updating `foundations/` (with a version bump) rather than by adding inter-team ceremony.
