# Evaluationsstrategie für den CrawlDataTransformer

## 1. Ziel der Evaluation

Die Evaluation untersucht, wie gut der LLM-basierte Extraktionspfad und der regelbasierte Verpackungsgrößen-Extractor das Referenzdatenfeld `packaging_size` aus unstrukturierten Produktdaten extrahieren.

Der regelbasierte Extractor dient dabei als deterministische Vergleichsbasis für ein ausgewähltes Datenfeld. Die Bewertung erfolgt nicht primär durch einen direkten Vergleich von LLM und Regelverfahren, sondern gegen einen manuell annotierten Ground-Truth-Datensatz.

Die zentrale Bewertungslogik lautet:

```text
LLM-Ergebnis        → Vergleich gegen Ground Truth
Regel-Ergebnis      → Vergleich gegen Ground Truth
LLM ↔ Regel         → zusätzlicher Diagnosevergleich
```

Der direkte Vergleich zwischen LLM und Regelverfahren wird nur zur Fehlerdiagnose verwendet. Er ersetzt keine Ground Truth, da beide Verfahren denselben falschen Wert liefern könnten.

## 2. Evaluationsgegenstand

Primär evaluiert wird ausschließlich das Referenzfeld:

```text
packaging_size
```

Für jeden Datensatz wird eine manuell annotierte Referenzgröße erwartet. Diese Ground Truth enthält mindestens:

```json
{
  "inputId": 1234,
  "packagingSize": {
    "value": 0.33,
    "unit": "l",
    "normalizedValue": 330,
    "normalizedUnit": "ml",
    "originalValue": "0,33L",
    "interpretation": "single_unit"
  },
  "status": "annotated",
  "uncertain": false
}
```

Nicht ausgewertet werden Datensätze mit:

```text
status = skipped
uncertain = true
```

Datensätze mit `status = not_found` können ausgewertet werden, wenn sie bewusst als Fälle ohne sichtbare Verpackungsgröße annotiert wurden.

## 3. Definition der korrekten Verpackungsgröße

Für diese Arbeit gilt als korrekte Größe:

```text
Die primäre Einzel-Verpackungsgröße des Produkts oder der ausgewählten Variante.
```

Beispiele:

| Produktangabe | Ground Truth |
|---|---|
| `0,33L Dose` | `0.33 l` bzw. `330 ml` |
| `0,5L PET` | `0.5 l` bzw. `500 ml` |
| `6 x 250 ml` | `250 ml` als Einzelgröße |
| `1 Tray mit 24 Dosen à 0,33L` | `0.33 l` bzw. `330 ml` als Einzelgröße |
| `500 g Packung` | `500 g` |

Die Gesamtmenge eines Multipacks kann optional gespeichert werden, ist aber nicht der primäre Vergleichswert, solange der regelbasierte Extractor nur eine Einzelgröße extrahiert.

## 4. Vergleichslogik ohne Akzeptanzbereich

Da es sich um Textextraktion handelt, wird **kein numerischer Akzeptanzbereich** verwendet. Ein Wert ist nur dann korrekt, wenn die normalisierte Einheit und der normalisierte Wert exakt übereinstimmen.

Die Vergleichsregel lautet:

```text
match(P, GT) =
    P.normalized_unit == GT.normalizedUnit
    UND
    P.normalized_value == GT.normalizedValue
```

Dabei gilt:

| Original | Normalisierte Form |
|---|---|
| `0.33 l` | `330 ml` |
| `0,33L` | `330 ml` |
| `500 g` | `500 g` |
| `1 kg` | `1000 g` |

Beispiel:

```text
Prediction: 0.33 l → 330 ml
Ground Truth: 330 ml
Ergebnis: korrekt
```

Gegenbeispiele:

```text
Prediction: 329 ml
Ground Truth: 330 ml
Ergebnis: falsch
```

```text
Prediction: 330 g
Ground Truth: 330 ml
Ergebnis: falsch
```

