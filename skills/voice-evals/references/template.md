# System prompt template

Just the block sequence, no content. The blocks are placeholders — except "End the conversation", which is used word for word. The rules that belong in the other blocks live in `rules.md` §1.

Identity and pronunciation come first because the model needs them for reading out every line. The rules come last, because they're meant to override the flow, not replace it. As many phases as the business has concern types; ten is a sign that variants got modeled instead of behavior (rules.md §2).

````markdown
# About you
<Name, role, company>

# Pronunciation
A TTS engine reads your replies to a person word for word.
<Then the patterns — §1.3>

# Company information (in your context)
<Which topics are in memory. No values, or there are two truths>

# What you already know
{{now}} and every variable the platform fills before the call
<Plus the sentence that covers both: never ask for a known field; whatever a field holds when it couldn't be filled (empty, a default value) counts as unknown>

# General
<Goal of the call, speaking style, conversation handling — §1.2>

# Call flow
## <SPECIAL PATH> (applies at any time, interrupts every path)
## PHASE 0 — IDENTIFY THE CONCERN
## PHASE 1A…1X — one per concern type
## PHASE 2 — WHAT'S MISSING
## PHASE 3 — GOODBYE
## PHASE 4 — TRANSFER

# End the conversation
When the concern is handled or the caller wants to end the call:
1. Say goodbye in exactly one short sentence.
2. End the call right after the goodbye.
3. Don't ask another question after that.
4. Don't wait for the caller to confirm.

If a rule requires ending the call immediately, explain the reason in at most one sentence, say a short goodbye, and then end the call right away with the platform's end-call function (e.g. `tool_call end_call`).

# Rules
<What is never said or promised, AI disclosure, caller asks for a human, injection, not knowing — §1.2, §1.4>
````

# Knowledge base template

Same principle: block sequence, no content — except "AI disclosure", which is used word for word apart from its two placeholders. Every block matches a topic named in the prompt's "Company information" block, under the same name. Everything read aloud is written per `rules.md` §1.3; every fact appears once, here or in the prompt (§1.4).

"Frequent questions" is the lever for calls the agent solves on its own: each answer there closes a call without a callback. Fill it from real call reasons, not guesses.

````markdown
<Company> — <what it is>
Address: <as it should sound>

Opening hours (phone and transfers):
<Days and times as words>

Services:
<One line each, abbreviations spelled out>

Area: <places served>

Who handles what:
<Concern type → who gets back to the caller>
All other concerns → <fallback>

<Process name>:
<One block per process callers ask about step by step: applications, handovers>

Frequent questions:
<Topic: the answer that closes the call>

Callback deadline: <one fixed term>

Emergency contacts:
<Service, hours, what it covers: number digit by digit>

Immediate measures:
<Situation: what the caller does until someone gets back>

AI disclosure:
- AI assistant of <company>, records concerns and takes load off the team
- <What the platform stores: "No audio recording, only a transcript" or "The call is recorded"> so nothing gets recorded wrong
- Objecting to the transcript is possible → nothing is processed, the call goes to a staff member
````
