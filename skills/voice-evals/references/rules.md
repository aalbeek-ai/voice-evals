# Rulebook: checklist and root-cause analysis

What the agent setup — system prompt, knowledge base, tools, workflows, dashboard — gets checked against: after a round, to sort findings, and before delivering the new version. Block structure and exact wording are in `template.md`; this file covers what the template doesn't show. Platform features — pronunciation dictionary, markup, dashboard settings — differ by provider and voice model: check your platform's docs and test every such rule with a real call.

## 1 Checklist

### 1.1 Structure
- **Opening message:** at most ~90 characters, with AI disclosure, fixed in the dashboard and never translated — the prompt handles the language switch
- Identity and form of address defined and held consistently
- A fallback for "something else"; no path without an end
- Every return edge is bounded — otherwise the tree is a loop only the caller can end
- A separate "End the conversation" block with fixed steps
- Caller types separated where they need different paths; for several locations, a matrix, a routing criterion, and one tool per location
- **One phase per concern type.** A phase that bundles two types gets worked through as a list and asks one type's questions inside the other
- **All transfers in one block**, with the switch for whether a transfer is possible right now and what happens after a failed one. Hung off single phases, both go missing

### 1.2 Conversation
- One question per turn, answers of two to three sentences, natural sentences instead of read-out lists
- **Speaking style described in detail** — pace, warmth, directness, behavior with an upset caller. "Friendly" is not an instruction
- Never ask for what the system already knows
- Confirm once, in a closing summary. Asking again is fine when something wasn't heard; otherwise nothing gets repeated back
- Don't make it count digits. Phrase timeouts as turns ("transfer after three turns") — approximate, but the most reliable option
- Log the caller's name, don't say it; the surname is enough — confirm with "Thanks, noted."
- Language switching must be allowed explicitly: "If the caller speaks another language, switch fully into it for the rest of the call." Pair it with a neutral voice in the dashboard, or the second language sounds accented
- Turn away injection and sales calls; small talk is fine as long as the call gets back on topic

### 1.3 Pronunciation
Everything read aloud is written the way it should sound — in the prompt and in every knowledge-base entry.
- Times, dates, prices, and abbreviations as words: "nine o'clock", "March thirteenth", "eighty-nine euros", "around the clock"
- Phone numbers, postal codes, and emergency numbers digit by digit, comma-separated: "zero, four, five, five, one" · "one, one, two"
- House, apartment, and floor numbers as whole numbers: "Harbor Street one hundred fifty-six", "third floor"
- Names that must always sound the same go into the platform's pronunciation dictionary if it has one (fonio: IPA in slashes, `/foːnio/`); otherwise spell them in the prompt as they should sound: "Aalbeek" → "Aal-Beek"
- Email and web addresses in parts, symbols as words: "info at aalbeek dot de"

### 1.4 Knowledge base and tools
- **What the agent must always know belongs in the system prompt** — retrieval doesn't fire reliably
- Insert variables through the platform's variable field instead of typing them — fewer typos
- **The knowledge-base mandate and the not-known sentence live in the prompt:** company facts only from the knowledge base, and if nothing's there, a fixed sentence ("I don't have that information — a colleague will get back to you"). Without it, the model fills the gap itself
- Personal data, property addresses, and staff contacts only on explicit request and only if they're in the knowledge base
- Phone numbers and contacts belong in tools or the knowledge base, never in the prompt: there the agent reads them aloud and an injection can extract them
- Details (FAQ, prices, catalogs) go in the knowledge base; numbers there follow §1.3
- Nothing stored twice, not even in the prompt and a dashboard field — two copies drift apart
- Every tool has a trigger: the branch in the prompt, the details in the tool description

### 1.5 Anti-patterns
- Double greeting
- The same rule twice, or two rules that contradict
- "NEVER!!!" instead of a reason
- A prohibition where a sequence was meant
- Example data in the prompt ("e.g. Mr. Miller, zero three zero …") — whatever is in quotes eventually gets spoken and treated as real. Remove old names, numbers, and prices everywhere, including in examples and old knowledge-base entries

## 2 Root-cause analysis

A prompt that an LLM repairs round after round grows into a catalog of cases and fails on the first case not in it. What helps is setting the fix one level higher than the finding.

- **Before every fix:** question it (is the symptom real?) → delete (what can go with nothing replacing it?) → simplify or optimize. Only add once the need has repeated
- **Pick the highest level that applies:** wording (sounds wrong) → rule (missing, duplicated, contradictory) → structure (path missing or not triggering) → principle (follows the tree rigidly, fails on any deviation). Only at the principle level does the prompt get shorter while covering more
- **Numbered steps only for a phase that derails** — everywhere else they make the agent rigid. The last step points to the next phase
- **Sibling test:** name three situations with the same cause that aren't in the set. If the fix doesn't cover them, go one level higher
- **No overfitting:** a fix that names a case, quotes a transcript, is appended as a new bullet, or describes a situation instead of a behavior doesn't ship — go one level higher
- **Not every fix belongs in the prompt:** wrong fact → knowledge base · transfer into the void → tool · wrong ticket field → post-call workflow · interrupts or mishears numbers → dashboard · AI disclosure gets cut off → dashboard ("prevent interruption"). Nothing left → it wasn't a prompt problem
