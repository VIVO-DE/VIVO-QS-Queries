# Abfragen
- Publikationen ohne Datum
- Dubletten - Mehrere Publikationen mit gleichen Titeln
- Dubletten - Mehrere Venues (Journals, Sammelbänder, Schriftenreihen) mit gleichen Titeln

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

## Journals und Sammelbändern mit gleichen Titeln

```

Select Distinct  ?journal1 ?label1  ?journal2 ?label2
Where
{
  ?journal1 a ?type .
  ?journal1  rdfs:label ?label1. 
  ?journal1 vivo:publicationVenueFor ?art1 .
  ?journal2 a  ?type .
  ?journal2  rdfs:label ?label2. 
  ?journal2 vivo:publicationVenueFor ?art2 .
  FILTER (sameTerm(?label1, ?label2)&& !sameTerm(?journal1, ?journal2))
}

```
