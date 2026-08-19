# fonio-Regelwerk: Audit, Fehler-Taxonomie, Ursachenanalyse

## 1 Prompt-Audit

Dieselbe Liste für beides: einen fremden Prompt auditieren und den eigenen vor Lieferung prüfen. Bewertung ✅ vorhanden · ⚠️ fehlt oder fehlerhaft · ❌ kritisch.

### 1.1 Struktur
- Identität: Name, Rolle, Unternehmen klar definiert
- Ansprechform (Du/Sie) definiert und durchgehend eingehalten
- Markdown mit Überschriften strukturiert
- Phasen-Entscheidungsbaum: PHASE 0 (Anliegenerkennung) → 1A–1X (Anliegen) → 2 (Kontaktdaten) → 3 (Verabschiedung)
- Jede Verzweigung mit klarer Wenn-Dann-Bedingung und Ziel (→ Pfeile)
- Sonstiges-Fallback (PHASE 1X) vorhanden
- Jeder Pfad endet in Kontaktdaten-Erfassung oder Verabschiedung — kein toter Ast
- Definierte Verabschiedung

### 1.2 Fundamentale Regeln
- Anrufer-E-Mail wird NIE abgefragt
- Keine Telefonnummern/E-Mail-Adressen im Prompt — nur Tool-Namen
- Prompt-Schutz gegen Injection (Anrufer versucht Prompt auszulesen)
- Nummern, die der Anrufer nennt: Ziffer für Ziffer vorlesen und bestätigen lassen — genau einmal, in der Abschluss-Zusammenfassung, nicht schon beim Erfassen; mehrziffrige Nummern (Kundennr. o. ä.) nie zählen; PLZ einzeln; Straßen bei Bedarf buchstabieren lassen
- Im Gesprächsverlauf wird keine Angabe wiederholt. Nachgefragt wird nur, was akustisch nicht ankam; alles andere quittiert die KI mit „Alles klar" und geht weiter. Sonst verhandelt der Anrufer jede Antwort zweimal und das Gespräch verliert den Fluss
- Aussprache (TTS): Zahlen/Preise/Uhrzeiten/Datum, die die KI vorliest, als Wörter ausgeschrieben; schwer aussprechbare Eigennamen/Marken lautmalerisch („Reidl"→„Raidl"); Aussprache-Block am Prompt-Anfang
- Themeneingrenzung definiert (worüber die KI nicht spricht) — aber nicht so restriktiv, dass Small Talk stirbt
- Immer nur eine Frage auf einmal
- Natürliche Sätze statt vorgelesener Listen
- Antwortlänge max. 2–3 Sätze
- Spam-/Kaltakquise-Erkennung und -Abweisung
- Anrufer-Name nur dokumentiert, nicht ausgesprochen

### 1.3 Pflicht-Bausteine
- Kontaktdaten-Ablauf: 1. Rückrufnummer (liegt sie als Caller-ID vor: still übernehmen, nicht erfragen) → 2. Name (nur dokumentieren, nicht aussprechen; der Nachname genügt — nach dem Vornamen fragen kostet einen Zug für nichts, ein Rückruf braucht ihn nie) → 3. Zusammenfassung, in der alles erfasste einmal vorgelesen und in einem Zug bestätigt wird — auch die Caller-ID, denn der Anrufer kann von einem fremden Apparat aus anrufen
- Branchenspezifischer Notfall-Pfad, eigenständig und jederzeit aktiv
- Dreifach-Missverständnis-Abbruch definiert (dreimal nicht verstanden → Ausstieg mit Rückruf-Zusage)
- Je Weiterleitung zwei Stellen, arbeitsteilig: die **Weiche** im Prompt, an der Phase, wo die Entscheidung fällt — eine Zeile, endend auf `tool_call <name>`; die **Auslöser-Details** (Aufzählung der konkreten Fälle) im Bedingungsfeld des Tools als Wenn-Dann-Satz. Die Bedingung allein im Tool zu führen, feuert unzuverlässig — beobachtet: dasselbe Szenario löste mal aus, mal nicht, weil das Modell im Baum keine Stelle hat, an der es entscheidet. Der Fehler ist nicht die Doppelung, sondern **ungleiche Abdeckung**: jede Weiterleitung folgt demselben Muster, oder keine. Prüfsignal: kommt jeder Tool-Name mindestens einmal im Prompt vor?
- Jeder Weiterleitungszweig hat einen Scheitern-Arm. Zurück kommt der Anruf nur bei aktiver Ablehnung durch das Ziel; eine Mailbox gilt als angenommen und beendet das Gespräch, eine unterdrückte Rufnummer lässt sich nicht verbinden. Ein Ziel ohne Erreichbarkeitszusage ist eine Sackgasse — vor dem Bau klären, nicht im Prompt kompensieren
- Widerspricht der Anrufer der Aufzeichnung und steht das Dashboard auf automatisches Löschen, verfallen Transkript und Extraktion: dann weiterleiten oder mit Öffnungszeiten beenden, aber kein Anliegen aufnehmen und keinen Rückruf zusagen — das Ticket entsteht nie
- Post-Call-E-Mail-Zusammenfassung ans Team (Tool, nicht an Anrufer), enthält alle bearbeitungsrelevanten Felder
- Jedes Tool hat einen klaren Auslöser im Entscheidungsbaum
- Eigener Abschluss-Block, der für jeden Ausstieg gilt: genau ein kurzer Verabschiedungssatz → Gespräch unmittelbar danach beenden → keine Frage mehr, keine Bestätigung abwarten. Der deckt alle Ausstiege mit Verabschiedung ab; sie beenden den Anruf von selbst, ein Tool-Aufruf gehört dort nicht hin
- Ein Ausstieg **ohne** Verabschiedung, der sofort feuern muss (Notruf-Ansage), nennt den Aufruf am Zweig selbst — Prosa („beende das Gespräch", „lege auf") lässt das Modell die Aktion ankündigen statt ausführen. Schreibweise bei fonio: `tool_call <name>`, ohne Anführungszeichen, auch für Weiterleitungen. `end_call` ist ein Built-in ohne Beschreibungsfeld, das Wann muss deshalb komplett in den Prompt; selbst angelegte Tools (Weiterleitungen) tragen es in ihrer Beschreibung
- Variablen-Extraktion deckt die businessrelevanten Datenpunkte ab

