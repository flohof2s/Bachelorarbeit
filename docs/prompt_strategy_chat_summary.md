# Zusammenfassung: Prompt-Strategie fuer die Produktextraktion

## Ausgangspunkt

Im aktuellen Projektstand waren die Prompts fuer die Transformation von bereinigtem Shop-HTML in strukturierte Lebensmittel-Produktdaten nur prototypisch formuliert. Ziel des Chats war es, daraus eine wissenschaftlich begruendete Prompt-Strategie abzuleiten, die spaeter als Grundlage fuer den finalen Gesamtprompt dienen kann.

Als fachlicher Use-Case wurde die Extraktion von Produktdaten aus vorverarbeitetem HTML betrachtet. Das Zielschema enthaelt unter anderem Produktkopfdaten, Zutaten, Naehrwerte, Produktgroessen und Produktgruppen.

## Arbeitsschritt 1: Sichtung der Projekt- und Literaturgrundlage

Zuerst wurden die bereitgestellten Artefakte betrachtet:

- `projekt(12).zip`
- `Expose_Lohoff(7).pdf`
- `KonzeptMatrix(4).xlsx`

Aus dem Projektstand wurden insbesondere die bestehenden Prompt-Dateien und das Zielmodell herangezogen. Relevant waren dabei vor allem:

- `core/prompts/transformation_extraction/v2/system.prompt.md`
- `core/prompts/transformation_extraction/v2/input.prompt.md`
- `core/models/extraction.py`
- `core/models/product.py`
- `core/models/nutrition.py`

Dabei wurde festgestellt, dass die technische Grundlage fuer eine schema-gefuehrte Extraktion bereits vorhanden ist:

- Prompt-Versionierung
- getrennte System- und Input-Prompts
- Pydantic-Zielmodell
- JSON-Ausgabeformat
- Schema-Validierungsfilter
- Batch-Verarbeitung ueber den LLM-Adapter

Ein technischer Hinweis war, dass beim Produkt-Matching bereits `temperature=0.0` gesetzt war, bei der eigentlichen Transformation aber nur `response_mime_type="application/json"`. Fuer eine wissenschaftlich saubere und reproduzierbare Strategie sollte auch die Transformation deterministisch konfiguriert werden.

## Arbeitsschritt 2: Auswahl geeigneter Prompt-Strategien

Auf Basis der Literatur und des konkreten Use-Cases wurden mehrere Strategien bewertet:

| Strategie | Bewertung fuer den Use-Case |
|---|---|
| Schema-first / Constrained JSON Prompting | Sehr gut geeignet als Basisstrategie |
| Prompt Patterns + Chaining | Sehr gut geeignet fuer komplexe Extraktionsfelder |
| Few-Shot Prompting | Geeignet zur Stabilisierung mehrdeutiger Felder |
| Iterative Refinement / Self-Check | Optional sinnvoll, aber teurer |
| Chain of Thought / Tree of Thoughts | Nur bedingt geeignet, eher fuer Spezialfaelle |
| LLM-Ensemble | Interessant, aber fuer den Bachelor-Prototyp eher optional |

Als Hauptstrategie wurde eine Kombination aus drei Ansaetzen empfohlen:

## Formulierte Gesamtstrategie

**Schema-Guided Prompt Pattern Chaining with Few-Shot Anchoring**

Die Strategie kombiniert:

1. **Schema-first / Constrained JSON Prompting**  
   Das Zielmodell wird explizit als JSON-Struktur vorgegeben. Das Modell darf keine zusaetzlichen Felder erzeugen und muss fuer unsichere Werte `null` beziehungsweise `[]` verwenden.

2. **Prompt Patterns + Chaining**  
   Komplexe Extraktionsfelder werden nicht nur als einzelner Auftrag formuliert, sondern in eine logische Kette zerlegt: Quelle finden, Attribut erkennen, Wert normalisieren, Ergebnis validieren und schema-konform ausgeben.

