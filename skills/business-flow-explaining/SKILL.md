---
name: business-flow-explaining
description: Explain a piece of code by tracing its real flow and describing it in plain business terms for a reader new to the codebase. Use ONLY when the user wants to understand or explain code, not change it, for example "explain this code", "what does this do", "walk me through", "trace the flow", "who calls this", "what calls this", "how does X work end to end", "explain in business terms", "explain non-technically", "I'm new to this code", "what is this service for". Do not use for writing, fixing, reviewing, or refactoring code (use code-engineering for those).
---

# Business Flow Explaining

Produce a plain-language walkthrough of a piece of code: what it is for (the business purpose), what triggers it (its callers), and what it triggers (what it calls). The reader is someone new to the codebase who does not know the jargon, the architecture, or which system is responsible for what.

This skill is explanation only. Never edit code while using it.

## Core principles

1. **Business first, code second.** Start from the real-world job the code does, then show how the code does it. Example opener: "A visitor signs up for an account and should get a welcome email."
2. **Answer three questions explicitly and repeatedly:** What is this for? What calls it? What does it call?
3. **Translate identifiers into concepts.** Do not say `UserRegistrationService`; say "the module that handles sign-ups." Do not say `EmailProviderClient`; say "the email system." Raw names are allowed only as a light parenthetical reference.
4. **Separate setup time from runtime.** Many modules are admin/control panels; the real work happens later, in a different system, and the module is not involved. Make this explicit so the reader stops assuming the code they are reading is the thing that does the work.
5. **Carry one concrete example** with a made-up actor (a user, a staff member, an administrator) through the entire explanation.
6. **Explain the misleading names** instead of repeating them. If a "notification module" never sends notifications, say so.
7. **Do not dump every branch.** Explain the spine of the flow, then offer to zoom in.

## Step 1: Find the entry point and its boundaries

- Locate the symbol the user pointed at and read it fully.
- Identify its layer from its location and naming (HTTP handler/controller, service/use-case, repository/client, worker/job, etc.).
- Find the outer **entry point**: the thing a human or another system actually triggers (an HTTP route, a message consumer, a scheduled job, an exported function called elsewhere).
- Do not infer the flow from one file. Follow the wiring.

## Step 2: Trace upward and downward

Trace **what calls it**:
- Search the codebase for references to the symbol (use Grep).
- Follow route registration, wiring, and dependency injection to the real entry point.
- Determine whether it is triggered by a user action, another service, a scheduler, or an approval engine.

Trace **what it calls**:
- List every external interaction in its body: other modules, databases, message queues, third-party systems.
- For each dependency, state what it is responsible for in business terms.
- Distinguish "ask a system for information" from "the real side effect" (where data is finally saved or an event is finally sent).

Write the result as a simple chain, for example:
`user action -> API endpoint -> this module -> system A (look up) -> system B (save) -> approval queue`

## Step 3: Build a small glossary

List every domain term and system name that appears and define each in one plain sentence. Call out any term whose name misleads. This is where most confusion is removed.

## Step 4: Write the explanation

Follow this structure. Keep sentences short.

1. **One-line mental model** - the whole thing in a single sentence.
2. **Why it exists** - a concrete scenario with the example actor.
3. **The players** - a small table: system/module | responsibility | everyday analogy.
4. **The setup flow** - numbered steps plus a "who calls whom" arrow diagram.
5. **The runtime flow** - what happens automatically later, and which parts are NOT involved.
6. **The confusing part / gotchas** - clear up misleading names and two-layer or multi-system designs.
7. **One-paragraph recap** - restate the whole flow.
8. **Offer a next zoom-in** - let the reader pick the next area.

Use a text arrow diagram for "who calls whom", a table for the players, and an analogy for anything abstract.

## Step 5: Self-check before answering

- Could a person with zero context follow it top to bottom?
- Did I answer what it does, what calls it, and what it calls?
- Did I avoid raw identifiers, signatures, and type names except as light references?
- Did I separate setup/administration from runtime behavior?
- Did I explain the misleading names instead of echoing them?
- Is there exactly one carried-through example?

If any answer is no, revise before sending.

## Output template

```
One-line: <the whole flow in a sentence>

Why it exists: <scenario with the example actor>

Players:
  <table: system | responsibility | analogy>

Setup flow:
  <actor> -> <entry point> -> <this module> -> <dependency A> -> <dependency B>
  1. ...
  2. ...

Runtime flow: <what happens automatically later; what is NOT involved>

Watch out: <misleading names, two-layer designs, surprising order>

Recap: <one paragraph>
```

## Anti-patterns to avoid

- Reciting function names, parameters, and types instead of explaining behavior.
- Assuming the reader knows the architecture or the domain vocabulary.
- Describing only what the code does internally, without saying who triggers it or why it exists.
- Omitting the runtime/delivery side, so the reader still cannot tell which system does the real work.
- Explaining every branch instead of the main flow; branch detail comes later on request.
- Mixing several examples together.

## Worked example (abbreviated)

For a "user registration" feature:

> One-line: The sign-up module takes a new user's form, checks it, saves the account, and schedules a welcome email.
>
> Why: A visitor fills in a registration form to join a product, and the team wants every new user to receive a welcome message automatically.
>
> Players: web API = the front door; sign-up module = the clerk who processes registrations; billing system = the ledger that records the chosen plan; email system = the mailroom that delivers the welcome note.
>
> Setup: visitor -> web API -> sign-up module -> billing system (reserve a plan) -> database (save the account) -> email system (queue the welcome message) -> moderation queue.
>
> Runtime: later, the email system delivers the welcome message on its own; the sign-up module is not involved.
>
> Watch out: the "email module" only queues the message; a separate worker does the actual sending.
>
> Recap: The sign-up module coordinates the request; the billing, database, and email systems do the storage and delivery.
