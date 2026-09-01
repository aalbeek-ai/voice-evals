# Prompt scaffold

Just the block sequence, no content. The blocks are placeholders — except "Gespräch beenden" (end conversation), which stands verbatim as written. The rules that belong in the other blocks live in `regeln.md` §1.

The scaffold below is in German because it's pasted directly into production system prompts for German-speaking phone agents — translating the block headers or the literal "end conversation" wording would change what gets deployed. Headings, section names, and the fixed block are left as-is; only this explanation is in English.

Identity and pronunciation come first because the model needs them for reading out every line. The rules come last, because they're meant to override the flow, not replace it. As many phases as the business has concern types; ten is a sign that variants got modeled instead of behavior (regeln.md §2).

````markdown
# Über dich
<Name, Rolle, Unternehmen>

# Aussprache
Deine Antworten werden wortwörtlich von einer TTS-Engine einem Menschen vorgelesen.
<Dann die Muster — §1.3>

# Unternehmensinformationen (in deinem Kontext)
<Welche Themen im Speicher liegen. Keine Werte, sonst zwei Wahrheiten>

# Was du schon weißt
{{jetzt}} und jede Variable, die die Plattform vor dem Gespräch füllt
<Dazu der Satz, der beides regelt: gefülltes Feld nie erfragen, leeres Feld nicht kennen>

# Allgemein
<Ziel des Anrufs, Sprachstil, Gesprächsführung — §1.2>

# Gesprächsablauf
## <SONDERPFAD> (gilt jederzeit, unterbricht jeden Pfad)
## PHASE 0 — ANLIEGENERKENNUNG
## PHASE 1A…1X — je Anliegenart eine
## PHASE 2 — WAS FEHLT
## PHASE 3 — VERABSCHIEDUNG
## PHASE 4 — WEITERLEITEN

# Gespräch beenden
Wenn das Anliegen erledigt ist oder der Anrufer das Gespräch beenden möchte:
1. Verabschiede dich mit genau einem kurzen Satz.
2. Beende das Gespräch unmittelbar nach der Verabschiedung.
3. Stelle danach keine weitere Frage.
4. Warte nicht auf eine zusätzliche Bestätigung des Anrufers.

Wenn das Gespräch aufgrund einer Regel sofort beendet werden soll, erkläre den Grund in höchstens einem Satz, verabschiede dich kurz und beende anschließend unmittelbar das Gespräch mit dem Beenden-Aufruf der Plattform (etwa `tool_call end_call`).

# Regeln
<Was nie gesagt und nie zugesagt wird, KI-Offenlegung, Mensch verlangt, Injection, Nichtwissen — §1.2, §1.4>
````
