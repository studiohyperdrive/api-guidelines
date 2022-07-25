# API Guidelines

# Introduction

At Studio Hyperdrive we often work in smaller teams on decoupled interfaces or microservices. While each of us individually strives to deliver a clean, robust and high-quality end result, we all have our own quirks, interests and expertise.

This document attempts to form a solid set of rules and guidelines to ensure a consistent level of quality in our APIs. As with all things in SHD, we will continue working together to make sure each and everyone of us agrees with and complies to these guidelines.

* * *

# Design Principles

## Decoupled architecture

We prefer to work in a decoupled architecture, which results in 2 key design aspects:

-   APIs can be truly RESTful
-   APIs are scalable

By approaching the API as a first-class citizen in the application or platform architecture, the focus shifts from end consumer to API consumer. We can design the API from a data point of view and abstract aggregation and interface specific knowledge to an intermediate layer.

This makes our APIs easily scalable and flexible in either a classic SPA setup or microservice architecture.

## Robustness principle

The [robustness principle](https://tools.ietf.org/html/rfc1122#page-11) states:

> _Be conservative in what you do, be liberal in what you accept from others_

Software should be written to deal with every conceivable error, no matter how unlikely. In general, it is best to assume that the network is filled with malevolent entities that will send in packets designed to have the worst possible effect.

We should always strive to document and log issues without breaking functionality or compromising security.

* * *

# General guidelines

## API specification

Each API should provide documentation in the form of an OpenAPI specification. The specification should be bundled with the code so it can be version controlled.

There are no strict guidelines on how this documentation should be written / generated, but it has to be accessible for download as a `json` file.

Additionally, a Postman collection should be provided in the code base with a clear example for each request the API provides.

> Credentials should never be included in the API spec or Postman collection.

## Manual

In case of public availability (although it is strongly suggested to also adhere to this guideline for internal APIs) the API should be well documented and provide the necessary information for consumers as well as contributors.

## Language

All API code, specification and documentation should be written in U.S. English.

* * *

# Meta information

## General information

All APIs should provide the following information in its specification:

-   `#/info/title`: a unique, functional descriptive name for the API
-   `#/info/version`: see [Semantic versioning](/API%20Guidelines.md)
-   `#/info/description`: a short, functional description of the API
-   `#/info/x-api-id`: see [Identification](/API%20Guidelines.md)

## Semantic versioning

All APIs should be versioned following the [Semantic Versioning 2.0](http://semver.org/spec/v2.0.0.html) format `1.x.x` :

- `major` version increase if an incompatible change was made
- `minor` version increase if new functionality was added, in a backwards-compatible manner
- `patch` version increase when backwards-compatible changes or fixes were made that offer no change in functionality

## Identification

All APIs should provide a unique identifier using the [UUID format](https://tools.ietf.org/html/rfc4122), to easily identify each component in a trace log.

This identifier should be added as a header on all machine-to-machine requests and contain basic information about the service (e.g.: `x-service-id=<client>.<project>.<version>`).

When passing through layers of the stack, a unique identifier (UUID) per request should be passed along or created if not provided.

This "correlation ID" makes for easier monitoring and logging and allows for tracing when debugging an issue.

The identifier should be added as a header on all machine-to-machine requests (`x-correlation-id`).

* * *

# Security

All APIs should be secured, preferably following OAuth2 standards:

-   machine-to-machine communication should follow the [Client Credentials Flow](https://auth0.com/docs/flows/concepts/client-credentials)
-   client-to-machine communication should follow the [Authorization Code Flow](https://auth0.com/docs/flows/concepts/auth-code)

If other standards are used, the API should adhere to them correctly and implement a proper security scheme.

We strongly suggest to make use of existing tools and providers from the [trusted tools & partners list](/API%20Guidelines.md).

* * *

# Compatibility

While this is highly specific to the project context and customer demands, as a general rule APIs should maintain backwards compatibility as much as possible.

## Design for the future, code for the past

API design should be conservative and focused on longevity:

-   only implement validation on relevant fields
    -   don't validate the full data model in each layer of the stack
    -   only implement strict validation for logic disrupting properties
-   be tolerant of unknown inputs
-   provide proper feedback and error codes
-   data should not be transformed when passing through a service (unless that is the main functionality of that service)

There are 2 ways to ensure your API stays compatible with older versions:

-   keep APIs compatible by implementing less strict rules and validation
-   support multiple (older) API versions

If you absolutely have to introduce a breaking change, we prefer the latter, with a clear [deprecation policy](/API%20Guidelines.md).

## Versioning

Avoid new API versions (especially major version) as much as possible, prefer creating a new resource or endpoint instead.

Add versions in the URL scheme, following semver conventions:

-   always add the major version
-   optionally add the minor version
-   optionally add the patch version

E.g.:

```javascript
/api/v1/users
/api/v1.3/users
/api/v1.3.1/users

/api/2/news
/api/2.4/news
/api/2.6.1/news
```

Always assume the latest matching version is used when a less specific version is requested.

* * *

# Deprecation

When choosing to support multiple versions to ensure backwards compatibility, make sure there is a clear deprecation policy in place.

A `Deprecation` header with the agreed upon date and time for the deprecation should be added to all requests. The specification and documentation should also be updated to inform consumers of the pending deprecation, a possible migration path and the exact date and time the deprecation will take place.

* * *

# Data formats

## JSON guidelines

All data, with the exception of binary data, should be transferred as [JSON objects](https://tools.ietf.org/html/rfc7159).

Additionally, the JSON payload must comply to:

-   using UTF-8 encoding
-   consisting of valid Unicode strings
-   containing only unique member names

as described in [RFC 7493](https://tools.ietf.org/html/rfc7493) (sections 2.1 & 2.3).

Binary data or data whose structure is not relevant (e.g. images in JPG or PNG) can be transferred using non JSON media types.

Payloads should use standard media type names, e.g. `application/json`. Custom media types should be avoided (with an exception for [media type versioning](/API%20Guidelines.md)).

## Common data types

### Dates

Date values in the JSON payload should be compliant with [RFC 3339](https://tools.ietf.org/html/rfc3339#section-5.6) (ISO8601):

-   for **date values** use the format: `date-fullyear "-" date-month "-" date-mday`, e.g. `2020-07-29`
-   for **date-time values** use the format `full-date "T" full-time`, e.g. `2020-07-29T17:35:19`

A zone offset can be added following UTC standards, preferring the shorthand notation (`2020-07-29T17:35:19Z`) over the offset notation (`2020-07-29T17:35:19+01:00`).

All dates should be stored consistently in UTC without a zone offset. Localization should be handled locally by the consumer.

Numerical timestamps should be avoided, as confusion can arise with regards to the used precision, e.g. `1460062925` vs `1460062925000` vs `1460062925.000`.

Date values used in HTTP headers should follow the format described in [RFC 7231](https://tools.ietf.org/html/rfc7231#section-7.1.1.1), e.g.:

`Wed, 29 Juli 2020 17:35:19 GMT`.

### Country codes

Country codes should follow the [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) standard.

### Language codes

Language codes should follow the [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) standard.

### Currency codes

Currency codes should follow the [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217) standard.

### Number and integer types

Numeric values should always specify precision in the API specification by choosing the correct format:

| **type** | **format** | **specified value range**  |
| -------- | ---------- | -------------------------- |
| integer  | int32      | integer between -231 and 231-1 |
| integer  | int64      | integer between -263 and 263-1 |
| integer  | bigint     | arbitrarily large signed integer number |
| number   | float      | [IEEE 754-2008/ISO 60559:2011](https://en.wikipedia.org/wiki/IEEE_754) binary32 decimal number |
| number   | double     | [IEEE 754-2008/ISO 60559:2011](https://en.wikipedia.org/wiki/IEEE_754) binary64 decimal number |
| number   | decimal    | arbitrarily precise signed decimal number |

* * *

# Common data structures

## Address

Addresses should always use the following structure:

```json
{
  "street": "string",
  "number": "string",
  "addendum": "string",
  "zipCode": "string",
  "city": "string",
  "country": "string"
}
```

If geographical data is available, it can be appended using the `location` field. The format of the location is dependent on the agreed upon location format.

```json
{
  "street": "string",
  "number": "string",
  "zipCode": "string",
  "city": "string",
  "country": "string",
  "location": {
    "lat": number,
    "lng": number
  }
}
```

Coordinates should always be transferred in [WGS84](https://en.wikipedia.org/wiki/World_Geodetic_System).

For more standard notations see [schema.org](https://schema.org/docs/schemas.html).

* * *

# API naming

**Resources** should be available on the root path, avoid using prefixes like `/api`. If a prefix is needed, it should be implemented with configuration during deployment.

**Trailing slashes** should always be avoided.

**Resource names** should always be plural, with the exception of singleton resources, e.g.:

`/employees`

`/articles`

`/status`

Use URL-friendly resource names (`[a-z0-9:._\-]`) to simplify encoding of resource IDs in URLs.

**Path segments** should use lowercase separate words with hyphens, e.g.:

`/employees/{employee-id}`

**Query parameters** should use camelCase, e.g. `lastUpdated`. The following naming conventions should be followed:

-   `query`: should be used as the default query parameter.
-   `sort`: comma-separated list of fields to define the sort order. To indicate sorting direction, fields may be prefixed with a `+` (ascending) or a `-` (descending).
-   `size`: should be used to indicate page size in case of a paginated resource.
-   `page` : should be used to indicate a numeric offset in case of a paginated resource.

**HTTP header fields** should use hyphenated-pascal-case, e.g.:

`Original-Message-ID`

* * *

# Resources

Try to be **RESTful** at all times. Consider all available entities and aim to model your API around them using the standard HTTP methods as operation indicators.

**Include all business processes** in the API, instead of focusing on the underlying data structures. By designing the API from a business process standpoint, the business logic can remain and be reflected in the API, instead of shifting to the consumer side.

**Resources should be useful** and contain as much information as necessary, while providing as little information as possible.

**Keep your URLs verb-free**, instead using the standard HTTP methods to indicate actions.

**Use domain-specific resource names** and be transparent and descriptive. A resource called `employees-events` immediately indicates what this resource should be used for, while `events` can quickly become confusing.

**Sub-resources should be identified** via path-segments, e.g.:

`/employees/{employee-id}/records/{record-id}`

**Limit the number of resource types.** If a resource is only used as a sub-resource it should not be available as a root level resource.

**Limit the number of sub-resource levels**, there should never be more than 3 levels.

* * *

# HTTP requests

## HTTP methods

HTTP methods should be used correctly, as described in [RFC 7231](https://tools.ietf.org/html/rfc7231#section-4) and [RFC 5789](https://tools.ietf.org/html/rfc5789#section-2):

-   `GET`: used to **read** either a single resource or a collection
    -   should generate a 404 for individual resources if the resource does not exist
    -   should generate either a 200 or a 404 for collections
    -   must not have a request body payload (use a POST instead)
-   `PUT`: used to **update** entire resources
    -   usually applied to individual resources
    -   should be robust against non-existence of resources (upsert)
    -   should return the entire update resource on success
    -   should generate either a 200 or 404, 201 if a resource was created
-   `POST`: used to **create** resources or for scenarios that are not sufficiently covered by the other methods
    -   should generate a 200 on successful completion
    -   should generate a 201 on resource creation
    -   should generate a 202 if the request has been accepted but has not finished
    -   should generate a 204 with `Location` header if nothing is returned
-   `PATCH`: used to **update** parts of single resources
    -   usually applied to individual resources
    -   should not be robust against non-existence of resources
    -   should return only the updated resource fields
    -   should generate a 200 or 204 if nothing is returned
    -   should be avoided if a solution with `POST`, `PUT` or [JSON Patch](https://tools.ietf.org/html/rfc6902) is an option
-   `DELETE`: used to **delete** resources
    -   usually applied to individual resources
    -   can be applied to multiple resources by filtering with query parameters
    -   should generate a 200 or 204 if nothing is returned
    -   should generate a 404 if the resource could not be found
    -   should generate a 410 if the resource has already been deleted
    -   must not have a request body payload (use a POST instead)
-   `HEAD`: used to **retrieve header information** of single resources and collections
    -   identical to `GET`, but only returns headers
    -   can be used to efficiently detect resource updates using the [ETag-header](https://tools.ietf.org/html/rfc7232#section-2.3)
-   `OPTIONS`: used to **inspect** available operations
    -   rarely implemented

## Query parameters

When using query parameters, the query language should be

-   consistent in naming
-   transparent to the resource fields
-   minimalistic

Complex query languages should use JSON or existing query languages like [GraphQL](https://graphql.org/learn/queries/) or the [Elastic Search DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html).

* * *

# HTTP status codes and errors

Status codes should be used correctly and well documented in the API specification.

Try to be as specific as possible and provide enough information to make the status of the request clear to the consumer.

## Problem JSON

In case of issues, the [Problem JSON](https://tools.ietf.org/html/rfc7807) format and media type (`application/problem+json`) should be used to provide feedback to the consumer, e.g.:

```json
{
  "type": "https://absolute/url/to/human-readable/documentation",
  "title": "Short summary of the problem type",
  "status": 500,
  "detail": "More detailed explanation of the issue that occured.",
  "instance": "https://absolute/url/to/specific/problem/occurence"
}
```

Additional properties may be defined on the problem object to further clarify the issue.

The problem JSON can be shortened to a status code and detail when working within strict security rules.

Never include stack traces or other sensitive information in the response to the consumer.

## Status codes

Below, the most commonly used status codes are documented. [Other valid status codes](https://httpstatuses.com/) are allowed, but should require some consideration before use.

### Success codes

| **Code** | **Meaning** | **Methods** |
| -------- | ----------- | ----------- |
| 200      | OK - this is the standard success responses. | all |
| 201      | Created - returned on successful entity creation. Should return either an empty response or the created resource. | POST, PUT |
| 202      | Accepted - the request was successful and will be processed asynchronously. | POST, PUT, PATCH, DELETE |
| 204      | No content - there is no response body. | HEAD, PUT, PATCH, DELETE |

### Redirection codes

| **Code** | **Meaning** | **Methods** |
| -------- | ----------- | ----------- |
| 301      | Moved permanently - this and all future requests should be redirected to the provided URL. | all |
| 302      | Moved temporarily - this request is temporarily available under a different URL. | all |
| 303      | See other - the response to the request can be found under another URI using a GET method. | POST, PUT, PATCH, DELETE |
| 304      | Not modified - indicates that a conditional GET or HEAD request would have resulted in 200 response if it were not for the fact that the condition evaluated to false, i.e. resource has not been modified since the date or version passed via request headers If-Modified-Since or If-None-Match. | GET, HEAD |

### Consumer error codes

| **Code** | **Meaning** | **Methods** |
| -------- | ----------- | ----------- |
| 400      | Bad input - generic / unknown error. Should also be delivered in case the input payload fails business logic validation. | all |
| 401      | Unauthorised - the consumer did not provide valid credentials (often the user is not authenticated). | all |
| 403      | Forbidden - the consumer is not authorised to use this resource. | all |
| 404      | Not found - the resource is not found. | all |
| 405      | Method not allowed - the method is not supported, see OPTIONS. | all |
| 406      | Not acceptable - the resource can only generate content that is not acceptable according to the Accept headers sent in the request. | all |
| 408      | Request timeout - the server times out waiting for the resource. | all |
| 409      | Conflict - the request cannot be completed due to conflict, e.g. 2 clients are trying to create the same resource or there are other concurrent, conflicting updates. | POST, PUT, PATCH, DELETE |
| 410      | Gone - the resource no longer exists. | all |
| 412      | Precondition failed - the request failed a preset condition, e.g. a validation or business rule. | POST, PUT, PATCH, DELETE |
| 415      | Unsupported media type - can occur when the content type is missing from the request body. | POST, PUT, PATCH, DELETE |
| 429      | Too many requests - the consumer did not consider rate limiting and sent too many requests. | all |

### Server side error codes

| **Code** | **Meaning** | **Methods** |
| -------- | ----------- | ----------- |
| 500      | Internal server error - a generic error to indicate an unexpected server execution problem. | all |
| 501      | Not implemented - the server cannot fulfil the request (usually implies future availability). | all |
| 503      | Service unavailable - the service is (temporarily) not available. If possible, the service should indicate how long the consumer should wait by setting the `Retry-After` header. | all |
| 504      | Gateway timeout - the server, while acting as a gateway or proxy, did not get a required response in time from the upstream server to fulfil the request. | all |

> Server side response codes are often dependent on the chosen infrastructure. Adhere to existing conventions as best as possible.

* * *

# Performance

APIs should reduce bandwidth usage as much as possible, or support techniques to achieve the same result. Especially when developing with mobile app clients in mind, the transferred data should be limited to the absolutely necessities.

Common techniques that can be implemented are:

-   compressing request and response bodies using **gzip compression**
    -   although gzip compression might be the default, the resource should support fetching uncompressed using the [Accept-Encoding](https://tools.ietf.org/html/rfc7231#section-5.3.4) request header
    -   compression should be indicated by the resource using the [Content-Encoding](https://tools.ietf.org/html/rfc7231#section-3.1.2.2) header
-   **querying field filters** to retrieve a subset of resource attributes
    -   see [Query parameters](/API%20Guidelines.md)
-   supporting **ETag and If-Match / If-None-Match headers** to avoid re-fetching of unchanged resources
    -   see the [conditional requests spec](https://tools.ietf.org/html/rfc7232) for more info
-   implementing** pagination** for large collections
    -   see [Pagination](/API%20Guidelines.md)
-   **caching master data items**, i.e. resources that change rarely or not at all after creation
    -   indicate available caching options by setting the [Cache-Control ](https://tools.ietf.org/html/rfc7234#section-5.2)header (if the header is not found, it should by default be set to `Cache-Control: no-store`)
    -   always take security and consistency into account when caching resources

* * *

# Pagination

Resources that provide access to a collection should always support pagination.

To keep endpoints consistent, the following format should be used in the response payload:

```json
{
  "_embedded": {
    "items": [],
    "total": 15,
    "pages": 5,
    "page": 1,
    "size": 25
  }
}
```

Querying a paginated resource should follow [common query parameters](/API%20Guidelines.md):

-   `query` for text searches
-   `page` to indicate where the returned resources should start
-   `size` to indicate how many resources should be retrieved

Pagination links may be provided, including an absolute path to the resource:

```json
{
  "items": [],
  "total": 15,
  "pages": 5,
  "page": 1,
  "size": 25,
  "self": "https://absolute/path/to/current/page",
  "first": "https://absolute/path/to/first/page",
  "last": "https://absolute/path/to/last/page",
  "next": "https://absolute/path/to/next/page",
  "prev": "https://absolute/path/to/prev/page"
}
```

* * *

# Resources

-   <https://opensource.zalando.com/restful-api-guidelines/#introduction>
-   <https://martinfowler.com/bliki/TolerantReader.html>
-   <https://tools.ietf.org/html/rfc1122#page-12>
-   <https://restful-api-design.readthedocs.io/en/latest/>
-   <https://www.srijan.net/blog/demystifying-the-decoupled-architecture>
-   <https://medium.com/@aliasav/perks-of-a-decoupled-architecture-fb6983890ed0>
-   <https://tools.ietf.org/html/rfc4122>
-   <https://auth0.com/docs/flows/concepts/auth-code>
          