# mercurius

- [HTTP Status Codes](#http-status-codes)
  - [Default behaviour](#default-behaviour)
    - [Response with data](#response-with-data)
    - [Invalid input document](#invalid-input-document)
    - [Multiple errors](#multiple-errors)
    - [Single error with `statusCode` property](#single-error-with-statuscode-property)
    - [Single error with no `statusCode` property](#single-error-with-no-statuscode-property)
  - [Custom behaviour](#custom-behaviour)
    - [`200 OK` on all requests](#200-ok-on-all-requests)

Mercurius exhibits the following behaviour when serving GraphQL over HTTP.

## HTTP Status Codes

### Default behaviour

Mercurius has the following default behaviour for HTTP Status Codes.

#### Response with data

When a GraphQL response contains `data` that is defined, the HTTP Status Code is `200 OK`.

- **HTTP Status Code**: `200 OK`
- **Data**: `!== null`
- **Errors**: `N/A`

#### Invalid input document

When a GraphQL input document is invalid and fails GraphQL validation, the HTTP Status Code is `400 Bad Request`.

- **HTTP Status Code**: `400 Bad Request`
- **Data**: `null`
- **Errors**: `MER_ERR_GQL_VALIDATION`

#### Multiple errors

When a GraphQL response contains multiple errors and no data, the HTTP Status Code is `400 Bad Request`.

- **HTTP Status Code**: `400 Bad Request`
- **Data**: `null`
- **Errors**: `Multiple`

#### Single error with `statusCode` property

When a GraphQL response contains a single error with the `statusCode` property set and no data, the HTTP Status Code is set to this value. See [ErrorWithProps](./api/options.md#errorwithprops) for more details.

- **HTTP Status Code**: `Error statusCode`
- **Data**: `null`
- **Errors**: `Single`

#### Single error with no `statusCode` property

When a GraphQL response contains a single error with no `statusCode` property set and no data, the HTTP Status Code is `500 Internal Server Error`.

- **HTTP Status Code**: `500 Internal Server Error`
- **Data**: `null`
- **Errors**: `Single with no .statusCode property`

### Custom behaviour

If you wish to customise the default HTTP Status Code behaviour, one can do this using the [`errorFormatter`](./api/options.md#plugin-options) option.

#### `200 OK` on all requests

Enable `200 OK` on all requests as follows:

```js
'use strict'

const Fastify = require('fastify')
const mercurius = require('..')

const app = Fastify()

const schema = `
  type Query {
    add(x: Int, y: Int): Int
  }
`

const resolvers = {
  Query: {
    add: async (_, obj) => {
      const { x, y } = obj
      return x + y
    }
  }
}

/**
 * Define error formatter so we always return 200 OK
 */
function errorFormatter (err, ctx) {
  const response = mercurius.defaultErrorFormatter(err, ctx)
  response.statusCode = 200
  return response
}

app.register(mercurius, {
  schema,
  resolvers,
  errorFormatter
})

app.listen(3000)
```

With an invalid request:

```graphql
{
  add(wrong: 1 x: 2)
}
```

The response is:

HTTP Status Code: `200 OK`

```json
{
  "data": null,
  "errors": [
    {
      "message": "Unknown argument \"wrong\" on field \"Query.add\".",
      "locations": [
        {
          "line": 2,
          "column": 7
        }
      ]
    }
  ]
}
```