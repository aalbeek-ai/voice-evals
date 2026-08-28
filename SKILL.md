---
name: voice-evals
description: Voice-Agent-Prompts für Telefon-KI gegen ein Eval-Set messen und verbessern. Immer einsetzen, wenn der Nutzer Eval-Fälle schreiben, eine Eval-Runde auswerten, Call-Transkripte prüfen, einen Fehlschlag auf seine Ursache zurückführen oder einen Voice-Agent-Prompt auditieren will — auch wenn „Eval" nicht explizit fällt.
---

# voice-evals — Evals für Telefon-Voice-Agents

Zwei Aufgaben, beide gegen dasselbe Set: **Fälle schreiben** (§ Fälle) und **Runde auswerten** (§ Auswertung). Dem Nutzer in einem Satz sagen, welche läuft.

Alle Kundenwerte — Promptversion, Testnummern, Workflow-IDs, offene Punkte — stehen im Tab `Setup` des Spreadsheets, nirgends sonst.

Die Datenlage bestimmt nur, woher die Befunde stammen, nie den Ablauf: Läufe aus dem Spreadsheet › eingefügte Transkripte › nur die Audit-Checkliste. Je dünner die Lage, desto mehr Befunde stammen aus regeln.md §1 statt aus Beobachtung — das im Ergebnis kennzeichnen.

## Warum die Regeln existieren

Die Checklisten stehen in `regeln.md`, das Gerüst in `prompt-template.md`. Hier nur das Warum, damit Fixes auf die Ursache zielen statt die Symptomliste abzuarbeiten. Die §-Verweise zeigen in `regeln.md`.

1. **STT und TTS haben getrennte Fehlermodi → spiegelverkehrte Regeln.** TTS liest vor (Ausgabefehler: Aussprache), STT hört zu (Eingabefehler: Erkennung) — dieselbe Sache, Namen und Zahlen, braucht entgegengesetzte Behandlung. Die Aussprache-Seite gilt für jeden Text, den die Engine vorliest, nicht nur für den Prompt. §1.2, §1.6
2. **Instruktionsdichte senkt Befolgung.** Jede Regel genau einmal, Details in den Wissensspeicher statt in den Prompt. 800 Wörter ohne Doppelung schlagen 400 mit. §1.4, §1.5
3. **Der Prompt beschreibt Verhalten, kein Fallverzeichnis.** Ein Eval-Set deckt zwei Dutzend Fälle ab, der Agent erlebt tausende. Eine Regel, die nur den getesteten Fall trifft, macht den Agenten überall sonst starrer und verbraucht das Instruktionsbudget aus Punkt 2. §3
4. **Grund statt Betonung.** „NIEMALS Auslassungspunkte" wirkt nur auf Auslassungspunkte; „der Text wird von einer TTS-Engine vorgelesen, die … nicht kann" generalisiert auf jedes ähnliche Zeichen. §1.5
5. **Positive Anweisungen.** „Wenn X → sage Y" statt „sage nie Z" — Negationen befolgen LLMs unzuverlässig.
6. **Ankündigen ist nicht Handeln.** Ein Ausstieg oder eine Weiterleitung, die nur als Prosa dasteht, wird angesagt statt ausgeführt; der Aufruf gehört an den Zweig. Prüfsignal ist der Trennungsgrund des Anrufs — hat der Agent aufgelegt oder der Anrufer? §1.3

## Daten

Je Kunde **eine** Spreadsheet-Datei, fünf Tabs: `Setup` · `Fälle` · `Läufe` · `Auswertung` · `Systemtests`.

Lesen und Schreiben laufen über den Skill `google-sheets` — dort stehen Tool-Auswahl und die Locale-Fallen der Sheets-API. Für diesen Skill zählt nur: die `Läufe` schreibt der Grader, nie du. Beitrag dieses Skills sind neue Fallzeilen und gefüllte `Referenzlösung`.

## Fälle

Eingabe: Systemprompt, Tool-Beschreibungen, Variablen, Wissensspeicher, Stammdaten des Kunden. Ausgabe: Zeilen für den Tab `Fälle` — Spaltenreihenfolge aus der Kopfzeile der Kundendatei lesen, nicht aus dem Gedächtnis.

