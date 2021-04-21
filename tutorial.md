## Exploitation of MetClassNet data

MetClassNet data are stored on a Neo4J graph database, which can be queried using the Cypher language.
There is a plethora of tutorials for Neo4j neophytes: the official [neo4j.com](https://neo4j.com/developer/get-started/) is a very good place to start. It is recommended to follow this guide, although, for a quick hands-on session by people familiar with working with graphs, some of the basics regarding the query language are covered here. Experienced users and people just here to have a look can directly jump to the samples queries section to see the kind of relevant information that can be extracted from our system.

### How to start : What's in there?

 - Get the list of nodes and edges types ("_node labels_" and "_relationship type_" in Neo4J speech)
 ```
 CALL db.relationshipTypes()
 ```
 ```
 CALL db.labels()
 ```
 - show how they are connected (the the data model):
 ```
 CALL db.schema.visualization()
 ```
 - see how many of them are in there:
 ```
 MATCH (n)
 RETURN count(n)
 ```
 
 ```
 MATCH ()-[r]-()
 RETURN count(r)
 ```
 - see the kind of attributes ("_properties_") they have
 ```
MATCH (n)
UNWIND keys(n) as k
RETURN distinct count(n),k,labels(n)
 ```
 
  ```
MATCH ()-[r]-()
UNWIND keys(r) as k
RETURN distinct count(r),k,type(r)
 ```
 
 _note: if the total number of nodes and relationship is large, a random sample can be extracted using_ `MATCH (n) WHERE rand() <= 0.10`_(here, 10% of nodes are sampled)_
 
 ### How to start: What can we do?
 
 - get the list of functions that can be used in cypher (the `count()`, `key()`, `type()` and `labels()` that we used in the previous steps can be found here):
 ```
 CALL dbms.functions()
 ```
 
- write a Cypher query

__the Cypher Query Language synthax:__

`()` :  a node  
`-->` : an edge (a.k.a. _relationship_)  

`(n)` :  a node stored in a variable named n  
`-[e]->` : a edge stored in a variable named e  

`(:feature)` :  a node of a given class (a.k.a. _label_)  
`-[:pears]->` :  a relationship of a given class (a.k.a. _ltype_)  

`(n {Formula:"C6H12O6"})` :  a node with a given attribute (a.k.a. _propertie_)  
`-[e {massdifference:38}]->` :  an edge with a given propertie  

__Main Cypher clauses:__ 

> A basic cypher query is usually structured in 3 parts:  
	 - a __pattern__ : a description of the kind of subgraph we're interested in.   
	  	_Example: A peak in a experimental layer + its neighbors that are mapped in the knowledge layer_  
	 - an __anchor__ : constraints that define where the pattern will be searched. (some basic property/type constraints can be defined directly in the pattern)  
		 _Example: __the__ peak __with mass xxxx__ in experimental layer + its neighbors that are mapped in the knowledge layer_  
	 - a __format__ : a description of what to keep and how we want the information to be displayed.  
		 _Example: __from__ the peak with mass xxxx in experimental layer + its neighbors that are mapped in the knowledge layer, __get the list of  names and masses of  the neighbors__._  

| Clause | Description |
| ----- | ---- |
| __PATTERN__| |
| [`MATCH`](https://neo4j.com/docs/cypher-manual/current/clauses/match/#query-match) | Specify the patterns to search for in the database. |
| [`OPTIONAL MATCH`](https://neo4j.com/docs/cypher-manual/current/clauses/optional-match/#query-optional-match) | Specify the patterns to search for in the database while allowing missing parts. |
| __ANCHOR__| |
| [`WHERE`](https://neo4j.com/docs/cypher-manual/current/clauses/where/#query-where) | Adds constraints to the patterns in a `MATCH` or `OPTIONAL MATCH` clause.|
| [`AND`/`OR`/`NOT`](https://neo4j.com/docs/cypher-manual/current/syntax/operators/#query-operators-boolean)| Logical operator to combine constraints. |
| __FORMAT__| |
| [`RETURN`](https://neo4j.com/docs/cypher-manual/current/clauses/return/#query-return) | Defines what to include in the query result set. Can be the matching nodes and/or relationships as a subgraph, the values of some of their properties, or the results of [functions](https://neo4j.com/docs/cypher-manual/current/functions/) applyed on them (`count()`, `max()`, `sum()`).|
| [`RETURN ... AS`](https://neo4j.com/docs/cypher-manual/current/clauses/return/#query-return)| Set results format by defining a column alias. |
| [`ORDER BY [ASC/DESC]`](https://neo4j.com/docs/cypher-manual/current/clauses/order-by/#query-order)| Follows a `RETURN` clause, specifying that the output should be sorted in either ascending (the default) or descending order.|
| [`LIMIT`](https://neo4j.com/docs/cypher-manual/current/clauses/limit/#query-limit)| Follows a `RETURN` clause, specifying a maximum number of results to be outputed. |
| [`DISTINCT`](https://neo4j.com/docs/cypher-manual/current/syntax/operators/#query-operators-aggregation)| Remove remove duplicates values. |

 Example: _from the peak with mass xxxx in experimental layer + its neighbors that are mapped in the knowledge layer, get the list of  names and masses of  the neighbors._
 ```
 MATCH (n1:feature)-[:mzdiff]-(n2:feature)
 WHERE n1.value = xxxx
 AND (n2)-[:match]-(:metabolites)
 RETURN n2.mass AS neighbor_mass, n2.name AS neighbor_name
 ORDER BY neighbor_mass DESC
 ```
 
 ### Going further: performing complex cypher queries

>More complex queries can be build by combining basic ones:  
	 - __nested__ queries : where an anchor can be definied as matching another pattern.  
	 - __chained__ queries : where the results of a query are passed as input to another one.  
	 - __combined__ queries : where the results are a combinaison of the results of multiple queries.  

| Clause | Description |
| ----- | ---- |
| [`WHERE EXISTS { ... }`](https://neo4j.com/docs/cypher-manual/current/clauses/where/#existential-subqueries) | Use a nested subquery to define the constraints to the patterns.|
| [`WITH ... [AS]`](https://neo4j.com/docs/cypher-manual/current/clauses/with/#query-with) | Same as `RETURN`, but create chained query by piping the results from one to the next. `AS` can be used to store results in variables. [Functions](https://neo4j.com/docs/cypher-manual/current/functions/) can also be used to process results before piping. |
| [`UNION`](https://neo4j.com/docs/cypher-manual/current/clauses/union/#query-union) | Combines the result of multiple queries into a single result set. __Duplicates are removed__.|
| [`UNION ALL`](https://neo4j.com/docs/cypher-manual/current/clauses/union/#query-union)| Combines the result of multiple queries into a single result set. __Duplicates are retained__.|


### In practice: Sample queries

- __get features' degree (number of neighbors)__  
```
MATCH (n1:feature)--(n2) 
RETURN n1.name, count(n2) AS degree 
ORDER BY degree DESC
```

- __get the ego network (network of neighbors) of a given node__  
```
MATCH (n1:feature {name:"Cluster_0619"})-[r1]-(n2) 
OPTIONAL MATCH (n2)-[r2]-(n3)--(n1) 
RETURN r1,n2,r2,n3
```

- __get pairs with mzdiff match + correlation__  
```
MATCH (n1:feature)-[diff:mzdiff]-(n2:feature)
MATCH (n1)-[corr:pears]-(n2)
RETURN n1.name, n2.name, diff.value, corr.pearson
```

- __shows subclasses of a chemical class__
```
MATCH p=(c:n4sch__Class {n4sch__label:"Flavonoids"})<-[*]-(c2) 
RETURN p;
```
> This query search for a path of undefined length, using a wildcard `[*]`.
> A specific length can be set using `[*3]`, and a range using `[*1..3]` .  
> ⚠ Please note that this kind of query (path search) can take a lot of time to compute if not well constrained. ⚠

- __get transitions from mass difference range of value__
```
MATCH (u:feature)-[e:mzdiff]-(v:feature) 
WHERE 50 < tofloat(e.massdifference) <= 70 
RETURN DISTINCT e.value
```
> While neo4j is optimized for query involving network traversal, selecting from properties values isn't its strength, compared to traditional SQL databases.
