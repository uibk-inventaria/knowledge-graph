# Inventaria Knowledge Graph v 0.1

A CIDOC CRM knowledge graph of historical inventories of Tyrolean castles and residences (14th–17th century). It was developed in the **Inventaria** project at the University of Salzburg and the University of Innsbruck.

The graph links the inventory documents with the people who made them, the castles and rooms they describe, the objects they list and the text passages in which those objects appear. Objects, materials, rooms and personal roles are typed with the [Inventaria Thesaurus](https://github.com/uibk-inventaria/thesaurus), which is included in the graph.

## Files

The knowledge graph is provided in two serialisations with the same content:

| File | Content |
|---|---|
| `Inventaria-KnowledgeGraph.trigs` | TriG-star (RDF-star). Roles of persons and periods of ownership are statements about statements (`<< … >>`). Loads into triple stores with RDF-star support, e.g. GraphDB or Apache Jena. |
| `Inventaria-KnowledgeGraph.nq` | N-Quads without RDF-star. The statements about statements are expressed with property classes from the [CIDOC CRM v7.1.3 PC module](https://cidoc-crm.org/rdfs/7.1.3/CIDOC_CRM_v7.1.3_PC.rdfs) (see below). Loads into any triple store. |
| `SPARQL_sample_queries.rq` | Sample SPARQL queries that trace the inventorying of a castle as an event (see [Sample queries](#sample-queries)). |

## Content

The graph has around 290,000 statements in named graphs. The main instance data are:

| Entity | CIDOC CRM class | Number |
|---|---|---|
| Inventory documents | `E31_Document`, `TX1_Written_Text` | 274 / 202 |
| Annotated text segments | `TX7_Written_Text_Segment` | 4,939 |
| Recorded objects | `E19_Physical_Object` | 13,407 |
| Creation and modification of inventories | `E65_Creation`, `E11_Modification` | 318 / 50 |
| Persons | `E21_Person` | 659 |
| Castles and residences | `E27_Site`, `B1_Built_Work` | 46 |
| Rooms and building sections | `B2_Morphological_Building_Section` | 1,085 |
| Places | `E53_Place`, `SP6_Declarative_Place` | 287 / 45 |
| Time-spans | `E52_Time-Span` | 249 |
| Types (thesaurus concepts) | `E55_Type` | 2,080 |

### Named graphs

| Graph | Content |
|---|---|
| `…/all_annotated_inventare_AI` | Annotated inventory texts: text segments and the objects, materials and rooms they mention |
| `…/inventare_metadata` | Metadata of the inventory documents |
| `…/inventare_struktur` | Structure of the inventories |
| `…/persons` | Persons |
| `…/personen_in_event_creation` | Persons involved in creating the inventories, with their roles |
| `…/person_functions` | Functions of persons |
| `…/burgen` | Castles and residences |
| `…/burg_owner` | Owners of the castles and periods of ownership |
| `…/castle_images` | Images of the castles |
| `…/termini` | Terms of document description |
| `https://w3id.org/uibk/inventaria/thesaurus` | Inventaria Thesaurus (SKOS) |

The ontologies used are included in their own graphs:
- CIDOC CRM and the PC module
- CRMsci, CRMinf (v1.2.1), CRMdig, CRMtex, CRMact, CRMgeo, CRMba
- LRMoo, GeoSPARQL, WGS84, SKOS and OWL
- the DARIAH Backbone Thesaurus (BBT)

## Modelling notes

### Roles of persons (P14.1)

The role a person had in creating an inventory (e.g. scribe, witness, addressee) is given with `P14.1 in the role of`.

**TriG-star:**

```turtle
<< :creation crm:P14_carried_out_by :person >> crm:P14.1_in_the_role_of :role .
```

**N-Quads (PC module):**

```turtle
:pc a crm:PC14_carried_out_by ;
    crm:P01_has_domain :creation ;
    crm:P02_has_range  :person ;
    crm:P14.1_in_the_role_of :role .
:creation crm:P14_carried_out_by :person .
```

### Periods of ownership (CRMinf J24)

The period for which a person owned a castle is given with `J24 held at least for` from [CRMinf](https://cidoc-crm.org/crminf/). The ownership statement is treated as an `I11 Situation`.

**TriG-star:**

```turtle
<< :castle crm:P51_has_former_or_current_owner :person >>
    crminf:J24_held_at_least_for :timespan_1542-1578 .
:timespan_1542-1578 a crm:E52_Time-Span ;
    crm:P82a_begin_of_the_begin "1542"^^xsd:gYear ;
    crm:P82b_end_of_the_end     "1578"^^xsd:gYear .
```

**N-Quads:** The PC module has no property class for `P51`. The graph therefore uses its own class `PC51_has_former_or_current_owner`. It is built like the classes of the PC module and defined in the graph `https://w3id.org/uibk/inventaria/ontology`:

```turtle
:pc a inv-ont:PC51_has_former_or_current_owner , crminf:I11_Situation ;
    crm:P01_has_domain :castle ;
    crm:P02_has_range  :person ;
    crminf:J24_held_at_least_for :timespan_1542-1578 .
```

### Dates

Time-spans carry their dates as typed literals:
- full dates as `xsd:date`
- years as `xsd:gYear`
- ranges as `P82a`/`P82b`

## Identifiers

Instances use URIs under `https://w3id.org/uibk/inventaria/`, e.g.

```
https://w3id.org/uibk/inventaria/E21_person/inv_per:000270
https://w3id.org/uibk/inventaria/E27_site/inv_castle:000024
https://w3id.org/uibk/inventaria/E55_type/inv_o:000066
```

## Using the knowledge graph

**Load into GraphDB:** create a repository, then use *Import → Upload RDF files* to load `Inventaria-KnowledgeGraph.trigs` (RDF-star) or `Inventaria-KnowledgeGraph.nq`.

**SPARQL**, e.g. persons and their roles in creating the inventories (N-Quads version):

```sparql
PREFIX crm: <http://www.cidoc-crm.org/cidoc-crm/>
SELECT ?creation ?person ?role WHERE {
  ?pc a crm:PC14_carried_out_by ;
      crm:P01_has_domain ?creation ;
      crm:P02_has_range ?person ;
      crm:P14.1_in_the_role_of ?role .
}
```

The same query on the RDF-star version:

```sparql
PREFIX crm: <http://www.cidoc-crm.org/cidoc-crm/>
SELECT ?creation ?person ?role WHERE {
  << ?creation crm:P14_carried_out_by ?person >> crm:P14.1_in_the_role_of ?role .
}
```

### Sample queries

`SPARQL_sample_queries.rq` contains five queries that trace the inventorying of a castle as an event. Each inventorying is an `E65_Creation` with date, place, occasion, participants and the resulting document. It consists of sub-events per room (`P9_consists_of` → `P7_took_place_at` room → `O8_observed` objects) and of moves from room to room (`E9_Move` with `P27_moved_from` / `P26_moved_to`).

| Query | Question | Result columns |
|---|---|---|
| 1 | When was the castle inventoried, on what occasion, and how many rooms and objects were recorded? | `castle_name`, `inventorying`, `date`, `title`, `occasion`, `occasion_note`, `document`, `rooms`, `objects`, `previous`, `next` |
| 2 | Who took part in each inventorying, and in which role? (RDF-star version) | `castle_name`, `date`, `inventorying`, `person`, `name`, `role` |
| 2b | The same for the N-Quads version with `PC14_carried_out_by` | as query 2 |
| 3 | Which route did the inventorying take through the castle? | `castle_name`, `page`, `room`, `room_name`, `is_start`, `next_room`, `objects` |
| 4 | Which objects were recorded in which room? | `castle_name`, `room`, `object`, `type`, `quantity`, `text` |

How to use them:
- Run each query separately (in GraphDB: one tab per query).
- **Choose a castle:** in queries 1 and 2, replace the label `"Schloss Fragenstein"@de`.
- **Choose a single inventorying:** in queries 3 and 4, enter its URI from the first column of query 1 under `VALUES`.
- **English values:** change the language filter from `"de"` to `"en"`. English labels exist for thesaurus types, roles, event titles and occasions.
- **Named graphs:** the queries assume that the default graph is the union of all named graphs (the GraphDB default).

## Status

This is a first release. Known issues that will be addressed in later versions:
- Some named graphs still use the base `https://uibk.ac.at/inventaria/` instead of `https://w3id.org/uibk/inventaria/`.
- Time-spans are additionally typed as `xsd:date` or `xsd:gYear`.
- For some events, `P82 at some time within` is attached directly to the event instead of to its time-span.

## Related resources

- [Inventaria Thesaurus](https://github.com/uibk-inventaria/thesaurus): the SKOS thesaurus used for typing
- Project website: [inventaria.at](https://www.inventaria.at/)

## License

The knowledge graph is licensed under the [Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). See [`LICENSE`](LICENSE).

You may share and adapt the data for any purpose, including commercially, provided you give appropriate credit.

### Third-party content

The ontologies and vocabularies included in the graph are subject to their own licenses:
- **CIDOC CRM and its extensions** (CRMsci, CRMinf, CRMdig, CRMtex, CRMact, CRMgeo, CRMba, LRMoo): © ICOM/CIDOC CRM SIG, CC BY 4.0
- **Getty Art & Architecture Thesaurus (AAT)**: © J. Paul Getty Trust, [ODC-By 1.0](https://opendatacommons.org/licenses/by/1-0/)
- **DARIAH Backbone Thesaurus (BBT)**: © DARIAH-EU, [license – please check]
- **GeoSPARQL**: © Open Geospatial Consortium

## Citation

> Christina Antenhofer, Elisabeth Gruber-Tokić, Gerald Hiebel, Ingrid Matschinegg, Milena Peralta Friedburg, Claudia Posch and Gerhard Rampl, *Inventaria Knowledge Graph*, University of Salzburg and University of Innsbruck, 2026. [DOI or URL]

## Contact

Gerald Hiebel, University of Innsbruck – ORCID [0000-0002-3799-8391](https://orcid.org/0000-0002-3799-8391)
