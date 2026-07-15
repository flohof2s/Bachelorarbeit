# Phase 1 – Input & Vorverarbeitung

## US 1.1: HTML-Cleaning zur Rausch- und KostenreduktionDetails

Als Daten-Engineer möchte ich unstrukturierte HTML-Rohtexte automatisch von irrelevanten Elementen (wie Scripts, Footern und Navigationsleisten) befreien, um das semantische Rauschen für das LLM zu minimieren und Token-Kosten zu sparen.
Akzeptanzkriterien:

- Ein Python-Skript entfernt nachweislich alle <script>-, <style>-, <footer>- und <nav>-Tags aus dem HTML-Body.
- Der verbleibende Text enthält ausschließlich die primären Produktinformationen (z. B. Name, Zutaten, Nährwerttabellen).Die durchschnittliche Textmenge (Zeichenanzahl) wird im Vergleich zum Roh-HTML um mindestens 40 % reduziert.

## US 1.2: Konfigurierbare Batch-Verarbeitung gegen Rate-LimitsDetails

Als System-Administrator möchte ich Datensätze sequenziell oder in konfigurierbaren Batches verarbeiten lassen können, um die API-Rate-Limits der Gemini API im Betrieb nicht zu verletzen.Akzeptanzkriterien:

- Die Batch-Größe (Anzahl der Produktseiten pro Durchlauf) sowie eine Verzögerungszeit (Delay) sind über eine zentrale Konfigurationsdatei (z. B. config.yaml) steuerbar.
- Das System fängt 429 (Too Many Requests)-Fehlermeldungen der Gemini API ab und reagiert mit einer automatischen Pausierung (z. B. Exponential Backoff).

# Phase 2 – Extraktion & LLM-Steuerung

## US 2.1: Strukturierte und hierarchische LLM-DatenextraktionDetails

Als Daten-Analyst möchte ich die Gemini API über strukturierte Prompts so steuern, dass komplexe und geschachtelte Produktstrukturen aufgelöst und zwingend in einem vordefinierten JSON-Format zurückgegeben werden.
Akzeptanzkriterien:

- Die Extraktion nutzt das native Gemini-Feature für Structured Outputs (bzw. den JSON-Mode).
- Das ausgegebene JSON entspricht exakt den strukturellen Vorgaben.
- Komplexe, geschachtelte Produktinformationen werden fehlerfrei in flache oder sauber hierarchisch strukturierte JSON-Objekte überführt.
- Wirtschaftlichkeit (NFR 2): Die durchschnittlichen API-Kosten der Gemini-Aufrufe liegen stabil bei unter 1–2 Cent pro transformierter Produktseite.

# Phase 3 – Normalisierung & Regelbasierter Vergleich (Das Extra-Modul)

## US 3.1: Einheiten-Normalisierung für MengenangabenDetails

Als Backend-Entwickler möchte ich ein isoliertes Normalisierungs-Modul in Python implementieren, das alle extrahierten Größen- und Mengenangaben in standardisierte Basiseinheiten (g und ml) konvertiert, um eine konsistente Datenbasis zu garantieren.
Akzeptanzkriterien:

- Das Modul fängt Strings wie "12g", "0.5 kg" oder "1.5 Liter" ab und splittet diese deterministisch in numerische Werte und Einheiten.
- Gewichte werden mathematisch korrekt in Gramm (g) und Flüssigkeiten in Milliliter (ml) umgerechnet (z. B. 0.5 kg => value: 500, unit: "g").
- Das Modul ist als eigenständige Pipeline-Komponente ohne direkte Datenbankanbindung realisiert (TA 1).

## US 3.2: Regelbasierte Verpackungsgrößen-Extraktion als BaselineDetails

Als Bachelorand möchte ich innerhalb des Extra-Moduls einen rein regelbasierten Ansatz (z. B. via Regex oder Heuristiken) zur Extraktion von Verpackungsgrößen entwickeln, um eine verlässliche Baseline für den wissenschaftlichen Vergleich mit dem LLM zu haben.
Akzeptanzkriterien:

- Verpackungsgrößen werden parallel zur LLM-Extraktion mittels regulärer Ausdrücke (Regex) oder Mustern (analog zu FoodIE) ausgewertet.
- Das Modul gibt die regelbasierten Ergebnisse getrennt aus, sodass ein direkter Vergleich von Precision, Recall und F1-Score in der Thesis ermöglicht wird.

# Phase 4 – Produkt-Matching & API-Übertragung

## US 4.1: KI-gestütztes Produkt-Matching zur DuplikatsvermeidungDetails

Als Daten-Engineer möchte ich, dass das System die transformierten Produktdaten mittels Gemini API (Few-Shot-Prompting) gegen den bestehenden Datenbestand abgleicht, um die Entstehung von redundanten Datensätzen (z. B. doppelte Cola-Flaschen) zu verhindern.
Akzeptanzkriterien:

- Das System sendet einen Few-Shot-Prompt an die Gemini API, um die Ähnlichkeit zwischen dem neu gecrawlten Produkt und bestehenden Produkten semantisch zu bewerten.
- Die für den Vergleich notwendigen Bestandsdaten werden dynamisch über die Ziel-API abgefragt.

## US 4.2: Entkoppelte Datenübergabe über die REST-APIDetails

Als Software-Architekt möchte ich, dass das fertige Produktobjekt ausschließlich an eine bereitgestellte REST- oder GraphQL-API übergeben wird, damit das System vollständig von der zugrundeliegenden MongoDB-Infrastruktur entkoppelt bleibt.
Akzeptanzkriterien:

- Die Transformations-Pipeline besitzt keinen direkten DB-Treiber (kein pymongo) und kommuniziert ausschließlich per HTTP-Post mit der Ziel-API (siehe ContextDiagram_2.png).
- Schema-Compliance (NFR 1): Mindestens 95 % der an die API übergebenen JSON-Objekte werden vom API-Schema fehlerfrei akzeptiert.
