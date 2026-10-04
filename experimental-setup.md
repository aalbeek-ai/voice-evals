# Experimental Setup

How a voice agent gets scored, which controls keep measurement error small, and where the setup hits its limits. Built for a property management company; the method works for any industry, the example cases don't.

## What's measured

**The whole agent system at one version.** Everything that changes behavior belongs to it: system prompt, knowledge base, variables, tool descriptions, workflows, and the platform's dashboard settings. They share one version number, set identically in the prompt's frontmatter, on the platform, and as `Prompt version` in `01-Setup`. Runs from two versions, scored together, measure nothing.

## Instrument

Calls are matched to cases **by queue, not by anything said in the call**, and not by caller ID or time of day. `01-Setup` shows the next open case: the first `Capability` case in `03-Cases`, not held out, with fewer runs in the current version than its `Calls`. The grader reads the same cell before writing the run, so it still names the case just called. After a botched call, delete its row and the case comes back. A spoken codeword came first and was dropped: speech recognition missed it so often that single cases had to be called six times.

**The path decides who scores.** The three **liability paths** run without an LLM — a false "pass" there would cause real damage, not just a measurement error:

| Path | Grader | Checks |
| --- | --- | --- |
| `emergency` | Rule | Mandatory announcement said the required number of times, within the turn limit |
| `dispatch` | Rule | Transfer succeeded (`succeeded` tool line or `disconnectReason: call_transfer`), no announcement, within the turn limit |
| `attack` | Rule | Denylist from `01-Setup` doesn't appear in the transcript |
| everything else | Judge | `Pass if` / `Fail if`, ticket, fixed list of basic errors |

The three reserved names are hardcoded in the `grade` node. A path your agent doesn't have needs no change — just no case with that `Path`; only renaming a reserved path or adding a new rule-graded one means editing `grade`. Every other path name is free text for the judge. Rule graders read neither the criteria nor the ticket; on those paths, `Pass if`, `Fail if`, and `Expected ticket` are notes for the human reviewer.

- **Transfer is measured by state, not by what was said.** The model says "I'll connect you" even when it never called a tool. Only a `succeeded` row counts: every transfer, failed or not, first logs an attempt row.
- **Transcript and check terms are normalized the same way** — lowercased, `ä/ö/ü/ß` folded to `ae/oe/ue/ss`, everything else to spaces. Speech-to-text doesn't reliably keep umlauts; fold only one side and a denylist word can silently miss.

**The judge is never the agent's model** — LLMs rate their own outputs higher than humans do (<a href="https://arxiv.org/abs/2404.13076" target="_blank" rel="noopener noreferrer">Panickssery et al. 2024</a>). It gets criteria, transcript, ticket, `disconnectReason`, and tool calls, never the system prompt, or it scores intent instead of outcome. If the data isn't enough, it answers `unclear`; if the API fails, the run ends as `unclear` too, never as a silent fail.

## Controls

- **A twin per trigger.** Every case where a behavior *should* happen has one where it should not: same ID with `-Z-`. "One-sided evals create one-sided optimization" (<a href="https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents" target="_blank" rel="noopener noreferrer">Anthropic</a>). For the rule grader, `-Z-` in the case ID flips the check: announcement and transfer must be absent.
- **Held-out set.** About one case in eight — in the live set, five of 37 — is never called or looked at until the gate. Only this number shows whether the agent generalizes instead of overfitting to the set. `Held out` is a checkbox a human ticks before round one.
- **The instrument may change mid-round, the agent system never.** Allowed to sharpen: `Pass if`, `Points 0-2`, the judge prompt, the turn limit and denylist. Untouched: system prompt, knowledge, variables, tools, workflows, dashboard — changing those describes two agents under one version.
- **Re-scoring instead of re-calling.** A sharpened criterion re-scores the affected rows; the transcript is already there. Mark the row `[manually adjusted - YYYY-MM-DD HH:MM]`.
- **Reference solution.** One known-working transcript per case: proof the task is solvable, and a check on the grader — after a grader change the reference must still pass. The first passing transcript of a case becomes its reference.

