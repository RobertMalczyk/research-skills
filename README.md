# Research Skills for Claude Code

Four composable skills for serious research work — investigating hard problems, pivoting to new directions when an approach fails, and producing auditable design documents before committing to code.

These skills are intended for **PhD-and-beyond level research** where the user knows the domain but is genuinely stuck, and Claude does substantive analytical work that the user audits and corrects. They are deliberately structured (YAML registers, explicit verdicts, mandatory checkpoints) because conversational AI is otherwise too eager to either capitulate or charge ahead.

## The four skills

| Skill | Question it answers | Output |
|---|---|---|
| `layered-research` | What novel hypotheses are worth pursuing? | Layered hypothesis register + position paper + experiments list |
| `hypothesis-verificator` | Why isn't this working? / Which hypothesis holds up? | Hypothesis register + findings |
| `project-pivot` | What should we do instead? | New project scaffolded on disk |
| `architecture-proposal` | How should we build it? | Self-contained HTML design document |

### `layered-research`
Generates novel hypotheses on hard research problems through iterated layers of literature-grounded synthesis, with parallel analogy search and disciplined provenance tracking. Layer 1 hypotheses cluster near the literature (most attested); deeper layers produce genuinely novel candidates by synthesizing patterns from prior layers. Novelty is graded by literature absence at each layer, not by depth or configuration — the groundedness dial controls search breadth, not grade inflation. "No novel hypothesis found" is a legitimate result.

Use when: existing literature does not contain the answer, obvious directions are exhausted, and you need disciplined exploration of less-obvious ones.

### `hypothesis-verificator`
Rigorous procedure for managing competing hypotheses in research investigations. Each hypothesis is taken to an explicit verdict (`confirmed`, `falsified`, `inconclusive`, `subsumed`, `superseded`). The skill enforces Popperian discipline (falsifiability criteria stated *before* evidence) and a holistic re-synthesis step where compound findings live.

Use when: you have a hard problem, multiple possible explanations, and need to know which (if any) is right. Frequently invoked downstream from `layered-research` to validate novel hypotheses.

### `project-pivot`
Procedure for converting a concluded investigation ("the current approach cannot work") into a concrete new project. Reads prior context, generates ranked solution candidates including disciplined novel syntheses, elicits user preconditions, gets a decision, and scaffolds files.

Use when: a `hypothesis-verificator` investigation (or equivalent) has terminated with a known-impossible current path, and you need to commit to a new direction.

### `architecture-proposal`
Generates a self-contained HTML design document as a review gate *before* code is written. Specific architectural commitments, falsifiable validation strategy (gate tests with pass/fail criteria), risks with mitigations, decision log, explicit approval gate. Calibrated to problem scope (light / medium / full).

Use when: you have a chosen direction and want a design review before implementation. Either invoked automatically by `project-pivot` (as Step 8.5) or used standalone for any major refactor.

## How they compose

The canonical cycle:

```
layered-research
       │
       │ ranked hypotheses, primary candidate,
       │ position paper, experiments list
       ▼
hypothesis-verificator
       │
       │ verdicts: confirmed / falsified / inconclusive
       │ findings on what works and what doesn't
       ▼
   ┌───────────────┴───────────────┐
   │                               │
[answer found]              [approach killed]
   │                               │
   │                               ▼
   │                       project-pivot
   │                               │
   │                               │ (Step 8.5 invokes architecture-proposal)
   │                               │
   │                               │ approved HTML proposal in docs/
   │                               ▼
   │                       scaffolded project on disk
   │                               │
   └───────────────┬───────────────┘
                   │
                   ▼
            (back to research,
             new hypothesis cycle,
             or project complete)
```

Four skills, one cycle. `project-pivot` will not write any files until `architecture-proposal` has produced an explicitly approved design document — enforced at three levels in the skill: operating principle, mandatory step, and Step 9 precondition check. `layered-research` produces deliverables that feed directly into `hypothesis-verificator` for validation.

## Standalone use

Each skill is also usable on its own:

- **`layered-research` alone** — when you want disciplined novel-hypothesis generation as a one-off, not necessarily feeding into validation. The position paper and ranked list are useful standalone artifacts.
- **`hypothesis-verificator` alone** — when you want structured investigation without any plan to pivot. The findings become input to whatever you do next.
- **`project-pivot` alone** — rarely useful; it expects prior investigation findings. Possible if you have those findings in another form (e.g., a prior project's documented dead-end without a formal hypothesis register).
- **`architecture-proposal` alone** — common. Any major refactor, new component, or architectural decision can benefit from an HTML review document. No pivot required.

## Repository layout

```
.
├── README.md
├── LICENSE
├── skills/
│   ├── layered-research/
│   │   └── SKILL.md
│   ├── hypothesis-verificator/
│   │   └── SKILL.md
│   ├── project-pivot/
│   │   └── SKILL.md
│   └── architecture-proposal/
│       └── SKILL.md
```

The folder name (e.g. `layered-research/`) is the skill identifier; `SKILL.md` is the fixed filename Claude Code looks for inside each folder.

## Installing

For Claude Code: copy a skill folder into your project's skills directory (or into `~/.claude/skills/` for personal use). The skill becomes available the next time Claude reads its skill index.

For other tooling: read the `SKILL.md` files directly. They are written as standalone procedural documents and remain useful as methodology guides outside any specific tool.

## Design principles common to all four skills

These show up repeatedly because they're what make the skills work in practice:

- **Falsifiability and verdicts before action.** A hypothesis without a kill criterion is a suspicion. An architectural commitment without a rollback criterion is wishful thinking.
- **Holistic re-synthesis.** After per-item work, always re-examine the set. Compound findings, shared upstream causes, and theory-level reformulations live in the pattern across items.
- **Structured register as system of record.** Each skill maintains a YAML register that survives across turns. Prose summarizes; the register decides.
- **Multi-axis confidence.** Single scores hide trade-offs. Each skill grades along multiple independent axes.
- **Claude commits, user audits.** A stuck user needs Claude's analytical work, not a list of neutral considerations. Claude takes positions; the user corrects.
- **Hard gates at high-cost transitions.** Especially: no file scaffolding without approved architecture; no termination without genuine verdicts.

## License

See `LICENSE`. Apache 2.0.

## Contributing

These skills are deliberately opinionated about research methodology. If you find a procedural gap or a failure mode they don't resist, open an issue with a concrete example. Generic suggestions ("make it more flexible") are harder to act on than specific ones ("on a real investigation I had X happen and the skill produced Y, when Z would have been correct").