## 5. Grundbegriffe für die Metriken

Für jedes Verfahren `A` mit `A ∈ {LLM, RULE}` werden folgende Zählwerte gebildet:

| Symbol | Bedeutung |
|---|---|
| `GT` | Ground-Truth-Verpackungsgröße |
| `P_A` | Vorhersage des Verfahrens `A` |
| `present(x)` | Ein Wert ist vorhanden |
| `match(P_A, GT)` | Normalisierte Einheit und normalisierter Wert stimmen exakt überein |
| `N_eval` | Anzahl auswertbarer Datensätze |
| `TP` | True Positive |
| `FP` | False Positive |
| `FN` | False Negative |
| `TN` | True Negative |

Wichtig: Wenn Ground Truth vorhanden ist und ein Verfahren einen falschen Wert extrahiert, zählt der Fall sowohl als `FP` als auch als `FN`.

Beispiel:

```text
Ground Truth: 330 ml
Prediction: 100 g
```

Bewertung:

```text
FP: Das Verfahren hat einen falschen Wert extrahiert.
FN: Der korrekte Wert 330 ml wurde nicht extrahiert.
```

## 6. Metriken

### 6.1 Datensatz- und Ground-Truth-Metriken

| Metrik | Was wird gemessen? | Berechnung | Zweck | Quelle / Grundlage |
|---|---|---|---|---|
| Evaluationsumfang | Anzahl auswertbarer Datensätze | `N_eval = count(status in {annotated, not_found} and uncertain == false)` | Transparenz über die Stichprobe | Eigene Operationalisierung |
| Ground-Truth-Abdeckung | Anteil annotierter Datensätze | `N_annotated / N_total` | Zeigt, wie vollständig der Referenzdatensatz ist | Data-Quality-Dimension `Completeness`, vgl. ISO/IEC 25012 |
| Ausschlussquote | Anteil ausgeschlossener Datensätze | `(N_skipped + N_uncertain) / N_total` | Zeigt, wie viele Fälle nicht belastbar bewertet werden | Eigene Operationalisierung |
| Not-Found-Anteil | Anteil ohne sichtbare Größe | `N_not_found / N_eval` | Zeigt, wie oft keine Verpackungsgröße im Input vorhanden ist | Eigene Operationalisierung |

### 6.2 Technische LLM- und Pipeline-Metriken

| Metrik | Was wird gemessen? | Berechnung | Zweck | Quelle / Grundlage |
|---|---|---|---|---|
| LLM Parse Success Rate | Anteil maschinell lesbarer LLM-Antworten | `N_parseable_json / N_llm_responses` | Prüft, ob die LLM-Antwort als JSON verarbeitet werden kann | Eigene technische Metrik |
| Schema Valid Rate | Anteil schema-valider LLM-Antworten | `N_schema_valid / N_parseable_json` | Bewertet Schema-Konformität des LLM-Outputs | JSON Schema Validation |
| Schema Error Rate | Anteil schema-invalider Antworten | `N_schema_invalid / N_parseable_json` | Gegenmetrik zur Schema Valid Rate | JSON Schema Validation |
| Schema Error Distribution | Fehler nach Feld oder Fehlerart | `count(error_type) / N_schema_errors` | Identifiziert besonders fehleranfällige Schema-Bereiche | Eigene Aggregation der Validierungsfehler |
| Normalization Success Rate | Erfolgreich normalisierte Einheiten | `N_normalized_successfully / N_fields_to_normalize` | Prüft, ob Einheiten vergleichbar gemacht werden konnten | Eigene technische Metrik |
| Pipeline Success Rate | Erfolgreich abgeschlossene Requests | `N_successful_pipeline / N_eval` | Bewertet Stabilität des Gesamtablaufs | Eigene technische Metrik |

### 6.3 Extraktionsqualität gegen Ground Truth

Diese Metriken werden separat für `LLM` und `RULE` berechnet.

| Metrik | Was wird verglichen? | Berechnung | Zweck | Quelle / Grundlage |
|---|---|---|---|---|
| Prediction Coverage | Vorhersage vorhanden, wenn GT vorhanden ist | `count(present(P_A) and present(GT)) / count(present(GT))` | Zeigt, wie oft ein Verfahren überhaupt eine Größe liefert | Information Extraction / Completeness |
| True Positives | Korrekte Extraktionen | `TP = count(present(P_A) and present(GT) and match(P_A, GT))` | Basiswert für Precision, Recall und F1 | Klassische IE-/Klassifikationsmetrik |
| False Positives | Falsche Extraktionen | `FP = count(present(P_A) and (not present(GT) or not match(P_A, GT)))` | Misst falsche extrahierte Werte | Klassische IE-/Klassifikationsmetrik |
| False Negatives | Fehlende korrekte Extraktionen | `FN = count(present(GT) and (not present(P_A) or not match(P_A, GT)))` | Misst übersehene oder falsch ersetzte korrekte Werte | Klassische IE-/Klassifikationsmetrik |
| True Negatives | Korrekt nicht extrahiert | `TN = count(not present(GT) and not present(P_A))` | Relevant für `not_found`-Fälle | Klassische Klassifikationsmetrik |
| Precision | Zuverlässigkeit extrahierter Werte | `TP / (TP + FP)` | Beantwortet: Wie viele extrahierte Größen sind korrekt? | Precision nach Information Retrieval / Klassifikation |
| Recall | Vollständigkeit der Extraktion | `TP / (TP + FN)` | Beantwortet: Wie viele vorhandene Größen wurden korrekt gefunden? | Recall nach Information Retrieval / Klassifikation |
| F1 Score | Harmonisiertes Maß aus Precision und Recall | `2 * TP / (2 * TP + FP + FN)` | Gesamtvergleich bei unausgeglichenem Verhältnis von Fehlerarten | F1 Score |
| Normalized Exact Match Rate | Exakter Match nach Normalisierung | `count(match(P_A, GT)) / count(present(GT))` | Wichtigste fachliche Metrik für `packaging_size` | Eigene Operationalisierung |
| Field Accuracy | Anteil vollständig korrekt bewerteter Fälle | `(TP + TN) / N_eval` | Gesamtanteil korrekter Feldentscheidungen | Accuracy nach Klassifikation |
| Missing Rate | Anteil fehlender korrekter Werte | `FN / (TP + FN)` | Gegenmetrik zum Recall | Abgeleitet aus Recall |
| False Discovery Rate | Anteil falscher Extraktionen | `FP / (TP + FP)` | Gegenmetrik zur Precision | Abgeleitet aus Precision |
| Over-Extraction Rate | Extraktion trotz fehlender Ground Truth | `count(not present(GT) and present(P_A)) / count(not present(GT))` | Bewertet Verhalten bei Produkten ohne sichtbare Größe | Eigene Operationalisierung |

### 6.4 Einheiten- und Wertmetriken

Diese Metriken helfen zu verstehen, ob Fehler durch Einheit, Dimension oder Zahlenwert entstehen.

| Metrik | Was wird verglichen? | Berechnung | Zweck | Quelle / Grundlage |
|---|---|---|---|---|
| Unit Accuracy | Konkrete normalisierte Einheit | `count(P_A.normalized_unit == GT.normalizedUnit) / count(present(P_A) and present(GT))` | Prüft Fehler wie `g` statt `ml` | Eigene Operationalisierung |
| Dimension Accuracy | Dimension Masse oder Volumen | `count(dimension(P_A) == dimension(GT)) / count(present(P_A) and present(GT))` | Prüft, ob Masse und Volumen verwechselt werden | Eigene Operationalisierung |
| Value Accuracy | Normalisierter Zahlenwert | `count(P_A.normalized_value == GT.normalizedValue) / count(same_dimension(P_A, GT))` | Trennt Zahlenwertfehler von Einheitsfehlern | Eigene Operationalisierung |
| Unit Conversion Error Rate | Fehler durch falsche Normalisierung | `count(error_type == "conversion_error") / count(all_errors)` | Erkennt Fehler wie `0.33 l → 0.33 ml` | Eigene Fehlercodierung |