### 1.4 Inhaltliche Qualität
- Detailinfos (FAQ, Preise, Kataloge) in Wissensdatenbank ausgelagert, nicht im Prompt
- fonio hat drei getrennte Speicher: **Unternehmensinformationen** (Freitext, immer im Kontext), **Öffnungszeiten-Feld** (strukturiert, eigenes UI-Element — vermutlich Basis der automatisierten Geschäftszeiten-Logik, fonio-seitig nicht bestätigt) und **Wissen/Wissensdatenbank** (Frage-Antwort-Paare, nur bei thematischem Bedarf abgerufen, retrieval-abhängig und praktisch unzuverlässig — beobachtet: eine im Prompt verankerte Wissens-Prüfung griff live trotzdem nicht). Ablage-Kriterium ist **Zuverlässigkeit**, nicht Kernfakt vs. Randfall: alles, das fast immer korrekt sitzen muss (Adresse, Leistungen, Notdienst-Kontakte) → Unternehmensinformationen. Zweite Achse: **Wortlaut-Determinismus** — Unternehmensinformationen wird frei paraphrasiert (ok bei egaler Formulierung); braucht eine Antwort exakten Wortlaut (Rechtliches), gezielt Wissen für genau diese Frage aktivieren (Kundenannahme, fonio-seitig nicht bestätigt). Default: Wissen aus, jede aktive Quelle ist eine weitere Fehlerfläche. Nie doppelt ablegen — erzeugt auseinanderlaufende Drittquellen (beobachtet: Öffnungszeiten in Wissen wich von Unternehmensinformationen ab)
- Unternehmensinformationen und Wissen sind Datenspeicher, kein Verhaltenscode: nur kurze Stichpunkte (Wert, Nummer, Kategorie) hinein — Bedingungen, Disclaimer, Tonalität und wann was gesagt wird gehören in den Prompt. Ein Speicher-Eintrag, der ein ganzer Satz mit Begründung ist, gehört umformuliert: der Fakt bleibt kurz im Speicher, die Logik wandert in die passende Phase
- KI beantwortet häufige Fragen direkt statt reflexhaft auf Rückruf zu verweisen
- Anrufertypen unterschieden (z. B. Neukunde vs. Bestandskunde)
- Empathie-Anweisungen bei sensiblen Branchen (Gesundheit, Recht, Versicherung, Pflege)
- Branchenspezifische Patterns eingebaut (z. B. Fristsachen bei Anwälten, Verordnungs-Fristen bei Therapeuten, Produktionsstillstand bei Industrie, Patientenversorgung bei Kliniken)