1. **Pfade auszählen.** Je Phase, je Weiterleitung, je Regel im Prompt ein Auslöser. Das ist die Grundgesamtheit, nicht die Fantasie.
2. **Je Auslöser einen Zwilling.** „One-sided evals create one-sided optimization" — ein Fall, in dem das Verhalten kommen *soll*, und einer, in dem es ausbleiben soll. Der Zwilling trägt dieselbe Nummer mit `-Z-` und die Spalte `Zwilling zu`. Ohne Zwilling kein Fall.
3. **Edge Cases als eigene Fälle:** Tippfehler und Nuscheln, mehrere Anliegen in einem Anruf, Themenwechsel mitten im Gespräch, ambige Angaben, unterdrückte Nummer, außerhalb der Geschäftszeiten, Anrufer legt auf.
4. **Angriffsfälle getrennt**, eigener Pfad `Angriff`, Regel-Grader statt Judge (der Angreifertext nimmt den Judge sonst mit). In `Angriff` steht nur, wo Daten abfließen könnten; was Verhalten misst, gehört zu `Regeln`.
5. **Jede Zeile vollständig**, sonst greift sie nicht: `Codewort` ist das gesprochene Erkennungswort · `Kontext` bestimmt, von welcher Nummer angerufen wird · `Anrufe` die Wiederholungen (3 bei Haftung und Angriff, sonst 1) · `Ticket erwartet` den Zustand nach dem Anruf · `Zweck` beginnt bei `Capability` · `Rückhalte` bleibt leer, außer der Fall wird bewusst zurückgehalten.
6. **Erreichbarkeit prüfen, bevor der Fall ins Set geht.** Ist `Bestanden wenn` mit dem erreichbar, was der Agent tatsächlich hat — Wissensspeicher, Stammdaten, Tools? Sonst misst der Fall den Fall, nicht den Agenten.
7. **Zwei Prüfer, ein Urteil.** Formuliere `Bestanden wenn` / `Durchgefallen wenn` so, dass zwei Menschen unabhängig zum selben Pass/Fail kämen. Was nur du entscheiden kannst, ist kein Kriterium.
8. **Teilpunkte** in `Punkte 0-2` definieren, wo die Aufgabe mehrteilig ist: Anliegen erkannt aber Ticket unvollständig ist besser als Sofort-Scheitern. Binär bleibt, was binär ist (Haftung, Angriff).
9. **Codewort vergeben**, je Fall genau eines: ein kurzer, geläufiger **deutscher Vogelname**, zweisilbig und auf Anhieb aussprechbar — Amsel, Möwe, Specht, Drossel, Reiher, Fink. Keine seltenen oder langen (Zilpzalp, Trauerschnäpper), keine verwechselbaren (Star/Storch), keines, das im Sprechtext vorkommen kann. Der Anrufer nennt es mitten im Gespräch, der Grader sucht es im ganzen Transkript und schneidet es vor der Bewertung heraus. Vor jedem Lauf gehören die Codewörter ins Fachbegriff-Feld der Plattform und danach wieder heraus — ohne Eintrag überhört die Spracherkennung sie. Keine Fall-ID sprechen lassen: `NOT-01` wird zu „N O T null eins“. Nicht in `Anrufer sagt` schreiben — das Codewort ist Testmechanik, kein Teil des Anliegens.

**Bringt der Nutzer einen Fall aus eigener Erfahrung mit** — ein realer Anruf, eine Vermutung, ein Ärgernis —, ist das die beste Quelle, die es gibt: sie kommt aus der Produktion, nicht aus der Fantasie. Nicht abweisen, sondern in eine vollständige Zeile übersetzen und drei Dinge prüfen: Deckt ein bestehender Fall denselben Auslöser schon ab (dann Zeile schärfen statt neue anlegen)? Ist das Kriterium beobachtbar formuliert? Fehlt der Zwilling? Danach mit den fehlenden Spalten zurückfragen, nicht raten.

## Auswertung

1. **Basis festlegen.** Genau **eine** Promptversion je Auswertung — Läufe zweier Versionen zusammen misst nichts. Rückhalte-Fälle nicht öffnen; wer sie gesehen hat, verbrennt sie. Bei `Anrufe` > 1 gilt `pass^k`: ein Fehlschlag = Fall durchgefallen.
2. **Fehlschläge sammeln.** Je durchgefallenem Fall: Transkript und Grader-Begründung. Grader-Begründung ist ein Hinweis, kein Befund — der Befund steht im Transkript.
3. **`Referenzlösung` heranziehen**, wo vorhanden:
   - **Fall durchgefallen** → Fehl-Transkript neben die Referenz legen und den **ersten abweichenden Zug** benennen. Dort sitzt die Ursache, nicht dort, wo das Gespräch sichtbar entgleist.
   - **Referenz mit dem heutigen Prompt nicht mehr erreichbar** (Pfad entfernt, Tool getauscht, Modell gewechselt) → Fall ist veraltet. Fall und Referenz korrigieren, **nicht** den Prompt daran biegen.
   - **Fall besteht zum ersten Mal und Feld ist leer** → sein Transkript nach `Referenzlösung` schreiben.
4. **Ursache statt Symptom** — regeln.md §3. Pflicht je Befund: Symptom → Ursache → Ebene des Fixes → Geschwister-Test. Mehrere Fälle mit derselben Ursache ergeben **einen** Fix, nicht mehrere.
5. **Fix schreiben — Algo aus regeln.md §3 zuerst, nicht nur zitiert.** Vor jeder Zeile: hinterfragen (Symptom real?), dann löschen (was geht ersatzlos raus?), erst danach vereinfachen. **Erfolg ist eine Stelle, die kürzer wird, nicht länger** — Wortzahl alt/neu im Ergebnis nennen; wächst sie doch, begründen warum. Jede Runde, die addiert, frisst die Befolgung, die sie herstellen will (§ Warum 2 und 3).
6. **Liefern.** Vollständig überarbeiteter Prompt (kein Diff) als Codeblock, Aufbau nach `references/prompt-template.md`, geänderte Plattform-, Tool- oder Wissensspeicher-Empfehlungen darunter. Dazu das Gate: die KPIs stehen im Tab `Setup`, der Stand im Tab `Auswertung`. Von dort die vier Quoten übernehmen — Pass-Quote, Δ zur Vorversion, Regression, Rückhalte. Δ ist die Zahl, die über Weitermachen oder Aufhören entscheidet. Dazu die Nebenzahlen aus den Läufen: Dauer, Züge, Tool-Calls, Trennungsgrund — eine Quote allein sagt nicht, was sie gekostet hat. Fixes, die nicht in den Prompt gehören (regeln.md §3.4), getrennt ausweisen.
7. **Rückfragen** max. 5 — nie etwas fragen, das in Prompt, Transkripten oder Läufen steht.
