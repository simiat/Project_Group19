---
layout: default
title: "Colour"
---

# Colour

Minerals exhibit a diverse array of colours, which contribute to their uniqueness among geological specimens. These colours can vary from white, violet, and green to bright red, making each mineral distinct. In this respect we decided to interrogate ChatGPT through the technique of **Generated Knowledge Prompting** in the hope of confirming and expanding our knowledge about this particular characteristic of minerals.

![20.ChatGPT_GeneratedKnowledge1](/immagini_markdown/20.ChatGPT_GeneratedKnowledge1.png)

![21.ChatGPT_GeneratedKnowledge2](/immagini_markdown/21.ChatGPT_GeneratedKnowledge2.png)

![22.ChatGPT_GeneratedKnowledge3](/immagini_markdown/22.ChatGPT_GeneratedKnowledge3.png)

![23.ChatGPT_GeneratedKnowledge4](/immagini_markdown/23.ChatGPT_GeneratedKnowledge4.png)

The difference between the two answers lies in the format and detail of the explanation. The initial answer is a straightforward "no" with a brief explanation. The revised answer starts with "no" and provides a detailed explanation, including examples of the different colors quartz can exhibit and specifying that the violet variety is known as amethyst. The revised answer is more comprehensive and informative.

Then we decided to explore the DBpedia KG to find how the property of [color](https://dbpedia.org/property/color) is described in it:

![24.DBpedia_Color](/immagini_markdown/24.DBpedia_Color.png)

After that we prompted ChatGPT again through the technique of **Self-Consistency** in order to find a property in ArCo which describes the color of a mineral. The answer is the following:

![25.ChatGPT_SelfConsistency](/immagini_markdown/25.ChatGPT_SelfConsistency.png)

Given our uncertainty about the accuracy of the LLM’s response, we decided to enhance our search by prompting again the model with the same technique. Upon further investigation, we found that ChatGPT provided an incorrect answer. After thoroughly reviewing ArCo ontologies, it became clear that there is no property named **arco:hasColour** in the Knowledge Graph.

![26.ChatGPT_SelfConsistency2](/immagini_markdown/26.ChatGPT_SelfConsistency2.png)

![27.ChatGPT_SelfConsistency3](/immagini_markdown/27.ChatGPT_SelfConsistency3.png)

ChatGPT provided the same response as before, despite our knowledge that it was incorrect. After reviewing both the ArCo ontology and the Natural Specimen Description ontology on ArCo website, we were unable to find properties related to colour. However, in the Denotative Description ontology, we discovered the property **hasColour**.

![28.ArCo_hasColour](/immagini_markdown/28.ArCo_hasColour.png) 

Afterwards we run the following query on ChatGPT:

![29.ChatGPT_ZeroShot1](/immagini_markdown/29.ChatGPT_ZeroShot1.png)

```sparql
PREFIX arco: <https://w3id.org/arco/ontology/arco/>
PREFIX arco-cd: <https://w3id.org/arco/ontology/context-description/>
PREFIX arco-des: <https://w3id.org/arco/ontology/description/>

SELECT ?mineral ?color
WHERE {
    VALUES ?mineralName {"Quartz" "Topaz"}
    ?mineral a arco-cd:Mineral ;
             arco-des:hasName ?mineralName ;
             arco-des:hasColor ?color .
}
```

Since the above query does not give any result, we tried to modify it. By doing so, we made some considerations:

- The namespace **arco-des** does not exist, hence was left out as well as **arco-cd**, because we do not need any properties or classes from the context description ontology.
- The operator **VALUES** does not appear in the official documentation about <a href="https://www.w3.org/TR/rdf-sparql-query/" target="_blank">SPARQL query language for RDF</a>. Therefore, we decided to eliminate it.

```sparql
PREFIX dd: <https://w3id.org/arco/ontology/denotative-description/>
PREFIX spe: <https://w3id.org/arco/ontology/natural-specimen-description/>

SELECT DISTINCT ?mineral ?colour
WHERE {
    ?mineral a spe:Mineral ;
            rdfs:label ?mineralLabel ;
            dd:hasColour ?colour .
    {FILTER(REGEX(?mineralLabel, "quartz", "i"))}
    UNION
    {FILTER(REGEX(?mineralLabel, "topaz", "i"))}
}
```

Below, we present our investigation using the few-shot technique with ChatGPT to determine whether the query above is correct. The model provided some suggestions for improving the syntax:

![30.ChatGPT_FewShot1](/immagini_markdown/30.ChatGPT_FewShot1.png) 

![31.ChatGPT_FewShot2](/immagini_markdown/31.ChatGPT_FewShot2.png) 

![32.ChatGPT_FewShot3](/immagini_markdown/32.ChatGPT_FewShot3.png)

The suggested correction is the following:

```sparql
PREFIX dd: <https://w3id.org/arco/ontology/denotative-description/>
PREFIX spe: <https://w3id.org/arco/ontology/natural-specimen-description/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?mineral ?colour
WHERE {
  ?mineral a spe:Mineral ;
           rdfs:label ?mineralLabel ;
           dd:hasColour ?colour .
  FILTER(REGEX(?mineralLabel, "quartz", "i") || REGEX(?mineralLabel, "topaz", "i"))
}
```

However explicative and detailed the description provided by the LLM is, this query does not give results.
We now create a query to find the subject of the property **hasColour**:

```sparql
PREFIX a-dd: <https://w3id.org/arco/ontology/denotative-description/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?subject ?subjectLabel ?colour ?colourLabel
WHERE {
  ?subject a-dd:hasColour ?colour;
           rdfs:label ?subjectLabel .
  ?colour rdfs:label ?colourLabel
  FILTER(REGEX(?subjectLabel, "zircone", "i"))
}
ORDER BY ASC (?subjectLabel)
LIMIT 200
```

The results of the query are shown in the table below. It should be noticed that all the property values under the column “colour” are identified by the class Technical Characteristic, which is more appropriate for colour than for names of minerals (see the [results](https://dati.cultura.gov.it/sparql?default-graph-uri=&query=PREFIX+rdfs%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F2000%2F01%2Frdf-schema%23%3E%0D%0A%0D%0ASELECT+%3FzirconIRI%0D%0AWHERE%0D%0A%7B%0D%0A++%3FzirconIRI+rdfs%3Alabel+%22zircone%22+.%0D%0A%7D%0D%0ALIMIT+100&format=text%2Fhtml&timeout=0&signal_void=on) of a query about zircon's IRI in the section ["Chemical Formula"](https://simiat.github.io/Project_Group19/page1.html))

![33.ColourLabel_Zircone](/immagini_markdown/33.ColourLabel_Zircone.png)

We can now write a new triple connecting the first entity of the table labeled “zircone (esemplare)” with the colour label “celeste”:

**NEW TRIPLE 3**

- [https://w3id.org/arco/resource/NaturalHeritage/1600302046](https://w3id.org/arco/resource/NaturalHeritage/1600302046) → Subject: zircone (esemplare)
- a-dd:hasColour → Predicate / Property
- [https://w3id.org/arco/resource/TechnicalCharacteristic/celeste](https://w3id.org/arco/resource/TechnicalCharacteristic/celeste) → Object / Property value: "celeste"

This is the same query as the previous one but instead of “zircone” we used “topazio”:

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX a-dd: <https://w3id.org/arco/ontology/denotative-description/>

SELECT DISTINCT ?subject ?subjectLabel ?colour ?colourLabel
WHERE {
  ?subject a-dd:hasColour ?colour;
           rdfs:label ?subjectLabel .
  ?colour rdfs:label ?colourLabel
  FILTER(REGEX(?subjectLabel, "topazio", "i"))
}
ORDER BY ASC (?subjectLabel)
LIMIT 200
```
![34.ColourLabel_Topazio](/immagini_markdown/34.ColourLabel_Topazio.png)

In order to create the next triple we chose the first result of the table connected to the value “giallo vinato”:

**NEW TRIPLE 4**

- [https://w3id.org/arco/resource/NaturalHeritage/1600302040](https://w3id.org/arco/resource/NaturalHeritage/1600302040) → Subject: topazio (esemplare)
- a-dd:hasColour → Predicate / Property
- [https://w3id.org/arco/resource/TechnicalCharacteristic/giallo-vinato](https://w3id.org/arco/resource/TechnicalCharacteristic/giallo-vinato) → Object / Property value: "giallo vinato"

[BACK](./)


<span class="site-footer-owner"> [Project_Group19](https://github.com/simiat/Project_Group19) is maintained by [Simone Iattoni](https://github.com/simiat).
</span>  
<span class="site-footer-credits">
This page was generated by [GitHub Pages](https://pages.github.com).
</span>
