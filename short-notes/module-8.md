# Module 8: System Design Fundamentals

## Scalability Concepts

### Scaling Strategies

**Vertical Scaling (Scale Up)**

Vertical scaling involves increasing the resources of a single server by adding more CPU, RAM, or storage. This approach is simple to implement as it doesn't require architectural changes, but has inherent limitations due to hardware constraints and single points of failure. When the maximum hardware capacity is reached, you must migrate to a larger server, which can cause downtime. Vertical scaling is suitable for early-stage applications with predictable growth patterns.

- **Pros**: Simple implementation, no code changes, easier to manage
- **Cons**: Hardware limitations, single point of failure, expensive at scale, migration downtime
- **Use case**: Early-stage applications, predictable workloads

**Horizontal Scaling (Scale Out)**

Horizontal scaling involves adding more servers to distribute load across multiple instances. This approach requires stateless services to ensure any server can handle any request. Load balancers distribute traffic across servers, and if one fails, others continue serving requests. Horizontal scaling offers virtually unlimited scalability and improved fault tolerance, but introduces complexity in maintaining consistent state across servers and coordinating distributed operations.

- **Pros**: Virtually unlimited scaling, fault tolerance, cost-effective (commodity hardware), no single point of failure
- **Cons**: Requires stateless architecture, distributed complexity, data consistency challenges
- **Use case**: High-traffic applications, unpredictable growth, fault-critical systems

```javascript
// Stateless service requirement for horizontal scaling
// BAD: State stored in memory (cannot scale horizontally)
let userSessions = {}; // Lost when server restarts

// GOOD: State stored externally (can scale horizontally)
const sessions = await redis.get(`session:${userId}`);
```

**The Scalability Cube**

The scalability cube provides a framework for scaling applications across three dimensions: X-axis (cloning), Y-axis (partitioning/sharding), and Z-axis (data partitioning). X-axis cloning involves creating identical copies of application instances behind a load balancer, which is the simplest form of horizontal scaling. Y-axis partitioning splits the application based on functional decomposition, where different services handle different business capabilities. Z-axis data partitioning distributes data based on attributes like user ID or geographic region, enabling efficient data access and localized processing.

```
          Z-Axis (Data Partitioning)
          ┌─────────────────────────┐
          │  Shard 1 │ Shard 2  │
          │ (Users A-M) (Users N-Z)│
          └─────────────────────────┘
                   │
    Y-Axis (Partitioning)
    ┌─────────────────────────────┐
    │ Auth │ Orders │ Payments   │
    │ Service │ Service │ Service   │
    └─────────────────────────────┘
                   │
    X-Axis (Cloning)
    ┌─────────────────────────────┐
    │ Instance 1 │ Instance 2  │
    │ (Clone)   │ (Clone)      │
    └─────────────────────────────┘
```

### Performance Metrics

**Latency vs Throughput**

Latency measures the time taken to process a single request, typically measured in milliseconds. Lower latency provides better user experience, especially for interactive applications. Throughput measures the number of requests processed per second (RPS), indicating system capacity under load. These metrics often trade off: optimizing for maximum throughput might increase latency, while minimizing latency might reduce throughput. System design must balance both based on application requirements—real-time applications prioritize low latency, while batch processing systems maximize throughput.

- **Latency**: Time per request (ms), affects user experience
- **Throughput**: Requests per second (RPS), indicates system capacity
- **Trade-off**: Often inverse relationship, must balance based on use case

```javascript
// Measuring latency and throughput
const startTime = Date.now();
await processRequest();
const latency = Date.now() - startTime;

const requestsPerSecond = totalRequests / timeElapsed;
```

**Availability and Reliability**

Availability measures the percentage of time a system is operational and accessible, often expressed as "nines" (99.9% = 8.76 hours downtime/year). Reliability measures the system's ability to function correctly without failures over time. High availability requires redundancy (multiple instances, multi-region deployment) and failover mechanisms. Reliability requires robust error handling, monitoring, and automated recovery. These metrics are critical for production systems where downtime directly impacts revenue and user trust.

- **Availability**: Percentage of uptime (99.9%, 99.99%, 99.999%)
- **Reliability**: Consistent correct operation over time
- **Redundancy**: Multiple instances, failover, multi-region

**Consistency Models**

Consistency models define how and when data updates propagate across distributed systems. Strong consistency ensures all reads return the most recent write, providing a single source of truth but potentially higher latency and reduced availability. Eventual consistency allows reads to return stale data temporarily, with updates propagating asynchronously, offering lower latency and higher availability at the cost of temporary inconsistency. The choice depends on application requirements—financial systems need strong consistency, while social media can tolerate eventual consistency.

- **Strong consistency**: All reads return latest write (higher latency, lower availability)
- **Eventual consistency**: Updates propagate asynchronously (lower latency, higher availability)
- **CAP theorem**: Can only achieve 2 of 3 (Consistency, Availability, Partition tolerance)

---

## Load Balancing

### Load Balancer Fundamentals

**L4 vs L7 Load Balancers**

Layer 4 (L4) load balancers operate at the transport layer (TCP/UDP), making routing decisions based on IP addresses and ports. They're fast and efficient but unaware of application content. Layer 7 (L7) load balancers operate at the application layer (HTTP), inspecting request headers, URLs, and cookies for intelligent routing. L7 balancers enable advanced features like SSL termination, path-based routing, and session affinity, but introduce more processing overhead. The choice depends on requirements—L4 for high-throughput simple routing, L7 for intelligent traffic distribution.

- **L4 (Transport layer)**: IP/port routing, fast, unaware of content
- **L7 (Application layer)**: HTTP-aware routing, advanced features, more overhead

**Load Balancing Algorithms**

Round robin distributes requests sequentially to each server in turn, ensuring equal distribution but not accounting for server capacity or current load. Least connections routes to the server with fewest active connections, ideal for requests with varying processing times. IP hash routes requests from the same IP to the same server consistently, useful for session persistence. Weighted round robin assigns more requests to more powerful servers, enabling heterogeneous infrastructure. Choosing the right algorithm depends on request characteristics and infrastructure heterogeneity.

```javascript
// Load balancing algorithms

// Round robin: Sequential distribution
let currentIndex = 0;
function roundRobin(servers) {
  const server = servers[currentIndex];
  currentIndex = (currentIndex + 1) % servers.length;
  return server;
}

// Least connections: Route to least busy server
function leastConnections(servers) {
  return servers.reduce((min, server) =>
    server.connections < min.connections ? server : min
  );
}

// IP hash: Consistent routing for same IP
function ipHash(servers, clientIP) {
  const hash = crypto.createHash('md5')
    .update(clientIP)
    .digest('hex');
  const index = parseInt(hash, 16) % servers.length;
  return servers[index];
}
```

**Health Checks**

Health checks continuously monitor server availability and responsiveness, directing traffic away from unhealthy instances. Load balancers send periodic requests (HTTP GET, TCP ping) to each server and mark them unhealthy if they fail to respond within timeout or return error status. Unhealthy servers are removed from rotation until they recover and pass health checks. This automatic failover ensures high availability and prevents users from experiencing errors from failed servers. Health check configuration must balance sensitivity—too aggressive causes unnecessary removals, too lenient delays failover.

- **Purpose**: Monitor server availability, automatic failover
- **Methods**: HTTP GET, TCP ping, custom health endpoints
- **Configuration**: Timeout, interval, unhealthy threshold

**Session Persistence (Sticky Sessions)**

Session persistence ensures requests from the same client are routed to the same server, maintaining session state stored in server memory. This is implemented using IP hash or inserting cookies that identify the target server. While simple to implement, sticky sessions create uneven load distribution and single points of failure—when a server fails, all its sessions are lost. Modern applications prefer stateless architectures with external session storage (Redis, database) to avoid this limitation, enabling any server to handle any request.

- **Implementation**: IP hash, cookie-based routing
- **Problem**: Uneven load, single point of failure, lost sessions on failure
- **Alternative**: Stateless services with external session storage

### AWS Load Balancers

**Application Load Balancer (ALB)**

ALB operates at Layer 7 (application layer), providing intelligent HTTP/HTTPS routing based on path, host header, and query parameters. It supports SSL termination, reducing load on application servers, and integrates with AWS services like EC2, ECS, and Lambda. ALB enables advanced routing rules like directing `/api/*` to API servers and `/static/*` to CDN. It's ideal for web applications requiring content-based routing, WebSocket support, and HTTP/2 capabilities.

- **Layer 7**: HTTP/HTTPS routing
- **Features**: Path-based routing, SSL termination, WebSocket support
- **Integration**: EC2, ECS, Lambda

**Network Load Balancer (NLB)**

