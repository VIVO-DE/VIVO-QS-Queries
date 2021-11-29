## Publikationen ohne Datum 


```

SELECT ?p ?date
WHERE
{
  ?p rdf:type  bibo:Document .
  MINUS { ?p rdf:type bibo:Document . ?p vivo:dateTimeValue ?date . }
}

```

