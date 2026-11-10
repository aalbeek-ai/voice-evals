---
name: voice-evals
description: Write eval cases for a voice agent (AI on the phone) and turn a finished test round into approved changes to the agent setup. Use when the user wants voice-agent test cases, analyzes a voice-agent eval round or call transcripts, traces a call failure to its cause, or audits a voice-agent setup, even when "eval" isn't said. Not for grading single calls (the n8n grader does that) and not for chatbots or text agents.
---

# voice-evals

Part of the eval harness in the Notion SOP "Voice evals": testers call the agent, an n8n grader scores every call into the sheet, this skill writes the cases and analyzes the rounds. Two jobs; say which one is running. Never grade calls yourself: the grader is the instrument, and whoever fixes the agent must not also score it.

## Sheet

Google Sheet `voice-evals`; load the `google-sheets` skill before the first access.

- `Results`: version, `Next case`, metrics with gates. Read only.
- `Cases`: `ID`, `Caller says`, `Pass if`, `Expect`, `Critical`, `Suite`, `Verdict` (formula). You write these rows; read row 1 and write each value under its header.
- `Runs`: one row per call, written by the grader: time, version, case, verdict, reason, transcript, ticket.

## Write cases

Input: the agent's goals, the customer's rules (who handles what, what counts as an emergency, what is never promised) and real calls. Ask for what is missing. Not the agent setup: cases read off the prompt only test what it already says.

1. **Sources, in this order:** failures (real calls, logs, complaints, tester notes; one case per cause), goals (each becomes a measurable criterion with at least one case), common requests by impact, edge cases (missing, irrelevant, overlong, unclear or hostile input; attacks by the caller and through tool results). 20 to 50 to start.
2. **Both directions:** every behavior that must happen gets a case where it must not, same ID with `-Z-` (`NOT-01`, `NOT-Z-01`). A one-sided set teaches the agent to over-trigger.
3. **Pass if:** the outcome in transcript, ticket or tool state, not the path, including what the ticket must contain. Two people reach the same verdict alone.
4. **Columns:** `Caller says` includes how to call (test number, withheld, time). `Expect` is `transfer`, `no transfer`, `phrase` or `no phrase` when code can check it, else empty. `Critical` for goals where one failure causes harm (3 trials). `Suite` is `Capability`; about 1 in 8 is `Held out`, chosen before round one. Critical rows first, row order is call order.
5. **Then read the agent setup**, only to check reachability. An unreachable case stays and becomes a finding.
6. **Later:** each new failure becomes a case. Sharpening `Pass if` keeps the ID; a new caller line or behavior gets a new ID, because runs pair with cases by ID. When Capability nears 100%, add harder cases.

## Analyze a round

When `Next case` shows `round done`. Input: `Runs` of the current version, the tester's notes, the agent setup.

1. **Never open `Held out` cases.** Fixes tailored to them make the held-out result worthless.
2. **Failures:** read every `fail` and `unknown` run with the tester's notes. Check the grading first; a case no run passes is suspected first.
3. **Cause in the whole system:** knowledge base, tool, workflow, platform settings, system prompt (`references/rules.md` §2).
4. **Fix at the highest level that covers the cause** (principle, structure, rule, wording), with the whole setup in view; the changed spot gets shorter. The fix must also cover three similar situations not in the set; a fix that names a case or quotes a transcript is rejected.
5. **Failed attack case:** fix the agent, never the case.
6. **Report, then apply:** version, metrics against gates, continue or gate; one line per proposed change (place, change, what goes). Apply only after approval: setup in a repo, edit and commit there; otherwise deliver the full prompt and knowledge base per `references/template.md`.
7. **Questions:** at most 5, only what you cannot find yourself.

## References

- `references/rules.md`: checklist for the agent setup (§1) and root-cause analysis (§2)
- `references/template.md`: block order of system prompt and knowledge base