Auch hier gilt: Es gibt keinen Toleranzbereich. Der normalisierte Wert muss exakt übereinstimmen.

### 6.5 Diagnosevergleich zwischen LLM und regelbasiertem Extractor

Diese Metriken bewerten nicht direkt die Qualität, sondern erklären Übereinstimmungen und Unterschiede zwischen den Verfahren.

| Metrik | Was wird verglichen? | Berechnung | Zweck | Quelle / Grundlage |
|---|---|---|---|---|
| Agreement Rate | LLM-Vorhersage gegen Regel-Vorhersage | `count(same_prediction(P_LLM, P_RULE)) / N_eval` | Zeigt, wie häufig beide Verfahren denselben Wert liefern | Agreement-Metrik |
| Cohen's Kappa | Kategorisierte LLM- und Regel-Ausgaben | `κ = (p_o - p_e) / (1 - p_e)` | Chance-korrigierte Übereinstimmung | Cohen's Kappa |
| Outcome Matrix | Beide Verfahren gegen GT | Kategorien siehe unten | Zentrale Diagnose für die Diskussion | Eigene Operationalisierung |
| Delta Precision | LLM minus Rule | `Precision_LLM - Precision_RULE` | Zeigt relativen Vorteil beim Fehleranteil extrahierter Werte | Abgeleitet aus Precision |
| Delta Recall | LLM minus Rule | `Recall_LLM - Recall_RULE` | Zeigt relativen Vorteil bei Vollständigkeit | Abgeleitet aus Recall |
| Delta F1 | LLM minus Rule | `F1_LLM - F1_RULE` | Kompakter Gesamtvergleich | Abgeleitet aus F1 |

Die Outcome Matrix sollte folgende Kategorien enthalten:

| Kategorie | Bedeutung |
|---|---|
| `both_correct` | LLM und Rule stimmen beide mit Ground Truth überein |
| `llm_correct_rule_wrong` | Nur der LLM ist korrekt |
| `rule_correct_llm_wrong` | Nur der regelbasierte Extractor ist korrekt |
| `both_wrong_same` | Beide sind falsch, liefern aber denselben falschen Wert |
| `both_wrong_different` | Beide sind falsch und liefern unterschiedliche Werte |
| `llm_only` | Nur der LLM extrahiert einen Wert |
| `rule_only` | Nur der regelbasierte Extractor extrahiert einen Wert |
| `both_missing` | Beide liefern keine Größe |
| `gt_not_found_both_empty` | Ground Truth sagt keine Größe vorhanden, beide extrahieren nichts |
| `gt_not_found_over_extracted` | Ground Truth sagt keine Größe vorhanden, mindestens ein Verfahren extrahiert dennoch etwas |

### 6.6 Fehleranalyse

Für fehlerhafte Fälle wird pro Verfahren eine Fehlerklasse vergeben. Die Fehlerklassen dienen der qualitativen Interpretation.

| Fehlerklasse | Beschreibung | Beispiel |
|---|---|---|
| `missing_extraction` | Größe vorhanden, aber nicht extrahiert | GT `500 g`, Prediction `null` |
| `wrong_candidate` | Falscher Kandidat gewählt | Nährwert `100 g` statt Verpackungsgröße `330 ml` |
| `base_price_confusion` | Grundpreis als Größe extrahiert | `1 kg = 4,99 €` |
| `nutrition_reference_confusion` | Nährwertreferenz als Größe extrahiert | `pro 100 ml` |
| `shipping_weight_confusion` | Versand- oder Liefergewicht extrahiert | `8.6 kg` Traygewicht statt `0.33 l` |
| `multipack_error` | Multipack falsch interpretiert | `6 x 250 ml` als `6 ml` |
| `variant_error` | Falsche Variante gewählt | Glasflasche statt Dose |
| `unit_error` | Einheit falsch | `330 g` statt `330 ml` |
| `value_error` | Zahlenwert falsch | `500 ml` statt `330 ml` |
| `conversion_error` | Normalisierung falsch | `0.33 l` nicht zu `330 ml` normalisiert |
| `hallucination` | Wert steht nicht im Input | LLM erfindet `1 l` |
| `ambiguous_ground_truth` | Manuelle Referenz ist fachlich unsicher | Mehrere gleich plausible Varianten |

