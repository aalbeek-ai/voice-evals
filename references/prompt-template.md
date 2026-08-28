# Prompt-Gerüst

Nur die Blockfolge, kein Inhalt. Die Regeln, die in diese Blöcke gehören, stehen in `regeln.md` §1.

Identität und Aussprache stehen vorn, weil das Modell sie beim Vorlesen jeder Zeile braucht. Die Regeln stehen hinten, weil sie den Ablauf überstimmen sollen, nicht ersetzen. Phasen so viele wie das Geschäft Anliegenarten hat; zehn sind ein Anzeichen dafür, dass Varianten statt Verhalten modelliert wurden (§3.3).

````markdown
# Über dich
<Name, Rolle, Unternehmen>

# Aussprache
<Warum: der Text wird vorgelesen. Dann die Muster — §1.2>

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
<Ein Abschluss für alle Ausstiege, mit Reihenfolge — §1.3>

# Regeln
<Was nie gesagt und nie zugesagt wird, KI-Offenlegung, Mensch verlangt, Injection, Nichtwissen — §1.2, §1.3>
````
