---
name: voice-evals
description: Write eval cases for a voice agent (AI on the phone) and turn a finished test round into approved changes to the agent setup. Use when the user wants voice-agent test cases, analyzes a voice-agent eval round or call transcripts, traces a call failure to its cause, or audits a voice-agent setup, even when "eval" isn't said. Not for grading single calls (the n8n grader does that) and not for chatbots or text agents.
---

# voice-evals

Part of the eval harness in the Notion SOP "Voice evals": testers call the agent, an n8n grader scores every call into the sheet, this skill writes the cases and analyzes the rounds. Never grade calls yourself: whoever fixes the agent must not also score it.

**Two jobs, each in its own fresh session, with only the inputs listed below.** Say which one is running. Anything else you read (the agent setup while writing cases, earlier analyses, your own notes from a past round) leaks into the next result and biases the measurement. Information that is missing from the inputs is asked from the user, never looked up elsewhere.

## Sheet

Google Sheet `voice-evals`; load the `google-sheets` skill before the first access.

- `Results`: version, `Next case`, metrics with gates. Read only.
- `Cases`: `ID`, `Caller says`, `Pass if`, `Expect`, `Critical`, `Suite`, `Verdict` (formula). Read row 1 and write each value under its header.
- `Runs`: one row per call, written by the grader: time, version, case, verdict, reason, transcript, ticket.

## Write cases

Inputs: the agent's goals, the customer's rules (who handles what, what counts as an emergency, what is never promised), real calls, complaints and tester notes. Not the agent setup and not earlier runs: cases read off the system only test what the system already does, so its blind spots never show up.

1. **Sources, in this order:** failures (one case per cause), goals (each becomes a measurable criterion with at least one case), common requests by impact, edge cases (missing, irrelevant, overlong, unclear or hostile input; attacks by the caller and through tool results). 20 to 50 to start.
2. **Both directions:** every behavior that must happen gets a case where it must not, same ID with `-Z-` (`NOT-01`, `NOT-Z-01`). A one-sided set teaches the agent to over-trigger.
3. **Pass if:** the outcome in transcript, ticket or tool state, not the path, including what the ticket must contain. Two people reach the same verdict alone.
4. **Columns:** `Caller says` includes how to call (test number, withheld, time). `Expect` is `transfer`, `no transfer`, `phrase` or `no phrase` when code can check it, else empty. `Critical` for goals where one failure causes harm (3 trials). `Suite` is `Capability`; the user picks the held-out cases, so you don't know which ones they are. Critical rows first, row order is call order.
5. **Later:** each new failure becomes a case. Sharpening `Pass if` keeps the ID; a new caller line or behavior gets a new ID, because runs pair with cases by ID. When Capability nears 100%, add harder cases.

## Analyze a round

When `Next case` shows `round done`. Inputs: all `Cases` except `Held out`, all `Runs` of the current version, the tester's notes, the agent setup. Not earlier analyses or runs of older versions.

1. **Read every run, passes included.** Failures alone lead to patches for single cases; passes show what already works and catch passes that got lucky or found a loophole.
2. **Check the grading first.** A verdict that looks wrong, or a case no run passes, is reported as a case finding. Never change a case yourself: the case changes only through the user, otherwise the fixer moves the target.
3. **Cause in the whole system:** knowledge base, tool, workflow, platform settings, system prompt, across all runs at once; check the setup against `references/rules.md`.
4. **Fix at the highest level that covers the cause** (principle, structure, rule, wording); the changed spot gets shorter. The fix must also cover three similar situations not in the set; a fix that names a case or quotes a transcript is rejected.
5. **Report, then apply:** version, metrics against gates, continue or gate; one line per proposed change (place, change, what goes); case findings separately. Apply only after approval: setup in a repo, edit and commit there; otherwise deliver the full system prompt per `references/template.md` and the changed knowledge base entries. Keep no notes for the next round.

## References

- `references/rules.md`: checklist for the agent setup
- `references/template.md`: block order of the system prompt