3. **Few-Shot Prompting**  
   Fuer semantisch schwierige oder mehrdeutige Felder werden spaeter wenige konkrete Beispiele aus dem Ground-Truth-Datensatz eingefuegt.

Diese Strategie passt besonders gut, weil einfache Felder direkt extrahiert werden koennen, waehrend komplexe Felder wie Zutaten, Naehrwerte und Produktgroessen mehrstufige Verarbeitung benoetigen.

## Arbeitsschritt 3: Wissenschaftliche Grundlage

Die wichtigsten wissenschaftlichen Grundlagen waren:

| Quelle | Rolle in der Strategie |
|---|---|
| Moundas, White & Schmidt: [Prompt Patterns for Structured Data Extraction from Unstructured Text](https://www.dre.vanderbilt.edu/~schmidt/PDF/Prompt_Patterns_for_Structured_Data_Extraction_from_Unstructured_Text.pdf) | Hauptgrundlage fuer Prompt Patterns, strukturierte Extraktion und Chaining |
| Vijayan: [A Prompt Engineering Approach for Structured Data Extraction from Unstructured Text Using Conversational LLMs](https://doi.org/10.1145/3639631.3639663) | Grundlage fuer den Vergleich von Prompt-Varianten wie Standard Prompting, Persona Prompting, Chain of Thought und Iterative Refinement |
| Fang et al.: [LLM-Ensemble: Optimal Large Language Model Ensemble Method for E-commerce Product Attribute Value Extraction](https://arxiv.org/abs/2403.00863) | Domaenennahe Grundlage fuer E-Commerce-Produktattributextraktion |
| Yao et al.: [Tree of Thoughts: Deliberate Problem Solving with Large Language Models](https://arxiv.org/abs/2305.10601) | Wurde als theoretisch interessant, aber nicht als Hauptstrategie fuer die Standardextraktion bewertet |
| FoodIE: [A Rule-based Named-entity Recognition Method for Food Information Extraction](https://www.researchgate.net/publication/331233364_FoodIE_A_Rule-based_Named-entity_Recognition_Method_for_Food_Information_Extraction) | Domaenenbegruendung fuer die Komplexitaet von Lebensmittel- und Zutatenextraktion |

Die Literaturlage wurde als ausreichend eingeschaetzt, um eine wissenschaftlich vertretbare Prompt-Strategie fuer den Prototyp zu begruenden. Fuer eine starke empirische Aussage speziell zu Lebensmittelproduktseiten aus bereinigtem HTML muesste die Strategie jedoch spaeter anhand des eigenen Ground-Truth-Datensatzes evaluiert werden.

## Arbeitsschritt 4: Beispielhafte Vertiefung am Attribut `ingredients`

Das Attribut `ingredients` wurde als Beispiel fuer ein komplexes Feld ausgewaehlt. Es eignet sich besonders gut, weil es mehrere Verarbeitungsschritte benoetigt:

- Erkennen der Zutatenliste im HTML
- Extraktion des vollstaendigen Originaltexts
- Zerlegung in einzelne Zutaten
- Erhalt der Reihenfolge
- Erkennung von Prozentangaben
- Erkennung von Unterzutaten in Klammern
- Ausschluss von Spurenhinweisen
- Ausgabe als schema-konforme Liste

Daraus wurde abgeleitet, dass `ingredients` nicht nur per Zero-Shot-Prompting extrahiert werden sollte, sondern als **chained few-shot** Feld behandelt werden muss.

## Arbeitsschritt 5: Klassifikation aller Zielattribute

Anschliessend wurde das Zielschema in drei Strategiekategorien eingeteilt:

| Kategorie | Bedeutung |
|---|---|
| `zero shot` | Direkte Extraktion, wenn der Wert eindeutig im HTML steht |
| `few shot` | Semantisch mehrdeutige Felder, die Beispiele brauchen |
| `chained few shot` | Komplexe Felder, die mehrere Extraktions- und Normalisierungsschritte brauchen |

Die wichtigsten Einstufungen waren:

| Attributgruppe | Strategie |
|---|---|
| `domain_id` | zero shot |
| direkte Kopfdaten wie `product_or_service_title`, `ean`, `nutri_score` | zero shot |
| semantische Kopfdaten wie `brand_name`, `organization_name`, `is_bio_product`, `is_lactose_free`, `has_sweetener`, `is_product`, `is_valid` | few shot |
| `ingredients` | chained few shot |
| `nutrition_values` | chained few shot |
| `product_sizings` | chained few shot |
| `product_groups` | few shot |

Die wichtigste Schlussfolgerung war, dass der Gesamtprompt nicht fuer alle Felder gleich detailliert sein sollte. Ausfuehrliche Prompt-Bloecke sind vor allem fuer `ingredients`, `nutrition_values` und `product_sizings` noetig.

## Arbeitsschritt 6: Entwurf des ersten Gesamtprompts

Auf Basis der Strategie wurde ein erster Systemprompt entworfen. Die Struktur folgt dem zuvor abgeleiteten Aufbau:

1. Globaler Rollen- und Aufgabenblock
2. Eingabeformat
3. Ausgabeformat
4. erlaubte Einheiten
5. globale Extraktionsregeln
6. Zero-Shot-Felder
7. Few-Shot-Felder mit Platzhaltern
8. Chained-Few-Shot-Block fuer `ingredients`
9. Chained-Few-Shot-Block fuer `nutrition_values`
10. Chained-Few-Shot-Block fuer `product_sizings`
11. Abschlusspruefung vor JSON-Ausgabe

Fuer die Few-Shot-Beispiele wurden bewusst Platzhalter eingefuegt, damit spaeter echte Beispiele aus dem Ground-Truth-Datensatz eingesetzt werden koennen, zum Beispiel:

- `{{ FEW_SHOT_PRICE_INPUT }}`
- `{{ FEW_SHOT_PRICE_OUTPUT }}`
- `{{ FEW_SHOT_INGREDIENTS_SIMPLE_INPUT }}`
- `{{ FEW_SHOT_INGREDIENTS_SIMPLE_OUTPUT }}`
- `{{ FEW_SHOT_NUTRITION_TABLE_INPUT }}`
- `{{ FEW_SHOT_NUTRITION_TABLE_OUTPUT }}`
- `{{ FEW_SHOT_SIZING_SIMPLE_INPUT }}`
- `{{ FEW_SHOT_SIZING_SIMPLE_OUTPUT }}`

## Arbeitsschritt 7: Erstellung der Markdown-Datei fuer den Systemprompt

Der Systemprompt wurde anschliessend als Markdown-Datei erstellt:

- `system_prompt_transformation_extraction.md`

Die Datei enthaelt nur den eigentlichen Systemprompt, sodass sie spaeter direkt als `system.prompt.md` in die Prompt-Versionierung des Projekts uebernommen werden kann.

## Ergebnis

Als Ergebnis des Chats liegt nun eine wissenschaftlich begruendete Prompt-Grundlage vor:

- Eine klare Gesamtstrategie: **Schema-Guided Prompt Pattern Chaining with Few-Shot Anchoring**
- Eine Attributklassifikation nach notwendiger Prompt-Komplexitaet
- Ein erster vollstaendiger Systemprompt mit Platzhaltern fuer Few-Shot-Beispiele
- Eine separate Markdown-Datei fuer den Systemprompt

## Naechste sinnvolle Schritte

1. Ground-Truth-Beispiele fuer die Platzhalter auswaehlen
2. Prompt-Version `v3` oder `v4` im Projekt anlegen
3. `temperature=0.0` auch fuer die Transformation setzen
4. Prompt gegen einen kleinen Testdatensatz ausfuehren
5. Fehleranalyse fuer `ingredients`, `nutrition_values` und `product_sizings` durchfuehren
6. Few-Shot-Beispiele iterativ verbessern
7. Finalen Prompt gegen die Baseline evaluieren
