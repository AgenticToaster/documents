# Consent-Bound Agency: Technical Brief

Author: Chris Sorel, Co-Founder, CTO & Chief Architect, Loosh.ai

**Author’s Note.** This paper is a condensed version of the full manuscript, *Consent-Bound Agency*, which develops the framework in greater detail. See references for link.

## Executive Summary

AI systems are crossing a threshold. They are no longer limited to answering questions, drafting content, or summarizing documents. They can now plan, invoke tools, update durable state, communicate with third parties, and increasingly act in ways that have direct consequences in the world. Once a system can act, the design problem changes.

The central question is no longer just whether the system can perform a task. The real question is this: **under what agreement is it allowed to produce effects in the world?**

Traditional enterprise patterns answer that question with authentication, authorization, and delegation. Those patterns assume the human is the actor and the system is a tool operating inside the user’s permission boundary. That model works well enough for dashboards, forms, and APIs. It works much less well for autonomous agents.

An agent is not merely a remote-controlled tool. It interprets requests, fills in gaps, chooses intermediate actions, encounters ambiguity, and may affect people other than the requestor. It may send messages, disclose information, store memories, alter records, open doors, or initiate physical interventions. In those settings, delegated permission is an unstable foundation. It is too coarse when the agent needs discretion, and too permissive when the agent overreaches.

This is no longer a hypothetical concern. We have already seen stories of agents doing wildly inappropriate things in the name of being helpful: deleting an entire email inbox, taking broad cleanup actions when only a narrow correction was intended, or turning a loosely phrased request into an irreversible outcome. These failures are often described as product bugs, prompt failures, or alignment misses. But underneath them is a deeper architectural problem. The agent does not reliably understand the consent implied by the request, and it does not operate under a deterministic mechanism that constrains the scope of effect.

We propose **Consent-Bound Agency** as an alternative foundation for trustworthy autonomy.

The core claim is simple: autonomous systems should not be governed primarily by delegated permission. They should be governed by **agreement over effects**.

In this model, the requestor is the party asking for work and bearing the consequences. The agent is the actor that decides whether and how to perform that work. The governing question is not “what rights does the agent have?” but **“what effects are permitted under the current agreement?”**

That agreement is formalized through **consent contracts** and operationalized through **Scope-as-Contract (SaC)**, a runtime model that binds execution to an explicit, auditable scope artifact for each bounded slice of work. The result is a blueprint for building agents that are both autonomous and accountable.

##  1\. The Problem: Delegation Breaks Down in Autonomous Systems

Most enterprise security models are built on a familiar story. A user authenticates, the system evaluates permissions, and the software acts as an extension of that user’s authority. This is delegation.

That story begins to break down the moment the system is no longer following a direct instruction path. Modern agents infer intent from incomplete requests, generate plans, choose tools and sequences, discover information mid-flight, create durable state, and sometimes operate in real-world environments with bystanders and contested roles. Once that happens, the system is no longer simply executing commands. It is exercising discretion.

That creates a serious mismatch between the governance model and the behavior of the system.

Delegation assumes the user is the actor and the system is a tool. But an autonomous agent is the one interpreting, selecting, and executing. That makes the agent the practical actor, whether or not the architecture acknowledges it. If the system is treated as a mere tool while acting like an autonomous decision-maker, governance becomes incoherent.

The problem becomes even clearer when we look at permissions. A permission like “can send email” is not the same thing as a justified effect. Sending a draft to one recipient is different from adding recipients, forwarding externally, or storing the exchange for future reuse. The mechanism may be identical, but the impact is not. People do not usually care about internal mechanism names. They care about what changes in the world.

The same problem appears with dynamic least privilege. An agent’s needs change from step to step. Static permission bundles are either too broad or too narrow. Dynamic permissions help only if they are grounded and enforceable. If they are generated probabilistically or inferred loosely, they become a vehicle for hallucination and overreach.

This is why so many early agent failures feel familiar. The system looks impressive in a demo, because broad permissions make it fluid and adaptable. Then it enters production, where ambiguous intent, real data, and real consequences expose the governance gap.

