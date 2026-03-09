# System Design general tips

## Table of Content

- [System Design general tips](#system-design-general-tips)
  - [Table of Content](#table-of-content)
  - [Questions to ask at the beginning of the project](#questions-to-ask-at-the-beginning-of-the-project)
  - [Document decisions](#document-decisions)
  - [Must Do steps](#must-do-steps)
  - [Choosing the right architecture](#choosing-the-right-architecture)

## Questions to ask at the beginning of the project

Do not try to satisfy too many constraints at once. Start with the 5 main constraints and build your system from here. Try to understand what is actually true in what you are being told. If without the system, the users do a task in 2 hours, you bring value if the system does the task in 1h59. No need for it to be immediate.

- What would happen if this system went down for an hour?
- What's the most sensitive data it handles?
- How many users do you expect in 12 months?
- Which part of the system, if it broke or slowed down, would cause the most damage?
- Is there anything in this system with legal, compliance, or security implications?

## Document decisions

When making architectural decisions, document why you are doing things. When constraints disappear or you have a change in your process, you want to be able to detect changes as early as possible. You also want to check if your solution stills holds up.

Don't do twice the same work.

## Must Do steps

**Step 1**: Capture the functional requirements.

When to do it: Before you write a single line of code or draw a single diagram.

Primary goal: Understand what the finished application needs to do.

Secondary goal: Identify the potential complex features early in the process.

What you need to do:

- List every feature the system needs at launch. You don’t need to prioritise it yet. Just brain dump it in written form.
- Identify which features are truly critical vs. nice-to-have.
- Flag any features that feel complex, risky, or unclear.
- Identify the users.

**Step 2**: Extract the non-functional requirements (NFRs)

When to do it: Immediately after step 1. This is what most developers skip.

Primary goal: Translate what the business says into the architectural constraints the system must satisfy.

Secondary goal: Expose the trade-offs you'll have to make before they surprise you later.

What you need to do:

- Read each requirement and ask: "What happens when this breaks? What happens under 10x load? What are the security demands?"
- Map business language to architectural characteristics. "We can't afford downtime" → Availability. "We're handling payments" → Compliance + Security. "We're expecting a spike at launch" → Scalability.
- Prioritise ruthlessly. You cannot optimise for everything. Pick the 3-5 that are the most critical. Accept that strengthening one will weaken another.
- Make educated guesses based on who your users are.

(Business stakeholders will never hand you this list. The quality of the questions you ask here matters more than any framework you choose.)

**Step 2.5**: Estimate size.

Do a few calculus to evaluate the scales of your components. It can help also see the costs of each feature.

**Step 3**: Apply your real-world constraints.

When to do it: Once your NFRs are prioritised.

Primary goal: Narrow the set of potential architectural options to what's actually achievable given your team, timeline, and budget.

Secondary goal: Avoid building the theoretically correct architecture for the wrong team.

What you need to do:

- Be honest about team size and experience. 2 juniors building microservices is a recipe for disaster.
- Are there any other constraints? For example, time to deliver and cost are the ones I face the most of the time.

**Step 4**: Match your requirements to an architectural style.

When to do it: Only after steps 1-3 are complete.

Primary goal: Choose an architecture based on analysis, not trends.

Secondary goal: Build the ability to justify your choice to any stakeholder who asks.

What you need to do:

- Evaluate each major architectural style/pattern against your top NFRs: clean architecture, vertical slice architecture, modular monolith, microservices.
- Which styles support NFRs? Which ones are in conflict?
- Identify the one or two styles that score best across your most critical requirements.

(The goal isn't a perfect fit, but what makes most sense now. Remember: architecture isn’t a one-time task. You will evolve it over time.)​

**Step 5**: Document the decision.

When to do it: Before implementation begins.

Primary goal: Create a permanent record of what you decided and why, so the context isn't lost when people leave, requirements change, or the team grows.

Secondary goal: Build the habit of treating architectural decisions as crucial things to permanently store in a written form. Instead of something that vanishes in Slack chat history.

What you need to do:

- Write an Architecture Decision Record (ADR) capturing: the context, the decision, the alternatives considered, and the trade-offs accepted.
- Store it alongside the code, where developers will actually find it.
- Share it with your team before they start building.
- Talk about drawbacks, bottlenecks, tradeoffs.
- Mention a bit of asynchronous, caching...

To repeat, the goal of this process isn't to find the perfect architecture.

## Choosing the right architecture

There are three main choices when it comes to architectures of code:

- **Monolith**: Everything runs in a single deployable unit. One codebase, one deployment, one process. A well-structured monolith is fast to build, easy to debug, and perfectly suited to most early-stage and medium-sized systems. It gets a bad reputation because of undisciplined monoliths, not the style itself. The 2 most popular monolith styles are Clean Architecture and the Vertical Slice Architecture, although you can combine them as well.
- **Modular Monolith**: Still one deployable unit, but the codebase is split into well-defined, independently organised modules. Each module represents a part of a business domain. The modules enforce boundaries without the operational complexity of separate services. The sweet spot for bigger teams that need internal structure but aren't ready (or don't need) independent deployments.
- **Microservices**: Each domain runs as its own independently deployable service, communicating over the network. Maximum flexibility, scalability, and team autonomy. The cost? You add the operational complexity, distributed systems challenges, and infrastructure overhead.

Evaluate tradeoffs:

- Simplicity + speed to market is the top priority - Monolith wins almost every time at this stage.
- Complex domain, growing team, need clean internal boundaries - Modular Monolith is the natural fit.
- Independent scaling per domain, multiple autonomous teams, global traffic - Microservices become genuinely justified.
- Small internal tool - Vertical Slice Architecture provides you with the ability to implement quickly and in an organized way.

CAREFUL: some choices are complicated to implement with reduced staff and budget (microservices can be tricky to put in place for example). You can have traps where you build the right architecture too early and end up with something that does not fit with your need or takes too long & the project is canceled.
