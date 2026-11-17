---
name: arena
description: "Spawn N parallel candidates at the same task, pick a base, graft the strongest parts of the losers into it. Use for /arena, 'arena this', 'throw it in the arena', or when one attempt at a non-trivial artifact would lock in the wrong shape."
disable-model-invocation: true
---

# Arena

Open a todolist by writing and updating `<appDataDir>/brain/<conversation-id>/task.md` with `write_file` or `write_to_file` (the system prompt names `<appDataDir>` and `<conversation-id>`). Antigravity has no native todolist tool, so that file is the list. Give it one entry per phase (Frame, Fan out, Cross-judge, Pick, Graft, Verify) before launching anything.

## Phase A: Frame

Every candidate gets the same prompt, so the prompt is the contract.

1. State the artifact each candidate is producing.
2. Derive the rubric, 3-6 concrete gradeable criteria for what success looks like on *this* task. It is the picker's tool in Phase D. Candidates only see the task.
3. Pick the runners from the `arena runners` line in `~/.gemini/config/rules/pstack-models.md`, by default one each on `Model: "pro"` and `Model: "flash"`. That file is an always-applied rule, already in context when present, so do not `read_file` or `view_file` it. An `inherit`, `auto`, or `inherit-parent` entry is the parent model (`Model: "inherit"`). If `invoke_subagent` rejects a tier, use the nearest valid tier (`pro`, `flash`, or `flash_lite`) or drop the seat, keep at least two runners, and say so. If dropped seats leave fewer than two runners, add runners until there are two, or one when the line has one entry. Run each added runner on a remaining runner's tier, or `Model: "inherit"` when none remain, and say so. Spawn more runners when the arena covers multiple design directions. Use the same model tier N times when the work is generation-bound rather than judgment-sensitive. Pick each subagent's model tier for the job, per the Subagents section of the **poteto-mode** skill.
4. Give each candidate its own output location (a git worktree or `Workspace: "branch"` where possible, otherwise `/tmp/arena-<slug>/candidate-<n>/`), per the **separate-before-serializing-shared-state** principle skill.

## Phase B: Fan out

Spawn all N subagents in one `invoke_subagent` call (`TypeName: "poteto-agent"`), each with the task, the path to the shared grounding, its own output path, and instructions to produce the artifact and a short rationale naming the alternatives it considered and what it rejected.

If a candidate produces no output, proceed with N-1 and note the dropout.

## Phase C: Cross-judge

After all Phase B candidates complete, spawn one read-only judge (`TypeName: "research"`) from the `arena cross-judge pool` line in `~/.gemini/config/rules/pstack-models.md`, by default `pro`, `flash`. That file is an always-applied rule, already in context when present, so do not `read_file` or `view_file` it. The judge is never the parent's tier, so use the first entry on a different tier. An `inherit`, `auto`, or `inherit-parent` entry is the parent model (`Model: "inherit"`), so it never judges. If you do not know the parent's tier, use the first real tier and say that the tier check did not run. If `invoke_subagent` rejects a tier, use the nearest valid tier, then the next entry, or skip the judge, and say so. While you read in Phase D, the judge scores each candidate, by path label, on each rubric criterion and recommends a base.

## Phase D: Pick a base

Read every candidate end to end before picking. Score each against the rubric criterion by criterion, not on holistic feel, and compare with the cross-judge when Phase C ran one. Agreement on the base confirms the pick. Disagreement means one of you is biased or the rubric was ambiguous. Read both rationales before deciding.

Pick the base a future maintainer can extend most easily without breaking invariants. When two feel tied, prefer the cleaner boundary or smaller API, per the Laziness Protocol.

## Phase E: Graft

Walk each losing candidate once more for what is worth porting into the base, usually one or two things per candidate, not most of it. Fold each graft in by hand, per the **redesign-from-first-principles** principle skill, so the result stays coherent under one mental model. Don't paste mechanically.

If the candidates converge on one shape, that is strong agreement. Note it and ship the consensus shape. No graft is needed. If they wildly diverge, Phase A was under-specified. Reframe and re-run rather than averaging the divergence.

## Phase F: Verify

Verify the synthesized artifact like any other output, per the **prove-it-works** principle skill. If verification finds a problem the arena missed, either Phase A was wrong (re-frame and re-run) or a candidate caught it and you missed the graft (go back to Phase E). Don't paper over it.

## Outputs

One synthesized artifact, with a short synthesis note alongside naming the base and why, the cross-judge's verdict or that Phase C skipped the judge, each graft with its source candidate, what was rejected and why, any convergence or dropouts, and the verification result.
