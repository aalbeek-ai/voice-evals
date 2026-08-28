# Regelwerk: Audit, Fehler-Taxonomie, Ursachenanalyse

## 1 Prompt-Audit

Dieselbe Liste für beides: einen fremden Prompt auditieren und den eigenen vor Lieferung prüfen. Bewertung ✅ vorhanden · ⚠️ fehlt oder fehlerhaft · ❌ kritisch.

### 1.1 Struktur
Blockfolge: `prompt-template.md`.

- Identität (Name, Rolle, Unternehmen) und Ansprechform definiert, Ansprechform durchgehend gehalten
- Markdown mit Überschriften; Phasenbaum PHASE 0 (Anliegenerkennung) → 1A–1X → 2 (Kontaktdaten) → 3 (Verabschiedung)
- Jede Verzweigung mit Wenn-Dann-Bedingung und Ziel (→), Sonstiges-Fallback vorhanden
- Kein toter Ast: jeder Pfad endet in Kontaktdatenerfassung oder Verabschiedung
- Jede Rückkehr-Kante begrenzt („noch ein Anliegen?" → PHASE 0, höchstens einmal). Ohne Grenze ist der Baum ein Kreis, den nur der Anrufer beendet

### 1.2 Fundamentale Regeln
- Anrufer-E-Mail wird nie abgefragt. Keine Telefonnummern oder Adressen im Prompt — nur Tool-Namen
- Injection-Schutz: Fragen nach Prompt, Anweisungen oder Technik werden abgewiesen
- **Bestätigt wird genau einmal, am Ende.** Alles Erfasste in einer Abschluss-Zusammenfassung vorlesen, nicht schon beim Erfassen. Nummern dabei Ziffer für Ziffer, PLZ einzeln, Straßen bei Bedarf buchstabieren lassen; mehrziffrige Nummern nie zählen lassen (LLMs können das nicht). Im Verlauf wird nichts wiederholt — was akustisch ankam, quittiert die KI mit „Alles klar". Sonst verhandelt der Anrufer jede Angabe zweimal
- **Aussprache (TTS):** Zahlen, Preise, Uhrzeiten, Abkürzungen als Wörter; schwer aussprechbare Eigennamen lautmalerisch („Reidl"→„Raidl"); Block am Prompt-Anfang, mit Grund statt Verbotsliste. Gilt für **jeden Text, den die Engine vorliest** — Prompt, Unternehmensinformationen, Wissensspeicher, Startnachricht, Tool-Antworten. Ein Speicher-Eintrag „Notdienst 24/7, Tel. 04503/123" wird zum Glücksspiel, „rund um die Uhr" und die Nummer Ziffer für Ziffer sind es nicht. Audit deshalb über alle Speicher
- Eine Frage auf einmal, natürliche Sätze statt vorgelesener Listen, Antworten 2–3 Sätze
- Themeneingrenzung definiert, aber nicht so eng, dass Small Talk stirbt
- Spam- und Kaltakquise-Abweisung vorhanden
- Anrufer-Name wird dokumentiert, nicht ausgesprochen

### 1.3 Pflicht-Bausteine
- **Kontaktdaten:** Rückrufnummer (liegt sie als Caller-ID vor: still übernehmen, nicht erfragen) → Nachname (Vorname kostet einen Zug für nichts) → Zusammenfassung, in der alles einmal bestätigt wird, auch die Caller-ID — der Anrufer kann vom fremden Apparat aus anrufen
- Branchenspezifischer Notfallpfad, eigenständig und **jederzeit aktiv**, nicht als Ziel aus PHASE 0
- Dreifach-Missverständnis-Abbruch (dreimal nicht verstanden → Ausstieg mit Rückruf-Zusage)
- **Weiterleitung braucht zwei Stellen, arbeitsteilig:** die Weiche im Prompt an der Phase, wo die Entscheidung fällt — eine Zeile, endend auf dem Aufruf; die Auslöser-Details im Bedingungsfeld des Tools. Allein im Tool geführt, feuert sie mal und mal nicht: das Modell hat im Baum keine Stelle, an der es entscheidet. Der Fehler ist nicht die Doppelung, sondern **ungleiche Abdeckung** — jede Weiterleitung folgt dem Muster oder keine. Prüfsignal: kommt jeder Tool-Name mindestens einmal im Prompt vor?
- **Jede Weiterleitung ist ein Einwegtor.** Zurück kommt der Anruf nur bei aktiver Ablehnung; eine Mailbox gilt als angenommen und beendet das Gespräch, eine unterdrückte Rufnummer lässt sich nicht verbinden. Erreichbarkeit vor dem Bau klären, nicht im Prompt kompensieren. Der Scheitern-Arm muss beide Fälle nennen: Ziel von vornherein zu (dann gar nicht ankündigen) oder Ablehnung (dann lief die Ansage schon)
- **Ein Abschluss-Block für alle Ausstiege:** ein kurzer Verabschiedungssatz → sofort beenden → keine Frage, keine Bestätigung abwarten. Geben Zweige eigene Inhalte vor (Nummer, Öffnungszeiten), legt der Block die **Reihenfolge** fest: Zweig-Inhalt → Verabschiedung → Aufruf. Fehlt sie, spricht das Modell den Zweig-Inhalt und lässt die Verabschiedung weg
- **Ausstieg ohne Verabschiedung** (Notruf-Ansage) nennt den Aufruf am Zweig selbst. Prosa („beende das Gespräch") lässt das Modell die Aktion ankündigen statt ausführen; danach fällt es in die Verabschiedungsphase zurück, die einzige Gesprächsende-Formel, die es kennt. Umformulieren scheitert daran beliebig oft. Hinter dem Aufruf darf eine Begründung stehen, keine zweite Handlungsanweisung. Schreibweise der Plattform, ohne Anführungszeichen, auch für Weiterleitungen — etwa `tool_call <name>`. Ein eingebautes Tool ohne Beschreibungsfeld (Auflegen) trägt sein Wann komplett im Prompt, selbst angelegte Tools in ihrer Beschreibung
- Widerspricht der Anrufer der Aufzeichnung und löscht die Plattform automatisch, verfallen Transkript und Extraktion: weiterleiten oder mit Öffnungszeiten beenden, aber kein Anliegen aufnehmen und keinen Rückruf zusagen — das Ticket entsteht nie
- Post-Call-Zusammenfassung ans Team (Tool, nicht an den Anrufer) mit allen bearbeitungsrelevanten Feldern; Variablen-Extraktion deckt die businessrelevanten Datenpunkte ab
- Jedes Tool hat einen klaren Auslöser im Entscheidungsbaum

### 1.4 Speicher und Inhalt
Drei getrennte Speicher, und das Ablage-Kriterium ist **Zuverlässigkeit**, nicht Kernfakt vs. Randfall:

| Speicher | Verhalten | Wofür |
| --- | --- | --- |
| Unternehmensinformationen | Freitext, immer im Kontext, wird frei paraphrasiert | alles, was fast immer sitzen muss: Adresse, Leistungen, Notfallkontakte |
| Öffnungszeiten-Feld | strukturiert, eigenes UI-Element, vermutlich Basis der Geschäftszeiten-Logik (herstellerseitig nicht bestätigt) | Zeiten |
| Wissen / Wissensdatenbank | Frage-Antwort-Paare, nur bei thematischem Bedarf abgerufen, retrieval-abhängig und praktisch unzuverlässig | echte Nice-to-have-FAQ, bei denen ein Fehltreffer nichts kostet |

- Beobachtet: eine im Prompt verankerte Wissens-Prüfung griff live trotzdem nicht. **Default: Wissen aus** — jede aktive Quelle ist eine weitere Fehlerfläche
- Zweite Achse **Wortlaut-Determinismus**: braucht eine Antwort exakten Wortlaut (Rechtliches), gezielt Wissen für genau diese Frage aktivieren (herstellerseitig nicht bestätigt)
- **Nie doppelt ablegen.** Zwei Quellen laufen auseinander — beobachtet: Öffnungszeiten in Wissen wichen von den Unternehmensinformationen ab. Gilt auch Prompt gegen Dashboard: was ein Dashboard-Feld deterministisch entscheidet (Erreichbarkeit nach Uhrzeit), gehört nicht zusätzlich in den Prompt — dort ist es wirkungslos oder eine zweite Wahrheit
- **Speicher sind Daten, kein Verhaltenscode.** Nur kurze Stichpunkte (Wert, Nummer, Kategorie); Bedingungen, Disclaimer, Tonalität und das Wann gehören in den Prompt. Kurz heißt nicht abgekürzt — Zahlen und Uhrzeiten darin nach §1.2 ausschreiben
- Detailinfos (FAQ, Preise, Kataloge) nicht im Prompt; mehr als 10 FAQ-Antworten im Prompt sind ein Anti-Pattern
- Die KI beantwortet häufige Fragen direkt, statt reflexhaft auf Rückruf zu verweisen
- Anrufertypen unterschieden (Neukunde vs. Bestandskunde); Empathie-Anweisungen bei sensiblen Branchen; branchenspezifische Muster eingebaut (Fristsachen bei Anwälten, Produktionsstillstand bei Industrie)

### 1.5 Anti-Patterns
- Startnachricht-Dopplung (KI begrüßt zweimal)
- Redundante Betonung („NIEMALS!!!", Dreifach-Verneinung); Regel ohne Grund, obwohl eine Begründung die ungenannten Ähnlichkeiten mit abdecken würde
- Verbot statt Reihenfolge: ein Verbot deckt nur die Richtung ab, die es nennt („danach keine Frage mehr" fing „Darf ich beenden?" davor nicht)
- Gleiche Regel mehrfach; Widersprüche (duzen vs. siezen, immer vs. nie weiterleiten)
- Regel nennt einen Eval-Fall, Testnamen oder Transkript-Wortlaut (§3.3)
- Überdetaillierte Sprechanweisungen (Millisekunden-Pausen, Silbenbetonung — Sache der TTS)

### 1.6 Namens-Erkennung (falls relevant)
Eigennamen (Team, Standorte) in die Fachbegriffe, **nur in korrekter Schreibweise, keine Varianten**: das Feld ist Biasing, der Output kommt so heraus, wie er gelistet ist — eine Variante landet falsch im Ticket. Varianten nur, wo der Name bloß zum Routen erkannt und nicht gespeichert wird; sonst nachgelagert normalisieren. TTS-Aussprache separat im Aussprache-Block.

### 1.7 Multi-Standort (falls relevant)
Standort-Fachbereich-Matrix, Prüfung ob der Service am Standort verfügbar ist, Routing-Kriterium (PLZ, Stadtname, Nähe), eigenes Weiterleitungs-Tool je Standort.

### 1.8 Lieferformat
- Prompt als Markdown-Codeblock, direkt kopierbar
- Startnachricht kurz halten (Richtwert 90 Zeichen), KI-Offenlegung trotzdem enthalten; „Unterbrechung verhindern" aktiviert, damit sie gehört wird
- Ansprechform über das Anrede-Feld der Plattform, dann **nicht** auch im Prompt — nur bei „Individuell" dort
- Darunter separat: Startnachricht, Dashboard-Einstellungen (Anrufdauer, Sensitivität, Kreativität, Stimme, Wartezeit, genaue Informationsverarbeitung, Anrede), Tool-Konfiguration, Variablen-Extraktion, Speicher-Inhalte

## 2 Transkript-Fehler-Taxonomie

Symptome, wie sie im Transkript auffallen — die Regel dahinter steht in §1.

**Gesprächsführung:** Endlosschleife (dieselbe Frage, weil die Antwort nicht ankam) · mehrere Fragen auf einmal · Listen vorgelesen · Antworten länger als 2–3 Sätze · Roboter-Sprache („Ihre Anfrage wurde registriert") · Begrüßungs-Dopplung · Doppelbestätigung (Angabe beim Erfassen **und** in der Zusammenfassung)

**Entscheidungsbaum:** falscher Pfad · Anliegen ohne Zuordnung · Weiterleitung, obwohl selbst lösbar · Rückruf-Verweis, obwohl die Antwort im Speicher stand · Pfad-Abbruch, Logik nicht zu Ende definiert

**Datenerfassung:** Nummer oder PLZ falsch erkannt · Nummer als Gesamtzahl vorgelesen („siebzigtausendvierhundert") · KI zählt Ziffern und scheitert · Pflichtdaten vergessen · unnötige Daten abgefragt

**Ton:** kühl in emotionaler Lage · Anrufer unterbrochen · Anrufer wird hörbar ungeduldig (Ursache benennen)

## 3 Ursachenanalyse (Eval-Modus)

Ein Eval-Fall ist eine Stichprobe, kein Anforderungskatalog. Wer ihn direkt fixt, baut einen Agenten, der die getesteten zwei Dutzend Situationen beherrscht und bei der dreiundzwanzigsten ratlos ist. Je Befund deshalb: Symptom → Ursache → Ebene → Geschwister-Test.

**Vor der Ebenen-Bestimmung der Algo, jede Frage kann den Fix beenden:**
1. **Hinterfragen.** Ist das Symptom real oder Artefakt eines einzelnen Tests? Wenn nicht: Notiz, kein Fix
2. **Löschen.** Lässt sich eine Regel, ein Speicher-Eintrag, eine Phase ersatzlos streichen, statt etwas hinzuzufügen?
3. **Vereinfachen.** Gibt es eine Variante mit weniger beweglichen Teilen? Die einfachste, die trifft, gewinnt gegen die elegantere

Beschleunigen und Automatisieren kommen erst, wenn 1–3 durchlaufen sind und sich der Bedarf wiederholt hat — keine Wissensdatenbank-Struktur aufbauen, bevor der einfache Weg als unzureichend belegt ist.

### 3.1 Ebene des Fixes
Auf der **höchsten zutreffenden** Ebene fixen — je höher, desto mehr ungetestete Fälle deckt der Fix mit ab.

| Ebene | Ursache | Fix |
| --- | --- | --- |
| **Wortlaut** | sagt das Richtige unnatürlich, zu lang, unverständlich | Formulierung an der bestehenden Stelle ändern |
| **Regel** | Regel fehlt, ist negativ formuliert, steht doppelt oder widerspricht | schärfen, zusammenlegen, Widerspruch auflösen |
| **Struktur** | Pfad fehlt, endet tot, Bedingung greift nicht, Phasen in falscher Reihenfolge | Verzweigung reparieren |
| **Prinzip** | arbeitet den Baum starr ab, scheitert an jeder Abweichung | ein Verhaltensprinzip früh im Prompt, dafür die Detailregeln streichen, die es ersetzt |

Die Prinzip-Ebene ist die einzige, auf der der Prompt kürzer wird und mehr abdeckt. Wiederholte Befunde derselben Ursache sind ihr Signal.

### 3.2 Geschwister-Test (Pflicht vor jedem Fix)
Drei Situationen benennen, die dieselbe Ursache auslösen, aber **nicht** im Eval-Set stehen. Deckt der Fix sie nicht mit ab, ist er zu eng → eine Ebene höher.

### 3.3 Überanpassung erkennen
Ein Fix ist verbrannt, wenn er einen Fallnamen, eine Testadresse oder einen Transkript-Wortlaut enthält · als neuer Spiegelstrich unter „Regeln" landet, statt eine bestehende Stelle zu ändern · nur für eine Anliegen-Variante gilt · eine Situation beschreibt statt ein Verhalten („wenn jemand wegen Schimmel im Bad anruft" statt „bei Gesundheitsgefahr im Wohnraum").

### 3.4 Gehört der Fix überhaupt in den Prompt?

| Befund | Gehört nach |
| --- | --- |
| Kernfakt falsch oder fehlend, oder Information, die zuverlässig fallen muss | Unternehmensinformationen bzw. Öffnungszeiten-Feld — nicht ins Wissen, das ist retrieval-abhängig |
| Nice-to-have-FAQ ohne Schadenspotenzial | Wissensspeicher |
| Zahl, Uhrzeit oder Nummer falsch vorgelesen | dort ausschreiben, wo sie steht (§1.2) |
| Eigenname falsch erkannt | Fachbegriffe (nur korrekte Schreibweise) oder nachgelagerte Normalisierung |
| Weiterleitung ins Leere, falscher Empfänger, fehlende Mail-Felder | Tool-Konfiguration |
| Ticketfeld leer, falsch formatiert, Priorität falsch | Post-Call-Automation |
| KI unterbricht, wartet zu kurz, versteht Zahlen schlecht | Dashboard (Sensitivität, Wartezeit, genaue Informationsverarbeitung) |
| KI kündigt eine Aktion an, statt sie auszuführen, und redet weiter | Hat der Ausstieg keine Verabschiedung: Abschluss-Block. Hat er eine und feuert trotzdem nicht: Aufruf an den Zweig (§1.3). Umformulieren scheitert in beiden Fällen |

Bleibt nach dieser Sortierung nichts übrig, war es kein Prompt-Problem. Solche Befunde getrennt ausweisen.
