## Benötigte Queries

* doppelte Organisationen (Verlage, Universitäten etc.)
* Publikationen mit gleichem/ähnlichen Titel
* Publikationen ohne Jahr
* Publication Venues (veröffentlicht in) mit selbem/ähnlichem Titel
* Personen mit gleichem/ähnlichen Namen
* vCards, die als Personen existieren
* doppelte/ähnliche Veranstaltungen
* Promotionen (ohne Beteiligte)
* Individiuals mit mehreren, inkonsistenten Labels
* Abfrage HSMW Organisationen ohne Strukturnummer
* Inkonsistente Daten
** fehlende Person an Rollen
** fehlende Aktivität an Rollen
** (HSMW) Personen ohne Forschungsprofillinie
** fehlende Publikationen an Autorenschaften

# Personen ohne Forschungsschwerpunkt

```
SELECT ?uri ?label
WHERE {
 	?uri a foaf:Person .
    ?uri a hsmw:HsMittweida .
    ?uri rdfs:label ?label .
  FILTER NOT EXISTS { ?uri hsmw:hatForschungsprofillinie ?fsp }
}
```

# Publizierende Personen mit fehlendem Forschungsschwerpunkt sortiert nach Fachgruppe

```
SELECT ?uri ?label ?fachgruppe
  (GROUP_CONCAT(?authorship; separator = ":") AS ?authorships)
WHERE {
 	?uri a foaf:Person .
    ?uri a hsmw:HsMittweida .
    ?uri rdfs:label ?label .
    ?uri vivo:relatedBy ?authorship .
    ?authorship a vivo:Authorship .
    ?uri vivo:relatedBy ?position .
    ?position a vivo:Position .
    ?position vivo:relates ?ad .
    ?ad a vivo:AcademicDepartment .
    ?ad rdfs:label ?fachgruppe .
  FILTER NOT EXISTS { ?uri hsmw:hatForschungsprofillinie ?fsp }
} GROUP BY ?uri ?label ?fachgruppe
ORDER BY ?fachgruppe?label
```

# Doppelte Organisationen

```
PREFIX rdf:      <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs:     <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd:      <http://www.w3.org/2001/XMLSchema#>
PREFIX foaf:     <http://xmlns.com/foaf/0.1/>
PREFIX hsmw:     <https://vivo.hs-mittweida.de/vivo/ontology/hsmw#>
PREFIX kdsf:     <http://kerndatensatz-forschung.de/owl/Basis#>
PREFIX vivo:     <http://vivoweb.org/ontology/core#>
PREFIX obo:      <http://purl.obolibrary.org/obo/>
PREFIX bibo:    <http://purl.org/ontology/bibo/>


SELECT (COUNT(?label) AS ?count) ?label (GROUP_CONCAT(?_uri; SEPARATOR = ";") AS ?uri)
{
    ?_uri a foaf:Organization .
    ?_uri rdfs:label ?label .
} GROUP BY ?label
 ORDER BY ?label
```
"?_uri a foaf:Organization" jeweils mit Publisher etc. ersetzen, um auf Organisationen eines bestimmten Typs zu beschränken.

# HSMW Organisationen ohne Strukturnummer

```
SELECT ?s ?label ?orgNum WHERE {
   ?s a vivo:Organization.
   ?s a <https://vivo.hs-mittweida.de/vivo/ontology/hsmw#HsMittweida> . 
  NOT EXISTS { ?s <https://vivo.hs-mittweida.de/vivo/ontology/hsmw#strukturnummer> ?orgNum }
}
```

# Individuals mit mehreren Labels

```
PREFIX rdf:      <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs:     <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd:      <http://www.w3.org/2001/XMLSchema#>
PREFIX foaf:     <http://xmlns.com/foaf/0.1/>
PREFIX hsmw:     <https://vivo.hs-mittweida.de/vivo/ontology/hsmw#>
PREFIX kdsf:     <http://kerndatensatz-forschung.de/owl/Basis#>
PREFIX vivo:     <http://vivoweb.org/ontology/core#>
PREFIX obo:      <http://purl.obolibrary.org/obo/>
PREFIX bibo:    <http://purl.org/ontology/bibo/>


SELECT (COUNT(?label) AS ?c) ?uri GROUP_CONCAT(?label;   SEPARATOR=";")
{
    ?uri rdfs:label ?label .
} 
GROUP BY ?uri
ORDER BY DESC(?c)
```

