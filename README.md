# voice-evals

**88% of AI pilots never reach production** — for every 33 proof-of-concepts started, four go live (<a href="https://www.cio.com/article/3850763/88-of-ai-pilots-fail-to-reach-production-but-thats-not-all-on-it.html" target="_blank" rel="noopener noreferrer">IDC/Lenovo, March 2025</a>). The models aren't the problem; evaluation, governance, and integration are.

For a voice agent the gap is wider than with text: background noise, dialects, latency, and one attempt per call with no retry. Without measurement, every prompt change is a guess.

This repo is the eval harness I use for that: method, grader, spreadsheet template, and a Claude Code skill.

## How it works

The spreadsheet shows which case to call next. The grader assigns the call to that case, scores it, and writes one row per call. The skill reads all runs of a round, finds the cause of each failure, and writes the prompt fixes.

![How voice-evals works: test call → call ends → grader assigns the next open case → the path decides between rule grader, judge, or unmatched → 04-Runs → voice-evals skill → prompt fix and new version, then the next test call](assets/flow.png)

The liability paths — emergency, dispatch, attack — are scored by fixed rules and never go to an LLM; everything else goes to a judge model. A false "pass" on one of these three would cause real damage, not just a measurement error.

## What's inside

| File | What |
| --- | --- |
| [experimental-setup.md](experimental-setup.md) | The method: what gets measured, how, and where it falls short |
| [SKILL.md](skills/voice-evals/SKILL.md) | The Claude Code skill: writes cases and turns a scored round into prompt fixes |
| [rules.md](skills/voice-evals/references/rules.md) | Skill reference: prompt checklist and root-cause analysis |
| [template.md](skills/voice-evals/references/template.md) | Skill reference: system prompt template, block by block |
| <a href="https://docs.google.com/spreadsheets/d/19SLbwL9aN61PI7MN0dhFoHuvjAXGuYoy4i9WfgAsJXg/edit?usp=sharing" target="_blank" rel="noopener noreferrer">Voice-Evals — Template</a> | Google Sheets: setup, system tests, cases, runs, and results |
| [eval-grader.json](eval-grader.json) | The grader as an importable n8n workflow |

## Using it

1. **Skill.** Install it, and it triggers in Claude Code whenever eval cases, a round, or a call transcript come up:

   ```bash
   git clone https://github.com/aalbeek-ai/voice-evals.git
   cp -r voice-evals/skills/voice-evals ~/.claude/skills/
   ```

2. **Spreadsheet.** <a href="https://docs.google.com/spreadsheets/d/19SLbwL9aN61PI7MN0dhFoHuvjAXGuYoy4i9WfgAsJXg/copy" target="_blank" rel="noopener noreferrer">Copy the template</a> and fill in the `Value` column of `01-Setup`; the `Note` column says who reads each value. The example rows in `02-System tests` and `03-Cases` show the format — overwrite them. To have the skill write your cases, give Claude Code your full agent setup — system prompt, knowledge base, variables, transfer rules, workflows — and ask for eval cases. `04-Runs` is written by the grader, `05-Results` updates automatically.

3. **Grader.** Import [eval-grader.json](eval-grader.json) into n8n. In `load-setup`, `load-cases`, and `write-run`, replace `YOUR_SPREADSHEET_ID` with your copy's ID (the part of its URL between `/d/` and `/edit`). Add three credentials: Google Sheets, Anthropic, and header auth on `call-details` (any header name and secret). The judge runs on Claude Sonnet 5 — if your agent does too, pick another model in `judge`. Activate the workflow.

4. **Post-call workflow.** For test numbers only, POST to `call-details` after the ticket is created, with the header from step 3. Send `transcript` as text, one `role: text` line per turn, and `ticket` as an object; the grader ignores everything else.

5. **Call.** `01-Setup` shows the next case and what to say. Call, hang up, and the row appears in `04-Runs`. When `01-Setup` shows `round done`, ask Claude Code to analyze the round: the skill finds the cause of each failure and writes the prompt fixes.

For the skill to read the spreadsheet itself, Claude Code needs a Google Sheets MCP server, and its Google Cloud project must be enrolled in the Workspace Developer Preview Program — otherwise it authenticates but returns no data. Without one, paste the runs into the chat.

## Status

Running live at a property management company. Pass rates get published once there are solid numbers. If there's demand, it becomes a product.

## Feedback

Especially welcome from anyone running voice agents in production:

- Technical, with evidence: <a href="https://github.com/aalbeek-ai/voice-evals/issues" target="_blank" rel="noopener noreferrer">open an issue</a>
- Anything else: **kresse@aalbeek.de**

If a number or a rule here is wrong, I want to know.

## License

[MIT](LICENSE) — use, change, redistribute.
