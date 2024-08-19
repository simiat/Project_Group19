---
layout: default
title: "Chemical Formula"
---

# Chemical Formula

## DBpedia and Minerals

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
- We changed the namespace **dbpedia:** to **dbo:** to characterize the Mineral class.
- We eliminated the variable **?mineralCategory** because we thought that by doing so the query would give us more relevant results.
- We eliminated the BIND statement, which introduced a variable not specified in the SELECT DISTINCT part of the query, namely **?labelLength**.
- We substituted it with the function FILTER LANG, which returns values for the variable **?mineralTypeLabel** in Italian. The query suggested by Gemini returned mineral types, but the related labels were in an Asian language, possibly Chinese or Japanese. The results are available at this [link](https://dbpedia.org/sparql?default-graph-uri=http%3A%2F%2Fdbpedia.org&query=PREFIX+dbo%3A+%3Chttp%3A%2F%2Fdbpedia.org%2Fontology%2F%3E%0D%0APREFIX+dbp%3A+%3Chttp%3A%2F%2Fdbpedia.org%2Fproperty%2F%3E%0D%0APREFIX+dbr%3A+%3Chttp%3A%2F%2Fdbpedia.org%2Fresource%2F%3E%0D%0A%0D%0ASELECT+DISTINCT++%3FmineralType+%3FmineralTypeLabel%0D%0AWHERE+%7B%0D%0A++%3FmineralType+a+dbo%3AMineral+.++%0D%0A++%3FmineralType+rdfs%3Alabel+%3FmineralTypeLabel+.++%0D%0A++FILTER+%28LANG%28%3FmineralTypeLabel%29+%3D+%22it%22+%29%0D%0A%7D%0D%0AORDER+BY+%3FmineralTypeLabel%0D%0A&format=text%2Fhtml&timeout=30000&signal_void=on&signal_unconnected=on)

From the list of results, we chose the IRIs for three types of minerals:

- [Zircon](https://dbpedia.org/page/Zircon)
- [Quartz](https://dbpedia.org/page/Quartz)
- [Topaz](https://dbpedia.org/page/Topaz)

## Chemical Formulas

Each mineral is described by a chemical formula, which represents the chemical elements (atoms) in a compound or molecule and the relative proportions of those elements. Using the resources at hand, we sought to retrieve the chemical formulas of two of the minerals under scrutiny.
We first queried ChatGPT, asking how to find the IRI of the mineral "topaz" in the ArCo Knowledge Graph using the Zero-shot-CoT technique:

![5.ChatGPT_ZeroShotCoT1](/immagini_markdown/5.ChatGPT_ZeroShotCoT1.png)

![6.ChatGPT_ZeroShotCoT2](/immagini_markdown/6.ChatGPT_ZeroShotCoT2.png)

![7.ChatGPT_ZeroShotCoT3](/immagini_markdown/7.ChatGPT_ZeroShotCoT3.png)

The LLM returns a query plus the IRI of topaz.
Firstly, we realized that the IRI of topaz provided by ChatGPT (<https://w3id.org/arco/resource/topaz>) is not correct, in that if we run the following query: 

```SPARQL
PREFIX a-spe: <https://w3id.org/arco/ontology/natural-specimen-description/>

SELECT DISTINCT *
WHERE
{
    <https://w3id.org/arco/resource/topaz> a-spe:hasChemicalFormula ?value .
    ?value rdfs:label ?valueLabel 
    FILTER(STR(?valueLabel) = "chemical formula" )
}
ORDER BY ASC(?valueLabel)
LIMIT 100
```

it does not give any results, as it is also evident from this empty table.

![8.EmptyTable_Value&ValueLabel](/immagini_markdown/8.EmptyTable_Value&ValueLabel.png)

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

Substantially, we added the REGEX operator, leaving out LCASE and substituting it with "i" within inverted commas, which means that the label of the corresponding IRI for topaz should be case insensitive. Results are shown in this table:

![9.TopazIRI](/immagini_markdown/9.TopazIRI.png)

From the list, we chose the eighth result whose RDF description is available at this [link](https://dati.beniculturali.it/lodview-arco/resource/NaturalHeritage/0900801226.html). Scrolling down the page, we stop at the object property **hasChemicalFormula**, which is the property we are interested in:

![10.Topazio_esemplare1](/immagini_markdown/10.Topazio_esemplare1.png)
![11.Topazio_esemplare2](/immagini_markdown/11.Topazio_esemplare2.png)

The chemical formula of topaz is: **Al2(SiO4)(F OH)2**, which is identified as _Simplified Chemical Formula_. This is evident from the rdf:type row that associates the formula with two classes:

- [Simplified Chemical Formula](https://w3id.org/arco/ontology/natural-specimen-description/SemplifiedChemicalFormula) 
- [Movable Property: Simplified Chemical Formula](https://w3id.org/arco/ontology/movable-property/SemplifiedChemicalFormula) 

![12.ChemicalFormula_Topaz](/immagini_markdown/12.ChemicalFormula_Topaz.png)
Having found the simplified chemical formula of topaz, we can write a new triple connecting the mineral and its chemical formula through the property hasChemicalFormula:

**NEW TRIPLE 1**

- [https://w3id.org/arco/resource/NaturalHeritage/0900801226](https://w3id.org/arco/resource/NaturalHeritage/0900801226) → Subject: “TOPAZIO (esemplare)"@en
- a-spe:hasChemicalFormula → Property/Predicate
- [https://w3id.org/arco/resource/ChemicalFormula/al2sio4f-oh2](https://w3id.org/arco/resource/ChemicalFormula/al2sio4f-oh2) → Object: “Al2(SiO4)(F OH)2”

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

This query returns exactly the subject of the above written triple

![13.ChemicalFormula_Topaz2](/immagini_markdown/13.ChemicalFormula_Topaz2.png)

We repeated the same procedure for zircon:

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?zirconIRI ?zirconIRILabel
WHERE {
  ?zirconIRI rdfs:label ?zirconIRILabel 
  FILTER(REGEX(?zirconIRILabel, "zircone", "i"))
}
LIMIT 100
```
![14.ZirconIRI](/immagini_markdown/14.ZirconIRI.png)

From the table of results we chose the eighth entity, “ZIRCONE (esemplare)”, whose related IRI connected to the label is available at this [link](https://w3id.org/arco/resource/NaturalHeritage/0900801498). The chemical formula of zircon is: **Zr(SiO4)**, categorized as _Empirical Chemical Formula_, not as Simplified Chemical Formula.

![15.ChemicalFormula_Zircon](/immagini_markdown/15.ChemicalFormula_Zircon.png)

**NEW TRIPLE 2**

- [https://w3id.org/arco/resource/NaturalHeritage/0900801498](https://w3id.org/arco/resource/NaturalHeritage/0900801498) → Subject: ZIRCONE (esemplare)
- a-spe:hasChemicalFormula → Predicate
- [https://w3id.org/arco/resource/ChemicalFormula/zrsio4](https://w3id.org/arco/resource/ChemicalFormula/zrsio4) → Object: “Zr(SiO4)”

---


Remarkably, the vast majority of the results of our previous researches belong to the class **Natural Heritage**, which is part of the ArCo ontology, the superclass of 'Mineral Heritage'.

![16.ArCo_NaturalHeritage](/immagini_markdown/16.ArCo_NaturalHeritage.png)

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

we find two results that are notably different from what we expected for a mineral

![17.ZirconIRI2](/immagini_markdown/17.ZirconIRI2.png)

We can see that zircon is described by the classes **TechnicalCharacteristic** and **CulturalPropertyDefinition** (they both belong to the Denotative Description Ontology in ArCo), although we know that, by consulting the ArCo Ontology, these minerals are included in the class 'Mineral', for example quartz ([https://w3id.org/arco/resource/Mineral/quarzo](https://w3id.org/arco/resource/Mineral/quarzo)), or 'Natural Heritage', like topaz or zircon. We are interested in how many entities labelled “zircone” we could find under the two classes cited above.
Firstly, we tried the following procedure for zircon: we used the operator **ASK** to ask ArCo knowledge graph if there are entities with the label “zircone” belonging to the class TechnicalCharacteristic. The query returns an affirmative response (“true”):

```sparql
PREFIX a-dd: <https://w3id.org/arco/ontology/denotative-description/>
ASK {
  ?mineral a a-dd:TechnicalCharacteristic ;
  rdfs:label ?label
  FILTER(REGEX(?label, "zircone", "i"))
}
```

Now, we are interested in finding out how many minerals are included in the class TechnicalCharacteristic. For this reason, we asked Gemini with the **Zero-Shot Prompting**:

![18.Gemini_ZeroShot](/immagini_markdown/18.Gemini_ZeroShot.png)

![19.Gemini_ZeroShot2](/immagini_markdown/19.Gemini_ZeroShot2.png)

Some corrections were needed:
- We added the PREFIX part which was missing in the code returned by the large language model.
- We maintained **COUNT**, but instead of ?mineralCount, we use **?n**.
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

Therefore, given the significant difference in the number of results from the two last queries, we can conclude that the class TechnicalCharacteristic, unlike NaturalHeritage, is not relavant to the description of minerals. The same research with the other two minerals returns the following results:

- Topaz: 13 vs. 232
- Quartz: 73 vs. 9160

These findings emphasize the importance of correctly identifying the relevant classes when working with ontologies to ensure accurate and meaningful results.

[BACK](./)

<span class="site-footer-owner"> [Project_Group19](https://github.com/simiat/Project_Group19) is maintained by [Simone Iattoni](https://github.com/simiat).
</span>  
<span class="site-footer-credits">
This page was generated by [GitHub Pages](https://pages.github.com).
</span>
