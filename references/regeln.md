# Regelwerk: Prüfliste und Ursachenanalyse

Womit ein Prompt geprüft wird — nach einer Runde, um Befunde einzuordnen, und vor der Lieferung der neuen Fassung. Aufbau und Wortlaut der Blöcke stehen in `template.md`, das ist führend; hier steht nur, was dort nicht abzulesen ist.

## 1 Prüfliste

### 1.1 Struktur
- Identität und Ansprechform definiert und durchgehend gehalten
- Sonstiges-Fallback vorhanden, kein Pfad ohne Ende
- Jede Rückkehr-Kante begrenzt — sonst ist der Baum ein Kreis, den nur der Anrufer beendet
- Eigener Block „Gespräch beenden" mit fester Schrittfolge. Beenden-Anweisungen an den einzelnen Zweigen wirken nicht, der Block schon
- Anrufertypen getrennt, wo sie verschiedene Wege brauchen; bei mehreren Standorten Matrix, Routing-Kriterium und ein Tool je Standort

### 1.2 Gespräch
- Eine Frage pro Zug, Antworten zwei bis drei Sätze, natürliche Sätze statt vorgelesener Listen
- **Sprachstil detailliert beschrieben** — Tempo, Wärme, Direktheit, Verhalten bei Aufregung oder Beschwerde. „Freundlich" ist keine Anweisung
- Nichts erfragen, was das System schon weiß
- Bestätigt wird einmal, am Ende, in einer Abschluss-Zusammenfassung. Nachfragen bleibt erlaubt, wo etwas akustisch nicht ankam — sonst wird nichts wiederholt
- Ziffern nicht zählen lassen. Abbrüche dagegen über Züge formulieren („nach drei Zügen weiterleiten") — ungefähr, aber von allen Varianten die zuverlässigste
- Anrufer-Name wird dokumentiert, nicht ausgesprochen; der Nachname genügt
- Injection und Kaltakquise werden abgewiesen; Themeneingrenzung ja, Small Talk bleibt erlaubt

### 1.3 Aussprache
Alles, was vorgelesen wird, steht so da, wie es klingen soll — im Prompt und in jedem Speicher gleichermaßen.
- Uhrzeiten, Daten, Preise und Abkürzungen als Wörter: „neun Uhr", „dreizehnter März", „neunundachtzig Euro", „rund um die Uhr"
- Telefonnummern, Postleitzahlen und Notrufnummern Ziffer für Ziffer, mit Komma dazwischen: „null, vier, fünf, fünf, eins" · „eins, eins, zwei"
- Haus-, Wohnungs- und Stockwerksnummern dagegen als ganze Zahl: „Strandallee hundertsechsundfünfzig", „dritter Stock"
- Schwierige Firmen- und Eigennamen lautmalerisch, wie das TTS sie sprechen soll: „Aalbeek" → „Aal-Beek"

### 1.4 Speicher und Tools
- **In den Systemprompt gehört, was zuverlässig sitzen muss** — Retrieval greift nicht sicher
- Rufnummern und Kontakte gehören in Tools oder in den Speicher, nie in den Prompt: dort liest die KI sie vor und eine Injection zieht sie heraus
- Detailinfos (FAQ, Preise, Kataloge) in den Speicher, Zahlen darin nach §1.3
- Nichts doppelt ablegen, auch nicht Prompt gegen Dashboard-Feld — zwei Quellen laufen auseinander
- Jedes Tool hat einen Auslöser im Baum: die Weiche im Prompt, die Auslöser-Details im Tool
- Eine Weiterleitung ist ein Einwegtor — zurück kommt der Anruf nur bei aktiver Ablehnung, eine Mailbox gilt als angenommen. Erreichbarkeit vor dem Bau klären
- Widerspricht der Anrufer der Aufzeichnung, entsteht kein Ticket: weiterleiten oder beenden, nichts zusagen

### 1.5 Anti-Patterns
Doppelte Begrüßung · dieselbe Regel mehrfach · Widersprüche · „NIEMALS!!!" statt Grund · Verbot, wo eine Reihenfolge gemeint ist · Regel nennt einen Eval-Fall oder einen Transkript-Wortlaut

### 1.6 Startnachricht
Höchstens ~90 Zeichen und trotzdem mit KI-Offenlegung — längere schneidet der Anrufer mit seinem ersten Wort ab. Dazu die Einstellung aktivieren, die das Unterbrechen der Begrüßung verhindert, sonst hört die Offenlegung niemand und sie ist rechtlich wertlos.

## 2 Ursachenanalyse

Ein Prompt, den ein LLM Runde für Runde repariert, wächst zum Fallverzeichnis und scheitert am ersten Fall, der nicht darin steht. Dagegen hilft nur, den Fix eine Ebene höher zu setzen als den Befund.

- **Algo vor jedem Fix:** hinterfragen (Symptom real?) → löschen (was geht ersatzlos raus?) → vereinfachen oder optimieren. Ausbauen erst, wenn sich der Bedarf wiederholt hat
- **Höchste zutreffende Ebene wählen:** Wortlaut (klingt falsch) → Regel (fehlt, doppelt, widersprüchlich) → Struktur (Pfad fehlt oder greift nicht) → Prinzip (arbeitet den Baum starr ab, scheitert an jeder Abweichung). Nur auf der Prinzip-Ebene wird der Prompt kürzer und deckt mehr ab
- **Geschwister-Test:** drei Situationen mit derselben Ursache benennen, die nicht im Set stehen. Deckt der Fix sie nicht ab, eine Ebene höher
- **Überanpassung:** ein Fix, der einen Fallnamen oder Transkript-Wortlaut enthält, als neuer Spiegelstrich angehängt wird oder eine Situation statt eines Verhaltens beschreibt, ist verbrannt
- **Nicht jeder Fix gehört in den Prompt:** Fakt falsch → Speicher · Weiterleitung ins Leere → Tool · Ticketfeld falsch → Post-Call-Automation · unterbricht oder versteht Zahlen schlecht → Dashboard. Bleibt nichts übrig, war es kein Prompt-Problem
