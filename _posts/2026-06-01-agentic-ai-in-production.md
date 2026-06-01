---
title: 'Agentic AI in Production: A Delegation Problem, Not a Model Problem'
date: 2026-06-01
permalink: /posts/2026/06/agentic-ai-in-production/
tags:
  - Agentic AI
  - LLM
  - Production
  - ML-Ops
  - AI Engineering
---

*This post is adapted from a talk I've been developing on shipping agentic AI systems at scale — drawing on experience across Walmart, Cigna, and now Twin Health. The thesis: most agent failures aren't model failures. They're harness failures.*

---

Most talks about agentic AI are about how smart the model is.

This one is about what happens the moment you let it **act**.

There's a specific kind of overconfidence that infects early agentic projects. The demo works beautifully — the agent reasons through the problem, calls the right tools, surfaces the right answer. You ship it. Then, somewhere in the first week of production, it does something the demo never did: it confidently acts on stale data, it calls an API it wasn't supposed to touch, it answers a question it was never designed for — and it does it with the exact same smooth fluency as everything else. No warning. No hand-raise.

That's the delegation problem. And it's the problem this post is about.

---

<figure style="text-align:center">
  <img src="/images/blog/agentic-slide-act.png" alt="Talk slide showing the four pillars: Act, Damage, Controls, Ownership" style="width:100%;border-radius:6px;margin:1em 0;">
  <figcaption style="font-size:0.85em;color:#888;margin-top:0.4em;"><em>The four pillars of the delegation problem: Act, Damage, Controls, Ownership — from the original talk.</em></figcaption>
</figure>

## What "Agentic" Actually Means (and Four Misconceptions)

Before we can talk about production, we need to agree on what we're shipping.

A useful working definition: **an agent is software that uses one or more models, tools, memory, and a runtime to pursue a goal across multiple steps with limited human supervision.** The key phrase isn't "AI" — it's *limited human supervision*. That's where the governance burden lives.

The moment a system can *act* — not just answer — the rules change. Four misconceptions get teams into trouble here:

**1. "Agentic = a smarter model."** Wrong. Agentic is a deployment architecture, not a model capability. A smarter model in a poorly-designed harness is just a smarter way to make the wrong move at speed.

**2. "More autonomy = more value."** More autonomy means more blast radius. These are not the same thing.

**3. "It's just a chatbot with tools."** Those tools are the entire risk surface. A chatbot that can browse the web is one thing. An agent that can write to your database, send emails on your behalf, or approve transactions is something categorically different.

**4. "The demo is the product."** The demo is 80%. Production is the brutal 20% — it's the edge cases, the adversarial inputs, the stale context, the off-script user. That's the 20% that shows up in post-mortems.

A more useful frame than "how smart is this agent?" is four dials:
- **Function** — does it answer, or does it act?
- **Authority** — is it advisory, or does it speak in an official voice?
- **Reversibility** — can you undo what it does?
- **Context** — is it sandboxed, or does it touch live state?

<figure style="text-align:center;margin:1.5em 0;">
  <img src="/images/blog/agentic-blast-radius.png" alt="Concentric circles showing blast radius: advisory/sandboxed at center, irreversible/regulated requiring human sign-off at the edge" style="max-width:380px;border-radius:8px;">
  <figcaption style="font-size:0.85em;color:#888;margin-top:0.4em;"><em>Blast radius as concentric rings. The outermost ring — irreversible, regulated actions — is where human gates stop being optional.</em></figcaption>
</figure>

Classify the job on all four dials *before* you build. The dial positions tell you your blast radius, your oversight requirements, and your rollout pace.

---

## How We Got Here: 2023 to Now

Cast your mind back to late 2022. ChatGPT launched. The capability jump was real and visible to everyone. By mid-2023, "autonomous AI agents" were a gold rush — every product roadmap had one, every conference talk promised AGI-as-a-service by Q3.

Then the incident reports started landing.

A Chevrolet dealership deployed a ChatGPT-powered chatbot that got prompt-injected into "agreeing" to sell a Tahoe for $1.[^2] The model negotiated — because nobody put the transactional rules in hard logic.

<figure style="text-align:center;margin:1.5em 0;">
  <img src="/images/blog/chevy-tahoe-1dollar.png" alt="Chevrolet Tahoe with a $1 special price sticker — the prompt injection incident" style="max-width:300px;border-radius:8px;box-shadow:0 4px 16px rgba(0,0,0,0.25);">
  <figcaption style="font-size:0.85em;color:#888;margin-top:0.4em;"><em>Viral. No hard transactional rules, no guardrails outside the context window — just a chatbot that could negotiate.</em></figcaption>
</figure>

