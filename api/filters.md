<a id="web-services-api-filters"></a>

# Filters

You can perform the GET and DELETE methods on a subset of resource records. A subset of records can be received by applying filters to some of the resource fields.

Available filters are listed in the **Documentation** tab of the method’s expanded area, in the **Filters** section.

To filter, perform the GET request and put your filters parameters in the query string.

**Ex 1. Filter in a Query String**

Retrieve all users of organization ‘1’.

*Request*

```http
GET /api/users?filter[organization]=1 HTTP/1.1
```

Similar to a field, a filter declares a data type and only takes specific values in the input.

Below are examples of requests and errors.

**Ex 2. Wrong Input Type**

A string value is passed as an input to a filter which can contain only integer values.

```http
GET /api/users?filter[id]=aaa HTTP/1.1

{ "errors": [{
  "status": "400",
  "title": "unexpected value exception",
  "detail": "Expected integer value. Given \"aaa\".",
  "source": {
    "parameter": "filter[id]"
  }
}] }
```

**Ex 3. Unknown Filter**

Unknown, mistyped or unsupported filter.

```http
GET /api/users?filter[unknown]=aaa HTTP/1.1

{ "errors": [{
  "status": "400",
  "title": "filter constraint",
  "detail": "Filter \"filter[unknown]\" is not supported.",
  "source": {
    "parameter": "filter[unknown]"
  }
}] }
```

The API enables you to use several types of filters. Filter types are briefly described in the table below.

