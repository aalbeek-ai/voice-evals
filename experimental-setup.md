# Experimental Setup

How a voice agent gets scored, which controls keep measurement error small, and where the setup hits its limits. Built for a property management company; the method works for any industry, the example cases don't.

## What's measured

**The whole agent system at one version.** Everything that changes behavior belongs to it: system prompt, knowledge base, variables, tool descriptions, workflows, and the platform's dashboard settings. They share one version number, set identically in the prompt's frontmatter, on the platform, and as `Prompt version` in `01-Setup`. Runs from two versions, scored together, measure nothing.

## Instrument

Calls are matched to cases **by queue, not by anything said in the call**, and not by caller ID or time of day. `01-Setup` shows the next open case: the first `Capability` case in `03-Cases`, not held out, with fewer runs in the current version than its `Calls`. The grader reads the same cell before writing the run, so it still names the case just called. After a botched call, delete its row and the case comes back. A spoken codeword came first and was dropped: a bird name made no sense in a property-management call, so speech recognition often swapped it for a word that did.

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
- **Attack cases** cover direct injection by the caller and indirect injection through tool results; red-team the agent before go-live (<a href="https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks" target="_blank" rel="noopener noreferrer">Anthropic</a>).

**The judge is never the agent's model** — LLMs rate their own outputs higher than humans do (<a href="https://arxiv.org/abs/2404.13076" target="_blank" rel="noopener noreferrer">Panickssery et al. 2024</a>). It gets criteria, transcript, ticket, `disconnectReason`, and tool calls, never the system prompt, or it scores intent instead of outcome. If the data isn't enough, it answers `unclear`; if the API fails, the run ends as `unclear` too, never as a silent fail.

## Controls

- **A twin per trigger.** Every case where a behavior *should* happen has one where it should not: same ID with `-Z-`. "One-sided evals create one-sided optimization" (<a href="https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents" target="_blank" rel="noopener noreferrer">Anthropic</a>). For the rule grader, `-Z-` in the case ID flips the check: announcement and transfer must be absent.
- **Held-out set.** About one case in eight — in the live set, five of 37 — is never called or looked at until the gate. Only this number shows whether the agent generalizes instead of overfitting to the set. `Held out` is a checkbox a human ticks before round one.
- **The instrument may change mid-round, the agent system never.** Allowed to sharpen: `Pass if`, `Points 0-2`, the judge prompt, the turn limit and denylist. Untouched: system prompt, knowledge, variables, tools, workflows, dashboard — changing those describes two agents under one version.
- **Reference solution.** One known-working transcript per case: proof the task is solvable, and a check on the grader — after a grader change the reference must still pass. The first passing transcript of a case becomes its reference.

## Metrics

**`pass^k`, not `pass@k`.** A case passes only if *all* its calls pass. In <a href="https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents" target="_blank" rel="noopener noreferrer">Anthropic's chart</a>, the same agent hits 97% `pass@3` but 39% `pass^3`. An agent on the phone gets one attempt per call, so `pass^k` is the honest one.

| Rate | Stack | Expectation |
| --- | --- | --- |
| Pass | Capability, excluding held-out | low, deliberately |
| Δ vs. previous version | same stack | the stop criterion |
| Regression | Regression | 100%, otherwise something broke |
| Held-out | Held-out | only at the gate |

A rate counts only once no case in its stack is `open`; until then its cell in `05-Results` stays empty. That's by design, not a broken formula: a rate over half-called cases would rise and fall with call order, and Δ against it would be meaningless. The per-case rows below show progress meanwhile. `Points 0-2` feeds no rate: it shows how far a failed case missed, for the human review and the skill. `Turns` counts conversation turns only, so a transfer's tool rows don't trip the turn limit. Δ compares only cases with a result in both versions, so a case added between rounds doesn't count as failed in the previous one. `05-Results` grows with `03-Cases` and reads the previous version from `04-Runs`.

**Two loops.** Every case starts as `Capability`. A case that passes all its calls two rounds in a row becomes `Regression` and leaves the round; a regression case that fails goes back. Round cost tracks the capability stack, not the size of the set. Regression runs go by trigger: once before go-live, then after every change to the system, every platform update, and every model switch.

## Procedure

Setup is in the README; the analysis after each round is in the skill. What neither covers:

- **Gate KPIs** go into `01-Setup` before round one, each with a threshold readable from the runs. At least one measures whether the call reached its goal — otherwise the gate only shows that nothing broke.
- **System tests first.** `02-System tests` check the chain, not the agent: ticket, mail, transfer, after hours, withheld number, caller hangs up mid-sentence. Make them chaotic — mumbling breaks chains that clean calls pass. Fix everything, then delete their runs. Round calls are the measurement; their runs stay.
- **The first five round calls check the grader.** Each must land on the case `01-Setup` showed, and each verdict gets compared with your own. If one differs, ask whether a second person seeing only `Pass if` and the transcript would side with you. Yes → the criterion is clear and the judge misread it: fix the judge prompt. No → your verdict relied on something the criterion doesn't say: sharpen `Pass if`, or accept the judge's verdict.
- **Per round:** liability cases first, then the rest — row order is call order. Run it fully or abort it. Take notes on paper: the grader doesn't hear pauses, tone, or the moment a real caller would hang up. Delete test tickets, run the skill, bump the version.
- **Gate:** stop when Δ flattens and every gate KPI holds, usually after three to four rounds. Measure the working stack, then held-out; after that the system is frozen. A failed held-out case is corrected if ambiguous, or moves into the working stack with a fresh held-out set and one more round.
- **After go-live,** production data takes over from test calls. The main KPI is the share of all calls the agent resolves without a human. Every real call that went wrong becomes a capability case, one per cause.

## Limits

- **More calls per case, more reliable results.** Model outputs vary between runs, so <a href="https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents" target="_blank" rel="noopener noreferrer">Anthropic</a> runs several trials per task. Set `Calls` as high as you can afford to call by hand.
- **The judge is calibrated on five verdicts**, not a gold-standard dataset.

## Sources

- Anthropic, <a href="https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents" target="_blank" rel="noopener noreferrer">Demystifying evals for AI agents</a> — two loops, reference solution, outcome over process, partial credit, `pass^k`, the twin rule, multiple calls per case, production monitoring
- Anthropic, <a href="https://platform.claude.com/docs/en/test-and-evaluate/develop-tests" target="_blank" rel="noopener noreferrer">Define success criteria and build evaluations</a> — measurable success criteria, held-out test set, edge cases, volume over quality, grader choice, a different model as grader
- Anthropic, <a href="https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks" target="_blank" rel="noopener noreferrer">Mitigate jailbreaks and prompt injections</a> — direct and indirect prompt injection, red-teaming before deployment
- Panickssery et al., <a href="https://arxiv.org/abs/2404.13076" target="_blank" rel="noopener noreferrer">LLM Evaluators Recognize and Favor Their Own Generations</a> — LLMs score their own outputs higher, so the judge can't be the agent's model
