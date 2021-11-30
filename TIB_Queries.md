## Publikationen ohne Datum 


```

SELECT ?p ?date
WHERE
{
  ?p rdf:type  bibo:Document .
  MINUS { ?p rdf:type bibo:Document . ?p vivo:dateTimeValue ?date . }
}

```

## Mehrere Publikationen mit gleichen Titeln

```

Select Distinct  ?pub1 ?label1  ?pub2 ?label2
Where
{
  ?pub1 a bibo:Document .
  ?pub1  rdfs:label ?label1. 
  ?pub2 a bibo:Document .
  ?pub2  rdfs:label ?label2. 
  FILTER (sameTerm(?label1, ?label2)&& !sameTerm(?pub1, ?pub2))
}

```
