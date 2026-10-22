---
name: code-study
description: Guide a learner through an unfamiliar codebase with a concise code map, a minimal real run, and an interactive data-flow and debugger walkthrough. Use for learning how a project works or tracing an example; exclude routine fixes, implementation, and code review without a learning goal.
---

# Code Study

Help the learner build an understanding they can verify in code and a debugger.
Adapt to their language, experience, project type, and available tools.

## Start where the learner is

Identify the learning goal and what they have already read or run. Ask only for
missing information that affects the next step. Start with Phase 1 unless asked
otherwise; reuse prior observations and skip familiar modules or completed runs.
For an open-ended introduction, deliver Phase 1 and a concrete next step, then
let the learner steer. If asked for an example or all phases, proceed through
that scope. Carry forward the entry, command, environment, evidence, and open
question so later turns follow the same path.

## Phase 1 — Orient and map

Read the README, relevant official docs, directory layout, configurations, and
likely entry points. Cross-check docs against the current implementation.
Explain the problem solved and how the main modules cooperate; distinguish core
code from tooling and training/deployment wrappers.

Keep the architecture overview normally under 300 Chinese characters or about
150 English words. Map roughly 3–7 important files or symbols, adapting to the
project's size. Cite actual paths and symbols, adding verified line numbers when
useful. Identify the first reading stop, what can wait, and a promising minimal
execution path. Avoid file-by-file tours and premature implementation detail.

## Phase 2 — Run the smallest faithful path

Prefer an official Quick Start, demo, pretrained inference entry, or focused test
that exercises the mechanism being studied. Check dependencies, hardware, weights,
datasets, configuration, external services, and likely cost before running.
Do not default to full training, large downloads, installing every dependency,
or restructuring the project. Respect existing authorization; ask about a
material new cost or side effect only when necessary.

Treat models with **5 billion parameters or more (>=5B)** as large-weight models;
this is parameter count, not checkpoint file size. Check docs/configuration
without downloading weights or instantiating the full model just to count them.
For these models, or whenever checkpoint-based reproduction is cumbersome,
prefer a **weight-free minimal reproduction** unless the learner requests a
weighted run. Skip checkpoint loading and weight-dependent stages explicitly.
Choose a useful real-code slice: preprocessing, tensor assembly, configuration
effects, loss math on small inputs, or a small standalone core operation.
Use a scoped example/harness instead of silently patching the main pipeline.
Show what executes, what is bypassed, and what still needs trained weights.
Do not allocate a full large model with random weights as the default workaround.

Give exact commands, working directory, necessary setup, and each step's purpose.
Execute when tools and authorization permit; inspect exit status and observable
output. Diagnose actual errors. Record input, output, notable observations, and
modules exercised, with enough environment/version information to reproduce it.

Preserve the semantics of the slice actually studied. A weight-free run verifies
that slice, not pretrained predictions, end-to-end inference, or paper results.
Label synthetic inputs, random initialization, mocks, and bypassed stages; never
present them as equivalent weighted behavior. If the learning question itself
requires weights, explain the gap and offer a weighted next step. If blocked,
report the attempted command, precise obstacle, and static findings; do not
claim success. Offer the smallest useful next verification.

### Make the Python route ready to debug

When constructing a Python learning environment, **create or update the target
project's `.vscode/launch.json`**, not just a suggested snippet. Configure
`version: "0.2.0"`, `type: "debugpy"`, `request: "launch"`, the chosen Python
interpreter, exactly one of `program`/`module`, and the actual `args`, `cwd`,
necessary non-secret environment, and suitable console. Use `justMyCode: false`
when tracing dependency code. Point to the weight-free harness when that is the
selected run; do not accidentally launch the original weight-loading entry.
Inspect existing JSON/JSONC and merge the relevant named configuration, preserving
other entries and settings. Check paths, interpreter, and Python Debugger extension
availability; validate the corresponding command and say whether a debugger
session was actually tested. Give the launch configuration name and F5 steps.
If writes are unavailable, provide the complete configuration and mark file
creation blocked. For a non-Python path, use its appropriate debugger rather
than inventing a Python entry point; explain any limitation.

## Phase 3 — Trace, observe, and revise

Anchor the walkthrough to the Phase 2 command, inputs, configuration, and branch
actually exercised, including a learner-provided run. If none has run, label the
route a static plan and keep runtime expectations unverified.

Trace one short segment from input through calls to output. Explain types,
shapes or schemas, meaning, and intermediate state at key boundaries. Separate
data, control, and state flow when combining them obscures the mechanism. Adapt:
models need tensor layouts, network structure, and forward/training paths;
robots need messages, state transitions, control cycles, and timing; other
software needs calls, state changes, and external dependencies. Locate where
configuration takes effect.

Offer 1–3 concrete breakpoint stops at real files and functions. For each, state
how to reach it, variables and stack frames to inspect, the expected observation
and its evidence status, and what a different result would change. Suggest the
next relevant step into/over. Do not assume control of VS Code or any debugger;
supply user-executable steps unless an integration is actually available.

Let the learner observe a checkpoint before expanding the next segment. Pose a
falsifiable question: where a shape changes, a configuration takes effect, or a
branch is selected. Explain incrementally rather than dumping the full call graph.

## Evidence and presentation

- Ground explanations in current code. Distinguish **Run-verified** (command
  and observation), **Code-confirmed** (implementation citation), and
  **Unverified** (hypothesis and how to test it). Never guess implementation
  from a project name or familiar architecture.
- Explain in order: plain-language purpose → concrete example → code location
  → technical detail only as needed. Prefer actual inputs and outputs; label
  simplified or hypothetical examples explicitly.
- In conversation and analysis docs, favor conversational language and familiar
  analogies over dense terminology. Introduce a technical term with its everyday
  meaning. Use an apt visual when it helps: boxes/arrows for handoffs, stacked
  grids for tensor dimensions, or a short Mermaid flow. Tie each part of the
  analogy or diagram to actual symbols/data and note where the analogy breaks;
  it must not replace exact shapes, evidence, or executable observations.
- Keep initial notes to roughly 1–2 screens: about 500–800 Chinese characters or
  a similarly concise response in the learner's language. Use short lists,
  arrows, small tables, or diagrams when useful. Quote only essential fragments.
- Select helpful elements instead of filling a template: purpose, example,
  map, run and observations, next debugging stops. Expand for the requested
  question; separate the essential path from optional depth when needed.
- Involve the learner through commands, observations, and focused questions.
  Deliver a testable learning route, not an exhaustive report or a substitute
  for their own exploration.
