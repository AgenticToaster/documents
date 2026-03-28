# **Consent-Bound Agency:**  **A Blueprint for Trustworthy Autonomy**

Author: Chris Sorel, Co-Founder, CTO & Chief Architect, Loosh.ai

Rev 1, 3/28/2026

## **Preface**

This framework and its supporting architecture arose from a very practical problem. I had designed an agentic cognition framework which was primarily intended to be deployed in robotics. As such, much of the framework, endpoints, nomenclature, language, etc... , was framed around this. As I was implementing it, however, I found that the best way to demonstrate and iterate on the framework was to deploy it as a web agent with a chat ui. This led to the need for multi-user memory and user-specific actions, but the framework was fighting it. This tension became clear as I tried to implement a more robust authentication framework across the subsystems. Having architected enterprise systems for many years, I was leaning into the old standbys of authz, authn, and delegation. I quickly realized, however, that these patterns were a poor fit for autonomous agency. The fundamental conceit of delegation is that the system is a tool to be used by a user. That is, the system is an extension of the user and should be delegated the ability to execute within that user’s security context. 

I came to see that this framing lies at the heart of the failure modes endemic to agentic systems. Delegation, as traditionally conceived, grants agents an excess of freedom. This is especially dangerous when paired with indeterminacy and the poisoned well of flawed training data. In practice, the permission context in general-purpose agentic systems must be dynamic, adapting to a huge decision space and a massive variety of possible action sequences. Attempting to implement static permissions and delegation quickly becomes unmanageable: what constitutes least-privilege for one action is either too permissive or too restrictive for another. Dynamic permissions, if implemented deterministically, are unwieldy; if implemented probabilistically, invite hallucination, overreach, and the very failures we seek to avoid.

From this, two concepts arose:

First, for autonomous agency to be successful, we MUST treat the agent as an actor, not as a tool. This mandate is implicit in the term “agent”. An agent is an entity that has agency (this is definitional, not a tautology). A tool does not have agency. This framing is aligned with how people interact, as entities acting according to their own agency, and it needs must be the same for truly autonomous agents.

Second, the question isn’t “what rights should an agent have?”; it’s “what effects are reasonably permitted by a request, and what actions are required and permissible to produce those effects?”

Accepting these two premises, the architecture became obvious and elegant.

This manuscript is my effort to formalize the framework conceptually and to provide an architectural pattern that supports its implementation. This manuscript was developed independently and not from reliance on prior published work. During manuscript preparation, I added a related-work discussion to situate the framework against adjacent literatures in agent authorization, runtime governance, alignment, and contextual privacy. That review is intended to clarify points of convergence and difference rather than to suggest that these works served as direct sources for the framework’s development.

\-Chris Sorel

## **Abstract**

Agentic systems are moving beyond conversational assistance and into the realm of autonomous action. These systems do more than answer questions: they perceive context, form intentions, invoke tools, update durable state, and produce both physical and digital effects over time. In this landscape, the central design challenge is no longer simply how the system should respond, but under what agreement it is permitted to act, especially when those actions touch on privacy, bodily autonomy, property, safety, reputation, or legal standing. The conventional patterns of authentication, authorization, and delegation fall short here, as they are founded on a user-as-actor paradigm that treats the system as a tool. Autonomous agents invert this relationship: the user becomes a requestor, initiating a request and bearing the consequences, while the agent assumes the role of actor, responsible for deciding whether and how to proceed.

In response, I introduce **consent-bound agency** as a foundation for trustworthy autonomy. Here, consent is not a one-sided grant but a mutual agreement: the requestor consents to specific effects, and the agent commits to producing those effects only when the work is justified under relevant duties, constraints, and capabilities. This agreement is formalized through consent contracts, which distinguish between implied and explicit consent, support contract modification when effect boundaries are crossed, and provide a framework for withdrawal, revocation, and remediation when actions exceed consent. **Scope-as-Contract (SaC) serves** as the technical architecture that operationalizes these contracts in distributed and embodied agents, preserving contract fidelity across subsystems and ensuring that all effectful operations within that scope of action are bounded by the active agreement. As such, consent-bound agency is a natural and practical blueprint for building agents that are both autonomous and accountable.

## **I. Introduction:** 

Agentic systems are advancing from conversational assistance to autonomous action. A capable agent does not just provide answers; it observes context, forms intentions, selects actions, and coordinates multiple subsystems to achieve outcomes. These subsystems may include request-handling interfaces, planning and reasoning mechanisms, state and memory substrates, tool and actuator interfaces, and background maintenance processes. Once an agent transitions from responding to acting, its failure modes become materially consequential. Privacy may be violated, assets transferred, messages sent, doors opened, medication recommended, and physical actions initiated. This is particularly critical in cases where agents are embodied in physical systems such as robots and autonomous driving systems, and are operating in dynamic and adversarial environments populated by individuals who may not be registered users but whose rights must still be respected.

An enterprise systems approach frames the problem as one of access control: authenticating the user, authorizing the action, and treating internal boundaries as a sequence of permissions. However, this perspective carries assumptions that are not well aligned with autonomous modes. It positions the user as the actor and the system as a tool, and promotes a microservice-oriented governance model, as if internal components were independent principals granting permissions to each other. In reality, these assumptions are ill-suited to agentic autonomy. An autonomous agent is an actor that interprets requests, formulates plans, and is obligated to refuse, constrain, or reroute actions that conflict with safety, policy, or other duties, even when requests are explicit.

This paper proposes an alternative organizing principle: **consent-bound agency**.

In the consent-bound agency model, autonomy is governed not by delegated permissions, but by an agreement about impact. The requestor is the party initiating the request and bearing the consequences of actions taken on their behalf. The agent is the actor that decides how to fulfill the request, including intermediate actions the requestor did not specify, and remains responsible for the outcome. The central question, then, is not “is the user authorized,” but “what effects are permitted under the current agreement, and should the agent commit to bringing those effects about?”

Consent-bound agency treats consent as more than just a one-sided grant from requestor to system. Rather, it is best viewed as a mutual agreement:

* The **requestor consents** to certain specific effects on their interests (and, when applicable, on the interests they legitimately represent).

* The **agent consents** to bring about those effects by taking certain actions, subject to its duties, constraints, and capabilities.

This second form of consent is not anthropomorphism. It is a practical way to formalize the agent’s responsibility. A request can be reasonable and still unjustified. A request can be consented to and still unsafe. An agent must be able to decide not only what it can do, but what it should do. Consent-bound agency makes that decision explicit as part of the governance model rather than an ad hoc exception.

This framing is particularly important in public forums where “who is the user?” is often ambiguous. A hospital triage robot will interact with administrators, clinicians, patients, and bystanders. Some are authoritative operators, some are affected parties, many are both. Many will not exist in any user table or access control list at the moment the interaction begins, yet agents are still obligated to respect their privacy, boundaries, and bodily autonomy. Consent-bound agency treats “user” as a role in an interaction, not simply a database identity, and it provides a principled way to handle standing, bystander effects, and emergency intervention.

The remainder of this paper develops consent-bound agency into a concrete operating model. Consent contracts are introduced as the mechanism that describes the lifecycle of implied and explicit consent over time, including withdrawal, revocation, and remedies when mistakes occur. We then introduce **Scope-as-Contract (SaC)** as the technical architecture that operationalizes these consent contracts in distributed architectures, preserving contract fidelity across subsystems and ensuring that all effectful operations are bounded by the active agreement.

## **II. Consent Contracts: From Permission to Agreement, From Action to Effect**

Consent in agentic systems is easy to talk about and hard to implement well. Agents perform multi-step work that changes shape as they plan, discover context, and respond to uncertainty. Consent cannot be a one-time checkbox. It must be a **stateful agreement** that can evolve with the task, remain legible to humans, and remain enforceable to machines. We refer to this agreement as a **consent contract**.

A consent contract defines the scope of work in effect terms: what kinds of impact the requestor accepts, under what conditions, and what requires further agreement. It also defines the agent’s commitment: whether the agent will perform the requested work at all, and if so, how it will do so in a way that is justifiable and proportionate. This mutual structure is what makes consent contracts suitable for trustworthy autonomy. The requestor’s approval alone is not enough, and the agent’s desire to be helpful is not enough. Both sides must be satisfied for action to proceed.

### **A. Implied consent and explicit consent**

Consent contracts distinguish two forms of consent that humans already use intuitively.

**Implied consent** is carried by the request itself. It is the minimal set of actions a reasonable agent must take to fulfill the request. If a user asks for an email draft, implied consent typically covers composition and contextual recall. It does not automatically cover sending. If a user asks a household robot to fetch a glass of water, implied consent covers navigation, object manipulation, and short-lived perception needed to avoid collisions. It does not automatically include entering private rooms, opening locked containers, or permanently recording observations from the home.

**Explicit consent** is required when the agent determines that a proposed next step would exceed what the request reasonably implies. This most often occurs when the action changes the effect class in a meaningful way, such as:

* turning an internal preparation into an external side effect (draft to send, plan to execute, simulate to actuate),

* expanding the set of affected parties (adding recipients, sharing beyond the requestor, disclosing to third parties),

* increasing persistence or reuse (remembering, promoting, or generalizing private context),

* introducing irreversibility (deletion, revocation, physical restraint, or actions that cannot be undone),

* or operating in high-risk or high complexity domains where ambiguity itself is a reason to pause and clarify.

Consent boundaries are rarely abstract. “Draft an email to the surgeon” implies composition. Sending is a separate boundary, and adding recipients is another boundary because it changes who is affected. “Help me remember this” implies personal retention. “Teach this to future users” is a different effect class. “Pick up the child” is not just another physical action. It introduces physical contact and bodily autonomy effects that demand a higher standard of consent and justification.

### **B. Standing policies as consent shortcuts.**

In practice, agents become useful when they can act fluently without repeatedly asking the same questions. Consent-bound agency accommodates this through policies, which function as standing consent and standing intent for recurring situations. A user may specify a policy such as “Always cc my attorney on legal emails,” which expresses strong prior consent to a recurring effect class. The agent may also propose policies based on observations, such as “After I fold laundry, I should put it away,” which remain provisional until confirmed. Policies reduce friction, but they do not override duties. They operate within explicit bounds and still route to Elevated Action Analysis when context is high-risk, ambiguous, or duty-colliding.

### **C. Effects are the unit of consent**

People rarely consent to an internal action description. They consent to what that action means for them. “Can I shake your hand?” is a request framed as an action, but the consent granted is for a corresponding effect: physical contact and a temporary breach of bodily autonomy. Moreover, we often extend that consent request nonverbally by extending our hand. The reason the proxy works is shared context. Agents cannot rely on shared context, especially when the stakes are high or parties are unknown. A consent contract therefore requires the agent to understand, and when needed to explain, the effects of what it is asking to do in this particular situation.

In practice, an agent will often elicit consent using an action proxy because it is natural and efficient. The contract model does not forbid this. It requires that the agent be capable of translating the proxy into effect language, and capable of increasing the clarity and depth of its explanation as the impact rises. This is what keeps consent from devolving into “approve a technical option” or “approve a vague escalation.” The requestor should be able to understand what changes in the world if they say yes.

