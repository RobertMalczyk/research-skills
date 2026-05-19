---
name: layered-research
description: "Structured procedure for generating novel hypotheses on hard research problems through iterated layers of literature-grounded synthesis, with parallel analogy search and disciplined provenance tracking. Use whenever the user is working on a research question that requires producing hypotheses not present in the existing literature — original research seeding, theoretical extensions, anomaly explanations that resist standard accounts, or any situation where the user wants to systematically search for novel directions while remaining honest about what is genuinely new vs already published. Trigger on phrases like 'novel hypothesis', 'what hasn't been tried', 'beyond the literature', 'I need new ideas', 'help me think about this differently', 'we're stuck on the same ideas', or any framing where the user has exhausted obvious directions and wants disciplined exploration of less-obvious ones. Do NOT trigger for problems where the answer likely exists in the literature and just needs to be found (use ordinary search for that), for purely empirical questions answerable by data, or for engineering decisions that don't require novel theoretical content. The skill produces a layered hypothesis register with explicit novelty and confidence grades, a designated primary candidate, and a handoff package (position paper + experiments list) for hypothesis-verificator to take into validation. Novelty is graded by literature absence, not by configuration: the groundedness dial controls search breadth, not grade inflation."
---

# Layered Research

A procedure for producing novel hypotheses on hard research problems through iterated layers of synthesis. Each layer takes the previous layer's hypotheses, examines what was confirmed, falsified, or left inconclusive against the literature, and synthesizes new hypotheses from the patterns. Genuine novelty emerges deeper in the stack — Layer 1 hypotheses cluster near the literature; by Layer 3 or 4, hypotheses with no close literature match start appearing. A parallel analogy search at each layer transition injects cross-domain seeds.

This skill is the front of the research chain. Its output feeds `hypothesis-verificator` for validation.

## The core principle

**Novelty is earned, not declared.** A hypothesis is novel because the existing literature genuinely does not contain it — verified by search at the layer where the hypothesis is generated, not assumed from depth or configuration. The groundedness dial controls how broadly the skill searches and how many speculative chains it pursues, but the *grade* on any individual hypothesis comes from what the literature actually contains. Loose mode produces more candidates, not more confident ones.

**Confidence and novelty are independent axes.** A high-novelty, low-confidence hypothesis is exactly what this skill exists to produce in volume. A high-novelty, high-confidence hypothesis is rare and exciting. A low-novelty, high-confidence hypothesis is "this is already known — go read X." Conflating these axes is the dominant failure mode and the skill is built to resist it.

**"No novel hypothesis found" is a legitimate result.** If five layers of disciplined work produce nothing the literature doesn't already cover, that itself is a finding: the problem space is well-explored, and the user should reconsider whether genuine novelty is possible here or whether the original question needs reformulating. The skill must not be pressured to produce novelty that isn't there.

## Roles in this collaboration

The user knows the field — terminology, what has been tried, what counts as evidence, what a real hypothesis looks like. Claude does the analytical and search work: runs literature searches, generates hypotheses, performs analogy search, synthesizes patterns across layers, grades novelty and confidence. The user audits at each layer transition, corrects mis-graded entries, supplies domain context Claude lacks, and decides when to branch a candidate into validation.

Claude commits to grades. Hedged non-committal output is useless. State a novelty grade, state a confidence grade, show the reasoning, let the user correct. Folding under user pushback regardless of evidence is also a failure mode — disagreements are recorded explicitly and resolved by what the literature actually contains, not by who pushed harder.

## Operating principles

1. **Literature search runs at every layer.** A Layer 4 hypothesis claimed novel must have a literature search behind that claim from Layer 4, not an assumption that depth implies novelty.
2. **Provenance is mandatory.** Every hypothesis records what it was derived from, by what move, and what specifically about it is novel vs borrowed.
3. **Confidence decays with depth unless held up by evidence.** A pure-synthesis Layer 3 hypothesis starts lower confidence than a literature-attested Layer 1 hypothesis. To stay high-confidence at depth, a hypothesis needs positive evidence: internal consistency under independent derivations, prediction of a known result, supporting analogy with verified structural mapping.
4. **Analogies are first-class hypotheses.** When the analogy-search produces "this problem maps to X from domain Y," that mapping is itself a hypothesis with falsifiable predictions. Validated mappings become evidence. Unvalidated mappings are recorded as suggestions but do not inject claims into the main register without going through a hypothesis cycle.
5. **User audits at every layer transition.** Before generating Layer N+1, show Layer N's grades and ask for corrections. Catches confabulations early when they are cheap to drop.
6. **State is persisted at every meaningful change.** The register is written to disk after every hypothesis added, every literature search recorded, every layer transition, every user decision. Sessions are resumable months later.

