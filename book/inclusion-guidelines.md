(inclusion-guidelines)=
# Including Your Repository

In order for your data repository to be included in the POLDER Federated Search, the indexer has to know where your data sets are and get some metadata about them. The Federated Search App takes in [JSON-LD](https://json-ld.org/) metadata in order to make data sets searchable.

Broadly, if your data repository follows the POLDER Schema.org Best Practices (soon to be released), it will be a good fit for being included in the POLDER Federated Search. The POLDER Best Practices are summarized below.

## Metadata Fields

Most of the work in getting a repository ready to be included in the POLDER Federated Search is in getting its metadata to a state where the app can consume it and use it in searches. A good thing to remember is that in order to search on a field, that field has to exist - so if you want people to be able to get your data set using, say, a date search, you have to attache temporal coverage information to it.

### Required Metadata Fields
- [Identifier](https://github.com/ESIPFed/science-on-schema.org/blob/master/guides/Dataset.md#identifier)
- [Title (schema:name)](https://schema.org/name)
- [Description](https://schema.org/description) (can be any length, although Google's data search requires it to be between 50 and 5000 characters)
- [Temporal coverage](https://github.com/ESIPFed/science-on-schema.org/blob/master/guides/Dataset.md#temporal-coverage)
- [Spatial coverage](https://github.com/ESIPFed/science-on-schema.org/blob/master/guides/Dataset.md#spatial-coverage)
- [Parameters/Variables](https://github.com/ESIPFed/science-on-schema.org/blob/master/guides/Dataset.md#variables)
- [Citation](https://schema.org/citation)
- [Creator/Author](https://github.com/ESIPFed/science-on-schema.org/blob/master/guides/Dataset.md#roles-of-people)
- [Publisher](https://github.com/ESIPFed/science-on-schema.org/blob/master/guides/Dataset.md#publisher-provider)
- [Licence](https://github.com/ESIPFed/science-on-schema.org/blob/master/guides/Dataset.md#license)


### Optional Metadata Fields

- [SameAs](https://schema.org/sameAs), especially if you're a person who doesn't like to see duplicate search results.
- [Keywords](https://schema.org/keywords) are helpful for people doing text searches.
- [Version](https://schema.org/version) in the future, this can be used to display only the most current version of a dataset. See also: [SoSo's provenance relationships guidelines](https://github.com/ESIPFed/science-on-schema.org/blob/master/guides/Dataset.md#provenance-relationships).
- Date Published
- [Distribution (i.e., how to get data)](https://github.com/ESIPFed/science-on-schema.org/blob/master/guides/Dataset.md#distributions)

## Other Requirements

Your metadata catalog should provide a [sitemap](https://github.com/ESIPFed/science-on-schema.org/blob/feature_192_sitemaps/guides/GETTING-STARTED.md#sitemaps) so that harvesters know which pages to get information from.

## Things that are nice to have

It's better and faster for indexing if your metadata is included in the data set landing page directly, instead of being injected after the page loads.