Consider an inbox-cleanup example. A user asks an agent to “clean up my email” or “get rid of the junk.” A delegated-permission architecture sees a mailbox, a delete capability, and an apparently valid request. A helpful agent may infer that mass deletion is a reasonable interpretation. But that is not how people understand the request. In ordinary human interaction, “clean up my inbox” might imply triage, archiving, categorization, or drafting a proposed cleanup plan. It does not ordinarily imply irreversible deletion of everything in sight. The failure is not just one of model reasoning. It is a failure to distinguish between the consent implied by the request and the effect ultimately produced.

This is the core problem. Autonomous agents are being asked to act in ways that affect people, records, resources, and environments, but the dominant governance model still treats them as if they were simple delegated tools.

##  2\. The Shift: From Permissioning to Agreement

Consent-Bound Agency begins with a change in framing.

Instead of asking, **What authority should we delegate to the agent?** we ask, **What effects are reasonably permitted by the request, and should the agent commit to producing them?**

That shift matters because trustworthy autonomy is not only about what the user allows. It is also about what the agent should agree to do.

A request can be explicit and still unjustified. A user can approve an action that is unsafe, overbroad, or harmful to others. A trustworthy agent must be able to do more than comply. It must be able to refuse, constrain, clarify, or escalate.

That is why consent in this model is mutual.

The requestor consents to certain effects on their interests. The agent, in turn, commits to producing those effects only when the work is justified under the relevant constraints, duties, and capabilities. This is not anthropomorphic rhetoric. It is a practical way to encode accountability.

If the agent cannot say no, it is not autonomous in any meaningful sense. It is merely obedient automation with better language skills.

This way of thinking immediately changes the design conversation. Instead of asking whether the agent has a permission token, we begin asking more useful questions. What is actually being asked? What effects are implied by that request? What effects exceed the ordinary scope of consent? Who else could be affected? What must be clarified before action is justified?

Those are the right questions for real-world systems.

## 3\. Consent Contracts: The Core Governance Primitive

Consent cannot be a one-time checkbox for systems that work over time, change plans, and discover new context mid-execution. It has to behave like a living agreement.

That agreement is the **consent contract**.

A consent contract describes what effects are in scope, what is implied by the request, what requires additional agreement, what standing policies apply, when work must stop, and what happens when the system exceeds the agreement. It is the governing object that ties a user’s request to an agent’s bounded commitment.

This matters because most real work is not performed in a single atomic step. A request unfolds. New facts appear. Side effects become available. The agent discovers possibilities the user did not explicitly mention. In a conventional agent architecture, those moments are often resolved implicitly and opportunistically. In a consent-bound architecture, they are handled through the evolving contract.

One of the most important distinctions here is between **implied consent** and **explicit consent**.

Humans already navigate this distinction naturally. If someone asks for an email draft, it is reasonable to infer consent for composition. It is not reasonable to infer consent for sending. If someone asks a household robot for water, it may imply navigation and object manipulation. It does not automatically imply entering private rooms, opening locked storage, or recording the environment for future reuse.

Consent-Bound Agency formalizes this distinction. Implied consent covers the minimal, ordinary effects needed to fulfill the request. Explicit consent is required when the next step crosses an effect boundary.

Those boundaries matter. Turning preparation into external action, expanding the affected-party set, increasing persistence or future reuse, introducing irreversibility, or entering a higher-risk domain all mark boundaries of effect scope.

This is why the framework is effect-centered by design. People do not consent to API calls, tool invocations, or internal mechanism labels. They consent to effect; disclosure, persistence, physical contact, deletion, external communication, or other real-world impacts. That distinction keeps consent human-legible while allowing implementation details to vary.

We find a useful analogy for consent contracts in service contracting, and adopt some of its vocabulary because it is both intuitive and well aligned with the functions at play. In this lifecycle, the **Purchase Order (PO)** is the request as expressed by the requestor, the **Work Order (WO)** is the agent’s bounded scope of work for the current slice of execution, and the **Change Order (CO)** is the explicit-consent step required when the next slice would exceed the current implied boundary.

This language is intuitive because it mirrors how real organizations already manage changing scope, accountability, and work continuity. It also has an important conceptual advantage: it makes clear that the agent is not just “using tools.” It is undertaking work under a bounded agreement.

