# Code Study

English | [Chinese version](README_zh.md)

A reusable Codex Skill: **map the code → run a minimal example → trace and debug**.

Learn open-source code through observations you can verify yourself. Code Study
works with models, robotics, training and inference frameworks, and general
software. It is independent of language, IDE, and hardware, and explains things
in the learner's language.

## Structure and installation

```text
.
├── .agents/skills/code-study -> ../../skills/code-study
├── skills/code-study/
│   ├── SKILL.md                 # Complete instructions; single source of truth
│   └── agents/openai.yaml       # Display metadata and default invocation
├── docs/smoke-test.md            # Example, run results, debugger route, validation
├── LICENSE                      # MIT
├── README.md                    # English guide
├── README_zh.md                 # Chinese guide
└── TASK.md                      # Original requirements
```

The [complete SKILL.md](skills/code-study/SKILL.md) contains the three-phase
workflow and its constraints. It does not load repository documentation or test
records, and requires no helper scripts or runtime dependencies.

According to the [Codex Skills documentation](https://developers.openai.com/codex/skills/),
project skills live in `.agents/skills`, personal skills in `~/.agents/skills`,
and symlinked skill directories are supported. This repository links to the
source directory to avoid maintaining duplicate instructions.

- **Use in this repository:** clone it, start Codex inside the repository, and
  use an invocation below.
- **Install elsewhere:** copy the entire `skills/code-study` directory into
  another project's `.agents/skills/code-study`, or into
  `~/.agents/skills/code-study` for personal use across projects. Inspect any
  existing installation before replacing local customizations.
- Codex discovers changes automatically. Restart the session if the skill does
  not appear; check supported discovery paths when using an older client.

For example, from this repository's root, install into a personal skill directory
where `code-study` does not already exist:

```bash
mkdir -p ~/.agents/skills
test ! -e ~/.agents/skills/code-study && test ! -L ~/.agents/skills/code-study && \
  cp -R skills/code-study ~/.agents/skills/code-study
```

## Invocation and interaction

```text
$code-study Help me understand this repository. Start with a concise code map and a minimal entry point.
$code-study I have already run the official demo. Skip the first two phases and trace its input to output.
$code-study Walk me through the Quick Start, then suggest debugger stops.
```

Implicit selection is enabled by default for learning an unfamiliar project,
understanding a core mechanism, or exploring code through an example. Routine
bug fixes, feature implementation, and code review without a learning goal
should not trigger this skill.

| Phase | Deliverable | Interaction |
| --- | --- | --- |
| 1: Map the code | Brief architecture overview, roughly 3–7 key files or symbols, a reading starting point, and what can wait | Choose a direction after the initial overview |
| 2: Run a minimal example | Dependency and cost checks, commands, Python `launch.json`, inputs, outputs, and run evidence | Prefer weight-free paths for large models; reuse completed runs |
| 3: Trace and debug | A short data/call flow, 1–3 breakpoints, variables, expected observations, and a verifiable question | Observe a checkpoint yourself, then continue; IDE control is not assumed |

An explicit request for all three phases proceeds through them. Skip familiar
modules and completed examples. Initial notes normally fit 1–2 screens: about
500–800 Chinese characters, or a similarly concise response in another language.
The architecture overview normally stays under 300 Chinese characters or about
150 English words. Start with familiar analogies, visuals, and examples, then
locate the code and introduce technical details as needed. Distinguish
run-verified facts, code-confirmed findings, and unverified hypotheses.

## Learning preferences

- **Explore large models without weights first.** Treat models with **at least
  5 billion parameters (≥5B)** as large-weight models. Also prefer weight-free
  reproduction whenever downloading or running checkpoints is cumbersome.
  Think of inspecting a production line's conveyors and sorters first: run real
  preprocessing, tensor assembly, or small core operations, and identify the
  weight-loading and weight-dependent stages being skipped. This verifies the
  selected slice; prediction quality still needs trained weights. Do not
  initialize an entire large model with random weights by default.
- **Prepare the environment for F5.** When constructing a Python learning
  environment, create or merge the target project's `.vscode/launch.json` with
  `debugpy`, the interpreter, entry point, arguments, and working directory.
  Debug the weight-free path when that is the selected run. Preserve existing
  configurations and state whether a debugger session was actually tested.
- **Explain it like a conversation.** Use familiar analogies alongside boxes,
  arrows, grids, or Mermaid diagrams. For example, describe data handoffs as
  parcels moving through stations, while mapping the picture to real functions
  and data and explaining the analogy's limits. Introduce terms as needed;
  shapes, code locations, and evidence remain precise.

## Validation and distribution

The [validation record](docs/smoke-test.md) (in Chinese) uses the local open-source
CPython `json.tool` example. It includes real outputs for valid and invalid
inputs, three breakpoints reached with `pdb`, and a Python launch configuration
checked with `debugpy`. No project clone or dataset download was needed. This
was a manual application of the skill; implicit selection received a
description-level review.

If the bundled `skill-creator` and a Python interpreter with PyYAML are available,
run its structure validator:

```bash
python /path/to/skill-creator/scripts/quick_validate.py skills/code-study
```

Distribute `skills/code-study/` and retain the [MIT license](LICENSE). The example
references local CPython source locations without copying its implementation;
CPython retains its own license.
