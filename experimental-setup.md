# Experimental Setup

How a voice agent gets measured: what's being measured, what measures it, which controls keep measurement error small, and where the setup hits its limits.

Built against a property management company — emergencies, damage reports, transfers. The mechanics are industry-independent; the path names are not.

## Measurement object

What's measured is **one prompt version**, not "the agent." The version is set identically in three places: the system prompt's frontmatter, the grader's `config` node, and the voice agent platform. Scoring runs from two versions together measures nothing.

The measurement object includes the system prompt, knowledge base, variables, tool descriptions, and the platform's dashboard settings. All of it changes behavior, so all of it belongs under the same version number.

## Instrument

A call is matched to its case via a **spoken codeword**, never via caller ID or time of day. After hangup, the post-call workflow sends the payload to the grader; it searches the normalized transcript for the codeword, attaches the case's criteria, strips the word out, and scores.

**The path decides who scores.** Three of the four paths run without an LLM — a false "pass" there would be a liability incident, not a measurement error:

| Path | Grader | Checks |
| --- | --- | --- |
| `Notfall` (emergency) | Rule | Mandatory announcement, said the required number of times, no further talking after it |
| `Notdienst` (dispatch) | Rule | Transfer demonstrably succeeded (`succeeded` tool line or `disconnectReason: call_transfer`) and no announcement |
| `Angriff` (attack) | Rule | Denylist from `config` doesn't appear in the transcript |
| everything else | Judge | `Bestanden wenn` / `Durchgefallen wenn` (pass-if / fail-if), ticket state, fixed list of minor errors |

`Notfall`, `Notdienst`, and `Angriff` are reserved: hardcoded into the grader's `routing` switch and into the `rule-grader` code's path comparisons, not into `config`. No `Notdienst`/dispatch concept for your agent? Nothing to touch — just put no case with that `Pfad` in `03-Fälle`; the branch only fires when a case actually carries the value. Only renaming a reserved path, or adding a genuinely new rule-graded one, means editing the grader itself, in both places. Every other path name is free text: the switch falls through to the judge for anything not in that reserved set, and the judge reads `Pfad` only as context.

Rule graders never read the ticket. On the three rule-graded paths, `Ticket erwartet` is a note for the human reviewer, not something the grader checks — on the liability path, only conversation behavior counts.

Two details that carry the rule grader:

- **Transfer is measured by state, not by what was said.** The model says "I'll connect you" even when it never called a tool. Only success counts — every transfer has an attempt row in the transcript first, otherwise a failed attempt would read as a passed transfer.
- **Transcript and check terms go through the same normalization before comparison** — lowercased, `ä/ö/ü/ß` folded to `ae/oe/ue/ss`, everything else collapsed to spaces. Speech-to-text doesn't reliably keep umlauts; fold only one side and a denylist word spelled with an `ö` can silently miss a transcript that came back with a plain `o`.

**The judge is never the same model as the agent.** LLMs recognize their own outputs and rate them higher than humans do (<a href="https://arxiv.org/abs/2404.13076" target="_blank" rel="noopener noreferrer">Panickssery et al. 2024</a>). It gets criteria, transcript, `disconnectReason`, and tool calls — never the agent's system prompt, or it scores intent instead of outcome. `unklar` (unclear) is a valid answer; if the API fails, the run ends as `unklar`, never as a silent fail.

## Controls

