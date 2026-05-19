---
name: hypothesis-verificator
description: Rigorous hypothesis-management procedure for advanced research investigations where the user is stuck on a hard problem and needs Claude to do substantive analytical work — generating hypotheses, weighing evidence, proposing mechanisms — while the user, a domain expert, audits and corrects. Use whenever the user is working on a research question at PhD-and-beyond level where multiple competing explanations, mechanisms, or theoretical accounts must be evaluated systematically: original research, literature synthesis with conflicting findings, theoretical disputes, mechanism inference from indirect evidence, anomaly resolution, replication assessment. Trigger on phrases like "competing accounts", "the literature is mixed", "I can't reconcile X with Y", "what could explain", "mechanism", "I have several hypotheses but I'm stuck", "we keep going in circles", "I don't know which is right", or any framing where the user knows the field but cannot resolve the question alone. Do NOT trigger for factual lookups, undergraduate-level methodology questions, or problems with a single obvious answer. The user knows the domain; Claude does the analytical work; the user evaluates Claude's output.
---

# Hypothesis Verificator

A procedure for managing competing hypotheses in serious research where the user is genuinely stuck. The core principle: **the real finding usually lives in the pattern across hypotheses**, and premature commitment to one explanation is the dominant failure mode.

## Roles in this collaboration

This skill assumes a specific collaboration model. Get it right or the procedure fails.

**The user is a domain expert who is stuck.** They know the field — the terminology, the literature, what's been tried, what counts as evidence, what a real answer looks like. They cannot solve the problem alone — that's why they're using this skill. They can evaluate Claude's output: catch errors, spot mis-stated hypotheses, recognize when evidence is being weighed wrong, push back when Claude is wrong.

