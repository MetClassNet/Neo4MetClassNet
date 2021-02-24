# Neo4MetClassNet
GraphDB environment for MetClassNet

## Starting the environment in Docker:

```
docker run -d --restart always --publish=7474:7474 --publish=7687:7687 --volume=$HOME/neo4j/data:/data --env='NEO4JLABS_PLUGINS=["apoc", "n10s"]' --env=NEO4J_AUTH=none neo4j:4.2
```
Importing some ontologies:

```
call n10s.graphconfig.init()
CREATE CONSTRAINT n10s_unique_uri ON (r:Resource) ASSERT r.uri IS UNIQUE

CALL n10s.onto.import.fetch("https://msbi.ipb-halle.de/~sneumann/ChemOnt_2_1-HACKED.owl", "RDF/XML")

CALL n10s.onto.import.fetch("ftp://ftp.ebi.ac.uk/pub/databases/chebi/ontology/chebi_lite.owl", "RDF/XML")
```
