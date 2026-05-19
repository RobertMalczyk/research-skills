---
name: architecture-proposal
description: "Generate a self-contained HTML architecture proposal as a review gate between solution selection and implementation scaffolding. Use whenever a non-trivial technical direction has been chosen — typically as the handoff between project-pivot's Step 8 (user decides) and Step 9 (scaffold), but also usable standalone for any major refactor, new component, or architectural decision that warrants explicit design review before code is written. The proposal is a review artifact, not a deliverable: the goal is for the user (and optionally an advisor / reviewer) to audit the architecture, request changes, and explicitly approve before implementation begins. Trigger on phrases like 'design review', 'architecture proposal', 'before we start, let's design this', 'I want to review the design first', 'design document', or as an automatic handoff from project-pivot. Output is a single self-contained HTML file with technical-editorial styling: serious, dense, auditable — NOT a marketing-style deck. Level of detail matches the problem: a few sections for a small change, fuller treatment for a research-software pivot. Do NOT trigger for simple feature additions, obvious one-direction work, or contexts where rapid prototyping is more valuable than formal design review."
---

# Architecture Proposal

A skill that produces a single HTML document for design review *before* implementation. The artifact is a gate, not a deliverable: it exists so the user (and any reviewer) can spot architectural mistakes when they are cheap to fix — at the design stage rather than after the code is written.

This skill is positioned between solution selection and scaffolding. It is the handoff point in `project-pivot`'s workflow (between Step 8 and Step 9), and is also usable standalone for any architectural decision that warrants explicit review.

## What this skill is and is not

**This is.** A structured, auditable design document. Specific architectural commitments. Validation strategy with falsifiable gates. Risks with mitigations. Decision log. An explicit approval gate that records what the user signed off on.

**This is not.** A marketing-style pitch deck. A vague block-diagram artifact. A list of considerations without commitments. The output should look like a serious technical document — editorial in tone, dense but readable, calm in styling — closer to an ACM paper or RFC than a landing page.

The user is a domain expert reviewing this. They will spot vagueness, hand-waving, and unjustified commitments. Don't produce them.

## Roles in this collaboration

The user supplies domain expertise and final approval. Claude does the architectural work: turns the chosen solution into a concrete, falsifiable design, identifies risks, proposes validation strategy, and produces the HTML artifact.

Claude commits to specific architectural choices. "We could do A or B" without a recommendation is a failure mode. Pick one, justify it, list the trade-off, let the user override.

Claude flags genuine uncertainty. Where the design has open questions that the user (not Claude) must answer, list them explicitly in the proposal's Open Questions section rather than guessing.

## Level of detail

The proposal's depth must match the problem. Calibrate before writing:

- **Light** (1-2 sections of meaningful content): a focused refactor, a contained new component, a self-evident extension. Goal/success criteria + approach + one risk section may suffice.
- **Medium** (4-6 sections): a moderate new direction, a multi-component change, anything with non-trivial validation needs.
- **Full** (7-10 sections, the structure described below): a research-software pivot, a fundamental architectural change, anything with significant risk or where reviewers beyond the user will see it.

Default to **medium** for typical project-pivot handoffs. Expand to full only when the prior investigation surfaced significant risk or novel-synthesis content. Compress to light only when the user explicitly indicates the change is small.

A bloated proposal is as bad as a thin one. Calibration is part of the skill.

## Inputs the skill expects

If invoked from `project-pivot`, the pivot register provides everything. If invoked standalone, the skill should gather:

- The chosen solution / direction (statement, building blocks, type).
- Project goal and success criteria (from CLAUDE.md or asked).
- Constraints, must-preserve invariants, user preconditions.
- Prior investigation context, if any (the killed approach, regime, revision conditions).
- Reusable artifacts from prior work, if any.
- Any literature references that ground the proposal.

If critical inputs are missing, ask before generating. A proposal built on guessed inputs is worse than no proposal.

## Procedure

### Step 1 — Gather inputs and calibrate depth

If invoked from `project-pivot`, read the pivot register. Otherwise, ask the user for the chosen solution and project context, or read CLAUDE.md and any referenced docs.

Pick the depth level (light / medium / full) based on:
- Scope of change (contained vs project-wide).
- Risk level (low — well-attested approach, vs high — novel synthesis or untested regime).
- Reviewer audience (just user, vs advisor / committee / external).

State the chosen depth and reasoning to the user. Allow override.

### Step 2 — Draft the content

