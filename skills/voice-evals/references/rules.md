# Rules: checklist and root-cause analysis

What the system prompt, tools, workflows and platform settings are checked against after a round. Platform features differ by provider: test every such rule with a real call.

## 1 Checklist

**Structure**
- Opening message short, with AI disclosure, fixed in the platform settings
- A fallback for "something else", every path has an end, every return edge is bounded
- One phase per concern type; all transfers in one block, including what happens when one fails
- One "End the conversation" block with fixed steps

**Conversation**
- One question per turn, one or two sentences per answer
- Speaking style described concretely (pace, warmth, upset callers); "friendly" is not an instruction
- Never ask for what the system already knows; confirm once, in the closing summary
- No counting or clock math: timeouts as turns, deadlines as fixed terms
- Language switch allowed explicitly if callers may speak another language

**Pronunciation:** everything read aloud is written as it should sound. Times, dates and prices as words; phone numbers digit by digit with commas; house numbers as whole numbers; names via the platform's pronunciation dictionary, else spelled as spoken.

**Anti-patterns**
- The same rule twice, or two rules that contradict
- Emphasis ("NEVER!!!") or a negation instead of a reason; say what the right path does
- Middle dots and dashes; the model copies them and the TTS stumbles. Plain punctuation only
- Example data in quotes; it gets spoken as if real
- Phone numbers and contacts in the prompt; the agent reads them out and an injection can extract them

## 2 Root-cause analysis

- **Before every fix:** question it, delete what can go, then simplify. Add only once the need repeats
- **Levels:** wording (sounds wrong), rule (missing, duplicated, contradictory), structure (path missing or not triggering), principle (follows the tree rigidly). Only at the principle level does the prompt get shorter while covering more
- **Numbered steps** only for a phase that derails; elsewhere they make the agent rigid
- **Not every fix is a prompt fix:** transfer into the void, tool; wrong ticket field, post-call workflow; interruptions or misheard numbers, platform settings