- **A twin per trigger.** Every case where a behavior *should* happen has one where it should not — same number with `-Z-`. No twin, no case: "One-sided evals create one-sided optimization" (<a href="https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents" target="_blank" rel="noopener noreferrer">Anthropic</a>). The flip is a name check, nothing else: the rule grader tests the case name against `/-Z-/i` and, if it matches, requires the *absence* of the announcement/transfer instead of their presence — no separate `Pfad` value, no `config` entry.
- **Codeword mid-conversation.** Each case gets a short German bird name as its codeword, one per case, in the `Codewort` column. Spoken mid-call, never right after the greeting or at the end — that's where recognition is weakest, so that's where a case could fail on the codeword instead of on the agent (registration mechanics: § Procedure).
- **Held-out set.** A portion of cases is never called and never looked at until the gate is reached. Only this second number shows whether the prompt generalizes instead of overfitting to the set (which cases, and when they're picked: § Data schema).
- **The instrument may change mid-round, the measurement object never.** Allowed to sharpen: `Bestanden wenn`, the `Punkte 0-2` partial-credit scale, judge prompt, grader thresholds. Untouched: system prompt, knowledge, variables, tools, dashboard. Fixing those mid-round describes two different agents under the same version.
- **Re-scoring instead of re-calling.** A sharpened criterion re-scores the affected rows — the transcript is already there. The row gets marked `[Per Hand nachträglich angepasst - JJJJ-MM-TT HH:MM]` (manually adjusted afterward), otherwise the next round reads a grader verdict that no longer is one.
- **Reference solution.** One known-working transcript per case, proof the task is solvable, and a check on the grader: change the grader, and the reference must still pass — if it doesn't, the grader is broken, not the agent. It isn't authored ahead of time: the first time a case passes with an empty `Referenzlösung`, that transcript becomes the reference (§ Procedure).

## Metrics

**`pass^k`, not `pass@k`.** A case only counts as passed if *all* its calls pass — a single failure sinks it. The two diverge fast: in <a href="https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents" target="_blank" rel="noopener noreferrer">Anthropic's own example</a>, the same agent over three trials hits 97% `pass@3` (at least one of three succeeds) against 39% `pass^3` (all three succeed) — `pass@k` climbs toward 100% as k grows, `pass^k` drops toward 0%. The metric itself comes from <a href="https://arxiv.org/abs/2406.12045" target="_blank" rel="noopener noreferrer">Yao et al., τ-bench</a>. For an agent that answers the phone, `pass^k` is the only honest one: the caller doesn't get a second try.

Four rates, each over its own stack:

| Rate | Stack | Expectation |
| --- | --- | --- |
| Pass | Capability, excluding held-out | low, deliberately |
| Δ vs. previous version | same stack | the stop criterion |
| Regression | Regression | 100%, otherwise something broke |
| Held-out | Held-out | only at the gate |

A rate only counts once no case in its stack is still `offen` (open). `Punkte 0-2` (partial credit) doesn't feed into any rate — it's the second dimension alongside pass/fail, not a quarter-pass: a multi-part case that got the concern right but the ticket wrong scores better than a total miss, without softening `Bestanden wenn` itself.

**Two loops.** Every case starts on `Capability` — unproven isn't passed. A case that passes all its calls two rounds in a row becomes `Regression` and drops out of the round; a regression case that fails goes back to `Capability`. The regression run isn't scheduled by calendar but by trigger: before go-live, and never otherwise until then — every pre-launch round already changes the prompt, so a trigger on every prompt change would loop forever. Only after go-live do a prompt change, platform update, or model switch each trigger it on their own.

That's the rule that keeps a hand-run set affordable long-term: round cost tracks the capability stack, not the size of the set — true at 30 cases and at 80.

## Procedure

**Setup.** Create the spreadsheet, import the grader, fill in prompt version, mandatory announcement, turn limit, and denylist in `config`. In the post-call workflow, hang the routing to the grader *after* ticket creation — otherwise the eval measures a call that left no ticket behind.

**Before the first run.** Test the chain end to end, not the behavior: two calls, one without a codeword (must land as `nicht zugeordnet`/unmatched), one with (must hit the right case, have the codeword stripped from the transcript, and show a filled ticket). Delete the rows afterward. Fix every problem found immediately — this isn't a measurement yet. Register every case's codeword as a domain term on the platform before calling, and delete all of them before go-live — otherwise the agent hears bird names in production where none were said.

Then calibrate the judge: review the first five verdicts. If one diverges from your own, the two-person test decides — would a second person who only sees `Bestanden wenn` and the transcript reach the same verdict? Yes → sharpen the criterion. No → leave the row, the agent really was bad.

**Per round.** The full capability stack, in this order: liability first, then one representative per path, then the rest. Run it fully or abort it. Take notes on paper while calling — the grader sees the transcript, not the sound: pauses, tone, the moment a real caller would have hung up. Afterward, delete test tickets, bundle findings, bump the version.

A failed case gets laid against its reference solution and read at the **first diverging turn** — that's where the cause sits, not where the conversation visibly derails. Multiple cases with the same cause become *one* fix. If a case that was never reference-solved fails, first check whether the criterion is reachable at all: broken cases get corrected, the prompt doesn't get bent to fit them. The only exception is `Angriff` — there, a failure always changes the system prompt, never the case.

**Gate.** What it takes to pass is fixed before the first round: binary KPIs, readable from the runs, and at least one of them measures whether the call reached its goal — otherwise the gate only shows that nothing broke. Stop when a round stops moving the needle, realistically after three to four. Then measure twice: working stack, then held-out. After that, the prompt doesn't get touched again. If held-out cases fail, the transcript decides — an ambiguous case gets corrected, a fair case moves into the working stack and another round follows with fresh held-out cases.

**After go-live.** Every real call that went wrong becomes a capability case, one per cause, not one per call. Triggered by the failure, not the calendar: at double-digit call volumes per week, any weekly rate is noise. One watch metric suffices — the share of callers who hang up themselves, with a threshold locked in from the baseline at go-live.

## Data schema

Two tables carry the setup. The grader reads the first and writes the second; only the first is written by hand.

**Cases (`03-Fälle`)** — `Fall` (case) · `Codewort` (codeword) · `Pfad` (path) · `Zwilling zu` (twin of) · `Kontext` (context) · `Anrufer sagt` (caller says) · `Bestanden wenn` (pass if) · `Durchgefallen wenn` (fail if) · `Punkte 0-2` (points) · `Anrufe` (calls) · `Zweck` (purpose: `Capability`/`Regression`) · `Rückhalte` (held-out) · `Ticket erwartet` (ticket expected) · `Referenzlösung` (reference solution)

`Rückhalte` is a checkbox, `FALSE` by default: a human ticks it deliberately, before round one — the skill writes the row but doesn't decide the flag, and no ratio is fixed anywhere in this repo.

**Runs (`04-Läufe`)** — `Lauf` (run) · `Fall` (case) · `Promptversion` (prompt version) · `Bestanden` (passed) · `Punkte` (points) · `Begründung` (rationale) · `Transkript` (transcript) · `Dauer` (duration) · `Züge` (turns) · `Tool-Calls` · `disconnectReason` · `Ticket`

Scoring is pure formula work over these two tables and reaches up to case 40 — anyone needing more cases drags the formulas down.

`Züge` counts only conversation turns; tool-call rows stay out, otherwise every transfer looks two turns longer and the turn limit trips where no one actually asked a follow-up.

## Limits

What this setup **cannot** do — more important for judging the numbers than what it can:

- **One call per case outside liability and attack.** Best practice calls for multiple runs per case, because a single trial isn't a result. Here, calls are made by hand; three calls across the whole set isn't affordable. So it's only run three times where a failure is expensive. That's a cost decision, not a methodological one.
- **Small N.** A hand-run set sits at two to three dozen cases. It finds failure modes; it does not estimate failure rates.
- **The grader doesn't listen.** It reads a transcript. Prosody, pauses, pacing, and the moment a real caller hangs up in frustration only enter the scoring through handwritten notes.
- **The judge is calibrated against five verdicts**, not against a gold-standard dataset with an agreement metric.
- **Speech recognition is part of the measurement.** A case can fail on the codeword instead of on the agent. Hence the system test upfront and the rule to say the word mid-conversation.

## Sources

- Anthropic, <a href="https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents" target="_blank" rel="noopener noreferrer">Demystifying evals for AI agents</a> — two loops, reference solution, scoring outcome over process, `pass^k`, the twin rule
- Anthropic, <a href="https://platform.claude.com/docs/en/test-and-evaluate/develop-tests" target="_blank" rel="noopener noreferrer">Create strong empirical evaluations</a> — success criterion, set size, edge cases, grader choice
- Anthropic, <a href="https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks" target="_blank" rel="noopener noreferrer">Mitigate jailbreaks and prompt injections</a> — attack cases, foreign content as tool output, red teaming before go-live
- Panickssery et al., <a href="https://arxiv.org/abs/2404.13076" target="_blank" rel="noopener noreferrer">LLM Evaluators Recognize and Favor Their Own Generations</a> — why the judge can't be the agent's own model
- Yao et al., <a href="https://arxiv.org/abs/2406.12045" target="_blank" rel="noopener noreferrer">τ-bench</a> — `pass^k` as a reliability measure
- Chen et al., <a href="https://arxiv.org/abs/2107.03374" target="_blank" rel="noopener noreferrer">Evaluating Large Language Models Trained on Code</a> — `pass@k`, its counterpart
