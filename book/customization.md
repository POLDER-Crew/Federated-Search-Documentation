# Customization

## Making your own Federated Search app

This app can serve as a template for making your own federated search application. If you want to crawl a set of data repositories and/or query DataONE for results, this might help you get started. You'll want to modify some areas of the code, though, and this chapter is about where to do that.

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
- `url`: The URL for something that tells you where to find all the data sets; in this case, it's a sitemap. TODO: what else could this be?
- `headless`: Whether or not to index it by rendering the page with a headless browser. This is significantly slower than indexing it by fetching the page and parsing the JSON-LD out of it. The reason you may want to use headless indexing is that many data repositories inject the metadata for a data set *after* its landing page loads, which means that just doing an HTTP `GET` will not get you metadata.
- `properName`: The full name for this data repository
- `domain`: The top-level domain for this data repository
- `type`: The type of thing that `url` is pointing to
- `active`: Whether to index this repository, or just keep the setting there for future use

### Searching

#### Implementing your own search class