## Metrics

**`pass^k`, not `pass@k`.** A case passes only if *all* its calls pass. In <a href="https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents" target="_blank" rel="noopener noreferrer">Anthropic's example</a>, the same agent hits 97% `pass@3` but 39% `pass^3`. An agent on the phone gets one attempt per call, so `pass^k` is the honest one.

| Rate | Stack | Expectation |
| --- | --- | --- |
| Pass | Capability, excluding held-out | low, deliberately |
| Δ vs. previous version | same stack | the stop criterion |
| Regression | Regression | 100%, otherwise something broke |
| Held-out | Held-out | only at the gate |

A rate counts only once no case in its stack is `open`. `Points 0-2` feeds no rate: it shows how far a failed case missed, for the human review and the skill. `Turns` counts conversation turns only, so a transfer's tool rows don't trip the turn limit. The `05-Results` formulas reach case 40; a longer set means dragging them down.

**Two loops.** Every case starts as `Capability`. A case that passes all its calls two rounds in a row becomes `Regression` and leaves the round; a regression case that fails goes back. Round cost tracks the capability stack, not the size of the set. Regression runs go by trigger: once before go-live, then after every change to the system, every platform update, and every model switch.

## Procedure

Setup is in the README; the analysis after each round is in the skill. What neither covers:

- **Gate KPIs** go into `01-Setup` before round one: yes/no, readable from the runs, and at least one measures whether the call reached its goal — otherwise the gate only shows that nothing broke.
- **System tests first.** One row in `02-System tests` per path the chain must survive — ticket, mail, transfer, after hours, withheld number, caller hangs up mid-sentence. Make those calls chaotic: mumbling breaks chains that clean calls pass. Fix everything, then delete the runs.
- **Calibrate the judge** on the first five verdicts. If one differs from yours, ask whether a second person seeing only `Pass if` and the transcript would agree with you. Yes → sharpen the criterion. No → the agent really was bad.
- **Per round:** liability cases first, then one per path, then the rest — row order is call order. Run it fully or abort it. Take notes on paper: the grader doesn't hear pauses, tone, or the moment a real caller would hang up. Delete test tickets, run the skill, bump the version.
- **Gate:** stop when Δ flattens and every gate KPI holds, usually after three to four rounds. Measure the working stack, then held-out; after that the system is frozen. A failed held-out case is corrected if ambiguous, or moves into the working stack with a fresh held-out set and one more round.
- **After go-live:** every real call that went wrong becomes a capability case, one per cause. At double-digit calls per week any weekly rate is noise; watch one number instead — the share of callers who hang up themselves, against a threshold set at go-live.

## Limits

- **One call per case outside the liability paths.** Calls are made by hand, so only expensive failures get three calls. A cost decision, not a methodological one.
- **Small N.** 37 cases find failure modes; they don't estimate failure rates.
- **The grader doesn't listen.** Prosody and pacing enter only through handwritten notes.
- **The judge is calibrated against five verdicts**, not a gold-standard dataset.
- **The queue trusts the caller.** Calling a different case than the one shown files the run under the wrong case without an error; the caller's first sentence in the rationale is the only check.

## Sources

- Anthropic, <a href="https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents" target="_blank" rel="noopener noreferrer">Demystifying evals for AI agents</a> — two loops, reference solution, scoring outcome over process, `pass^k`, the twin rule
- Anthropic, <a href="https://platform.claude.com/docs/en/test-and-evaluate/develop-tests" target="_blank" rel="noopener noreferrer">Define success criteria and build evaluations</a> — success criterion, set size, edge cases, grader choice
- Anthropic, <a href="https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks" target="_blank" rel="noopener noreferrer">Mitigate jailbreaks and prompt injections</a> — attack cases, untrusted content in tool results, red teaming before go-live
- Panickssery et al., <a href="https://arxiv.org/abs/2404.13076" target="_blank" rel="noopener noreferrer">LLM Evaluators Recognize and Favor Their Own Generations</a> — why the judge can't be the agent's own model