### 1.5 Anti-Patterns
- Startnachricht-Dopplung (KI begrüßt zweimal)
- Redundante Betonungen („NIEMALS!!!", Dreifach-Verneinung) — einmal klar reicht
- Gleiche Regel mehrfach im Prompt
- Widersprüche (duzen vs. siezen, immer vs. nie weiterleiten)
- Regel nennt einen konkreten Eval-Fall, Testnamen oder Transkript-Wortlaut (§3.3)
- Mehr als 10 FAQ-Antworten im Prompt statt Wissensdatenbank
- Derselbe Kernfakt gleichzeitig in Unternehmensinformationen und als eigener Wissen-Eintrag gepflegt (Divergenz-Risiko, siehe § 1.4)
- Überdetaillierte Sprechanweisungen (Millisekunden-Pausen, Silbenbetonung — Sache des TTS, nicht des Prompts)
- Regel ohne Grund, obwohl eine Begründung unlistete Ähnlichkeiten mit abdecken würde („Grund statt Betonung", → tools/claude-api.md)

### 1.6 Namens-Erkennung (falls relevant)
- Eigennamen (Team, Standorte) in den Fachbegriffen: nur korrekte Schreibweise, KEINE Varianten (Feld ist Biasing, Output = wie gelistet → Variante landet falsch im Ticket). Varianten nur, wo der Name bloß zum Routen erkannt und nicht gespeichert wird; sonst n8n-Normalisierung. TTS-Aussprache separat im Aussprache-Block (lautmalerisch)

### 1.7 Multi-Standort (falls relevant)
- Standort-Fachbereich-Matrix vorhanden
- Prüfung: gewünschter Service am gewünschten Standort verfügbar?
- Routing-Kriterium definiert (PLZ, Stadtname, Nähe)
- Eigenes Weiterleitungs-Tool pro Standort

### 1.8 Lieferformat
- Prompt als Markdown-Codeblock, direkt kopierbar
- Startnachricht ≤ 90 Zeichen (fonio-Empfehlung), KI-Offenlegung trotzdem enthalten
- Anrede über das fonio-Feld (Formell/Informell) gesetzt → Ansprechform NICHT auch im Prompt; nur bei „Individuell" im Prompt
- „Unterbrechung verhindern" für kurze Begrüßung aktiviert (KI-Offenlegung wird gehört)
- Darunter separat: Startnachricht, Dashboard-Einstellungen (Anrufdauer, Sensitivität, Kreativität, Stimme & Sprache; zusätzlich Maximale Wartezeit, genaue Informationsverarbeitung, Anrede-Feld), Tool-Konfiguration, Variablen-Extraktion, Wissensdatenbank-Inhalte

## 2 Transkript-Fehler-Taxonomie

### 2.1 Gesprächsführung
- **Endlosschleife**: dieselbe Frage wiederholt, weil Antwort nicht verstanden
- **Mehrfach-Fragen**: mehrere Fragen auf einmal
- **Listen vorgelesen** statt natürlich formuliert
- **Zu lange Antworten**: Absätze statt 2–3 Sätze
- **Roboter-Sprache**: „Ihre Anfrage wurde registriert", „Ich stehe Ihnen gerne zur Verfügung"
- **Begrüßungs-Dopplung**
- **Doppelbestätigung**: Angabe erst beim Erfassen und dann nochmal in der Zusammenfassung vorgelesen — kostet einen ganzen Gesprächszug und wirkt misstrauisch

### 2.2 Entscheidungsbaum
- **Falscher Pfad**: z. B. Bestandskunde im Neukunden-Pfad
- **Fehlender Pfad**: Anliegen ohne Zuordnung
- **Vorzeitige Weiterleitung**: obwohl selbst lösbar
- **Reflexhafter Rückruf-Verweis**: obwohl Antwort in Wissensdatenbank stand
- **Pfad-Abbruch**: Logik nicht zu Ende definiert

### 2.3 Datenerfassung
- Nummern/PLZ falsch erkannt
- Nummern als Gesamtzahl vorgelesen („siebzigtausendvierhundert" statt „sieben null vier null null")
- KI versucht Ziffern zu zählen und scheitert
- Pflichtdaten (Name, Rückrufnummer) vergessen
- Unnötige Daten abgefragt

### 2.4 Ton und Empathie
- Kühle Reaktion auf emotionale Situation
- Anrufer unterbrochen
- Anrufer wird hörbar ungeduldig — Ursache benennen

## 3 Ursachenanalyse (Eval-Modus)

Ein Eval-Fall ist eine Stichprobe, kein Anforderungskatalog. Wer ihn direkt fixt, baut einen Agenten, der die zwei Dutzend getesteten Situationen beherrscht und bei der dreiundzwanzigsten ratlos ist. Deshalb je Befund: Symptom → Ursache → Ebene → Geschwister-Test.

**Vor der Ebenen-Bestimmung (§3.1): der Algo aus operating-system.md.** Jede Frage kann den Fix beenden, bevor er anfängt:
1. **Hinterfragen.** Ist das Symptom real oder Artefakt eines einzelnen Tests — würde ein kritischer Prüfer es genauso werten? Wenn nicht: kein Fix, sondern Notiz.
2. **Löschen.** Lässt sich eine bestehende Regel, ein Speicher-Eintrag oder eine Phase ersatzlos streichen, statt etwas hinzuzufügen? Löschen schlägt Hinzufügen.
3. **Vereinfachen.** Gibt es eine Variante mit weniger beweglichen Teilen (weniger Speicher, weniger Instruktionsketten, weniger Abhängigkeiten)? Die einfachste Lösung, die trifft, gewinnt gegen die elegantere.

Beschleunigen/Automatisieren (Schritt 4–5 des Algo) kommen erst, wenn 1–3 durchlaufen sind und sich der Bedarf wiederholt hat — nicht vorab strukturell ausbauen (z. B. eine Wissensdatenbank-Struktur aufbauen, bevor der einfache Weg als unzureichend belegt ist).

### 3.1 Ebene des Fixes

Symptom in §2 einordnen, dann die Ebene bestimmen, auf der die Ursache sitzt. Auf der **höchsten zutreffenden** Ebene fixen — je höher, desto mehr ungetestete Fälle deckt der Fix mit ab.

| Ebene | Ursache | Fix |
| --- | --- | --- |
| **Wortlaut** | KI sagt das Richtige unnatürlich, zu lang oder unverständlich | Formulierung an der bestehenden Stelle ändern |
| **Regel** | Regel fehlt, ist negativ formuliert, steht doppelt oder widerspricht einer anderen | bestehende Regel schärfen, zwei zusammenlegen, Widerspruch auflösen |
| **Struktur** | Pfad fehlt, endet tot, Bedingung greift nicht, Phase in falscher Reihenfolge | Verzweigung reparieren |
| **Prinzip** | KI arbeitet den Baum starr ab, statt das Anliegen zu verstehen; scheitert an jeder Abweichung vom erwarteten Verlauf | ein Verhaltensprinzip früh im Prompt, dafür die Detailregeln streichen, die es ersetzt |

Die Prinzip-Ebene ist die einzige, auf der der Prompt kürzer wird, während er mehr abdeckt. Wiederholte Befunde derselben Ursache sind ihr Signal.

### 3.2 Geschwister-Test (Pflicht vor jedem Fix)

Drei Situationen benennen, die dieselbe Ursache auslösen, aber **nicht** im Eval-Set stehen. Deckt der Fix sie nicht mit ab, ist er zu eng → eine Ebene höher.

### 3.3 Überanpassung erkennen

Ein Fix ist verbrannt, wenn er:
- einen Fallnamen, eine Testadresse, einen Anrufernamen oder einen Wortlaut aus dem Transkript enthält
- als neuer Spiegelstrich unter „Regeln" landet, statt eine bestehende Stelle zu ändern
- nur für eine einzige Anliegen-Variante gilt
- eine Situation beschreibt statt ein Verhalten („Wenn jemand wegen Schimmel im Bad anruft" statt „Bei Gesundheitsgefahr im Wohnraum")

### 3.4 Gehört der Fix überhaupt in den Prompt?

| Befund | Gehört nach |
| --- | --- |
| Kernfakt falsch/fehlt (Adresse, Öffnungszeiten, Leistungen) ODER Information, die zuverlässig fallen muss (Notdienst-Kontakte, handlungskritische Selbsthilfe) | Unternehmensinformationen bzw. strukturiertes Öffnungszeiten-Feld — nicht als Wissen-Eintrag, Wissen ist retrieval-abhängig |
| Echtes Nice-to-have-FAQ, bei dem ein Fehltreffer keinen Schaden anrichtet | Wissensdatenbank/Wissen (Frage-Antwort-Paar) |
| Eigenname falsch erkannt | STT-Fachbegriffe (nur korrekte Schreibweise) oder n8n-Normalisierung |
| Weiterleitung ins Leere, falsche Empfänger, fehlende Mail-Felder | Tool-Konfiguration |
| Ticketfeld leer, falsch formatiert, Priorität falsch gesetzt | n8n-Post-Call |
| KI unterbricht, wartet zu kurz, versteht Zahlen schlecht | Dashboard (Sensitivität, Wartezeit, genaue Informationsverarbeitung) |
| KI kündigt eine Aktion an, statt sie auszuführen („ich beende jetzt das Gespräch", „ich verbinde Sie") und redet danach weiter | Erst prüfen, ob der Ausstieg überhaupt einen Abschluss hat: fehlt die Verabschiedung, ist es der Abschluss-Block (§1.3). Hat er einen und feuert trotzdem nicht, gehört der Tool-Aufruf an den Zweig. Umformulieren scheitert in beiden Fällen beliebig oft |

Bleibt nach dieser Sortierung nichts übrig, war es kein Prompt-Problem. Solche Befunde getrennt ausweisen, nicht in den Prompt schreiben.