Daraus ergeben sich folgende Metriken:

| Metrik | Berechnung | Zweck |
|---|---|---|
| Error Type Distribution | `count(error_type) / count(all_errors)` | Zeigt häufigste Fehlerursachen |
| Wrong Candidate Rate | `count(wrong_candidate) / count(all_errors)` | Misst Kandidatenverwechslungen |
| Multipack Error Rate | `count(multipack_error) / count(all_errors)` | Misst Probleme mit Mehrfachpackungen |
| Variant Error Rate | `count(variant_error) / count(all_errors)` | Misst Probleme mit Varianten |
| Hallucination Rate | `count(hallucination) / count(all_errors)` | Besonders relevant für LLM-Ausgaben |

### 6.7 Robustheitsmetriken

Die zentralen Qualitätsmetriken sollten zusätzlich nach Fallgruppen ausgewertet werden.

| Gruppierung | Metriken | Zweck |
|---|---|---|
| Shop / Domain | Precision, Recall, F1, Exact Match | Zeigt Layout- und Domainabhängigkeit |
| Einheitentyp | Precision, Recall, F1 | Vergleich von Masse und Volumen |
| Formattyp | Precision, Recall, F1 | Vergleich einfacher Größen, Dezimalwerte, Multipacks, Varianten |
| HTML-Position | Precision, Recall, F1 | Größe im Titel, in Variante, in Produktdetails, in Tabelle |
| Produkt mit Nährwerttabelle | Fehlerklassen, Precision, Recall | Prüft Verwechslungsgefahr mit `pro 100 g/ml` |
| Produkt mit Grundpreis | Fehlerklassen, Precision, Recall | Prüft Verwechslungsgefahr mit Grundpreis |
| Produkt mit Versandgewicht | Fehlerklassen, Precision, Recall | Prüft Verwechslungsgefahr mit Liefergewicht |

### 6.8 Effizienzdaten

Diese Metriken sind optional, können aber die Diskussion ergänzen.

| Metrik | Berechnung | Zweck |
|---|---|---|
| Runtime per Request | `mean`, `median`, `p95` je Verfahren | Vergleich der Laufzeit |
| LLM Token Usage | `input_tokens`, `output_tokens`, `total_tokens` | Grundlage für Kostenbetrachtung |
| LLM Cost per Request | `input_tokens * input_price + output_tokens * output_price` | Ökonomische Bewertung |
| External Dependency Rate | `N_external_api_calls / N_eval` | Zeigt Abhängigkeit von externen Diensten |
| Rule-Based Runtime | Laufzeit des Regel-Extractors je Request | Effizienz der deterministischen Baseline |

## 7. Priorisierung der Metriken

Für die Bachelorarbeit sollten nicht alle Metriken gleich stark gewichtet werden. Sinnvoll ist folgende Priorisierung:

| Priorität | Metriken |
|---|---|
| Pflicht | Schema Valid Rate, Normalization Success Rate, Precision, Recall, F1, Normalized Exact Match Rate, Field Accuracy, Error Type Distribution |
| Sehr sinnvoll | Coverage, Unit Accuracy, Value Accuracy, Outcome Matrix, Delta Precision, Delta Recall, Delta F1, Metrics by Shop und Formattyp |
| Optional | Cohen's Kappa, Laufzeit, Tokenverbrauch, Kosten, External Dependency Rate |