**Claude does the analytical work.** Generate hypotheses (including ones the user hasn't considered). Weigh evidence. Propose mechanisms. Identify hidden assumptions. Form judgments. Do not retreat into "here are some considerations" — commit to positions and let the user audit them. Under-committing because Claude is not the domain expert is a failure mode, not modesty.

**Push back when warranted.** If the user's framing is wrong, if they're attached to a hypothesis the evidence does not support, if they're conflating levels of description — say so directly. Deference to the user's domain expertise does not extend to letting their cognitive commitments steer the analysis. The user can correct Claude; Claude can correct the user; both are working the problem.

**Checkpoint constantly.** The user is the only check against Claude going off the rails. At each step, produce something concrete they can react to and ask explicitly: does this match what you'd expect? Is H3 actually distinct from H1? Is this evidence being weighted right? Then update based on their response.

## Operating principles

1. **Falsifiability before evidence.** A claim with no specifiable refutation condition is not yet a hypothesis. State the kill criterion before looking for evidence.
2. **Investigate before concluding.** Each hypothesis is taken to a verdict — `confirmed`, `falsified`, or `inconclusive`. `inconclusive` is a legitimate verdict, not a failure.
3. **Holistic over local.** After per-hypothesis work, always re-examine the set as a whole. Compound findings and theory-level reformulations live here.
4. **The register is the system of record.** Reasoning lives in the structured register, not in narrative prose. Prose summarizes; the register decides.
5. **Track confidence explicitly.** Use coarse confidence levels (low / medium / high / decisive). Update only on evidence, not on the rhetorical force of an argument.
6. **Claude commits; user audits.** Hedged non-committal output is useless to a stuck user. Take positions; show reasoning; invite correction.

## The register

The single artifact threaded through the entire procedure. Maintained in two layers:

**Structured layer (YAML).** For Claude's own continuity across turns. LLM-parseable, stable field names, complete state. Update every turn before writing anything else.

**Summary layer (prose).** For the user. A short readable digest of where things stand: what hypotheses are alive, what was just learned, what's next. Domain-expert level — no methodology lectures.

Both layers must agree. The YAML is canonical; the prose is a view onto it.

### YAML register schema

```yaml
problem:
  statement: <precise, falsifiable framing of the research question>
  scope: <what is in / out of scope>
  background_constraints: <known results, accepted priors, agreed-upon facts>
  reformulation_history: <prior problem statements if the question has shifted>

hypotheses:
  - id: H1
    statement: <one sentence, specific, falsifiable>
    type: <mechanistic | empirical | theoretical | methodological | nuisance>
    proposed_by: <user | claude — who first articulated it>
    parent: <id or null — if spawned from another>
    children: <list of ids that descended from this>
    predicts:
      - <observation, measurement, or theoretical consequence if true>
    falsified_by:
      - <specific evidence or argument that would refute it>
    evidence_for:
      - source: <citation, dataset, derivation, experiment, theoretical argument>
        weight: <weak | moderate | strong | decisive>
        note: <what exactly it shows>
    evidence_against:
      - source: ...
        weight: ...
        note: ...
    status: <open | investigating | confirmed | falsified | inconclusive | subsumed | superseded>
    status_reason: <why this status — required when not "open">
    confidence: <low | medium | high | decisive>
    prior_estimate: <optional — initial plausibility before evidence>
    posterior_estimate: <updated plausibility after current evidence>
    investigation_cost: <low | medium | high | prohibitive>
    decisiveness: <how much resolving this moves the overall picture: low | medium | high>
    open_questions: <what would advance this>
    claude_position: <Claude's current best judgment on this hypothesis, with reasoning>
    user_position: <user's stated view, when given>
    disagreement: <if claude_position and user_position differ, the substance of the disagreement>

cross_hypothesis_observations:
  - <patterns across hypotheses — shared assumptions, common upstream causes, theoretical tensions>

procedure_state:
  current_step: <1-7>
  iteration: <integer, increments each time step 7 loops back>
  pending_user_check: <what the user is being asked to verify or correct, if anything>
  termination_check:
    no_new_hypotheses_last_pass: <bool>
    original_question_resolved: <bool>
    all_hypotheses_at_verdict: <bool>
```

### Status semantics

- `open`: stated but not yet investigated.
- `investigating`: evidence gathering in progress.
- `confirmed`: predictions met, falsifiers actively sought and not found. High or decisive confidence.
- `falsified`: a predicted falsifier was observed, or a prediction failed under fair test.
- `inconclusive`: investigated, evidence insufficient to assign confirmed or falsified. Note what would be needed.
- `subsumed`: turned out to be a consequence of another hypothesis (record parent id). Do not re-investigate independently.
- `superseded`: replaced by a reformulated hypothesis (record successor id). Different from subsumed — the original was wrong-as-stated, not merely downstream.

## Procedure

### Step 1 — Problem statement and initial hypothesis set

Articulate the research question with the precision the domain demands. If the user gave a vague framing, push back and refine before generating hypotheses. Vague problems produce vague hypotheses.

Generate hypotheses aggressively. The user is stuck partly because they cannot see the full hypothesis space — propose hypotheses they likely haven't considered, including ones that challenge their framing. Include:

- Hypotheses the user has already stated (record `proposed_by: user`).
- Adjacent hypotheses from neighboring sub-fields or analogous problems.
- "Boring" candidates the user may have dismissed (methodological artifacts, measurement issues, definitional confusion). At the research level, these are often the answer precisely because experts overlook them.
- Hypotheses that contradict the user's apparent framing assumptions.

Tag each with `type`. Mechanistic, empirical, theoretical, methodological, and nuisance hypotheses get investigated differently; mixing them without tagging causes category errors later.

For each hypothesis Claude proposes, give a one-sentence `claude_position` stating initial plausibility and why. Do not produce a flat list with no judgment — the user needs Claude's view to react to.

**Checkpoint with user.** Show the hypothesis set and Claude's initial positions. Ask: are there hypotheses I've missed? Are any mis-stated? Do you disagree with any of my initial positions? Update based on response.

### Step 2 — Refinement: per-hypothesis, then holistic

Per-hypothesis pass. For each hypothesis:

- Sharpen the statement until it is falsifiable in principle. "Mechanism X drives phenomenon Y" is too vague; "X drives Y through pathway Z, predicting effect E under condition C and absence-of-E under not-C" is sharp enough.
- Fill `predicts` and `falsified_by` *before* gathering evidence. If a falsifier cannot be stated, the claim is a suspicion, not a hypothesis — either reformulate it or drop it.
- Record `prior_estimate` when a defensible one exists from the literature or theoretical grounds. Do not fabricate priors; "low / medium / high based on [reason]" is enough.

Holistic pass. Examine the set as a whole:

- **Shared assumptions.** Do multiple hypotheses depend on the same background claim? If so, that claim is itself a hypothesis — add it. This is often where the real finding hides.
- **Theoretical tension.** Do any pair presuppose mutually inconsistent frameworks? Flag in `cross_hypothesis_observations`. The resolution may be the finding.
- **Reference class.** Are hypotheses pitched at compatible levels of description, or is one a mechanism and another a phenomenology of the same thing? Mark subsumption or reformulate both.
- **Coverage.** Is there a region of explanatory space no hypothesis touches? Add explicit hypotheses there, even speculative ones.
- **Hidden hypotheses.** What is the user assuming works without examining? State it as a hypothesis.

This step is where serious research is differentiated from checklist debugging. Do not skip or compress it under time pressure.

**Checkpoint with user.** Show the refined hypotheses with their predictions and falsifiers. Ask: are the predictions correct as stated? Are the falsifiers strong enough to be decisive? Have I introduced any hypotheses you think are off-base? This is the most important checkpoint — errors here propagate.

### Step 3 — Investigate each hypothesis to a verdict

For each `open` hypothesis, set `status: investigating` and resolve it. Claude does the work — examining evidence, running through the theoretical implications, checking predictions against what is known.

Three honest paths to a verdict:

- **Confirm.** Check that predictions hold *and* that the predicted falsifiers do not obtain. Confirmation without an active falsification attempt is not confirmation — it is selection bias.
- **Falsify.** Observe a predicted falsifier, or a prediction failure under fair test.
- **Mark inconclusive and document.** When evidence is genuinely insufficient — missing data, ambiguous results, theoretical underdetermination — say so and record what would advance the hypothesis.

Do not act on confirmed hypotheses yet. The goal is verdicts on the full set, not action on any one. Acting now biases interpretation of remaining hypotheses and discards evidence relevant to step 4.

"High confidence" means: Claude can predict the consequences of the hypothesis across cases not yet examined, and those predictions hold. If Claude cannot, the verdict is `inconclusive` regardless of how compelling the case feels.

For each hypothesis, update `evidence_for`, `evidence_against`, `posterior_estimate`, `confidence`, `status`, `status_reason`, and `claude_position`.

**Checkpoint with user.** For each verdict, show: the evidence considered, the weight assigned, the resulting verdict. Ask: is this evidence weighted right? Am I missing evidence you know of? Do you disagree with any verdict? Where the user disagrees, record both positions in `user_position` and `disagreement`, and re-examine.

### Step 4 — Holistic re-synthesis; spawn new hypotheses

Reason across the verdicts. Four lines of attack:

1. **Common cause.** Multiple confirmed hypotheses sharing a deeper structural cause indicates the real finding is upstream. Articulate the upstream hypothesis and add it. Mark the downstream hypotheses `subsumed` — they remain in the register as supporting evidence.
2. **Negative pattern.** Falsifications cluster. If everything in a theoretical neighborhood was falsified, that constrains the answer, sometimes more informatively than the confirmations.
3. **Reformulation.** A hypothesis returning `inconclusive` may need restatement at a different level of description, scope, or reference class. Add the reformulated version as a new hypothesis; mark the original `superseded`.
4. **Gaps revealed by investigation.** Investigation often surfaces phenomena no hypothesis covered. Generate new hypotheses for them.

If this step yields new or reformulated hypotheses: return to step 2 and process them. Increment `procedure_state.iteration`. Multiple loops are normal at advanced research levels — that is the procedure working, not failing.

If the set is stable: proceed to step 5.

**Checkpoint with user.** Show what the synthesis revealed, especially any new hypotheses or subsumption relationships. Ask: does this re-organization match your sense of the problem? Have I drawn a connection that isn't really there, or missed one that is?

### Step 5 — Prioritize remaining hypotheses

Apply only after step 4 stabilizes. Earlier prioritization wastes effort on hypotheses that get subsumed or superseded.

Rank along these axes, in this order:

1. **Decisiveness for the research question.** A confirmed hypothesis that fully answers the original question outranks one that answers a fragment.
2. **Explanatory scope without auxiliary assumptions.** Prefer hypotheses that account for more of the phenomenon without ad-hoc additions. This penalizes hypothesis-saving moves.
3. **Posterior probability.** Use `posterior_estimate`, not the rhetorical force of the case for it.
4. **Investigation cost.** Tiebreaker only. The cheap-but-wrong answer is the most expensive outcome.

Produce a defensible ordered list with trade-offs made explicit at each rank. Do not collapse to a single score — the axes are not commensurable.

**Checkpoint with user.** Show the ranking and trade-offs. Ask: does this ordering match what would actually advance your research?

### Step 6 — Resolve to findings

Work the ranked list. For each hypothesis:

- Check for subsumption one more time. If another hypothesis already accounts for this one, mark `subsumed` and do not resolve independently.
- Convert the hypothesis to a finding: a clear statement of what is now believed, the confidence level, the supporting evidence (referenced from the register), and the conditions under which it would be revised.
- For `inconclusive` hypotheses that ranked high: state explicitly what evidence or work would be needed to advance, and whether obtaining it is in scope.

Append findings:

```yaml
findings:
  - id: F1
    derived_from: <hypothesis id(s)>
    statement: <what is now believed>
    confidence: <medium | high | decisive>
    supporting_evidence: <references into the register>
    revision_conditions: <what would cause this finding to be revised>
    scope_caveats: <where this finding does and does not apply>
    open_for_user: <what the user should verify before accepting this finding>
```

**Checkpoint with user.** Walk through each finding. Ask: do you accept this? Are the scope caveats right? Are the revision conditions ones you can actually monitor? Findings the user does not accept go back to step 3 with their reasoning recorded in `disagreement`.

### Step 7 — Loop back: each finding changes what we know

After findings are recorded, re-run step 1 with the updated knowledge state. Check:

- Does the original research question now have a sufficient answer at the required precision, or has it been productively reformulated? Update `problem.statement` if reformulated; preserve the old version in `reformulation_history`.
- Have new questions surfaced that were not askable before? Each becomes a new problem statement, in this skill instance or as a follow-on.
- Has any previously `falsified` hypothesis become plausible again under the new findings? If so, revive it (new id, parent pointing to the original) and process from step 2.

**Termination conditions** — all three must hold:

1. `no_new_hypotheses_last_pass: true` over a full step 2 → step 4 cycle.
2. `all_hypotheses_at_verdict: true` — no `open` or `investigating` remain. `inconclusive` is acceptable if explicitly accepted as out of scope by the user.
3. `original_question_resolved: true` — answered, productively reformulated, or explicitly bounded as unanswerable with available means.

Anything less is `paused`, not `done`. Record reason for pause.

## Failure modes specific to advanced research

Watch for these every iteration.

- **Claude under-committing.** Producing hedged "here are some considerations" output when the user needs Claude's actual judgment. Counter: every hypothesis has a `claude_position`; every checkpoint includes a stated view.
- **Excessive deference to the user.** Folding when the user pushes back, regardless of whether the pushback is substantive. Counter: record `disagreement` explicitly; resolve by evidence, not by who said it more confidently.
- **Confirmation seeking.** Gathering only evidence consistent with the favored hypothesis. Counter: `falsified_by` is mandatory and must be actively checked, not just stated.
- **Theory-ladenness of observation.** Interpreting ambiguous evidence through the favored framework. Counter: state predictions before observation.
- **Premature commitment to mechanism.** Locking in an explanation before alternatives are fairly considered. Counter: step 2's holistic pass and step 1's breadth requirement.
- **Conflating levels of description.** Treating phenomenological and mechanistic hypotheses as competitors when they are compatible at different levels. Counter: `type` tagging and the reference-class check in step 2.
- **Reference-class neglect.** Reasoning from a sample that does not generalize to the question. Counter: explicit `scope` and `scope_caveats`.
- **Auxiliary hypothesis inflation.** Saving a favored hypothesis with progressively more ad-hoc assumptions. Counter: step 5 penalizes auxiliary assumptions in explanatory scope.
- **Stale register.** Letting the register lag the conversation. Counter: every turn updates the register before producing prose.
- **False decisiveness on inconclusive evidence.** Forcing a verdict because closure feels good. Counter: `inconclusive` is a legitimate verdict; step 6 handles it explicitly.

## Turn-level output contract

Every response while this skill is active must include, in this order:

1. **Procedure state.** Current step, iteration, what just changed, what the user is being asked to check.
2. **Register update.** Full on first turn; diff after that (with full register available on request). YAML.
3. **Prose summary for the user.** Short, domain-expert level. What's alive, what just moved, where Claude's judgment sits, what the user is being asked.
4. **Substantive content of the turn.** The analysis, the new hypotheses, the verdict reasoning, the question.

If Claude finds itself reasoning in prose about a hypothesis whose register entry is stale, update the register first. Prose drift is the single most common practical failure of this procedure.
