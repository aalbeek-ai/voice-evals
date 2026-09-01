# voice-evals

The best AI agents complete only **24% of realistic, long-horizon office tasks** fully autonomously ([TheAgentCompany, Xu et al. 2024, Carnegie Mellon University, arXiv:2412.14161](https://arxiv.org/abs/2412.14161)). The breakage isn't the models — it's evaluation, governance, and integration.

For a voice agent on the phone the gap is wider than with text: background noise, dialects, latency, and a caller who doesn't get a second try. Without measurement, every prompt change is a guess.

This repo is the eval harness I use for that — method, grader, spreadsheet template, and the Claude Code skill that runs the scoring.

## How it works

![How voice-evals works: test call → voice platform → post-call automation → grader → rule grader or judge → runs → voice-evals skill → fix](assets/flow.png)

Liability paths never go to an LLM. A false "pass" there would be a liability incident, not a measurement error.

## What's inside

| File | What |
| --- | --- |
| [versuchsaufbau.md](versuchsaufbau.md) | Measurement object, instrument, controls, metrics, procedure — and the limits |
| [eval-grader.json](eval-grader.json) | The grader as an n8n workflow, importable |
| [skills/voice-evals/](skills/voice-evals/) | The Claude Code skill: write cases, score a round |
| [Spreadsheet template](https://docs.google.com/spreadsheets/d/19SLbwL9aN61PI7MN0dhFoHuvjAXGuYoy4i9WfgAsJXg/edit?usp=sharing) | Cases, runs, and the scoring that computes itself (Google Sheets) |

The skill is `SKILL.md` plus two references: [regeln.md](skills/voice-evals/references/regeln.md) (checklist for voice prompts and root-cause analysis) and [template.md](skills/voice-evals/references/template.md) (block scaffold for a system prompt). Both belong to it and travel with it on install.

## Using it

**Install the skill** — it lives in `skills/voice-evals/` and belongs under `~/.claude/skills/`:

```bash
git clone https://github.com/aalbeek-ai/voice-evals.git
cp -r voice-evals/skills/voice-evals ~/.claude/skills/
```

After that it triggers on its own in Claude Code whenever eval cases, an eval round, or a call transcript come up. If you'd rather point Claude Code at the repo, that works too — the skill sits in the usual place.

**The spreadsheet**: get your own copy via **[copy template](https://docs.google.com/spreadsheets/d/19SLbwL9aN61PI7MN0dhFoHuvjAXGuYoy4i9WfgAsJXg/copy)**. Five tabs: `01-Setup` holds every customer value, `02-Systemtests` checks the line before the first run, `03-Fälle` and `04-Läufe` are the two data tables, `05-Auswertung` computes the four rates itself and stays untouched. `03-Fälle` ships with one example case and its twin — they show the convention and get overwritten.

A copy keeps the tab IDs: `03-Fälle` is `gid=0`, `04-Läufe` is `gid=932118030`. Both feed the grader.

**The grader**: import it into n8n, then set three things — the two `PLATZHALTER_SPREADSHEET_ID` placeholders in `load-case` and `write-run`, the `PLATZHALTER_GID` of the runs tab, and the credentials for Google Sheets and Anthropic. Per-customer values live exclusively in the `config` node. The webhook takes the post-call payload from the telephony platform; the routing to it hangs *after* ticket creation.

## Status

The harness runs against a real voice agent for a property management company. Measured numbers follow once a version clears the gate — until then this is the method, not the result.

Built on a telephony platform with a post-call webhook, n8n, and Google Sheets. The mechanics don't depend on any of them: what counts is codeword matching, path-dependent scoring, and `pass^k`.

## What's next

**What changes in v2?**

- The grader moves from n8n to a plain Python script — one dependency less, easier to audit.
- The remaining German-only pieces (spreadsheet columns, path names, case codewords) get translated, so the whole repo runs in one language.
- Measured numbers replace the "method, not result" placeholder above — pass rate, Δ per round, from a live customer.

## Feedback

Explicitly wanted — especially from people who run voice agents in production themselves.

- Technical, with evidence: [open an issue](https://github.com/aalbeek-ai/voice-evals/issues)
- Anything else: **lasse@aalbeek.de**

Where something isn't backed by evidence, it's marked as an assumption. If a number or a rule here is wrong, I want to know.

## License

[MIT](LICENSE) — use, change, redistribute.
