# Asynchronous APIs

## Overview

- Useful when work takes too long for one request
- Separate job acceptance from job completion
- Common Uses
  - Data exports
  - Payment processing
  - Video encoding

## Job Status

- Client submits work with `POST /exports`
- Server returns `202 Accepted` and a job ID

  - Acceptance does not guarantee success
- Include `Location: /exports/{id}` for status checks
- Track job status

  - Pending
  - Processing
  - Completed
  - Failed
- Keep status available if notifications fail

## Security

- Require HTTPS
- Verify signatures over the timestamp and raw body
- Use a shared secret or documented authentication scheme
- Reject stale timestamps and duplicate events
- Prevent SSRF (Server-Side Request Forgery)
  - Block internal/private callback destinations
  - Recheck resolved addresses when connecting
  - Block or revalidate redirects

## Polling

- Client repeatedly requests job status
- Example: `GET /exports/{id}`
- Stop when the job completes or fails
- Tradeoffs
  - Simple to implement
  - Client does not need a public callback endpoint
  - Repeated requests add load
  - Updates arrive only after the next status check
- Best Practices
  - Space out status checks
  - Use backoff and a maximum waiting period

## Webhooks

- Server sends HTTP notifications when subscribed events occur
- Client registers an endpoint and event types
- Common Uses
  - Payment updates
  - Booking changes
  - Integration notifications
- Example Events
  - `payment.succeeded`
  - `booking.cancelled`
- Compared with Polling
  - Avoid repeated status requests
  - Require an endpoint reachable by the sender

## Callbacks

- Notify clients when asynchronous work finishes
- Client supplies a URL for later notifications
- Reduce repeated status requests (polling)
- Common Uses

  - Data exports
  - Payment processing
  - Video encoding
- Request Flow

  1. Client submits a job with a callback URL
  2. Server returns `202 Accepted` and a job ID
     - Acceptance does not guarantee success
  3. Server processes the job
  4. Server sends a completion or failure notification
  5. Client verifies and durably queues the notification
     - Return 2xx quickly
     - Process slower business logic afterward
- Example Request

  ```http
  POST /exports
  {
      "event_id": "123",
      "callback_url": "https://client.example.com/callbacks/exports"
  }
  ```
- Example Response

  ```http
  202 Accepted
  Location: /exports/export-456
  {
      "id": "export-456",
      "status": "pending"
  }
  ```
- Example Notification

  ```http
  POST https://client.example.com/callbacks/exports
  {
      "event_id": "evt-789",
      "type": "export.completed",
      "export_id": "export-456",
      "status": "completed"
  }
  ```

  - `event_id`: notification identifier
  - `export_id`: job identifier
  - Failure event: `export.failed`
- Callbacks vs. Webhooks

  - Callback: often tied to one request
  - Webhook: often an ongoing event subscription
  - Both can use HTTP notifications
  - Terminology varies across APIs
- Delivery

  - Set delivery timeouts
  - Retry transient failures with backoff and jitter
    - Jitter adds random delay between retries
  - Define retryable responses and delivery limits
  - Deduplicate using a stable `event_id`
  - Do not assume delivery order
  - Verify notifications before processing
  - Keep `GET /exports/{id}` for status checks
  - Record failed deliveries and support redelivery

## Documentation

- Define payloads, event types, and authentication
- Document acknowledgment responses and delivery guarantees
- Explain retries, ordering, and duplicate handling
