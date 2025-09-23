<a id="simple-search"></a>

# Simple Search API

REST API allows to search by all text fields in all entities.

Parameters for APIs requests:

- **search** - the search string
- **offset** - the integer value of the offset
- **max_results** - the count of the result records in the response

REST API url: `http://domail.com/api/rest/latest/search`

REST API works with GET requests only. The search request to the search
must be as in the following example:
`http://domail.com/api/rest/latest/search?max_results=100&offset=0&search=search_string`

## Result

The request returns an array with data:

- **records_count** - the total number of results (without `offset`
  and `max_results`) parameters
- **count** - count of records in current request
- **data** - array with data. Data consists of values:
  - **entity_name** - class name of entity
  - **record_id** - id of record from this entity
  - **record_string** - the title of this record
