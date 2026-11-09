---
name: voice-evals
description: Write eval cases for voice agents (AI on the phone) and turn a graded round into fixes for the agent setup. Use when the user wants to write voice-agent eval cases, analyze a voice-agent eval round, review call transcripts, trace a call failure to its root cause, or audit a voice agent system — even when "eval" isn't said. Not for evals of chatbots, text agents, or other LLM apps.
---

# voice-evals

Two jobs on the same set: **write cases** and **analyze a round**. Say which one is running.

Every customer value — prompt version, test numbers, mandatory announcement, turn limit, denylist, agent and judge model, gate KPIs, workflow IDs — lives in `01-Setup` and nowhere else.

Findings come from runs in `04-Runs`, else pasted transcripts, else the checklist in `references/rules.md` §1 alone. The thinner the data, the more findings are checklist, not observation — say so.

## References

`references/rules.md`: checklist for the agent setup (§1) and root-cause analysis (§2). `references/template.md`: block order of system prompt and knowledge base. Read both before judging or writing any part of the setup.

## Data

Tabs: `01-Setup` · `02-System tests` · `03-Cases` · `04-Runs` · `05-Results`. Load the `google-sheets` skill before the first access, if installed.

The grader writes `04-Runs`; you write case rows and `Reference solution`. Only exception: correcting a misjudged verdict — then append `[manually adjusted - YYYY-MM-DD HH:MM]` to `Run`, so it doesn't read as the grader's.

## Cases

Success first, setup last: cases read off the prompt test only what it already says, so the set measures the setup against itself.

Output: rows for `03-Cases`. Read row 1 of `03-Cases` first (`Case`, `Path`, `Twin of`, …) and write each value under its column. Rows with `Path` `emergency`, `dispatch`, or `attack` go on top, everything else below, twins included — the queue calls the rows top-down.

1. **Define success before opening the setup.** Per concern type: what a good call leaves behind in transcript, ticket, or tool state — what the caller got, what the team receives, what must never happen. Sources, best first: real calls and complaints, the business's rules (who handles what, what an emergency is, what a ticket needs), common caller tasks; ask the user for the rest. Missing gate KPIs in `01-Setup` come from here, at least one on whether the call reached its goal.
2. **One case per criterion, a twin per conditional behavior** (transfer, emergency, answer from the knowledge base, end of call): one case where it should happen, one where it shouldn't. Twin ID with `-Z-` (`HAV-01` → `HAV-Z-01`), the original's ID in `Twin of`; only twins fill `Twin of`.
3. **Edge cases get their own cases:** mumbling, several concerns, topic switch, ambiguous input, withheld number, after hours, caller hangs up.
4. **`Path` decides who scores.** The rule grader checks `emergency` (announcement), `dispatch` (successful transfer), and `attack` (no denylist term in the transcript). Any other value names a concern type and goes to the judge, which checks `Pass if`, `Fail if`, `Points 0-2`, `Expected ticket`, and a fixed list of basic errors. `attack` only catches leaked terms; any other outcome of an attack needs a judge path.
5. **Fill every column.** `Context`: `known` by default, since a case should vary only what it tests. `unknown`, `withheld`, or `after hours` only when the behavior under test depends on it. `Calls`: as many as the user can afford by hand; liability cases get the most. `Expected ticket`: the state after the call. `Purpose`: `Capability`. `Held out`: `FALSE` unless deliberately held out.
6. **Grade the outcome, not the path.** `Pass if` names what the call produced, not the order of questions — the agent may find a valid route nobody planned. Phrase it so two reviewers reach the same verdict independently.
7. **Only now read the setup** (prompt, tools, variables, knowledge base, workflows), as a check, not a source. A criterion it can't reach stays as a capability gap and goes to the findings.
8. **Partial credit** in `Points 0-2` for multi-part tasks; liability paths stay binary.

A new real call or complaint whose criterion an existing case covers sharpens that case instead of adding one. Sharpening `Pass if` keeps the ID; if `Caller says` or the behavior under test changes, the row gets a new ID (`INT-Z-01` → `INT-Z-01b`), since runs and Δ pair by ID.

## Analysis

1. **Baseline.** One `Prompt version` per analysis. Never open held-out cases. A case counts only once all its `Calls` are in — if `Calls` is unknown, it stays open; then `pass^k` — one failed run fails the case. `Points 0-2` feeds no rate, so whether a basic error sinks a case is decided by `Pass if` alone. A failed `attack` case always changes the agent setup, never the case.
2. **Collect failures:** transcript and grader rationale per failed case. The rationale is a hint; the finding is in the transcript. Note who hung up, agent or caller.
3. **Reference solution:**
   - Case failed → compare with the reference and name the **first diverging turn**. The cause sits there, not where the call visibly derails.
   - No call passed → suspect the case first. Ambiguous, or a human in the agent's seat couldn't pass it either → fix the case; otherwise it's a finding.
   - Reference no longer possible because the system was changed on purpose (path removed, tool swapped) → the case is outdated: update case and reference instead of bending the system back.
   - First pass with an empty field → write the transcript in.
4. **Cause, not symptom** (`references/rules.md` §2): symptom → cause → fix level → sibling test. One cause, one fix.
5. **Write the fix** with the algorithm from §2: question it, delete, then simplify or optimize — lift a rule one level rather than add one beside it. The spot should get shorter; state old/new word count and justify any growth.
6. **Proposals first.** Findings, one line per fix (level · change · what goes), and two or three sentences of status: version, Δ, continue or gate (Δ flat and every gate KPI holds). Fixes outside the prompt (§2, last point) in a separate list. After approval, apply them: agent setup in a repo → edit and commit there; otherwise deliver the full prompt and knowledge base per `references/template.md`, plus changed tool and platform content, in the chat. Explanations always go in the chat.
7. **Follow-up questions:** at most 5, and only about what you can't find yourself in the agent setup, transcripts, or runs.