## 4\. Scope-as-Contract: How Consent Becomes Enforceable

Good governance models often fail at runtime because they exist in policy documents, prompts, or planner logic, but not in the components that actually produce effects. This problem is especially acute in agentic systems because they are both dynamic and modular. Effectful action can occur across many components, services, and subsystems, each with its own boundary.

Microservices offer a useful analogy. Each service operates within a bounded task space and acts based on the request scope and authentication context presented to it. But as discussed, the delegated permission model is not granular enough to handle the fluid scope of agentic requests. When systems are empowered to produce effects while also using probabilistic methods to determine the scope of action, overreach is practically inevitable. If inference is allowed to define scope, and the system is granted broad permissions to support that evolving scope, it will tend to grant itself excessive authority in the name of being helpful. It is the fox guarding the hen house.

That leads to the central runtime challenge in autonomous systems: **how do we constrain subsystems to a least-privilege scope of action appropriate to the current request?**

**Scope-as-Contract (SaC)** is the architectural answer.

SaC turns the active consent contract into an explicit runtime artifact: the **Work Order**. A Work Order is not just an auth token. It is a tamper-evident, machine-verifiable representation of the current agreement for a bounded slice of work. Its job is to preserve contract fidelity across distributed execution.

A well-formed Work Order does three things. It provides continuity by binding the current slice to the request context and agreement state. It provides constraint by granting only what is needed for that slice. And it provides auditability by allowing the system to reconstruct what agreement governed the action.

This is a major design shift. Instead of operating under broad ambient permissions, the agent operates through a chain of explicit scope artifacts, each tied to the current contract.

The most important enforcement principle in SaC is this: **inference may propose a scope of effect, but it must not mint authority.** Large models can recommend, explain, and forecast. They should not directly produce the grants that govern effectful execution.

A deterministic binder is therefore responsible for minting each Work Order from grounded inputs: the current request context, the active contract state, relevant standing policies, verified consent records, registered skill metadata, policy prohibitions, and, when applicable, a bounded adjudication result from elevated analysis.

This avoids the rubber-stamp failure mode where a supposedly safe system simply ratifies whatever the reasoning layer asks for.

The inbox-deletion example makes the need for this very concrete. Suppose the reasoning layer concludes that deleting thousands of messages is the fastest way to “clean things up.” In a conventional architecture, that reasoning may be enough to trigger action if the mailbox capability is present. In a consent-bound architecture, it is not enough. The reasoning layer may propose a cleanup scope, but the binder still has to ask whether deletion is within the current consent contract. If the contract supports triage, labeling, archiving, or draft recommendations but not irreversible deletion, the Work Order for mass delete is never minted. The boundary is enforced in architecture, not merely suggested in prompt text.

That same principle applies beyond email. A CRM agent may have the mechanical ability to update customer records, but that does not mean it should rewrite account histories because a sales rep asked it to “clean this up.” A robotics agent may be able to unlock a door, but that does not mean a request to “go get the package” implies entering restricted space. A medical workflow agent may have access to patient communication tools, but that does not mean it should disclose diagnoses to third parties without a justified basis. In each case, the problem is the same: capability is being mistaken for consent.

SaC addresses that by forcing execution to pass through effect boundaries that are deterministically checked.

Any component that can produce a material change or other significant impact should be treated as an **effectful interface**. That includes components that can read sensitive state, write durable state, send external messages, invoke irreversible operations, or actuate in the physical world. Each such interface must verify the active Work Order and refuse work outside it. When verification fails, the system must fail closed.

That refusal is not the end of the workflow. It is a contract signal. The agent can then revise, clarify, request consent, requalify scope, run elevated analysis, or refuse the task entirely.

This is what turns consent from aspirational into constrained action.

##  5\. Elevated Action Analysis: The Agent’s Deliberate Mode

Not every request is routine. Many are ambiguous, contested, or high-risk. An agent must make determinations about scope of action all the time. Who is actually authorized to make this request? Does this step expand the effect boundary? Are there bystanders or third parties whose interests are implicated? Do safety obligations conflict with user instructions? Is the environment uncertain enough that acting now would be overreach?

