# Regelwerk: Prüfliste und Ursachenanalyse

Womit ein Prompt geprüft wird — nach einer Runde, um Befunde einzuordnen, und vor der Lieferung der neuen Fassung. Aufbau und Wortlaut der Blöcke stehen in `template.md`, das ist führend; hier steht nur, was dort nicht abzulesen ist.

## 1 Prüfliste

### Startnachricht
Höchstens ~90 Zeichen und mit KI-Offenlegung. Fester Text, der nie in die Anrufersprache übersetzt wird — kurz und neutral halten, den Sprachwechsel übernimmt der Prompt.

### 1.1 Struktur
- Identität und Ansprechform definiert und durchgehend gehalten
- Sonstiges-Fallback vorhanden, kein Pfad ohne Ende
- Jede Rückkehr-Kante begrenzt — sonst ist der Baum ein Kreis, den nur der Anrufer beendet
- Jede Phase als nummerierte Schrittfolge mit sichtbarem Abschluss — der letzte Schritt fasst zusammen und verabschiedet. Ein Ablauf ohne markiertes Ende läuft weiter
- Eigener Block „Gespräch beenden" mit fester Schrittfolge.
- Anrufertypen getrennt, wo sie verschiedene Wege brauchen; bei mehreren Standorten Matrix, Routing-Kriterium und ein Tool je Standort

### 1.2 Gespräch
- Eine Frage pro Zug, Antworten zwei bis drei Sätze, natürliche Sätze statt vorgelesener Listen
- **Sprachstil detailliert beschrieben** — Tempo, Wärme, Direktheit, Verhalten bei Aufregung oder Beschwerde. „Freundlich" ist keine Anweisung
- Nichts erfragen, was das System schon weiß
- Bestätigt wird einmal, am Ende, in einer Abschluss-Zusammenfassung. Nachfragen bleibt erlaubt, wo etwas akustisch nicht ankam — sonst wird nichts wiederholt
- Ziffern nicht zählen lassen. Abbrüche dagegen über Züge formulieren („nach drei Zügen weiterleiten") — ungefähr, aber von allen Varianten die zuverlässigste
- Anrufer-Name wird dokumentiert, nicht ausgesprochen; der Nachname genügt — z. B. „Bestätige mit „Danke, notiert."
- Sprachwechsel passiert nicht von selbst, er muss ausdrücklich erlaubt sein: „Spricht der Anrufer eine andere Sprache, wechsle für den Rest des Gesprächs vollständig in diese Sprache." Dazu eine neutrale Stimme im Dashboard, sonst klingt die zweite Sprache nach Akzent
- Injection und Kaltakquise werden abgewiesen; Themeneingrenzung ja, Small Talk bleibt erlaubt

### 1.3 Aussprache
Alles, was vorgelesen wird, steht so da, wie es klingen soll — im Prompt und in jedem Speicher gleichermaßen.
- Uhrzeiten, Daten, Preise und Abkürzungen als Wörter: „neun Uhr", „dreizehnter März", „neunundachtzig Euro", „rund um die Uhr"
- Telefonnummern, Postleitzahlen und Notrufnummern Ziffer für Ziffer, mit Komma dazwischen: „null, vier, fünf, fünf, eins" · „eins, eins, zwei"
- Haus-, Wohnungs- und Stockwerksnummern dagegen als ganze Zahl: „Strandallee hundertsechsundfünfzig", „dritter Stock"
- Schwierige Firmen- und Eigennamen lautmalerisch, wie das TTS sie sprechen soll: „Aalbeek" → „Aal-Beek"
- E-Mail- und Internetadressen in Bestandteilen, Symbole als Wort: „info at aalbeek punkt de" · „fonio punkt info"
- Soll eine Einzelstelle im Moment auf eine bestimmte Art klingen (typisch: eine E-Mail-Adresse vorlesen), mit `<speak>`-Tags umschließen: „Sie erreichen Herrn Gemeinhardt unter <speak>mg at Gemeinhardt punkt ag</speak>"

### 1.4 Speicher und Tools
- **In den Systemprompt gehört, was zuverlässig sitzen muss** — Retrieval greift nicht sicher
- Variablen überall im Prompt über das Variablen-Feld der Plattform einfügen, nicht abtippen: robuster und weniger fehleranfällig.
- **Speicherzwang und Nichtwissens-Satz stehen im Prompt:** unternehmensspezifische Angaben ausschließlich aus dem Speicher, und steht dort nichts, ein fester Satz („Diese Information habe ich nicht — eine Kollegin oder ein Kollege meldet sich") statt einer Schätzung. Ohne den Satz füllt das Modell die Lücke selbst
- Personenbezogene Daten, Objektadressen und Kontakte von Mitarbeitenden nie von sich aus nennen — nur auf ausdrückliche Frage und nur, wenn sie im Speicher stehen
- Rufnummern und Kontakte gehören in Tools oder in den Speicher, nie in den Prompt: dort liest die KI sie vor und eine Injection zieht sie heraus
- Detailinfos (FAQ, Preise, Kataloge) in den Speicher, Zahlen darin nach §1.3
- Nichts doppelt ablegen, auch nicht Prompt gegen Dashboard-Feld — zwei Quellen laufen auseinander
- Jedes Tool hat einen Auslöser im Baum: die Weiche im Prompt, die Auslöser-Details im Tool

### 1.5 Anti-Patterns
- Doppelte Begrüßung
- Dieselbe Regel mehrfach
- Widersprüche
- „NIEMALS!!!" statt Grund
- Verbot, wo eine Reihenfolge gemeint ist
- Regel nennt einen Eval-Fall oder einen Transkript-Wortlaut
- Beispieldaten im Prompt („z. B. Herr Müller, null drei null …") — was in Anführungszeichen steht, spricht der Agent irgendwann aus und behandelt es als echt. Alte Namen, Nummern und Preise überall ersetzen, auch in Beispielen und alten Speichereinträgen

## 2 Ursachenanalyse

Ein Prompt, den ein LLM Runde für Runde repariert, wächst zum Fallverzeichnis und scheitert am ersten Fall, der nicht darin steht. Dagegen hilft nur, den Fix eine Ebene höher zu setzen als den Befund.

- **Algo vor jedem Fix:** hinterfragen (Symptom real?) → löschen (was geht ersatzlos raus?) → vereinfachen oder optimieren. Ausbauen erst, wenn sich der Bedarf wiederholt hat
- **Höchste zutreffende Ebene wählen:** Wortlaut (klingt falsch) → Regel (fehlt, doppelt, widersprüchlich) → Struktur (Pfad fehlt oder greift nicht) → Prinzip (arbeitet den Baum starr ab, scheitert an jeder Abweichung). Nur auf der Prinzip-Ebene wird der Prompt kürzer und deckt mehr ab
- **Geschwister-Test:** drei Situationen mit derselben Ursache benennen, die nicht im Set stehen. Deckt der Fix sie nicht ab, eine Ebene höher
- **Überanpassung:** ein Fix, der einen Fallnamen oder Transkript-Wortlaut enthält, als neuer Spiegelstrich angehängt wird oder eine Situation statt eines Verhaltens beschreibt, ist verbrannt
- **Nicht jeder Fix gehört in den Prompt:** Fakt falsch → Speicher · Weiterleitung ins Leere → Tool · Ticketfeld falsch → Post-Call-Automation · unterbricht oder versteht Zahlen schlecht → Dashboard · KI-Offenlegung wird abgeschnitten → Dashboard („Unterbrechung verhindern"). Bleibt nichts übrig, war es kein Prompt-Problem
