# API Endpoints

There are a few API endpoints that this app exposes.

## /api/count
The [count](https://search.polder.info/api/count) endpoint gives you a count of the datasets in the PFS triplestore - so it's a count of the ones that the PFS has indexed itself.

## /api/sparql
The [sparql](https://search.polder.info/api/sparql) endpoint allows users to make read-only SPARQL queries directly to the PFS triplestore. Requests that attempt to modify the data in the triplestore will be rejected. The 'query' parameter is required. Try it out with something like `https://search.polder.info/api/sparql?query=SELECT * { ?s ?p ?o } LIMIT 100`

## /api/search
This is used by the PFS itself, and returns rendered template partials for search results. It is mainly included here for completeness; it isn't really meant for use by third parties.