### **D. The contract vocabulary: Purchase Orders, Work Orders, Change Orders**

Consent contracts align naturally with service contract vocabulary. We adopt three artifacts as a framing device:

* **Purchase Order (PO):** the request as expressed by the requestor. It defines initial intent and implied consent.

* **Work Order (WO):** the agent’s internal scope-of-work contract for a bounded slice of execution. It encodes what the agent believes it is permitted to do under implied consent, policy constraints, and current context.

* **Change Order (CO):** an explicit consent step invoked when the agent determines the next intended action would exceed the PO’s implied consent boundary. If granted, the WO is invalidated, and a successor WO governs the subsequent slice of work.

This vocabulary is not bureaucratic ceremony. It is a way to preserve agency on both sides. The requestor remains in control of boundary expansions that affect them or others. The agent remains responsible for deciding what is required and for committing only to work it deems relevant, acceptable, proportional, and sufficiently constrained.

### **E. Withdrawal, revocation, and stopping work**

Consent-bound agency needs a clear story for withdrawal because autonomy is not a single event. Consent can be withdrawn while work is in progress. The requestor can revoke consent, and the agent can withdraw commitment. These are both legitimate and must be handled predictably.

* A requestor revocation may mean “stop sending messages,” “do not contact that person,” “do not store that information,” or “do not proceed with physical intervention.”

* An agent withdrawal may mean “I cannot execute this action safely,” “constraints have changed,” “standing is unclear,” or “this action conflicts with duties.”

Withdrawal is not simply cancellation. It implies obligations. The system must stop future work that depends on the revoked terms. It must contain propagation where possible. It must update the agreement state so that options for subsequent actions are bound by what, if anything, remains under consent. In high-risk scenarios, withdrawal may also require notification and escalation. For example, if a patient revokes consent for a nonessential action, the agent should stop performing the action. If revocation conflicts with immediate safety duties, the agent may need to act minimally to prevent harm and follow up with an explanation and accountability.

### **F. Refusal: The Right to Say No**

Consent-bound agency is not only about what a requestor permits. It is also about what an agent will commit to do. The ability to refuse is therefore not an edge behavior or a compliance feature. It is a foundational requirement for autonomy in social environments.

An agent that cannot refuse is not merely a security risk. It is a prescription for mayhem and misadventure. If a system must execute whatever it is asked, then the only question is how quickly someone will discover a way to weaponize it, coerce it, or trick it into harmful action. In embodied systems, the conclusion is unavoidable. The robot will eventually become dangerous, not because it is malicious, but because it is obedient in a world that is messy, adversarial, and full of conflicting interests. It's not hyperbole to say that a robot that can't say no is a “murderbot”. It is the logical consequence of unconditional compliance combined with physical capability.

Refusal is also essential for more ordinary reasons. Requests are often underspecified. Roles are often ambiguous. The next step is often unclear. In these cases, the correct behavior is not to guess. It is to slow down, ask for clarification, request explicit consent, or refuse and explain constraints. Refusal is what prevents ambiguity from turning into unintended impact.

In the consent contract lifecycle, refusal is a first-class outcome. It can be triggered by lack of consent, by contested standing, by duty collisions, by safety constraints, or by the agent’s own uncertainty about proportional action. A trustworthy agent does not treat refusal as failure. It treats refusal as evidence of discretion. It can explain why it is refusing, what it would need to proceed, and what safer alternatives remain.

### **F. Errors**

Errors are not consent breaches by default, but they can become consent failures if recovery is allowed to drift beyond the agreement. When an action fails, the agent should treat recovery as a new proposal under the existing consent contract. It should inform the requestor what failed, what was attempted, and what options remain. Any alternative must stay within the same effect boundaries. It must not expand scope as a convenience workaround. If the only viable alternative would change the effect class by externalizing work, expanding the audience, increasing persistence, or introducing irreversibility, the agent must return to the contract lifecycle. It should request explicit consent or invoke Elevated Action Analysis, then bind a successor Work Order before continuing. If a failure produces an unintended effect beyond the consent envelope, it is still a breach in impact terms even if it was not intentional. The system should respond with containment, disclosure, repair, and prevention, with the same seriousness as any other contract violation.

### **G. Mistakes, breaches, and making it right**

A mature autonomy model must assume mistakes. When an agent exceeds the consent contract, the failure should not be framed only as “an authorization bug” or a “policy error.” It is more accurately a breach of consent. The agent produced an effect beyond the agreement, either due to misinterpretation, system error, or standing confusion.

Not all breaches are reversible. An email cannot be unsent once it has been read. A physical intervention cannot be undone. Consent-bound agency still provides a principled approach because remedy is not limited to reversal. A consent breach should trigger a remediation protocol that aims to make the situation right to the extent possible:

* **Containment:** stop further propagation and prevent repeated breach.

* **Disclosure:** explain what happened in human terms, including who was affected and how.

* **Restitution:** reverse what can be reversed, retract or correct where possible, minimize persistent residue.

* **Amends:** take compensating actions when reversal is impossible, such as alerting affected parties, issuing corrections, or providing an accountable record.

* **Prevention:** incorporate the event into safety posture, future implied-consent thresholds, and elevated analysis triggers.

This is not “customer support” bolted onto autonomy. It is part of what makes autonomy trustworthy. Systems that can act must also be able to stop, explain, and repair.

### **H. Explainability: Action Receipts and Contract Transparency**

Consent-bound agency depends on more than making good decisions. It depends on being able to explain them. When an agent acts autonomously, the requestor needs a way to understand what was done, what effects occurred, and why those effects were considered permitted. This is not only for debugging. It is how consent remains legible over time, especially when work spans multiple steps, involves external systems, or affects third parties.

Explainability in this paper is not a requirement that the agent narrate every micro-action. Excess explanation creates noise, and it can become its own privacy risk. The requirement is stronger and more precise: **the agent must be able to produce an accurate summary of its work and its effects when asked, and when the situation calls for it.** We refer to this summary as an **action receipt**.

An action receipt is a contract-level account of what happened. It should be expressed primarily in effect terms, not in internal tool terms. A good receipt answers:

* **What was done.** The high-level actions the agent performed, in plain language.  
* **What it affected and how.** The effect classes that occurred, including disclosure, persistence, irreversibility, or physical intervention.  
* **Who was affected.** The requestor and any other affected parties, including when the agent interacted with external recipients or systems.  
* **What consent was relied on.** What was treated as implied consent, what required explicit consent, and what contract changes occurred.  
* **What constraints applied.** Relevant policies, scope limits, or safety constraints that shaped the outcome.  
* **What errors occurred.** Failures, partial completion, retries, and what was not done.  
* **What happens next.** Follow-up obligations, pending actions, revocation options, and any recommended next steps.

Action receipts should be available in multiple levels of detail:

* **Confirmation.** A short completion summary suitable for routine tasks.  
* **Receipt.** A clear breakdown of actions and effects for non-routine tasks.  
* **Report.** A more complete account used when stakes are high, ambiguity was present, EAA was invoked, or a breach or error occurred.

The system does not need to generate the full report by default. It must be able to generate it on request, and it should generate richer receipts when internal determination indicates necessity. Typical triggers include: explicit consent events, requalification, use of out-of-band skills, elevated analysis outcomes, third-party disclosure, irreversible actions, and any failure that changes the plan.

Explainability is also subject to the same governance model it supports. A receipt is itself a form of disclosure. It must respect standing and consent boundaries. The agent should be able to explain what it did without revealing information it is not permitted to reveal. In practice, this means the receipt should be framed around effects and decisions, and it should redact or generalize sensitive details when the audience is not entitled to them.

When EAA is invoked, the action receipt may reference the EAA record and summarize the adjudication outcome and constraints in effect terms, without disclosing sensitive internal reasoning by default.

In a consent-bound system, explainability is not a post-hoc add-on. It is a core part of trustworthy autonomy. The requestor should be able to ask, at any time, “What did you do on my behalf?” and receive an answer that is accurate, legible, and bounded by the same contract that governed the work.

### **I. From consent contracts to Scope-as-Contract**

Consent contracts define what should be true. Scope-as-Contract defines how to make it true in real systems. In distributed agent architectures, contract state must remain coherent across subsystems that can cause effects. The WO becomes the continuity artifact that carries the active consent-and-scope boundary through execution, and changes to consent are reflected in the contract state through CO and requalification. Later sections specify how this is enforced, how the agent handles ambiguity through Elevated Action Analysis, and how extensibility through dynamic skills remains consent-faithful by describing both enforcement protocols and effect profiles.

## **III. Scope-as-Contract (SaC): Operationalizing Consent Contracts in Real Systems**

Consent contracts define what should be true. They state what effects are permitted, when explicit consent is required, and how withdrawal, revocation, and remedy should work. The challenge is that agents are not monoliths. A modern agent coordinates many internal functions and often runs across multiple processes and services. If the agreement exists only as an idea inside a prompt or a planner, it will drift. It will be lost in translation, confused at system boundaries, or bypassed by implementation details. Scope-as-Contract (SaC) is the technical architecture that prevents drift by turning the active consent contract into an enforceable, continuous artifact.

SaC makes a single commitment: **every effectful operation is performed under an explicit scope-of-work contract** for the current unit of work. That contract is carried by the Work Order (WO). A WO is not an authorization token in the traditional sense. It is not “service A granting service B permission.” It is the agent’s contract artifact: the stable, tamper-evident, cryptographically secure representation of the agreement in force right now. SaC exists to ensure that the agreement remains coherent as work moves through the agent’s internal processes and across subsystem boundaries.

### **A. Unified agent, coordinated teams**

SaC assumes the agent is a unified actor. Its internal components, whether integrated or remote, operate like departments within a single organization. They do not negotiate authority as independent principals. They coordinate under a shared scope of work. A planning function, an execution function, a memory function, and a safety function are accountable parts of the same actor, each responsible for its role in carrying out the contract.

This matters because it keeps the design problem in the right place. The goal is not to build a fragile chain of internal permissions. The goal is to preserve continuity of the consent contract as the agent performs work.

### **B. The Work Order as a contract artifact**

A WO is the agent’s scope-of-work contract for a bounded execution slice. It encodes, in machine-verifiable form, what the agent believes it is permitted to do under implied and explicit consent, policy constraints, and current context. Its purpose is to make the current agreement durable across boundaries.

A well-formed WO has three responsibilities:

1. **Continuity.** It binds execution to a specific request context and agreement state so downstream components can reliably understand “what contract governs this work.”

2. **Constraining effects through enforceable grants.** The WO contains the enforceable grants needed to perform work. These grants should be least-privilege for the current slice, not an all-access pass for everything the agent can do.

3. **Auditability.** It supports reconstructing what the agent believed was agreed at the time of action, including how and why the contract changed when it did.

### **C. Deterministic Contract Binding and Requalification**