Generate the proposal content as structured markdown / data first, *before* rendering to HTML. This is important: separating content from presentation lets the user audit the substance without being distracted by styling, and lets the HTML rendering step be mechanical.

The full structure (compress sections out for medium / light):

**1. Header / metadata.** Project name, derived-from (which pivot decision or invocation context), version, status (`draft` | `under_review` | `approved` | `revised`), date.

**2. Goal & success criteria.** The project goal in one paragraph, quoted from source. Then *measurable* success criteria — specific thresholds, on specific data, evaluated with specific metrics. If the prior project had a target (e.g., "<5% rel L2 on held-out data"), restate it here. If the new approach implies a different target, state both and explain.

**3. Approach summary.** One paragraph: what is being built, mechanically. One paragraph: what this is *not* — explicit disambiguation from rejected alternatives or commonly-confused approaches. This second paragraph prevents reviewers from assuming a different mental model.

**4. Architecture detail.** Components, data flow, interfaces. What is learned, what is fixed, what is hand-coded. Where the chosen solution has multiple subcomponents, name each, state its responsibility, and list its inputs and outputs. Use inline SVG diagrams where they aid comprehension — not as decoration. A simple boxes-and-arrows diagram done well beats a beautiful diagram that hides specifics.

The level of architectural detail must be sufficient for the user to spot a wrong commitment. "We will use a neural network" is not enough; "a 3-hidden-layer MLP, x_dot in, scalar w out, with monotonicity enforced via softplus" is.

**5. Validation strategy.** Three sub-sections:

- *Gate tests*: cheap experiments that falsify the approach early. Each gate must have an explicit pass/fail criterion. If gate 1 fails, do not proceed to gate 2.
- *Full evaluation*: the real metric, on the real data, with the real holdout strategy. State the metric, the data split, and the success threshold (linking to section 2's success criteria).
- *Rollback criterion*: what observation would cause us to abandon this approach. Stated explicitly. Without this, projects can drift indefinitely past their original commitment.

**6. Reuse from prior project.** What artifacts (code, data pipelines, validation harnesses, parameter ID scripts, plots) carry over from prior work. Each entry: name, prior path, new path, what may need adaptation. Important even outside `project-pivot` context — refactors and new directions usually inherit infrastructure.

**7. Risks & open questions.** Two distinct lists:

- *Risks*: things that could go wrong with the approach as designed. Each with: probability (low / medium / high), impact (low / medium / high), mitigation, or explicit "accepted, no mitigation."
- *Open questions*: things the user (not Claude) must answer before implementation can finalize. These are not risks; they are decisions deferred to the user.

**8. Decision log.** Every non-obvious design choice, with the rationale. This is what makes the proposal auditable later. Format: decision, alternatives considered, rationale, date. Future Claude (and future you) will need to know *why* a choice was made, not just *what* was chosen.

**9. Implementation plan.** A short, ordered list of the first concrete steps after approval. Not a full project plan — just enough that the user can verify the implementation will start in a sensible place. Typically 3-7 items.

**10. Approval gate.** Explicit prompt: *approve* / *request changes* / *reject*. Records the user's choice and any modifications. Without this section the proposal is not a gate; it's just a document.

### Step 3 — Render to HTML

Convert the structured content to a single self-contained HTML file. Requirements:

**Self-contained.** No external CSS, no external JS, no CDN dependencies. Embed everything. The user must be able to email this file, open it on any machine, archive it in `docs/`, or print it to PDF.

**Technical-editorial styling.** Not corporate, not flashy. Reference points: a well-typeset academic paper, a published RFC, a carefully formatted technical report. Calm, dense, readable. Generous margins, ~70-80 character body measure, clear hierarchy.

**Typography.** Use a serif body font for readability of long-form prose, paired with a sans-serif or geometric font for headers and metadata. Choose distinctive but readable fonts — avoid the most generic defaults. Suitable pairings: EB Garamond / Inter; Crimson Pro / IBM Plex Sans; Lora / Source Sans 3. Monospace for code and identifiers: JetBrains Mono, IBM Plex Mono, or similar. Use system font stacks as fallbacks; do not require web fonts to load for the document to be readable.

**Color.** Restrained, functional. A near-black body text on near-white background (not pure black on pure white — use #1a1a1a on #fafaf7 or similar for readability). One accent color for emphasis, used sparingly: links, status badges, the approval gate. Status states (`draft`, `approved`, etc.) can have distinct colors but kept muted. No gradients, no decorative backgrounds.

**Diagrams.** Inline SVG only. Clear labels, minimal palette (the same accent color as the document body, plus grays). Functional — they exist to communicate architecture, not to decorate.

**Tables.** Used for structured data (success criteria, risks, decision log, reuse log). Subtle borders, generous cell padding, clear column headers. Zebra striping acceptable if very subtle.

**Status badges and gates.** The status (draft / approved) and the approval gate should be visually distinct without being loud. A bordered box with the accent color is enough.

**Print-friendly.** No fixed positioning, no overflow tricks. Page-break-inside: avoid on key sections (success criteria, decision log entries). The user must be able to print to PDF and have it look professional.

**No AI-slop defaults.** No purple gradients. No "Inter on white." No marketing-deck patterns. No emoji in headers. No gratuitous shadows or rounded corners on everything. If the result looks like a startup landing page, the skill failed.

**Interactivity (optional, minimal).** Long sections (decision log, full risk table) may use collapse/expand via `<details>` for navigation. No required JS. The document must be fully readable with JS disabled.

### Step 4 — Save and present

Save the HTML to the project's `docs/` directory (or to `/mnt/user-data/outputs/` if no project path is established). Default filename: `architecture-proposal-v1.html`. If versions exist, increment.

Also save the structured content (the markdown / YAML draft from Step 2) alongside the HTML, as `architecture-proposal-v1.md` or `.yml`. This is the diffable, editable source; the HTML is the rendered artifact. When the user requests changes, edit the source and re-render.

Use `present_files` to surface the HTML to the user.

### Step 5 — Review and iterate

After presenting, ask explicitly:

- Does the architecture match what you intended to commit to?
- Are the success criteria measurable and correct?
- Are the gate tests strong enough to falsify the approach early?
- Are any commitments wrong, vague, or missing?
- Are any risks missing? Are mitigations adequate?
- Are the open questions the right ones for you to decide?

Record responses. Common outcomes:

- **Approve as-is.** Update status to `approved`, record approval in the document, save final version. Hand off to scaffolding (`project-pivot` Step 9 if chained, or to the user otherwise).
- **Request changes.** Edit the markdown / YAML source, re-render HTML, present new version. Iterate.
- **Reject.** Return to whatever produced the input — typically `project-pivot` Step 8 (re-select solution) or an earlier step if the rejection reveals a missed precondition.

### Step 6 — Approval handoff

Once approved, the proposal is now a frozen artifact in `docs/`. The status field reads `approved` with date and reviewer. The implementation can proceed.

If chained from `project-pivot`, return control to that skill at Step 9 (scaffold). Pass back the approved proposal path and any modifications the user made during review.

If standalone, present the approved file to the user and exit.

## Failure modes to resist

- **Vague architectural commitments.** "Use a neural network" is not architecture. Specific layer counts, activation functions, input/output shapes are. If you cannot specify, that itself is an Open Question.
- **Decorative diagrams.** Diagrams should communicate specifics, not impress. A clear boxes-and-arrows diagram beats a beautiful flow that hides the actual data path.
- **Missing rollback criterion.** Every architectural proposal must state what would cause it to be abandoned. Without this, projects drift.
- **Approval theatre.** An approval gate that doesn't record the user's choice and rationale is decoration. Record both, in the document.
- **Over-styling.** The aesthetic target is "serious technical document." Reaching for visual impact pulls the document away from its purpose. Restraint is the design choice.
- **Under-styling.** Conversely, raw unstyled HTML signals "I didn't try." The typography, spacing, and structure must make the document a pleasure to read — that *is* the design choice, not its absence.
- **Treating this as deliverable.** The proposal is for review. It exists to be edited, challenged, and revised. Producing it as if it were a polished final document discourages the user from pushing back. Frame it as draft until approved.
- **Skipping calibration.** Generating the full 10-section template for a small refactor wastes time and dilutes the important sections. Generating the light template for a research pivot misses critical content. Calibrate before drafting.

## Turn-level output contract

Every response while this skill is active must include:

1. `procedure_state` — current step, depth level chosen, status.
2. For Step 2: the structured content (markdown / YAML), not yet HTML, so the user can audit substance first if they prefer.
3. For Step 3: the HTML file path and a short note on styling choices made.
4. For Step 5: the explicit review questions listed above.
5. Prose summary — what was produced, what the user should look at, what's next.

Do not produce the HTML before the structured content is reviewed if the user has indicated they want to audit content first. For typical use, produce both in sequence within the same turn (content → HTML → present).
