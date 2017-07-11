# neo4j-sparql-extension-yars

Neo4j [unmanaged extension](http://docs.neo4j.org/chunked/stable/server-unmanaged-extensions.html)
for [RDF](http://www.w3.org/TR/rdf-primer/) storage and
[SPARQL 1.1 query](http://www.w3.org/TR/sparql11-protocol/) features with support for [YARS](https://www.researchgate.net/publication/309695477_RDF_Data_in_Property_Graph_Model) serialization.

## Pre-install notices

This extension depends on [sesame-rio-yars](https://github.com/lszeremeta/sesame-rio-yars) and [modified sesame-rio-api](https://github.com/lszeremeta/sesame-rio-api).

## Installation

Download the latest release from the [releases page](https://github.com/lszeremeta/neo4j-sparql-extension-yars/releases) (or build it manually) and place it
inside the `/plugins/` directory of the Neo4j server installation. [Neo4j 2.1.8](http://info.neo4j.com/download-thanks.html?edition=community&release=2.1.8&flavour=unix) is currently fully supported.

To enable the extension add it to the
`org.neo4j.server.thirdparty_jaxrs_classes` key in the
`/conf/neo4j-server.properties` file. For example:

```
org.neo4j.server.thirdparty_jaxrs_classes=de.unikiel.inf.comsys.neo4j=/rdf
```

The RDF/SPARQL extension is then avaiable as `/rdf` resource on the
Neo4j server.

Please note that if there is any data in the database that
was not imported using the `/rdf/graph` resource the plugin might crash,
because the plugin expects the data to be stored in a special way to
support RDF storage in Neo4j.

### SPARQL Protocol (SPARQL 1.1 Queries)

Base resource: `/rdf/query`

Use this resource to execute [SPARQL queries](http://www.w3.org/TR/sparql11-query/).

```bash
$ curl -v -X POST localhost:7474/rdf/query \
       -H "Content-Type: application/sparql-query" \
       -H "Accept: application/sparql-results+json" \
       -d "SELECT ?s ?p ?o WHERE { ?s ?p ?o } LIMIT 5"
```

`CONSTRUCT` and `ASK` are also supported.

```bash
$ curl -v -X POST localhost:7474/rdf/query \
       -H "Content-Type: application/sparql-query" \
       -H "Accept: text/turtle" \
       -d "CONSTRUCT ?s ?p ?o WHERE { ?s ?p ?o } LIMIT 5"
```

See [SPARQL 1.1 Protocol "2.1 query operation"](http://www.w3.org/TR/sparql11-protocol/#query-operation).

### SPARQL Graph Protocol

Base resource: `/rdf/graph`

Use this resource to add, replace or delete RDF data.

```bash
$ curl -v -X PUT \
       localhost:7474/rdf/graph \
       -H "Content-Type:text/yarsc" --data-binary @data.yarsc
```

```bash
$ curl -v localhost:7474/rdf/graph
```

See [SPARQL 1.1 Graph Store HTTP Protocol "5 Graph Management Operations"](http://www.w3.org/TR/sparql11-http-rdf-update/#graph-management).

### SPARQL Protocol (SPARQL 1.1 Update Queries)

Base resource: `/rdf/update`

Use this resource to execute [SPARQL update](http://www.w3.org/TR/sparql11-update/) queries.

```bash
$ curl -v -X POST localhost:7474/rdf/query \
       -H "Content-Type: application/sparql-update" \
       -d "@prefix dc: <http://purl.org/dc/elements/1.1/> . \
           @prefix ns: <http://example.org/ns#> . \
           <http://example/book1> ns:price 42 ."
```

See [SPARQL 1.1 Protocol "2.1 update operation"](http://www.w3.org/TR/sparql11-protocol/#update-operation).

## OWL-2 Inference

The plugin supports (limited) OWL-2 reasoning using query rewriting of SPARQL
algebra expressions. For a list of supported axioms, see
[the inference wiki page](https://github.com/niclashoyer/neo4j-sparql-extension/wiki/Inference).

To use inference the TBox must be uploaded to the special graph
`urn:sparqlextension:tbox`:

```bash
$ curl -v -X PUT \
       localhost:7474/rdf/graph\?graph=urn%3Asparqlextension%3Atbox \
       -H "Content-Type:text/turtle" --data-binary @tbox.ttl
```

Now it is possible to send SPARQL queries that additionally return
inferrend solutions. There are two ways to enable inference:

#### Using a Query Parameter

Just send your SPARQL query to `/rdf/query` and add a query parameter
`inference=true`:

```bash
$ curl -v -X POST localhost:7474/rdf/query\?inference=true \
       -H "Content-Type: application/sparql-query" \
       -H "Accept: application/sparql-results+json" \
       -d "SELECT ?s ?p ?o WHERE { ?s ?p ?o } LIMIT 5"
```

#### Using the Inference Resource

Send your SPARQL query to `/rdf/query/inference`:

```bash
$ curl -v -X POST localhost:7474/rdf/query/inference \
       -H "Content-Type: application/sparql-query" \
       -H "Accept: application/sparql-results+json" \
       -d "SELECT ?s ?p ?o WHERE { ?s ?p ?o } LIMIT 5"
```

## Chunked Imports

If you want to import a large amount of RDF data, you should enable the
chunked import. The import will be split into smaller chunks and each chunk
will be committed seperately to the database. To enable a chunked import
set the query parameter `chunked=true` when using the `/rdf/graph` resource.

## Configuration

To change the configuration add a `sparql-extension.properties` file in the
`/conf` folder of the Neo4j server installation.

The default configuration is as follows:

```
de.unikiel.inf.comsys.neo4j.query.timeout = 120
de.unikiel.inf.comsys.neo4j.query.patterns = p,c,pc
de.unikiel.inf.comsys.neo4j.inference.graph = urn:sparqlextension:tbox
de.unikiel.inf.comsys.neo4j.chunksize = 1000
```

## See also
* [sesame-rio-api](https://github.com/lszeremeta/sesame-rio-yars) - modified [Sesame](https://bitbucket.org/openrdf/sesame) API with added support for YARS serialization,
* [sesame-rio-yars](https://github.com/lszeremeta/sesame-rio-api) - YARS parser for Sesame,
* [ttl-to-yars](https://github.com/lszeremeta/ttl-to-yars) - simple [Turtle](https://www.w3.org/TR/turtle/) to YARS converter
* [yars-samples](https://github.com/lszeremeta/yars-samples) - ready to use sample YARS files

## Author
Copyright (c) 2017 [Łukasz Szeremeta](https://github.com/lszeremeta). All rights reserved.

Copyright (c) 2014 [Niclas Hoyer](https://github.com/niclashoyer). All rights reserved.

This extension based on [neo4j-sparql-extension](https://github.com/niclashoyer/neo4j-sparql-extension) originally developed by [Niclas Hoyer](https://github.com/niclashoyer).

Distributed under the [GNU General Public License version 3](https://github.com/lszeremeta/neo4j-sparql-extension-yars/blob/master/LICENSE) license.