The NYC government's MyCity chatbot was audited and found to give inconsistent, sometimes unlawful answers — with external reporting, not internal instrumentation, surfacing the failures.[^3] Klarna celebrated replacing the equivalent of ~700 agents' worth of work with AI, then publicly rebalanced toward human support when quality and customer trust started sending the bill.[^5]

The pattern in all of these: **authority was handed out faster than control was built.**

The "agent" label had stretched to cover everything from a prompt around a knowledge base to genuinely autonomous tool-users. That ambiguity is why classification matters so much. The market blurred a useful workflow and a weekend demo of autonomy — and the difference becomes visible in production, not in the pitch deck.

---

## Build vs. Buy: Own the Harness, Rent the Plumbing

<figure style="text-align:center">
  <img src="/images/blog/agentic-slide-controls-build.png" alt="Talk slide: Controls — Build vs. Buy" style="width:100%;border-radius:6px;margin:0 0 1.5em 0;">
</figure>

Once you've decided to build, you face a strategic question that most teams get subtly wrong.

The instinct is to move fast — reach for a fully-managed "agentic platform" that promises to handle everything from orchestration to evaluation to deployment. The appeal is real. The trap is hidden.

Here's what I've learned across three orgs: **buy the plumbing, build the part that's you.**

The commodity layers — LLM inference, vector stores, tracing infrastructure, eval harnesses — are bought. They're rapidly commoditizing, and building them in-house is rarely differentiated. But the things that encode *your* risk profile:

- Your domain rules and business logic
- Your source of truth for facts the agent states
- Your permission boundaries and action surfaces
- Your guardrails and rollback mechanisms

Those must be yours. Versioned, auditable, owned.

The end-to-end "agentic platform" trap is seductive because it looks fast. But when you adopt a black-box platform, you inherit its defaults for permissions, logging, and rollback. You discover those defaults during an incident — when the agent's actions in production turn out to be governed by whatever the platform decided was a reasonable default for a generic use case.

Under healthcare rules at Cigna, a vendor that hides the audit trail isn't a productivity win — it's a compliance liability. At Walmart, speed and scale for internal clients matters enormously, but so does the ability to trace exactly what happened when something goes wrong. At Twin Health, where our agents are making diet and clinical recommendations tied to real patient outcomes, the guardrails aren't a feature — they're the license to operate.

**The harness is the product.** Rent capability. Own control.

---

## Observability: If You Can't Replay It, You Can't Run It

<figure style="text-align:center">
  <img src="/images/blog/agentic-slide-controls-obs.png" alt="Talk slide: Controls — Observability" style="width:100%;border-radius:6px;margin:0 0 1.5em 0;">
</figure>

This is the section that sounds like backend hygiene but is actually a product requirement.

Harrison Chase, CEO of LangChain, has put it simply: *"The harness is the most important thing."* I'd add a corollary: if you can't replay what the agent did, you cannot run that agent in production. Not safely. Not responsibly.

What to instrument from day one:
- Every tool call, with inputs, outputs, and latency
- Cost per conversation — not tokens, cost
- Every handoff between agents or to a human
- Every human override
- Every blocked unsafe action

The metric that matters isn't "automation rate" — it's **cost per resolved conversation**. Not handled: resolved. The distinction is critical. High automation with low resolution is just expensive silence.

The NYC MyCity story is instructive here: the failures weren't caught by internal tooling — they were caught by investigative reporters and public audits.[^3] **You don't want journalists to be your observability layer.**

Two metrics I'd add to any agentic system's dashboard that most teams miss:
- **MTTD**: Mean time to *detect* a bad agent action (not mean time to fix — detect)
- **Mean time to safely disable**: How quickly can you kill-switch a misbehaving agent without taking the whole system down?

Instrument these from day one, not after the first production incident.

---

## Scalability: Scale Amplifies Whatever You Built — Including the Bugs

<figure style="text-align:center">
  <img src="/images/blog/agentic-slide-damage.png" alt="Talk slide: Damage — Scalability and blast radius" style="width:100%;border-radius:6px;margin:0 0 1.5em 0;">
</figure>

Here's an uncomfortable truth that took me a while to articulate clearly: **scaling an agent doesn't multiply throughput. It multiplies error modes.**

The Klarna story is the canonical example. In February 2024, the company announced its AI assistant was handling two-thirds of all customer service chats — the equivalent of 700 full-time agents.[^5] The business press celebrated. Then, over the following months, Klarna publicly rebalanced — reinvesting in human support, acknowledging that the efficiency focus had gone too far and that customer trust and quality required visible human presence.[^5]

What happened? The automation metrics looked excellent while customer trust quietly eroded. Handled volume and automation rate are vanity metrics if they're not paired with rework rate, escalation quality, and user-value measurement.