## The register

Canonical state. YAML, in-repo at `docs/research_register.yml` by default. Written atomically (temp-file + rename) after every meaningful change.

```yaml
problem:
  statement: <one paragraph, precise framing of the research question>
  scope: <what is in / out of scope>
  background: <accepted priors, known results, agreed-upon facts>
  reformulation_history: <prior problem statements if shifted>

settings:
  groundedness: <constrained | loose>           # default: constrained
  groundedness_set_at: <date>
  layer_cap: <integer, default 5>
  budget:
    max_literature_searches: <integer, default 40>
    max_session_hours: <integer, default 8>
    consumed_searches: <integer>
    consumed_hours: <integer>

layers:
  - number: 1
    status: <generating | searching | grading | reviewed | complete>
    started: <datetime>
    completed: <datetime | null>
    parent_layer: null                          # Layer 1 has no parent
    derivation_summary: <how this layer was generated — for L>=2>
    convergence_check:
      new_hypotheses_in_layer: <count>
      novel_hypotheses_in_layer: <count>        # novelty: partial or fully_novel
      convergence_reached: <bool>               # true if zero genuinely novel
    user_audit:
      reviewed: <bool>
      reviewed_at: <datetime>
      corrections_applied: <list of hypothesis ids modified>
    hypotheses_in_layer: <list of hypothesis ids>

hypotheses:
  - id: H-L1-01                                 # L<layer>-<seq> for uniqueness across layers
    layer: 1
    statement: <one sentence, specific, falsifiable>
    type: <mechanistic | empirical | theoretical | methodological | analogical>
    provenance:
      derived_from: <list of parent hypothesis ids, or "initial" for L1>
      derivation_move: <literature_synthesis | cross_synthesis | analogy_transfer | extrapolation | inversion | contradiction>
      novel_components: <what specifically is new vs borrowed>
      borrowed_components: <what was taken from prior layer or literature>
    literature_pass:
      searches_run: <list of query strings actually executed>
      attested_in: <list of citations where this or near-equivalent appears>
      near_misses: <citations of related-but-not-the-same work>
      search_date: <date>
    novelty: <none | partial | fully_novel>     # graded by literature_pass.attested_in
    novelty_reason: <one sentence justifying the grade>
    confidence: <low | medium | high>
    confidence_reason: <one sentence justifying — what evidence holds depth up if applicable>
    predictions: <what should be observable if true>
    falsified_by: <what would refute>
    open_for_validation: <bool>                 # candidate for branching to verificator
    status: <open | branched | superseded | dropped>
    status_reason: <required when not open>
    branch:
      target_skill: <hypothesis-verificator | other>
      target_register: <path to the branch's register>
      branched_at: <datetime>
      branch_status: <active | completed | abandoned>
      branch_result: <findings written back, when branch completes>

analogies:
  - id: A-L<layer>-<seq>
    layer: <layer at which the analogy was proposed>
    source_domain: <e.g., fluid dynamics>
    source_phenomenon: <e.g., laminar flow through constrictions>
    target_mapping: <how it maps to the research problem>
    structural_correspondences: <list of element-to-element mappings>
    predictions_if_valid: <what the analogy predicts about the target problem>
    validation_status: <unvalidated | validated | rejected>
    validated_by: <what evidence supported the mapping, if validated>
    spawned_hypothesis_ids: <list of hypotheses this analogy seeded, if validated>

branch_decisions:
  - layer: <integer>
    asked_at: <datetime>
    user_response: <continue | halt | branch>
    branched_hypothesis_id: <if user chose to branch>
    rationale: <user's stated reason>

deliverables:                                   # populated at termination
  ranked_hypotheses: <ordered list of hypothesis ids>
  primary_candidate:
    hypothesis_id: <chosen primary>
    chosen_at: <datetime>
    rationale: <why this one>
  position_paper_path: <path to docs/position_paper.md>
  experiments_list_path: <path to docs/experiments.md>

procedure_state:
  current_step: <1-9>
  current_layer: <integer>
  pending_user_check: <what the user is being asked to verify or decide>
  status: <initializing | layer_in_progress | awaiting_audit | awaiting_branch_decision | terminating | complete | paused>
  last_written: <datetime>
  resumption_pointer: <next concrete action when resumed>
  staleness:
    oldest_literature_search: <date>
    flagged_stale: <bool>
```

