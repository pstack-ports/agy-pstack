---
name: setup-pstack
description: Configure which model tiers pstack uses per role and at what reasoning budget. Writes an always-applied rule that overrides the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

Write `~/.gemini/config/rules/pstack-models.md`, an always-applied rule that sets pstack's model tier per role.

## Steps

### 1. Detect available model tiers

Antigravity's `invoke_subagent` accepts four `Model` tiers: `pro`, `flash`, `flash_lite`, and `inherit`. You can also run `agy models` to inspect the underlying models available in the current session. Passing `pro`, `flash`, or `flash_lite` resolves to the backend's default model and default reasoning effort for that tier, whereas `inherit` (and the aliases `inherit-parent` and `auto`, which map to `Model: "inherit"`) clones the parent session's exact model and reasoning effort.

### 2. Load current state

The default role-to-model mapping is the rule shape shown in step 5 below. If `~/.gemini/config/rules/pstack-models.md` already exists, read it and treat its `# budget` line and its role values as the current choices. Otherwise start from those defaults. A line whose role is not in step 5, such as `how explorer` or `why investigators`, is from a retired role. Drop it.

A rule without the `# defaults: 2` line comes from an older setup. That setup wrote a line for every role, so some of its values are old defaults, not choices. An old slug is one that matches `grok-*`. In this port the old fast-model default was written as `flash`. Before step 3, replace these values:

- `flash`, or an old slug, in `feature, refactoring`, `bug-fix`, `perf-issue`, or `hillclimb` takes the role's step 5 default.
- `flash`, or an old slug, in `reflect tooling` or `swarm workers` takes the role's step 5 default.
- `pro`, or a `claude-opus-*` value, in `hardest tasks` takes the role's step 5 default.
- In a panel role (`arena runners`, `arena cross-judge pool`, `architect runners`, `interrogate reviewers`), a list of exactly `pro` and `flash` (either order), or exactly one `claude-opus-*` entry and one old slug, takes the role's step 5 list. In any other list, each old slug becomes `flash`.

Every other value is the user's choice and stays, including an old slug in any other role, `flash_lite`, and every `inherit`, `inherit-parent`, or `auto` value. Tell the user which values you replaced.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Prefer `ask_question` over free text. Offer these four options with these exact labels, and name the current budget when the rule records one. With no rule, say that `large` matches the skill defaults.

- `unlimited - pro reasoning`
- `large - pro and flash split`
- `medium - flash reasoning`
- `small - flash_lite reasoning`

**(b) Apply it.** Build the working table from the skill defaults, and on a re-run keep any role you changed by tier, list, or alias (`inherit`, `inherit-parent`, `auto`). Each role has a target tier, which is its default moved by the budget on the ladder `flash_lite` < `flash` < `pro`. `large` keeps the skill defaults (`pro` for code delegates, judgment, prose, and hardest tasks; `flash` for swarm workers and reflect tooling). `unlimited` raises each role one step, capped at `pro`. `medium` lowers each role one step (`pro` to `flash`, `flash` to `flash_lite`, each panel entry too). `small` lowers each role two steps. A target at either end of the ladder stays there. `inherit`, `inherit-parent`, and `auto` do not change.

**(c) Show the roles and confirm.** Show every role with its model tier. Also list each line step 2 dropped. Ask whether to accept as-is or change specific roles, offering `pro`, `flash`, `flash_lite`, and `inherit` (which runs that role on the parent chat model and preserves its reasoning effort) as the options. Prefer `ask_question` over free text. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena runs one judge. It tries only values whose tier differs from the parent's, in list order, so order it by preference. An `inherit`, `inherit-parent`, or `auto` value never judges. `swarm workers` is the default tier for every worker unless a race or comparison assigns another tier per arm.

### 4. Validate

Every tier written must be one of `pro`, `flash`, `flash_lite`, `inherit`, `inherit-parent`, or `auto`.

### 5. Write the rule

Write `~/.gemini/config/rules/pstack-models.md` as plain Markdown (no YAML frontmatter) with the `# defaults: 2` line, a `# budget` line, and one line per role, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent. Shape:

```
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit`, `inherit-parent`, or `auto` as a value: the role runs on the parent chat model and preserves its reasoning effort (`Model: "inherit"`). Alias entries in a panel list still count toward its fan-out.
# defaults: 2
# budget: large (pro / flash)
feature, refactoring: pro
bug-fix: pro
perf-issue: pro
hillclimb: pro
judgment and prose: pro
hardest tasks: pro
reflect tooling: flash
reflect judgment, divergent, synthesizer: pro
arena runners: pro, flash
arena cross-judge pool: pro, flash
swarm workers: flash
architect runners: pro, flash
interrogate reviewers: pro, flash
```

### 6. Confirm

Tell the user the rule was written and that it applies to new sessions. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.

### 8. Offer a formal verification skill (optional)

Check whether the project already has a formal verification skill (a project skill that runs a model checker, a proof, or another verifier against the code). If not, offer once: "want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with /create-formal-verification-skill." On yes, invoke `/create-formal-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.
