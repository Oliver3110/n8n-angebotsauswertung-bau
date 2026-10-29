# Testnachweis: Angebotsauswertung Müller & Partner Bau

Testdaten: 15 fiktive Übungsangebote aus dem Kurs (10 PDF, 3 DOCX, 1 Scan, 1 Foto), Projekt „Neubau Wohnanlage A“, Gewerke Elektro, Sanitär, Trockenbau und Rohbau. Alle Läufe in n8n 2.43.7 (Docker), Testzeitraum 27. bis 29.10.2026.

[Zurück zur README](README.md)

## Übersicht

| Nr. | Test | Erwartung | Ergebnis | Status |
|---|---|---|---|---|
| T1 | Ordner lesen | Alle Dateien aus `/files/Müller_Angebote` werden gefunden | 15 von 15 Dateien, Liste im Prüfformular | ✅ |
| T2 | Freigabe „Nein“ | Nichts wird gesendet, Lauf wird protokolliert | Test läuft noch | ⏳ |
| T3 | Text gewinnen | PDF, DOCX und Bilder werden gelesen | 10 PDF und 3 DOCX immer gelesen. Scan und Foto per OCR, wenn Gemini verfügbar ist | ✅ |
| T4 | OCR-Ausfall (Fallback) | Lauf läuft weiter, Bericht nennt die nicht gelesene Datei | Gemini überlastet (Fehler 503): Lauf lief mit 14 von 15 Dateien weiter, Bericht wies auf nicht gelesene Dateien hin (Bericht vom 27.10.2026, 20:23). Die Namensliste war dabei noch fehlerhaft und wurde danach korrigiert, siehe unten. | ✅ |
| T5 | Groq-Limit | Lauf bricht nicht wegen Rate Limit ab | Vorher Abbruch beim 11. Angebot. Nach Staffelung (8 s Abstand, 60 s Pause) alle Läufe durchgelaufen | ✅ |
| T6 | Vollständiger Lauf | Positionen in Notion, Bericht in Notion und als Datei, Empfehlung vollständig | 15 von 15 Dateien ausgewertet, Bericht mit Empfehlung je Gewerk (Lauf vom 28.10.2026, 21:29) | ✅ |
| T7 | Protokoll bei „Ja“ | Eine Zeile mit Ergebnis „ausgewertet“ | Test läuft noch | ⏳ |
| T8 | RAG-Chat | Richtige Antworten mit Anbieter, Datei und Zitat | Mit `qwen3:8b` 3 von 3 richtig (siehe unten) | ✅ |

## Beispielergebnis: Vergleichsbericht (Auszug)

Datei `Angebotsvergleich_2026-10-28_2129.md`, Lauf mit 15 von 15 Dateien:

> **Empfehlung**
> Da die Angebote unterschiedliche Gewerke abdecken, sollten sie nach Fachbereich bewertet werden:
> - **Elektro:** Elektro Meier GmbH: detaillierte Aufschlüsselung, niedrigster Gesamtpreis (9.500,00 €)
> - **Sanitär:** Sanitär Profi GmbH: günstigster Gesamtpreis (9.000,00 €), 1-Jahres-Wartung enthalten
> - **Trockenbau:** Trockenbau Huber: 10.800,00 €, zusätzlich Schallschutz
>
> Die übrigen Angebote sind unvollständig, veraltet oder nicht direkt vergleichbar.

Prüfung gegen die Originale: Preise und Hinweise stimmen. Die Baufirma Meier (Foto, Datum 2021) und Musterbau (Scan, Datum 2022) wurden richtig als veraltet bzw. nicht passend markiert.

## Beispielergebnis: Protokoll

Datei `protokoll.csv`:

```
(wird nach den Testläufen ergänzt)
```

## Beispielergebnis: RAG-Chat

Abgleich mit den Original-Angeboten. Links das erste Modell, rechts das verbesserte Setup.

| Frage | `llama3.2:3b`, 4 Treffer | `qwen3:8b`, 8 Treffer, Zitatpflicht |
|---|---|---|
| Welche Sanitär-Angebote schließen Fliesenarbeiten aus? | 3 von 4 richtig, ein Anbieter erfunden (Sanitär Profi) | 3 von 3 richtig |
| Was kostet das Angebot von Elektro Stark? | Richtig: 13.750 €, 4 bis 6 Wochen, Datei `angebot_11.docx` | nicht erneut getestet |
| Welcher Anbieter rechnet nach Aufwand ab? | Falsch, drei falsche Anbieter | Richtig |
| Antwortzeit | ca. 20 s | ca. 60 bis 75 s |

**Antwort `qwen3:8b` auf „Welche Sanitär-Angebote schließen Fliesenarbeiten aus?“:**
> - Sanitär & Co KG, angebot_12.docx: „Hinweis: Fliesenarbeiten nicht Bestandteil dieses Angebots.“
> - Sanitärbetrieb Keller, angebot_7.pdf: „Nicht enthalten sind Fliesenarbeiten.“
> - Schmidt Sanitärtechnik, angebot_2.pdf: „Hinweise: Ohne Fliesenarbeiten.“

**Antwort `qwen3:8b` auf „Welcher Anbieter rechnet nach Aufwand ab?“:**
> Bauleistungen Richter, angebot_9.pdf: „Eine genaue Abrechnung erfolgt nach tatsächlichem Aufwand.“

## Gefundene und behobene Fehler

| Fehler | Ursache | Lösung |
|---|---|---|
| OCR-Modell nicht gefunden | `gemini-2.5-flash` für den Zugang nicht mehr verfügbar | Modelltest, Wechsel auf `gemini-3.5-flash` |
| Foto nur halb gelesen | Antwortlänge der OCR zu knapp (2.000 Tokens) | 8.000 Tokens |
| Liste „nicht gelesen“ widersprüchlich | Gemini gibt den Dateinamen nicht zurück | Node „OCR-Dateinamen ergänzen“ |
| Groq lehnt Antwort ab | Schema erlaubte keinen leeren Preis | Schema erlaubt `null` |
| Empfehlung abgeschnitten | Antwortlänge 3.000 Tokens | 4.000 Tokens, Vorgabe kurzer Bericht |
| RAG-Chat erfindet Anbieter | Kleines Modell `llama3.2:3b` | `qwen3:8b`, Zitatpflicht, 8 Treffer |