NLB operates at Layer 4 (transport layer), providing ultra-low latency TCP/UDP routing with millions of requests per second. It preserves source IP addresses and supports static IP addresses, making it ideal for applications requiring true IP visibility. NLB doesn't inspect HTTP content, so it's faster but lacks advanced routing capabilities. Use NLB for high-throughput TCP applications, gaming servers, and scenarios requiring static IPs or preserving client IP addresses.

- **Layer 4**: TCP/UDP routing
- **Features**: Ultra-low latency, static IPs, preserves source IP
- **Use case**: High-throughput TCP, gaming, IP visibility

**Classic Load Balancer (CLB)**

CLB is the legacy load balancer providing basic Layer 4 and Layer 7 capabilities. It supports both TCP and HTTP traffic but lacks advanced features like path-based routing and WebSocket support. CLB is maintained for backward compatibility, but new deployments should use ALB or NLB. Migration from CLB to ALB/NLB provides better features, performance, and integration with modern AWS services.

- **Legacy**: Basic L4/L7 capabilities
- **Limitations**: No path-based routing, no WebSocket support
- **Recommendation**: Use ALB or NLB for new deployments

### Load Balancer Placement

Load balancers can be placed at multiple layers in the architecture to distribute load effectively. Between users and web servers, load balancers distribute incoming traffic across multiple web server instances, handling SSL termination and DDoS protection. Between web servers and app servers, another load balancer can distribute application logic processing across multiple instances. Between app servers and database, load balancers can distribute read queries across read replicas while directing writes to the master. Multi-tier load balancing optimizes resource utilization and enables independent scaling of each tier.

```
Users
  ↓
[Load Balancer] → Web Server 1, 2, 3
  ↓
[Load Balancer] → App Server 1, 2, 3
  ↓
[Load Balancer] → DB Read Replica 1, 2, 3
  ↓
DB Master (writes only)
```

---

## Caching Strategies

### Caching Fundamentals

**Cache Hit vs Cache Miss**

A cache hit occurs when requested data is found in the cache, returning immediately without hitting the primary data store. A cache miss occurs when data is not in cache, requiring retrieval from the primary store and subsequent cache population. Cache hit ratio measures effectiveness—higher is better, but depends on access patterns. Understanding hit/miss patterns helps optimize cache size, eviction policies, and preloading strategies. Caching is most effective for read-heavy workloads with repetitive access to the same data.

- **Cache hit**: Data found in cache (fast, no primary store access)
- **Cache miss**: Data not in cache (slow, requires primary store access)
- **Goal**: Maximize hit ratio, minimize misses

**Cache Eviction Policies**

When cache is full, eviction policies determine which items to remove. LRU (Least Recently Used) evicts items that haven't been accessed for the longest time, assuming recent access predicts future access. LFU (Least Frequently Used) evicts items with lowest access count, assuming popularity predicts future access. FIFO (First In First Out) evicts oldest items regardless of access pattern. LRU is most common due to good performance and simplicity, while LFU works well for stable access patterns. Choosing the right policy depends on access patterns and cache size.

- **LRU**: Evict least recently used (most common)
- **LFU**: Evict least frequently used (stable patterns)
- **FIFO**: Evict oldest items (simple)

```javascript
// Simple LRU cache implementation
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }

  get(key) {
    if (this.cache.has(key)) {
      const value = this.cache.get(key);
      this.cache.delete(key); // Remove and re-add to update order
      this.cache.set(key, value);
      return value;
    }
    return null; // Cache miss
  }

  set(key, value) {
    if (this.cache.size >= this.capacity) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey); // Evict oldest
    }
    this.cache.set(key, value);
  }
}
```

**Cache Stampede Prevention**

Cache stampede (also called thundering herd) occurs when multiple requests simultaneously miss the cache and all try to populate it, overwhelming the primary data store. Prevention strategies include request coalescing, where concurrent requests for the same data share a single fetch operation, and lock-based population, where only one request fetches while others wait. Another approach is probabilistic early expiration, where cache entries are refreshed randomly before expiration to spread out the load. These techniques are crucial for high-traffic applications with expensive data retrieval operations.

- **Problem**: Multiple requests miss cache simultaneously, overwhelming primary store
- **Solutions**: Request coalescing, lock-based population, early expiration

### Caching Patterns

**Cache-Aside (Lazy Loading)**

Cache-aside is the most common caching pattern where the application code manages cache explicitly. On read, first check cache—if hit, return data; if miss, load from database, populate cache, and return data. On write, update database and invalidate cache (or update cache). This pattern is simple to implement and works well for read-heavy workloads. However, it can lead to stale cache if database is updated directly without cache invalidation, and cache stampede can occur if multiple requests miss simultaneously.

```javascript
// Cache-aside pattern
async function getUser(userId) {
  // 1. Check cache
  const cached = await redis.get(`user:${userId}`);
  if (cached) return JSON.parse(cached); // Cache hit

  // 2. Cache miss: load from database
  const user = await db.query('SELECT * FROM users WHERE id = ?', [userId]);

  // 3. Populate cache
  await redis.set(`user:${userId}`, JSON.stringify(user), 'EX', 3600);

  return user;
}

async function updateUser(userId, data) {
  // Update database
  await db.query('UPDATE users SET ? WHERE id = ?', [data, userId]);

  // Invalidate cache
  await redis.del(`user:${userId}`);
}
```

**Write-Through Cache**

Write-through cache updates both cache and database synchronously on write operations. The cache is always consistent with the database because writes don't complete until both are updated. This pattern eliminates stale cache issues but increases write latency because both cache and database must be updated. It's suitable for applications that can tolerate slightly slower writes in exchange for guaranteed cache consistency and simpler read logic (always read from cache).

```javascript
// Write-through cache
async function updateUser(userId, data) {
  // Update cache synchronously
  await redis.set(`user:${userId}`, JSON.stringify(data), 'EX', 3600);

  // Update database (must succeed for write to complete)
  await db.query('UPDATE users SET ? WHERE id = ?', [data, userId]);
}

async function getUser(userId) {
  // Always read from cache (guaranteed to be fresh)
  const cached = await redis.get(`user:${userId}`);
  if (cached) return JSON.parse(cached);

  // Cache miss shouldn't happen with write-through
  return await db.query('SELECT * FROM users WHERE id = ?', [userId]);
}
```

**Write-Back (Write-Behind) Cache**

Write-back cache updates only cache on write, with asynchronous background writes to database. This provides the fastest write performance because database writes are deferred and batched. However, it introduces risk of data loss if cache fails before database is updated. Write-back is suitable for write-heavy workloads where write performance is critical and some data loss is acceptable, like analytics or logging systems.

- **Advantage**: Fastest write performance
- **Risk**: Data loss if cache fails before database write
- **Use case**: Write-heavy workloads, analytics, logging

**Refresh-Ahead Cache**

Refresh-ahead cache proactively updates entries before they expire, preventing cache misses for frequently accessed data. This is implemented by scheduling background refreshes slightly before expiration time. While it increases cache load, it ensures data is always available and reduces load on primary data store. Refresh-ahead is ideal for predictable access patterns and data with expensive retrieval operations.

```javascript
// Refresh-ahead cache
async function getUser(userId) {
  const cached = await redis.get(`user:${userId}`);
  if (cached) {
    const data = JSON.parse(cached);
    const ttl = await redis.ttl(`user:${userId}`);

    // Refresh before expiration (e.g., at 10% TTL remaining)
    if (ttl < 360) { // Assuming 3600s TTL
      // Background refresh without blocking request
      refreshUserInBackground(userId);
    }

    return data;
  }

  return await loadAndCacheUser(userId);
}

async function refreshUserInBackground(userId) {
  const user = await db.query('SELECT * FROM users WHERE id = ?', [userId]);
  await redis.set(`user:${userId}`, JSON.stringify(user), 'EX', 3600);
}
```

### Cache Implementation

**In-Memory Caching (Node.js)**

In-memory caching stores data in the application process memory, providing the fastest access but limited by process memory and not shared across multiple instances. Simple implementations use JavaScript objects or Map, while more sophisticated ones use libraries like `node-cache` with TTL (time-to-live) support. In-memory caching is ideal for small datasets, frequently accessed data, and single-instance deployments. However, it's not suitable for distributed systems or large datasets that exceed memory limits.

```javascript
const NodeCache = require('node-cache');
const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour TTL

// Set with expiration
cache.set('user:1', { name: 'John', email: 'john@example.com' });

// Get with automatic expiration
const user = cache.get('user:1');

// Delete
cache.del('user:1');

// Clear all
cache.flushAll();
```

**Redis Caching**

Redis is an in-memory data structure store used as a distributed cache, message broker, and session store. It supports various data structures (strings, hashes, lists, sets, sorted sets) with O(1) access time. Redis provides persistence options (RDB snapshots, AOF logs), clustering for horizontal scaling, and replication for high availability. It's the industry standard for distributed caching, session management, rate limiting, and real-time analytics due to its performance, feature set, and reliability.

