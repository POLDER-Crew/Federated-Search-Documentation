# Using Blazegraph as your triplestore

This project originally used Blazegraph instead of GraphDB. We changed because we wanted GraphDB's GeoSPARQL support and nice development console - but GraphDB is not open source, although a free version is available. If you wish to build a project that only has open-source software in it, you can use Blazegraph instead.

## Changes to the search code

The differences between BlazeGraph and GraphDB necessitate some changes to the Python code in the web app. The first section is the most important. Please note that tests will need to be changed accordingly, as well.

### POST versus GET
BlazeGraph allows large queries to be sent via a `GET`, so calling `self.sparql.setMethod(POST)` in `app/search/gleaner.py` is optional.

### SPARQL query
BlazeGraph allows named subqueries and uses a different full-text search engine, which means that the template string in `app/search/gleaner.py` should look more like this:

```
  return f"""
      PREFIX schema: <https://schema.org/>

      SELECT ?total_results ?score ?id ?abstract ?url ?title ?sameAs ?keywords ?license ?temporal_coverage ?spatial_coverage ?author

      WITH {{
      SELECT
          (MAX(?relevance) AS ?score)
          ?id
          ?url
          ?title
          ?author
          ?license
          (GROUP_CONCAT(DISTINCT ?abstract ; separator=", ") as ?abstract)
          (GROUP_CONCAT(DISTINCT ?sameAs ; separator=", ") as ?sameAs)
          (GROUP_CONCAT(DISTINCT ?keywords ; separator=", ") as ?keywords)
          (GROUP_CONCAT(DISTINCT ?temporal_coverage ; separator=", ") as ?temporal_coverage)
          (GROUP_CONCAT(DISTINCT ?spatial_coverage ; separator=", ") as ?spatial_coverage)

      {{

          ?s a schema:Dataset  .
          ?s schema:name ?title .
          {{ ?s schema:keywords ?keywords . }} UNION {{
              ?catalog ?relationship ?s .
              ?catalog schema:keywords ?keywords .
          }}
          ?s schema:description | schema:description/schema:value  ?abstract .
          ?s schema:temporalCoverage ?temporal_coverage .
          ?s schema:spatialCoverage ?spatial_coverage .

          OPTIONAL {{
              ?s schema:sameAs ?sameAs .
          }}
          OPTIONAL {{
              ?s schema:license ?license .
          }}
          OPTIONAL {{
              ?s schema:url ?url .
          }}
          OPTIONAL {{
              ?s schema:identifier | schema:identifier/schema:value ?id .
              FILTER(ISLITERAL(?id)) .
          }}
          OPTIONAL {{
              ?s schema:creator/schema:name ?author .
          }}
        {user_query}
        BIND(COALESCE(?id, ?s) AS ?id)
      }}
      GROUP BY ?id ?url ?title ?license ?author
  }} AS %search
  {{
      {{
          SELECT (COUNT(*) as ?total_results)
          {{ INCLUDE %search . }}
      }}
      UNION
      {{
          SELECT ?score ?s ?id ?abstract ?url ?title ?sameAs ?keywords ?license ?temporal_coverage ?spatial_coverage ?author
          {{ INCLUDE %search . }}
          OFFSET {page_start}
          LIMIT {GleanerSearch.PAGE_SIZE}
      }}
  }}
      ORDER BY DESC(?total_results) DESC(?score)
  """
```

Additionally, the method that builds your full text search should look like this:

```
    @staticmethod
    def _build_text_search_query(text=None):
        if text:
            return f"""
                ?lit bds:search '''{text}''' .
                ?lit bds:matchAllTerms "false" .
                ?lit bds:relevance ?relevance .
                ?s ?p ?lit .
            """
        else:
            # A blank search in this will give NO results, which seems like
            # the opposite of what we want.
            return ""
```


### Full-text search ranking
Because BlazeGraph uses a different full-text search algorithm than DataONE, you may wish to normalize DataONE's search scores; all BlazeGraph scores are normalized to be between 0 and 1.

In order to do this, you might modify `app/search/dataone.py` in the following fashion:

You can get the maximum score from the DataONE response body like so:

```
self.max_score = body['maxScore']
```

and apply it to the `SearchResult` object like

```
return SearchResult(
   score=(result.pop('score') / self.max_score),
   ...
```

## Changes to docker-compose and Helm charts