> NOTE: Query muss auf jeden Fall angepasst werden, wenn Labels mit verschiedenen Sprach-Tags im Einsatz sind.

# Personen mit mehreren Räumen

```sql
PREFIX rdf:      <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs:     <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd:      <http://www.w3.org/2001/XMLSchema#>
PREFIX foaf:     <http://xmlns.com/foaf/0.1/>
PREFIX hsmw:     <https://vivo.hs-mittweida.de/vivo/ontology/hsmw#>
PREFIX kdsf:     <http://kerndatensatz-forschung.de/owl/Basis#>
PREFIX vivo:     <http://vivoweb.org/ontology/core#>
PREFIX obo:      <http://purl.obolibrary.org/obo/>
PREFIX bibo:    <http://purl.org/ontology/bibo/>

SELECT (COUNT(?room) AS ?c) ?label GROUP_CONCAT(?rname;   SEPARATOR=";")
{
    ?uri a foaf:Person .
    ?uri rdfs:label ?label .
    ?uri <http://purl.obolibrary.org/obo/RO_0001025> ?room .
    ?room rdfs:label ?rname .
} 
GROUP BY ?uri ?label
ORDER BY DESC(?c)
```

# Query für Rollen mit fehlenden Aktivitäten

```sql
PREFIX rdf:      <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs:     <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd:      <http://www.w3.org/2001/XMLSchema#>
PREFIX foaf:     <http://xmlns.com/foaf/0.1/>
PREFIX hsmw:     <https://vivo.hs-mittweida.de/vivo/ontology/hsmw#>
PREFIX kdsf:     <http://kerndatensatz-forschung.de/owl/Basis#>
PREFIX vivo:     <http://vivoweb.org/ontology/core#>
PREFIX obo:      <http://purl.obolibrary.org/obo/>
PREFIX bibo:    <http://purl.org/ontology/bibo/>


SELECT ?role ?activity
{
  ?role a <http://purl.obolibrary.org/obo/BFO_0000023> .
  OPTIONAL { ?role <http://vivoweb.org/ontology/core#relatedBy> ?activity . }
  OPTIONAL { ?role <http://vivoweb.org/ontology/core#roleContributesTo> ?activity .}
  OPTIONAL { ?role <http://purl.obolibrary.org/obo/BFO_0000054> ?activity .}
  FILTER (!bound(?activity))
} 
```

### Query für Autorenschaften mit fehlenden Publikationen

```sql
PREFIX rdf:      <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs:     <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd:      <http://www.w3.org/2001/XMLSchema#>
PREFIX foaf:     <http://xmlns.com/foaf/0.1/>
PREFIX hsmw:     <https://vivo.hs-mittweida.de/vivo/ontology/hsmw#>
PREFIX kdsf:     <http://kerndatensatz-forschung.de/owl/Basis#>
PREFIX vivo:     <http://vivoweb.org/ontology/core#>
PREFIX obo:      <http://purl.obolibrary.org/obo/>
PREFIX bibo:    <http://purl.org/ontology/bibo/>


SELECT ?role ?activity
{
  ?role a <http://vivoweb.org/ontology/core#Authorship> .
  OPTIONAL { 
    ?role <http://vivoweb.org/ontology/core#relates> ?publication .
  	?publication a <http://purl.org/ontology/bibo/Document> .
  }
  FILTER (!bound(?publication))
} 
```
### Query für Journals ohne Beiträge

```sql
PREFIX rdf:      <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs:     <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd:      <http://www.w3.org/2001/XMLSchema#>
PREFIX foaf:     <http://xmlns.com/foaf/0.1/>
PREFIX hsmw:     <https://vivo.hs-mittweida.de/vivo/ontology/hsmw#>
PREFIX kdsf:     <http://kerndatensatz-forschung.de/owl/Basis#>
PREFIX vivo:     <http://vivoweb.org/ontology/core#>
PREFIX obo:      <http://purl.obolibrary.org/obo/>
PREFIX bibo:    <http://purl.org/ontology/bibo/>


SELECT ?uri ?pubVenue {
		?uri a bibo:Journal .
		OPTIONAL {
    		?uri vivo:publicationVenueFor ?pubVenue .
  		}
  		FILTER (!bound(?pubVenue))
} 
```