- **Data structures**: Strings, hashes, lists, sets, sorted sets
- **Persistence**: RDB snapshots, AOF logs
- **Scalability**: Clustering, replication
- **Use cases**: Distributed cache, sessions, rate limiting, analytics

```javascript
const redis = require('redis');
const client = redis.createClient();

// String operations
await client.set('key', 'value', 'EX', 3600); // With TTL
const value = await client.get('key');

// Hash operations (for objects)
await client.hSet('user:1', 'name', 'John');
await client.hSet('user:1', 'email', 'john@example.com');
const user = await client.hGetAll('user:1');

// List operations
await client.lPush('queue', 'task1');
await client.lPush('queue', 'task2');
const task = await client.rPop('queue');

// Set operations (for unique values)
await client.sAdd('tags', 'javascript');
await client.sAdd('tags', 'nodejs');
const tags = await client.sMembers('tags');
```

**CDN Caching (CloudFront)**

Content Delivery Networks (CDNs) cache static assets (images, CSS, JS, videos) at edge locations close to users worldwide. CloudFront is AWS's CDN that integrates with S3, EC2, and other origins, providing low-latency content delivery. CDN caching reduces origin server load, improves user experience through geographic proximity, and handles DDoS attacks through distributed infrastructure. Configure cache control headers (Cache-Control, Expires) to control caching behavior, and use invalidation (cache key versioning, explicit invalidation) to update content.

- **Edge locations**: Distributed worldwide, close to users
- **Benefits**: Low latency, reduced origin load, DDoS protection
- **Configuration**: Cache-Control headers, invalidation strategies

```javascript
// Cache-Control headers for CDN caching
app.use(express.static('public', {
  maxAge: '1y', // 1 year cache
  etag: true,
  lastModified: true
}));

// Dynamic API: No caching
app.get('/api/data', (req, res) => {
  res.set('Cache-Control', 'no-cache, no-store, must-revalidate');
  res.json(data);
});

// Static assets: Long caching with versioning
app.get('/static/app.v123.js', (req, res) => {
  res.set('Cache-Control', 'public, max-age=31536000, immutable');
  res.sendFile('/app.js');
});
```

### Caching Use Cases

**Database Query Results**

Caching database query results reduces load on the database and improves response times for frequently accessed data. Cache keys typically include query parameters or identifiers, and cache entries are invalidated when underlying data changes. This is especially effective for expensive queries (joins, aggregations) and read-heavy workloads. However, stale cache can serve outdated data, so invalidation strategies (time-based TTL, event-based invalidation) must be carefully designed.

```javascript
// Cache expensive database queries
async function getPopularPosts(limit = 10) {
  const cacheKey = `posts:popular:${limit}`;

  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);

  // Expensive query with joins and aggregations
  const posts = await db.query(`
    SELECT p.*, COUNT(c.id) as comment_count
    FROM posts p
    LEFT JOIN comments c ON p.id = c.post_id
    GROUP BY p.id
    ORDER BY comment_count DESC
    LIMIT ?
  `, [limit]);

  await redis.set(cacheKey, JSON.stringify(posts), 'EX', 300); // 5 min TTL

  return posts;
}
```

**API Responses**

Caching API responses reduces processing overhead and improves response times for repeated requests. GET requests with identical parameters are ideal candidates, especially for data that doesn't change frequently. Cache keys include URL, query parameters, and authentication headers if responses vary by user. Implement cache headers (ETag, Last-Modified) for client-side caching, and use reverse proxy caching (Nginx, Varnish) for server-side caching. Be careful with caching authenticated or personalized data to prevent data leakage.

```javascript
// Cache API responses with ETag
app.get('/api/users/:id', async (req, res) => {
  const user = await getUser(req.params.id);

  // Generate ETag based on user data
  const etag = crypto.createHash('md5')
    .update(JSON.stringify(user))
    .digest('hex');

  // Check If-None-Match header
  if (req.headers['if-none-match'] === etag) {
    return res.status(304).end(); // Not Modified
  }

  res.set('ETag', etag);
  res.set('Cache-Control', 'public, max-age=60');
  res.json(user);
});
```

**Computed Values**

Caching computed values avoids redundant calculations for expensive operations like data transformations, aggregations, or complex algorithms. Examples include formatted data, analytics metrics, and processed images. Cache keys include input parameters and version of computation logic. This pattern is particularly effective when computation is expensive relative to cache access, and inputs have high repetition. Invalidate cache when computation logic changes or input data updates.

```javascript
// Cache expensive computations
async function generateReport(startDate, endDate) {
  const cacheKey = `report:${startDate}:${endDate}`;

  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);

  // Expensive computation: data aggregation, transformations
  const report = {
    totalRevenue: await calculateRevenue(startDate, endDate),
    userGrowth: await calculateUserGrowth(startDate, endDate),
    topProducts: await getTopProducts(startDate, endDate)
  };

  await redis.set(cacheKey, JSON.stringify(report), 'EX', 3600);

  return report;
}
```

**Session Data**

Caching session data in fast stores like Redis enables stateless web servers that can scale horizontally. Session ID is stored in client cookie, while session data is stored server-side in Redis with TTL matching session timeout. This approach allows any server to handle any request by looking up session data from shared cache. Redis clustering provides high availability and scalability for session storage. Alternative approaches include sticky sessions (less scalable) and JWT stateless sessions (no server-side storage, but limited data size).

```javascript
// Session management with Redis
const session = require('express-session');
const RedisStore = require('connect-redis')(session);

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: 'session-secret',
  cookie: { maxAge: 3600000 } // 1 hour
}));

// Access session data
app.get('/api/profile', (req, res) => {
  const userId = req.session.userId;
  // Session data retrieved from Redis automatically
  res.json({ userId });
});
```

**Static Assets**

Static assets (images, CSS, JavaScript, fonts) are ideal for CDN caching because they don't change frequently and benefit from geographic distribution. Use long cache durations (months to years) with cache busting via filename versioning or query parameters. Configure origin servers to send proper cache headers (Cache-Control, ETag, Last-Modified). CDN caching dramatically reduces origin load and improves page load times, especially for users far from origin servers.

```javascript
// Static asset versioning for cache busting
// Build process generates versioned filenames
// app.v123.js (instead of app.js)

app.use(express.static('public', {
  maxAge: '1y', // Cache for 1 year
  immutable: true, // Never changes
  etag: true,
  lastModified: true
}));

// HTML: Short cache, always revalidate
app.get('*', (req, res) => {
  res.set('Cache-Control', 'public, max-age=0, must-revalidate');
  res.sendFile('index.html');
});
```

---

## Database Design

### Database Selection

**SQL vs NoSQL Decision Factors**

SQL databases (PostgreSQL, MySQL) provide structured schemas, ACID transactions, complex queries, and relational integrity. They're ideal for applications with consistent data models, complex relationships, and strong consistency requirements (financial systems, inventory management). NoSQL databases (MongoDB, DynamoDB, Cassandra) offer flexible schemas, horizontal scalability, and high performance for specific access patterns. They're suitable for unstructured data, rapid iteration, and massive scale with simple access patterns (social media, IoT, real-time analytics).

- **SQL**: Structured schema, ACID, complex queries, relationships
- **NoSQL**: Flexible schema, horizontal scaling, specific access patterns

**ACID Properties**

ACID (Atomicity, Consistency, Isolation, Durability) ensures database transactions are reliable and predictable. Atomicity means transactions are all-or-nothing—either all operations succeed or none do. Consistency ensures database transitions from one valid state to another, maintaining all constraints. Isolation ensures concurrent transactions don't interfere with each other. Durability ensures committed transactions survive system failures. ACID is critical for applications requiring data integrity like financial systems and inventory management.

- **Atomicity**: All-or-nothing transactions
- **Consistency**: Valid state transitions, constraints maintained
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed transactions survive failures

**CAP Theorem**

CAP theorem states that a distributed data store can only provide two of three guarantees: Consistency, Availability, and Partition tolerance. Consistency ensures all nodes see the same data simultaneously. Availability ensures every request receives a response (success or failure). Partition tolerance ensures system continues operating despite network failures between nodes. In practice, you must choose between CP (consistent but unavailable during partitions) and AP (available but possibly inconsistent during partitions). Understanding CAP trade-offs helps choose the right database for your requirements.

- **Consistency (C)**: All nodes see same data simultaneously
- **Availability (A)**: Every request receives a response
- **Partition tolerance (P)**: System operates despite network partitions
- **Trade-off**: Choose 2 of 3 (CP or AP typically)

**BASE Properties**

