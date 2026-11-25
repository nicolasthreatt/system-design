# API Design

## REST

- Representational State Transfer
- Built around resources or core entities
- Best practice to use plural nouns for resources

  ```Markdown
  GET /events                 # Get all events
  GET /events/{id}            # Get specific event
  GET /venues/{id}            # Get specific venue
  GET /events/{id}/tickets    # Get available tickets for an event
  POST /events/{id}/bookings  # Create new booking for an event
  GET /bookings/{id}          # Get specific booking
  ```
- HTTP Methods

  - GET (retrieve data)
  - POST (create data)
  - PUT (idempotent update/replace)
  - PATCH (partial update)
  - DELETE (remove data)

## Inputs Parameters

- Query/URL Parameters

  - Optional filters

  ```Markdown
  GET /events?city=CITY&date=DATE&venue=VENUE
  ```
- Path Parameters

  - Specify which resource you're working with
  - Use when required to identify the resource

  ```Markdown
  GET /events/{id}
  ```
- Header Parameters

  - Send metadata with the request
  - Example: authentication token

  ```Markdown
  GET /events
  Authorization: Bearer <token>
  ```
- Request Body

  ```Markdown
  POST /events
  {
      "title": "Concert",
      "description": "Live show",
      "location": "NYC",
      "date": "2026-02-01"
  }
  ```

## Responses

- Commonly contains two parts (some responses have no body):

  1. Status Code

     - Three-digit HTTP code describing the result
       - 1xx — Informational (interim updates)
       - 2xx — Success (request succeeded or was accepted)
       - 3xx — Redirection (follow another URL or reuse cached data)
       - 4xx — Client error (problem with the request)
       - 5xx — Server error (server could not fulfill the request)
     - Higher first digits progress from information and success to redirection and errors
  2. Response Body

     Success Example:

     ```json
     {
         "data": {
             "id": "123",
             "status": "confirmed"
         }
     }
     ```

     Error Example:

     ```json
     {
         "error": {
             "code": "VALIDATION_ERROR",
             "message": "date is required",
             "details": {
                 "field": "date"
             }
         }
     }
     ```

## GraphQL

- Single endpoint with strongly typed schema
- Clients request exactly the fields they need
- Supports nested queries
- Reduces over/under-fetching
- Operation Types

  - Query: read data
  - Mutation: change data
  - Subscription: receive ongoing updates

  ```HTML
  POST /graphql
  query {
      event(id: "123") {
          name
          date
          venue {
              name
              address
          }
          tickets {
              section
              price
              available
          }
      }
  }
  ```

## RPC

- Remote Procedure Call
- Intra-microservice communication

  - One microservice calls another service's procedure over the network
- Very efficient
- Often defined via IDL (Protocol Buffers) or JSON-RPC
- Requires both sides to agree on protocol and schema

  ```text
  getEvent(eventId: "123")
  createBooking(eventId: "123", userId: "456", tickets: [...])
  getAvailableTickets(eventId: "123", section: "VIP")
  ```

## Pagination

- Useful for any endpoint that will retrieve many resources
- Two Types

  1. Page

     - Client requests a page number and page size
     - Commonly uses an offset to skip earlier results
       - Offset: `(PAGE_NUM - 1) * LIMIT_NUM`
     - Easy to jump to a specific page
     - Large offsets can be slow
     - Inserts or deletions can shift results between requests

     ```http
     GET /events?page=PAGE_NUM&limit=LIMIT_NUM
     ```

     - Example: page 2 with limit 20 skips 20 items and returns up to 20
  2. Cursor

     - Client sends a token marking where to continue
     - Server returns the next batch and a new cursor
     - Treat the token as opaque
       - Send it back without interpreting its contents
     - Often tracks the last item's sort values, not an array index
     - Useful for feeds and large, changing datasets
     - Usually navigates sequentially rather than jumping to a page

     ```http
     GET /events?cursor=CURSOR_TOKEN&limit=LIMIT_NUM
     ```

     - Example: send the previous response's `next_cursor` to continue
- Stable Ordering

  - Use a consistent sort order for both types
  - Include a unique tie-breaker, such as an ID
- Common Fields

  - `next_cursor`: token for the next batch
  - `has_more`: whether more results remain
  - `total`: optional count of matching items

## Security

- Authentication

  - Verifies who is making the request
