- Was ist die Schnittstelle
- Wo sind die Crawl Daten
- Welche Formate werden erwartet

# Preprocessing

- Main wird extrahiert?

- REST Schnittstelle
- 

Output:
- Repro dashboard api

Attribute an Dashboard API orientieren
- isValid: Existieren die Mindestattribute
- isProduct: Ist es wirklcih ein Lebensmittel
- noAdditions: ?

- groupFat/groupSugar/productType: Es gibt ein Regelkatalog, ist verschollen
- prodcutType: Gibt ein enum
- referenceValue: pro 100g / pro 100ml
- hasIngredients: Collection der Zutaten aus ingredientText
    - sequenceNumber: Stelle, an der die Zutat im Text steht
- productGroup gehört zu productType: Breadcrumb Kategorisierung mit unterkategorien Wo Dokument?
- uses: Alle Produktgrößen. Auch das eigene
- technicalName: Bezeichnung vor der Inhaltsangabe

- Bis gdm:uses: Nur der Crawl
- Danach: Meta Information über das Produkt

- memberOf: Produkt Gruppe aus dem meta

- Strategie zur Überschreibung der Meta daten: Neu vor alt

- edm, ddm, Was heißt das? Für Attribute übernehmen