BASE (Basically Available, Soft state, Eventual consistency) is an alternative to ACID for distributed systems prioritizing availability over immediate consistency. Basically available means the system guarantees availability, though responses might not be consistent. Soft state allows state to change over time even without input, due to eventual consistency. Eventual consistency guarantees that if no new updates occur, eventually all accesses return the last updated value. BASE is suitable for systems where availability is more important than immediate consistency, like social media feeds and caching systems.

- **Basically Available**: System is always available
- **Soft state**: State may change over time
- **Eventual consistency**: Updates propagate eventually

### SQL Databases

**Indexing Strategies**

Indexes dramatically improve query performance by creating data structures that enable fast lookups without scanning entire tables. Primary indexes automatically created on primary key columns. Secondary indexes can be created on frequently queried columns, but each index increases write overhead and storage. Composite indexes on multiple columns optimize queries with WHERE clauses on multiple columns. Understanding query patterns and creating appropriate indexes is crucial for database performance—missing indexes cause slow queries, while excessive indexes slow down writes.

- **Primary index**: Automatic on primary key
- **Secondary index**: On frequently queried columns
- **Composite index**: On multiple columns for multi-column queries
- **Trade-off**: Faster reads vs slower writes, more storage

```sql
-- Create indexes for common queries
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);
CREATE INDEX idx_posts_author_status ON posts(author_id, status);

-- Composite index for multi-column queries
-- Optimizes: WHERE author_id = ? AND status = ? ORDER BY created_at DESC
CREATE INDEX idx_posts_author_status_created ON posts(author_id, status, created_at DESC);

-- Query using composite index
SELECT * FROM posts
WHERE author_id = 123 AND status = 'published'
ORDER BY created_at DESC
LIMIT 20;
```

**Query Optimization**

Query optimization involves analyzing and improving SQL queries for better performance. Use EXPLAIN to understand query execution plans and identify bottlenecks (full table scans, inefficient joins). Select only needed columns instead of using SELECT *. Use appropriate JOIN types (INNER, LEFT, RIGHT) based on requirements. Avoid subqueries in WHERE clause—use JOINs instead. Use LIMIT to restrict result sets. Regularly analyze slow query logs and optimize problematic queries. Proper indexing combined with optimized queries enables databases to handle high throughput efficiently.

```sql
-- BAD: Selects all columns, full table scan
SELECT * FROM users WHERE email = 'user@example.com';

-- GOOD: Selects only needed columns, uses index
SELECT id, name, email FROM users WHERE email = 'user@example.com';

-- BAD: Subquery in WHERE clause
SELECT * FROM posts
WHERE author_id IN (SELECT id FROM users WHERE active = true);

-- GOOD: JOIN instead of subquery
SELECT p.* FROM posts p
INNER JOIN users u ON p.author_id = u.id
WHERE u.active = true;

-- Use EXPLAIN to analyze query
EXPLAIN SELECT * FROM posts WHERE author_id = 123;
```

**Normalization vs Denormalization**

Normalization organizes data to minimize redundancy and ensure data integrity by splitting data into related tables and using foreign key relationships. This reduces storage space and avoids anomalies (update, insert, delete anomalies). However, highly normalized schemas require complex JOINs for queries, impacting read performance. Denormalization intentionally introduces redundancy to improve read performance by combining related data into fewer tables. The choice depends on read/write patterns—read-heavy systems benefit from denormalization, while write-heavy systems benefit from normalization.

- **Normalization**: Reduces redundancy, ensures integrity, slower reads
- **Denormalization**: Increases redundancy, faster reads, slower writes
- **Trade-off**: Choose based on read/write patterns

```sql
-- Normalized schema (3NF)
CREATE TABLE users (id, name, email);
CREATE TABLE posts (id, user_id, title, content);
CREATE TABLE comments (id, post_id, user_id, content);

-- Query requires JOINs (slower reads)
SELECT p.*, u.name as author_name
FROM posts p
INNER JOIN users u ON p.user_id = u.id;

-- Denormalized schema for read performance
CREATE TABLE posts (
  id, user_id, user_name, title, content
);

-- Query is simple (faster reads)
SELECT * FROM posts;
-- But updates require changing user_name in all posts (slower writes)
```

**Master-Slave Replication**

Master-slave replication involves a primary database (master) that handles all write operations, and one or more replicas (slaves) that replicate data from master and serve read operations. This architecture scales read capacity by adding more replicas, while writes still go through single master. Replication introduces replication lag—slaves might be slightly behind master, so reads might be stale. This pattern is ideal for read-heavy workloads where eventual consistency is acceptable, like content delivery and analytics.

- **Master**: Handles all writes, replicates to slaves
- **Slaves**: Serve reads, replicate from master
- **Benefits**: Scales read capacity, improves availability
- **Challenges**: Replication lag, eventual consistency

```
Application
  ↓
[Load Balancer]
  ↓
┌─────────┬─────────┬─────────┐
│ Slave 1 │ Slave 2 │ Slave 3 │ (Reads)
└─────────┴─────────┴─────────┘
  ↑
[Master] (Writes)
```

**Read Replicas**

Read replicas are database copies that serve read queries, distributing load away from the primary instance. Writes still go to the primary, which then asynchronously replicates changes to replicas. This architecture enables horizontal scaling of read capacity without affecting write performance. Read replicas improve availability by providing failover targets if the primary fails. However, replication lag means replicas might serve slightly stale data, which must be acceptable for the application. Configure connection routing to direct reads to replicas and writes to the primary.

- **Purpose**: Distribute read load, improve availability
- **Routing**: Reads → replicas, writes → primary
- **Challenge**: Replication lag (stale reads)

```javascript
// Direct reads to replicas, writes to primary
const readPool = createPool({ hosts: replicaHosts });
const writePool = createPool({ host: primaryHost });

async function getUser(userId) {
  return await readPool.query('SELECT * FROM users WHERE id = ?', [userId]);
}

async function updateUser(userId, data) {
  return await writePool.query('UPDATE users SET ? WHERE id = ?', [data, userId]);
}
```

### NoSQL Databases

**Document Stores (MongoDB)**

Document stores like MongoDB store data in flexible JSON-like documents within collections. Each document can have different schema, enabling rapid iteration and accommodating varying data structures. Documents support nested structures and arrays, naturally modeling complex data. Indexes can be created on any field for fast queries. MongoDB provides rich query language, aggregation framework, and horizontal scaling via sharding. It's ideal for content management systems, catalogs, and applications with evolving schemas.

- **Flexible schema**: Documents can have different structures
- **Nested data**: Supports objects and arrays
- **Querying**: Rich query language, aggregations
- **Scaling**: Horizontal sharding

```javascript
// MongoDB document structure
{
  _id: ObjectId("..."),
  title: "Blog Post",
  content: "Post content...",
  author: {
    id: 123,
    name: "John Doe"
  },
  tags: ["javascript", "nodejs", "mongodb"],
  comments: [
    { user: "Jane", text: "Great post!", createdAt: Date }
  ],
  metadata: {
    views: 100,
    likes: 25
  }
}

// Query with index
db.posts.createIndex({ "author.id": 1, "createdAt": -1 });
db.posts.find({ "author.id": 123 }).sort({ createdAt: -1 }).limit(20);
```

**Key-Value Stores (DynamoDB, Redis)**

Key-value stores provide simple O(1) access to data via unique keys, offering extremely high performance for simple access patterns. DynamoDB is a managed NoSQL service with automatic scaling, low latency, and pay-per-request pricing. It supports primary keys (partition key + optional sort key) and secondary indexes for alternative query patterns. Redis provides in-memory storage with various data structures (strings, hashes, lists, sets). Key-value stores are ideal for sessions, caching, real-time leaderboards, and simple lookup tables.

- **Simple access**: O(1) lookup by key
- **DynamoDB**: Managed, auto-scaling, low latency
- **Redis**: In-memory, multiple data structures
- **Use cases**: Sessions, caching, leaderboards, lookups

```javascript
// DynamoDB operations
const AWS = require('aws-sdk');
const dynamodb = new AWS.DynamoDB.DocumentClient();

// Put item
await dynamodb.put({
  TableName: 'Users',
  Item: { userId: '123', name: 'John', email: 'john@example.com' }
}).promise();

// Get item
const user = await dynamodb.get({
  TableName: 'Users',
  Key: { userId: '123' }
}).promise();

// Query with secondary index
const posts = await dynamodb.query({
  TableName: 'Posts',
  IndexName: 'AuthorIndex',
  KeyConditionExpression: 'authorId = :authorId',
  ExpressionAttributeValues: { ':authorId': '123' }
}).promise();
```

**Wide-Column Stores (Cassandra)**

Wide-column stores like Cassandra organize data in columns rather than rows, with flexible schema where different rows can have different columns. Data is partitioned across nodes based on partition key and replicated for fault tolerance. Cassandra is designed for massive scale, high write throughput, and multi-region deployment. It offers tunable consistency—choose between strong and eventual consistency per query. Cassandra is ideal for time-series data, logging, and applications requiring high write throughput with global distribution.

