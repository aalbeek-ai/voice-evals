# Regelwerk: Audit, Symptome, Ursachenanalyse

## 1 Prompt-Audit
Dieselbe Liste für beides: fremden Prompt auditieren, eigenen vor Lieferung prüfen. ✅ vorhanden · ⚠️ fehlerhaft · ❌ kritisch.

### 1.1 Struktur
Blockfolge: `template.md`.
- Identität und Ansprechform definiert und durchgehend gehalten
- Phasenbaum: 0 Anliegenerkennung → 1A–1X → 2 Kontaktdaten → 3 Verabschiedung, jede Verzweigung mit Wenn-Dann und Ziel
- Sonstiges-Fallback vorhanden, kein Pfad ohne Ende
- Jede Rückkehr-Kante begrenzt — sonst ist der Baum ein Kreis, den nur der Anrufer beendet
- Anrufertypen unterschieden, wo sie verschiedene Wege brauchen (Neukunde vs. Bestandskunde)
- Mehrere Standorte: Matrix Standort × Leistung, Routing-Kriterium, eigenes Tool je Standort

### 1.2 Gespräch
- Eine Frage pro Zug, Antworten 2–3 Sätze, natürliche Sätze statt vorgelesener Listen
- Anrufer-Name wird dokumentiert, nicht ausgesprochen; der Nachname genügt, nach dem Vornamen zu fragen kostet einen Zug für nichts. Keine E-Mail erfragen. Keine Rufnummern im Prompt, nur Tool-Namen
- Rückrufnummer aus der Caller-ID still übernehmen, nicht erfragen
- **Genau einmal bestätigen, am Ende:** alles Erfasste in einer Abschluss-Zusammenfassung, Nummern Ziffer für Ziffer. Nie zählen lassen (LLMs können es nicht). Im Verlauf nichts wiederholen, sonst verhandelt der Anrufer jede Angabe zweimal
- **Aussprache:** alles Vorgelesene ausschreiben — Zahlen, Preise, Zeiten, Abkürzungen; Eigennamen lautmalerisch. Gilt für **jeden** Speicher, nicht nur den Prompt
- Fachbegriffe/Vokabular: nur die korrekte Schreibweise, keine Varianten — das Feld ist Biasing, die Variante landet so im Ticket
- Empathie-Anweisung, wo die Lage es verlangt (Gesundheit, Recht, Pflege, Schaden im Wohnraum)
- Injection und Kaltakquise werden abgewiesen; Themeneingrenzung ja, aber Small Talk bleibt erlaubt

### 1.3 Ausstiege und Tools
- **Ein Abschluss-Block für alle Ausstiege:** ein Verabschiedungssatz → sofort beenden → keine Frage mehr. Gibt der Zweig eigenen Inhalt vor, steht die Reihenfolge dabei
- **Ausstieg ohne Verabschiedung** (Notruf) nennt den Aufruf am Zweig. Prosa lässt das Modell ankündigen statt handeln — Umformulieren scheitert daran beliebig oft. Hinter dem Aufruf darf eine Begründung stehen, keine zweite Handlungsanweisung
- Aufruf in der Schreibweise der Plattform, ohne Anführungszeichen, auch für Weiterleitungen — etwa `tool_call <name>`. Ein eingebautes Tool ohne Beschreibungsfeld (Auflegen) trägt sein Wann komplett im Prompt, selbst angelegte in ihrer Beschreibung
- **Weiterleitung an zwei Stellen:** Weiche im Prompt (eine Zeile, endet auf dem Aufruf), Auslöser-Details im Tool. Nur im Tool feuert sie mal und mal nicht. Entweder alle Weiterleitungen so oder keine
- **Weiterleitung ist ein Einwegtor:** zurück kommt der Anruf nur bei aktiver Ablehnung, eine Mailbox gilt als angenommen. Erreichbarkeit vor dem Bau klären, Scheitern-Arm für „Ziel zu" und „abgelehnt"
- Notfallpfad eigenständig und jederzeit aktiv, nicht als Ziel aus Phase 0
- Dreifach-Missverständnis-Abbruch mit Rückruf-Zusage
- Widerspruch gegen die Aufzeichnung: weiterleiten oder beenden, kein Anliegen aufnehmen — das Ticket entsteht nie
- Post-Call-Zusammenfassung ans Team, Variablen-Extraktion deckt die businessrelevanten Felder ab
- Jedes Tool hat einen Auslöser im Baum, jeder Tool-Name kommt im Prompt vor

