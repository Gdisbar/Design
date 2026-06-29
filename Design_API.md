# API Design Patterns & Protocols
====================================================================
# Core API Styles Comparison

┌─────────────────────────────────────────────────────────────────┐
│                    API STYLES COMPARISON                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────┬──────────────────┬─────────────────────────┐│
│  │ Feature       │ REST             │ GraphQL    │ gRPC       ││
│  ├───────────────┼──────────────────┼────────────┼────────────┤│
│  │ Protocol      │ HTTP/1.1, HTTP/2  │ HTTP       │ HTTP/2     ││
│  │ Data Format   │ JSON/XML         │ JSON       │ Protobuf   ││
│  │ Over-fetching │ Yes              │ No         │ No         ││
│  │ Caching       │ HTTP cache       │ Complex    │ No         ││
│  │ Tooling       │ Mature           │ Growing    │ Mature     ││
│  │ Learning Curve│ Low              │ Medium     │ High       ││
│  └───────────────┴──────────────────┴────────────┴────────────┘│
│                                                                 │
│  WHEN TO USE:                                                   │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  REST: Public APIs, CRUD, simple resources          │      │
│  │  GraphQL: Complex data requirements, mobile apps    │      │
│  │  gRPC: Microservices, high-performance, streaming   │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘


# Application Protocols

┌─────────────────────────────────────────────────────────────────┐
│               APPLICATION LAYER PROTOCOLS                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  HTTPS                                                         │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Secure HTTP with TLS/SSL encryption               │      │
│  │  - Most common for web APIs                         │      │
│  │  ✅ Wide support, firewall-friendly                 │      │
│  │  ❌ Overhead for small messages                     │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  WEBSOCKETS                                                    │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Real-time bidirectional communication             │      │
│  │  - Persistent connection                             │      │
│  │  ✅ Low latency, push notifications                 │      │
│  │  ❌ Connection overhead, scaling complexity         │      │
│  │  🔧 Recovery: Reconnect on drop, keep-alive        │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  MQTT                                                          │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Lightweight IoT messaging                        │      │
│  │  - Pub/sub model                                     │      │
│  │  ✅ Low bandwidth, battery friendly                 │      │
│  │  ❌ QoS levels complexity, binary protocol          │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  gRPC                                                          │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - High-performance RPC                             │      │
│  │  - Protocol Buffers serialization                   │      │
│  │  ✅ Fast, streaming support, typed contracts        │      │
│  │  ❌ Not browser-friendly, binary format             │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

# API Design Considerations

┌─────────────────────────────────────────────────────────────────┐
│              API DESIGN CHECKLIST                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  WHAT TO CONSIDER:                                              │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  1. API Type: REST/GraphQL/gRPC                     │      │
│  │  2. Communication Protocol: HTTP/WebSocket          │      │
│  │  3. Data Transport: JSON/XML/Protobuf              │      │
│  │  4. Input/Output Validation                         │      │
│  │  5. Naming Conventions                              │      │
│  │  6. Security: Authentication, Rate Limiting         │      │
│  │  7. Sorting, Filtering, Pagination                 │      │
│  │  8. Error Handling                                  │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  API DESIGN APPROACHES:                                        │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Top-Down: High-level design first                  │      │
│  │  Bottom-Up: Existing data model drives              │      │
│  │  Contract-First: Define contract before code        │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  API LIFECYCLE:                                                 │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Design → Development → Monitor → Maintenance →     │      │
│  │  Deprecation → Retirement                           │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘


# RESTful API Design : REST API Structure