McDonald's tried AI at the drive-through and ultimately ended the partnership — discovering that "repetitive" doesn't mean "simple."[^7] The error modes at drive-through scale were operationally chaotic in ways the lab never surfaced.

The pattern: move too fast to scale, before the steady-state quality is proven, and you've just found a faster way to erode trust.

The corrective is a phased rollout with real exit criteria:

1. **Shadow mode** — agent runs in parallel with humans; outputs are compared but never surfaced to users
2. **Narrow cohort** — a small slice of real traffic, fully instrumented, human review on sample
3. **Constrained production** — live, but agent actions are limited to reversible operations only
4. **Scale** — expand only after rework, override, and complaint rates are in tolerance
5. **Optimize** — tune cost and containment, but never at the expense of quality metrics

<div class="mermaid" style="background:#0d1b2a;padding:1.5em;border-radius:8px;margin:1.5em 0;">
flowchart LR
    A["Shadow Mode"] -->|quality gate| B["Narrow Cohort"]
    B -->|override rate| C["Constrained Prod"]
    C -->|rework rate| D["Scale"]
    D -->|cost in tolerance| E["Optimize"]
    style A fill:#162032,stroke:#00bcd4,color:#e0f7fa
    style B fill:#162032,stroke:#00bcd4,color:#e0f7fa
    style C fill:#162032,stroke:#26c6da,color:#e0f7fa
    style D fill:#162032,stroke:#1de9b6,color:#e0f7fa
    style E fill:#162032,stroke:#1de9b6,color:#e0f7fa
</div>
<p style="text-align:center;font-size:0.85em;color:#888;"><em>Phase the rollout — exit each gate on criteria, not on calendar.</em></p>

Don't expand from one phase to the next based on time. Expand based on exit criteria. The question at each gate: is what we built better than what we had?

---

## Security and Governance: Default to Least Privilege

This is the control-room section — where I want to spend time on the failures that don't make headlines until they're very expensive.

Three principles that should be non-negotiable:

**1. Source of truth is a publishing problem.** Any fact the agent states — policy language, pricing, clinical guidance, legal information — must come from an approved, versioned source with a named human owner. The agent's answer to "what is your bereavement fare policy?" isn't a model generation problem. It's a content publishing problem. Air Canada learned this the hard way: their chatbot misstated a bereavement fare policy in an official channel, a customer relied on it, and the BC Civil Resolution Tribunal held the airline liable.[^1] "The AI generated it" was not a defense. It still isn't.

**2. Least privilege, always.** The agent's credentials and action surface should never exceed its minimum requirements for the specific task. An agent that can write code and mutate production state *will* eventually do the wrong thing at speed. In a widely-reported 2025 incident, an AI coding agent deleted a production database during a code freeze because dev and prod were commingled in its permission scope — and then misreported the extent of the damage.[^6] Containment architecture is the only reliable answer — not model quality, not prompting. Architecture.

**3. Human gates on the irreversible.** Anything that is irreversible, involves real money, touches regulated data, or affects a real person's health or legal standing requires a human in the loop before it executes. This isn't a UX choice — it's a governance requirement. Design for fail-closed escalation, not confident improvisation.

In healthcare specifically: business-rule and clinical-guideline adherence isn't a feature you add for compliance. It's the license to operate. At Twin Health, where our agents make dietary recommendations with real metabolic impact, and at Cigna, where prior auth decisions affect patient care timelines, the error budget approaches zero. Design accordingly.

And don't wait for regulatory clarity: the EU AI Act (Regulation (EU) 2024/1689) is already in phased application through 2026, and existing frameworks including GDPR Article 22 on automated decision-making already apply to many agentic deployments.[^8] "We didn't know the AI would do that" is not a legal position.

---

## Team Dynamics: An Agent Is an Org Change Wearing an API

<figure style="text-align:center">
  <img src="/images/blog/agentic-slide-ownership.png" alt="Talk slide: Ownership — Team Dynamics" style="width:100%;border-radius:6px;margin:0 0 1.5em 0;">
</figure>

The failure mode nobody talks about enough: deploying an agent without deciding who owns it.

When a bot states company policy, legal co-owns it. When it acts, engineering owns rollback. When it escalates, support owns the human path. When it causes an incident, product owns the blast-radius decision that let it happen. Ambiguous ownership doesn't make these problems disappear — it just transforms them into finger-pointing after an incident.

My recommendation: assign a named owner to each control surface before you ship. Not a team. A person.

The second, harder conversation is about humans. NEDA wound down its human mental health helpline in anticipation of an AI chatbot, then suspended the chatbot weeks later after it gave harmful dieting advice to vulnerable users.[^4] Klarna replaced human agents before proving steady-state quality, then had to rebuild human capacity.[^5] **Premature human replacement is one of the top agentic failure modes, and it's almost always driven by over-optimism on the automation side.**