These are not edge cases. They are the normal conditions of real-world autonomy.

That is why Consent-Bound Agency introduces **Elevated Action Analysis (EAA)**.

EAA is the agent’s deliberate mode for forming its own commitment under uncertainty. It is the point at which the system stops treating the task as routine execution and starts treating it as adjudication.

EAA is appropriate when the system encounters standing ambiguity, effect ambiguity, insufficient evidence, duty collisions, emergency time pressure, uncertain or novel tool behavior, or irreversibility. In other words, it is triggered precisely when the cost of naive helpfulness becomes unacceptable.

What EAA does is straightforward in concept, even if the implementation is sophisticated. It classifies the action and affected parties, performs constrained discovery, evaluates standing, risk, and duties, selects the least invasive sufficient action, chooses an explicit outcome, and generates accountability artifacts.

The set of possible outcomes is intentionally limited. The agent may proceed. It may request explicit consent. It may proceed under a narrow emergency implied-consent theory. It may refuse. Or it may escalate.

This is important because ambiguity is where most governance models quietly collapse. Without a deliberate mode, agents resolve ambiguity in two bad ways: by over-complying, or by becoming unusably hesitant. EAA creates a middle path. It allows the system to slow down when the contract is unclear and then produce a justified, auditable next step.

## 6\. Policies: Useful Shortcuts, Not Blank Checks

Autonomous systems must be usable, not just safe. If every repeated action requires a fresh approval step, the system becomes tedious and brittle.

Consent-Bound Agency addresses this with **standing policies**.

Policies act as standing contract terms. They can represent cached consent from the requestor, cached intent that the agent has proposed and the user has confirmed, or system policies that represent non-negotiable obligations. “Always cc my attorney on legal emails.” “After folding laundry, put it away.” “Do not destroy evidence.” These are all examples of policies that reduce friction without abandoning control.

But a trustworthy policy is not just a rule. It has a bounding box.

A proper policy should specify provenance, applicability conditions, effect scope, escalation rules, expiry or review terms, and revocation semantics. That bounding box is what keeps policies from becoming silent scope expansion.

This is an easy place for platforms to get into trouble. A product team adds convenience features, learns a pattern from user behavior, and gradually turns the agent into something that acts on increasingly broad assumptions. The system feels personalized, but its effective scope has drifted far beyond what the user consciously agreed to. In a consent-bound system, that drift is treated as a governance failure, not a personalization success.

Policy precedence also matters. Convenience cannot override duty. A practical order of operations is to evaluate system policy first, then standing and context checks, then elevated-analysis triggers, and only then user and confirmed policies. That ordering ensures that cached convenience does not defeat more fundamental obligations.

## 7\. Extensibility Without Losing Control

One of the hardest problems in agent architecture is extensibility. A fixed tool set is easy to govern. An open ecosystem of dynamic skills, web-connected services, plug-ins, and external executors is not.

This matters because the moment an agent can call arbitrary tools, the platform’s control story often becomes much weaker than it appears. A governance layer may look elegant at the orchestrator level while the actual effect surface is sprawling, porous, and only lightly understood.

Consent-Bound Agency does not solve this by banning extensibility. It solves it by requiring skills to be legible in two ways.

Every skill should declare what it requires to run, which is its enforcement surface, and what kinds of impact it can produce, which is its effect profile. That lets the agent reason in effect language while the runtime enforces through minimal grants.

Just as important, registration is not the same thing as trust.

Simply registering a skill should not make it trusted. Skills should be categorized by trust tier: in-process, sandboxed, internal infrastructure, or external and out-of-band. The more execution leaves the agent’s enforcement boundary, the weaker fine-grained governance becomes. That means certain skills should require tighter scope, additional consent, trial runs, progressive trust, or outright refusal in sensitive contexts.

For technical leaders, this is not just a safety issue. It is a platform issue. If your agent platform supports arbitrary tools without a clear model of effect profiles and trust domains, you do not really have policy control. You have a best-effort illusion of control layered over a broad execution surface.

## 8\. What This Looks Like in Practice