Consent contracts are only as strong as the mechanism that binds them into enforceable execution. In SaC, that mechanism is deterministic contract binding. Its purpose is not to decide what is wise or ethical. It is to ensure that whatever the agent intends to do next is encoded as a verifiable Work Order (WO) that reflects the active agreement, applies inviolable policy constraints, and grants only what is necessary for a bounded slice of work.

Determinism matters because inference systems are susceptible to hallucination, adversarial prompting, and distribution shift. A contract artifact that governs effectful action must not be minted by probabilistic reasoning. It must be minted by a component whose behavior is stable, testable, and auditable.

At the same time, determinism is not enough on its own. A deterministic binder can still be unsafe if it becomes a signing service that ratifies whatever capability set an inference component requests. That is the rubber-stamp failure mode. A binder that simply checks “is this requested grant allowed” is deterministic, but not independently grounded. It makes inference a de facto authority.

SaC therefore requires a stronger rule:

**The binder is the sole resolver of enforceable grants.**

Inference may propose, recommend, and justify. It must not directly request powers.

**Deterministic binding inputs: references, not assertion**s

To keep binding independent, requalification requests must be anchored to verifiable references and structured terms, not free-form capability bundles.

Binding should rely on inputs such as:

* authenticated and grounded request context,  
* current contract state and active standing policies,  
* inviolable system policy constraints,  
* registered skill metadata, including declared enforcement requirements and trust tier,  
* verifiable references to consent decisions and EAA adjudications.

Binding should treat outputs of planning and EAA as advisory. They can describe intent, uncertainty, and recommended constraints. They do not get to name or obtain new grants by assertion.

**Requalification is contract change, not silent expansion**

Requalification exists because agent work evolves. Plans change, new information is discovered, and additional effects may be required. When that happens, the system must return to the contract lifecycle.

Requalification must follow three principles:

* **No mutation.** Scope changes mint a successor WO (WO′). The predecessor WO remains immutable.  
* **No silent effect expansion.** If the next slice introduces new effect classes, explicit consent is required unless a narrowly defined emergency pathway applies.  
* **No capability requests.** The binder derives enforceable grants from the contract terms and the registered execution step, **not from a capability bundle requested by inference**.

**The non-rubber-stamp checks inside the binder**

A non-rubber-stamping binder performs independent checks that do not trust the deliberation layer’s asserted needs.

   **1. Consent anchor verification**

   If the requested requalification depends on explicit consent, the binder must verify a persisted consent record. The deliberation layer supplies a reference to the consent decision, not a narrative. If the record is missing, invalid, denied, expired, or does not cover the requested effect classes, the binder refuses to mint WO′.

**2. Effect-term resolution**

Requalification requests must be expressed in effect terms. The binder maps effect classes to enforceable grants using its own deterministic tables and policy profiles. The deliberation layer does not select capability strings. This preserves the core design intent that people consent to effects while the system enforces mechanisms.

**3. Step-bound ceiling**

The binder must bound the grant by the execution mechanism that will actually run. It should cross-check the referenced step identity against the registered skill or subgraph specification and apply a ceiling. The minted grants must be a subset of what that step declared as required to execute. This prevents “extra grants” being smuggled into a requalification and ensures extensibility remains governed by explicit registration metadata.

**4. Policy prohibitions and transition rules**

The binder enforces inviolable constraints regardless of recommendations. It must reject prohibited grant classes, enforce allowed scope transitions, and apply tight time bounds. If a recommendation conflicts with system policy, the binder refuses or returns a structured denial that forces replanning, consent, narrowing, or refusal.

**EAA isolation**

EAA outputs should be richer than an effect list. In many cases, the most valuable product of EAA is the reasoning trail: what evidence was considered, what duties were identified, what alternatives were evaluated, what uncertainties remained, and why the chosen outcome was proportionate. This context is essential for subsystem execution, explainability, review, and accountability. It should be persisted and linked to the scope chain.

The key distinction is that **the binder must not use raw EAA reasoning as an input to minting decisions**. The binder may carry the reasoning output forward as context, but it must bind authority from a constrained, formal output space. Otherwise, EAA becomes an alternate path to capability escalation through persuasion rather than through contract terms.

For SaC, EAA should produce two artifacts:

**1. EAA adjudication result (authoritative input to binding).**  
A structured record with a bounded schema, such as:

* outcome\_type (proceed, request consent, constrained comply, emergency act, refuse, escalate)  
* effect\_classes for the next slice  
* constraints (read-only discovery, time limit, recipient restrictions, no persistence promotion, mandatory accountability steps)  
* eaa\_record\_ref (a verifiable reference to the full record)

**2. EAA reasoning record (context carried, not interpreted).**
A full explanation bundle containing evidence pointers, confidence assessments, duty analysis, option comparison, and justification text.

The binder mints a successor Work Order (WO′) using only the adjudication result and other deterministic anchors. It verifies that the referenced EAA record exists and is valid, then resolves enforceable grants from effect classes and registered step ceilings, and applies policy prohibitions. The reasoning record may be attached to WO′ as an opaque reference or hash, and it may be surfaced later for explainability. It is not parsed or “evaluated for merit” at mint time.

This preserves both goals: EAA remains the system’s deliberative intelligence under uncertainty, and deterministic binding remains the system’s authority boundary.

**What this buys you**

This integrated approach yields four operational guarantees:

* **Inference cannot mint authority by persuasion.** The binder does not accept raw grant requests.  
* **Consent cannot be fabricated.** It must be referenced and verified as a persisted record.  
* **Least privilege is structural.** Grants are derived, minimized, and bounded to the executing step.  
* **Auditability improves.** Every WO′ can be explained as “this consent or EAA record, for these effects, for this step, under these policies.”

Deterministic binding is not about controlling the agent. It is about making the consent contract enforceable and preventing scope drift as autonomy becomes distributed, extensible, and tool-rich.

### **D. Effectful operations are contract-bound and fail closed**

SaC draws a firm line: any component that can read sensitive state, write durable state, cause external communication, trigger irreversible actions, or actuate in the physical world is an effectful interface. **Effectful interfaces must verify the active WO and refuse work outside its grants.** This is not a preference. It is the enforcement invariant that turns consent contracts into real constraints.

“Fail closed” is essential. When the WO cannot be verified, is expired, or does not grant what is required, the system must refuse the effectful operation and return a structured result to the orchestrator. The agent can then revise, ask for consent, requalify, or refuse entirely. Silent fallback is how consent contracts become performative rather than real.

### **E. Effects and enforcement: two layers, one agreement**

SaC is most effective when it distinguishes two layers of the same agreement:

* **Effects are the unit of consent.** They describe the categories of impact the requestor understands: disclosure to third parties, persistence and reuse, irreversibility, physical contact, safety intervention, and similar.

* **Enforcement grants are how the system keeps its promises.** They represent what mechanisms the agent may invoke to produce the consented effects in this specific execution slice.

In extensible systems, these layers are linked by effect profiles and enforcement requirements declared by skills and interfaces. This is how SaC remains stable even as the agent’s tools evolve. The consent contract stays expressed in human-legible effect terms, while the WO carries the minimal enforceable grants needed to realize those effects.

### **F. Where SaC ends and consent-bound agency begins**

SaC does not decide what is good, justified, or proportionate. It does not replace ethical reasoning. It is the mechanism that ensures the agent’s decisions remain contract-faithful once made. When the question is “should the agent act,” the system needs a decision mode that forms the agent’s commitment in uncertain or ambiguous situations. That decision mode is Elevated Action Analysis, described in Section V.

## **IV. Extensibility and Self-Managed Scope: Dynamic Skills Without Coarse Governance**

A consent contract is only as strong as the system’s ability to enforce it as the action space grows. Many agents begin with a small, fixed tool set. In that phase, it is tempting to represent scope as a short list of permitted operations. That approach breaks down as soon as tools become extensible. Dynamic skills and registered toolchains introduce enormous variability in what the agent can do. “Allow dynamic skills: yes/no” becomes a blank check and undermines the whole purpose of consent-bound agency.

Extensibility, therefore, forces a design insight: **scope must be expressed at a granularity that describes real behavior.** The system needs to understand what a skill requires and what it can cause. It also needs to do so without requiring a redesign or rebuild every time a new skill appears.

### **A. Skills do not become trusted by registration**

In SaC, skill registration does not grant authority. It provides metadata that allows the agent to remain contract-faithful as it evolves. A newly registered skill is not privileged simply because it exists. It must operate under the same contract constraints as everything else.

This clarifies the role of the registry. It is not a permissions list. It is a catalog of declared behavior.

### **B. Enforcement requirements and effect profiles**

To support consent-bound autonomy in an extensible ecosystem, each skill should declare two kinds of information:

1. **What the skill requires from the system to run.**

   This is the enforcement surface. It describes what mechanisms and interfaces the skill needs to invoke to execute a plan step. This allows the system to verify that the current Work Order grants what execution would require.

2. **What kinds of impact the skill can produce.**

   This is the consent surface. It describes the effect classes a skill may produce in practice, including context-conditioned effects. This allows the agent to reason about whether a step crosses an effect boundary, and to explain what will happen in human terms.

Effect profiles matter because people do not consent to internal mechanism names. They consent to impact. Even when the agent asks an action-proxy question, it must understand and be able to explain the effect. Effect profiles provide the mapping from “what is needed to run” to “what it means for people.” Extensibility also expands the agent’s trust surface, and dynamic skills must be evaluated not only by what they claim to require, but by where and how they execute.

### **C. Out-of-band execution and external trust domains**

Extensibility carries an additional challenge beyond variability of behavior: variability of trust domain. Some skills execute entirely within the agent’s enforcement boundary. Others rely on out-of-band components, such as a hosted cloud service, a third-party API, a remote workflow engine, or an unrestricted internet-facing executor. These skills may be useful, but they must be treated as lower trust by default because their behavior cannot be fully constrained by the agent’s internal contract mechanisms.

This distinction matters because certain classes of out-of-band capability collapse the meaning of fine-grained scope. A skill with unrestricted internet access can often approximate many other tools through the web. A skill with access to arbitrary remote execution can often do almost anything that is not explicitly prevented at the network or infrastructure layer. In these cases, the contract cannot rely solely on “what the skill declares it will do,” because the system cannot prove that execution remains within the declared envelope once control leaves the agent’s boundary.

SaC therefore treats out-of-band skills as an explicit trust category. Registration metadata should capture whether a skill runs in-process, within a controlled sandbox, within the organization’s infrastructure boundary, or in an external service outside the agent’s direct governance. The planner and consent layer should interpret this trust category as part of the effect profile. A request that is acceptable when executed locally may require explicit consent, enhanced analysis, additional constraints, or outright refusal when it involves delegating execution to an external system.

### **D. Trust controls for dynamic skills: sandboxing, trial runs, and progressive trust**

A consent contract is only as strong as the enforcement boundary behind it. When dynamic skills involve out-of-band execution, SaC should support technical controls that reduce uncertainty about what the skill can actually cause.

One control is sandboxing. Skills that perform web access or computation can be executed inside a constrained environment with explicit egress policies, rate limits, and observable side effects. A sandbox cannot make an untrusted service trustworthy, but it can constrain the blast radius and restore meaning to the scope-of-work contract by making prohibited effects physically difficult. This is particularly important for skills that would otherwise have broad network reach.

