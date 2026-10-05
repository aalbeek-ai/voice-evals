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
<Plus the sentence that covers both: never ask for a filled field, don't know an empty one>

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
