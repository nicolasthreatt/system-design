# System Design Fundamentals

## Scaling

- Handle increased load
- Horizontal scaling (scale out)
  - Add machines, database nodes, or workers
  - Improves fault tolerance and availability
  - Often uses sharding or consistent hashing
  - Common in cloud systems
- Vertical scaling (scale up)
  - Add CPU, memory, or disk to one machine
  - Simple to start
  - Limited by hardware and cost
  - Can leave a single point of failure
- Best Practices
  - Measure before scaling
  - Fix design bottlenecks before adding capacity
  - Use autoscaling when appropriate

## Work Distribution

- Route work to the right machines
- Load balancing
  - Distributes incoming requests
  - Prevents overload and improves availability
- Common Strategies
  - Round robin
  - Least connections
  - Hash-based
  - Weighted routing
- Asynchronous processing
  - Use queues for background jobs
  - Decouples producers and consumers
  - Improves resilience
- Best Practices
  - Distribute load evenly
  - Detect hotspots early
  - Support graceful degradation
  - Use retries with backoff

## Data Distribution

- Decide where and how data is stored
- Shared database
  - All nodes read and write to the same database
  - Simple, but can become a bottleneck
  - May require database scaling
- Partitioning and sharding
  - Split data across nodes
  - Each node owns part of the data
  - Improves scalability
- Replication
  - Maintain multiple copies
  - Improves availability and supports failover
- Geographic partitioning
  - Partition by region
  - Can reduce latency and support regional requirements
  - Example: US, EU, and APAC shards
- Challenges
  - Race conditions
  - Data consistency
  - Synchronization
  - Network latency
- Best Practices
  - Minimize cross-node communication
  - Keep fan-out small
  - Use distributed locks carefully
  - Use eventual consistency when suitable

## CAP Theorem

- Describes tradeoffs during a network partition
- Consistency
  - Reads see the latest successful write
- Availability
  - Every request to a non-failing node receives a response
- Partition tolerance
  - System continues operating despite dropped or delayed messages between nodes
- During a partition, a distributed system must choose between consistency and availability
- Choose based on the needs of the operation
  - Example: a bank balance may require consistency
  - Example: some product views can tolerate stale data

## Locking and Concurrency

- Coordinate access to shared resources
- Lock Types
  - Operating system locks
  - Application locks
  - Database locks
  - Distributed locks
- Key Considerations
  - Granularity
    - Lock the smallest practical unit
    - Avoid global locks
  - Duration
    - Hold locks briefly
    - Minimize critical sections
  - Alternatives
    - Use optimistic locking with version numbers or timestamps
- Common Problems
  - Race conditions
  - Deadlocks
  - Starvation
- Best Practices
  - Prefer optimistic concurrency when suitable
  - Use transactions
  - Avoid nested locks
  - Add timeouts

## Indexing

- Improves read performance
- Purpose
  - Avoid full table scans
  - Speed up queries
  - Support filtering and sorting
- Basic Index Types
  - Hash index: fast equality lookups
  - B-tree index: efficient lookups and range queries
- Specialized Indexes
  - Geospatial
  - Full-text
  - Vector
  - Time-series
- External Search Indexes
  - Elasticsearch
  - OpenSearch
  - Solr
- Example Flow

  ```text
  Primary database -> CDC -> Search index
  ```

- Best Practices
  - Index fields used in queries
  - Avoid over-indexing
  - Monitor index size
  - Rebuild only when needed

## Communication Protocols

- Define how systems communicate
- Internal Communication
  - HTTP
    - Simple
    - Stateless
    - Easy to scale
  - gRPC
    - High performance
    - Strongly typed
    - Supports streaming
    - Often used between microservices
- External Communication
  - REST
    - Request and response
    - Widely supported
  - Long polling
    - Server holds a request until data is available or a timeout occurs
    - Can approximate push updates
  - Server-Sent Events (SSE)
    - Server-to-client streaming
    - One-way updates
  - WebSockets
    - Full-duplex communication
    - Useful for real-time apps such as chat, gaming, and trading
- Best Practices
  - Prefer stateless protocols when suitable
  - Centralize shared state
  - Handle disconnects
  - Use heartbeats when needed

## Security

- Authentication and authorization
  - OAuth 2.0
  - JWT
  - API keys
  - IAM roles
- Encryption
  - TLS for data in transit
  - Encryption at rest
  - Key rotation
- Data Protection
  - Least privilege
  - Field-level masking
  - Audit logging
- Best Practices
  - Use API gateways where appropriate
  - Rotate secrets
  - Validate inputs
  - Protect internal services

## Monitoring and Observability

- Needed to understand production system health
- Infrastructure Metrics
  - CPU
  - Memory
  - Disk
  - Network
- Service Metrics
  - Latency
  - Error rate
  - Throughput
  - Saturation
- Application Metrics
  - Active users
  - Sessions
  - Business metrics
- Logging
  - Centralized logs
  - Structured logs
  - Correlation IDs
- Tracing
  - Distributed traces
  - Request flow across services
- Best Practices
  - Define service level indicators (SLIs) and objectives (SLOs)
  - Alert on symptoms
  - Avoid alert fatigue
  - Monitor user experience

## Reliability and Resilience

- Design for failure
- Techniques
  - Retries with backoff
  - Timeouts
  - Circuit breakers
  - Bulkheads
  - Failover
- Disaster Recovery
  - Backups
  - Multi-region deployments
  - Runbooks
- Best Practices
  - Assume components can fail
  - Test failover regularly
  - Automate recovery

## Design Checklist

- Can the system scale horizontally?
- Is data partitioned appropriately?
- Are failures handled?
- Is security enforced?
- Is monitoring in place?
- Are APIs documented?
- Is performance measured?
