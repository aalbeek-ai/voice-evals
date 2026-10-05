---
name: voice-evals
description: Write eval cases for voice agents (AI on the phone) and turn a graded round into prompt fixes. Use when the user wants to write voice-agent eval cases, analyze a voice-agent eval round, review call transcripts, trace a call failure to its root cause, or audit a voice agent system — even when "eval" isn't said. Not for evals of chatbots, text agents, or other LLM apps.
---

# voice-evals

Two jobs on the same set: **write cases** and **analyze a round**. Say in one sentence which one is running.

Every customer value — prompt version, test numbers, mandatory announcement, turn limit, denylist, agent and judge model, gate KPIs, workflow IDs — lives in `01-Setup` and nowhere else.

Findings come from runs in `04-Runs`, else pasted transcripts, else the checklist in `references/rules.md` §1 alone. The thinner the data, the more findings are checklist, not observation — say so.

## Why the rules exist

Checklists: `references/rules.md`. Prompt template: `references/template.md`. Fix the cause, not the symptom:

1. **TTS and STT fail in opposite directions.** TTS mispronounces, STT mishears — names and numbers need different handling on each side. Pronunciation rules apply to every text read aloud, not just the prompt. §1.3
2. **Instruction density lowers compliance.** Every rule once; details go to the knowledge base. §1.4, §1.5
3. **The prompt describes behavior, not cases.** The set has dozens of cases, the agent meets thousands. A rule for one tested case makes the agent rigid everywhere else. §2
4. **Reasons, not emphasis.** "NEVER use ellipses" covers ellipses; "a TTS engine reads your replies aloud and can't …" covers every similar character. §1.5
5. **Positive instructions.** "If X, say Y" beats "never say Z" — models follow negations unreliably.
6. **One exit block.** End-conversation rules hung off single branches fail; one block with fixed steps works (`references/template.md`). Check who hung up: agent or caller.

## Data

Tabs: `01-Setup` · `02-System tests` · `03-Cases` · `04-Runs` · `05-Results`. Load the `google-sheets` skill before the first access, if installed.

The grader writes `04-Runs`; you write case rows and `Reference solution`. Only exception: correcting a misjudged verdict — then append `[manually adjusted - YYYY-MM-DD HH:MM]` to `Run`, so it doesn't read as the grader's.

## Cases

Input: the full agent setup — system prompt, tool descriptions, variables, knowledge base, workflows, master data. Output: rows for `03-Cases`. Read row 1 of `03-Cases` first (`Case`, `Path`, `Twin of`, …) and write each value under its column. Liability cases go on top, because the queue calls the rows top-down.

1. **List the triggers:** every phase, transfer, and rule in the prompt, and every branch in the workflows.
2. **A twin per trigger:** one case where the behavior should happen, one where it shouldn't — same ID with `-Z-`, linked in `Twin of`. No twin, no case.
3. **Edge cases get their own cases:** mumbling, several concerns, topic switch, ambiguous input, withheld number, after hours, caller hangs up.
4. **`Path` decides who scores.** The rule grader checks `emergency` (announcement), `dispatch` (successful transfer), and `attack` (no denylist term in the transcript). Any other value names a concern type and goes to the judge, which checks `Pass if`, `Fail if`, `Points 0-2`, `Expected ticket`, and a fixed list of basic errors. `attack` only catches leaked terms; any other outcome of an attack needs a judge path.
5. **Fill every column.** `Context`: `known` · `unknown` · `withheld` · `after hours`. `Calls`: as many as the user can afford by hand — the agent answers differently on each call, so more calls give a surer verdict; liability cases get the most. `Expected ticket`: the state after the call. `Purpose`: `Capability`. `Held out`: `FALSE` unless deliberately held out.
6. **Reachable:** `Pass if` must be achievable with the agent's knowledge, data, and tools — otherwise the case measures itself.
7. **Two reviewers, one verdict:** phrase `Pass if` / `Fail if` so two people would reach the same verdict independently.
8. **Partial credit** in `Points 0-2` for multi-part tasks; liability paths stay binary.

A case from a real call, hunch, or complaint is the best source. Turn it into a row; if an existing case covers the trigger, sharpen that one instead. Check that the criterion is observable and the twin exists. Ask the user only for what you can't fill in yourself.

## Analysis

1. **Baseline.** One `Prompt version` per analysis. Never open held-out cases. A case counts only once all its `Calls` are in; then `pass^k` — one failed run fails the case. `Points 0-2` feeds no rate, so whether a basic error sinks a case is decided by `Pass if` alone. A failed `attack` case always changes the system prompt, never the case.
2. **Collect failures:** transcript and grader rationale per failed case. The rationale is a hint; the finding is in the transcript.
3. **Reference solution:**
   - Case failed → compare with the reference and name the **first diverging turn**. The cause sits there, not where the call visibly derails.
   - Reference no longer reachable (path removed, tool swapped, model changed) → the case is stale: fix case and reference, not the prompt.
   - First pass with an empty field → write the transcript in.
4. **Cause, not symptom** (`references/rules.md` §2): symptom → cause → fix level → sibling test. One cause, one fix.
5. **Write the fix** with the algorithm from §2: question it, delete, then simplify — lift a rule one level rather than add one beside it. The spot should get shorter; state old/new word count and justify any growth.
6. **Proposals first.** Findings, then one line per fix: level · change · what comes out. Status in two or three sentences: version, direction vs. the previous version, continue or gate. Gate = Δ has flattened and every gate KPI in `01-Setup` holds. Don't copy numbers from `05-Results`; explain them.
   After approval: the full prompt (no diff) per `references/template.md`, then changed knowledge-base, tool, and platform content. Skip approval if the user asked for the prompt directly.
   In a repo: replace the files and commit. The file holds the artifact only — no heading, rationale, or sources; explanations go in the chat.
   List fixes outside the prompt (§2, last point) separately.
7. **Follow-up questions:** at most 5, never about what's in the prompt, transcripts, or runs.