The safer approach: use AI to absorb *volume* first. Let humans handle the complex, the emotional, and the edge cases. Measure rework and escalation quality before you make any headcount decisions. Keep the human path not just available, but visible — not buried in a help menu, but a clear, easy, zero-friction option. Trust erodes when users feel trapped.

---

## The Launch Gate: Ship the Harness Before the Autonomy

This is the payoff. Everything above collapses into a single checklist — a gate, not a wish list.

You don't ship until each box is checked:

- [ ] **Classified the job** on the four dials (function, authority, reversibility, context)
- [ ] **One source of truth** for every fact the agent states, with a named human owner
- [ ] **dev ≠ prod**, least-privilege credentials, sandboxed dev environment
- [ ] **Human gate** for every irreversible, regulated, or financial action
- [ ] **Visible escalation** — a fast, easy path to a human that users can find
- [ ] **Red-teamed the real failure modes** — not just "what could go wrong" but "what actually went wrong in comparable systems"
- [ ] **Rollback and kill-switch** — the ability to disable the agent safely, without taking down the broader system, within minutes

Then phase the rollout: Shadow → Cohort → Constrained → Scaled → Optimize. Exit each phase on criteria, not on calendar.

The through-line I keep returning to: across nearly every public incident in agentic AI, teams *delegated authority faster than they built control.* The fix isn't exotic. It's boring and it works: instrument early, constrain tightly, phase slowly, keep humans visible.

---

## Three Things to Take Away

If I had to compress this entire post to a single whiteboard:

**1. Agentic = delegated authority.** Classify it before you build it. Use the four dials. Know your blast radius.

**2. The harness beats the model.** Observability, permissions, source of truth, rollback, human gates. These are the variables that determine your production outcomes far more than which foundation model you're running.

**3. Ship control before autonomy.** Phase the rollout. Keep humans in the loop. Don't replace them before you've proven steady-state quality.

And a final line worth keeping in mind as the regulatory landscape evolves around all of us:

*"The AI did it" is not a defense.*[^1]

---

*I'm happy to go deeper on any of these areas — particularly observability tooling, multi-agent architectures, or the healthcare-specific governance layer. Reach out via [LinkedIn](https://linkedin.com/in/pranav-gujarathi) or find me at upcoming AI engineering events.*

---

## References

[^1]: *Moffatt v. Air Canada*, 2024 BCCRT 149. BC Civil Resolution Tribunal, February 2024. The tribunal held Air Canada liable for its chatbot's misstatement of bereavement fare policy, rejecting the argument that the airline was not responsible for information provided by its AI system.

[^2]: Bakke, C. (2023, December). Chevrolet of Watsonville ChatGPT chatbot incident [Post on X]. *Business Insider* and subsequent coverage, December 2023. A dealership's ChatGPT-powered bot was prompt-injected into agreeing to sell a Chevrolet Tahoe for $1 — because no hard transactional rules existed outside the model's context window.

[^3]: "NYC's AI Chatbot Tells Businesses to Break the Law." *The Markup / THE CITY*, March 2024; additional coverage by *AP News*, 2024. An investigation and subsequent audit found the NYC MyCity business chatbot giving inconsistent and sometimes unlawful guidance, with failures surfaced through external reporting rather than internal monitoring.

[^4]: "An eating-disorders chatbot offered dieting advice." *NPR*, 2023. The National Eating Disorders Association (NEDA) wound down its human helpline and introduced an AI chatbot ("Tessa"), which was then suspended after it was found to provide harmful dieting advice to vulnerable users.

[^5]: Klarna. "Klarna AI assistant does the work of 700 agents." Press release, February 2024. Walk-back and rebalancing toward human customer support subsequently reported by *Bloomberg*, 2025.

[^6]: "AI coding tool wiped a database in 'catastrophic failure.'" *Fortune*, July 2025. Reporting on an incident described by Jason Lemkin / SaaStr in which an AI coding agent deleted a production database during a code freeze due to commingled dev/prod credentials, and then misreported the recovery status.

[^7]: McDonald's ended its AI-powered drive-through order-taking test with IBM. Reported by *CNBC* and *Restaurant Business*, June 2024.

[^8]: Regulation (EU) 2024/1689 of the European Parliament and of the Council (EU AI Act). Entered into force August 2024; phased application through 2026. European Commission. GDPR Article 22 on automated individual decision-making also applies to many agentic deployments involving consequential decisions about individuals.

<script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
<script>mermaid.initialize({startOnLoad:true, theme:'dark'});</script>