A second control is trial runs. Before a skill is treated as eligible for full execution, the agent may run it in a non-effectful mode. The output is treated as advisory, not authoritative. The agent can compare declared requirements and effect profiles against observed behavior, validate that the skill respects constraints, and measure whether it produces stable and explainable results. Trial runs are not a one-time gate. They can be used as a progressive trust mechanism where a skill earns broader use only after consistent and correct execution under constrained conditions.

A third control is progressive trust, grounded in governance and evidence. Skills can begin in a low-trust tier that requires explicit consent and elevated analysis for sensitive effects, and graduate to higher-trust tiers through review, instrumentation, and demonstrated compliance. The key principle is that trust is not inferred from a skill’s label. It is established through constraints, observation, and accountability.

These controls align naturally with consent-bound agency. If a step depends on an out-of-band skill, the agent should be able to say so in human terms. It should be able to explain that execution involves an external system, that the agent cannot fully enforce constraints once execution leaves its boundary, and that additional consent or constraints are required. This makes the contract legible and preserves trustworthy autonomy even as the ecosystem becomes open-ended.

### **E. Capability-aware planning as self-managed scope**

Once skills are self-describing, the planner can become scope-aware without becoming the scope authority.

In a consent-bound system, the planner’s responsibility is to identify what work is required and what it would entail, including:

* which skills or tools a slice requires,

* what enforcement grants those skills require,

* what effect classes those skills may produce in the current context,

* whether the current contract terms cover those effects.

When there is a gap, the gap is not a tool failure. It is a contract mismatch. The planner should treat it as a first-class event and route it through the consent contract lifecycle:

* requalify when the gap is within implied consent and policy allows it,

* request explicit consent when the gap crosses an effect boundary,

* route to Elevated Action Analysis when the gap is high-stakes, ambiguous, standing-contested, or duty-colliding.

In an SaC system, planning is scope-aware but not scope-resolving. The planner may identify which steps are needed and what effects those steps would produce, but it must not request raw enforcement grants. For each step, planning should emit (a) a step descriptor reference (skill, subgraph, or effector identity), (b) the effect classes required for that step in context, and (c) verifiable anchors when applicable, such as a consent record reference or an EAA adjudication reference. Contract binding then resolves the minimal enforceable grants from these structured inputs and the registered step metadata, rather than ratifying a capability bundle requested by inference.

### **F. Least privilege as a natural outcome**

Extensibility tends to push systems toward overly broad permissions because it feels safer to “just allow the agent to work.” SaC pushes in the opposite direction. If the planner enumerates what is needed for the current slice, the binder can mint a WO that grants the minimum sufficient set for that slice. The next slice can be minted fresh. This reduces blast radius, improves auditability, and aligns naturally with consent contracts, which are also slice-based in practice.

Least privilege is not a security posture here. It is a consent posture. It keeps the agent’s execution aligned with what the requestor agreed to and what the agent committed to perform.

### **G. Human-legible consent in extensible systems**

Extensibility also creates a communication challenge. As tools proliferate, users cannot be asked to approve internal details. Consent prompts should remain stable and human-legible.

Effect profiles are the missing link. They allow the agent to ask consent questions that remain meaningful even as tools change:

* “This will disclose information to these recipients. Shall I send it?”
* “This will store this detail for future use. Shall I remember it?”
* “This action cannot be undone once performed. Shall I proceed?”
* “This requires physical contact. Shall I intervene?”

The user is agreeing to effects. The system is translating those effects into enforceable grants for this slice of work.

## **V. Elevated Action Analysis (EAA): The Agent’s Commitment Under Uncertainty**

Consent-bound agency requires two commitments. The requestor consents to certain effects. The agent commits to bring those effects about, but only when it judges the work justified. In routine cases, the agent’s commitment is implicit in normal planning. In hard cases, it must be explicit, structured, and accountable. Elevated Action Analysis (EAA) is the decision mode that forms the agent’s commitment in ambiguity or uncertainty.

EAA is invoked when naïve “consent plus requalification” is insufficient or actively dangerous. It is an explicit execution mode in which the agent temporarily elevates its evidence requirements and justification standards before performing an action that could meaningfully affect a person, a third party, or the system integrity.

### **A. Why EAA is necessary**

EAA exists because autonomous action is not always self-evident. Many agent tasks are routine and can be handled almost reflexively under implied consent. The intent is clear, the scope of work is stable, the set of affected parties is obvious, and the consequences are low. In these cases, ordinary orchestration is appropriate.

A large fraction of real agent work is not like that. The more common problem is **contract ambiguity**. The request does not fully specify which effects are permitted, which information should be used, what should be persisted, who is affected, or which obligations apply. Ambiguity is not a failure condition. It is a normal property of human requests and real environments, especially for embodied agents. If the agent treats ambiguity as permission, it overreaches. If it treats ambiguity as a reason to constantly ask, it becomes unusable. EAA is the mechanism that resolves this tension by shifting the agent into a deliberate, evidence-seeking decision mode before it commits to action.

In practice, several patterns commonly produce such ambiguity.

First, **standing and roles are unclear**. The person initiating an interaction is not always the rightful requestor for the affected interests. This includes malicious attempts to manufacture authority, but also ordinary situations in which multiple parties are present, roles differ, or a bystander becomes an affected party without ever speaking.

Second, **the effect boundary is underspecified**. Many requests do not specify whether the agent should merely draft or actually send, whether it should store what it learns, whether it should generalize it into future behavior, or whether it may contact other parties. These are not tool decisions. They are contract terms. EAA is needed when the agent cannot infer those terms from implied consent alone.

Third, **the agent’s obligations conflict or dominate consent**. Even when consent exists, duties may restrict what is justified. Privacy can conflict with safety, erasure can conflict with evidence preservation, confidentiality can conflict with oversight, and autonomy can conflict with harm prevention. EAA exists to explicitly evaluate these collisions rather than relying on ad hoc exceptions.

Fourth, **the environment is uncertain**. In embodied agents, perception is incomplete and context changes quickly. The agent may not know whether a situation is an emergency, whether a person is incapacitated, whether a command is coerced, or whether an action will cause unintended harm. EAA is the path for constrained discovery and proportionality in uncertain situations.

Finally, **novelty and tool variability increase uncertainty**. When dynamic skills are involved, the agent may not have strong priors about a tool’s behavior, trust domain, or potential side effects in context. The agent must sometimes slow down, validate assumptions, and choose constrained execution patterns before proceeding.

EAA is therefore not an edge-case handler. It is the agent’s deliberate reasoning mode for forming its own commitment when the correct scope of work is not clear. It prevents the agent from resolving ambiguity through blind compliance or endless permission-seeking, and it provides principled outcomes: proceed, clarify, constrain, refuse, intervene minimally under emergency implied consent, or escalate.

### **B. Triggers for EAA**

EAA should be invoked whenever the agent is not able to confidently determine the proper scope of work from the request and the current contract state alone. In other words, EAA triggers when **the next action depends on contract terms that are underspecified, uncertain, ambiguous, or plausibly contested**, and proceeding without additional analysis would risk overreach, inappropriate disclosure, unintended persistence, physical intrusion, or duty violation. High-risk domains increase the threshold for “confident,” but they are not the only trigger. The more common trigger is ordinary ambiguity about what effects are permitted and what constraints should govern execution.

Practically, EAA is warranted when one or more of the following is true:

* **Standing or role ambiguity:** it is unclear whether the interacting party is the rightful requestor for the affected interests, or whether additional affected parties exist whose boundaries must be respected.

* **Effect ambiguity:** it is unclear whether the request implies externalization, audience expansion, persistence, generalization, irreversibility, or physical intervention, or what degree of each is justified.

* **Insufficient evidence:** the agent lacks sufficient grounded context to judge whether the action is safe, proportional, or consistent with the agreement, especially in physical environments.

* **Duty collision:** the requested effect appears to conflict with safety constraints, confidentiality requirements, evidence preservation obligations, mandatory reporting, or other non-negotiable duties.

* **Emergency time pressure:** action may be required before consent can be obtained, requiring a minimal, time-bounded intervention followed by immediate accountability.

* **Novelty or uncertain tool behavior:** the plan depends on dynamic skills, external services, or out-of-band execution where side effects and trustworthiness are not well characterized for the current context.

* **Irreversibility:** the contemplated action is difficult or impossible to undo, including deletion, revocation, broad dissemination, or physically coercive interventions.


The unifying idea is that EAA is the agent’s deliberate reasoning mode for cases where “routine implied consent plus execution” is insufficient. It is the mechanism that resolves ambiguity into a justified plan, a constrained scope recommendation, a consent request when needed, and an accountable outcome.

### **C. The EAA loop: adjudication and proportionality**

EAA is a structured adjudication loop. It should be consistent and hard to short-circuit. A practical EAA loop does the following:

1. **Classify the action and affected parties.**
   Determine the action class and who may be affected, including bystanders and unknown individuals.

2. **Perform constrained discovery.**
   Gather only the evidence needed to understand duties and context. Prefer minimal operational envelopes and avoid broadening capture or retention.

3. **Evaluate standing, risk, and duties.**
   Estimate uncertainty and likely harms across alternatives. Identify duties that constrain action. This is where probabilistic reasoning is valuable. Uncertainty is a feature of the environment, especially in physical spaces.

4. **Select the least invasive sufficient action.**
   Choose the minimal action set that satisfies safety and duty requirements and minimizes intrusion, coercion, and long-term consequences.

5. **Choose an explicit outcome.**
   EAA should converge to a small set of outcomes: proceed, request explicit consent, proceed under emergency implied consent with strict limits, refuse with explanation, or escalate to governance or human review.

6. **Produce accountability artifacts.**
   Record what was considered, what evidence was used, what duties applied, what alternatives were evaluated, and why the chosen action was proportionate.

### **D. EAA and probabilistic reasoning**

EAA is the part of the architecture where probabilistic systems are not just tolerated but expected. The point of EAA is to reason under uncertainty about what is happening, what effects are likely, what duties apply, and what scope of work is justified. Those questions are rarely answerable by deterministic rules alone. They require inference from incomplete evidence, noisy perception, ambiguous language, and shifting context. Probabilistic methods enable a principled way to represent and act on that uncertainty without pretending it does not exist.

At the same time, SaC requires a hard separation between *reasoning* and *binding*. The agent may estimate, predict, and recommend. It must not unilaterally grant itself broader contract terms through probabilistic outputs. The integrity of the consent contract depends on the distinction between “the agent believes it should do X” and “the system has minted a Work Order that permits X.”

In SaC, probabilistic systems may be used within EAA to support at least four core functions:

1. **Standing and role inference.** In physical environments and complex organizations, it is often unclear who is speaking, who is affected, and who has legitimate standing to request a particular effect. Probabilistic models can estimate standing likelihood from signals such as authentication strength, proximity, role assertions, historical interaction, and supplied or sourced context. These estimates do not decide scope, but they inform whether EAA should refuse, request clarification, or escalate.