| Filter   | Usage Example                                                                                                                             | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|----------|-------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| fields   | fields[owner]=id,name                                                                                                                     | Used for limiting the response data only to specified fields.<br/>Depends on the **include** filter if the filter is applied to a relation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| filter   | filter[id]=1<br/>or<br/>filter[id]=5,7<br/>or<br/>filter[id][gt]=8&filter[name]=a<br/>or<br/>filter[id][neq]=8<br/>or<br/>filter[id]=5..7 | Used for filtering the response data by specific values of a specific<br/>field.<br/><br/>A filter is specified using the syntax “key[operator_name]=value”.<br/>The full list of supported operators is described in<br/>the [Data Filter (filter)](#web-services-api-filters-data) section.<br/>The equality operator (**eq**) is optional and can be omitted, so the<br/>following filters are the same: **filter[id]=1** and **filter[id][eq]=1**.<br/><br/>May accept several values separated by comma. In such case,<br/>they will be considered connected by the logical **OR** operator,<br/>e.g. **filter[id]=5,7** represents the following expression:<br/>**id = 5 OR id = 7**.<br/><br/>May accept a data range. The syntax is “from_value..to_value”.<br/>The range is inclusive (i.e. it includes the interval boundaries,<br/>and the same range can be obtained by executing the following<br/>expression: **field >= from_value AND field <= to_value**).<br/>E.g. **filter[id]=5..7** represents the following expression:<br/>**id >= 5 AND id <= 7**.<br/><br/>And in case of several filters in request, all of them will be perceived as<br/>connected using a logical **AND** operator.<br/>E.g. **filter[id][gt]=8&filter[name]=a** represents the following<br/>expression: **id > 8 AND name = ‘a’**. |
| include  | include=[owner,organization]                                                                                                              | Used for inclusion into response the related resources data.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| page     | page[size]=10&page[number]=1                                                                                                              | Used for pagination purposes.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| sort     | sort=id<br/>or<br/>sort=id,-name                                                                                                          | Used for data sorting. By default the **ASC** sorting applies.<br/><br/>To perform **DESC** sorting specify `-` before field name.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| meta     | meta=property1,property2                                                                                                                  | Used for requesting additional meta properties for API resources.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

<a id="web-services-api-filters-fields"></a>

## Fields Filter (**fields**)

All objects are composed of fields. By default, all fields are returned in API response.

To request particular fields, use the **fields** filter and specify the fields you need in the response as its values.

#### IMPORTANT
We recommend you always to use the fields filter and retrieve only the fields you will use in your application.

**Example of Retrieving Only Required Fields**

Select the **username** and the **email** fields of the **users** resource.

> *Request*

> ```http
> GET api/users?fields[users]=username,email HTTP/1.1
> Accept: application/vnd.api+json
> ```

> *Response*

> ```json
> {
>   "data": [
>     {
>       "type": "users",
>       "id": "1",
>       "attributes": {
>         "username": "admin",
>         "email": "admin@local.com"
>       }
>     },
>     {
>       "type": "users",
>       "id": "2",
>       "attributes": {
>         "username": "sale",
>         "email": "sale@example.com"
>       }
>     }
>   ]
> }
> ```

<a id="web-services-api-filters-data"></a>

## Data Filter (**filter**)

Depending on the type of the filter, certain operators are allowed. For example, by default for integer filter type it
is allowed to use eight operators: **eq**, **neq**, **lt**, **lte**, **gt**, **gte**, **exists**, **neq_or_null**,
for string filter type - only four: **eq**, **neq**, **exists**, **neq_or_null**.
The operators **contains**, **not_contains**, **starts_with**, **not_starts_with**, **ends_with**, **not_ends_with**, **empty**
are not allowed by default and should be enabled by a developer who creates API resources.

| Operator            | Description                                                                                                                                                                                                                                                                                                                | Request Example                                                                                                      |
|---------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| **eq**              | Equality for fields<br/>and *to-one*<br/>associations<br/><br/>Contains any of<br/>specified element<br/>for *to-many*<br/>associations                                                                                                                                                                                    | GET /api/users?filter[id]=1 HTTP/1.1<br/><br/><br/>GET /api/users?filter[id][eq]=1 HTTP/1.1<br/><br/>                |
| **neq**             | Inequality for<br/>fields and *to-one*<br/>associations<br/><br/>Not contains any of<br/>specified element<br/>for *to-many*<br/>associations                                                                                                                                                                              | GET /api/users?filter[id][neq]=2 HTTP/1.1<br/><br/>                                                                  |
| **lt**              | Less than                                                                                                                                                                                                                                                                                                                  | GET /api/users?filter[id][lt]=3 HTTP/1.1<br/><br/>                                                                   |
| **lte**             | Less than or equal                                                                                                                                                                                                                                                                                                         | GET /api/users?filter[id][lte]=4 HTTP/1.1<br/><br/>                                                                  |
| **gt**              | Greater than                                                                                                                                                                                                                                                                                                               | GET /api/users?filter[id][gt]=5 HTTP/1.1<br/><br/>                                                                   |
| **gte**             | Greater than or equal                                                                                                                                                                                                                                                                                                      | GET /api/users?filter[id][gte]=6 HTTP/1.1<br/><br/>                                                                  |
| **exists**          | Is not null for<br/>fields and *to-one*<br/>associations and<br/>is not empty for<br/>*to-many* associations<br/>if filter value is<br/>*true*, *1* or *yes*<br/><br/>Is null for<br/>fields and *to-one*<br/>associations and<br/>is empty for *to-many*<br/>associations if<br/>filter value is<br/>*false*, *0* or *no* | GET /api/users?filter[id][exists]=yes HTTP/1.1<br/><br/><br/>GET /api/users?filter[id][exists]=no HTTP/1.1<br/><br/> |
| **neq_or_null**     | Inequal or is null<br/>for fields and<br/>*to-one* associations<br/><br/>Inequal or empty<br/>for *to-many*<br/>associations                                                                                                                                                                                               | GET /api/users?filter[id][neq_or_null]=test HTTP/1.1<br/><br/>                                                       |
| **contains**        | Contains a text<br/>for *string* fields<br/><br/>Contains all<br/>specified elements<br/>for *to-many*<br/>associations                                                                                                                                                                                                    | GET /api/users?filter[id][contains]=test HTTP/1.1<br/><br/>                                                          |
| **not_contains**    | Not contains a text<br/>for *string* fields<br/><br/>Not contains all<br/>specified elements<br/>for *to-many*<br/>associations                                                                                                                                                                                            | GET /api/users?filter[id][not_contains]=test HTTP/1.1<br/><br/>                                                      |
| **starts_with**     | Starts with a text                                                                                                                                                                                                                                                                                                         | GET /api/users?filter[id][starts_with]=test HTTP/1.1<br/><br/>                                                       |
| **not_starts_with** | Not starts with a text                                                                                                                                                                                                                                                                                                     | GET /api/users?filter[id][not_starts_with]=test HTTP/1.1<br/><br/>                                                   |
| **ends_with**       | Ends with a text                                                                                                                                                                                                                                                                                                           | GET /api/users?filter[id][ends_with]=test HTTP/1.1<br/><br/>                                                         |
| **not_ends_with**   | Not ends with a text                                                                                                                                                                                                                                                                                                       | GET /api/users?filter[id][not_ends_with]=test HTTP/1.1<br/><br/>                                                     |
| **empty**           | Empty or is null,<br/>e.g. an empty string<br/>or null, an empty<br/>array or null                                                                                                                                                                                                                                         | GET /api/users?filter[id][empty]=yes HTTP/1.1<br/><br/><br/>GET /api/users?filter[id][empty]=no HTTP/1.1<br/><br/>   |

#### NOTE
It is possible to use a filter by the same field several times with different operators. For example:
GET /api/users?filter[id][gte]=5&filter[id][lte]=7.

**Example of Using Operators to Filter Data**

*Request*

```http
GET /api/users?filter[id][gt]=5$page[number]=1&page[size]=2&fields[users]=username,email HTTP/1.1
Accept: application/vnd.api+json
```

*Response*

```json
{
  "data": [
    {
      "type": "users",
      "id": "6",
      "attributes": {
        "username": "jimmy.henderson_c4261",
        "email": "jimmy.henderson_c428e@example.com"
      }
    },
    {
      "type": "users",
      "id": "7",
      "attributes": {
        "username": "gene.cardenas_c760d",
        "email": "gene.cardenas_c7620@yahoo.com"
      }
    }
  ]
}
```

<a id="web-services-api-filters-inclusion"></a>

## Inclusion Filter (**include**)

As mentioned above, the **include** filter allows you to extend the response data with the related resources information.
It is usually used to reduce the number of requests to the server or, in other words, to retrieve all the necessary data
in a single request.

All included resources will be represented in **included** section at the end of the response body.

![An included section at the end of the response body](img/backend/api/api_filter_included.png)

#### IMPORTANT
Please note, in case of using **fields** filter for the main resource (e.g. users), it must contain
the field(s) used in the **include** filter.

**Example of Including Related Resources Information**

Include the **roles** relation with the **fields** filter.

*Request*

```http
GET api/users?fields[users]=username,email,roles&include=roles&page[number]=1&page[size]=1 HTTP/1.1
Accept: application/vnd.api+json
```

*Response*

```json
{
  "data": [
    {
      "type": "users",
      "id": "1",
      "attributes": {
        "username": "admin",
        "email": "admin@local.com"
      },
      "relationships": {
        "userRoles": {
          "data": [
            {
              "type": "userroles",
              "id": "3"
            }
          ]
        }
      }
    }
  ],
  "included": [
    {
      "type": "userroles",
      "id": "3",
      "attributes": {
        "extend_description": null,
        "role": "ROLE_ADMINISTRATOR",
        "label": "Administrator"
      },
      "relationships": {
        "organization": {
          "data": null
        }
      }
    }
  ]
}
```

Also, it is possible to limit fields that will be retrieved from the relation. For such purposes, the **fields** filter
should be used.

**Example of Retrieving Only Required Fields of a Related Resource**

*Request*

```http
GET api/users?fields[userroles]=label&fields[users]=username,email,roles&include=roles&page[number]=1&page[size]=1 HTTP/1.1
Accept: application/vnd.api+json
```

*Response*

```json
{
  "data": [
    {
      "type": "users",
      "id": "1",
      "attributes": {
        "username": "admin",
        "email": "admin@local.com"
      },
      "relationships": {
        "userRoles": {
          "data": [
            {
              "type": "userroles",
              "id": "3"
            }
          ]
        }
      }
    }
  ],
  "included": [
    {
      "type": "userroles",
      "id": "3",
      "attributes": {
        "label": "Administrator"
      }
    }
  ]
}
```

<a id="web-services-api-filters-pagination"></a>

## Pagination Filter (**page**)

By default, the page size is limited to 10 records and the page number is 1. However, it is possible to ask the server to
change the page size or page number to get the records that will fit your needs. Pagination
parameters should be passed as the parameters of the query string.

| Parameter name   | Type    |   Default value | Description                                                                                                                                                      |
|------------------|---------|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| page[size]       | integer |              10 | Set a positive integer number.<br/>To disable the pagination, set it as -1. In this case<br/>**page[number]** will not be taken into account and can be omitted. |
| page[number]     | integer |               1 | The number of the page.                                                                                                                                          |

**Example of Retrieving a Particular Page of a Paged Response**

Get the 2nd page of the retrieved records for the **users** resource with 20 records per page.

*Request*

```http
GET /api/users?page[number]=2&page[size]=20 HTTP/1.1
Accept: application/vnd.api+json
```

<a id="web-services-api-filters-sorting"></a>

## Sorting Filter (**sort**)

When the response to your call is a list of objects, you can also sort this list by using the sort filter with any of the
available values listed in the API reference.

**Example of Sorting by a Field Value**

Sort by **username** in descending order.

*Request*

```http
GET /api/users?filter[id][gt]=5$page[number]=1&page[size]=2&fields[users]=username,email&sort=-username HTTP/1.1
Accept: application/vnd.api+json
```

*Response*

```json
{
  "data": [
    {
      "type": "users",
      "id": "24",
      "attributes": {
        "username": "william.morrison_247fe",
        "email": "william.morrison_2482c@msn.com"
      }
    },
    {
      "type": "users",
      "id": "31",
      "attributes": {
        "username": "victor.nixon_54050",
        "email": "victor.nixon_5406f@gmail.com"
      }
    }
  ]
}
```

<a id="web-services-api-filters-meta"></a>

## Meta Property Filter (**meta**)

The **meta** filter allows you to request additional meta properties for the resource. Meta properties will be generated for every item and will be returned in the item’s meta object in the response data.

The following table contains a list of supported meta properties that may be requested using  *?meta=meta_property_name* filter:

| Name   | Description                            |
|--------|----------------------------------------|
| title  | A text representation of the resource. |

**Example of Retrieving Text Representation of the Resource**

> *Request*

> ```http
> GET api/users?meta=title HTTP/1.1
> Accept: application/vnd.api+json
> ```

> *Response*

> ```json
> {
>   "data": [
>     {
>       "type": "users",
>       "id": "1",
>       "meta": {
>         "title": "John Doe",
>       },
>       "attributes": {
>         "username": "john.doe",
>       }
>     },
>     {
>       "type": "users",
>       "id": "2",
>       "meta": {
>         "title": "Ellen Rowell",
>       },
>       "attributes": {
>         "username": "ellen.rowell"
>       }
>     }
>   ]
> }
> ```
<!-- fa-bars = fa-navicon -->
<!-- Ic Tiles is used as Set As Default in saved views, and as tiles in display layout options -->
<!-- IcPencil refers to Rename in Commerce and Inline Editing in CRM -->
<!-- Check mark in the square. -->
<!-- SortDesc is also used as drop-down arrow -->
