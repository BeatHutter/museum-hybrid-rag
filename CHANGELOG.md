# Changelog

Alle wesentlichen Änderungen an diesem Repository. Das Format folgt [Keep a Changelog](https://keepachangelog.com/de/1.1.0/), die Versionsnummern folgen [Semantic Versioning](https://semver.org/lang/de/). Bis der Code vollständig veröffentlicht ist, bleiben die Versionen unter 1.0.0.

## [Unreleased]

### Geändert

- Projektseite, Datenschutzerklärung, README und Dokumentation: KI-Hinweis im Chat und Löschung der Fragetexte nach 90 Tagen sind umgesetzt (vorher als geplant beschrieben).

## [0.4.0] – 2026-11-18

### Hinzugefügt

- Dokumentation unter `documentation/`, jeweils Englisch und Deutsch in einer Datei: Architektur, elf wichtige Designentscheide, Evaluation (Testset, Messgrössen, Ergebnisse Schritt für Schritt, Grenzen) und eine Übersicht der rechtlichen Punkte bei KI-Projekten mit Archivmaterial.
- Projektseite: Links auf die Evaluation und die rechtliche Übersicht.

### Geändert

- Projektseite: Grundsatz 5 nennt die Regeln vor der Öffnung (Persönlichkeits- und Urheberrechte), Roadmap entsprechend nachgeführt; keine Zahl zum Rate Limiting mehr.
- README: Übersicht der Dokumente im Repository.

### Korrigiert

- Umfang des Testsets: 98 statt 99 Testfälle (30 Textsuche, 40 Absichtserkennung, 13 Fotosuche, 15 Dialogschritte), auf der Projektseite und im README.

## [0.3.0] – 2026-11-17

### Hinzugefügt

- README: Abschnitt «Governance» mit den Grundsätzen des Projekts und eine Übersicht, welche Dateien im Repository liegen und was als Nächstes folgt.
- README: Messgrösse «Lücken bei mehrteiligen Fragen korrekt benannt» und Angabe zum Testset (99 Testfälle in vier Suiten).

### Geändert

- Projektseite auf Deutsch und Englisch neu aufgebaut: Projektpräsentation mit Stand 16.11.2026, Ablauf einer Suche von A bis Z, Ergebnisse vom ersten Wurf bis zum MVP, acht Governance-Grundsätze, Erkenntnisse und Roadmap.
- README: Status auf «MVP im Testbetrieb» nachgeführt; Ausgangslage nennt die Museumsdatenbank MuseumPro statt einer Excel-Liste.
- Protokoll korrekt als pseudonym statt anonym beschrieben (README und Datenschutzerklärung).
- Projektseite: neue Kontakt-Adresse im Fuss und auf den Datenschutzseiten.

## [0.2.0] – 2026-11-14

### Hinzugefügt

- Ausformuliertes README auf Englisch und Deutsch: Zweck, Funktionsweise, Messergebnisse, Technik.
- Lizenz (MIT) für den Code.
- Dieser Changelog.
- Hinweise zum Melden von Sicherheitslücken (`SECURITY.md`).

### Geändert

- Projektseite: Kennzahlen zur Evaluation an die gemessenen Werte angeglichen (99 Testfälle in vier Suiten; «5/5 Gegenfragen ohne erfundene Antwort» statt einer nicht gemessenen Prozentangabe).

## [0.1.0] – 2026-11-13

### Hinzugefügt

- Erste Veröffentlichung mit Platzhalter-README.
- Projektseite auf Deutsch und Englisch im Ordner `docs/`.