### Setting up Docker with Blazegraph
Assuming that you're starting from **this directory**:
1. To build and run the web app, Docker needs to know about some environment variables. There are examples ones in `dev.env` - copy it to `.env` and fill in the correct values for you. Save the file and then run `source .env`.
1. Install [Docker](https://docker.com)
   1. Also, For windows, click [here](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi) to install the WSL 2 (Windows Subsystem for Linux, version 2)
1. `cd docker`
1. `docker-compose up -d`
1. `docker-compose --profile setup up -d` in order to start all of qthe necessary services and set up Gleaner for indexing.
1. If you want to try queries out on Blazegraph directly, Go to your [local Blazegraph instance](http://localhost:9999/blazegraph/#namespaces) and be sure to click 'Use' next to the polder namespace.
NOTE: There is a missing step here. The crawled results need to be written to the triplestore. For now, you can run `./write-to-triplestore.sh`.
     1. For windows, you need to download [Cygwin](https://www.cygwin.com/setup-x86_64.exe).
     1. Change directory to the docker in Cygwin (`cd docker`).
     1. Run the `./write-to-triplestore.sh`. to write to triplestore.
1. Run the web app: `docker-compose --profile web up -d`

## Differences in docker-compose.yaml
The `triplestore-setup` and `triplestore` sections will need to be changed to something like the following; the exact images are up to you, of course, but here are the ones that the Polder Federated Search App was using at the time the switch to GraphDB was made:

### triplestore-setup
```
triplestore-setup:
    image: curlimages/curl:7.82.0
    depends_on:
    - triplestore
    command:
    - curl
    - -X
    - POST
    - -H
    - 'Content-type: application/xml'
    - --data
    - >
      <?xml version="1.0" encoding="UTF-8" standalone="no" ?>
        <!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
        <properties>
          <entry key="com.bigdata.rdf.store.AbstractTripleStore.textIndex">true</entry>
          <entry key="com.bigdata.rdf.store.AbstractTripleStore.axiomsClass">com.bigdata.rdf.axioms.NoAxioms</entry>
          <entry key="com.bigdata.rdf.sail.isolatableIndices">false</entry>
          <entry key="com.bigdata.rdf.sail.truthMaintenance">false</entry>
          <entry key="com.bigdata.rdf.store.AbstractTripleStore.justify">false</entry>
          <entry key="com.bigdata.rdf.sail.namespace">polder</entry>
          <entry key="com.bigdata.rdf.store.AbstractTripleStore.quads">true</entry>
          <entry key="com.bigdata.namespace.polder.spo.com.bigdata.btree.BTree.branchingFactor">1024</entry>
          <entry key="com.bigdata.journal.Journal.groupCommit">false</entry>
          <entry key="com.bigdata.namespace.polder.lex.com.bigdata.btree.BTree.branchingFactor">400</entry>
          <entry key="com.bigdata.rdf.store.AbstractTripleStore.geoSpatial">true</entry>
          <entry key="com.bigdata.rdf.store.AbstractTripleStore.statementIdentifiers">false</entry>
        </properties>
    - 'http://triplestore:8080/bigdata/namespace'

```

### triplestore
```
  triplestore:
    image: islandora/blazegraph:1.0.0-alpha-15
    ports:
      - 9999:8080
    volumes:
      - triplestore:/var/lib/blazegraph
```

### Docker scripts
The above instructions mention `write-to-triplestore.sh`. There is also `clear-triplestore.sh`. Those are both a little different with BlazeGraph; here is what they will need to look like:

#### write-to-triplestore

```
#! /bin/bash

mc config host add minio http://localhost:9000
for i in $(mc find minio/gleaner/milled); do
  mc cat $i | curl -X POST -H 'Content-Type:text/rdf+n3;charset=utf-8' --data-binary  @- http://localhost:9999/bigdata/namespace/polder/sparql
done
```

#### clear-triplestore
```
#! /bin/bash
curl --get -X DELETE -H 'Accept: application/xml' http://localhost:9999/bigdata/namespace/polder/sparql
```

## Helm

In `helm/templates/_helpers.tpl`, your `gleaner.triplestore.endpoint` definition will become

```
http://triplestore-svc.{{ .Release.Namespace }}.svc.cluster.local:{{ .Values.triplestore_service.port }}/bigdata/
```

In `values-*.yaml`, the triplestore port is different:

```
triplestore_service:
  type: ClusterIP
  port: 8080
```

And `GDB_JAVA_OPTS` in `values-*.yaml` is replaced by

```
javaXMS: 2g
javaXmx: 8g
javaOpts: -Xmx6g -Xms2g --XX:+UseG1GC
```

`GLEANER_ENDPOINT_URL`, wherever it appears, will be `value: {{ include "gleaner.triplestore.endpoint" . }}namespace/polder/sparql/`

### Writing to the triplestore
This occurs in a few Helm templates; anything called `write-to-triplestore` will look like this:

```
 - name: write-to-triplestore
   image: bitnami/minio-client:2022
   env:
   - name: MINIO_CLIENT_ACCESS_KEY
     valueFrom:
       secretKeyRef:
         key:  minioAccessKey
         name: {{ .Release.Name }}-secrets
   - name: MINIO_CLIENT_SECRET_KEY
     valueFrom:
       secretKeyRef:
         key:  minioSecretKey
         name: {{ .Release.Name }}-secrets
   - name: MINIO_SERVER_HOST
     value: {{ include "gleaner.s3system.endpoint" . }}
   - name: MINIO_SERVER_PORT_NUMBER
     value: "{{ .Values.s3system_service.api_port }}"
   command:
   # the first line of the following bash command is supposed to happen automatically, according to
   # the documentation on docker hub, but it does not.
   - /bin/bash
   - -c
   - >
     mc config host add minio "http://${MINIO_SERVER_HOST}:${MINIO_SERVER_PORT_NUMBER}" "${MINIO_CLIENT_ACCESS_KEY}" "${MINIO_CLIENT_SECRET_KEY}" &&
     for i in $(mc find minio/{{ .Values.storageNamespace }}/milled); do
       mc cat $i | curl -X POST -H 'Content-Type:text/rdf+n3' --data-binary  @- {{ include "gleaner.triplestore.endpoint" . }}namespace/{{ .Values.storageNamespace }}/sparql
     done
```

### Clearing the triplestore
This occurs in a few Helm templates; anything called `clear-triplestore` will look like this:
```
 - name: clear-triplestore
   image: curlimages/curl:7.82.0
   command:
   - curl
   - --get
   - -X
   - DELETE
   - -H
   - 'Accept: application/xml'
   - {{ include "gleaner.triplestore.endpoint" . }}namespace/{{ .Values.storageNamespace }}/sparql
```

### Setting up the triplestore
`setup-triplestore` and `wait-for-triplestore-up` should be, respectively:

```
- name: wait-for-triplestore-up
  image: curlimages/curl:7.82.0
  command:
  - /bin/sh
  - -c
  # yes, this is how it has to work, no I am not happy about it
  - >
    set -x;
    while [ $(curl -sw '%{http_code}' "{{ include "gleaner.triplestore.endpoint" . }}" -o /dev/null) -ne 200 ]; do
      sleep 15;
    done
# Next, create the blazegraph namespace that we want to use for this app
- name: setup-triplestore
  image: curlimages/curl:7.82.0
  command:
  - curl
  - -X
  - POST
  - -H
  - 'Content-type: application/xml'
  - --data
  - >
    <?xml version="1.0" encoding="UTF-8" standalone="no" ?>
      <!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
      <properties>
        <entry key="com.bigdata.rdf.store.AbstractTripleStore.textIndex">true</entry>
        <entry key="com.bigdata.rdf.store.AbstractTripleStore.axiomsClass">com.bigdata.rdf.axioms.NoAxioms</entry>
        <entry key="com.bigdata.rdf.sail.isolatableIndices">false</entry>
        <entry key="com.bigdata.rdf.sail.truthMaintenance">false</entry>
        <entry key="com.bigdata.rdf.store.AbstractTripleStore.justify">false</entry>
        <entry key="com.bigdata.rdf.sail.namespace">{{ .Values.storageNamespace }}</entry>
        <entry key="com.bigdata.rdf.store.AbstractTripleStore.quads">true</entry>
        <entry key="com.bigdata.namespace.{{ .Values.storageNamespace }}.spo.com.bigdata.btree.BTree.branchingFactor">1024</entry>
        <entry key="com.bigdata.journal.Journal.groupCommit">false</entry>
        <entry key="com.bigdata.namespace.{{ .Values.storageNamespace }}.lex.com.bigdata.btree.BTree.branchingFactor">400</entry>
        <entry key="com.bigdata.rdf.store.AbstractTripleStore.geoSpatial">true</entry>
        <entry key="com.bigdata.rdf.store.AbstractTripleStore.statementIdentifiers">false</entry>
      </properties>
  - '{{- include "gleaner.triplestore.endpoint" . }}namespace'
```
