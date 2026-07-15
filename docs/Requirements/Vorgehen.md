# Methodischer Kontext (DSR nach Hevner)

Im Sinne von Hevner et al. (2004) dient das Requirements Engineering dazu, die Business Needs des Anwendungskontexts präzise zu erfassen. Du nutzt das bestehende Wissen (Knowledge Base, z. B. deine KonzeptMatrix zu LLM-Schwächen und Extraktionsmethoden), um Anforderungen an das neue Artefakt zu definieren.

## Konkrete Schritte im Requirements Engineering

### Schritt 1: Analyse des organisatorischen und technischen Kontextes (Basis: Starke, 2024)

- Aktion: Analysiere, wie die unstrukturierten Produktdaten bei snoopmedia ankommen (z. B. APIs, CSV-Dateien, Freitexte) und wie das Zielsystem aufgebaut ist.
- Fokus: Nutze dein hochgeladenes MongoDB-Schema (DB_Modell.jpg), um die strukturellen Zielvorgaben zu verstehen. Welche Pflichtfelder existieren? Welche Datentypen (Arrays, Embedded Documents, Strings) erwartet die MongoDB?

### Schritt 2: Definition der Datenqualitätsdimensionen (Basis: Wang & Strong, 1996)

- Aktion: Leite aus der Datenqualitäts-Literatur konkrete Anforderungen ab. Produktdaten im E-Commerce erfordern eine hohe semantische Korrektheit (Accuracy) und Vollständigkeit (Completeness).
- Fokus: Definiere, was "gute" transformierte Daten für snoopmedia bedeuten. Wenn das LLM beispielsweise eine Zutat oder ein Produktmerkmal falsch zuordnet, sinkt die Datenqualität.

### Schritt 3: Identifikation von LLM-spezifischen Restriktionen und Risiken (Basis: KonzeptMatrix)

- Aktion: Nutze die Erkenntnisse deiner Literaturmatrix (z. B. das Paper Application of AI in Information Extraction...), um Anforderungen zur Risikominimierung zu formulieren.
- Fokus: Da LLMs zu Halluzinationen neigen, nicht-deterministisch agieren und API-Kosten verursachen, musst du Anforderungen definieren, wie das System damit umgeht (z. B. "Das System muss eine deterministische Validierung der JSON-Ausgabe gegen das MongoDB-Schema vornehmen"). Zudem zeigt das FoodIE-Paper, dass oft zusammengesetzte Phrasen ("fettarme Milch") statt einzelner Tokens extrahiert werden müssen – das impliziert Anforderungen an die Granularität der Extraktion.
  Schritt 4: Spezifikation der funktionalen und nicht-funktionalen Anforderungen
- Aktion: Leite aus den Schritten 1–3 konkrete, messbare Anforderungen ab und strukturiere sie (z. B. nach der MoSCoW-Methode: Must, Should, Could, Won't).

## Liste der zu erarbeitenden Artefakte

Für deine Thesis (insbesondere für das Kapitel "Requirements Engineering" bzw. "Anforderungsanalyse") solltest du folgende Artefakte schriftlich und visuell ausarbeiten:

1. Systemkontext-Abgrenzung (Diagramm & Beschreibung)
   - Inhalt: Eine grafische Darstellung (nach Starke, 2024), die zeigt, wo die Grenzen deines Transformationskonzepts liegen.
   - Details: Wer/Was liefert die unstrukturierten Daten (Input)? Welche LLM-Schnittstelle (z. B. OpenAI API) wird aufgerufen? Wie werden die Daten in die MongoDB (DB_Modell.jpg) geschrieben?
2. Funktionaler Anforderungskatalog (Functional Requirements)
   Dies beschreibt, was das Transformationskonzept tun muss.
3. Nicht-Funktionaler Anforderungskatalog (Non-Functional Requirements / NFRs)
   Dies beschreibt die Qualitätsattribute (wie gut das System es tun muss, stark gestützt durch Wang & Strong, 1996 und deine Matrix)
4. Use Cases / User Stories (Anwendungsszenarien)
   - Inhalt: Konkrete textuelle Beschreibungen aus Sicht eines snoopmedia-Mitarbeiters oder eines angebundenen Systems.
   - Beispiel: "Als Content-Manager möchte ich eine unstrukturierte Hersteller-Beschreibung hochladen, damit das System mir automatisch ein valides, für die MongoDB strukturiertes BSON/JSON-Dokument ausgibt."
5. Daten-Mapping-Matrix (Die Brücke zur MongoDB)
   - Inhalt: Eine tabellarische Gegenüberstellung, die zeigt, wie unstrukturierte Informationseinheiten in das relationale/Dokumenten-Schema deiner MongoDB übersetzt werden sollen.
   - Details: Wenn dein DB_Modell.jpg beispielsweise ein Feld attributes: { brand: String, weight: Number } vorgibt, zeigt diese Matrix, wie das LLM aus dem Text "250g Packung von Firma XY" die Werte trennt und zuordnet.

- Abschließendes Dokument an Gerri + PM zur Abnahme
