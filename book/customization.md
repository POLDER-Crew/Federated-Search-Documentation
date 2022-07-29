# Customization

## Making your own Federated Search app

This app can serve as a template for making your own federated search application. If you want to crawl a set of data repositories and/or query DataONE for results, this might help you get started. You'll want to modify some areas of the code, though, and this chapter is about where to do that.

### General development, building and running
For the nuts and bolts of setting up a development environment, see the [Github README](https://github.com/nein09/polder-federated-search#development).

### Indexing
Data repositories that are indexed by the Polder Federated Search app are listed in the configuration files for [Gleaner](https://gleaner.io/), which is what is actually doing the indexing here. These files are in two places, because there are two ways to deploy the federated search- with Docker, and with Helm. The Docker config file is [here](https://github.com/nein09/polder-federated-search/blob/main/docker/gleaner.yaml), and the Helm config file is [here](https://github.com/nein09/polder-federated-search/blob/main/helm/templates/gleaner-config.yaml).

Either way, these files look something like this:

```
minio:
  address: s3system
  port: 9000
  ssl: false
  bucket: gleaner
gleaner:
  runid: polder # Run ID used in prov and a few others
  summon: true # do we want to visit the web sites and pull down the files
  mill: true
context:
  cache: true
contextmaps:
- prefix: "https://schema.org/"
  file: "./schemaorg-current-https.jsonld"
- prefix: "http://schema.org/"
  file: "./schemaorg-current-https.jsonld"
- prefix: "http://schema.org/"
  file: "./schemaorg-current-https.jsonld"
summoner:
  after: ""      # "21 May 20 10:00 UTC"
  mode: full  # full || diff:  If diff compare what we have currently in gleaner to sitemap, get only new, delete missing
  threads: 5
  delay:  # milliseconds (1000 = 1 second) to delay between calls (will FORCE threads to 1)
  headless: http://headless:9222  # URL for headless see docs/headless
millers:
  graph: true
sources:
- name: nsidc
  url: https://nsidc.org/sitemap.xml
  headless: false
  properName: National Snow and Ice Data Center
  domain: https://nsidc.org
  type: sitemap
  active: false
- name: usap-dc
  type: sitemap
  headless: true
  url: https://www.usap-dc.org/view/dataset/sitemap.xml
  properName: U.S. Antarctic Program Data Center
  domain: https://www.usap-dc.org
  active: true
- name: gem
  type: sitemap
  headless: true
  url: https://data.g-e-m.dk/sitemap
  properName: Greenland Ecosystem Monitoring Database
  domain: https://data.g-e-m.dk
  active: true
```

To index other repositories, you are interested in the *sources* section of that configuration file. Each entry there is one data repository to crawl; here's what the attributes here mean:

- `name`: Your shorthand name for this data repository
- `url`: The URL for something that tells you where to find all the data sets; in this case, it's a sitemap. It can also be the url for the `robots.txt` file on that domain, if that file points to a sitemap. You can also use things like a csv, a Google Drive, or a JSON sitegraph.
- `headless`: Whether or not to index it by rendering the page with a headless browser. This is significantly slower than indexing it by fetching the page and parsing the JSON-LD out of it. The reason you may want to use headless indexing is that many data repositories inject the metadata for a data set *after* its landing page loads, which means that just doing an HTTP `GET` will not get you metadata.
- `properName`: The full name for this data repository
- `domain`: The top-level domain for this data repository
- `type`: The type of thing that `url` is pointing to, like `sitemap` or `robots`
- `active`: Whether to index this repository, or just keep the setting there for future use

### Searching
Currently, the POLDER Federated Search App supports making queries against its own [BlazeGraph](https://github.com/nein09/polder-federated-search/blob/main/app/search/gleaner.py) instance (which is where all the data you indexed goes) and [DataONE](https://github.com/nein09/polder-federated-search/blob/main/app/search/dataone.py). Note that because this is an app focused on polar data searches, it includes a [latitude filter](https://github.com/nein09/polder-federated-search/blob/main/app/search/dataone.py#L13) for the DataONE query, as a very basic way of filtering out irrelevant results. You could change that filter, or leave it out entirely.

#### Implementing your own search class
If you're querying a source other than BlazeGraph or DataONE, you'll need to add a search class to do that, and to fetch results. It should be derived from the [search base class](https://github.com/nein09/polder-federated-search/blob/main/app/search/search.py#L126). That class outlines the basic methods that a search class has to include in order to provide results to the search app; it has to be able to make queries that the app supports, and pass back results that are formatted in the way that the app expects. The [search tests](https://github.com/nein09/polder-federated-search/tree/main/app/tests/search) are a good place to look and see how such a class might be expected to operate - and if you write your own class, adding tests is always a good idea!

Once you've made your class, you can include it in the code that kicks off searches and displays results to users [here](https://github.com/nein09/polder-federated-search/blob/main/app/routes.py#L29).


### User Interface
Aside from editing verbiage in templates, you might want to customize styles and colors. The [SCSS constants](https://github.com/nein09/polder-federated-search/blob/main/app/static/css/_constants.scss) are a great place to start experimenting with that. Don't forget to assess your color choices for [sufficient contrast](https://contrastchecker.com/) so that people with vision impairments can still use the app.

Note that the UI offers a no-JavaScript experience for searchers on slow internet connections. It's easy to overlook the templates and pages associated with that, but those will also need to be updated and tested.

#### Maps
One big part of the UI is a map to show the dataset search results, which is built in [OpenLayers](https://openlayers.org/). Since the POLDER Federated Search App is meant for polar research data, it uses polar projections of the Arctic and Antarctic to display search results. Unless you're building another polar app, it's highly unlikely that you want this particular functionality. Quite a bit of the [map code](https://github.com/nein09/polder-federated-search/blob/main/app/static/js/map.js) is doing the work of building polar-projected views and making the one for Antarctica in particular a useful map that looks nice. There's also twice as much of it as you may want, because there are two maps - so if you only want one map, in the default projection on the web, don't panic if you find yourself itching to delete a bunch of stuff. We've tried to leave comments telling you which parts you are and aren't likely to need.

You may also want to read the [README](https://github.com/nein09/polder-federated-search/blob/main/app/static/maps/README.md) in the `static/maps` directory, about map styles and place names in particular, especially if you want to customize a tileset.

### Deployment
You're going to want to run this somewhere! The best place to read about deployment for this app is in the [GitHub README](https://github.com/nein09/polder-federated-search#deployment).