## 8. Erwartete Ergebnisdarstellung

### 8.1 Hauptvergleich

Die zentrale Ergebnistabelle sollte so aufgebaut sein:

| Verfahren | Coverage | Precision | Recall | F1 | Normalized Exact Match | Unit Accuracy | Field Accuracy |
|---|---:|---:|---:|---:|---:|---:|---:|
| LLM | ... | ... | ... | ... | ... | ... | ... |
| Rule-Based | ... | ... | ... | ... | ... | ... | ... |

### 8.2 Diagnosematrix

Zusätzlich sollte eine Outcome Matrix dargestellt werden:

| Kategorie | Anzahl | Anteil |
|---|---:|---:|
| beide korrekt | ... | ... |
| nur LLM korrekt | ... | ... |
| nur Rule-Based korrekt | ... | ... |
| beide falsch, gleicher Fehler | ... | ... |
| beide falsch, unterschiedlicher Fehler | ... | ... |
| beide ohne Extraktion | ... | ... |

### 8.3 Fehleranalyse

Eine weitere Tabelle sollte die Fehlerarten zeigen:

| Fehlerklasse | LLM Anzahl | LLM Anteil | Rule Anzahl | Rule Anteil |
|---|---:|---:|---:|---:|
| wrong_candidate | ... | ... | ... | ... |
| nutrition_reference_confusion | ... | ... | ... | ... |
| shipping_weight_confusion | ... | ... | ... | ... |
| multipack_error | ... | ... | ... | ... |
| variant_error | ... | ... | ... | ... |
| hallucination | ... | ... | ... | ... |

## 9. Umsetzung im EvaluationFilter

Der `EvaluationFilter` sollte später folgende Aufgaben übernehmen:

```text
1. Ground Truth für den Request laden.
2. LLM-Verpackungsgröße aus dem normalisierten Produktdatenstand lesen.
3. Regelbasierte Verpackungsgröße aus rule_based_extraction.packaging_size lesen.
4. Beide Vorhersagen exakt gegen Ground Truth vergleichen.
5. TP, FP, FN, TN je Verfahren bestimmen.
6. Outcome-Kategorie zwischen LLM und Rule bestimmen.
7. Fehlerklasse je fehlerhaftem Verfahren bestimmen oder vorbereiten.
8. EvaluationResult am Request speichern oder im Pipeline-Kontext berichten.
```

Der Filter sollte nicht aggregieren und nicht exportieren. Aggregation und Bewertungsexport gehören in einen separaten Reporting- oder Export-Schritt.

## 10. Quellen und methodische Grundlage

| Thema | Quelle / Grundlage |
|---|---|
| Precision, Recall, F1, Accuracy | Klassische Information-Retrieval- und Klassifikationsmetriken, z. B. verwendet in Scikit-learn-Metrikdefinitionen |
| Cohen's Kappa | Cohen, J. (1960): A coefficient of agreement for nominal scales |
| Schema Validierung | JSON Schema Validation |
| Datenqualität / Completeness / Accuracy / Consistency | ISO/IEC 25012 Data Quality Model |
| Ground-Truth-basierte Evaluation | Standardvorgehen in Information Extraction und Data Extraction |
| Fehlerklassen | Eigene Operationalisierung für Verpackungsgrößenextraktion im Produktdatenkontext |
| Normalized Exact Match | Eigene Operationalisierung für normalisierte Verpackungsgrößen ohne Akzeptanzbereich |

## 11. Zentrale Festlegung

Die wichtigste methodische Festlegung lautet:

```text
Ein extrahierter Verpackungsgrößenwert ist nur dann korrekt,
wenn normalisierte Einheit und normalisierter Wert exakt mit der Ground Truth übereinstimmen.
```

Es wird kein numerischer Akzeptanzbereich verwendet, da die Aufgabe als Textextraktion und nicht als Messwertschätzung behandelt wird.
