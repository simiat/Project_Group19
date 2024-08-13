# Chemical Formula

## Methodology, Results & Analysis

This [DBpedia page of minerals](https://dbpedia.org/page/Mineral) introduces us to a wide variety of minerals. We decided to explore three types of minerals: Zircon, Quartz, and Topaz. These minerals were selected because they belong to the category of Silicates.

Before analyzing ArCo ontologies to find more information about minerals, we decided to use DBpedia to discover the existing types of minerals and their categories on the platform. We then asked Gemini to create a SPARQL query to find "types of minerals on DBpedia". As a prompt engineering technique, we used **Zero-shot-CoT**. The result is available at this [link](https://gemini.google.com/app/dd370266fbd1ce5c?hl=it).

```SPARQL
PREFIX dbo: <http://dbpedia.org/ontology/>
PREFIX dbp: <http://dbpedia.org/property/>
PREFIX dbr: <http://dbpedia.org/resource/>

SELECT DISTINCT ?mineralType ?mineralTypeLabel
WHERE {
  ?mineralType a dbo:Mineral .
  ?mineralType rdfs:label ?mineralTypeLabel .
  FILTER (LANG(?mineralTypeLabel) = "it" )
}
ORDER BY ?mineralTypeLabel
```

The above query is the correct version:

- We changed prefixes to ones more appropriate for DBpedia.
- We changed the namespace `dbpedia:` to `dbo:` to characterize the Mineral class.
- We eliminated the variable `?mineralCategory` because we thought that by doing so the query would give us more relevant results.
- We eliminated the BIND statement, which introduced a variable not specified in the SELECT DISTINCT part of the query, namely `?labelLength`.
- We substituted it with the function `FILTER LANG`, which returns values for the variable `?mineralTypeLabel` in Italian. The query suggested by Gemini returned mineral types, but the related labels were in an Asian language, possibly Chinese or Japanese.

From the list of results, we chose the IRIs for three types of minerals:

- [Zircon](https://dbpedia.org/page/Zircon)
- [Quartz](https://dbpedia.org/page/Quartz)
- [Topaz](https://dbpedia.org/page/Topaz)

## Chemical Formulas

Each mineral is described by a chemical formula, which represents the chemical elements (atoms) in a compound or molecule and the relative proportions of those elements. Using the resources at hand, we sought to retrieve the chemical formulas of two of the minerals under scrutiny.

We first queried ChatGPT, asking how to find the IRI of the mineral "topaz" in the ArCo Knowledge Graph using the Zero-shot-CoT technique:

```SPARQL
PREFIX a-spe: <https://w3id.org/arco/ontology/natural-specimen-description/>

SELECT DISTINCT *
WHERE
{
    <https://w3id.org/arco/resource/topaz> a-spe:hasChemicalFormula ?value .
    ?value rdfs:label ?valueLabel 
    FILTER(STR(?valueLabel) = "Al2(SiO4)(F OH)2" )
}
ORDER BY ASC(?valueLabel)
LIMIT 100
```

We first queried ChatGPT, asking how to find the IRI of the mineral "topaz" in the ArCo Knowledge Graph using the Zero-shot-CoT technique:

```SPARQL
PREFIX a-spe: <https://w3id.org/arco/ontology/natural-specimen-description/>

SELECT DISTINCT *
WHERE
{
    <https://w3id.org/arco/resource/topaz> a-spe:hasChemicalFormula ?value .
    ?value rdfs:label ?valueLabel 
    FILTER(STR(?valueLabel) = "Al2(SiO4)(F OH)2" )
}
ORDER BY ASC (?valueLabel)
LIMIT 100
```

The IRI of topaz provided by ChatGPT ([https://w3id.org/arco/resource/topaz](https://w3id.org/arco/resource/topaz)) is incorrect, as this query returned no results.

![Table Image](path/to/your/image.png)

Secondly, we decided to correct the resulting query in order to return more interesting results than those provided. This is the query that provides ChatGPT:

```SPARQL
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?iri ?label
WHERE {
    ?iri rdfs:label ?label .
    FILTER(CONTAINS(LCASE(?label), "topaz"))
}
LIMIT 10
```

We tried to make some corrections to this query. It is by no means wrong, it nevertheless gave us results which are by most part not relevant. Therefore, we modified it as follows:

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?topazIRI ?topazIRILabel
WHERE 
{
    ?topazIRI rdfs:label ?topazIRILabel 
    FILTER(REGEX(?topazIRILabel, "topazio", "i"))
}
LIMIT 50
```

Substantially, we added the REGEX operator, leaving out LCASE and substituting it with "i" within brackets, which means that the label of the corresponding IRI for topaz should be case insensitive. Results are shown in this table:

From the list, we chose the eighth result whose RDF description is available at this [link](https://dati.beniculturali.it/lodview-arco/resource/NaturalHeritage/0900801226.html).

Scrolling down the page, we stop at the object property **hasChemicalFormula**, which is the property we are interested in:

The chemical formula of topaz is: **Al2(SiO4)(F OH)2**, which is identified as the Simplified Chemical Formula. This is evident from the rdf:type row that associates the formula with two classes:

- [Simplified Chemical Formula](https://w3id.org/arco/ontology/natural-specimen-description/SemplifiedChemicalFormula)
- [Movable Property: Simplified Chemical Formula](https://w3id.org/arco/ontology/movable-property/SemplifiedChemicalFormula)

Having found the simplified chemical formula of topaz, we can write a new triple connecting the mineral and its chemical formula through the property **hasChemicalFormula**:

**NEW TRIPLE 1**

- [https://w3id.org/arco/resource/NaturalHeritage/0900801226](https://w3id.org/arco/resource/NaturalHeritage/0900801226) —> Subject: “TOPAZIO (esemplare)"@en
- a-spe:hasChemicalFormula —> Property/Predicate
- [https://w3id.org/arco/resource/ChemicalFormula/al2sio4f-oh2](https://w3id.org/arco/resource/ChemicalFormula/al2sio4f-oh2) —> Object: “Al2(SiO4)(F OH)2”

This is the query to check the correctness of the above triple:

```sparql
PREFIX a-spe: <https://w3id.org/arco/ontology/natural-specimen-description/>

SELECT DISTINCT *
WHERE
{
<https://w3id.org/arco/resource/NaturalHeritage/0900801226> a-spe:hasChemicalFormula ?value ;
?value rdfs:label ?valueLabel 
FILTER(STR(?valueLabel) = "Al2(SiO4)(F OH)2" )
}
ORDER BY ASC (?valueLabel)
LIMIT 100
```

We repeated the same procedure for the mineral zircon:

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?zirconIRI ?zirconIRILabel
WHERE {
  ?zirconIRI rdfs:label ?zirconIRILabel 
  FILTER(REGEX(?zirconIRILabel, "zircone", "i"))
}
LIMIT 100
```

The correct IRI corresponding to the label “ZIRCONE (esemplare)” is available at this [link](https://w3id.org/arco/resource/NaturalHeritage/0900801498). The chemical formula of zircon is: **Zr(SiO4)**, categorized as an Empirical Chemical Formula, not as a Simplified Formula.

**NEW TRIPLE 2**

- [https://w3id.org/arco/resource/NaturalHeritage/0900801498](https://w3id.org/arco/resource/NaturalHeritage/0900801498) —> Subject: ZIRCONE (esemplare)
- a-spe:hasChemicalFormula —> Predicate
- [https://w3id.org/arco/resource/ChemicalFormula/zrsio4](https://w3id.org/arco/resource/ChemicalFormula/zrsio4) —> Object: “Zr(SiO4)”

Remarkable is the fact that the majority of the hits belong to the class Natural Heritage, which is part of the ARCO ontology, the superclass of Mineral Heritage.

However, if we run this query:

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?zirconIRI
WHERE
{
  ?zirconIRI rdfs:label "zircone" .
}
LIMIT 100
```

We find two results that are notably different from what we expected for a mineral:

We can see that zircon is described by the classes “TechnicalCharacteristic” and “CulturalPropertyDefinition” (they belong to the Denotative Description Ontology on ArCo), although we know that, by consulting the ArCo Ontology, these minerals are included in the class Mineral, for example quartz ([https://w3id.org/arco/resource/Mineral/quarzo](https://w3id.org/arco/resource/Mineral/quarzo)), or Natural Heritage, like topaz or zircon. We are interested in how many entities labelled “zircone” we could find under the two classes cited above.

Firstly, we tried the following procedure for zircon: we used the operator ASK to ask the KG if there are entities with the label “zircone” respectively belonging to the class “TechnicalCharacteristic”. The query returns an affirmative response (“true”):

```sparql
PREFIX a-dd: <https://w3id.org/arco/ontology/denotative-description/>
ASK {
  ?mineral a a-dd:TechnicalCharacteristic ;
  rdfs:label ?label
  FILTER(REGEX(?label, "zircone", "i"))
}
```

Now, we are interested in finding out how many minerals are included in the class TechnicalCharacteristic. For this reason, we asked Gemini with the **Zero-Shot Prompting**:

Some corrections were needed:
- We added the PREFIX part which was missing in the code returned by the large language model.
- Instead of ?mineralCount, we use ?n.
- After SELECT, we added DISTINCT to ensure that no duplicates will be returned in the resulting table.

Corrected query for the class TechnicalCharacteristic:

```sparql
PREFIX a-dd: <https://w3id.org/arco/ontology/denotative-description/>
SELECT DISTINCT (COUNT(?mineral) AS ?n) 
WHERE 
{
  ?mineral a a-dd:TechnicalCharacteristic ;
           rdfs:label ?label .
  FILTER(REGEX(?label, "zircone", "i"))
}
```

The result is **3**. We ran this query to get the resources labelled “zircone” present in the class TechnicalCharacteristic. We used the same query to check how many entities called “zircone” are present under the class Natural Heritage:

```sparql
PREFIX a-dd: <https://w3id.org/arco/ontology/arco/>
SELECT DISTINCT (COUNT(?mineral) AS ?n) 
WHERE 
{
  ?mineral a arco:NaturalHeritage ;
           rdfs:label ?label .
  FILTER(REGEX(?label, "zircone", "i"))
}
```

The result is **156**.

Therefore, given the significant difference in the number of results from the two queries, we can conclude that the class TechnicalCharacteristic, unlike NaturalHeritage, is not relevant to the description of minerals like zircon. The same research with the other two minerals returns the following results:

- Topaz: 13 vs. 232
- Quartz: 73 vs. 9160

These findings emphasize the importance of correctly identifying the relevant classes when working with ontologies to ensure accurate and meaningful results.

[Back](./)

<footer class="site-footer">
  
  <span class="site-footer-owner"><a href="https://github.com/Simiat/project-site-">project-site-</a> is maintained by <a href="https://github.com/simiat">simiat</a>.</span>
  
  <span class="site-footer-credits">This page was generated by <a href="https://pages.github.com">GitHub Pages</a>.</span>
</footer>
