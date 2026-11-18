# Legal considerations for AI projects with archive material

**English** · [Deutsch](#deutsche-fassung)

> A German version follows below. · Eine deutsche Fassung folgt weiter unten.

> **Disclaimer:** This page is meant as orientation for anyone planning a similar project. It refers to Swiss law as of November 2026, is not exhaustive and is not legal advice. Before making such a tool public, have your specific case reviewed by a specialist.

Making an archive searchable with AI changes how its content can be found. A booklet that had to be picked up in the museum becomes a search that finds every name, address and photo within seconds. That raises legal questions which do not arise for the printed original.

## What needs to be clarified before going public

- **Living persons:** names, addresses, properties and personal details are personal data. Publishing them online generally requires consent or another justification under data protection law.
- **Deceased persons:** data protection no longer applies, but relatives can defend the memory of a family member.
- **Pictures of people:** recognisable persons have a right to their own image. Whether a photo may be shown depends on the individual case.
- **Copyright:** authors, photographers and publishers hold rights to their texts and pictures, often for decades. Owning a booklet or a photo does not mean owning these rights.
- **From print to digital:** someone who agreed to appear in a printed publication has not automatically agreed to a digital search with a much larger reach.

These questions have to be checked for each publication and each photo collection before a tool is opened to the public. Depending on the result, content may need consent, may only be shown in part or may have to stay out of the index.

## What applies in any case

These points apply as soon as a system is in operation, even during testing:

- **Language model abroad:** questions and the matching text passages are sent to the language model provider, often in the USA. Use an API whose terms exclude training on your data, check whether the provider offers adequate data protection safeguards, send only what the answer needs and say so in the privacy notice.
- **Logs:** questions typed into a chat can themselves contain names. Store as little as possible, set a fixed retention period with automatic deletion and describe the log in the privacy notice.
- **Transparency:** users should know that an AI is answering. The EU AI Act requires this for chatbots from August 2026, and it is good practice in any case.
- **Contact for objections:** there should be a simple way to object to a passage or photo and have it removed.

## How this project handles it

The chat stays closed until the rights have been clarified with the society that owns the collection. The chat already shows an AI notice, and the log deletes question texts after 90 days. Before it opens, rights and consent are checked for each publication and photo collection. The language model is used via an API that does not train on submitted data, and the privacy notice of the project page describes the data flows.

## Further reading

- [Federal Act on Data Protection (FADP)](https://www.fedlex.admin.ch/eli/cc/2022/491/en)
- [Copyright Act](https://www.fedlex.admin.ch/eli/cc/1993/1798_1798_1798/en)
- [Swiss Civil Code](https://www.fedlex.admin.ch/eli/cc/24/233_245_233/en) (protection of personality, Art. 28)
- FDPIC: [Data protection in genealogy](https://www.edoeb.admin.ch/dam/edoeb/de/Dokumente/aDSG/Datenschutz%20in%20der%20Familienforschung_DE.pdf.download.pdf/Datenschutz%20in%20der%20Familienforschung_DE.pdf) (German)
- [EU AI Act](https://eur-lex.europa.eu/eli/reg/2024/1689/oj) (transparency obligations, Art. 50)

---

## Deutsche Fassung

# Rechtliche Punkte bei KI-Projekten mit Archivmaterial

> **Hinweis:** Diese Seite dient als Orientierung für alle, die ein ähnliches Projekt planen. Sie bezieht sich auf Schweizer Recht mit Stand November 2026, ist nicht abschliessend und keine Rechtsberatung. Bevor ein solches Werkzeug öffentlich wird, sollte eine Fachperson den konkreten Fall prüfen.

Wer ein Archiv mit KI durchsuchbar macht, verändert, wie man seine Inhalte findet. Aus einem Heft, das man im Museum in die Hand nehmen musste, wird eine Suche, die jeden Namen, jede Adresse und jedes Foto in Sekunden findet. Damit stellen sich rechtliche Fragen, die sich beim gedruckten Original nicht stellen.

## Was vor einer Veröffentlichung zu klären ist

- **Lebende Personen:** Namen, Adressen, Liegenschaften und persönliche Angaben sind Personendaten. Wer sie online veröffentlicht, braucht in der Regel eine Einwilligung oder einen anderen Rechtfertigungsgrund nach Datenschutzrecht.
- **Verstorbene Personen:** Der Datenschutz gilt nicht mehr, aber Angehörige können das Andenken an ein Familienmitglied verteidigen.
- **Bilder von Menschen:** Erkennbare Personen haben ein Recht am eigenen Bild. Ob ein Foto gezeigt werden darf, hängt vom Einzelfall ab.
- **Urheberrecht:** Autorinnen, Fotografen und Verlage haben Rechte an ihren Texten und Bildern, oft für Jahrzehnte. Wer ein Heft oder ein Foto besitzt, hat damit nicht diese Rechte.
- **Vom Druck zum Digitalen:** Wer eingewilligt hat, in einer gedruckten Schrift vorzukommen, hat damit nicht automatisch einer digitalen Suche mit viel grösserer Reichweite zugestimmt.

Diese Fragen müssen für jede Schrift und jeden Fotobestand geprüft werden, bevor ein Werkzeug öffentlich zugänglich wird. Je nach Ergebnis braucht es eine Einwilligung, Inhalte werden nur teilweise gezeigt oder bleiben ganz draussen.

## Was in jedem Fall gilt

Diese Punkte gelten, sobald ein System in Betrieb ist, auch schon im Test:

- **Sprachmodell im Ausland:** Fragen und die passenden Textstellen gehen an den Anbieter des Sprachmodells, oft in den USA. Eine API wählen, deren Bedingungen das Training mit den eigenen Daten ausschliessen, prüfen, ob der Anbieter angemessene Datenschutzgarantien bietet, nur übermitteln, was die Antwort braucht, und das in der Datenschutzerklärung offenlegen.
- **Protokolle:** Fragen im Chat können selbst Namen enthalten. So wenig wie möglich speichern, eine feste Aufbewahrungsfrist mit automatischer Löschung festlegen und das Protokoll in der Datenschutzerklärung beschreiben.
- **Transparenz:** Nutzerinnen und Nutzer sollen wissen, dass eine KI antwortet. Der EU AI Act verlangt das für Chatbots ab August 2026, und gute Praxis ist es ohnehin.
- **Kontakt für Widerspruch:** Es braucht einen einfachen Weg, um einer Textstelle oder einem Foto zu widersprechen und es entfernen zu lassen.

## Wie dieses Projekt damit umgeht

Der Chat bleibt geschlossen, bis die Rechte mit dem Verein geklärt sind, dem der Bestand gehört. Der Chat zeigt bereits einen KI-Hinweis, und das Protokoll löscht Fragetexte nach 90 Tagen. Vor der Öffnung werden Rechte und Einwilligungen für jede Schrift und jeden Fotobestand geprüft. Das Sprachmodell läuft über eine API, die nicht mit den übermittelten Daten trainiert, und die Datenschutzerklärung der Projektseite beschreibt die Datenflüsse.

## Weiterführend

- [Datenschutzgesetz (DSG)](https://www.fedlex.admin.ch/eli/cc/2022/491/de)
- [Urheberrechtsgesetz (URG)](https://www.fedlex.admin.ch/eli/cc/1993/1798_1798_1798/de)
- [Zivilgesetzbuch (ZGB)](https://www.fedlex.admin.ch/eli/cc/24/233_245_233/de) (Persönlichkeitsschutz, Art. 28)
- EDÖB: [Datenschutz in der Familienforschung](https://www.edoeb.admin.ch/dam/edoeb/de/Dokumente/aDSG/Datenschutz%20in%20der%20Familienforschung_DE.pdf.download.pdf/Datenschutz%20in%20der%20Familienforschung_DE.pdf)
- [EU AI Act](https://eur-lex.europa.eu/eli/reg/2024/1689/oj) (Transparenzpflichten, Art. 50)
