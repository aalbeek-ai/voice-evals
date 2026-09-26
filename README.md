# voice-evals

**88% of AI pilots never reach production** — for every 33 proof-of-concepts started, four go live (<a href="https://www.cio.com/article/3850763/88-of-ai-pilots-fail-to-reach-production-but-thats-not-all-on-it.html" target="_blank" rel="noopener noreferrer">IDC/Lenovo, March 2025</a>). The breakage isn't the models — it's evaluation, governance, and integration.

For a voice agent the gap is wider than with text: background noise, dialects, latency, and one attempt per call with no retry. Without measurement, every prompt change is a guess.

This repo is the eval harness I use for that — method, grader, spreadsheet template, and the Claude Code skill that writes cases and turns a graded round's failures into a fix at the root cause.

## How it works

The spreadsheet shows which case to call next. The grader assigns the call to that case, scores it by path — liability paths through fixed rules, everything else through a judge model — and writes one row per call. The skill reads the round, traces the failures to their cause, and ships one fix per cause.

![How voice-evals works: test call → call ends → grader assigns the next open case → path decides rule grader or judge or unmatched → runs → voice-evals skill → fix, looping back to the next test call](assets/flow.png)

Liability paths (emergency, dispatch, attack) never go to an LLM. A false "pass" there would be a liability incident, not a measurement error.

## What's inside

| File | What |
| --- | --- |
| [experimental-setup.md](experimental-setup.md) | Measurement object, instrument, controls, metrics, procedure — and the limits |
| [eval-grader.json](eval-grader.json) | The grader as an n8n workflow, importable — eight nodes, one of them code |
| [skills/voice-evals/](skills/voice-evals/) | The Claude Code skill: write cases, root-cause a graded round into one fix per cause |
| <a href="https://docs.google.com/spreadsheets/d/19SLbwL9aN61PI7MN0dhFoHuvjAXGuYoy4i9WfgAsJXg/edit?usp=sharing" target="_blank" rel="noopener noreferrer">Spreadsheet template</a> | Cases, runs, and the scoring that computes itself (Google Sheets) |

The skill is `SKILL.md` plus two references: [rules.md](skills/voice-evals/references/rules.md) (checklist for voice agent systems and root-cause analysis) and [template.md](skills/voice-evals/references/template.md) (system prompt template, block by block). Both belong to it and travel with it on install.

## Using it

**Install the skill**:

```bash
git clone https://github.com/aalbeek-ai/voice-evals.git
cp -r voice-evals/skills/voice-evals ~/.claude/skills/
```

It then triggers on its own in Claude Code whenever eval cases, an eval round, or a call transcript come up.

**The spreadsheet**: get your own copy via **<a href="https://docs.google.com/spreadsheets/d/19SLbwL9aN61PI7MN0dhFoHuvjAXGuYoy4i9WfgAsJXg/copy" target="_blank" rel="noopener noreferrer">copy template</a>**. Five tabs: `01-Setup` holds every customer value, `02-System tests` tests the full chain end to end before the first real run — `03-Cases` and `04-Runs` are the two data tables, `05-Results` pulls all of it together and computes itself — nothing in it gets filled in by hand. `02-System tests` and `03-Cases` each ship with two example rows — a case and its twin, a matched and an unmatched test call. They show the convention and get overwritten. The top of `01-Setup` shows which case to call next and what the caller says.

For the skill to read and write that spreadsheet, Claude Code needs a Google Sheets MCP server, and the underlying Google Cloud project needs to be enrolled in the Workspace Developer Preview Program — without it, the MCP authenticates but returns no data.

**The grader**: import [eval-grader.json](eval-grader.json) into n8n, point `load-setup`, `load-cases` and `write-run` at your copy, set the Google Sheets and Anthropic credentials and a header-auth credential on the `call-details` webhook. Everything customer-specific comes from `01-Setup`; the grader itself has no settings. The post-call workflow needs to call the grader's `call-details` webhook *after* ticket creation, for test numbers only, with the ticket added as `ticket` — before it, the ticket isn't in the payload yet and nothing the post-call workflow produced enters the scoring. The grader needs at least the transcript, as a list of turns or as plain text with one `role: text` line each; `disconnectReason`, `duration`, and a ticket make the grading sharper.

## Status

The harness runs against a real voice agent for a property management company. Numbers get published once a version clears the gate.

Built on <a href="https://fonio.ai" target="_blank" rel="noopener noreferrer">fonio</a>, n8n, and Google Sheets. The mechanics don't depend on any of them: what counts is case assignment that doesn't depend on speech recognition, path-dependent scoring, and `pass^k`.

## What's next

**v2** is in its first round:

- Cases are matched through a queue in the spreadsheet instead of a spoken codeword.
- The grader shrank from twelve nodes to eight; every customer value moved into `01-Setup`.
- Spreadsheet columns, tab names, and path names are in English, so the whole repo runs in one language. Case content stays in the agent's language.

Still open: measured pass rate and Δ per round, published once a version clears the gate.

A Python grader was considered and dropped: the voice platform only pushes calls via webhook and offers no API to fetch them afterward, so Python would need its own hosted endpoint — more to run than the n8n workflow it replaces.

## Feedback

Explicitly wanted — especially from people who run voice agents in production themselves.

- Technical, with evidence: <a href="https://github.com/aalbeek-ai/voice-evals/issues" target="_blank" rel="noopener noreferrer">open an issue</a>
- Anything else: **kresse@aalbeek.de**

Where something isn't backed by evidence, it's marked as an assumption. If a number or a rule here is wrong, I want to know.

## License

[MIT](LICENSE) — use, change, redistribute.