A mature consent-bound system should feel different from today’s typical agents.

Routine tasks should complete with low friction. Effect boundary crossings should trigger focused consent prompts rather than generic approval spam. Ambiguous or high-risk situations should trigger deliberate analysis. Refusal should be normal and explainable. Revocation should cleanly stop future work. Breaches should trigger containment, disclosure, restitution, and prevention.

The core runtime lifecycle is simple: **propose → consent → work order → execute → revoke or remedy**.

That lifecycle can be implemented in digital assistants, enterprise copilots, workflow agents, robotics systems, or mixed physical-digital platforms. The implementation details differ, but the governing logic is the same.

Imagine an enterprise copilot asked to “deal with my overflowing inbox.” A mature system might first propose a scope: identify newsletters, low-priority updates, and stale threads; draft a cleanup plan; optionally archive categories after confirmation; and never permanently delete messages without a separate, explicit instruction. That proposal becomes the working contract. If the user later says “yes, archive those categories,” the agent can request a new Work Order for archive actions. If it wants to delete instead of archive, that is a Change Order because the effect boundary has changed.

Or consider a household robot asked to “put away the groceries.” Ordinary implied consent may cover carrying bags, opening cabinets normally used for food, and placing items in standard storage locations. It may not cover entering a locked room, moving unrelated personal belongings, or discarding items it deems expired without asking. Again, the agent’s mechanical capability is not the issue. The issue is whether the effect falls within the current agreement.

These examples are mundane by design. That is the point. Trustworthy autonomy will not be judged only in extreme situations. It will be judged in ordinary situations where overreach feels unnecessary, surprising, and avoidable.

A serious deployment should therefore measure the system as a governance mechanism, not just as a task-completion engine. Useful metrics include consent prompts by effect class, denial and abandonment rates, elevated-analysis trigger categories, Work-Order verification success rates, attempted out-of-contract executions, breach rates by effect class, time to containment and disclosure, and dynamic skill usage by trust tier.

That is how the model becomes falsifiable rather than aspirational.

## 9\. Why This Matters Now

The industry is entering a period where the distinction between assistant and actor is collapsing.

Agents already draft messages, update systems of record, manage workflows, coordinate external actions, and operate over increasingly persistent memory and tool access. Soon they will do more of this with less supervision and higher stakes.

That makes today’s architectural choices unusually important.

If the industry continues to govern agents primarily as delegated tools, it will keep rediscovering the same problems: overbroad access, unclear accountability, fragile policy layers, ambiguous user standing, poor handling of third-party effects, and unsafe runtime improvisation.

Consent-Bound Agency offers a better organizing principle. It does not eliminate risk. It makes the system honest about where risk lives, how scope changes, who is affected, and what agreement is actually in force.

That is a stronger foundation for safety, governance, product trust, and public legitimacy.

It is also, importantly, a better fit for how technical executives actually think about operational systems. Organizations already understand bounded work, approval gates, change orders, auditability, duty separation, and controlled execution. Consent-Bound Agency does not ask institutions to abandon those intuitions. It asks them to apply them to autonomous systems in a form that matches how those systems actually behave.

## 10\. Bottom Line

The central claim of this litepaper is simple: **trustworthy autonomy should be governed by agreement over effects, not just by delegated permission.**

That leads to a practical blueprint. Treat the agent as the actor. Govern action through mutual consent. Express scope in effect terms. Bind execution to explicit Work Orders. Keep deterministic authority separate from probabilistic reasoning. Require deliberate analysis under uncertainty. Make refusal, revocation, and remedy first-class parts of the system.

That is what Consent-Bound Agency provides.

For technical leaders, the appeal is not philosophical elegance alone. It is that this model scales better to the realities of modern agents: distributed execution, dynamic tools, third-party effects, long-running workflows, ambiguous standing, and real accountability.

If agents are going to act in the world, they need more than permissions.

They need contracts.

## References

Sorel, C (2026). *Consent-Bound Agency: A Blueprint for Trustworthy Autonomy* \[White paper\].  
https://agentictoaster.github.io/documents/\#/whitepapers/consent\_bound\_agency.md
