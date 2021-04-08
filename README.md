# Neo4MetClassNet
GraphDB environment for MetClassNet

## Starting the container:

docker run -d --restart always --publish=7474:7474 --publish=7687:7687 --volume=$HOME/neo4j/data:/data --env='NEO4JLABS_PLUGINS=["apoc", "n10s"]' --env=NEO4J_AUTH=none neo4j:4.2

```
(
cat <<EOF
call n10s.graphconfig.init()
CREATE CONSTRAINT n10s_unique_uri ON (r:Resource) ASSERT r.uri IS UNIQUE

CALL n10s.onto.import.fetch("https://msbi.ipb-halle.de/~sneumann/ChemOnt_2_1-HACKED.owl", "RDF/XML")

CALL n10s.onto.import.fetch("ftp://ftp.ebi.ac.uk/pub/databases/chebi/ontology/chebi_lite.owl", "RDF/XML")
EOF
) | cypher-shell 

```

## Loading Data 

MTBLS1586 (Würmchen)
```
python gml2cypher/gml2cypher.py -n 'MTBLS1586:feature' -e 'similarity' MTBLS1586/similarity.gml | cypher-shell
python gml2cypher/gml2cypher.py -n 'MTBLS1586:feature' -e 'mzdiff' MTBLS1586/mzdiff.gml | cypher-shell
python gml2cypher/gml2cypher.py -n 'MTBLS1586:feature' -e 'pears' MTBLS1586/pears.gml | cypher-shell 
python gml2cypher/gml2cypher.py -n 'MTBLS1586:feature' -e 'homol' MTBLS1586/homol.gml | cypher-shell 
```

and the corresponding SBML:
```
python gml2cypher/gml2cypher.py -n 'metabolites' -e 'reactions' MTBLS1586/WormJam-GEM-20190101_L3_no-side_no-comp.gml | cypher-shell 
```
