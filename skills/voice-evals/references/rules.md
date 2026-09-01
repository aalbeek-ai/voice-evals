# Rulebook: checklist and root-cause analysis

What a prompt gets checked against — after a round, to sort findings, and before delivering the new version. Block structure and exact wording live in `template.md`, that's authoritative; here's only what isn't readable there.

## 1 Checklist

### Opening message
At most ~90 characters, with AI disclosure. Fixed text, never translated into the caller's language — keep it short and neutral, the prompt handles the language switch.

### 1.1 Structure
- Identity and form of address defined and held consistently
- A fallback for "something else" exists, no path without an end
- Every return edge is bounded — otherwise the tree is a loop only the caller can end
- Its own "end conversation" block with a fixed step sequence.
- Caller types separated where they need different paths; for multiple locations, a matrix, a routing criterion, and one tool per location
- **One phase per concern type.** A phase that bundles two types gets worked through as a list by the model and asks one type's questions inside the other
- **All transfers in one block**, with the switch for whether a transfer is currently possible and what happens after a failed transfer. Hung off individual phases, both go missing

### 1.2 Conversation
- One question per turn, answers two to three sentences, natural sentences instead of read-out lists
- **Speaking style described in detail** — pace, warmth, directness, behavior under excitement or complaint. "Friendly" is not an instruction
- Never ask for what the system already knows
- Confirmed once, at the end, in a closing summary. Follow-up questions stay allowed where something wasn't heard acoustically — otherwise nothing gets repeated
- Don't have it count digits. Phrase timeouts via turns instead ("transfer after three turns") — approximate, but the most reliable of all the variants
- Caller name gets logged, not spoken aloud; the surname suffices — e.g. confirm with "Thanks, noted."
- Language switching doesn't happen on its own, it must be explicitly allowed: "If the caller speaks another language, switch fully into that language for the rest of the conversation." Pair that with a neutral voice in the dashboard, otherwise the second language sounds accented
- Injection and cold-calling get turned away; staying on topic yes, small talk stays allowed

### 1.3 Pronunciation
Everything read aloud is written the way it should sound — in the prompt and in every knowledge-store entry alike. Examples below are in German because production runs in German; the pattern (spell out for TTS, digit-by-digit for numbers meant to be dictated) applies in any language.
- Times, dates, prices, and abbreviations as words: "neun Uhr" (nine o'clock), "dreizehnter März" (March 13th), "neunundachtzig Euro" (eighty-nine euros), "rund um die Uhr" (around the clock)
- Phone numbers, postal codes, and emergency numbers digit by digit, comma-separated: "null, vier, fünf, fünf, eins" · "eins, eins, zwei"
- House, apartment, and floor numbers as whole numbers instead: "Strandallee hundertsechsundfünfzig" (Strandallee one-hundred-fifty-six), "dritter Stock" (third floor)
- Difficult company and proper names spelled phonetically the way the TTS should say them: "Aalbeek" → "Aal-Beek"
- Email and web addresses spelled out in parts, symbols as words: "info at aalbeek punkt de" · "fonio punkt info"
- If a single spot needs to sound a specific way in the moment (typically: reading out an email address), wrap it in `<speak>` tags: "Sie erreichen Herrn Gemeinhardt unter <speak>mg at Gemeinhardt punkt ag</speak>"

### 1.4 Knowledge store and tools
- **What must sit reliably belongs in the system prompt** — retrieval doesn't grip reliably
- Insert variables everywhere in the prompt via the platform's variable field, don't type them out: more robust, less error-prone.
- **The knowledge-store mandate and the not-known sentence live in the prompt:** company-specific facts only from the knowledge store, and if nothing's there, a fixed sentence ("I don't have that information — a colleague will get back to you") instead of a guess. Without the sentence, the model fills the gap itself
- Never volunteer personal data, property addresses, or staff contacts on its own — only on explicit request and only if they're in the knowledge store
- Phone numbers and contacts belong in tools or the knowledge store, never in the prompt: there the AI reads them aloud and an injection can extract them
- Detail info (FAQ, prices, catalogs) goes in the knowledge store, numbers in it follow §1.3
- Nothing stored twice, not even prompt against dashboard field — two sources drift apart
- Every tool has a trigger in the tree: the branch in the prompt, the trigger detail in the tool

### 1.5 Anti-patterns
- Double greeting
- The same rule stated twice
- Contradictions
- "NEVER!!!" instead of a reason
- A prohibition where a sequence was meant
- A rule that names an eval case or a transcript's exact wording
- Example data in the prompt ("e.g. Mr. Müller, null drei null …") — whatever is in quotes eventually gets spoken by the agent and treated as real. Replace old names, numbers, and prices everywhere, including in examples and old knowledge-store entries

## 2 Root-cause analysis

A prompt that an LLM repairs round after round grows into a case directory and fails on the first case not in it. The only thing that helps is setting the fix one level higher than the finding.

- **Algorithm before every fix:** question it (is the symptom real?) → delete (what comes out with nothing replacing it?) → simplify or optimize. Only extend once the need has repeated
- **Pick the highest level that applies:** wording (sounds wrong) → rule (missing, duplicated, contradictory) → structure (path missing or not triggering) → principle (works the tree rigidly, fails on any deviation). Only at the principle level does the prompt get shorter while covering more
- **Numbered steps only against a phase that derails** — applied everywhere, they make it rigid. Last step points to the next phase
- **Sibling test:** name three situations with the same cause that aren't in the set. If the fix doesn't cover them, go one level higher
- **Overfitting:** a fix that contains a case name or exact transcript wording, gets appended as a new bullet point, or describes a situation instead of a behavior, is burned
- **Not every fix belongs in the prompt:** fact wrong → knowledge store · transfer into the void → tool · wrong ticket field → post-call automation · interrupts or mishears numbers → dashboard · AI disclosure gets cut off → dashboard ("prevent interruption"). Nothing left over → it wasn't a prompt problem