- Authorization

  - Determines what the caller can access
  - Check permissions on every request
  - Check resource ownership, not just login status
    - Example: users can only view their own bookings
  - Grant only required permissions
- HTTPS

  - Encrypts requests and responses in transit
  - Protects credentials and tokens
- JWT (JSON Web Token)

  ```http
  Authorization: Bearer <token>
  ```

  - Encodes user data and permissions
  - Signed tokens are not automatically encrypted
  - Do not include secrets in the payload
  - Verify signature, expiration, issuer, and audience
  - Includes expiry and refresh rules
- Session Token

  - A random token used to find user info server side
  - Use `Secure`, `HttpOnly`, and appropriate `SameSite` cookie settings
  - Expire sessions and invalidate them on logout
- API Keys

  - Identify calling applications
  - Scope permissions and rotate keys
  - Keep keys out of URLs and source control
- CORS (Cross-Origin Resource Sharing)

  - Browser mechanism for controlling which origins can read a response
  - An origin is a scheme, host, and port
    - Example: `https://app.example.com`
  - Browser sends the requesting origin
  - Server allows it with the `Access-Control-Allow-Origin` response header
  - Some requests trigger an `OPTIONS` preflight to check allowed methods and headers
  - Applies to browser requests; server-to-server clients are not restricted by CORS
  - Allow only required origins
  - Does not replace authentication or authorization
- CSRF (Cross-Site Request Forgery)

  - Tricks browsers into sending unwanted authenticated requests
  - Protect cookie-authenticated writes with CSRF defenses
    - CSRF tokens and appropriate `SameSite` settings
- Sensitive Data

  - Redact credentials and tokens from logs
  - Avoid exposing internal details in errors

## Versioning

- Never break existing clients
- Use explicit versioning

  ```text
  /api/v1/events
  /api/v2/events
  ```
- Deprecate old versions gradually
- Communicate breaking changes early

## Rate Limiting

- Protect APIs from abuse and overload
- Apply per-user and per-IP limits
- Return 429 when exceeded

  ```text
  X-RateLimit-Limit
  X-RateLimit-Remaining
  Retry-After
  ```

## Idempotency

- Prevent duplicate operations
- Especially important for POST requests
- Use idempotency keys

  ```http
  Idempotency-Key: abc-123
  ```

## Caching

- Improve performance and reduce load
- Cache safe GET requests
- Use HTTP cache headers

  ```http
  Cache-Control: max-age=60
  ETag
  If-None-Match
  ```

## Validation

- Validate all incoming data
  - Pydantic: Validates API input types and rules
- Enforce schemas
  - SQLAlchemy: Defines database models and constraints
- Reject invalid input early
- Normalize data formats (dates, IDs, enums)

## Observability

- Log all requests and errors
- Add correlation/request IDs
- Export metrics (latency, errors, throughput)
- Enable distributed tracing

  ```text
  X-Request-Id
  ```

## Asynchronous Operations

- Run long tasks in the background
  - Examples: exports, payment processing, video encoding
- Set timeouts and retry transient failures
- Return `202 Accepted` with a job ID
  - Acceptance does not guarantee success
- Provide status checks or completion notifications
  - Polling, callbacks, or webhooks

## Frameworks

- FastAPI
  - Python framework for building APIs
  - Generates OpenAPI definitions from routes and data models
  - Includes interactive Swagger UI at `/docs` by default
  - Provides the OpenAPI schema at `/openapi.json` by default
  - Uses Python types and Pydantic for data validation
  - Best For
    - APIs with typed request and response models
    - Services using async I/O
- Django REST Framework (DRF)
  - API toolkit built on Django
  - Integrates with Django models and ORM
  - Serializers validate and convert data
  - Includes authentication, permissions, and pagination
  - Best For
    - Existing Django applications
    - CRUD APIs backed by relational models
- Flask
  - Lightweight Python web framework
  - Provides routing and request/response handling
  - Add extensions for validation, authentication, and API documentation
  - Best For
    - Small services and prototypes
    - Projects needing flexible library choices

## Documentation

- OpenAPI
  - Standard format for describing HTTP APIs
  - Written in YAML or JSON
  - Defines endpoints, methods, inputs, responses, and authentication
- Swagger
  - Tools built around OpenAPI definitions
  - Swagger UI: interactive documentation for exploring and trying requests
  - Swagger Editor: write and edit API definitions
  - Swagger Codegen: generate client libraries and server stubs
