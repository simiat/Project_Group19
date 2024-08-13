# Colour

Minerals exhibit a diverse array of colors, which contribute to their uniqueness among geological specimens. These colors can vary from white, violet, and green to bright red, making each mineral distinct. For this reason, we decided to interrogate ChatGPT through the technique of **Generated Knowledge Prompting** in the hope of confirming and expanding our knowledge about this particular characteristic of minerals.

![20.ChatGPT_GeneretatedKnowledge1](https://github.com/simiat/Project_Group19/blob/master/immagini_markdown/20.ChatGPT_GeneretatedKnowledge1.png)

![21.ChatGPT_GeneratedKnowledge2](https://github.com/simiat/Project_Group19/blob/master/immagini_markdown/21.ChatGPT_GeneratedKnowledge2.png)

![22.ChatGPT_GeneratedKnowledge3](https://github.com/simiat/Project_Group19/blob/master/immagini_markdown/22.ChatGPT_GeneratedKnowledge3.png)

![23.ChatGPT_GeneratedKnowledge4](https://github.com/simiat/Project_Group19/blob/master/immagini_markdown/23.ChatGPT_GeneratedKnowledge4.png)

The difference between the two answers lies in the format and detail of the explanation. The initial answer is a straightforward "no" with a brief explanation. The revised answer starts with "no" and provides a detailed explanation, including examples of the different colors quartz can exhibit and specifying that the violet variety is known as amethyst. The revised answer is more comprehensive and informative.

Then we decided to explore the DBpedia KG to find how the property of color is described in it:

![24.DBpedia_Color](https://github.com/simiat/Project_Group19/blob/master/immagini_markdown/24.DBpedia_Color.png)

After that we prompted ChatGPT again through the technique of **Self-Consistency** in order to find a property in ArCo which describes the color of a mineral. The answer is the following:

![25.ChatGPT_SelfConsistency](https://github.com/simiat/Project_Group19/blob/master/immagini_markdown/25.ChatGPT_SelfConsistency.png)

Given our uncertainty about the accuracy of the LLM’s response, we decided to enhance our search by prompting again the model with the same technique. Upon further investigation, we found that ChatGPT provided an incorrect answer. After thoroughly reviewing ArCo ontologies, it became clear that there is no property named **arco:hasColour** in the Knowledge Graph.

![26.ChatGPT_SelfConsistecy2](https://github.com/simiat/Project_Group19/blob/master/immagini_markdown/26.ChatGPT_SelfConsistecy2.png)

![27.ChatGPT_SelfConsistency3](https://github.com/simiat/Project_Group19/blob/master/immagini_markdown/27.ChatGPT_SelfConsistency3.png)

ChatGPT provided the same response as before, despite our knowledge that it was incorrect. After reviewing both the ArCo ontology and the Natural Specimen Description ontology on ArCo website, we were unable to find any property related to color. However, in the Denotative Description ontology, we discovered the property **dd:hasColour**.

![28.ArCo_hasColour](https://github.com/simiat/Project_Group19/blob/master/immagini_markdown/28.ArCo_hasColour.png) 

Afterwards we run the following query on ChatGPT:

![29.ChatGPT_ZeroShot1](https://github.com/simiat/Project_Group19/blob/master/immagini_markdown/29.ChatGPT_ZeroShot1.png)

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

- The namespace `arco-des` does not exist, hence was left out as well as `arco-cd`, because we do not need any properties or classes from the context description ontology.
- The operator `VALUES` does not appear in the official documentation of W3C [here](https://www.w3.org/TR/rdf-sparql-query/), so we decided to eliminate it.

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

![30.ChatGPT_FewShot1](https://github.com/simiat/Project_Group19/blob/master/immagini_markdown/30.ChatGPT_FewShot1.png) 

![31.ChatGPT_FewShot2](https://github.com/simiat/Project_Group19/blob/master/immagini_markdown/31.ChatGPT_FewShot2.png) 

![32.ChatGPT_FewShot3](https://github.com/simiat/Project_Group19/blob/master/immagini_markdown/32.ChatGPT_FewShot3.png)

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

However explicative and detailed was the description provided by the LLM, this query does not give results.

We now create a query to find the subject of the property `hasColour`:

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

The results of the query are shown in the table. Notice that, unlike the result of the last query, all the property values under the column “colour” are identified by the class Technical Characteristic, which is more appropriate for color than for names of minerals.

![33.ColourLabel_Zircone](https://github.com/simiat/Project_Group19/blob/master/immagini_markdown/33.ColourLabel_Zircone.png)

We can now write a new triple connecting the first entity of the table labeled “zircone (esemplare)” with the color value “celeste”:

**NEW TRIPLE 3**

- [https://w3id.org/arco/resource/NaturalHeritage/1600302046](https://w3id.org/arco/resource/NaturalHeritage/1600302046) → Subject: zircone (esemplare)
- `a-dd:hasColour` → Property / Predicate
- [https://w3id.org/arco/resource/TechnicalCharacteristic/celeste](https://w3id.org/arco/resource/TechnicalCharacteristic/celeste) → Property value / Object (Paillettes)

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
![34.ColourLabel_Topazio](https://github.com/simiat/Project_Group19/blob/master/immagini_markdown/34.ColourLabel_Topazio.png)

In order to create the next triple we chose the first result of the table connected to the value “giallo vinato”:

**NEW TRIPLE 4**

- [https://w3id.org/arco/resource/NaturalHeritage/1600302040](https://w3id.org/arco/resource/NaturalHeritage/1600302040) → Subject: topazio (esemplare)
- `a-dd:hasColour` → Predicate / Property
- [https://w3id.org/arco/resource/TechnicalCharacteristic/giallo-vinato](https://w3id.org/arco/resource/TechnicalCharacteristic/giallo-vinato) → Property value: giallo vinato

[Back](./)

—
<span class="site-footer-owner">
[Project_Group19-](https://github.com/simiat/project-site-) is maintained by [simiat](https://github.com/simiat).
</span>  
<span class="site-footer-credits">
This page was generated by [GitHub Pages](https://pages.github.com).
</span>