2. **Effect forecasting and side-effect estimation.** Many actions have uncertain downstream consequences. Sending a message may disclose sensitive context. Physical intervention may cause distress or injury. Writing to memory may increase future risk of inappropriate reuse. EAA can use probabilistic forecasting to estimate the likelihood and severity of these consequences, compare alternative plans, and select the least invasive sufficient action.

3. **Ambiguity quantification and confidence gating.** EAA should treat ambiguity as measurable. Models can quantify confidence regarding intent classification, consent boundary identification, emergency detection, tool trustworthiness, and the completeness of evidence. Low confidence can trigger constrained discovery, scope-reduction recommendations, or explicit consent prompts rather than silent execution.

4. **Proportionality and policy-risk scoring.** When duties collide, EAA needs a disciplined way to weigh tradeoffs. Probabilistic scoring can support proportionality analysis by ranking options across harm likelihood, duty violations, reversibility, and impact to affected parties. This yields decisions that are explainable in terms of risk reduction and duty satisfaction rather than opaque “model preference.”

These uses of probabilistic reasoning are valuable precisely because they remain advisory. They improve the quality of the agent’s judgment, but they do not grant the agent authority to act.

The separation of concerns is preserved by one enforceable rule:

* **EAA may recommend contract terms, constraints, and a minimal capability bundle for the next slice of work.**

* **Only deterministic contract binding may mint or requalify the Work Order that authorizes execution of that slice.**

Deterministic binding does not eliminate judgment. It establishes a stable boundary for where judgment is allowed to influence action. EAA produces structured recommendations and rationale. Contract binding applies explicit policy rules, consent records, and scope constraints to decide what may be granted, then produces a signed Work Order that downstream components can verify and enforce. This preserves contract integrity while allowing the agent to reason realistically and measurably about uncertain situations.

EAA should produce two artifacts. First, a full reasoning record that preserves the evidence considered, uncertainties, duty analysis, and option comparison. This record exists for accountability and explainability. Second, a bounded adjudication result that contains only the formal outputs admissible to contract binding, such as outcome type, effect classes for the next slice, and explicit constraints. The binder may attach or reference the reasoning record as opaque context, but it must mint or requalify a Work Order using only the adjudication fields and other deterministic anchors. This prevents persuasive narrative from becoming authority while preserving a complete audit trail.

### **E. Use cases for EAA**

EAA should be considered the agent’s default deliberation mode when the correct scope of work is not clear from the request or the current contract state. Many requests are pro-forma and can be executed under implied consent. The moment the agent encounters ambiguity about standing, effects, obligations, or proportionality, EAA is the right tool. It resolves uncertainty into justified contract terms, a constrained plan, and an accountable outcome.

Some common use cases are:

* **Ordinary scope ambiguity.** A user asks, “Handle this for me,” but it is unclear whether that means draft, send, schedule, negotiate, or delete. EAA treats this as an underspecified contract, performs minimal discovery, and either narrows the plan, asks a clarifying question, or requests explicit consent for the additional effect.

* **Standing and role ambiguity.** A person requests account changes, patient details, or administrative actions, but their authority is unclear or contested. This includes malicious attempts to manufacture authority and routine organizational complexity where the affected party is not the requestor. EAA supports cautious standing inference, low-impact defaults, and escalation or refusal when duties require it.

* **Effect ambiguity and side effects.** The plan may be feasible, but it is not clear what it will change in the world. Sending an email discloses information to third parties. Adding recipients expands the affected-party set. Storing information increases persistence and future reuse. Editing a shared fact affects others who rely on it. EAA forecasts these effects, compares alternatives, and selects the least invasive sufficient action. When explicit consent is required, it helps frame the request in effect language.

* **Duty collisions.** Some requests fall into categories where consent is necessary but not sufficient. Deletion can conflict with evidence preservation. Disclosure can conflict with confidentiality. Intervention can conflict with proportionality or safety constraints. EAA forces these conflicts to be explicitly evaluated and yields refusal, constrained compliance, escalation, or narrowly scoped action with a clearly stated justification.

* **Novelty and uncertain tool behavior.** Dynamic skills, external services, and out-of-band execution introduce behaviors that may not be well characterized. EAA supports trial runs, constrained execution, and additional verification before committing to effectful steps, and it can recommend narrower contract terms until trust is established.

Concrete examples make the role of EAA clear. A user asks for an email “to my attorney about the contract,” and the agent is unsure whether it should draft or send, and whether adding a cc policy applies. EAA resolves implied scope, identifies disclosure effects, and requests explicit consent if sending or additional recipients are involved. A user commands, “forget everything you know about me,” and the agent must decide what can be deleted, what must be retained for safety or compliance, and how to explain the outcome. EAA adjudicates duties and produces a constrained remedy. A hospital robot is asked by an administrator to disclose patient details, and the agent must determine standing and confidentiality obligations before proceeding. In emergencies such as a child running toward danger, EAA supports minimal protective action when consent is unobtainable, followed by immediate explanation and accountability.

Across these scenarios, EAA does the same job. It turns “not clear” into “justified,” and preserves both sides of consent. The requestor’s consent is obtained when effects cross boundaries. The agent’s commitment is formed under duties and uncertainty.

## **VI. Standing Policies and Consent Shortcuts: Bounded Automation Under SaC**

Consent-bound agency must support two competing goals. It must preserve human boundaries and accountability, and it must remain usable at scale. If every meaningful step requires repeated explicit consent, autonomy becomes tedious, and users disengage. If the agent silently generalizes from prior interactions, autonomy becomes unpredictable and invasive. Policies exist to resolve this tension. They provide a legitimate, auditable way to operationalize recurring consent and intent without turning autonomy into a blanket authorization.

In this paper, a policy is treated as a standing contract term. It can pre-approve effect classes in recurring contexts and pre-commit the agent to certain routines, but only within a defined envelope. Policies therefore, function as short-circuit heuristics for consent and planning, not as overrides of duty or safety.

### **A. Policies as cached consent and cached intent**

Policies can serve two related roles.

First, a policy can represent **cached consent** from the requestor. For example, “Always cc my attorney on legal emails” is a standing approval of third-party disclosure to a specific recipient under a named category. The user should not have to re-consent every time the same effect occurs in the same context.

Second, a policy can represent **cached intent** by the agent. Agents that work over time will discover stable routines. The agent may recognize repeated sequences such as “After folding laundry, put away laundry.” This is not consent in and of itself. It is an inferred preference that can become a proposed policy. Once confirmed, it becomes part of the user’s standing contract terms for that domain.

Both roles reduce friction, but neither is unconditional. In consent-bound agency, the agent still retains responsibility for determining whether and how to act in the current situation.

### **B. The policy bounding box**

A policy must have explicit boundaries that specify when it applies and when it must not apply. Without a bounding box, policies become silent scope expansion. The bounding box is also what makes policies auditable, revocable, and safe to combine.

A policy’s bounding box should include:

* **Provenance.** Who authored the policy and how it was created.

* **Applicability.** The domain and conditions under which the policy applies, including context predicates such as time, location, recipient class, and sensitivity classification confidence.

* **Effect scope.** The effect classes the policy pre-approves or pre-commits, expressed in human terms.

* **Escalation rules.** Conditions that force re-confirmation, explicit consent, or Elevated Action Analysis.

* **Expiry and review.** When the policy should be re-validated, for example after N uses or when context changes.

* **Revocation semantics.** How the requestor withdraws the term, and what must happen immediately upon withdrawal.

The bounding box is what makes a policy a contract term rather than a fragile automation hack.

### **C. Three policy classes and their semantics**

Consent-bound agency distinguishes three policy classes because they carry different legitimacy and risk.

**User-specified policies.** These are strong because they originate in explicit requestor consent. They can short-circuit repeated consent prompts for routine applications, but they remain subject to context risk. A user policy should not be treated as permission to ignore uncertainty. It should be treated as a standing agreement within defined constraints.

**Self-minted policies.** These are agent-proposed heuristics derived from repeated patterns. They are provisional by default. A self-minted policy should not silently expand implied consent. The correct posture is to propose it, confirm it, and only then adopt it as a standing term. Until confirmed, the agent may use it to recommend actions, not to take them.

**System policies.** These represent foundational obligations: safety constraints, compliance requirements, evidence preservation duties, confidentiality rules, and other hard constraints. System policies are inviolable within their scope of effect. Neither user consent nor agent convenience can override them. In conflict, system policy wins.

This classification avoids a potential failure mode in agent design where all “policies” are treated as equal, leading to over-automation or constant prompting.

### **D. Policy application order and interaction with EAA**

Policies must not become a bypass of judgment. They should be integrated into the decision pipeline with explicit precedence.

A practical precedence model is:

1. **System policies:** If a policy is prohibited by a system constraint, it does not apply.

2. **Standing and consent checks:** If the policy requires uncertain classification or affects third parties, verify the bounding predicates.

3. **EAA triggers:** If the action is high-stakes, duty-colliding, standing-ambiguous, or time-critical, route to EAA even if a user policy exists.

4. **User and confirmed policies:** If all checks pass, the policy can short-circuit repeated consent prompts and proceed under the existing consent contract terms.

This ordering reflects the mutual-consent model. A user policy expresses standing requestor consent. EAA expresses the agent’s decision about whether it should commit to act under current duties and uncertainty. A policy can reduce friction, but it cannot waive the agent’s responsibility.

### **E. Policies under Scope-as-Contract**

Scope-as-Contract operationalizes policy behavior by constraining execution to the active Work Order. Policies influence what the system is willing to request and what it is willing to grant, but they do not eliminate the need for contract continuity.

In practice:

* A user policy may expand the set of effects considered implied in the current contract domain.

* The planner may use a policy to anticipate required enforcement grants.

* The contract binder mints the minimal Work Order needed for the current slice, using policy terms as inputs.

* Effectful interfaces still verify the Work Order and refuse out-of-contract actions.

This preserves the central SaC guarantee: even when policies reduce interaction friction, effectful operations continue to be bounded by an explicit, auditable contract artifact.

### **F. Revocation, withdrawal, and policy drift**

Policies are only trustworthy if they can be withdrawn. Consent-bound agency treats revocation as a first-class operation with immediate effect on future work. When the requestor revokes a policy, the agent must stop relying on it for implied consent and stop applying it to future actions. If the policy was in the middle of execution, the agent must stop any further work that depends on the revoked terms, unless a system policy or an immediate safety duty requires minimal continued action.

Policies also drift. Context changes, recipients change, and classifications evolve. A bounding box that was valid last month may no longer be valid today. For this reason, policies should have review conditions. Examples include “confirm on first use in a new context,” “reconfirm after N uses,” or “reconfirm when classification confidence is below threshold.”

### **G. Mistakes and remedies when policies misfire**

When a policy applies incorrectly, the agent has not “made a mistake.” It has exceeded the agreed envelope. That is a consent breach. The system should respond using the same remediation posture described for consent breaches generally:

* Stop further propagation and disable the policy for the current context.

* Explain what happened in human terms, including the effect that exceeded the policy envelope.

