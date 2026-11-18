# Key design decisions

**English** · [Deutsch](#deutsche-fassung)

> A German version follows below. · Eine deutsche Fassung folgt weiter unten.

A selection of eleven decisions from the project's internal decision log (architecture decision records). Each entry states the problem, the decision, the reasoning and the price paid. Where possible, the effect was measured; the figures come from the evaluation described in [evaluation.md](evaluation.md).

## 1. Own code instead of an orchestration framework

- **Problem:** connecting the vector database, the language model and two user interfaces needs orchestration. Frameworks such as LangChain or LlamaIndex offer this ready-made.
- **Decision:** a thin layer of own Python code with FastAPI and direct calls to the Qdrant and OpenAI SDKs.
- **Why:** the scope is manageable (two search paths, one model). Without a framework, every step is visible, testable and explainable, and there are fewer dependencies that change underneath the project.
- **Price:** more code to write and maintain, for example for retries and prompt handling.

## 2. Self-hosted Qdrant instead of a cloud search service

- **Problem:** about 1 GB of text and 1,800 photo records with a low query volume. Managed cloud search services were evaluated, as was the PostgreSQL extension pgvector.
- **Decision:** Qdrant in Docker on the project's own server.
- **Why:** native hybrid search (dense vectors, BM25, Reciprocal Rank Fusion) in a single call, low cost, and full control over the data. With pgvector, the fusion of both search types would have had to be built by hand.
- **Price:** one more container to operate and update.

## 3. Two search routes for text: exact for names, hybrid for topics

- **Problem:** questions about a person, farmstead or address expect completeness and exact matches. Open questions ("What was school like?") need semantic similarity. One search method cannot do both well.
- **Decision:** the planner classifies each question. Names and addresses go to an exact word search with word boundaries and spelling variants; topics go to hybrid search.
- **Why and effect:** house number 2 no longer matches 20, and all passages on a name are found. Recall for names and addresses rose from 86.6 % to 98.7 %, precision to 93.5 %.
- **Price:** the classification has to be reliable. Misclassified name questions fall back to hybrid search.

## 4. Clean the text and add context to every passage

- **Problem:** hyphenation at line ends split thousands of words, paragraphs were cut at page breaks, and a passage from the middle of an interview did not say who was speaking.
- **Decision:** resolve hyphenation, merge paragraphs across pages, exclude lists of publications, and give every passage a header with work, chapter or narrator, year and a short summary (contextual retrieval).
- **Why and effect:** cleaning raised recall for names and addresses from 77.3 % to 86.6 %. The context header raised recall for topics from 48.5 % to 57.6 %.
- **Price:** longer input when the embeddings are created once (below two US cents for the whole archive) and an additional caching step.

## 5. A language model as reranker, with a relevance threshold

- **Problem:** hybrid search returned the right pages among the first 35, but often ranked introductions and marginal pages higher. Questions without an answer in the archive still produced an answer.
- **Decision:** gpt-4o-mini scores up to 35 candidate pages on a scale of 0 to 3. If no page reaches the threshold, the system answers "no information" deterministically, without generating text.
- **Why and effect:** no extra GPU or cross-encoder on the server. Recall for topics rose from 60.4 % to 83.6 %. Questions without evidence are answered correctly with "no information" in 5 of 5 cases, stable over three runs.
- **Price:** one extra model call per topic question (about 1.5 seconds). The threshold slightly lowers recall for topics, because weakly supported pages are dropped.

## 6. Code builds the citation, not the language model

- **Problem:** when the model wrote the source line itself, it occasionally swapped or invented page numbers.
- **Decision:** the model only inserts numbered markers. Code validates them against the passages actually provided, removes them from the text and builds the source line from the catalogue.
- **Why and effect:** every cited page demonstrably comes from the retrieved passages. The web interface also separates the passages used in the answer from further hits.
- **Price:** some post-processing logic in the backend.

## 7. One planner call with Structured Outputs

- **Problem:** two consecutive model calls (location analysis and intent detection) added latency. Follow-up questions such as "Are there photos of it too?" were misrouted.
- **Decision:** a single call with a typed schema (Pydantic, OpenAI Structured Outputs) returns the standalone question, the intent, the search mode, the location, names and photo filters.
- **Why and effect:** half as many calls before the search, validated output without JSON parsing errors. Follow-up questions understood correctly rose from 13 of 15 to 15 of 15, intent detection to 40 of 40.
- **Price:** the schema has to be maintained when new capabilities are added.

## 8. Borderline questions go to the archive, not to a refusal

- **Problem:** the first intent router had an "off-topic" category to block irrelevant questions. It was too strict: instructions such as "Summarise that", small talk and legitimate questions on unusual subjects were rejected, and the conversation broke off abruptly. A wrong refusal is worse than a search that finds nothing, because the user never learns whether the archive has something to say.
- **Decision:** the broad off-topic category was removed, and everything that is not a photo request goes through the archive search. Once the relevance threshold was in place, a narrow off-topic flag returned in the planner. It only applies to requests that clearly have nothing to do with the village or the museum, such as a weather forecast or a poem. A question about what the archive says on a subject is always searched, however unlikely the subject.
- **Why and effect:** whether the archive has something to say is now decided by the relevance threshold, based on what was actually found, not by a guess in advance. The router only has to recognise the intent, not judge relevance. Intent detection, including clearly off-topic requests, reached 40 of 40; questions without evidence are answered with "no information" in 5 of 5 cases.
- **Price:** borderline questions cost a search and a reranker call, even when nothing is found.

## 9. A name register against typos and spelling variants

- **Problem:** a single typo in a name led nowhere, and historical sources spell the same farmstead or family in different ways.
- **Decision:** a language model extracted all names from texts and photo records once. They were consolidated into a register of 8,331 entries with variants, held in memory and matched with RapidFuzz.
- **Why:** typos are corrected before the search, and all known spellings are searched together, in under a millisecond.
- **Price:** the register needs a manual review now and then, and new documents have to be added to it.

## 10. Photo search without a generative model, and a small model for the rest

- **Problem:** a photo request needs a complete, predictable list, not a summary. For the text answers, the question was whether a larger model would be better.
- **Decision:** the photo search uses metadata filters, exact word conditions and hybrid search, but no generative model. It separates people shown in a picture from people who took it. All model calls use gpt-4o-mini.
- **Why and effect:** correct pictures among the top 10 rose from 29.9 % to 94.2 % (October 2026; with the later planner, a re-measurement in November gave 85.8 %, see [evaluation.md](evaluation.md)). In a comparison, the larger gpt-4o reached the same recall but answered only 2 of 5 questions without evidence correctly, compared with 4 of 5 for gpt-4o-mini at the time. The smaller model costs about one fifteenth per token (FinOps).
- **Price:** the photo search depends on the quality of the catalogue descriptions; pictures without descriptions cannot be found yet.

## 11. Conversation design: a museum guide, not a search index

- **Problem:** after the source lists were built by code, every answer, even the reply to a greeting, came with a wall of raw text passages. The chat felt like a technical search index rather than a conversation, and visitors were overwhelmed.
- **Decision:** answers are layered. Greetings are answered instantly, without a model call or database query. Knowledge questions get two to three readable paragraphs in the tone of a friendly museum guide, with one short source line underneath. In the web chat, the detailed passages sit in a collapsed panel, split into "used in the answer" and "further hits". The full wording of all passages is shown only when the user asks for it.
- **Why:** most visitors want an answer, researchers want the evidence. The layering serves both, without trading accuracy for readability.
- **Price:** additional intents and prompt rules, which the evaluation has to cover as well.

---

## Deutsche Fassung

# Die wichtigsten Designentscheide

Eine Auswahl von elf Entscheiden aus dem internen Entscheidungsprotokoll des Projekts (Architecture Decision Records). Jeder Eintrag nennt das Problem, den Entscheid, die Begründung und den Preis. Wo möglich wurde die Wirkung gemessen; die Zahlen stammen aus der Evaluation, die in [evaluation.md](evaluation.md) beschrieben ist.

## 1. Eigener Code statt Orchestrierungs-Framework

- **Problem:** Vektordatenbank, Sprachmodell und zwei Oberflächen müssen zusammenspielen. Frameworks wie LangChain oder LlamaIndex bieten das fertig an.
- **Entscheid:** eine dünne Schicht eigener Python-Code mit FastAPI und direkten Aufrufen der SDKs von Qdrant und OpenAI.
- **Begründung:** Der Umfang ist überschaubar (zwei Suchpfade, ein Modell). Ohne Framework ist jeder Schritt sichtbar, testbar und erklärbar, und es gibt weniger Abhängigkeiten, die sich unter dem Projekt verändern.
- **Preis:** mehr Code zum Schreiben und Pflegen, etwa für Wiederholungsversuche und Prompts.

## 2. Selbst betriebenes Qdrant statt Cloud-Suchdienst

- **Problem:** rund 1 GB Text und 1'800 Fotoeinträge bei wenigen Anfragen. Geprüft wurden verwaltete Cloud-Suchdienste und die PostgreSQL-Erweiterung pgvector.
- **Entscheid:** Qdrant in Docker auf dem eigenen Server.
- **Begründung:** eingebaute hybride Suche (dichte Vektoren, BM25, Reciprocal Rank Fusion) in einem Aufruf, tiefe Kosten und volle Kontrolle über die Daten. Mit pgvector hätte die Verbindung beider Sucharten von Hand gebaut werden müssen.
- **Preis:** ein Container mehr, der betrieben und aktualisiert werden muss.

## 3. Zwei Suchwege für Text: wortgenau bei Namen, hybrid bei Themen

- **Problem:** Wer nach einer Person, einem Hof oder einer Adresse fragt, erwartet Vollständigkeit und exakte Treffer. Offene Fragen («Wie war die Schule früher?») brauchen inhaltliche Ähnlichkeit. Eine einzige Suchmethode kann nicht beides gut.
- **Entscheid:** Der Planer ordnet jede Frage ein. Namen und Adressen gehen an eine wortgenaue Suche mit Wortgrenzen und Schreibweisen, Sachthemen an die hybride Suche.
- **Begründung und Wirkung:** Die Hausnummer 2 trifft nicht mehr die 20, und alle Stellen zu einem Namen werden gefunden. Die Trefferquote bei Namen und Adressen stieg von 86.6 % auf 98.7 %, die Präzision auf 93.5 %.
- **Preis:** Die Einordnung muss zuverlässig sein. Falsch eingeordnete Namensfragen fallen auf die hybride Suche zurück.

## 4. Text bereinigen und jedem Abschnitt Kontext geben

- **Problem:** Silbentrennungen am Zeilenende zerrissen Tausende Wörter, Absätze brachen am Seitenwechsel ab, und ein Abschnitt mitten aus einem Interview sagte nicht, wer erzählt.
- **Entscheid:** Silbentrennungen auflösen, Absätze über Seiten hinweg zusammenführen, Schriftenverzeichnisse ausschliessen und jedem Abschnitt eine Kopfzeile mit Werk, Kapitel oder Erzähler, Jahr und kurzer Zusammenfassung geben (Contextual Retrieval).
- **Begründung und Wirkung:** Die Bereinigung hob die Trefferquote bei Namen und Adressen von 77.3 % auf 86.6 %. Die Kopfzeile hob die Trefferquote bei Sachthemen von 48.5 % auf 57.6 %.
- **Preis:** längere Eingaben beim einmaligen Erstellen der Embeddings (Kosten unter zwei US-Cent für das ganze Archiv) und ein zusätzlicher Zwischenspeicher.

## 5. Ein Sprachmodell als Reranker, mit Relevanzschwelle

- **Problem:** Die hybride Suche fand die richtigen Seiten unter den ersten 35, setzte aber oft Einleitungen und Randseiten weiter oben an. Fragen ohne Antwort im Archiv erzeugten trotzdem eine Antwort.
- **Entscheid:** gpt-4o-mini bewertet bis zu 35 Kandidatenseiten auf einer Skala von 0 bis 3. Erreicht keine Seite die Schwelle, antwortet das System fest mit «keine Angaben», ohne Text zu erzeugen.
- **Begründung und Wirkung:** keine zusätzliche GPU und kein Cross-Encoder auf dem Server. Die Trefferquote bei Sachthemen stieg von 60.4 % auf 83.6 %. Fragen ohne Beleg werden in 5 von 5 Fällen korrekt mit «keine Angaben» beantwortet, stabil über drei Läufe.
- **Preis:** ein zusätzlicher Modellaufruf pro Sachfrage (rund 1.5 Sekunden). Die Schwelle senkt die Trefferquote bei Sachthemen leicht, weil schwach belegte Seiten wegfallen.

## 6. Der Code baut die Quellenangabe, nicht das Sprachmodell

- **Problem:** Wenn das Modell die Quellenzeile selbst schrieb, vertauschte oder erfand es gelegentlich Seitenzahlen.
- **Entscheid:** Das Modell setzt nur nummerierte Marker. Der Code prüft sie gegen die tatsächlich übergebenen Textstellen, entfernt sie aus dem Text und baut die Quellenzeile aus dem Katalog.
- **Begründung und Wirkung:** Jede zitierte Seite stammt nachweislich aus den gefundenen Textstellen. Die Web-Oberfläche trennt zudem die in der Antwort verwendeten Stellen von weiteren Fundstellen.
- **Preis:** etwas Nachbearbeitungslogik im Backend.

## 7. Ein einziger Planer-Aufruf mit Structured Outputs

- **Problem:** Zwei Modellaufrufe nacheinander (Ortsanalyse und Absichtserkennung) kosteten Zeit. Folgefragen wie «Gibt es dazu auch Fotos?» landeten im falschen Suchpfad.
- **Entscheid:** Ein einziger Aufruf mit einem typisierten Schema (Pydantic, OpenAI Structured Outputs) liefert die eigenständige Frage, die Absicht, den Suchmodus, den Ort, Namen und Fotofilter.
- **Begründung und Wirkung:** halb so viele Aufrufe vor der Suche, geprüfte Ausgaben ohne JSON-Fehler. Richtig verstandene Folgefragen stiegen von 13 von 15 auf 15 von 15, die Absichtserkennung auf 40 von 40.
- **Preis:** Das Schema muss bei neuen Fähigkeiten nachgeführt werden.

## 8. Grenzfälle gehen ans Archiv, nicht an eine Absage

- **Problem:** Der erste Intent Router hatte eine Kategorie «themenfremd», um irrelevante Fragen abzuweisen. Sie war zu streng: Anweisungen wie «Fasse das zusammen», Smalltalk und berechtigte Fragen zu ungewöhnlichen Themen wurden abgelehnt, und das Gespräch brach abrupt ab. Eine falsche Absage ist schlimmer als eine Suche, die nichts findet, denn der Nutzer erfährt nie, ob das Archiv etwas dazu enthält.
- **Entscheid:** Die breite Kategorie «themenfremd» wurde gestrichen, und alles, was keine Fotoanfrage ist, läuft über die Archivsuche. Als die Relevanzschwelle stand, kam im Planer eine schmale Markierung für Themenfremdes zurück. Sie greift nur bei Anfragen, die eindeutig nichts mit dem Dorf oder dem Museum zu tun haben, etwa eine Wettervorhersage oder ein Gedicht. Eine Frage, was im Archiv zu einem Thema steht, wird immer gesucht, auch wenn das Thema unwahrscheinlich ist.
- **Begründung und Wirkung:** Ob das Archiv etwas zu sagen hat, entscheidet jetzt die Relevanzschwelle, gestützt auf das tatsächlich Gefundene, nicht eine Vermutung im Voraus. Der Router muss nur noch die Absicht erkennen, nicht die Relevanz beurteilen. Die Absichtserkennung erreichte 40 von 40, eindeutig themenfremde Anfragen eingeschlossen; Fragen ohne Beleg werden in 5 von 5 Fällen mit «keine Angaben» beantwortet.
- **Preis:** Grenzfälle kosten eine Suche und einen Reranker-Aufruf, auch wenn nichts gefunden wird.

## 9. Ein Namensregister gegen Tippfehler und Schreibvarianten

- **Problem:** Schon ein Tippfehler im Namen führte ins Leere, und historische Quellen schreiben denselben Hof oder dieselbe Familie unterschiedlich.
- **Entscheid:** Ein Sprachmodell hat einmalig alle Namen aus Texten und Fotoeinträgen gezogen. Daraus entstand ein Register mit 8'331 Einträgen und ihren Schreibweisen, das im Arbeitsspeicher liegt und mit RapidFuzz abgeglichen wird.
- **Begründung:** Tippfehler werden vor der Suche korrigiert, und alle bekannten Schreibweisen werden gemeinsam gesucht, in weniger als einer Millisekunde.
- **Preis:** Das Register braucht ab und zu eine Durchsicht von Hand, und neue Dokumente müssen eingepflegt werden.

## 10. Fotosuche ohne generatives Modell, und ein kleines Modell für den Rest

- **Problem:** Eine Fotoanfrage braucht eine vollständige, vorhersehbare Liste, keine Zusammenfassung. Bei den Textantworten stellte sich die Frage, ob ein grösseres Modell besser wäre.
- **Entscheid:** Die Fotosuche nutzt Metadaten-Filter, wortgenaue Bedingungen und hybride Suche, aber kein generatives Modell. Sie trennt abgebildete Personen von Fotografierenden. Alle Modellaufrufe nutzen gpt-4o-mini.
- **Begründung und Wirkung:** Richtige Bilder unter den ersten zehn stiegen von 29.9 % auf 94.2 % (Oktober 2026; mit dem späteren Planer ergab eine Nachmessung im November 85.8 %, siehe [evaluation.md](evaluation.md#deutsche-fassung)). Im Vergleich erreichte das grössere gpt-4o dieselbe Trefferquote, beantwortete aber nur 2 von 5 Fragen ohne Beleg korrekt, gpt-4o-mini damals 4 von 5. Das kleinere Modell kostet pro Token rund ein Fünfzehntel (FinOps).
- **Preis:** Die Fotosuche hängt von der Qualität der Katalogbeschreibungen ab; Bilder ohne Beschreibung lassen sich noch nicht finden.

## 11. Conversation Design: ein Museumsführer, kein Suchindex

- **Problem:** Nachdem der Code die Fundstellenlisten baute, hing an jeder Antwort, sogar an der Antwort auf eine Begrüssung, eine Wand aus rohen Textauszügen. Der Chat wirkte wie ein technischer Suchindex statt wie ein Gespräch, und Besucherinnen und Besucher wurden erschlagen.
- **Entscheid:** Die Antworten sind gestuft. Begrüssungen beantwortet das System sofort, ohne Modellaufruf und ohne Datenbankabfrage. Wissensfragen erhalten zwei bis drei gut lesbare Absätze im Ton eines freundlichen Museumsführers, darunter eine kurze Quellenzeile. Im Web-Chat liegen die detaillierten Fundstellen in einem zugeklappten Bereich, getrennt nach «in der Antwort verwendet» und «weitere Fundstellen». Den vollständigen Wortlaut aller Stellen zeigt das System erst, wenn jemand ausdrücklich danach fragt.
- **Begründung:** Die meisten Besucher wollen eine Antwort, Forschende wollen die Belege. Die Stufung bedient beide, ohne Genauigkeit gegen Lesbarkeit zu tauschen.
- **Preis:** zusätzliche Absichten und Prompt-Regeln, die auch die Evaluation abdecken muss.
