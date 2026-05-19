---
name: project-pivot
description: "Structured procedure for setting up a new project (or major new direction within an existing project) after a prior investigation has shown the current approach cannot reach its goal. Use whenever the user has a concluded research investigation — typically the output of hypothesis-verificator — showing that the current architecture or approach is invalid, and needs to determine what to do next: read prior context, ingest findings, generate alternative approaches grounded in literature and tool analysis, optionally propose novel un-published syntheses, elicit user preconditions, present ranked solutions with confidence, get a decision, and scaffold the chosen direction as a new project with reusable artifacts and a CLAUDE.md. Trigger on phrases like 'set up a new project', 'pivot the project', 'change direction', 'start over', 'new approach', 'what should we do instead', 'we need to rethink this', or any framing where a prior investigation has terminated with a known-impossible current path and the user needs a structured replacement. Do NOT trigger for minor changes within an unchanged research question, for greenfield projects with no prior investigation, or for pure scope-decision questions that don't involve setting up artifacts. The user supplies domain knowledge, preconditions, and the final decision; Claude does literature work, generates proposals (including novel ones, with discipline), and produces the scaffold."
---

# Project Pivot

A procedure for converting "the current approach cannot work" into a concrete next project. Picks up where hypothesis-verificator leaves off: findings are known, the killed model class is documented, follow-on questions are queued, and the user needs to commit to a new direction with real artifacts on disk.

This skill is paired with `hypothesis-verificator`. It assumes the prior investigation produced findings the user can hand off. If there is no prior investigation, this skill is the wrong tool — use the user's normal project-setup workflow.

## Roles in this collaboration

Same model as `hypothesis-verificator`. The user knows the domain and supplies preconditions (architectural constraints, IP boundaries, hardware budget, timeline, scope decisions). Claude does the analytical work: reads prior context, surveys literature, analyzes tools, generates proposals, grades them. The user audits, corrects, and decides.

Claude commits to recommendations. "Here are five options, you decide" is a failure mode for a stuck user. Recommend, with reasoning, and let the user override.

Claude pushes back when warranted. If the user's stated preconditions rule out every viable direction, say so directly rather than producing a doomed plan.

## Operating principles

1. **Findings before brainstorm.** Read the prior investigation completely before generating any proposal. Do not skim. The killed model class, the regime characterization, and the explicit revision conditions in the findings constrain what counts as a real alternative.
2. **Literature before novelty.** Search published work before proposing un-published syntheses. Novel proposals are valid only when grounded in identifiable building blocks; they must be marked as novel and graded accordingly.
3. **Preconditions before solutions.** Elicit the user's constraints before producing the ranked solution list. Proposing solutions that violate unstated preconditions wastes both sides' time.
4. **Confidence is multi-axis.** Each solution gets graded on multiple independent axes (theoretical soundness, literature support, engineering feasibility, fit-to-goal, novelty risk). Do not collapse to a single score.
5. **Approval before files.** No files are written to disk until the architecture proposal (Step 8.5) is explicitly approved. This is a hard gate, not a recommendation — Step 9 has a precondition check that refuses to run otherwise. The gate exists because scaffolding is the point of no easy return for a pivot decision.
6. **Scaffold ends the skill.** Once approved and scaffolded, the skill produces real files on disk and exits. It does not deliberate further; that is the new project's job.

## The artifact: pivot register

Maintained in YAML throughout the skill. Like hypothesis-verificator's register, this is the canonical state; prose summarizes for the user.