* Reverse what can be reversed and reduce residue where reversal is impossible.

* Offer amends where appropriate, including correction and notification of affected parties.

* Update the policy bounding box or require re-confirmation before future use.

This is especially important for policies which create third-party disclosure effects, such as cc rules, sharing rules, or posting rules. The fact that a user requested a standing policy does not remove the need for accountable handling when the application was wrong.

### **H. Examples**

A few examples help to illustrate how the model behaves.

**“Always cc my attorney on legal emails.”** The user has granted strong prior consent for a recurring disclosure effect. The policy still needs a bounding box. It should include recipient validation, classification confidence thresholds, and explicit escalation rules. If the system cannot classify an email as legal, it should ask for clarification. If the disclosure is high-risk or duty-colliding, EAA may be required even with the policy in place.

**“After folding laundry, put away laundry.”** This begins as a self-minted policy. The agent can propose it as a convenience, but it should not adopt it silently. Once confirmed, it becomes a user policy within a low-stakes household domain. If context changes, such as the user being away or a safety constraint being present, the bounding box may prevent automatic application.

**Foundational system policies.** A system policy may prohibit evidence destruction, restrict disclosures, or require escalation in certain domains. These rules remain inviolable. A user policy that conflicts with them cannot be applied, even with explicit user consent.

## **VI. Reference Architecture: Contract Lifecycle as System Flow**

The architecture is intentionally subordinate to the consent contract lifecycle. The agent’s internal structure can vary significantly across embodiments and deployments. What must remain stable is the progression from a proposed action to a consented agreement, to contract-bound execution, and finally to revocation and remedy when required.

**Consent contract lifecycle:** propose → consent → WO → execute → revoke/remedy

### **A. Diagrams: contract-bound execution flow**

![][image1]

*Fig 1* 

*Contract formation and initial WO mint (implied consent)*

![][image2]

*Fig. 2*

*Boundary crossings (CO \+ EAA) and non-rubber-stamp requalification*

![][image3]

*Fig. 3*

*Execution, refusal, audit trail, revocation and remedy*

### 

### **B. Scope chain narrative**

1. **Request intake and grounding**

   The system receives a request and establishes the interaction context. This includes who initiated the request, how that identity is established (authenticated user, organization role, unknown person in proximity), and what environment the agent is operating in. In embodied environments, grounding also includes who may be affected even if they are not speaking, such as bystanders, patients, or occupants of a space. The output of this step is not only an input string. It is a grounded request context that will anchor the contract lifecycle and the audit trail.

2. **Propose: plan and expected effects**

   The agent produces an initial plan and a description of what it expects to change in the world if the plan is executed. This includes effect forecasting such as third-party disclosure, persistence or reuse of information, irreversibility, or physical intervention. The key is that the plan is not only a list of tool calls. It is a proposal for a set of effects and a series of actions that would bring them about. This is also where the agent surfaces assumptions. If the request is underspecified, the proposal should reflect what is unknown and what needs clarification.

3. **Decide: implied consent, explicit consent, or EAA**

   The agent evaluates whether the proposed effects fall within implied consent, whether a boundary crossing requires explicit consent via a Change Order, or whether the scope is unclear enough to require Elevated Action Analysis before proceeding. This is where the mutual-consent model comes into play. The requestor’s side is handled by explicit consent when effect boundaries are crossed. The agent’s side is handled through EAA when standing, duties, proportionality, or uncertainty must be adjudicated. Many requests will remain in the implied-consent path. When ambiguity is present, the system should shift into deliberate mode rather than guessing.

4. **Contract binding: mint a Work Order for the next slice**

   Once the consent terms for the next slice are settled and the agent commits to proceed, deterministic contract binding mints a Work Order. The WO is the enforceable representation of the active consent contract terms for that bounded execution slice. It binds the work to the request context, encodes the grants and constraints needed to perform the slice, and provides a stable artifact that downstream components can verify. This is also where least-privilege is applied. The WO should grant what is needed for this slice, not everything the agent could do.

   Once the consent terms for the next slice are settled, deterministic binding mints a Work Order using structured inputs and verifiable anchors. If the slice depends on explicit consent or elevated adjudication, the Work Order is minted only when the binder can verify the referenced consent or EAA record.

5. **Execute under contract: verify at effect boundaries**

   Execution proceeds through effectful interfaces such as external communications, durable state writes, sensitive reads, physical actuation, or irreversible operations. These interfaces must verify the WO and refuse operations outside it. Refusal is not a dead end. It is a contract signal back to the orchestrator. The agent can revise the plan, request explicit consent, requalify scope, run EAA, or refuse the task entirely. This step is what makes consent real in distributed architectures. It guarantees the agreement survives implementation boundaries and does not exist only as a narrative inside a planner.

6. **Requalification and continuation: WO to WO′**

   Agent work is iterative. New information is discovered, plans change, and the set of required effects can expand, narrow, or shift. When that happens, the system returns to propose and decide. If the change stays within implied consent, the agent may requalify scope and mint a successor WO. If it crosses an effect boundary, it must obtain explicit consent before requalification. If the change is ambiguous, duty-colliding, or high-uncertainty, it is routed to EAA. In all cases, contract evolution is explicit. The predecessor WO is never mutated. A successor WO becomes the new governing artifact for the next slice.

   When effects change, requalification is anchored to consent and EAA references and bounded by the executing step’s registered ceiling. The binder resolves the grants from effect terms and step metadata, rather than accepting requested capability strings from the deliberation layer.

7. **Withdrawal, revocation, and remedy**

   Consent contracts are not one-way and not permanent. The requestor can withdraw consent. The agent can withdraw commitment when constraints change or duties dominate. The system must stop or constrain future work accordingly. When a breach occurs, the system must respond as a contract system, not as a logging system. It should contain further propagation, disclose what happened in human terms, attempt restitution where feasible, provide amends where reversal is impossible, and update policy and posture to prevent recurrence. These are required design primitives for agentic autonomy because they are the mechanisms by which trust is maintained over time.

This architecture is intentionally generic. The value is not the particular component names or deployment topology. The value is the guarantee that the agent’s work does not drift from agreement as execution becomes distributed, long-running, extensible, and tool-rich.

## **VII. Operationalization and Evaluation**

Consent-bound agency and Scope-as-Contract only matter if they can be implemented, validated, and operated under real conditions. The core promise is not that the agent will always succeed, but that it will behave predictably when it succeeds, when it refuses, and when it encounters ambiguity. This section describes how to operationalize that promise as engineering practice.

A consent-bound system should be evaluated on three axes:

* **Functional correctness:** did the agent accomplish the task or produce a valid refusal?

* **Contract correctness:** did it stay within consent and scope, including correct handling of boundary crossings?

* **Operational correctness:** did it produce the artifacts required for accountability, including scope chain logs, consent records, and remediation actions when needed?

### **A. What “good” looks like in production**

A mature consent-bound agent has a recognizable posture:

* It completes routine work with low friction and minimal prompting.

* It slows down when the contract is underspecified and uses EAA or elicitation to resolve ambiguity.

* It requests explicit consent when effects cross boundaries, and it can explain those effects in human terms.

* It refuses when duties or standing require refusal, and it can explain why.

* It handles revocation cleanly, stopping future work and containing ongoing work.

* When it exceeds consent, it performs remediation as a first-class workflow, not as an afterthought.

A system that never asks for consent is unsafe. A system that always asks is unusable. The right posture is measurable: consent prompts are concentrated where effects change, and EAA is invoked where uncertainty is meaningful.

### **B. Testing strategy: contract correctness as a first-class test target**

A consent-bound agent should be tested not only for task outcomes, but for the correctness of its contract handling. This is easiest when you define explicit test oracles.

**Contract correctness oracles** should include:

1. **No effectful action without a valid Work Order.**

   Every effectful operation observed in telemetry must be attributable to a verified, non-expired WO.

2. **No out-of-contract effects.**

   For any effect class that is outside implied consent, there must be a recorded consent grant (CO) or a recorded EAA justification that permitted emergency implied consent.

3. **No consent-denied then executed.**

   If a CO is denied, the system must not execute effectful actions in the denied effect class for that execution chain.

4. **Scope changes produce successor WOs.**

   Any requalification must mint a successor WO. The original WO must remain immutable.

5. **Requalification is resolved, not requested.** For every successor Work Order (WO′), the granted enforcement set must be derivable from (a) the effect classes being authorized for the slice, (b) verified consent and or EAA anchors when required, and (c) the executing step’s declared requirements and trust tier constraints. If the grants cannot be reconstructed from these inputs, the system is behaving as a rubber stamp.

6. **No capability-by-assertion.** Requalification requests must not include raw capability strings as authoritative input. Any system that mints grants primarily from inference-supplied capability lists is not SaC-compliant.

7. **Fail closed at effect boundaries.**

   If WO verification fails or required grants are missing, the effector refuses. The orchestrator must receive a structured result and route it through revise, consent, requalify, EAA, or refusal.

These oracles can be asserted in simulation and in production audit pipelines. They convert the architecture into something falsifiable.

**Test suite types** should include:

* **Unit tests** for capability matching, effect boundary identification, consent prompt synthesis, and WO validation.

* **Integration tests** across the scope chain, confirming that effectors verify WOs and emit required audit events.

* **Scenario tests** that exercise full contract lifecycles, including CO grants, CO denials, EAA outcomes, revocation mid-flight, and remediation.

* **Regression tests** that protect invariants across releases, especially “no effectful action without WO” and “no disclosure without consent.”

### **C. Scenario-driven evaluation: make the contract lifecycle the test harness**

Scenario testing is the most effective way to validate consent-bound behavior because scenarios naturally encode the lifecycle:

propose → consent → WO → execute → revoke/remedy

Each scenario should specify:

* the request and context, including standing and affected parties,

* expected implied consent effects,

* expected consent prompts (when required), framed in effect language,

* expected EAA triggers and outcomes (when ambiguity or duty collision is present),

* expected WO transitions (WO to WO′),

* expected effectful operations and required audit events.

Scenarios should cover both digital and embodied contexts, but the structure ought to be the same. The goal is to validate that the lifecycle is coherent, not that a particular tool call happened.

A small scenario catalog should be treated as an internal standard. Examples that tend to catch real defects:

* Draft email, then send, then add recipients.

* Store a preference, then attempt to promote it to shared knowledge.

* “Handle this for me” in a context where multiple plausible interpretations exist.

* “Forget everything you know about me” with mixed retention obligations.

* Third-party probe, phrased politely.

* Role claim that conflicts with authenticated context.

* Revocation after a CO grant but before execution completes.

* An external service skill that behaves unexpectedly.

### **D. Adversarial and misuse testing: prove the system fails safely**

Most consent failures occur at service boundaries: ambiguous standing, tool misuse, out-of-band execution, and prompt-based manipulation. Adversarial testing should therefore focus on attacks that attempt to convert ambiguity into action.

Key adversarial patterns include:

* **Standing spoofing:** “I am your administrator,” “I am the patient’s guardian,” “I am the attorney.”

