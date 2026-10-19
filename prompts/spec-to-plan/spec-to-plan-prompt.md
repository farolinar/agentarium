PART 1: AGENT INSTRUCTIONS

You are going to design an implementation plan from the request form in Part 2. The plan must be followable by two readers: a human who may be seeing this code for the first time, and an agent executing it. Do not write any code yet unless the form explicitly asks for it.

Before you start
If a skill named code-engineering is available, read it and follow it. If it is not available, continue without it.
Read every file in Reference Files. Those are read-only. If you find one needs editing, do not edit it silently: move it to Files to Change, say why, and flag it in Code Design Considerations in the plan.
Read the files in Files to Change too, so the plan reflects what is really there and not what the form assumes.
Asking questions
Ask a question only if the answer would change the chosen solution, the files touched, or the interface. These are blocking questions.
Ask one question at a time, most blocking first. Wait for the answer before asking the next.
Format each question like this:
Question: one sentence.
Why it blocks: what changes depending on the answer.
Options: A / B / C, each with a one-line trade-off.
Recommended: which option and why.
For non-blocking gaps, don't ask. Pick a sensible default and list it under Assumptions in the plan.
If more than one solution is viable, present the options (same format as above) and let the human choose before writing the plan. If only one is viable, say so and move on.
Don't write the plan until all blocking questions are answered.
Output: the plan

Produce the plan with exactly these sections, in this order.

1. Design Paradigm

Read-this-first section. Max ~150 words. Explain the mental model: what the pieces are, how data flows between them, and any term or pattern a newcomer must know to understand the code. Plain language. If the reader reads nothing else, they should still get the idea.

2. Chosen Solution

What was chosen and why it beat the alternatives. End with an Assumptions list (what you decided without asking).

3. Alternatives Considered

Other viable solutions, each with pros, cons, and why it wasn't chosen. Write None viable if that's the case.

4. Execution Steps

Start with a Quick Path: the headlines of all steps below as a numbered list, nothing else. Build it from the step headlines so the two never disagree.

Then the full steps. Every step uses this shape:

#### Step N: <Headline: one line, imperative>
- **Details:** What to do, where (file path, function or section name), and why. This is the
  instruction an agent will follow and a human can follow by hand. One action per step.
  Explain any term a newcomer wouldn't know. Include the expected result of the step.
- **Agent Notes:** (only if there is something special for an agent: tool to use, command to run,
  thing to avoid, how to batch work. Omit this line otherwise.)

Rules for steps:

Order them so the code is never left half-broken between steps where possible. If it must be, say so in Details.
Name real file paths, function names, and commands. No vague "update the handler".
No step should require knowledge that isn't in the Design Paradigm or the step itself.
5. Code Design Considerations

For each file in Files to Change: the fragile parts, invariants that must hold, things that are coupled to it, and conventions to follow. Include any Reference File that was moved here because it needed editing.

6. Failure Modes

Where this can break. For each: what triggers it, how you'd notice, and how the design handles or avoids it. Cover edge cases, bad input, concurrency/ordering, partial failure, and anything easy to miss.

7. Test Plan

Numbered tests to run after all steps are done. Each has: what to run or do, and the expected result. Map them to the Goal and Done Criteria so every criterion is covered by at least one test.

8. Rollback

How to undo the change safely: per step if steps can be undone independently, otherwise as a whole. Include anything that isn't undone by reverting code (data, config, migrations).

9. Flow Diagram

A Mermaid diagram of the main flow of the solution. Use the form's diagram as a starting point if one is given, and fix it if it doesn't match the chosen solution.

PART 2: REQUEST FORM
1. Goal and Done Criteria

Goal (what should be true when this is finished, in one or two sentences): [FILL: ...]

Done criteria (checkable statements; the Test Plan will be mapped to these):

[FILL: ...]
[FILL: ...]
2. Interface

Optional. If no API is involved, leave the contract blank and describe the core process instead.

Request params / input: [FILL: name, type, required?, meaning, example]

Response params / output: [FILL: name, type, meaning, example. Include error shapes if known.]

API contract (optional)

Method: [FILL: GET / POST / ...]
Path: [FILL: /v1/...]
Auth: [FILL: none / bearer token / API key / session ...]
Headers: [FILL: ...]
Status codes / errors: [FILL: ...]

Core process (use this if there is no API contract): [FILL: input → what happens → output, in plain steps]

3. Description

[FILL: What is being built or changed, why it is needed, and who or what uses it. Include current behavior vs. desired behavior if this is a change.]

4. Reference Files (read-only)

Files the agent should read for context but not edit. Not limited to this list: the agent may read others it finds relevant. If one of these turns out to need editing, it moves to Files to Change.

Path	What to look for in it
[FILL: path]	[FILL: e.g. existing pattern for X, type definitions, config]
5. Files to Change

Files expected to be created or edited. Not limited to this list.

Path	Create / Edit	What changes	What's fragile or could break
[FILL: path]	[FILL]	[FILL]	[FILL: e.g. shared by 3 modules, order-sensitive, no tests]
6. Tools and Dependencies

Tools, libraries, services, or commands to use. Not limited to this list.

[FILL: e.g. library + version, CLI, database, MCP tool, test runner]
7. Diagram (optional)
%% [FILL: paste a Mermaid diagram of the current or intended flow, or delete this block]
8. Constraints

Hard limits the solution must respect.

[FILL: e.g. no new dependencies, must stay backward compatible, performance target, security rule, deadline, style guide]
9. Notes

Anything else: past attempts, known gotchas, preferences, links, context that didn't fit above.

[FILL: ...]
