# HTTP Fundamentals

## HTTP vs. HTTPS

- HTTP (Hypertext Transfer Protocol)
  - Exchanges requests and responses between clients and servers
  - Uses `http://`
  - Default port: 80
  - No traffic encryption
  - Network observers may read or alter traffic
- HTTPS (HTTP Secure)
  - HTTP protected by TLS
  - Uses `https://`
  - Default port: 443
  - Encrypts data in transit
  - Protects against undetected changes
  - Uses certificates to authenticate the server for the requested domain
- Both use the same methods and status codes
- Use HTTPS for production websites and APIs

## TLS vs. SSL

- SSL (Secure Sockets Layer)
  - Older protocol for securing network traffic
  - Replaced by TLS
  - Deprecated and insecure
- TLS (Transport Layer Security)
  - Successor to SSL
  - Protects HTTPS connections
  - Encrypts data in transit
  - Detects changes to transmitted data
  - Authenticates servers using certificates
  - Can also authenticate clients with mutual TLS (mTLS)
- Common Terminology
  - "SSL certificate" commonly means a certificate used with TLS
  - Modern HTTPS uses TLS, even when tools call it SSL
- Versions
  - TLS 1.0 and 1.1 are deprecated
  - Prefer TLS 1.3
  - TLS 1.2 remains supported in many systems

## HTTP Methods

- GET
  - Retrieve data
- POST
  - Submit data
  - Commonly creates a resource
- PUT
  - Create or fully replace a resource at the target URL
  - Idempotent
- PATCH
  - Partially update a resource
- DELETE
  - Remove a resource

## HTTP Status Codes

- Three-digit code describing the result
- Use an appropriate status and helpful error body

### 1xx: Informational

- Interim protocol messages
  - Usually handled by HTTP libraries, proxies, or servers
  - Not job progress notifications
- 100 Continue
  - Permits sending the request body
  - Used with `Expect: 100-continue`
- 101 Switching Protocols
  - Agrees to change protocols
  - Used by the HTTP/1.1 WebSocket handshake
- 103 Early Hints
  - Sends resource hints before the final response

### 2xx: Success

- 200 OK

  - Request succeeded
  - Example: `GET /events/123` returns the event
- 201 Created

  - Resource created
  - Example: `POST /events`
  - Include `Location: /events/123`
- 202 Accepted

  - Request accepted for processing
  - Work may still be pending or fail later
  - Example: `POST /exports` queues a job
  - Check completion through status requests or callbacks
- 204 No Content

  - Request succeeded with no response body
  - Example: successful `DELETE /events/123`
- 206 Partial Content

  - Returns part of a resource
  - Example: requested byte range of a large file
- Common Decisions

  - 200 vs. 201 vs. 202
    - Completed normally vs. created a resource vs. accepted work

### 3xx: Redirection

- Redirects normally use the `Location` header
- 301 Moved Permanently / 302 Found
  - Permanent / temporary redirect
  - Clients may change POST to GET
- 303 See Other
  - Directs the client to retrieve another resource
  - Example: GET a confirmation page after form submission
- 304 Not Modified
  - Conditional GET matched the cached version
  - Reuse cached data
  - No response body
  - Does not redirect to another URL
- 307 Temporary Redirect / 308 Permanent Redirect
  - Preserves the request method and body

### 4xx: Client Error

- 400 Bad Request

  - Malformed request
  - Example: invalid JSON
- 401 Unauthorized

  - Missing or invalid credentials
  - Include a `WWW-Authenticate` challenge
- 403 Forbidden

  - Server refuses the request
  - Example: attendee attempts an organizer-only action
- 404 Not Found

  - Resource not found
  - Example: `GET /events/unknown`
- 405 Method Not Allowed

  - Method unsupported for this resource
  - Example: POST to a GET-only route
  - Include `Allow: GET, HEAD`
- 408 Request Timeout

  - Client took too long to send the request
- 409 Conflict

  - Request conflicts with current resource state
  - Example: booking an already reserved seat
- 410 Gone

  - Resource is known to be permanently unavailable
- 412 Precondition Failed

  - Request condition did not match
  - Example: `If-Match` contains an old ETag
  - Prevents overwriting newer changes
- 413 Content Too Large

  - Request content exceeds the size limit
  - Example: oversized image upload
- 415 Unsupported Media Type

  - Unsupported request content format
  - Example: XML sent to a JSON-only endpoint
- 422 Unprocessable Content

  - Valid format with invalid meaning
  - Example: event end date precedes start date
  - Historically called Unprocessable Entity
- 429 Too Many Requests

  - Rate limit exceeded
  - May include `Retry-After`
- Common Decisions

  - 401 vs. 403
    - Missing/invalid credentials vs. refused request
  - 400 vs. 422
    - Malformed request vs. invalid meaning
    - Validation conventions vary across APIs
    - Choose and document a consistent convention
  - 409 vs. 422
    - Conflict with resource state vs. invalid submitted values
  - Empty Results
    - `GET /events` with no matches returns 200 and an empty list
    - `GET /events/unknown` returns 404 for a missing resource

### 5xx: Server Error

- 500 Internal Server Error

  - Unexpected server failure
  - Example: unhandled application exception
- 501 Not Implemented

  - Server lacks support for requested functionality
- 502 Bad Gateway

  - Gateway received an invalid upstream response
- 503 Service Unavailable

  - Temporary overload or maintenance
  - May include `Retry-After`
- 504 Gateway Timeout

  - Gateway waited too long for an upstream response
- Common Decisions

  - 408 vs. 504
    - Receiving the client's request vs. waiting on an upstream server
  - Retries
    - Retry only when appropriate
    - Use backoff
    - Prevent duplicate operations
    - A timeout or 5xx does not prove a write never happened

## Cookies

- Small name-value pairs stored by the browser
- Help maintain state across HTTP requests
- Categories by Purpose

  - Essential
    - Support core site functions
    - Examples: login sessions, shopping carts
  - Preference
    - Remember user choices
    - Examples: language, theme, region
  - Analytics
    - Measure site usage and performance
    - Examples: page visits, navigation patterns
  - Advertising
    - Support ad targeting and measurement
    - Examples: interests, ad interactions
  - These categories describe purpose, not HTTP cookie attributes
- How They Work

  - Server sends a `Set-Cookie` response header
  - Browser stores the cookie
  - Browser sends it in the `Cookie` header on matching requests
  - Session cookies often contain an ID for server-side session data
- Lifetime

  - Session cookie: no explicit expiration
    - Browser session restore may preserve it
  - Persistent cookie: expires using `Max-Age` or `Expires`
  - `Max-Age`: lifetime in seconds
  - `Expires`: expiration date
- Important Attributes

  - `Secure`: send only over HTTPS
  - `HttpOnly`: block JavaScript access
  - `SameSite`: control sending on cross-site requests
    - `Strict`: most restrictive
    - `Lax`: allows some top-level navigations
    - `None`: permits cross-site use; requires `Secure`
  - `Domain`: eligible hosts
  - `Path`: eligible URL paths
- Security

  - Use `Secure` and `HttpOnly` for session cookies
  - Use appropriate `SameSite` settings and CSRF protection
  - Avoid storing passwords in cookies
- Example

  ```YAML
  Set-Cookie: session_id=abc123; Path=/; Max-Age=3600; Secure; HttpOnly; SameSite=Lax
  ```
  ```YAML
  Cookie: session_id=abc123
  ```