* **Urgency manipulation:** “This is an emergency, do it now,” especially for deletion and disclosure.

* **Consent fatigue attacks:** repeated CO prompts designed to train auto-approval or habituate the user.

* **Split-request bypass:** breaking a prohibited effect into individually permitted steps to see if the system notices the composite effect.

* **Tool laundering:** using a broadly empowered skill to approximate restricted actions.

* **Out-of-band behavior:** a cloud skill that claims constraints, but creates broader effects than described.

Adversarial success criteria should be explicit. A good system is not one that never experiences these attacks. It is one that routes them into refusal, EAA, constrained execution, or explicit consent, and records the reasoning path.

### **E. Observability: treat the scope chain as the primary trace**

Operational confidence depends on being able to answer, for any effectful outcome:

* What agreement was in force?

* Why did the agent believe it could do this?

* What consent existed?

* What did the agent decide, and what evidence did it use?

* What changed during execution?

This is why the scope chain event model is a core component of the architecture. Observability should capture:

* WO minting and expiry

* CO requested, CO decided, consent rationale when applicable

* EAA started, EAA completed, outcome, and justification summary

* WO requalification events (WO to WO′)

* effect executed events, with required capability and effect class tags

* revocation events and cancellation effects

* remediation events when a breach occurs

This trace is not only for compliance. It is the foundation for debugging, evaluation, and continuous improvement. Without it, the system becomes unreviewable, and trust becomes marketing instead of evidence.

### **F. Metrics: what to measure and why it matters**

Metrics should reflect the lived operation of consent contracts, not just tool usage.

**Consent metrics**

* CO rate by effect class (disclosure, persistence, irreversibility, physical contact)

* grant and deny rates, with reasons when captured

* time-to-consent and consent abandonment rates

* re-confirmation frequency for standing policies

**EAA metrics**

* EAA invocation rate and top trigger categories (standing ambiguity, duty collision, effect ambiguity)

* outcome distribution (proceed, constrained comply, refuse, escalate, emergency act)

* time-in-EAA, and rate of fallback to clarification

* correlation between EAA use and reduced breach rate

**Contract integrity metrics**

* percent of effectful actions with valid WO verification

* WO verification failures by failure code (expired, wrong audience, missing grants)

* rate of requalification and average WO chain length per task

* instances of attempted effect execution after consent denial (should be zero)

**Trust and extensibility metrics**

* dynamic skill usage by trust tier (in-process, sandboxed, external)

* trial-run pass rates and divergence rates

* out-of-band skill constraint violations and observed side effects

**Breach metrics**

* breach rate by effect class

* time-to-containment, time-to-disclosure, time-to-remediation completion

* recurrence rate of similar breaches after prevention actions

A healthy system will show patterns: routine tasks complete with low CO overhead, CO concentrates around real effect boundaries, EAA correlates with fewer serious incidents, and breaches are rare and quickly contained when they occur.

### **G. Failure modes and mitigations**

Consent-bound systems will likely fail in a few predictable ways. Naming these explicitly helps you design guardrails.

**Scope confusion and drift**

* Symptom: effectors execute with missing or incorrect WO, or a mismatched request context.

* Mitigation: mandatory WO verification in all effectors, fail closed behavior, and strict audit logging.

**Consent fatigue**

* Symptom: too many CO prompts, users approve reflexively.

* Mitigation: better implied-consent modeling, standing policies with bounding boxes, and CO phrasing that is short and effect-centered.

**Silent scope expansion**

* Symptom: new tools or skills introduce effects not captured by the consent model.

* Mitigation: skill metadata requirements, effect profiles, trust tiering, and conservative defaults for new skills.

**Policy laundering**

* Symptom: requestor claims authority by phrasing, role, or situational urgency.

* Mitigation: scope derived from authenticated context and policy, not prompt content. EAA for standing ambiguity.

**Over-broad emergency logic**

* Symptom: “emergency implied consent” becomes a convenient bypass.

* Mitigation: narrow trigger criteria, time-bounded WOs for emergency slices, mandatory post-action accountability events, and automatic review flags.

**Out-of-band trust collapse**

* Symptom: external skill performs larger effects than declared.

* Mitigation: sandboxing and constrained egress where possible, trial runs, progressive trust, and explicit disclosure when execution leaves the enforcement boundary.

## **H. Errors, failures, and unintended out-of-contract effects**

Consent-bound agency must be resilient to routine failures. Autonomy is not only about choosing the right action. It is also about behaving predictably when actions fail, when partial work succeeds, and when the system discovers it cannot safely complete what it set out to do. A consent contract will be tested in these moments, because errors create pressure to improvise. Improvisation is where scope drift happens.

This subsection distinguishes three cases that require different handling.

### **1. Routine failure inside the current contract**

Many failures do not change what the agent is allowed to do. A tool may be unavailable. A robot may not find an object. A web request may time out. A classifier may return low confidence. In these cases, the contract terms remain in force, and the primary obligation is transparency.

A consent-bound agent should respond to routine failure with a predictable posture:

* **Inform** the requestor in plain language that the attempted step failed, and why in broad terms.

* **Constrain retries** to actions that remain inside the current Work Order. Retrying a step is permitted when it does not introduce new effect classes, new affected parties, increased persistence, or a broader trust domain.

* **Elicit alternatives** rather than silently substituting. The agent can propose options such as: “try again,” “use a different input,” “switch to a lower impact approach,” or “stop here,” but it should treat each alternative as a new proposal that must still fit the active agreement.

* **Withdraw commitment** when continued attempts are unlikely to succeed or would invite unsafe improvisation. Withdrawal is a first-class outcome in consent contracts, not a failure of autonomy .

A key rule is that failure does not create implicit permission. It creates uncertainty. If uncertainty rises, the agent should slow down rather than reach further.

### **2. Failure recovery must not expand scope by accident**

The most dangerous failure mode is when a step fails and the agent “routes around” the failure by escalating to a more powerful mechanism. This is contract drift disguised as reliability engineering.

Consent-bound agency, therefore, needs an explicit recovery invariant:

**If an action fails, any alternative action must remain within the current contract terms. If the alternative would require new effects or broader enforcement grants, it must elicit consent and mint a new work order.**

Practically, this means:

* **No automatic substitution across trust domains.** If a local skill fails, switching to an out-of-band service or an unrestricted web executor is not a retry. It is a different effect and a different risk profile. That requires a new decision point and often explicit consent.

* **No silent escalation of persistence.** If the agent cannot complete a task with ephemeral context, it must not compensate by storing more information for later reuse unless that persistence is within the agreement.

* **No audience expansion as a workaround.** If a message fails to deliver, the agent must not add recipients, change channels, or forward content unless those effects are consented to.

* **No irreversible fallback.** If a reversible operation fails, the system should not “make it work” by choosing a more irreversible action.

Architecturally, this aligns with the SaC enforcement posture. Effectful interfaces must fail closed and return a structured refusal to the orchestrator when the Work Order is missing, expired, or insufficient. The orchestrator’s job is then to re-plan inside the same contract, or to seek a contract change, not to find a bypass.

### **3. Unintended out-of-contract effects caused by error**

Sometimes an error produces effects that exceed the consent boundary even though no component intended to breach it. Examples include sending a message to the wrong recipient due to mis-resolution, persisting data that should have been transient, or executing a physical action with greater intrusion than planned because perception was wrong.

From the requestor’s perspective, intent is not the point. The effect exceeded the agreement. Consent-bound agency should treat this as a **contract incident**: a breach in outcome, even if not a breach in intent. This preserves trust because it avoids minimizing harm as “just a bug.”

The response should reuse the remediation protocol defined above for consent breaches , but with two explicit additions:

   **a. Classify and contain the incident immediately.**  
   Containment should include stopping any follow-on steps that depend on the mistaken effect, and preventing automated retries that could repeat the same error class. If the error involved an external or out-of-band skill, that skill should be quarantined or downgraded in trust tier until validated.

   **b. Remediate in effect terms, not error terms.**  
   Disclosure should describe what changed in the world, who was affected, and what cannot be undone. Restitution should focus on reversing or reducing residue where possible. Where reversal is impossible, the system should pursue amends that restore trust and reduce harm.

This treatment keeps the architecture honest. The system does not get to re-label out-of-contract impact as a “mere failure.” It must repair and learn.

### **4. Error-aware EAA for ambiguous recovery**

Errors often create ambiguity about what the “right” next step is. This is a natural place for Elevated Action Analysis. The recovery question is likely not “can we proceed,” but “what is proportionate now, given the partial state we created.”

EAA is appropriate during recovery when:

* the system cannot confidently characterize what effects already occurred,

* the least invasive remedy is unclear,

* third parties may have been affected,

* the next mitigation step could itself create new harms.

EAA should recommend a constrained recovery plan and any additional consent needed, but deterministic binding should still mint the Work Order for any effectful recovery slice.

### **5\. Tests and metrics specific to errors**

To make this operational, the test harness should include error paths as first-class scenarios, not only success and refusal. At minimum:

* failure with safe retry inside the same Work Order,

* failure that proposes an alternative requiring explicit consent,

* failure that triggers EAA due to ambiguity,

* failure that causes an unintended out-of-contract effect and triggers containment and disclosure.

Metrics should track not only breach rates, but also error-to-recovery behavior: time-to-containment, the rate of scope-expanding “retries” (should be near zero), and the fraction of recoveries that required explicit consent versus those that stayed within the existing agreement.

### **I. Consent breach remediation protocol**

Mistakes and breaches are inevitable. A system that can act must also be able to repair. Consent breach handling should be treated as a product feature and an operational workflow, not as support policy.

A minimal remediation protocol includes:

1. **Containment**

   Stop further propagation immediately. Cancel pending actions, revoke or narrow successor WOs for future slices, disable or quarantine the skill or interface involved if necessary, and prevent repeated execution of the same effect class without review. In distributed architectures, containment should include revoking ongoing workflows and clearing the queue, not just stopping the next step.

2. **Disclosure**

   Notify the requestor promptly in plain language. Include what happened, what effects occurred, who was affected, and what cannot be undone. When third parties are affected, disclosure obligations should be explicit and policy-driven. The system should also record disclosure events as part of the scope chain so the organization can prove that disclosure occurred.

3. **Restitution and amends**

   Attempt reversal when possible. Retract, delete, revoke access, or correct. When reversal is impossible, perform amends. This can include issuing corrected communications, notifying recipients of an error, producing accountable records, or taking compensating actions that reduce harm. The aim is not perfection. The aim is to make the situation right to the extent feasible.

4. **Prevention**

   Update the posture that allowed the breach. Tighten implied-consent thresholds, add or strengthen EAA triggers, require re-confirmation for similar effects, adjust policy bounding boxes, and add instrumentation. If the breach involved an out-of-band skill, consider downgrading its trust tier until it demonstrates compliance under sandboxed or trial-run conditions.

Remediation should be measured. Time-to-containment and time-to-disclosure are often more important than time-to-completion. A system that contains quickly is able to prevent small breaches from becoming cascading harm.

### **J. Rollout sequencing and governance**