- **Column-oriented**: Flexible schema, different columns per row
- **Distributed**: Partitioned and replicated across nodes
- **Scalability**: Massive scale, high write throughput
- **Consistency**: Tunable (strong or eventual)

```sql
-- Cassandra table
CREATE TABLE posts (
  user_id UUID,
  post_id UUID,
  title TEXT,
  content TEXT,
  created_at TIMESTAMP,
  PRIMARY KEY (user_id, post_id)
) WITH CLUSTERING ORDER BY (post_id DESC);

-- Query by partition key
SELECT * FROM posts WHERE user_id = ? LIMIT 50;
```

**Graph Databases (Neo4j)**

Graph databases store data as nodes (entities) and edges (relationships), optimized for traversing complex relationships. They excel at queries involving multiple hops through relationships (friends of friends, recommendation engines, fraud detection). Neo4j provides Cypher query language for expressing graph patterns declaratively. Graph databases avoid expensive JOINs required in relational databases for relationship queries. They're ideal for social networks, recommendation systems, and any application where relationships are first-class citizens.

- **Nodes and edges**: Entities and relationships
- **Relationship queries**: Optimized for traversals
- **Cypher language**: Declarative graph pattern matching
- **Use cases**: Social networks, recommendations, fraud detection

```cypher
-- Neo4j Cypher query
-- Find friends of friends (2-hop relationship)
MATCH (user:User {id: '123'})-[:FRIEND]->(friend)-[:FRIEND]->(fof)
RETURN fof.name

-- Recommendation engine
MATCH (user:User {id: '123'})-[:BOUGHT]->(product)-[:BOUGHT]->(other)
RETURN other.name, count(*) as frequency
ORDER BY frequency DESC
LIMIT 10
```

### Data Partitioning

**Horizontal Partitioning (Sharding)**

Horizontal partitioning splits data across multiple databases based on a partition key (shard key), distributing load and enabling horizontal scaling. Each shard contains a subset of data and can be hosted on separate servers. Common shard keys include user ID, geographic region, or hash of data. Sharding enables virtually unlimited scale but introduces complexity in cross-shard queries, rebalancing, and maintaining consistent hashing. It's essential for applications with massive datasets exceeding single database capacity.

- **Partition key**: User ID, region, hash
- **Distribution**: Data split across multiple databases
- **Benefits**: Horizontal scaling, reduced load per shard
- **Challenges**: Cross-shard queries, rebalancing, consistent hashing

```javascript
// Consistent hashing for sharding
function getShard(key, totalShards) {
  const hash = crypto.createHash('md5').update(key).digest('hex');
  const hashNum = parseInt(hash.substring(0, 8), 16);
  return hashNum % totalShards;
}

// Route queries to appropriate shard
const shardId = getShard(userId, 10);
const shard = shards[shardId];
const user = await shard.query('SELECT * FROM users WHERE id = ?', [userId]);
```

**Vertical Partitioning**

Vertical partitioning splits a database into smaller databases based on functionality or data access patterns. Each vertical partition contains a subset of tables related to a specific domain (users, orders, payments). This allows independent scaling of different domains—user service can scale separately from payment service. Vertical partitioning also enables microservices architecture where each service owns its database. However, cross-domain queries require joining data across multiple databases, which can be complex.

- **Split by domain**: Users, orders, payments
- **Independent scaling**: Each domain scales separately
- **Microservices**: Each service owns its database
- **Challenge**: Cross-domain queries require distributed joins

```
┌─────────────┬─────────────┬─────────────┐
│ Users DB    │ Orders DB   │ Payments DB  │
│ - users     │ - orders     │ - payments   │
│ - profiles  │ - items      │ - invoices   │
└─────────────┴─────────────┴─────────────┘
```

**Consistent Hashing**

Consistent hashing is an algorithm that distributes data across servers while minimizing data movement when servers are added or removed. Each server is assigned multiple points on a hash ring, and data is assigned to the nearest server clockwise. When a server is added, only data between new server and next server moves. When a server is removed, its data moves to the next server. This minimizes data rebalancing compared to simple modulo hashing, where adding/removing a server moves most data.

- **Hash ring**: Servers placed on circular hash space
- **Minimal movement**: Adding/removing servers affects minimal data
- **Virtual nodes**: Each server has multiple points for better distribution
- **Use case**: Distributed caches, sharding, load balancing

```javascript
// Consistent hashing implementation
class ConsistentHash {
  constructor(replicas = 100) {
    this.replicas = replicas;
    this.ring = new Map(); // hash → server
    this.sortedKeys = [];
  }

  addServer(server) {
    for (let i = 0; i < this.replicas; i++) {
      const key = `${server}:${i}`;
      const hash = this.hash(key);
      this.ring.set(hash, server);
      this.sortedKeys.push(hash);
    }
    this.sortedKeys.sort((a, b) => a - b);
  }

  getServer(key) {
    const hash = this.hash(key);
    // Find first server with hash >= key hash
    const index = this.sortedKeys.findIndex(k => k >= hash);
    const serverHash = index === -1 ? this.sortedKeys[0] : this.sortedKeys[index];
    return this.ring.get(serverHash);
  }

  hash(str) {
    const hash = crypto.createHash('md5').update(str).digest('hex');
    return parseInt(hash.substring(0, 8), 16);
  }
}
```

**Hotspot Mitigation**

Hotspots occur when certain data receives disproportionate access, overwhelming specific shards or cache nodes. Mitigation strategies include adding random suffixes to partition keys to distribute load (e.g., `user:123:1`, `user:123:2`, `user:123:3`), using more sophisticated partition keys, and implementing caching layers. For write hotspots, consider batching writes or using write-behind patterns. Identifying hotspots requires monitoring access patterns and load distribution across shards.

- **Problem**: Disproportionate access to certain data
- **Solutions**: Random suffixes, better partition keys, caching
- **Monitoring**: Track access patterns and load distribution

```javascript
// Hotspot mitigation with random suffix
function getCacheKey(userId) {
  const suffix = Math.floor(Math.random() * 10); // 0-9
  return `user:${userId}:${suffix}`;
}

// Write to multiple suffixes (fan-out)
async function updateUser(userId, data) {
  const promises = [];
  for (let i = 0; i < 10; i++) {
    promises.push(redis.set(`user:${userId}:${i}`, JSON.stringify(data), 'EX', 3600));
  }
  await Promise.all(promises);
}

// Read from random suffix (fan-in)
async function getUser(userId) {
  const suffix = Math.floor(Math.random() * 10);
  return JSON.parse(await redis.get(`user:${userId}:${suffix}`));
}
```

---

## Microservices Architecture

### Monolith vs Microservices

**Monolith Pros and Cons**

Monolithic architecture deploys the entire application as a single unit, with all functionality (UI, business logic, database) in one codebase. Monoliths are simpler to develop, test, and deploy initially—no network communication, shared memory, straightforward debugging. However, as the application grows, monoliths become difficult to maintain, scale, and understand. Changes require redeploying entire application, and scaling requires scaling the entire monolith even if only one component needs more resources.

- **Pros**: Simple initial development, no network overhead, easy debugging, shared memory
- **Cons**: Difficult to maintain at scale, tight coupling, redeploy entire app, scale everything together
- **Use case**: Small teams, early-stage products, simple domains

**Microservices Pros and Cons**

