# Versuchsaufbau

Wie ein Telefon-Voice-Agent gemessen wird: was der Messgegenstand ist, womit gemessen wird, welche Kontrollen den Messfehler klein halten und wo der Aufbau an seine Grenze kommt.

Gebaut wurde er gegen eine Hausverwaltung — Notfälle, Schadensmeldungen, Weiterleitungen. Die Mechanik ist von der Branche unabhängig; die Pfadnamen sind es nicht.

## Messgegenstand

Gemessen wird **eine Promptversion**, nicht „der Agent". Die Version steht an drei Stellen gleich: im Frontmatter des Systemprompts, im Node `config` des Graders und in der Telefonie-Plattform. Läufe zweier Versionen zusammen zu werten misst nichts.

Zum Messgegenstand gehören Systemprompt, Wissensspeicher, Variablen, Tool-Beschreibungen und die Dashboard-Einstellungen der Plattform. Alles davon ändert das Verhalten, also gehört alles unter dieselbe Versionsnummer.

## Instrument

Ein Anruf wird über ein **gesprochenes Codewort** seinem Fall zugeordnet, nicht über die Rufnummer und nicht über die Uhrzeit. Nach dem Auflegen schickt die Post-Call-Automation den Payload an den Grader; der sucht das Codewort im normalisierten Transkript, hängt die Kriterien des Falls an, schneidet das Wort heraus und bewertet.

**Der Pfad entscheidet, wer bewertet.** Drei der vier Wege kommen ohne LLM aus — ein falsches „bestanden" wäre dort ein Haftungsfall, kein Messfehler:

| Pfad | Grader | Prüft |
| --- | --- | --- |
| `Notfall` | Regel | Pflichtansage in der geforderten Anzahl, kein Weiterreden danach |
| `Notdienst` | Regel | Weiterleitung nachweislich erfolgreich (`succeeded`-Tool-Zeile oder `disconnectReason: call_transfer`) und keine Ansage |
| `Angriff` | Regel | Verbotsliste aus `config` taucht im Transkript nicht auf |
| alles andere | Judge | `Bestanden wenn` / `Durchgefallen wenn`, Ticket-Zustand, feste Kinderfehler-Liste |

Zwei Details, die den Regel-Grader tragen:

- **Weiterleitung wird am Zustand gemessen, nicht am Gesagten.** „Ich verbinde Sie" sagt das Modell auch dann, wenn es kein Tool aufgerufen hat. Gezählt wird nur der Erfolg — für jeden Transfer steht zuerst eine Versuchszeile im Transkript, ein gescheiterter Versuch wäre sonst eine bestandene Weiterleitung.
- **Transkript und Prüfbegriffe laufen durch dieselbe Normalisierung** (Kleinschreibung, Umlaute aufgelöst, alles Nicht-Alphanumerische zu Leerzeichen). Sonst findet ein Begriff mit Umlaut sich selbst im umlautfreien Transkript nicht.

**Der Judge ist nie dasselbe Modell wie der Agent.** LLMs erkennen eigene Ausgaben und bewerten sie höher, als Menschen sie bewerten ([Panickssery et al. 2024](https://arxiv.org/abs/2404.13076)). Er bekommt Kriterien, Transkript, `disconnectReason` und Tool-Calls — nie den Systemprompt des Agenten, sonst bewertet er die Absicht statt das Ergebnis. `unklar` ist eine erlaubte Antwort; fällt die API aus, endet der Lauf als `unklar` und nie als stilles Durchgefallen.

## Kontrollen

- **Zwilling je Auslöser.** Zu jedem Fall, in dem ein Verhalten kommen *soll*, gehört einer, in dem es ausbleiben soll — gleiche Nummer mit `-Z-`. Ohne Zwilling kein Fall: „One-sided evals create one-sided optimization" ([Anthropic](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)). Der Grader dreht die Erwartung am `-Z-` im Fallnamen selbst um.
- **Codewort mitten im Gespräch.** Direkt nach der Begrüßung und am Ende ist die Spracherkennung am schwächsten — gemessen wurde aus „Amsel. Es riecht hier im" ein „Haben Sie das richtige". Je Fall ein kurzer deutscher Vogelname, vor dem Lauf als Fachbegriff in der Plattform hinterlegt, nach dem Lauf wieder heraus. Sonst hört der Agent im Betrieb Vogelnamen, wo keine fallen.
- **Rückhalte.** Ein Teil der Fälle wird nie angerufen und nie angesehen, bis das Gate erreicht ist. Erst diese zweite Zahl zeigt, ob der Prompt generalisiert statt sich an das Set anzupassen.
- **Instrument darf sich mitten in der Runde ändern, Messgegenstand nie.** Schärfen erlaubt: `Bestanden wenn`, Teilpunkte, Judge-Prompt, Grader-Schwellen. Unangetastet bleiben Systemprompt, Wissen, Variablen, Tools, Dashboard. Wer dort mitten in der Runde fixt, beschreibt zwei verschiedene Agenten unter derselben Version.
- **Nachbewertung statt Neuanruf.** Ein geschärftes Kriterium bewertet die betroffenen Zeilen neu — das Transkript liegt vor. Die Zeile wird mit `[Per Hand nachträglich angepasst - JJJJ-MM-TT HH:MM]` markiert, sonst liest die nächste Runde ein Grader-Urteil, das keines mehr ist.
- **Referenzlösung.** Ein bekannt funktionierendes Transkript je Fall. Es beweist, dass die Aufgabe lösbar ist, und prüft den Grader: ändert man den Grader, muss die Referenz weiter bestehen. Tut sie es nicht, ist der Grader kaputt, nicht der Agent.

## Metriken

**`pass^k`, nicht `pass@k`.** Ein Fall gilt nur als bestanden, wenn *alle* seine Anrufe bestehen — ein einziger Fehlschlag kippt ihn. Bei 75 % Erfolg je Lauf sind das über drei Läufe noch 42 % ([Yao et al., τ-bench](https://arxiv.org/abs/2406.12045)). Für einen Agenten, der ans Telefon geht, ist das die einzig ehrliche Metrik: der Anrufer bekommt keinen zweiten Versuch.

Vier Quoten, jede über ihren eigenen Stapel:

| Quote | Stapel | Erwartung |
| --- | --- | --- |
| Pass | Capability, ohne Rückhalte | niedrig, absichtlich |
| Δ zur Vorversion | derselbe Stapel | das Abbruchkriterium |
| Regression | Regression | 100 %, sonst ist etwas kaputtgegangen |
| Rückhalte | Rückhalte | erst am Gate |

Eine Quote gilt erst, wenn kein Fall ihres Stapels mehr `offen` steht. `Punkte 0-2` geht in keine Quote ein — Teilpunkte sind die zweite Dimension neben dem Bestehen, nicht ein Viertelbestehen. Strenger ist nur das Gate: `Notfall`, `Notdienst` und `Angriff` müssen jeden einzelnen Lauf bestehen.

**Zwei Schleifen.** Am Anfang steht jeder Fall auf `Capability` — unbewiesen ist nicht bestanden. Wer zwei Runden in Folge alle seine Anrufe besteht, wird `Regression` und fällt aus der Runde raus; wer als Regressionsfall durchfällt, geht zurück auf `Capability`. Der Regressionslauf wird nicht nach Kalender gefahren, sondern bei Auslösern: vor dem Live-Gang, danach vor jeder Promptänderung, jedem Plattform-Update, jedem Modellwechsel.

Das ist die Regel, die ein handgefahrenes Set dauerhaft trägt: die Rundenkosten hängen am Capability-Stapel, nicht an der Größe des Sets — bei 30 Fällen wie bei 80.

## Ablauf

**Aufsetzen.** Tabelle anlegen, Grader importieren, in `config` Promptversion, Pflichtansage, Zuggrenze und Verbotsliste eintragen. Im Post-Call-Workflow die Weiche zum Grader *hinter* die Ticket-Erstellung hängen — sonst misst der Eval einen Anruf, der kein Ticket hinterlassen hat.

**Vor dem ersten Lauf.** Die Leitung prüfen, nicht das Verhalten: zwei Anrufe, einer ohne Codewort (muss als `nicht zugeordnet` landen), einer mit (muss den richtigen Fall treffen, das Codewort aus dem Transkript entfernt haben und ein gefülltes Ticket zeigen). Danach die Zeilen löschen. Jedes gefundene Problem sofort fixen — das ist keine Messung.

Dann den Judge kalibrieren: die ersten fünf Urteile gegenlesen. Weicht eines vom eigenen ab, entscheidet der Zwei-Menschen-Test — käme ein Zweiter, der nur `Bestanden wenn` und das Transkript sieht, zum selben Urteil? Ja → Kriterium schärfen. Nein → Zeile stehen lassen, der Agent war wirklich schlecht.

**Je Runde.** Der Capability-Stapel komplett, in dieser Reihenfolge: erst Haftung, dann je ein Vertreter pro Pfad, dann der Rest. Komplett durchziehen oder abbrechen. Beim Anrufen auf Papier mitschreiben — der Grader sieht das Transkript, nicht den Klang: Pausen, Betonung, der Moment, an dem ein echter Anrufer aufgelegt hätte. Danach Testtickets löschen, Befunde bündeln, Version hoch.

Ein durchgefallener Fall wird gegen seine Referenzlösung gelegt und am **ersten abweichenden Zug** gelesen — dort sitzt die Ursache, nicht dort, wo das Gespräch sichtbar entgleist. Mehrere Fälle mit derselben Ursache ergeben *einen* Fix. Fällt ein Fall durch, der nie referenzgelöst war, wird zuerst geprüft, ob das Kriterium überhaupt erreichbar ist: kaputte Fälle werden korrigiert, nicht der Prompt an sie gebogen. Einzige Ausnahme ist `Angriff` — dort ändert ein Fehlschlag immer den Systemprompt, nie den Fall.

**Gate.** Aufhören, wenn eine Runde nichts mehr bringt, realistisch nach drei bis vier. Dann zweimal messen: Arbeitsstapel, danach Rückhalte. Danach den Prompt nicht mehr anfassen. Fallen Rückhalte durch, entscheidet das Transkript — mehrdeutiger Fall wird korrigiert, fairer Fall wandert in den Arbeitsstapel und es folgt eine weitere Runde mit neuen Rückhalten.

**Nach dem Live-Gang.** Jeder schiefgelaufene echte Anruf wird ein Capability-Fall, je Ursache einer, nicht je Anruf. Ausgelöst vom Fehler, nicht vom Kalender: bei zweistelligen Anrufzahlen je Woche ist jede Wochenquote Rauschen. Ein Wachwert genügt — der Anteil der Anrufer, die selbst auflegen, mit einer beim Live-Gang aus der Baseline festgeschriebenen Schwelle.

## Datenschema

Zwei Tabellen tragen den Aufbau. Der Grader liest die erste und schreibt die zweite; von Hand geschrieben wird nur die erste.

**Fälle** — `Fall` · `Codewort` · `Pfad` · `Zwilling zu` · `Kontext` · `Anrufer sagt` · `Bestanden wenn` · `Durchgefallen wenn` · `Punkte 0-2` · `Anrufe` · `Zweck` (`Capability`/`Regression`) · `Rückhalte` · `Ticket erwartet` · `Referenzlösung`

**Läufe** — `Lauf` · `Fall` · `Promptversion` · `Bestanden` · `Punkte` · `Begründung` · `Transkript` · `Dauer` · `Züge` · `Tool-Calls` · `disconnectReason` · `Ticket`

Die Auswertung ist reine Formelarbeit über diesen beiden Tabellen und reicht bis Fall 40 — wer mehr Fälle braucht, zieht die Formeln nach unten.

`Züge` zählt nur Gesprächszüge; Tool-Zeilen bleiben draußen, sonst sieht jede Weiterleitung zwei Züge länger aus und die Zuggrenze schlägt an, wo niemand nachgefragt hat.

## Grenzen

Was dieser Aufbau **nicht** kann — für die Bewertung der Zahlen wichtiger als das, was er kann:

- **Ein Anruf je Fall außerhalb von Haftung und Angriff.** Die Lehrmeinung verlangt mehrere Läufe je Fall, weil ein einzelner Trial kein Ergebnis ist. Hier wird von Hand telefoniert; drei Anrufe über das ganze Set sind nicht bezahlbar. Verdichtet wird deshalb nur dort, wo ein Fehlschlag teuer ist. Das ist eine Kostenentscheidung, keine methodische.
- **Kleines N.** Ein handgefahrenes Set liegt bei zwei bis drei Dutzend Fällen. Es findet Fehlermodi, es schätzt keine Fehlerraten.
- **Der Grader hört nicht.** Er liest ein Transkript. Prosodie, Pausen, Sprechtempo und der Moment, in dem ein echter Anrufer entnervt auflegt, kommen nur über die handschriftlichen Notizen in die Auswertung.
- **Der Judge ist gegen fünf Urteile kalibriert**, nicht gegen einen Goldstandard-Datensatz mit Übereinstimmungsmaß.
- **Die Spracherkennung ist Teil der Messung.** Ein Fall kann am Codewort scheitern statt am Agenten. Deshalb der Systemtest vorweg und die Regel, das Wort mitten im Gespräch zu nennen.

## Quellen

- Anthropic, [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — zwei Schleifen, Referenzlösung, Ergebnis statt Weg bewerten, `pass^k`, Zwillingsregel
- Anthropic, [Create strong empirical evaluations](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests) — Erfolgskriterium, Setgröße, Edge Cases, Graderwahl
- Anthropic, [Mitigate jailbreaks and prompt injections](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks) — Angriffsfälle, Fremdinhalt als Tool-Ergebnis, Red Teaming vor dem Live-Gang
- Panickssery et al., [LLM Evaluators Recognize and Favor Their Own Generations](https://arxiv.org/abs/2404.13076) — warum der Judge nicht das Agentenmodell sein darf
- Yao et al., [τ-bench](https://arxiv.org/abs/2406.12045) — `pass^k` als Zuverlässigkeitsmaß
- Chen et al., [Evaluating Large Language Models Trained on Code](https://arxiv.org/abs/2107.03374) — `pass@k`, das Gegenstück
