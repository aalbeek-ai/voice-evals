---
name: voice-evals
description: Score and improve voice agent systems for phone AI against an eval set. Always use when the user wants to write eval cases, score an eval round, review call transcripts, trace a failure to its root cause, or audit a voice agent system — even when "eval" isn't said explicitly.
---

# voice-evals — evals for phone voice agents

Two jobs, both against the same set: **write cases** (§ Cases) and **score a round** (§ Scoring). Tell the user in one sentence which one is running.

All customer values — prompt version, test numbers, master data, agent and judge model, gate KPIs, workflow IDs — live in tab `01-Setup`, nowhere else.

The data available only determines where findings come from, never the procedure: runs from `04-Läufe` › pasted-in transcripts › audit checklist alone. The thinner the data, the more findings come from `references/regeln.md` §1 instead of observation — flag that in the result.

## Why the rules exist

The checklists live in `references/regeln.md`, the scaffold in `references/template.md`. Here's just the why, so fixes target the cause instead of working through the symptom list. The § references point into `references/regeln.md`.

1. **STT and TTS have separate failure modes → mirrored rules.** TTS reads aloud (output error: pronunciation), STT listens (input error: recognition) — the same thing, names and numbers, needs opposite treatment. The pronunciation side applies to every text the engine reads out, not just the prompt. §1.3
2. **Instruction density lowers compliance.** Every rule exactly once, details in the knowledge store instead of the prompt. 800 words without duplication beat 400 with. §1.4, §1.5
3. **The prompt describes behavior, not a case directory.** An eval set covers two dozen cases, the agent experiences thousands. A rule that only hits the tested case makes the agent more rigid everywhere else and burns the instruction budget from point 2. §2
4. **Reason instead of emphasis.** "NEVER use ellipses" only works on ellipses; "your responses are read aloud by a TTS engine that can't …" generalizes to every similar character. §1.5
5. **Positive instructions.** "If X → say Y" instead of "never say Z" — LLMs follow negations unreliably.
6. **An exit needs its own block.** End-conversation instructions hung off individual branches don't work — a single "end conversation" block with a fixed step sequence does; wording in `references/template.md`. The check signal is the disconnect reason: did the agent hang up, or the caller?

## Data

One spreadsheet file per customer, five tabs: `01-Setup` · `02-Systemtests` · `03-Fälle` · `04-Läufe` · `05-Auswertung`.

**Call the `google-sheets` skill before the first spreadsheet access** — it holds the tool choice and the pitfalls of the sheets API and loads into the same session, no subagent needed. If it's missing, reading and writing go directly through the sheets tools.

`04-Läufe` is written by the grader, never by you. This skill contributes new case rows and a filled-in `Referenzlösung`. The only exception is re-scoring a misjudged verdict. Then the `Lauf` column gets `[Per Hand nachträglich angepasst - JJJJ-MM-TT HH:MM]` appended — otherwise the next round reads a grader verdict that no longer is one.

## Cases

Input: system prompt, tool descriptions, variables, knowledge store, customer master data. Output: rows for tab `03-Fälle` — read the column order from the customer file's header row, not from memory.

1. **Count the paths.** One trigger per phase, per transfer, per rule in the prompt. That's the population, not imagination.
2. **A twin per trigger.** "One-sided evals create one-sided optimization" — one case where the behavior *should* happen, one where it should not. The twin carries the same number with `-Z-` and the `Zwilling zu` column. No twin, no case.
3. **Edge cases as their own cases:** typos and mumbling, multiple concerns in one call, topic switch mid-conversation, ambiguous input, withheld number, outside business hours, caller hangs up.
4. **`Pfad` controls who scores:** a rule grader checks `Notfall` and `Notdienst` for the announcement or transfer, `Angriff` against a denylist, everything else a judge against `Bestanden wenn`, `Ticket erwartet`, and a fixed list of minor errors. The grader only knows these three special values; the rest are named after the customer's concern types and are interchangeable to it. Attack cases are kept separate — rule grader instead of judge, or the attacker's text would take the judge down with it. `Angriff` only covers where data could leak; anything that measures behavior belongs under `Regeln`.
5. **Every row complete**, or it doesn't take effect: `Codewort` is the spoken recognition word · `Kontext` sets the caller number (`Bekannt`/known · `Unbekannt`/unknown · `Unterdrückt`/withheld · outside business hours) · `Anrufe` the repeat count (3 for liability and attack, otherwise 1) · `Ticket erwartet` the state after the call · `Zweck` starts at `Capability` · `Rückhalte` is a checkbox and stays `FALSE` unless the case is deliberately held out.
6. **Check reachability before the case goes into the set.** Is `Bestanden wenn` reachable with what the agent actually has — knowledge store, master data, tools? Otherwise the case measures itself, not the agent.
7. **Two reviewers, one verdict.** Phrase `Bestanden wenn` / `Durchgefallen wenn` so two people would independently reach the same pass/fail. Anything only you can decide isn't a criterion.
8. **Define partial credit** in `Punkte 0-2` where the task has multiple parts: concern recognized but ticket incomplete beats an instant fail. Stays binary where it's binary (liability, attack).
9. **Assign a codeword**, a different short German bird name per case (Amsel, Möwe, Specht, Fink). Nothing rare, nothing confusable, nothing that appears in the spoken text — and no case ID, which speech recognition falls apart on. Goes in the `Codewort` column, not `Anrufer sagt`: it's test mechanics, not a concern. Add it to the platform's domain-term field before every run, remove it afterward.