Microservices architecture decomposes the application into small, independently deployable services, each focused on a specific business capability. Each service has its own database, can be developed in different languages, and scales independently. This enables faster development cycles (small teams work independently), fault isolation (one service failure doesn't crash entire system), and technology diversity. However, microservices introduce complexity: distributed transactions, inter-service communication, deployment orchestration, and operational overhead.

- **Pros**: Independent deployment, fault isolation, technology diversity, team autonomy
- **Cons**: Distributed complexity, inter-service communication, operational overhead, data consistency
- **Use case**: Large teams, complex domains, independent scaling needs

**When to Use Each Approach**

Start with monolith for early-stage products with small teams—simplicity enables faster development and iteration. Transition to microservices when the application grows large enough that teams are blocked by each other, when specific components need different scaling strategies, or when the domain naturally separates into bounded contexts. Avoid premature microservices—they add complexity that might not be justified. A common pattern is to extract services from monolith incrementally as needed (strangler fig pattern), rather than rewriting everything upfront.

- **Start with**: Monolith for simplicity and speed
- **Transition to**: Microservices when justified by scale and team size
- **Avoid**: Premature microservices (unnecessary complexity)

### Microservices Design

**Service Boundaries**

Service boundaries define the scope and responsibilities of each microservice, ideally aligned with business capabilities and bounded contexts. Services should be loosely coupled (minimal dependencies) and highly cohesive (related functionality together). Identify boundaries by analyzing domain model, data ownership, and team structure—each service should own its data and expose well-defined APIs. Poor boundaries lead to distributed monoliths where services are tightly coupled and chatty, negating microservices benefits.

- **Alignment**: Business capabilities, bounded contexts
- **Loose coupling**: Minimal dependencies between services
- **High cohesion**: Related functionality together
- **Data ownership**: Each service owns its database

**API Design (REST, GraphQL, gRPC)**

REST APIs use HTTP verbs (GET, POST, PUT, DELETE) and resource-based URLs, providing simple, stateless communication. GraphQL enables clients to request exactly the data they need in a single query, reducing over-fetching and under-fetching. gRPC uses Protocol Buffers for efficient binary serialization and HTTP/2 for multiplexing, offering high performance for internal service communication. Choose REST for public APIs (simple, widely supported), GraphQL for flexible client requirements, and gRPC for high-performance internal communication.

- **REST**: HTTP verbs, resource URLs, simple, stateless
- **GraphQL**: Query exactly what you need, flexible, single endpoint
- **gRPC**: Binary serialization, HTTP/2, high performance

**Service Discovery**

Service discovery enables microservices to find and communicate with each other dynamically without hardcoding addresses. In containerized environments, service IPs change frequently, making static configuration impractical. Service registries (Consul, Eureka, etcd) maintain a directory of available services, and services register themselves on startup and deregister on shutdown. DNS-based service discovery (AWS Route 53, Kubernetes DNS) provides simpler alternatives. Client-side load balancing uses service discovery to distribute requests across healthy instances.

```javascript
// Service discovery with Consul
const consul = require('consul')();

// Register service
consul.agent.service.register({
  name: 'user-service',
  address: '10.0.1.5',
  port: 3000,
  check: {
    http: 'http://10.0.1.5:3000/health',
    interval: '10s'
  }
});

// Discover service
const services = await consul.agent.service.list();
const instances = services['user-service'];

// Client-side load balancing
const instance = instances[Math.floor(Math.random() * instances.length)];
const response = await fetch(`http://${instance.Address}:${instance.Port}/users/123`);
```

**Configuration Management**

Configuration management in microservices involves managing environment-specific settings across multiple services. Use centralized configuration servers (Spring Cloud Config, Consul KV, AWS Parameter Store) instead of embedding config in code or individual files. Configuration should be externalized and loaded at runtime, enabling changes without redeployment. Sensitive configuration (API keys, database passwords) must be stored securely (AWS Secrets Manager, HashiCorp Vault) and injected via environment variables or secret management systems.

- **Centralized**: Configuration servers (Consul, AWS Parameter Store)
- **Externalized**: Load at runtime, no redeployment for changes
- **Secure**: Secrets in vaults (AWS Secrets Manager, HashiCorp Vault)

### Inter-Service Communication

**Synchronous Communication (HTTP/REST)**

Synchronous communication involves the calling service waiting for the response before proceeding, typically using HTTP/REST or gRPC. This is simple to implement and understand—request/response pattern similar to function calls. However, synchronous communication creates tight temporal coupling—if the called service is slow or unavailable, the caller is blocked. It's suitable for operations requiring immediate response and when failure should propagate immediately (user-facing requests).

- **Pattern**: Request/response, caller waits for response
- **Pros**: Simple, immediate feedback, error propagation
- **Cons**: Tight coupling, cascading failures, blocking
- **Use case**: User-facing requests, immediate response required

**Asynchronous Communication (Message Queues)**

Asynchronous communication involves the calling service sending a message and continuing without waiting for a response. Message queues (RabbitMQ, SQS, Kafka) buffer messages and deliver them to consumers when ready. This decouples services temporally—the producer doesn't need the consumer to be available. Asynchronous communication improves resilience (queues absorb spikes), enables eventual consistency, and supports event-driven architectures. It's suitable for background processing, notifications, and operations that don't require immediate response.

```javascript
// Asynchronous communication with message queue
const AWS = require('aws-sdk');
const sqs = new AWS.SQS();

// Producer: Send message to queue
async function sendUserCreatedEvent(userId) {
  await sqs.sendMessage({
    QueueUrl: 'https://sqs.us-east-1.amazonaws.com/user-events',
    MessageBody: JSON.stringify({
      eventType: 'USER_CREATED',
      userId: userId,
      timestamp: Date.now()
    })
  }).promise();
}

// Consumer: Process messages from queue
async function processUserEvents() {
  const response = await sqs.receiveMessage({
    QueueUrl: 'https://sqs.us-east-1.amazonaws.com/user-events',
    MaxNumberOfMessages: 10,
    WaitTimeSeconds: 20
  }).promise();

  for (const message of response.Messages) {
    const event = JSON.parse(message.Body);
    await handleUserEvent(event);
    await sqs.deleteMessage({
      QueueUrl: 'https://sqs.us-east-1.amazonaws.com/user-events',
      ReceiptHandle: message.ReceiptHandle
    }).promise();
  }
}
```

**Message Brokers (RabbitMQ, SQS, Kafka)**

Message brokers facilitate asynchronous communication by storing and delivering messages between services. RabbitMQ is a traditional message broker with support for various messaging patterns (pub/sub, routing, RPC). AWS SQS is a fully managed queue service with automatic scaling and pay-per-request pricing. Kafka is a distributed streaming platform optimized for high throughput and real-time data pipelines. Choose RabbitMQ for complex routing and reliability features, SQS for simple queue operations with AWS integration, and Kafka for event streaming and log aggregation.

- **RabbitMQ**: Complex routing, pub/sub, reliability features
- **AWS SQS**: Managed queues, auto-scaling, AWS integration
- **Kafka**: High throughput, event streaming, log aggregation

**Event-Driven Architecture**

Event-driven architecture uses events to communicate state changes between services, promoting loose coupling and scalability. When a service performs an operation (creates user, updates order), it publishes an event (USER_CREATED, ORDER_UPDATED). Other services subscribe to relevant events and react accordingly (send welcome email, update analytics). This pattern enables real-time updates, easy addition of new consumers, and temporal decoupling. Implement with message brokers (Kafka, RabbitMQ) or event streaming platforms (AWS EventBridge).

```javascript
// Event-driven architecture
// Service A: User Service
async function createUser(userData) {
  const user = await db.users.insert(userData);

  // Publish event
  await eventBus.publish({
    type: 'USER_CREATED',
    data: { userId: user.id, email: user.email },
    timestamp: Date.now()
  });

  return user;
}

// Service B: Email Service (subscribes to USER_CREATED)
eventBus.subscribe('USER_CREATED', async (event) => {
  await emailService.sendWelcomeEmail(event.data.email);
});

// Service C: Analytics Service (subscribes to USER_CREATED)
eventBus.subscribe('USER_CREATED', async (event) => {
  await analytics.trackUserRegistration(event.data.userId);
});
```

**Circuit Breaker Pattern**

Circuit breaker pattern prevents cascading failures by stopping calls to a failing service after a threshold of failures is reached. When the circuit is open, calls fail immediately without attempting the remote service, preventing resource exhaustion and allowing the failing service time to recover. After a timeout, the circuit moves to half-open state, allowing a test request to check if the service has recovered. If successful, circuit closes; if failed, it stays open. This pattern is essential for resilient microservices architectures.

```javascript
// Circuit breaker implementation
class CircuitBreaker {
  constructor(threshold = 5, timeout = 60000) {
    this.threshold = threshold;
    this.timeout = timeout;
    this.failureCount = 0;
    this.lastFailureTime = null;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
  }

  async execute(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.timeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }

  onFailure() {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    if (this.failureCount >= this.threshold) {
      this.state = 'OPEN';
    }
  }
}

// Usage
const circuitBreaker = new CircuitBreaker(5, 60000);

try {
  const result = await circuitBreaker.execute(() =>
    fetch('http://user-service/users/123')
  );
} catch (error) {
  console.error('Service unavailable:', error.message);
}
```

### Microservices Challenges

**Distributed Transactions**

Distributed transactions involve coordinating updates across multiple services' databases, which is challenging because traditional ACID transactions don't work across services. Two-phase commit (2PC) protocol attempts to solve this but has performance and availability issues. Saga pattern breaks transactions into a sequence of local transactions, each with compensating actions to roll back if needed. For example, creating an order saga involves: 1) Create order (local transaction), 2) Reserve inventory (local transaction), 3) Process payment (local transaction). If payment fails, compensating actions cancel order and release inventory.

- **Challenge**: ACID transactions don't work across services
- **2PC**: Two-phase commit (performance issues, availability problems)
- **Saga**: Sequence of local transactions with compensating actions

```javascript
// Saga pattern for distributed transaction
async function createOrderSaga(userId, items) {
  const orderId = generateId();

  try {
    // Step 1: Create order
    const order = await orderService.createOrder(orderId, userId, items);

    // Step 2: Reserve inventory
    await inventoryService.reserveInventory(orderId, items);

    // Step 3: Process payment
    await paymentService.processPayment(orderId, order.total);

    // Step 4: Confirm order
    await orderService.confirmOrder(orderId);

    return order;
  } catch (error) {
    // Compensating actions
    await orderService.cancelOrder(orderId);
    await inventoryService.releaseInventory(orderId);
    await paymentService.refundPayment(orderId);
    throw error;
  }
}
```

**Data Consistency**

Data consistency in microservices is challenging because each service owns its database, and updates across services must be coordinated. Eventual consistency is often accepted—data becomes consistent over time through asynchronous events. Strong consistency requires distributed transactions (expensive, complex) or synchronous communication (reduces availability). Implement consistency patterns like read-your-writes consistency (read from the same service that wrote), causal consistency (track causal relationships), and version vectors (detect conflicts). Choose consistency level based on business requirements—financial systems need strong consistency, while social media can accept eventual consistency.

- **Challenge**: Each service owns its database
- **Eventual consistency**: Data becomes consistent over time (common)
- **Strong consistency**: Requires distributed transactions (expensive)
- **Patterns**: Read-your-writes, causal consistency, version vectors

**Observability and Monitoring**

Observability in microservices requires centralized logging, metrics, and tracing across distributed services. Structured logging with correlation IDs (request IDs) enables tracing requests across service boundaries. Metrics collection (Prometheus, CloudWatch) tracks performance, error rates, and throughput. Distributed tracing (Jaeger, Zipkin, AWS X-Ray) visualizes request flow through multiple services, identifying bottlenecks and failures. Without proper observability, debugging distributed systems is nearly impossible.

- **Logging**: Structured logs, correlation IDs, centralized aggregation
- **Metrics**: Performance, error rates, throughput (Prometheus, CloudWatch)
- **Tracing**: Request flow across services (Jaeger, Zipkin, AWS X-Ray)

```javascript
// Observability: Logging with correlation ID
const { v4: uuidv4 } = require('uuid');

app.use((req, res, next) => {
  req.correlationId = req.headers['x-correlation-id'] || uuidv4();
  res.setHeader('x-correlation-id', req.correlationId);
  next();
});

app.get('/users/:id', async (req, res) => {
  logger.info('Fetching user', {
    correlationId: req.correlationId,
    userId: req.params.id
  });

  const user = await userService.getUser(req.params.id);

  logger.info('User fetched', {
    correlationId: req.correlationId,
    userId: user.id
  });

  res.json(user);
});
```

**Deployment Complexity**

Deploying microservices involves orchestrating multiple services, each with its own database, configuration, and dependencies. Containerization (Docker) provides consistent runtime environments across development, testing, and production. Orchestration platforms (Kubernetes, ECS) manage container deployment, scaling, and service discovery. CI/CD pipelines must handle multiple services, often deploying services independently. Blue-green deployments and canary releases reduce deployment risk. Infrastructure as code (Terraform, CloudFormation) enables reproducible deployments across environments.

- **Containerization**: Docker for consistent environments
- **Orchestration**: Kubernetes, ECS for deployment and scaling
- **CI/CD**: Multi-service pipelines, independent deployments
- **Strategies**: Blue-green, canary releases

---

## System Design Patterns

### Common Patterns

**URL Shortener (Bit.ly)**

URL shortener systems generate short aliases for long URLs, redirecting users when they access the short URL. The core challenge is handling high write throughput (1M writes/sec) while ensuring unique short codes. Use base62 encoding (0-9, a-z, A-Z) to convert sequential IDs to short strings, maximizing URL space with minimal characters. Database schema stores original URL, short code, creation timestamp, and expiration. Cache frequently accessed URLs to reduce database load. Consider pre-generating short codes or using distributed unique ID generators (Snowflake) for high throughput.

```javascript
// Base62 encoding for URL shortening
const chars = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';

function encode(num) {
  let encoded = '';
  while (num > 0) {
    encoded = chars[num % 62] + encoded;
    num = Math.floor(num / 62);
  }
  return encoded || '0';
}

function decode(str) {
  let num = 0;
  for (const char of str) {
    num = num * 62 + chars.indexOf(char);
  }
  return num;
}

// Generate short URL
const id = await db.insert('INSERT INTO urls (original_url) VALUES (?)', [longUrl]);
const shortCode = encode(id);
const shortUrl = `https://short.ly/${shortCode}`;

// Cache for fast lookup
await redis.set(`url:${shortCode}`, longUrl, 'EX', 86400);
```

**Chat System**

Chat systems require real-time bidirectional communication between users, typically implemented with WebSockets for persistent connections. Each user maintains a WebSocket connection to the server, and the server routes messages between connected users. Message delivery must handle offline users—store undelivered messages and deliver when they come online. Online status tracking requires presence management (connected, away, offline). Scale horizontally with connection-aware load balancers that route WebSocket connections to the same server (sticky sessions) or use connection migration.

```javascript
// WebSocket chat server
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

const users = new Map(); // userId → WebSocket

wss.on('connection', (ws) => {
  ws.on('message', async (message) => {
    const { from, to, text } = JSON.parse(message);

    // Send to recipient if online
    if (users.has(to)) {
      users.get(to).send(JSON.stringify({ from, text }));
    } else {
      // Store for offline user
      await db.insert('INSERT INTO messages (from, to, text, delivered) VALUES (?, ?, ?, false)', [from, to, text]);
    }
  });

  ws.on('close', () => {
    users.delete(userId);
  });
});
```

**News Feed**

News feed systems generate personalized content for users, balancing freshness and relevance. Fan-out on write approach generates feed entries for all followers when a user posts, making reads fast but writes expensive. Fan-out on read approach generates feed on-demand by querying followees' posts, making writes fast but reads expensive. Hybrid approaches pre-generate feeds for active users and generate on-demand for inactive users. Pagination uses cursor-based pagination (efficient for infinite scroll) rather than offset-based (slow for large offsets). Cache feed entries to reduce database load.

```javascript
// Fan-out on write: Generate feed for all followers
async function createPost(userId, content) {
  const post = await db.insert('INSERT INTO posts (user_id, content) VALUES (?, ?)', [userId, content]);

  // Get all followers
  const followers = await db.query('SELECT follower_id FROM follows WHERE user_id = ?', [userId]);

  // Generate feed entries for all followers
  for (const follower of followers) {
    await db.insert('INSERT INTO feed (user_id, post_id, created_at) VALUES (?, ?, ?)', [
      follower.follower_id,
      post.id,
      Date.now()
    ]);
  }
}

// Cursor-based pagination
async function getFeed(userId, cursor = null, limit = 20) {
  let query = 'SELECT f.*, p.* FROM feed f JOIN posts p ON f.post_id = p.id WHERE f.user_id = ?';
  const params = [userId];

  if (cursor) {
    query += ' AND f.created_at < ?';
    params.push(cursor);
  }

  query += ' ORDER BY f.created_at DESC LIMIT ?';
  params.push(limit);

  const feed = await db.query(query, params);
  const nextCursor = feed.length > 0 ? feed[feed.length - 1].created_at : null;

  return { feed, nextCursor };
}
```

**File Storage System**

File storage systems handle uploading, storing, and serving user files (images, videos, documents). Upload flow involves client sending file to application server, which uploads to object storage (S3) and stores metadata in database. Download flow involves client requesting file, application retrieving metadata from database, and redirecting to object storage URL or streaming through application. CDN integration (CloudFront) caches files at edge locations for fast delivery. Implement multipart uploads for large files, resumable uploads for unreliable networks, and generate thumbnails/previews for media files.

```javascript
// File upload to S3
const AWS = require('aws-sdk');
const s3 = new AWS.S3();

async function uploadFile(userId, file) {
  const key = `users/${userId}/${Date.now()}-${file.originalname}`;

  // Upload to S3
  await s3.upload({
    Bucket: 'my-bucket',
    Key: key,
    Body: file.buffer,
    ContentType: file.mimetype
  }).promise();

  // Store metadata in database
  const fileRecord = await db.insert('INSERT INTO files (user_id, s3_key, filename, size) VALUES (?, ?, ?, ?)', [
    userId,
    key,
    file.originalname,
    file.size
  ]);

  // Generate CDN URL
  const cdnUrl = `https://cdn.example.com/${key}`;

  return { ...fileRecord, url: cdnUrl };
}
```

### Design Process

**Requirements Clarification**

System design interviews begin with clarifying requirements to ensure understanding of the problem. Ask questions about functional requirements (what the system should do), non-functional requirements (performance, scalability, availability), constraints (traffic, data size), and assumptions. This step is crucial—building the wrong system is worse than not building at all. Document requirements explicitly and confirm with interviewer before proceeding to architecture design.

- **Functional requirements**: What features are needed?
- **Non-functional requirements**: Performance, scalability, availability, consistency
- **Constraints**: Traffic (QPS), data size, user base
- **Assumptions**: Document and confirm

**Capacity Estimation**

Capacity estimation involves calculating storage, bandwidth, and throughput requirements based on constraints. Estimate number of users, requests per second, data per user, and growth rate. Calculate storage requirements (user data × users × retention period). Calculate bandwidth requirements (requests × data size). These estimates guide architectural decisions—whether to use SQL vs NoSQL, caching strategy, and partitioning approach. Be prepared to explain assumptions and calculations.

```javascript
// Capacity estimation example
// Requirements: 10M users, 100 requests/user/day, 1KB per request

const totalUsers = 10_000_000;
const requestsPerUserPerDay = 100;
const dataSizePerRequest = 1024; // 1KB

// Requests per second
const requestsPerSecond = (totalUsers * requestsPerUserPerDay) / (24 * 3600);
// 10M * 100 / 86400 = ~11,574 RPS

// Storage per day
const storagePerDay = totalUsers * requestsPerUserPerDay * dataSizePerRequest;
// 10M * 100 * 1KB = 1TB per day

// Storage per year
const storagePerYear = storagePerDay * 365;
// 1TB * 365 = 365TB per year
```

**High-Level Architecture**

High-level architecture provides a bird's-eye view of the system components and their interactions. Start with a simple architecture and add complexity as needed. Identify major components: load balancers, application servers, databases, caches, message queues, and external services. Show data flow between components. Discuss trade-offs of architectural decisions (why use NoSQL vs SQL, why use message queues). Keep the diagram clear and explainable within interview time constraints.

```
┌─────────────────────────────────────────────────────────┐
│                    Users                            │
└────────────────────┬────────────────────────────────┘
                     │
              [Load Balancer]
                     │
    ┌────────────────┴────────────────┐
    │                                 │
[App Server 1]                  [App Server 2]
    │                                 │
    └────────────────┬────────────────┘
                     │
              [Cache Layer]
                     │
    ┌────────────────┴────────────────┐
    │                                 │
[Database Master]               [Database Replica]
    │
[Message Queue]
    │
[Background Worker]
```

**Data Model Design**

Data model design defines how data is structured and stored, based on access patterns and scalability requirements. For SQL databases, design normalized tables with appropriate indexes and foreign key relationships. For NoSQL databases, design documents or key-value structures optimized for query patterns. Consider read vs write patterns—denormalize for read-heavy workloads, normalize for write-heavy. Estimate data size and growth to choose appropriate partitioning strategy. Document the data model with entity-relationship diagrams or document structure examples.

```sql
-- SQL data model for URL shortener
CREATE TABLE users (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_email (email)
);

CREATE TABLE urls (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  user_id BIGINT NOT NULL,
  original_url TEXT NOT NULL,
  short_code CHAR(7) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  expires_at TIMESTAMP NULL,
  FOREIGN KEY (user_id) REFERENCES users(id),
  INDEX idx_short_code (short_code),
  INDEX idx_user_id (user_id),
  INDEX idx_created_at (created_at)
);

CREATE TABLE analytics (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  url_id BIGINT NOT NULL,
  clicked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  ip_address VARCHAR(45),
  user_agent TEXT,
  FOREIGN KEY (url_id) REFERENCES urls(id),
  INDEX idx_url_id (url_id),
  INDEX idx_clicked_at (clicked_at)
);
```

**Detailed Component Design**

Detailed component design dives deep into specific components of the system, explaining their internal workings. Choose 1-2 components to design in detail rather than trying to explain everything. For each component, discuss data structures, algorithms, APIs, and scalability considerations. Common components to design: URL shortening algorithm, feed generation algorithm, message delivery system, cache invalidation strategy. Be prepared to discuss trade-offs and alternatives.

**Bottleneck Identification**

Bottleneck identification involves analyzing the system to find components that limit performance or scalability. Common bottlenecks include database (slow queries, connection limits), cache (low hit ratio, stampede), network (bandwidth, latency), and application (CPU-bound operations, inefficient algorithms). Discuss how to identify bottlenecks (monitoring, profiling, load testing) and how to mitigate them (indexing, caching, partitioning, optimization). Interviewers want to see that you can think critically about system limitations.

- **Database**: Slow queries, connection limits, disk I/O
- **Cache**: Low hit ratio, stampede, eviction
- **Network**: Bandwidth, latency, packet loss
- **Application**: CPU-bound, inefficient algorithms, memory leaks

**Scalability Discussion**

Scalability discussion explores how the system handles growth in users, data, and traffic. Discuss horizontal vs vertical scaling strategies for each component. Explain how to add more application servers, database replicas, or cache nodes. Discuss partitioning strategies for databases (sharding, consistent hashing). Discuss caching strategies to reduce load. Discuss CDN for static assets. Be prepared to discuss trade-offs of different scalability approaches and when to use each. Interviewers want to see that you can think about long-term growth.

---

## Quick Reference

**Scalability Cube Dimensions**

The scalability cube provides a framework for scaling applications across three dimensions. X-axis cloning creates identical copies of the application behind a load balancer, the simplest form of horizontal scaling. Y-axis partitioning splits the application based on functional decomposition, enabling independent scaling of different services. Z-axis data partitioning distributes data based on attributes like user ID or region, enabling efficient data access and localized processing. Combine dimensions as needed—most large systems use all three.

- **X-axis**: Cloning (identical copies behind load balancer)
- **Y-axis**: Partitioning (functional decomposition)
- **Z-axis**: Data partitioning (sharding by user/region)

**Load Balancing Algorithms**

Choosing the right load balancing algorithm depends on request characteristics and infrastructure. Round robin provides equal distribution but doesn't account for server capacity or load. Least connections routes to least busy server, ideal for varying request processing times. IP hash ensures same client routes to same server, useful for session persistence. Weighted round robin assigns more requests to more powerful servers, enabling heterogeneous infrastructure.

- **Round robin**: Equal distribution, simple
- **Least connections**: Routes to least busy, handles varying load
- **IP hash**: Consistent routing, session persistence
- **Weighted round robin**: Heterogeneous infrastructure, capacity-based

**Caching Strategies**

Caching strategies determine when and how cache is populated and invalidated. Cache-aside (lazy loading) checks cache first, loads from database on miss, and populates cache—simple and common. Write-through updates cache and database synchronously, ensuring consistency but slower writes. Write-back updates only cache with asynchronous database writes—fastest but risk of data loss. Refresh-ahead proactively updates cache before expiration, preventing misses. Choose based on consistency and performance requirements.

- **Cache-aside**: Check cache, load on miss, populate (simple)
- **Write-through**: Update cache and database synchronously (consistent)
- **Write-back**: Update cache, async database write (fast, risky)
- **Refresh-ahead**: Proactive update before expiration (prevent misses)

**CAP Theorem Trade-offs**

CAP theorem states that distributed systems can only achieve 2 of 3 guarantees. CP systems prioritize consistency and partition tolerance, sacrificing availability during network partitions—suitable for financial systems where data accuracy is critical. AP systems prioritize availability and partition tolerance, sacrificing immediate consistency—suitable for social media where user experience is critical. Understanding these trade-offs helps choose the right database and consistency model for your requirements.

- **CP**: Consistency + Partition tolerance (unavailable during partitions)
- **AP**: Availability + Partition tolerance (eventually consistent)
- **CA**: Consistency + Availability (not possible with partitions)

**Microservices Communication Patterns**

Microservices communicate synchronously or asynchronously based on requirements. Synchronous (HTTP/REST, gRPC) is simple but creates tight coupling—use for immediate response requirements. Asynchronous (message queues, event streaming) decouples services temporally—use for background processing and resilience. Event-driven architecture promotes loose coupling through events—use for real-time updates and extensibility. Choose pattern based on coupling, latency, and resilience requirements.

- **Synchronous**: HTTP/REST, gRPC (simple, tight coupling)
- **Asynchronous**: Message queues, event streaming (decoupled, resilient)
- **Event-driven**: Pub/sub, events (loose coupling, extensible)

**System Design Interview Tips**

System design interviews test your ability to think about architecture, trade-offs, and scalability. Start by clarifying requirements and constraints. Estimate capacity to guide architectural decisions. Propose a simple high-level architecture first, then dive into details of 1-2 components. Always discuss trade-offs—why you chose one approach over another. Identify bottlenecks and how to mitigate them. Think about scalability from day one, not as an afterthought. Practice common patterns (URL shortener, chat system, news feed) to build intuition.

- **Clarify**: Requirements, constraints, assumptions
- **Estimate**: Capacity, storage, bandwidth
- **Propose**: High-level architecture, then details
- **Discuss**: Trade-offs, bottlenecks, scalability
- **Practice**: Common patterns, build intuition