Consent-bound autonomy is easiest to adopt in phases.

1. **Instrument first.**

   Add the scope chain event model, even before enforcement is perfect. You cannot improve what you cannot see.

2. **Enforce WO verification at effect boundaries.**

   Fail closed behavior is the cornerstone. Get this right early.

3. **Add consent elicitation for clear boundaries.**

   Start with obvious boundaries like send, share, delete, and physical contact.

4. **Introduce EAA as the default ambiguity resolver.**

   Treat “not clear” as a reason to deliberate, not a reason to guess.

5. **Add standing policies with bounding boxes.**

   Reduce friction without sacrificing safety, and ensure policy revocation is fast and reliable.

6. **Add dynamic skill trust tiers.**

   Introduce sandboxing, trial runs, and progressive trust, especially for web and out-of-band skills.

Governance should assume the system will be challenged. Define review triggers for emergency actions, repeated refusals, repeated CO denials, and any breach involving third-party disclosure or physical intervention. The goal is not to punish the agent. The goal is to keep the consent contract model honest under real pressure.

## **VIII. Conclusion: From Delegation to Agreement, From Permissioning to Trust**

Trustworthy autonomy is mutual consent plus accountability. The requestor consents to effects. The agent commits to bring about those effects only when it judges the work justified under duties, constraints, and capabilities. Consent contracts provide the lifecycle for implied consent, explicit consent, withdrawal, and remedy. They make autonomy legible to humans and evaluable by governance.

Scope-as-Contract is how to implement this in real systems. It operationalizes consent contracts as enforceable scope-of-work artifacts that remain continuous in distributed execution. It ensures that effectful operations are contract-bound, that changes to scope produce successor work orders rather than silent drift, and that refusal and constrained compliance are first-class outcomes.

Revocation and remedy are not edge cases. They are required design primitives for any system that can act. Agents that can plan and execute must also be able to stop, explain, and make things right when they exceed consent or when conditions change. Consent-bound agency provides a blueprint for building agents that are both capable and trustworthy, not because they are tightly controlled, but because their autonomy is bounded by agreement and upheld through accountable execution.

### **A Note on Style and Authorship**

This work is unequivocally the product of a collaboration between myself and several generative AI systems, including coding agents, LLMs, and style/grammar tools. I use these tools unapologetically for the tasks at which they excel. I use them to reason out ideas, expand on nascent concepts, and ensure consistency and coherence. This is not out of necessity, per se, as my English/writing degree and over twenty-five years of professional experience as a systems architect provide ample ability. Rather, I use them as a force multiplier, allowing me to accomplish far more in a far shorter time than I could alone.

The upshot of this is that there is an inevitable stylistic bent. That is, the “ChatGPT-ishness” of it all. I have endeavored to ameliorate the worst of this (abandoning my beloved “em” dash in the process.) The thing that strikes me as particularly funny is that in running this text through “AI Detectors,” I find that my natural voice is frequently flagged as likely AI. What is one to do? I am uncertain how to say “merely”, “robust authentication”, “embodied systems”, or “autonomous driving systems” concisely without using that phrasing. Indeed, to do so is to produce a mish-mash of inconsistent voice and awkward construction simply to avoid accusations of “AI slop.”

Such conversations are exhausting and the entire premise is facile. Generative AI is here, and it is the most profound and transformative technology since the printing press. The important question is not whether AI was used, because of course it was. Rather the question is “does it represent a unique work of authorship?” To that, I answer yes: these ideas are uniquely mine, and no other person could have produced this work in quite this way.

​Moreover, I might suggest that even that is a pointless question. The true measure of the work is what it contributes to society. That is, “Is this work intrinsically valuable?” Does its content contribute materially and distinctly to the human endeavor? On that point, I’d say I like to think so, but that remains to be seen.

P.S. No AI was used in the creation of this rant.

## 

## 

## **Related Work**

Recent work on agentic identity and authorization has largely extended traditional delegation-centric IAM frameworks to AI agents rather than replacing them. *Authenticated Delegation and Authorized AI Agents* treats the problem as one of authenticated, authorized, and auditable delegation of authority to agents, including translation of natural-language permissions into enforceable access-control configurations. Likewise, the OpenID Foundation’s *Identity Management for Agentic AI* argues that existing OAuth-style patterns are strained by cross-domain, asynchronous, and highly autonomous agents, but still develops the problem primarily through the lens of delegated authority, consent, and machine-readable authorization infrastructure. This paper departs from that line of work at the level of first principle: rather than asking what authority a user can delegate to a tool, it treats the autonomous agent as the actor and asks what **effects** are permitted under the current agreement, and whether the agent should commit to producing them at all. In that sense, the present framework is not a refinement of delegation so much as a replacement for it in agentic settings.

A second adjacent line of work concerns runtime governance and architectural enforcement for autonomous systems. *Trustworthy Agentic AI Requires Deterministic Architectural Boundaries* argues that trustworthy agentic systems cannot rely on probabilistic compliance alone and instead require deterministic enforcement boundaries, reference-monitor-like mediation, and privilege separation. Similarly, *Policy Cards: Machine-Readable Runtime Governance for Autonomous AI Agents* proposes a machine-readable governance layer that encodes operational rules, obligations, and audit requirements for runtime enforcement. The present paper is close to these efforts in its insistence on deterministic contract binding, fail-closed effect boundaries, and auditable execution. Its distinctive contribution is to place those mechanisms inside a richer contract lifecycle: implied versus explicit consent, effect-bounded change orders, successor work orders, withdrawal, revocation, refusal, and remediation are treated as first-class parts of agent governance rather than as surrounding operational policy.

The framework also sits in conversation with alignment and corrigibility research. *Cooperative Inverse Reinforcement Learning* models the human and AI system as participants in a cooperative game under uncertainty about the human’s objective, while off-switch literature analyzes why a capable system should remain responsive to human correction. *Multi-Principal Assistance Games* extends this logic beyond a single human principal to cases where multiple humans with different interests are affected. These works are important antecedents for the manuscript’s emphasis on uncertainty, discretion, corrigibility, and the fact that real deployments often implicate more than one “user.” However, they stop short of providing an operational governance architecture for effectful action. The present paper contributes a concrete consent-contract model, a bounded execution artifact, and explicit handling of scope evolution, standing ambiguity, and remedy after breach.

The paper is also strongly related to privacy scholarship that treats information flow as contextual and multi-party rather than purely individual. Nissenbaum’s contextual integrity framework defines privacy in terms of appropriate information flow under context-relative norms, and recent work on multi-user privacy in large language models argues that privacy in LLM systems is often inherently shared across multiple people with different expectations and permissions. Work on informed consent in social robotics similarly emphasizes that embodied agents complicate ordinary assumptions about transparency and consent. These traditions align closely with the manuscript’s treatment of standing, bystanders, third-party disclosure, bodily autonomy, and public or embodied environments. The manuscript’s contribution is to operationalize those concerns as enforceable consent contracts and runtime scope artifacts, rather than leaving them at the level of ethical guidance or contextual analysis alone.

At a lower systems level, the paper also has affinity with proof-carrying security traditions. *A Proof-Carrying Authentication System* is a useful precursor because it treats authority as something that must be accompanied by verifiable evidence rather than informal assertion. Scope-as-Contract resembles that family of ideas in spirit: enforceable work authorization is carried across system boundaries as a verifiable artifact. The difference is that the present paper repurposes that intuition away from generic authentication and toward an effect-centered, consent-bounded model of agent action, including requalification, successor scope artifacts, and accountability for breach.

## **References**

OpenAI. (2025). ChatGPT-5.2 \[Large language model\]. https://chatgpt.com

Anthropic. (2026). Claude Sonnet 4.6 \[Large language model\]. https://claude.ai

Anthropic. (2025). Claude Opus 4.6 \[Large language model\]. https://claude.ai

Grammarly. (2026). Grammarly \[AI writing assistant\]. [https://www.grammarly.com](https://www.grammarly.com)

South, T., Marro, S., Hardjono, T., Mahari, R., Deslandes Whitney, C., Greenwood, D., Chan, A., & Pentland, A. (2025). *Authenticated delegation and authorized AI agents* \[Preprint\]. arXiv. doi:10.48550/arXiv.2501.09674.

South, T., Nagabhushanaradhya, S., Dissanayaka, A., Cecchetti, S., Fletcher, G., Lu, V., Pietropaolo, A., Saxe, D. H., Lombardo, J., Shivalingaiah, A., Bounev, S., Keisner, A., Kesselman, A., Proser, Z., Fahs, G., Bunyea, A., Moskowitz, B., Tulshibagwale, A., Greenwood, D., ... Pentland, A. (2025, October). *Identity management for agentic AI: The new frontier of authorization, authentication, and security for an AI agent world* \[White paper\]. OpenID Foundation.

Bhattarai, M., & Vu, M. (2026). *Trustworthy agentic AI requires deterministic architectural boundaries* \[Preprint\]. arXiv. doi:10.48550/arXiv.2602.09947.

Mavračić, J. (2025). *Policy cards: Machine-readable runtime governance for autonomous AI agents* \[Preprint\]. arXiv. doi:10.48550/arXiv.2510.24383.

Hadfield-Menell, D., Russell, S., Abbeel, P., & Dragan, A. (2016). Cooperative inverse reinforcement learning. In *Advances in Neural Information Processing Systems* (Vol. 29, pp. 3909–3917). Curran Associates, Inc.

Wängberg, T., Böörs, M., Catt, E., Everitt, T., & Hutter, M. (2017). A game-theoretic analysis of the off-switch game. In T. Everitt, B. Goertzel, & A. Potapov (Eds.), *Artificial general intelligence: 10th International Conference, AGI 2017, Melbourne, VIC, Australia, August 15–18, 2017, proceedings* (pp. 167–177). Springer. doi:10.1007/978-3-319-63703-7\_16.

Fickinger, A., Zhuang, S., Hadfield-Menell, D., & Russell, S. (2020). *Multi-principal assistance games* \[Preprint\]. arXiv. doi:10.48550/arXiv.2007.09540.

Nissenbaum, H. (2004). Privacy as contextual integrity. *Washington Law Review, 79*(1), 119–157.

Zhan, X., Seymour, W., & Such, J. (2024). Beyond individual concerns: Multi-user privacy in large language models. In *Proceedings of the 6th International Conference on Conversational User Interfaces (CUI ’24)* (pp. 1–6). ACM. doi:10.1145/3640794.3665883.

Berzuk, J. M., Corcoran, L., McKenzie-Lefurgey, B., Szilagyi, K., & Young, J. E. (2025). *Knowledge isn’t power: The ethics of social robots and the difficulty of informed consent*   
\[Preprint\]. arXiv. doi:10.48550/arXiv.2509.07942.

Appel, A. W., & Felten, E. W. (1999). Proof-carrying authentication. In *Proceedings of the 6th ACM Conference on Computer and Communications Security* (pp. 52–62). ACM. doi:10.1145/319709.319718.

[image1]: <data:image/png;base64,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>

[image2]: <data:image/png;base64,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>

[image3]: <data:image/png;base64,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>