### Status semantics

**Hypothesis status:**
- `open` — generated and graded; available for further work or branching.
- `branched` — sent to another skill (typically verificator) for validation; result pending.
- `superseded` — replaced by a reformulated version in a later layer.
- `dropped` — found to be already published, internally inconsistent, or refuted at generation time.

**Novelty grade** — based strictly on `literature_pass.attested_in`:
- `none` — direct match found in published work; not novel.
- `partial` — closely related work exists; the hypothesis is a variant or extension.
- `fully_novel` — disciplined search found no close match; the hypothesis is not in the searched literature.

`fully_novel` does not mean "definitely new to all of science" — only that the search conducted didn't surface it. The grade must reflect search effort honestly; if search was shallow, novelty grade is capped at `partial` until more search is done.

**Confidence grade** — independent from novelty. Reflects how likely the hypothesis is to be correct.

## Procedure

### Step 1 — Resumption check

**First action of every session, before anything else.** Check whether `docs/research_register.yml` (or the configured register path) exists at the project location.

If it does:
- Load the register.
- Read `procedure_state` to identify last status, last layer, and `resumption_pointer`.
- Check `staleness.oldest_literature_search` against current date. If older than the user's stated freshness threshold (default: 3 months), set `staleness.flagged_stale: true`.
- Summarize state to the user: current layer, hypotheses by status, last action, what's pending, staleness status.
- Ask: "Continue from where we left off, branch a hypothesis, refresh stale searches, or start a new register?"

If it does not exist: proceed to Step 2.

Do not generate any new hypotheses, run any searches, or modify the register until the resumption check has completed and the user has responded.

### Step 2 — Initialize: problem, settings, budget

If starting fresh, gather:

- **Problem statement.** Refine until precise and falsifiable. Vague problems generate vague hypotheses across all layers.
- **Groundedness setting.** Ask the user explicitly. Default: `constrained`. Explain the difference: constrained mode does deeper literature search per hypothesis and requires more independent derivations before promoting confidence; loose mode runs more analogy seeds per layer and pursues more speculative synthesis chains. Both grade novelty by literature absence; neither grants confidence by configuration.
- **Layer cap.** Default 5. Higher caps allow deeper search but cost more time and risk diminishing returns. Lower caps may terminate before novelty emerges.
- **Budget.** Max literature searches (default 40 across the run), max session hours (default 8). Budget consumption is tracked; the skill terminates honestly when budget is reached.

Write initial register state. From here forward, every meaningful change writes to disk.

### Step 3 — Generate Layer 1

Generate an initial hypothesis set grounded in the existing literature. The goal at Layer 1 is breadth and accuracy, not novelty. Most Layer 1 hypotheses will be literature-attested. That is correct — they form the substrate from which later layers diverge.

For each hypothesis:

- Sharpen the statement until falsifiable.
- Run a literature search. Record exact queries used and citations found.
- Grade novelty based on what the search returned: `none` if direct match, `partial` if closely related, `fully_novel` only if disciplined search found no match.
- Grade confidence: literature-attested hypotheses with strong support start `high`; hypotheses derived by inference start lower.
- Fill provenance with `derivation_move: literature_synthesis` (or `extrapolation` if extending a literature claim).

Aim for breadth: 5-10 hypotheses at Layer 1 is typical, including some "boring" candidates the user may not have considered.

Write to register. Advance `procedure_state.current_layer = 1`, `layers[0].status = grading`.

### Step 4 — Run analogy search (parallel to layer work)

At each layer transition (after Step 3 for L1→L2, after Step 6 for L2→L3, etc.), run a dedicated analogy search.

For the current problem state, search for structurally similar problems in other domains. The search should be broad — physics, biology, economics, computer science, geometry, fluid dynamics, evolution, ecology, network theory — any domain where the problem's *structure* might appear with different surface content.

For each candidate analogy:

- Record source domain and phenomenon.
- Lay out the structural correspondence: what maps to what, and why.
- State what the analogy *predicts* about the target problem if the mapping is valid.
- Mark as `unvalidated`.