┌─────────────────────────────────────────────────────────────────┐
│                   REST API PATTERNS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  RESOURCE NAMING:                                              │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Products:  /api/v1/products                        │      │
│  │  Orders:    /api/v1/orders                          │      │
│  │  Reviews:   /api/v1/products/{id}/reviews           │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  FILTERING, SORTING, PAGINATION:                                │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  GET /products?category=books&inStock=true          │      │
│  │  GET /products?sort=price_asc                       │      │
│  │  GET /products?page=2&limit=3                       │      │
│  │  GET /products?cursor=hash_of_the_page              │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  STATUS CODES:                                                 │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  2xx: Success                                      │      │
│  │    200 OK, 201 Created, 204 No Content              │      │
│  │  3xx: Redirection                                   │      │
│  │    301 Moved, 304 Not Modified                      │      │
│  │  4xx: Client Error                                  │      │
│  │    400 Bad Request, 401 Unauthorized, 404 Not Found │      │
│  │  5xx: Server Error                                  │      │
│  │    500 Internal Server Error, 503 Service Unavailable│     │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

# GraphQL API Design : GraphQL Schema & Queries
```bash
type User {
    id: ID!
    name: String!
    posts: [Post!]!
}

type Post {
    id: ID!
    title: String!
    body: String!
    author: User!
}

type Query {
    user(id: ID!): User
    posts(limit: Int): [Post!]!
}

type Mutation {
    createUser(name: String!): User!
    createPost(title: String!, body: String!): Post!
}
```

# Query Examples

```bash
# QUERY: Get user with their posts
query {
    user(id: "123") {
        name
        posts {
            title
            body
        }
    }
}

# MUTATION: Create post
mutation {
    createPost(title: "New Post", body: "Hello World!") {
        id
        title
        createdAt
    }
}

# ERROR RESPONSE
{
    "data": { "user": null },
    "errors": [{
        "statusCode": 404,
        "message": "User not found",
        "path": ["user"]
    }]
}
```
# GraphQL Best Practices

┌─────────────────────────────────────────────────────────────────┐
│               GRAPHQL BEST PRACTICES                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. N + 1 Problem Prevention                                    │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Use DataLoader for batched DB queries              │      │
│  │  Example: 100 users → 1 query for posts (not 100)   │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  2. Complexity Limits                                          │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Set max query depth                               │      │
│  │  - Limit number of fields                            │      │
│  │  - Cost-based analysis                               │      │
│  │  - Reject expensive queries                          │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  3. Caching Strategy                                           │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Use persisted queries (stored queries)           │      │
│  │  - Hash-based cache keys                            │      │
│  │  - Apollo caching with normalized store              │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  4. Security                                                    │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Validate input                                    │      │
│  │  - Rate limiting per query type                     │      │
│  │  - Authentication in resolvers                      │      │
│  │  - Disable introspection in production              │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  5. When GraphQL Fails                                         │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Problem: Complex queries overload DB               │      │
│  │  Solution: Cache, pagination, query cost analysis    │      │
│  │                                                     │      │
│  │  Problem: Malicious nested queries                  │      │
│  │  Solution: Depth limiting, timeouts                 │      │
│  │                                                     │      │
│  │  Problem: Exposed internal data                     │      │
│  │  Solution: Field-level authorization                │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

# HTTP Status Codes Reference

┌─────────────────────────────────────────────────────────────────┐
│                   HTTP STATUS CODES                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  2xx: SUCCESS                                                  │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  200 OK: Request succeeded                         │      │
│  │  201 Created: Resource created successfully         │      │
│  │  204 No Content: Success, no response body         │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  3xx: REDIRECTION                                              │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  301 Moved Permanently: Resource moved permanently  │      │
│  │  304 Not Modified: Cache hit                       │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  4xx: CLIENT ERROR                                             │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  400 Bad Request: Malformed request                 │      │
│  │  401 Unauthorized: Authentication failed            │      │
│  │  403 Forbidden: Authentication success, but not     │      │
│  │              authorized to access resource          │      │
│  │  404 Not Found: Resource doesn't exist             │      │
│  │  429 Too Many Requests: Rate limited               │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  5xx: SERVER ERROR                                             │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  500 Internal Server Error: Generic server error    │      │
│  │  502 Bad Gateway: Upstream server error             │      │
│  │  503 Service Unavailable: Server overloaded         │      │
│  │  504 Gateway Timeout: Upstream timeout              │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘
