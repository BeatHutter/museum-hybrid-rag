# Evaluation

**English** · [Deutsch](#deutsche-fassung)

> A German version follows below. · Eine deutsche Fassung folgt weiter unten.

Every change to content preparation, search or prompts was measured against a fixed test set before it was adopted. This document describes the test set, the metrics and the results from the first version to the MVP (as of 16 November 2026). The test questions contain material from the archive and are therefore not published.

## Test set (ground truth)

99 test cases in four suites:

| Suite | Test cases | What is checked |
| --- | ---: | --- |
| Text search | 30 | 13 questions about names and addresses, 8 topical questions, 4 multi-part questions with partial knowledge, 5 questions without evidence in the archive |
| Intent detection | 40 | Knowledge question, photo search, both, question about the collection, greeting, off-topic; plus the search mode (name or topic) |
| Photo search | 14 | Buildings, places, an event, a subject, two searches by photographer, and 2 searches without matching photos |
| Dialogue | 15 | Follow-up questions in 5 conversations of three turns each |

For questions about names and addresses, a script determined the expected pages by scanning all PDFs for verbatim occurrences, including spelling variants. It works independently of the system's search, so that the system is not graded against its own logic. For topical questions, the key pages were selected by hand. Questions without evidence were checked to have no match in the archive. For photo searches, the expected inventory numbers were defined from the catalogue.

## Metrics

- **Recall (text search):** share of the expected pages that the search returns. Passages spanning several pages count for every expected page they cover.
- **Precision (text search):** share of the returned pages that are expected.
- **Questions without evidence:** the system must answer "no information" and must not invent an answer.
- **Partial knowledge:** for multi-part questions, the answer must state which part is not covered.
- **Recall@10 and precision@5 (photo search):** share of the expected pictures among the first ten results, and share of correct pictures among the first five.
- **Intent detection and dialogue:** share of questions or conversation turns that are routed correctly.

## Results: first version and MVP

| Metric | First version | MVP |
| --- | ---: | ---: |
| Text search: recall for names and addresses | 77.3 % | 98.7 % |
| Text search: precision for names and addresses | – | 93.5 % |
| Text search: recall for topics | 45.0 % | 79.4 % |
| Text search: questions without evidence answered with "no information" | 4 of 5 | 5 of 5 |
| Text search: gaps named correctly in multi-part questions | – | 4 of 4 |
| Photo search: correct pictures among the top 10 | 29.9 % | 85.8 % |
| Photo search: share of correct pictures among the top 5 | 33.3 % | 86.7 % |
| Photo search: searches without matches answered correctly | 0 of 2 | 1 of 2 |
| Intent detection | 33 of 40 | 40 of 40 |
| Follow-up questions in dialogue | 13 of 15 | 15 of 15 |

"First version" is the state of 28 and 29 October 2026, measured with the same test set as the MVP. All runs use OpenAI gpt-4o-mini.

**Photo search re-measured (18 November 2026).** The photo figures were first measured on 29 October 2026 (94.2 % among the top 10, 95.0 % among the top 5, 2 of 2). The next day, the unified planner and the name correction were added, and the photo suite was not re-run. A re-measurement with the current code gives the values in the table above. The search itself is unchanged; the differences come from the planner: "Photos of <person>" is now read as "taken by", so pictures showing the person are missing; the name correction turns an unknown full name into a similar local name; and one building query passes a different search term. These three points are being fixed.

## Step by step: what moved the numbers (text search)

| Step | Recall names | Recall topics | No evidence |
| --- | ---: | ---: | ---: |
| First version | 77.3 % | 45.0 % | 4 of 5 |
| Text cleaning, paragraphs across pages | 86.6 % | 48.5 % | 4 of 5 |
| Context header per passage (contextual retrieval) | 86.6 % | 57.6 % | 4 of 5 |
| Exact name search with word boundaries and variants | 98.7 % | 60.4 % | 4 of 5 |
| LLM reranker | 98.7 % | 83.6 % | 4 of 5 |
| Relevance threshold | 98.7 % | 81.8 % | 5 of 5 |
| Answer prompt with named gaps (MVP) | 98.7 % | 79.4 % | 5 of 5 |

The last two steps traded a little recall for topics for reliability: weakly supported pages are dropped, and the system no longer speculates. For a museum, an honest "no information" is worth more than a few percentage points of recall.

A model comparison during development showed the same recall for the larger gpt-4o, but only 2 of 5 correct answers to questions without evidence, compared with 4 of 5 for gpt-4o-mini at the time. The project therefore stayed with the smaller model.

## Limitations

- **Small test set:** 99 cases cover the most important question types, but not the full range of real questions. The cases were compiled by the developer.
- **No broad user test yet:** the MVP has passed technical and functional testing. Tests with people from the society and the village are the next step.
- **Variability of language models:** answers can vary between runs. Critical behaviour such as the "no information" answer was checked over several runs.
- **Retrieval, not answer quality:** most metrics measure whether the right pages and pictures are found. The wording of the answers was checked by hand, not scored automatically.

## Next steps

- User tests with a broader group and analysis of the log: which questions remain open, where answers are rated poorly.
- Extend the test set with real questions from the tests.
- Tracing with Langfuse to follow each step of a request.

---

## Deutsche Fassung

# Evaluation

Jede Änderung an Aufbereitung, Suche oder Prompts wurde vor der Übernahme gegen ein festes Testset gemessen. Dieses Dokument beschreibt das Testset, die Messgrössen und die Ergebnisse vom ersten Wurf bis zum MVP (Stand 16.11.2026). Die Testfragen enthalten Inhalte aus dem Archiv und werden deshalb nicht veröffentlicht.

## Testset (Ground Truth)

99 Testfälle in vier Suiten:

| Suite | Testfälle | Was geprüft wird |
| --- | ---: | --- |
| Textsuche | 30 | 13 Fragen zu Namen und Adressen, 8 Sachfragen, 4 mehrteilige Fragen mit Teilwissen, 5 Fragen ohne Beleg im Archiv |
| Absichtserkennung | 40 | Wissensfrage, Fotosuche, beides, Frage zum Bestand, Begrüssung, themenfremd; dazu der Suchmodus (Name oder Thema) |
| Fotosuche | 14 | Gebäude, Orte, ein Ereignis, ein Motiv, zwei Suchen nach Fotografen und 2 Suchen ohne passende Fotos |
| Dialog | 15 | Folgefragen in 5 Gesprächen mit je drei Schritten |

Bei Fragen zu Namen und Adressen ermittelte ein Skript die erwarteten Seiten, indem es alle PDFs nach wörtlichen Vorkommen samt Schreibvarianten durchsuchte. Es arbeitet unabhängig von der Suche des Systems, damit das System nicht an seiner eigenen Logik gemessen wird. Bei Sachfragen wurden die Kernseiten von Hand ausgewählt. Bei Fragen ohne Beleg wurde geprüft, dass das Archiv tatsächlich nichts dazu enthält. Für die Fotosuchen wurden die erwarteten Inventarnummern aus dem Katalog festgelegt.

## Messgrössen

- **Trefferquote (Recall, Textsuche):** Anteil der erwarteten Seiten, die die Suche liefert. Abschnitte über mehrere Seiten zählen für jede erwartete Seite, die sie abdecken.
- **Präzision (Textsuche):** Anteil der gelieferten Seiten, die erwartet sind.
- **Fragen ohne Beleg:** Das System muss mit «keine Angaben» antworten und darf keine Antwort erfinden.
- **Teilwissen:** Bei mehrteiligen Fragen muss die Antwort sagen, welcher Teil nicht belegt ist.
- **Recall@10 und Precision@5 (Fotosuche):** Anteil der erwarteten Bilder unter den ersten zehn Treffern und Anteil richtiger Bilder unter den ersten fünf.
- **Absichtserkennung und Dialog:** Anteil der Fragen oder Gesprächsschritte, die richtig zugeordnet werden.

## Ergebnisse: erster Wurf und MVP

| Messgrösse | Erster Wurf | MVP |
| --- | ---: | ---: |
| Textsuche: Trefferquote bei Namen und Adressen | 77.3 % | 98.7 % |
| Textsuche: Präzision bei Namen und Adressen | – | 93.5 % |
| Textsuche: Trefferquote bei Sachthemen | 45.0 % | 79.4 % |
| Textsuche: Fragen ohne Beleg mit «keine Angaben» beantwortet | 4 von 5 | 5 von 5 |
| Textsuche: Lücken bei mehrteiligen Fragen korrekt benannt | – | 4 von 4 |
| Fotosuche: richtige Bilder unter den ersten 10 | 29.9 % | 85.8 % |
| Fotosuche: Anteil richtiger Bilder unter den ersten 5 | 33.3 % | 86.7 % |
| Fotosuche: Suchen ohne Treffer korrekt beantwortet | 0 von 2 | 1 von 2 |
| Absichtserkennung | 33 von 40 | 40 von 40 |
| Folgefragen im Dialog | 13 von 15 | 15 von 15 |

«Erster Wurf» ist der Stand vom 28. und 29.10.2026, gemessen mit demselben Testset wie das MVP. Alle Läufe nutzen gpt-4o-mini von OpenAI.

**Fotosuche nachgemessen (18.11.2026).** Die Fotowerte wurden zuerst am 29.10.2026 gemessen (94.2 % unter den ersten zehn, 95.0 % unter den ersten fünf, 2 von 2). Am Tag danach kamen der vereinigte Planer und die Namenskorrektur dazu, und die Foto-Suite lief nicht erneut. Eine Nachmessung mit dem aktuellen Code ergibt die Werte in der Tabelle oben. Die Suche selbst ist unverändert; die Unterschiede kommen vom Planer: «Fotos von <Person>» wird jetzt als «fotografiert von» gelesen, sodass Bilder fehlen, auf denen die Person zu sehen ist; die Namenskorrektur macht aus einem unbekannten vollen Namen einen ähnlichen Namen aus dem Dorf; und eine Gebäudeanfrage gibt einen anderen Suchbegriff weiter. Diese drei Punkte werden behoben.

## Schritt für Schritt: was die Zahlen bewegt hat (Textsuche)

| Schritt | Trefferquote Namen | Trefferquote Themen | Ohne Beleg |
| --- | ---: | ---: | ---: |
| Erster Wurf | 77.3 % | 45.0 % | 4 von 5 |
| Textbereinigung, Absätze über Seiten hinweg | 86.6 % | 48.5 % | 4 von 5 |
| Kopfzeile pro Abschnitt (Contextual Retrieval) | 86.6 % | 57.6 % | 4 von 5 |
| Wortgenaue Namenssuche mit Wortgrenzen und Schreibweisen | 98.7 % | 60.4 % | 4 von 5 |
| LLM-Reranker | 98.7 % | 83.6 % | 4 von 5 |
| Relevanzschwelle | 98.7 % | 81.8 % | 5 von 5 |
| Antwort-Prompt mit benannten Lücken (MVP) | 98.7 % | 79.4 % | 5 von 5 |

Die letzten beiden Schritte tauschten etwas Trefferquote bei Sachthemen gegen Verlässlichkeit: Schwach belegte Seiten fallen weg, und das System spekuliert nicht mehr. Für ein Museum ist ein ehrliches «keine Angaben» mehr wert als ein paar Prozentpunkte Trefferquote.

Ein Modellvergleich während der Entwicklung zeigte für das grössere gpt-4o dieselbe Trefferquote, aber nur 2 von 5 richtigen Antworten auf Fragen ohne Beleg, gegenüber 4 von 5 bei gpt-4o-mini zu jenem Zeitpunkt. Das Projekt blieb deshalb beim kleineren Modell.

## Grenzen

- **Kleines Testset:** 99 Fälle decken die wichtigsten Fragetypen ab, aber nicht die ganze Breite echter Fragen. Zusammengestellt hat sie der Entwickler.
- **Noch kein breiter Nutzertest:** Das MVP hat das technische und funktionale Testing durchlaufen. Tests mit Personen aus Verein und Dorf sind der nächste Schritt.
- **Schwankungen von Sprachmodellen:** Antworten können von Lauf zu Lauf leicht abweichen. Kritisches Verhalten wie die Antwort «keine Angaben» wurde über mehrere Läufe geprüft.
- **Suche, nicht Antwortqualität:** Die meisten Messgrössen prüfen, ob die richtigen Seiten und Bilder gefunden werden. Die Formulierung der Antworten wurde von Hand geprüft, nicht automatisch bewertet.

## Nächste Schritte

- Nutzertests mit einem breiteren Kreis und Auswertung des Protokolls: welche Fragen offen bleiben, wo Antworten schlecht bewertet werden.
- Das Testset mit echten Fragen aus den Tests erweitern.
- Tracing mit Langfuse, um jeden Schritt einer Anfrage nachzuverfolgen.