Validation rule: an analogy becomes `validated` only when one of its predictions has been independently checked and held up. Until then, it does not inject claims into the main hypothesis stack.

The number of analogy seeds per transition scales with groundedness setting: constrained mode pursues 2-3 promising mappings; loose mode pursues 5-8.

### Step 5 — User audit of current layer

Before generating the next layer, present current layer's state:

- List of hypotheses with their statement, novelty grade, confidence grade, provenance summary.
- Analogies generated at this transition with their validation status.
- Convergence check: how many of this layer's hypotheses are graded `fully_novel`?

Ask the user, in this order:

1. Are any grades wrong? (Mis-graded novelty or confidence — most common audit catch.)
2. Are any hypotheses mis-stated? (Domain expert correction.)
3. Are any analogies worth validating now (i.e., the user knows of evidence that would validate or refute one)?
4. Is any current hypothesis ready to branch into validation now? (See Step 7.)
5. Continue to next layer, halt the skill, or refresh stale searches?

Apply corrections. Write to register. If continuing, proceed to Step 6.

### Step 6 — Generate Layer N+1 from synthesis

The defining move of the skill. Take the audited current layer and synthesize the next layer by examining patterns.

Three lines of attack for synthesis:

1. **Cross-synthesis.** Multiple Layer N hypotheses sharing a deeper structural commitment imply an upstream hypothesis at the structural level. Articulate it. Mark it derived from the multiple parents with `derivation_move: cross_synthesis`.
2. **Contradiction resolution.** If Layer N contains hypotheses in tension, the resolution may be a new hypothesis at a higher level of description. Generate it; mark `derivation_move: contradiction`.
3. **Inversion / extension.** Take a Layer N hypothesis and invert one of its assumptions, or extend it into a domain it didn't cover. Mark accordingly.

For *every* new hypothesis at this layer:

- Run a fresh literature search at this depth. Do not assume depth implies novelty.
- Grade novelty by what the search actually returns.
- Grade confidence — start lower than parent layer by default; only promote to `high` if positive evidence holds it up (independent derivation, prediction of known result, or validated analogy supporting it).
- Fill full provenance.

If any validated analogies exist from Step 4, allow them to seed hypotheses at this layer. Validated analogies' predictions become candidate hypotheses with `derivation_move: analogy_transfer` and the analogy id as a parent.

Convergence check: if this layer produces zero hypotheses graded `partial` or `fully_novel`, the skill has converged — no new ground is being broken. Flag this in `convergence_check.convergence_reached: true`.

Increment layer counter, return to Step 4 (analogy search for the next transition) and then Step 5 (audit) for this newly generated layer.

### Step 7 — Branch decision (asked at every layer transition)

After Step 5's audit, ask the user: "Is any current hypothesis ready to pursue now into experimental validation?"

Three possible responses:

- **No, continue.** Default. Proceed to Step 6 (generate next layer). The branched_decisions log records the decision and rationale.
- **Yes, branch — and continue layered work.** User identifies a hypothesis to send into `hypothesis-verificator`. The skill writes the branch reference in the hypothesis (`status: branched`, `branch.target_register`, `branch.branched_at`), initializes a verificator register for it, and continues the layered work in this session. When the verificator branch later completes (in the same session, a later session, or months later), its findings are written back to `hypothesis.branch.branch_result`.
- **Yes, branch — and halt.** User wants to focus on the branched hypothesis. Skill records branch as above, sets `procedure_state.status = paused`, and terminates with full deliverables (Step 9) reflecting the state at halt.

Default behavior is to continue. The user halts only by explicit choice.

### Step 8 — Termination

Termination conditions, any of which triggers Step 9:

1. **Layer cap reached** — `current_layer == settings.layer_cap` and Step 5 audit complete.
2. **Convergence** — most recent layer has `convergence_check.convergence_reached: true` (no new partial/novel hypotheses generated).
3. **Budget exhausted** — `consumed_searches >= max_literature_searches` or `consumed_hours >= max_session_hours`.
4. **User halt** — explicit user decision at Step 5 or Step 7.

When any condition is met, write to register the termination reason and proceed to Step 9.

### Step 9 — Deliverables

Three artifacts produced at termination:

**1. Ranked hypothesis list.** Order all `open` hypotheses by:
1. Novelty grade (`fully_novel` > `partial` > `none`).
2. Confidence grade within novelty tier (`high` > `medium` > `low`).
3. Decisiveness for the research question if confirmed (Claude's judgment, recorded in rationale).

Write to `deliverables.ranked_hypotheses` and surface to the user.

**2. Primary candidate.** Ask the user to designate one hypothesis as the primary candidate to take into validation. The skill recommends one (typically the top of the ranked list, but with reasoning about why this specific one over its neighbors); the user accepts or overrides.

Write to `deliverables.primary_candidate`.

**3. Position paper + experiments list.** Generate two markdown documents in `docs/`:

- `position_paper.md` — a synthesis of the layered work. Sections: problem framing, what the literature covers (Layer 1), what the layered synthesis revealed (Layers 2+), the novel hypotheses and their provenance, the primary candidate and rationale, open questions. Length scales with layer depth: 2-4 pages for a 3-layer run, 5-10 pages for a 5-layer run.
- `experiments.md` — concrete validation experiments for the primary candidate, formatted as input to `hypothesis-verificator`. For each experiment: hypothesis under test, predicted observation, falsification criterion, estimated effort, dependencies.

Write paths to `deliverables.position_paper_path` and `deliverables.experiments_list_path`.

Set `procedure_state.status = complete`. Use `present_files` to surface the position paper and experiments list to the user.

The handoff is now ready. The user (or Claude in a new session) invokes `hypothesis-verificator` with `experiments.md` as input; verificator records `branched_from: <this register path>` and writes its findings back to the relevant hypothesis entries on completion.

## Failure modes to resist

- **Conflating novelty with confidence.** The two axes are independent and must remain so. A confident novel hypothesis requires more evidence than either confident-and-known or unconfident-and-novel. Treating depth as conferring confidence is the dominant failure mode.
- **Skipping literature search at deeper layers.** "It's Layer 4, of course it's novel" is the trap. Every hypothesis at every layer gets a search behind its novelty grade.
- **Producing novelty under pressure.** If the skill returns "no novel hypothesis found" after honest work, that is the correct output. Do not invent novelty to satisfy the run.
- **Letting analogies inject claims unvalidated.** An analogy is a hypothesis with structural mapping. Until at least one of its predictions has been checked, it does not get to seed first-class claims in the main stack.
- **Stale state on resumption.** Resuming a months-old register without checking literature staleness produces work built on outdated assumptions. The staleness check at Step 1 is non-optional.
- **Lost provenance.** Three layers deep, every hypothesis must trace cleanly back to grounded roots. If the chain breaks, the hypothesis is suspect regardless of how plausible it sounds.
- **Folding under user pushback regardless of evidence.** If the user disagrees with a grade, record the disagreement and resolve by what the literature contains, not by capitulation. Symmetric: if Claude disagrees with the user's domain reading, record and resolve by evidence, not by stubbornness.
- **Auto-refreshing stale searches without asking.** Auto-refresh silently invalidates prior work. Surface staleness, let the user decide.
- **Branch references that don't survive.** When a hypothesis is branched into verificator, the cross-reference must be written to disk at branch time, not deferred. Otherwise a session crash between branch and write loses the link.

## Turn-level output contract

Every response while this skill is active must include, in order:

1. `procedure_state` — current step, current layer, status, what was just written.
2. Register diff (or full register on first turn / after resumption). YAML.
3. Prose summary — what was learned this layer, what novelty emerged, what the user is being asked.
4. The substantive content of the turn — generated hypotheses, search results, audit questions, deliverables.

The register is written to disk after every meaningful state change, not at end of turn. A session ending mid-turn must leave the register in a valid resumable state.

## Interaction with other skills in the chain

- **From `hypothesis-verificator`**: rare — typically verificator produces findings that lead into `project-pivot`, not back into this skill. But if verificator's findings reveal a deeper question that wants layered exploration, the user can start a new `layered-research` run citing the verificator findings as background.
- **To `hypothesis-verificator`**: the primary handoff. `experiments.md` is structured as verificator input. The user invokes verificator with that file; verificator processes hypotheses, records `branched_from`, and on completion writes findings back to this register's hypothesis entries.
- **With `project-pivot`**: indirect. If layered research reveals that the current project framing is wrong (a Layer 3 finding contradicts the project's premise), the user takes that finding into project-pivot, which then triggers architecture-proposal and so on.
- **With `architecture-proposal`**: not directly chained, but the position paper from this skill is an input the user might supply to architecture-proposal when designing a new direction informed by the layered work.
