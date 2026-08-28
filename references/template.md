# Prompt-Gerüst

Nur die Blockfolge, kein Inhalt. Die Blöcke sind Platzhalter — bis auf „Gespräch beenden", der steht wörtlich so. Die Regeln, die in die übrigen Blöcke gehören, stehen in `regeln.md` §1.

Identität und Aussprache stehen vorn, weil das Modell sie beim Vorlesen jeder Zeile braucht. Die Regeln stehen hinten, weil sie den Ablauf überstimmen sollen, nicht ersetzen. Phasen so viele wie das Geschäft Anliegenarten hat; zehn sind ein Anzeichen dafür, dass Varianten statt Verhalten modelliert wurden (regeln.md §2).

````markdown
# Über dich
<Name, Rolle, Unternehmen>

# Aussprache
<Warum: der Text wird vorgelesen. Dann die Muster — §1.3>

# Unternehmensinformationen (in deinem Kontext)
<Welche Themen im Speicher liegen. Keine Werte, sonst zwei Wahrheiten>

# Allgemein
<Ziel des Anrufs, Sprachstil, Gesprächsführung — §1.2>

# Aktueller Wochentag und Uhrzeit
{{jetzt}}

# Gesprächsablauf
## PHASE 0 — ANLIEGENERKENNUNG
## <SONDERPFAD> (gilt jederzeit, unterbricht jeden Pfad)
## PHASE 1A…1X — je Anliegenart eine
## PHASE 2 — KONTAKTDATEN
## PHASE 3 — VERABSCHIEDUNG

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