**If the user brings a case from real experience** — a real call, a hunch, a complaint — that's the best source there is: it comes from production, not imagination. Don't wave it off — translate it into a complete row and check three things: does an existing case already cover the same trigger (then sharpen that row instead of adding a new one)? Is the criterion phrased observably? Is the twin missing? Then ask back only for the missing columns, don't guess.

## Scoring

1. **Set the baseline.** Exactly **one** prompt version per scoring pass — runs from two versions together measure nothing. Don't open held-out cases; whoever has seen them has burned them. A case only counts as scored once all its `Anrufe` are in — otherwise it stays `offen` and no rate that includes it counts. Then `pass^k` applies to **every** case: a single run with `Bestanden = FALSE` fails it. `Punkte 0-2` feeds into no rate, it's the second dimension alongside passing. Whether a minor error sinks the case is therefore decided solely by `Bestanden wenn` — where tolerance is wanted, it belongs in the criterion, not in the math. The only stricter rule is the gate: liability (`Notfall`, `Notdienst`) and `Angriff` must pass every run, 100% or nothing.
2. **Collect failures.** Per failed case: transcript and grader rationale. The grader's rationale is a hint, not a finding — the finding is in the transcript.
3. **Pull in `Referenzlösung`** where it exists:
   - **Case failed** → lay the failed transcript next to the reference and name the **first diverging turn**. That's where the cause sits, not where the conversation visibly derails.
   - **Reference no longer reachable with today's prompt** (path removed, tool swapped, model changed) → the case is stale. Fix the case and the reference, **don't** bend the prompt to fit it.
   - **Case passes for the first time and the field is empty** → write its transcript into `Referenzlösung`.
4. **Cause, not symptom** — `references/regeln.md` §2. Mandatory per finding: symptom → cause → fix level → sibling test. Multiple cases with the same cause get **one** fix, not several.
5. **Write the fix — run the algorithm from `references/regeln.md` §2 first, don't just cite it.** Before every line: question it (is the symptom real?), then delete (what comes out with nothing replacing it?), only then simplify or optimize — usually optimize here: lift the same rule to a higher level instead of placing a second one next to it. **Success is a spot that gets shorter, not longer** — state old/new word count in the result; if it grows anyway, say why. Every round that adds erodes the compliance it's trying to produce (§ Why 2 and 3).
6. **Proposals first, artifacts after.** Deliver the findings and one line per fix: level · what changes · what comes out for it. Add two to three sentences of status — version, direction versus the previous version, continue or gate. Rates and secondary numbers live in tab `05-Auswertung`; copying them out helps no one, their meaning does.
   After approval comes the artifact: the full prompt (no diff) as a code block per `references/template.md`, changed knowledge-store, tool, and platform content below it. Anyone who's already asked for the prompt as a deliverable skips the approval step.
   **If a repo exists, you only swap the files there and commit.** The file then contains the artifact and nothing else — no heading, no rationale, no source list, no "deliberately left out." What the agent doesn't read doesn't belong in it. Every explanation — what changed and why — goes briefly in the chat, never in the file.
   Flag fixes that don't belong in the prompt (`references/regeln.md` §2, last point) separately.
7. **Follow-up questions**, max 5 — never ask something that's already in the prompt, transcripts, or runs.