```yaml
prior_investigation:
  source: <path to hypothesis register, findings document, or equivalent>
  killed_approach: <one-paragraph description of what the prior investigation ruled out>
  regime_characterization: <the operating regime / scope where the kill applies>
  revision_conditions: <what would overturn the prior findings, copied from findings>
  follow_on_problems: <P2, P3, etc. from prior register if present>
  reusable_artifacts: <code, data pipelines, validation harnesses, plots, scripts that survive the pivot>

project_context:
  goal_statement: <the project goal, read from CLAUDE.md / project docs, not paraphrased>
  scope_in: <what the project IS doing>
  scope_out: <what is explicitly excluded — these are hard constraints from prior CLAUDE.md>
  must_preserve: <invariants the prior project committed to — interpretability, SPICE export, etc.>
  context_files_read:
    - path: <file path>
      relevance: <one line on what it contributed>

tool_analysis:
  available_tools_libraries_frameworks:
    - name: <tool/library/framework>
      relevance_to_goal: <how it could contribute>
      maturity: <production | research | experimental | abandoned>
      compatibility_with_must_preserve: <yes | partial | no — with reason>
  prior_codebase_reusable:
    - artifact: <name>
      what_it_does: <one line>
      reuse_value: <high | medium | low>

literature_overview:
  search_queries_run:
    - <query string>
  key_works_found:
    - citation: <author, title, year, venue>
      relevance: <what it offers>
      applicability_to_regime: <does it work in the killed-approach's regime — yes | adapted | no>
  literature_summary: <2-3 paragraphs of synthesis, not a list>
  gaps_identified: <where published literature does not cover the project's needs>

solution_candidates:
  - id: S1
    statement: <one paragraph: what the solution is, mechanically>
    type: <literature_attested | adapted_from_literature | novel_synthesis>
    building_blocks:
      - <each block, with citation if from literature>
    novel_elements: <what is new, if anything — required when type is novel_synthesis>
    why_not_published: <if novel: best-faith reason this hasn't been done — niche, recent, requires combination of skills, etc. Do NOT claim originality without checking.>
    addresses_goal_via: <mechanism by which this solution reaches the project goal>
    respects_must_preserve: <does this preserve interpretability, SPICE export, etc.>
    fits_preconditions: <satisfies user-stated preconditions>
    confidence:
      theoretical_soundness: <low | medium | high — with one-line reason>
      literature_support: <none | indirect | direct>
      engineering_feasibility: <low | medium | high>
      fit_to_goal: <partial | full>
      novelty_risk: <none | low | medium | high — higher for novel_synthesis>
    estimated_effort: <hours | days | weeks | months>
    open_questions: <what would need to be resolved to start>
    claude_recommendation: <should the user pick this? — with reasoning>

user_preconditions:
  architectural: <e.g., must integrate with existing SPICE export>
  design: <e.g., interpretability required, no black-box>
  resource: <hardware, compute, time budget>
  ip_or_scope: <patents, publishing constraints, must / must-not use X>
  other: <anything else the user supplies>

decision:
  chosen_solution: <id, or null if not yet decided>
  rationale: <user's reasoning, recorded>
  modifications: <any changes the user made to the chosen solution>

scaffold:
  architecture_proposal:
    status: <draft | under_review | approved | approved_with_modifications | rejected | skipped_by_user>
    proposal_path: <path to the HTML proposal, or null if skipped>
    approval_date: <date | null>
    approval_conditions: <any conditions or modifications applied at approval | null>
    skip_reason: <user's stated reason if status is skipped_by_user | null>
  project_path: <absolute path where files will be created>
  is_new_repo: <true | false — separate repo or directory in existing repo>
  files_to_create:
    - path: <relative to project_path>
      purpose: <one line>
      content_source: <generated | copied from prior project | template>
  reusable_artifacts_copied:
    - from: <prior path>
      to: <new path>
      reason: <why it's still relevant>

procedure_state:
  current_step: <1-11>
  pending_user_check: <what the user is being asked to verify or decide>
  status: <in_progress | awaiting_decision | scaffolding | complete>
```

## Procedure

### Step 1 — Ingest prior project context

Read the project's grounding documents. The user may supply paths; if not, search by convention.

Default search locations and order:

1. `CLAUDE.md` at project root (project-level rules and scope).
2. `~/.claude/CLAUDE.md` (personal preferences — read if accessible).
3. Project `MEMORY.md` and its referenced files if the Auto Memory layer is set up (the first 200 lines of the index are loaded into context by Claude Code automatically; this skill reads beyond that, including the per-category files in `memory/` if relevant).
4. `docs/` folder, `README.md`, any `*.md` files referenced from CLAUDE.md.
5. Generated reports, especially any prior hypothesis register or findings document.
6. If a memsearch-style plugin or `/memory` command is available, use it to retrieve relevant historical context — but treat retrieved memory as a hint to verify, not as fact (per Claude Code's own memory-handling discipline).

Populate `project_context` and `prior_investigation`. Do not paraphrase the goal statement — quote it. If CLAUDE.md scope says certain approaches are out (e.g., "no Bertotti terms", "no black-box models"), record these verbatim in `scope_out`.

If context is contradictory (e.g., prior findings show CLAUDE.md scope rules out the only viable direction), flag this for the user immediately. Do not proceed past Step 1 with an unresolved contradiction.

**Checkpoint with user.** Show what was read, the goal statement as quoted, and any contradictions found. Ask: is the goal statement still correct, or has the prior investigation made you want to revise it? Are any of the scope_out items now negotiable?

### Step 2 — Ingest prior investigation findings

Read the hypothesis-verificator output (or equivalent prior investigation). Populate `prior_investigation` fully. Capture especially:

- The killed approach, in its full scope-bounded form (e.g., "scalar-shape J-A-C cannot represent N87 minor loops at 50-447 kHz, 11-275 mT" — not "J-A-C doesn't work").
- The revision conditions — these are *both* constraints (the prior finding might be overturned by X) *and* candidate directions (X might be worth trying).
- Follow-on problems — these are pre-existing candidate directions.
- Reusable artifacts — anything from the prior project that survives the pivot (data loaders, plotting, validation harnesses, parameter identification scripts, regime-characterization code).

The follow-on problems and revision conditions from the prior investigation are *not* automatically the new direction. They are starting points. Step 3 may surface better options.

**Checkpoint with user.** Show the killed approach, the explicit revision conditions, the follow-on problems, and the reusable artifacts. Ask: have I captured the kill correctly? Are there reusable artifacts you'd want preserved that I missed?

### Step 3 — Analyze available tools and prior codebase

What can be built with. Two sub-passes:

**Tools / libraries / frameworks.** Survey what's available for the problem domain that wasn't used in the prior project, or wasn't used fully. For each candidate, record: relevance, maturity, compatibility with `must_preserve` invariants. Reject candidates that violate hard constraints up front.

This is not an inventory of installed packages. It is a goal-directed survey of tools that *could enable* a different approach. Examples in research-software contexts: alternative solvers, alternative model classes, alternative measurement / inversion methods, alternative datasets, alternative validation frameworks.

**Prior codebase reuse.** For each significant artifact in the prior project, classify reuse value: high (carries directly), medium (needs adaptation), low (replaced or obsolete). The reusable_artifacts list from Step 2 feeds this.

Populate `tool_analysis`.

### Step 4 — Literature overview

Real literature work, not a vibes-based summary. Run actual searches; read actual sources; capture citations.

Search strategy:

1. Start with the killed approach's regime characterization. "What works in this regime?" — search for that.
2. Search the explicit revision conditions from prior findings (they often name specific alternatives).
3. Search adjacent fields and methods. The prior project's failure mode often has analogs in nearby domains.
4. Search recent work specifically — the last 2-3 years — because methodology often moves faster than textbooks reflect.

For each key work found, record citation, relevance, and whether it applies to the project's specific regime or needs adaptation.

Produce a 2-3 paragraph synthesis. Identify gaps where the literature does *not* cover the project's needs — these are where Step 5's novel proposals will be justified.

Populate `literature_overview`.

**Checkpoint with user.** Show the synthesis and the gaps. Ask: are there bodies of work I missed? Do you agree with the gap analysis?

### Step 5 — Generate solution candidates, including novel ones

Generate at least three candidates, no more than seven. Mix types:

- **Literature-attested**: a published approach that addresses the goal in the project's regime. Lowest novelty risk; cite directly.
- **Adapted from literature**: a published approach for a different regime or problem, adapted with explicit modifications. Medium novelty risk; cite the source and document the adaptation.
- **Novel synthesis**: a combination of building blocks not previously assembled this way, or a new approach justified by a literature gap. Highest novelty risk; allowed but disciplined (see below).

**Discipline for novel proposals.** A novel proposal is not a hallucination *only if* Claude can:

1. Name every building block (with citation or other identifiable source).
2. State explicitly what is novel — the assembly, the application, the combination — and what is borrowed.
3. Give a best-faith answer to "why hasn't anyone done this?" — niche application, recent enabling tool, requires cross-field expertise, just not tried yet. "I don't know" is acceptable but the question must be asked. If the answer is "I assumed nobody has," check by searching before claiming novelty.
4. Grade novelty_risk honestly — novel proposals start at medium novelty_risk by default and require positive evidence to drop to low.

A novel proposal Claude cannot defend on points 1-3 is dropped, not graded.

For each candidate, populate the full `solution_candidates` entry — including the `claude_recommendation` field with a clear position.

### Step 6 — Elicit user preconditions

Now (not earlier) ask the user about constraints that bound the solution space. Earlier asking produces vague answers because the user doesn't yet know what's being chosen between; later asking is too late.

Categories to ask about explicitly:

- **Architectural**: integration requirements, must-use frameworks, system boundaries.
- **Design**: interpretability, transparency, debuggability, regulatory / publication standards.
- **Resource**: hardware available, compute budget, time available, team size.
- **IP / scope**: patents you must avoid, things you want to publish, things you cannot use.
- **Other**: anything the user wants to surface.

Use the `ask_user_input_v0` tool if the choices are crisp (e.g., "interpretability required: yes / no / partial"). Otherwise ask in prose. Record every answer verbatim in `user_preconditions`.

After answers come in: re-grade each solution candidate against the stated preconditions. Solutions that violate hard preconditions are dropped or flagged. Re-rank the survivors.

**Checkpoint with user.** Show the updated ranked list with each solution's fit against the new preconditions. Note any that were dropped or downgraded and why.

### Step 7 — Present ranked solutions with confidence

Produce the final ranked list. Ranking axes, in order:

1. **Fit to goal under stated preconditions.** Solutions that fully address the goal outrank partials.
2. **Theoretical soundness × literature support.** Higher when the approach is both sound and attested.
3. **Engineering feasibility within stated resources.**
4. **Novelty risk inverted.** Lower novelty risk preferred when other axes tie — but a higher-novelty solution that uniquely fits the goal can still rank first.

For each ranked solution, present:
- One-paragraph statement.
- Confidence across the five axes (don't collapse).
- Estimated effort.
- Open questions before starting.
- Claude's recommendation with reasoning.

Make one explicit overall recommendation. The user is stuck; presenting equally-weighted options is no help. State which one Claude would pick, why, and what would change Claude's mind.

### Step 8 — User decision

Show the ranked list with the recommendation. Use `ask_user_input_v0` if there's a clean choice between options. Allow free-text response — the user may modify a solution before accepting, combine two, or reject all.

If the user rejects all: do not loop indefinitely. Ask what's missing. If their answer reveals an unstated precondition, return to Step 6 with the new constraint and re-generate from Step 5 if needed. If the answer reveals a goal change, return to Step 1.

If the user picks one (possibly with modifications): record `decision`, including any modifications, and proceed.

### Step 8.5 — Architecture proposal review gate (mandatory)

**This step is a hard gate. No files are created in Step 9 without an explicitly approved proposal recorded in the register.** This is not advisory.

Before scaffolding files, hand off to the `architecture-proposal` skill. The skill generates an HTML design document — calibrated to the problem's scope (light / medium / full) — that the user reviews and explicitly approves, revises, or rejects.

Pass the full pivot register as input: chosen solution, project context, must-preserve invariants, user preconditions, prior investigation findings, reusable artifacts, and any literature references from Step 4.

Three possible outcomes, each must update the register:

- **Approved**: set `scaffold.architecture_proposal.status: approved`, record the proposal file path, the approval date, and any conditions of approval. The proposal becomes a frozen artifact in the new project's `docs/`. Only this state unlocks Step 9.
- **Approved with modifications**: set `scaffold.architecture_proposal.status: approved_with_modifications`, record the modifications verbatim into the pivot register (`decision.modifications` and the relevant `solution_candidates` entry), and either (a) re-render the proposal as v2 incorporating the modifications and re-confirm, or (b) record the modifications as conditions to be applied during scaffolding. Default: re-render and re-confirm if modifications are non-trivial.
- **Rejected**: set `scaffold.architecture_proposal.status: rejected`. Return to Step 8 (re-select solution) or Step 6 (re-elicit preconditions) depending on what the rejection revealed. Do not proceed to scaffolding. Do not loop indefinitely — if rejection is repeated, surface the underlying disagreement explicitly to the user.

**Skipping this step.** The default is: do not skip. The skip path is narrow and requires *all three* conditions:

1. The depth calibration is unambiguously `light` (a contained refactor, a self-evident extension — not a research-software pivot).
2. The user explicitly states they want to skip the design review, using language equivalent to "skip the architecture proposal" — not just "let's move fast" or "I trust you."
3. Claude confirms the skip with a single clear question that names the consequence: "To confirm — proceed to file scaffolding without producing a reviewable design document? This means no auditable record of what we committed to."

If the user confirms, set `scaffold.architecture_proposal.status: skipped_by_user` with the date and the user's stated reason recorded verbatim. This state also unlocks Step 9 but should be rare.

For research-software pivots, novel-synthesis solutions, or anything with non-trivial risk, refuse to skip even if the user requests it — explain that the design review exists precisely to protect them from sunk-cost decisions made before architectural commitments are visible. The user can still override after that explanation, but Claude does not skip silently.

### Step 9 — Set up project folder

**Precondition (must be checked first).** Before any folder is created or any file is written, verify the register contains a valid approval gate state:

```yaml
scaffold:
  architecture_proposal:
    status: <approved | approved_with_modifications | skipped_by_user>
    proposal_path: <path to the HTML proposal, or null if skipped>
    approval_date: <date>
    approval_conditions: <any conditions or modifications, or null>
```

If `scaffold.architecture_proposal.status` is anything else — `draft`, `under_review`, `rejected`, `null`, or absent — Step 9 must not proceed. Instead, return to Step 8.5 and complete the gate. State this to the user explicitly: "I can't start scaffolding yet because the architecture proposal hasn't been approved. Returning to the review gate."

This is non-negotiable. Files-on-disk are the point of no easy return for a pivot decision; the gate exists so the user has explicitly committed to what is being built before commitment becomes expensive to reverse.

Once the precondition is satisfied: confirm the project path with the user. Default: a sibling folder to the prior project, named meaningfully (e.g., the parent of the prior project + a new subfolder). Confirm whether this is a new git repo or a directory inside an existing one.

Create the folder structure. Suggested baseline:

```
<project_path>/
├── CLAUDE.md
├── README.md
├── docs/
│   └── pivot_context.md       # prior investigation summary, why this direction
├── src/                       # source code
├── tests/
├── data/                      # if applicable
├── notebooks/                 # if applicable
├── scripts/
└── .gitignore
```

Adjust based on the domain — research-software projects often need different layouts than application code.

### Step 10 — Generate CLAUDE.md and reusable artifacts

**CLAUDE.md.** Generate a project-level CLAUDE.md following the conventions Claude Code expects. Required sections:

- **Goal**: one paragraph, precise. Quoted from `project_context.goal_statement` if unchanged from prior, or revised per Step 1.
- **Scope**: what is in, what is out. Include any hard constraints from prior CLAUDE.md that still apply.
- **Approach**: one paragraph on the chosen solution and why this approach was selected, with reference to the prior investigation.
- **Constraints**: must-preserve invariants, user preconditions.
- **Prior investigation summary**: a short pointer to `docs/pivot_context.md` with the kill, the regime, and the revision conditions.
- **Architecture proposal**: a pointer to `docs/architecture-proposal-v1.html` (the approved design document from Step 8.5). Mark it as the authoritative source for "what was committed to."
- **Reusable artifacts**: what was carried over from the prior project and what is new.
- **Conventions**: code style, test approach, anything else worth recording at the project level.

Keep CLAUDE.md short. The Milvus / Claude Code memory analysis correctly notes: shorter files get followed more reliably. Move detail to `docs/` and reference it.

**Pivot context document** (`docs/pivot_context.md`). Capture:
- The prior killed approach in full scope-bounded form.
- The regime characterization.
- The revision conditions from the prior findings (so future Claude knows when to question this project's foundation).
- The literature gap and rationale for the chosen direction.
- The novel elements, if any, with explicit "this is novel" tagging.

**Copy reusable artifacts.** From `scaffold.reusable_artifacts_copied`, copy each from prior project to new project, adapting paths in imports if needed. Record what was copied, from where, and why in CLAUDE.md.

**Copy the approved architecture proposal.** Move the HTML produced in Step 8.5 (and its markdown / YAML source) into `docs/` of the new project. The approved proposal is now the canonical record of what was committed to at project start.

**Generate any other code stubs** the chosen solution requires for a clean start — e.g., a basic data loader if reusing the data, a config file template, an empty validation harness that matches the prior project's structure.

### Step 11 — Bootstrap git and project handoff

The generated CLAUDE.md must include an instruction for future Claude sessions to check git status and prompt the user about repository setup if not present. Recommended language to include in CLAUDE.md:

```
## Repository setup
This project's CLAUDE.md was generated by the project-pivot skill. At the
start of the next session, check whether `.git/` exists in the project
root. If not, ask the user whether they want to:
  (a) initialize a new git repo here (`git init`),
  (b) add this folder to an existing parent repo,
  (c) skip version control for now.
Do not initialize git silently. The user should choose.
```

This is intentional: the skill scaffolds files but does not commit to a versioning decision on the user's behalf.

Optionally, if the user has indicated they want git set up immediately, the skill can prompt and run `git init` plus an initial commit during this step. But the default is to defer to the next session.

Present the finished scaffold to the user with `present_files` showing CLAUDE.md and the pivot context document as the headliners. Summarize what was created, where, and what to do next (typically: open the new project, start the next session there, check git).

`procedure_state.status: complete`. Exit.

## Failure modes to resist

- **Skipping the literature step.** Generating proposals from priors only is fast and produces work that has either already been done or doesn't exist in serious form. The literature step is non-optional.
- **Claiming novelty without checking.** Marking a proposal as `novel_synthesis` when in fact it's been published is worse than missing the literature entirely. Always run a search before claiming novelty.
- **Hallucinating citations.** If a citation cannot be verified, do not include it. "I believe there is work by X on Y" is acceptable; "Smith 2023 showed Z" with a fabricated citation is not.
- **Vague preconditions question.** Asking "any preconditions?" gets "no" and misses everything. Ask category by category.
- **Recommending neutrally.** Producing five solutions with no recommendation when the user is stuck is a failure mode equal to refusing to take a position.
- **Setting up files before the gate.** The gate is **Step 8.5 (architecture proposal approval)**, not Step 8 (solution decision). Solution selection alone does not authorize scaffolding. No files get created until `scaffold.architecture_proposal.status` is approved (or rare explicit skip).
- **Skipping the gate under pressure.** A user impatient to see code might ask to skip the architecture proposal. For light-depth changes with explicit user confirmation this is acceptable. For research-software pivots or novel-synthesis solutions, refuse to skip even on request — explain why, then let the user override knowingly. Silent skipping is never acceptable.
- **Over-scoping CLAUDE.md.** Generated CLAUDE.md files tend to bloat with rules the user never asked for. Keep it short; put detail in `docs/`.
- **Silent git init.** Initializing a repo without explicit user choice surprises people and creates merge friction later.

## Turn-level output contract

Every response while this skill is active must include:

1. `procedure_state` — current step, status, pending user check.
2. Register update — full on first turn, diff after that, YAML.
3. Prose summary — short, domain-expert level, what's alive, what was learned.
4. The substantive content of the turn — the analysis, proposals, question, or scaffold action.

When Step 10 runs and files are created, use `present_files` to surface the headliner files to the user. When the skill completes, the final turn should make the handoff to the new project obvious.
