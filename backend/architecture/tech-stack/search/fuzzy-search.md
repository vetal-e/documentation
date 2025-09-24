<a id="elasticsearch-fuzzy-search"></a>

# Fuzzy Search

#### IMPORTANT
The feature is available for the Enterprise edition only.

This feature enables the use of error-tolerant (fuzzy) search in search index requests to Elasticsearch. It works only with the Elasticsearch search engine.

It is assumed that the first character of every word in a request is correct. So, the application does not try to change the first character to fix an error.

The exact match has higher relevancy compared to the results with errors. It means that results that match the request word by word are at the top of the result set, while those with errors are at the bottom.

The error-tolerant search is applied only to the *contains* and *not contains* operators.

<a id="elasticsearch-fuzzy-search-configuration"></a>

## Configuration

The fuzzy search options can be configured both for the back-office and storefront. The options are identical.

The back-office settings are configured under **System Configuration > General Setup > Search > Fuzzy Search** on the global level only.

The storefront settings are configured under **System Configuration > Commerce > Search > Fuzzy Search > Fuzzy Search in Storefront** on the global and website levels.

The fuzzy search options are the following:

* **Enable Fuzzy Search** enables fuzzy search in the appropriate area.
* **Error Tolerance** sets the number of errors in each word the application can ignore. The default value is *One*, meaning that one error per word can be tolerated.
* **Tolerance Starts From** sets a threshold for the error-tolerant search usage. The default value is *4*, meaning that the application uses the exact match search for small words with 1-3 characters and the error-tolerant search for words with 4+ characters.
* **Tolerance Exclusions** enables setting a regular expression for words that must not use the error-tolerant search; exact match search is used instead. This option is beneficial for SKUs, manufacturer IDs, and other identifiers that may have similar values and lead to false-positive results when the error-tolerant search is used.

<a id="elasticsearch-fuzzy-search-important-notes"></a>

## Important Notes

The error-tolerant search is not an automatic correction. It tries to find similar results for the passed request phrase word by word. It is important to remember that the error-tolerant search can lead to several false-positive results.

The error-tolerant search changes only how the request is built, not the index mapping, structure, or content. The search is performed against the tokens stored in the Elasticsearch index. So, it behaves differently for the default, language-optimized, and custom index configurations.