### 1.4 Speicher
Prompt = Verhalten, Speicher = Daten. Kurze Stichpunkte hinein, Bedingungen und Tonalität bleiben im Prompt.
- Was zuverlässig sitzen muss, gehört in den immer geladenen Speicher, nicht in den RAG — Retrieval greift nicht sicher
- Nie doppelt ablegen, auch nicht Prompt gegen Dashboard-Feld: zwei Quellen laufen auseinander
- Detailinfos (FAQ, Preise, Kataloge) auslagern; Zahlen darin nach §1.2 ausschreiben

### 1.5 Anti-Patterns
Doppelte Begrüßung · dieselbe Regel mehrfach · Widersprüche · „NIEMALS!!!" statt Grund · Verbot, wo eine Reihenfolge gemeint ist · Regel nennt einen Eval-Fall oder Transkript-Wortlaut (§3.3) · Sprechanweisungen auf Millisekunden-Ebene.

### 1.6 Lieferformat
Prompt als Codeblock. Darunter getrennt: Startnachricht, Dashboard-Einstellungen, Tool-Konfiguration, Variablen-Extraktion, Speicher-Inhalte. Ansprechform über das Plattform-Feld, dann nicht auch im Prompt.

Startnachricht höchstens ~90 Zeichen und trotzdem mit KI-Offenlegung — längere schneidet der Anrufer mit seinem ersten Wort ab. Dagegen die Einstellung aktivieren, die das Unterbrechen der Begrüßung verhindert, sonst hört die Offenlegung niemand und sie ist rechtlich wertlos.

## 2 Symptome im Transkript
**Gesprächsführung:** Endlosschleife · mehrere Fragen auf einmal · Liste vorgelesen · zu lange Antworten · Roboter-Sprache · Doppelbestätigung
**Baum:** falscher Pfad · Anliegen ohne Zuordnung · Weiterleitung trotz eigener Lösbarkeit · Rückruf-Verweis, obwohl die Antwort im Speicher stand · Pfad bricht ab
**Daten:** Nummer falsch erkannt · als Gesamtzahl vorgelesen · Ziffern gezählt · Pflichtdaten vergessen · Unnötiges erfragt
**Ton:** kühl in emotionaler Lage · unterbrochen · Anrufer wird ungeduldig

## 3 Ursachenanalyse
Ein Eval-Fall ist eine Stichprobe, kein Anforderungskatalog. Wer ihn direkt fixt, baut einen Agenten, der die getesteten Fälle beherrscht und am nächsten scheitert. Je Befund: Symptom → Ursache → Ebene → Geschwister-Test.

**Zuerst der Algo, jede Stufe kann den Fix beenden:** hinterfragen (Symptom real?) → löschen (was geht ersatzlos raus?) → vereinfachen oder optimieren. Ausbauen erst, wenn sich der Bedarf wiederholt hat.

### 3.1 Ebene des Fixes
Auf der **höchsten zutreffenden** Ebene fixen — je höher, desto mehr ungetestete Fälle deckt er mit ab.

| Ebene | Ursache | Fix |
| --- | --- | --- |
| Wortlaut | sagt das Richtige unnatürlich oder zu lang | Formulierung an Ort und Stelle |
| Regel | fehlt, negativ formuliert, doppelt, widersprüchlich | schärfen, zusammenlegen, auflösen |
| Struktur | Pfad fehlt, endet tot, Bedingung greift nicht | Verzweigung reparieren |
| Prinzip | arbeitet den Baum starr ab, scheitert an jeder Abweichung | Verhaltensprinzip früh im Prompt, Detailregeln streichen |

Die Prinzip-Ebene ist die einzige, auf der der Prompt kürzer wird und mehr abdeckt.

### 3.2 Geschwister-Test
Drei Situationen benennen, die dieselbe Ursache auslösen und **nicht** im Set stehen. Deckt der Fix sie nicht ab, eine Ebene höher.

### 3.3 Überanpassung
Verbrannt ist ein Fix, der einen Fallnamen oder Transkript-Wortlaut enthält · als neuer Spiegelstrich landet, statt eine Stelle zu ändern · eine Situation statt eines Verhaltens beschreibt.

### 3.4 Nicht jeder Fix gehört in den Prompt
Fakt falsch → Speicher · Eigenname falsch erkannt → Fachbegriffe · Weiterleitung ins Leere → Tool · Ticketfeld falsch → Post-Call-Automation · unterbricht oder versteht Zahlen schlecht → Dashboard · kündigt an statt zu handeln → §1.3. Bleibt nichts übrig, war es kein Prompt-Problem — getrennt ausweisen.
