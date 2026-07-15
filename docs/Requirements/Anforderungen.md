# Funktionale Anforderungen:

## Phase 1: Input & Vorverarbeitung

    - FA 1.1 (HTML-Cleaning): Das System muss HTML-Rohtexte von unrelevantem Ballast (Scripts, Footer, Navigationsleisten) befreien, um Token-Kosten zu sparen und das „Rauschen“ für das LLM zu minimieren.
    - FA 1.2 (Batching): Das System muss in der Lage sein, Datensätze sequenziell oder in konfigurierbaren Batches zu verarbeiten, um API-Rate-Limits nicht zu verletzen.

## Phase 2: Extraktion & LLM-Steuerung (Basis: KonzeptMatrix.xlsx)

    - FA 2.1 (JSON-Contract): Das System muss das LLM (Gemini API) über strukturierte Prompts (z. B. via Structured Outputs / JSON-Mode) dazu zwingen, ausschließlich ein valides, vordefiniertes JSON-Format zurückzugeben.
    - FA 2.2 (Hierarchische Extraktion): Das System muss komplexe, geschachtelte und unstrukturierte Produktstrukturen aus den Webdaten auflösen und in flache oder sauber strukturierte Objekte überführen.

## Phase 3: Normalisierung & Regelbasierter Vergleich

    - FA 3.1 (Einheiten-Normalisierung): Nach der Überführung in strukturierte Daten müssen alle extrahierten Größen- und Mengenangaben (z. B. KG, L, g, ml) in standardisierte Basiseinheiten transformiert werden. Jedes Gewicht muss in Gramm (g) und jede Flüssigkeitsmenge in Milliliter (ml) normalisiert werden (z. B. Typ-Konvertierung von „Fett: 12g“ in fats.value: 12 und fats.unit: "g").
    - FA 3.2 (Regelbasierte Extraktion & Evaluation): In diesem Modul muss zusätzlich ein rein regelbasierter Ansatz (z. B. über reguläre Ausdrücke/Heuristiken analog zu klassischen Wrapper-Verfahren oder Named-Entity-Recognition-Ansätzen wie FoodIE) zur Extraktion und Normalisierung von Verpackungsgrößen implementiert werden. Dieser dient als deterministischer Vergleichsmaßstab, um die Güte der LLM-Extraktion zu evaluieren.

## Phase 4: Produkt-Matching & API-Übertragung

    - FA 4.1 (KI-basiertes Produkt-Matching): Nach der Transformation und Normalisierung müssen die Produktdaten mit dem bereits existierenden Datenbestand abgeglichen werden (z. B. Prüfung: „Existiert diese Cola-Flasche bereits in der DB?“). Dies soll KI-gestützt (z. B. mittels Few-Shot-Prompting) geschehen, um redundante Datensätze zu vermeiden.
    - FA 4.2 (API-Datenübergabe): Das bereinigte, normalisierte und abgeglichene Produktobjekt darf nicht direkt in die Datenbank geschrieben werden. Das System muss die strukturierten Daten stattdessen an eine bereitgestellte API übergeben, welche die finale Validierung und Persistierung übernimmt.

# Nicht-Funktionale Anforderungen

    - NFR 1 (Robustheit / Schema-Compliance): Mindestens 95 % der vom System aufbereiteten JSON-Objekte müssen ohne strukturellen Fehler direkt vom Schema der Ziel-API akzeptiert werden.
    - NFR 2 (Kostenkontrolle & Wirtschaftlichkeit): Die durchschnittlichen API-Kosten der Gemini-Aufrufe pro transformierter Produktseite dürfen einen Betrag von X Cent (z. B. 1–2 Cent) nicht überschreiten.

# Technische Anforderungen

    - TA 1 (Programmiersprache): Das gesamte System sowie das Normalisierungs-Modul müssen vollständig in Python implementiert werden.
    - TA 2 (Schnittstellen-Architektur): Die Datenübergabe erfolgt ausschließlich über eine vorgegebene REST- oder GraphQL-API. Das zu entwickelnde Modul besitzt keine direkten Schreibrechte oder direkten Treiberverbindungen (wie z. B. native pymongo-Verbindungen) zur produktiven MongoDB.

# Rahmenbedingungen

    - RA 1 (LLM-Infrastruktur): Als Large Language Model zur Extraktion und zum Produkt-Matching muss die Gemini API verwendet werden.

- Externer Zugang für Herr Alda? Inwiefern muss der Code mitgeliefert weerden?
