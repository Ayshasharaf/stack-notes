# Complete Guide to APIs and Authentication

**From Beginner to Advanced Concepts**

---

## Part 1: Foundations (Beginner Level)

### 1.1 What is an API?

**API** stands for **Application Programming Interface**.

Think of it as a **menu at a restaurant**:
- You don't go to the kitchen and cook your own food
- You ask the waiter for what you want
- The kitchen prepares it and gives it back to you

Similarly:
- **Client** (your app, browser, mobile phone) = You, the customer
- **API** = The waiter taking your order
- **Server** (backend) = The kitchen preparing the food
- **Response** = The dish delivered back to you

**In real terms**: An API lets different software applications talk to each other and share data, without needing to know all the internal details.

#### Real-World Examples

| What You Do | What Happens Behind |
|---|---|
| Open Google Maps and search for a restaurant | Your phone talks to Google's API: "Give me restaurants near this location" |
| Check the weather on your phone | Your app talks to a weather API: "What's the weather in Mumbai?" |
| Login to Facebook using your Google account | Your app talks to Google's API: "Is this user who they say they are?" |
| Buy something on Amazon | Your browser talks to Amazon's API: "Add this item to my cart" |

---

### 1.2 Client-Server Architecture (The Basic Flow)

```
┌─────────────────────────────────────────────────────────┐
│  CLIENT (Browser, Mobile App, Desktop App)              │
│  - User clicks a button                                 │
│  - App makes a REQUEST to the server                    │
└──────────────────┬──────────────────────────────────────┘
                   │
                   │ "Hey server, give me user details"
                   ↓
┌─────────────────────────────────────────────────────────┐
│  SERVER (Backend, API)                                  │
│  - Receives the request                                 │
│  - Does something (query database, process data)        │
│  - Sends back a RESPONSE                                │
└──────────────────┬──────────────────────────────────────┘
                   │
                   │ "Here's your user data"
                   ↓
┌─────────────────────────────────────────────────────────┐
│  CLIENT (receives & displays data)                      │
│  - Shows the information to the user                    │
└─────────────────────────────────────────────────────────┘
```

**Key Point**: The client and server communicate over the **internet** using a shared language.

---

### 1.3 How APIs Talk: REST (The Most Popular Way)

**REST** = **Representational State Transfer**

REST is just a set of **rules** for how to ask for data using **HTTP**. Think of HTTP as the postal service and REST as the addressing system.

#### The Four Basic Operations (CRUD)

| Operation | What It Does | HTTP Method | Example |
|---|---|---|---|
| **Create** | Add new data | POST | Create a new user |
| **Read** | Get existing data | GET | Fetch user details |
| **Update** | Modify existing data | PUT/PATCH | Change user's email |
| **Delete** | Remove data | DELETE | Delete a user account |

#### How a REST Request Works

```
Request from Client:
┌──────────────────────────────────────────┐
│ GET /users/123 HTTP/1.1                  │
│ Host: api.example.com                    │
│ Authorization: Bearer token_here         │
│                                          │
│ (optional body)                          │
└──────────────────────────────────────────┘

Response from Server:
┌──────────────────────────────────────────┐
│ HTTP/1.1 200 OK                          │
│ Content-Type: application/json           │
│                                          │
│ {                                        │
│   "id": 123,                             │
│   "name": "John Doe",                    │
│   "email": "john@example.com"            │
│ }                                        │
└──────────────────────────────────────────┘
```

**Breaking it down:**
- **GET** = "I want to read data"
- **/users/123** = "Get the user with ID 123"
- **Authorization** = "Here's proof I'm allowed to access this"
- **200 OK** = "Success! Here's your data"

---

### 1.4 Data Format: JSON

**JSON** = **JavaScript Object Notation**

It's just a way to **structure and send data** that both computers can understand. It's simple text.

```json
{
  "user_id": 456,
  "username": "alice",
  "email": "alice@example.com",
  "is_active": true,
  "roles": ["admin", "moderator"],
  "profile": {
    "bio": "Software engineer",
    "location": "San Francisco"
  }
}
```

**Why JSON?** It's:
- Human-readable (easy for developers to debug)
- Lightweight (small file size, fast to send)
- Works in all programming languages

---

### 1.5 Status Codes: Server Talking Back

When the server responds, it gives you a **status code** (a number) that tells you what happened:

| Code | Meaning | Example |
|---|---|---|
| **200** | Success! Here's your data | Your request worked perfectly |
| **201** | Created! New data was added | A new user was successfully created |
| **400** | Bad Request - You asked wrong | You sent invalid data |
| **401** | Unauthorized - Who are you? | You didn't send authentication proof |
| **403** | Forbidden - You can't have this | You're logged in but not allowed to access this |
| **404** | Not Found | The resource doesn't exist (e.g., user ID 999 doesn't exist) |
| **500** | Server Error - Our fault | Something went wrong on the server |

---

## Part 2: Authentication Basics (Intermediate Level)

### 2.1 Authentication vs Authorization

These two are **different things** but work together:

| Aspect | Authentication | Authorization |
|---|---|---|
| **Question** | "Who are you?" | "What can you do?" |
| **Meaning** | Verifying identity | Granting permissions |
| **Example** | Checking your passport | Boarding a specific flight |
| **When** | At login | After login, for each action |

**In sequence:**
1. You login → Server checks: "Is this really you?" → **Authentication**
2. You try to delete a user → Server checks: "Do you have permission?" → **Authorization**

---

### 2.2 Basic Authentication (The Simple Way)

**How it works:**
You send your username and password **with every request**.

```
Request:
GET /api/users
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
                      ↑ This is username:password encoded in Base64
```

**Encoding (Base64)**:
- `username:password` → `dXNlcm5hbWU6cGFzc3dvcmQ=`
- It's not encryption (anyone can decode it), just encoding

**Problems with Basic Auth:**
- ❌ Password sent with **every request** (risky)
- ❌ **No encryption** unless using HTTPS
- ❌ Cannot revoke access (need to change password)
- ✅ Simple to implement

**When to use**: Only for internal/trusted systems or always over HTTPS.

#### Quick Node.js Example

```javascript
const express = require('express');
const app = express();

app.get('/api/users', (req, res) => {
  const authHeader = req.headers.authorization;
  // authHeader looks like: "Basic dXNlcm5hbWU6cGFzc3dvcmQ="
  
  if (!authHeader) {
    return res.status(401).json({ error: 'No credentials provided' });
  }

  // Decode the Base64
  const encoded = authHeader.replace('Basic ', '');
  const decoded = Buffer.from(encoded, 'base64').toString();
  // decoded = "username:password"
  
  const [username, password] = decoded.split(':');
  
  // Check if credentials match (in real app, check against hashed password in database)
  if (username === 'admin' && password === 'secret123') {
    res.json({ users: [...] });
  } else {
    res.status(401).json({ error: 'Invalid credentials' });
  }
});

app.listen(3000);
```

---

### 2.3 Token-Based Authentication

Instead of sending username/password every time, the server gives you a **token** (a unique string).

**The Flow:**
```
1. Client: "Here's my username and password"
   ↓
2. Server: Checks credentials, if valid, creates a token
   ↓
3. Client: Receives token, stores it locally
   ↓
4. For all future requests, Client: "Here's my token"
   ↓
5. Server: Checks if token is valid, serves the request
```

**Advantages over Basic Auth:**
- ✅ Password sent **only once** (at login)
- ✅ Can set **token expiration** (expires after 1 hour)
- ✅ Can **revoke access** by invalidating the token
- ✅ Token can have **limited permissions** (scope)

---

## Part 3: JWT - The Modern Way (Intermediate Level)

### 3.1 What is JWT?

**JWT** = **JSON Web Token**

It's a **signed token** that contains user data and a signature. The server can verify that the token wasn't tampered with.

**Structure**: Three parts separated by dots (`.`)

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjMiLCJuYW1lIjoiSm9obiBEb2UiLCJpYXQiOjE2fQ.
TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

Part 1: HEADER (metadata)
Part 2: PAYLOAD (data)
Part 3: SIGNATURE (proof it's real)
```

---

### 3.2 Understanding JWT Parts

#### Part 1: Header

Contains **information about the token**:

```json
{
  "alg": "HS256",    // Algorithm used to sign it
  "typ": "JWT"       // Token type
}
```

When Base64 encoded: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`

**Algorithm (alg)**:
- `HS256` = HMAC with SHA-256 (symmetric - same key to sign and verify)
- `RS256` = RSA with SHA-256 (asymmetric - different keys for signing and verifying)

---

#### Part 2: Payload (Claims)

Contains **user data**:

```json
{
  "sub": "123",              // Subject (user ID)
  "name": "John Doe",        // User's name
  "email": "john@example.com",
  "roles": ["user", "admin"], // What the user can do
  "iat": 1516239022,         // Issued at (when token was created)
  "exp": 1516325422         // Expiration time
}
```

When Base64 encoded: `eyJzdWIiOiIxMjMiLCJuYW1lIjoiSm9obiBEb2UifQ`

**Important**: 
- ⚠️ Base64 is **NOT encryption** - anyone can decode it
- ⚠️ Don't store sensitive data (passwords, credit cards) in JWT
- ✅ OK to store: user ID, email, roles, permissions

---

#### Part 3: Signature

This is the **proof that the token is real** and hasn't been modified.

**How it's created:**
```
signature = HMAC-SHA256(
  header + "." + payload,
  secret_key
)
```

**How it works:**
1. Server creates token with secret key: `secret123`
2. Signature is calculated using the secret
3. Client stores the complete token
4. Client sends token with requests
5. Server verifies the signature using the same secret key
6. If someone modifies the payload, the signature won't match → **Token rejected**

---

### 3.3 JWT Advantages

✅ **Stateless**: Server doesn't need to store tokens in database
- No need to query session database for every request
- Easy to scale across multiple servers
- Works great for microservices

✅ **Self-contained**: Token carries all info (user ID, roles, permissions)

✅ **Secure**: Cannot be modified without the secret key

❌ **Downside**: Once issued, token is valid until expiration
- Can't revoke a token immediately
- If token is stolen, attacker can use it until it expires
- Mitigation: Keep expiration time short (1 hour)


**Request flow:**
```
1. Client: POST /login (with username/password)
2. Server: Creates token, sends: { token: "eyJ..." }
3. Client: Stores token (localStorage, cookie, etc.)
4. Client: Requests GET /api/profile + header: "Authorization: Bearer eyJ..."
5. Server: Verifies token using secret key
6. If valid: Serves the data
7. If invalid/expired: Returns 401 error
```

---

## Part 4: OAuth 2.0 - Delegated Access (Intermediate to Advanced)

### 4.1 The Problem OAuth Solves

**Scenario**: You want to login to Spotify using your Google account.

**The OLD dangerous way:**
```
Spotify: "Give us your Google password"
You: "Here's my Google password"
Spotify: Stores your Google password 😱
```

**Problems:**
- ❌ You give your password to Spotify
- ❌ Spotify has access to EVERYTHING in your Google account
- ❌ If Spotify gets hacked, your Google account is compromised

**The OAuth 2.0 way:**
```
You: "I want to login to Spotify using Google"
   ↓
Spotify: "Redirect to Google"
   ↓
Google: "Spotify is asking for permission to access your profile. Allow?"
   ↓
You: "Yes, allow"
   ↓
Google: "Here's a temporary permission token for Spotify"
   ↓
Spotify: Uses token to get your profile info (NOT your password)
   ↓
Spotify: Logs you in
```

**Benefits:**
✅ You never give your password to Spotify
✅ Google never shares your password
✅ You can revoke access anytime
✅ Spotify can only access what you allowed

---

### 4.2 OAuth 2.0 Flow (The Main Players)

```
┌──────────────┐          ┌────────────────┐          ┌─────────────────┐
│ RESOURCE     │          │ AUTHORIZATION  │          │   RESOURCE      │
│ OWNER        │          │ SERVER         │          │   SERVER        │
│ (You)        │          │ (Google)       │          │ (Google Data)   │
│              │          │                │          │                 │
│ "I want to   │          │ Checks if app  │          │ Returns user    │
│ login to     │          │ is valid,      │          │ info when token │
│ Spotify"     │          │ asks for       │          │ is valid        │
└──────────────┘          │ permission     │          └─────────────────┘
        │                 │                │                  │
        └─────────────────→ (1) Request auth code ─────────────│
                          │                │                   │
        ┌──────────────────────────────────────────────────────┘
        │
        │ (2) Redirects back with code
        │
┌───────↓──────────────┐
│    CLIENT APP        │
│ (Spotify)            │
│                      │
│ Takes the code and   │
│ exchanges for token  │
└──────────────────────┘
```

---

### 4.3 OAuth 2.0 Step-by-Step

#### Step 1: User Initiates Login
```
You click "Login with Google" on Spotify
```

#### Step 2: Redirect to Google
```
Spotify redirects you to:
https://accounts.google.com/o/oauth2/v2/auth?
  client_id=spotify_app_id
  redirect_uri=https://spotify.com/callback
  scope=profile email
  response_type=code
  state=random_string
```

**Parameters:**
- `client_id` = Spotify's unique ID (registered with Google)
- `redirect_uri` = Where to send you back (Spotify's callback)
- `scope` = Permissions Spotify is asking for (profile, email)
- `response_type=code` = "Give me an authorization code"

#### Step 3: Google Asks for Permission
You see: "Spotify wants access to your profile and email. Allow?"

You click: "Allow"

#### Step 4: Google Sends Code Back
Google redirects back to Spotify with:
```
https://spotify.com/callback?
  code=authorization_code_here
  state=random_string
```

#### Step 5: Spotify Exchanges Code for Token
**Behind the scenes** (user doesn't see):
```
Spotify talks to Google's server:
POST /token
  code=authorization_code_here
  client_id=spotify_app_id
  client_secret=spotify_secret_key  ← Spotify keeps this secret

Google checks everything, then sends back:
{
  "access_token": "token_for_spotify",
  "refresh_token": "use_to_get_new_token_later",
  "expires_in": 3600  // 1 hour
}
```

#### Step 6: Spotify Gets Your Data
```
Spotify uses token to call Google's API:
GET /userinfo
  Authorization: Bearer token_for_spotify

Google returns:
{
  "id": "user123",
  "name": "John Doe",
  "email": "john@google.com"
}
```

#### Step 7: You're Logged In
Spotify creates its own session and logs you in.

---

### 4.4 Why OAuth Is Better

| Aspect | Basic Auth | Token Auth | OAuth 2.0 |
|---|---|---|---|
| Give password to app? | ❌ Yes | ❌ Once | ✅ No |
| App access to everything? | ❌ Yes | ❌ Yes | ✅ Only what you allow |
| Can revoke immediately? | ❌ No | ❌ No | ✅ Yes |
| Secure? | ❌ No | ✅ Medium | ✅ High |
| Complexity | ✅ Simple | ✅ Medium | ❌ Complex |

---

## Part 5: OpenID Connect (Advanced)

### 5.1 What is OpenID Connect?

**OpenID Connect** = **OAuth 2.0 + Identity Information**

OAuth is about **access** ("What data can I get?")
OpenID Connect is about **identity** ("Who are you?")

---

### 5.2 OpenID vs OAuth

```
OAuth 2.0:
"Here's a token to access Google Drive, Gmail, etc."
→ Focused on AUTHORIZATION (what resources you can access)

OpenID Connect:
"Here's a token that proves you are John Doe"
→ Focused on AUTHENTICATION (proving who you are)
```

---

### 5.3 How OpenID Connect Works

It's the **same as OAuth** but with an extra step:

```
1. Same as OAuth: User approves, app gets authorization code
   ↓
2. App exchanges code for: 
   - access_token (for API calls)
   - id_token (NEW! Contains user identity info)
   ↓
3. App verifies id_token to prove user identity
   ↓
4. App logs user in (doesn't need separate login)
```

**The ID Token** (a JWT):
```json
{
  "iss": "https://accounts.google.com",  // Who issued it
  "sub": "user123",                      // Subject (user ID)
  "aud": "spotify_client_id",            // Intended audience
  "iat": 1622500000,                     // Issued at
  "exp": 1622503600,                     // Expires
  "name": "John Doe",
  "email": "john@google.com",
  "email_verified": true
}
```

---

## Part 6: GraphQL - An Alternative to REST (Advanced)

### 6.1 What is GraphQL?

**GraphQL** = **Query Language for APIs**

It's an alternative to REST. Instead of having many endpoints, you have **one endpoint** and ask for exactly what you need.

**Comparison:**

| Aspect | REST | GraphQL |
|---|---|---|
| Endpoints | Many (`/users`, `/posts`, `/comments`) | One (`/graphql`) |
| What you get | All data | Only what you ask for |
| Over-fetching | ❌ Gets extra data you don't need | ✅ Gets only what you ask for |
| Under-fetching | ❌ Multiple requests needed | ✅ One request can get related data |
| Learning curve | ✅ Simple | ❌ Steeper |

---

### 6.2 REST vs GraphQL Example

**REST Approach:**

To get user posts and their comments:
```javascript
// Request 1: Get user
GET /users/123
Response: { id, name, email, phone, address, ... } (lots of extra stuff)

// Request 2: Get user's posts
GET /users/123/posts
Response: [{ id, title, content, likes, created_at, ... }, ...]

// Request 3: Get comments for each post
GET /posts/1/comments
GET /posts/2/comments
// ... multiple requests!
```

**GraphQL Approach:**

```graphql
query {
  user(id: 123) {
    name
    email
    posts {
      title
      comments {
        text
        author {
          name
        }
      }
    }
  }
}
```

Response: Only the fields you asked for!

```json
{
  "data": {
    "user": {
      "name": "John Doe",
      "email": "john@example.com",
      "posts": [
        {
          "title": "My First Post",
          "comments": [
            {
              "text": "Great post!",
              "author": { "name": "Jane" }
            }
          ]
        }
      ]
    }
  }
}
```

---

### 6.3 GraphQL Advantages

✅ **Efficient**: Get exactly what you need (no over-fetching)
✅ **Fewer requests**: One query can get nested data
✅ **Self-documenting**: Schema describes all available data
✅ **Great for mobile**: Smaller payloads = less bandwidth

❌ **Harder to learn** than REST
❌ **Caching is more complex** (not just by URL)
❌ **Overkill for simple APIs**

---

## Part 7: SOAP - The Old Way (Advanced)

### 7.1 What is SOAP?

**SOAP** = **Simple Object Access Protocol**

It's an **old, heavier** way to call APIs using XML.

**SOAP vs REST:**

| Aspect | SOAP | REST |
|---|---|---|
| Data format | XML | JSON |
| Complexity | ❌ Heavy, complex | ✅ Simple |
| Standards | ❌ Strict, rigid | ✅ Flexible |
| Learning curve | ❌ Steep | ✅ Gentle |
| Modern use | ❌ Legacy systems | ✅ Modern web |

**Example SOAP request:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap-envelope/">
  <soap:Body>
    <GetUserRequest xmlns="http://example.com">
      <UserID>123</UserID>
    </GetUserRequest>
  </soap:Body>
</soap:Envelope>
```

**Why REST won**: Simpler, lighter, easier to use.

---

## Part 8: Security Best Practices (Advanced)

### 8.1 Common Mistakes

❌ **Storing passwords in plain text**
```javascript
// WRONG
users.push({ username: "john", password: "mypassword123" });
```

✅ **Hash passwords using bcrypt**
```javascript
const bcrypt = require('bcrypt');
const hashedPassword = await bcrypt.hash('mypassword123', 10);
users.push({ username: "john", password: hashedPassword });
```

---

❌ **Storing sensitive data in JWT**
```javascript
// WRONG
jwt.sign({
  user_id: 123,
  credit_card: "4532-1234-5678-9999", // Never do this!
  password_hash: "..."
}, secret);
```

✅ **Only store non-sensitive data**
```javascript
jwt.sign({
  user_id: 123,
  email: "user@example.com",
  roles: ["user"]
}, secret);
```

---

❌ **Sending tokens in URLs**
```javascript
// WRONG
https://api.example.com/data?token=eyJ...
// Token visible in browser history, logs
```

✅ **Send tokens in headers**
```javascript
// RIGHT
Authorization: Bearer eyJ...
// Secure, follows standards
```

---

### 8.2 Token Expiration Strategy

**Short-lived access token + Refresh token**:

```javascript
// At login:
const accessToken = jwt.sign(userData, secret, { expiresIn: '15m' });
const refreshToken = jwt.sign(userData, secret, { expiresIn: '7d' });

// Send both to client
res.json({
  accessToken,      // Use for API calls (expires in 15 min)
  refreshToken      // Use to get new access token (expires in 7 days)
});
```

**When access token expires:**
```
Client: "Access token expired, here's my refresh token"
Server: Validates refresh token, creates new access token
Client: Uses new access token for next request
```

**Advantages:**
✅ If access token is stolen, attacker has only 15 minutes
✅ Even if refresh token is stolen, you can invalidate it
✅ Better security than one long-lived token

---

### 8.3 HTTPS is Essential

❌ **HTTP (no encryption)**: Anyone can see your data
```
GET /login
Username: john
Password: mypassword123
↑ Anyone on the network can see this!
```

✅ **HTTPS (encrypted)**: Data is scrambled
```
GET /login [ENCRYPTED]
↑ Only your computer and the server can see the actual data
```

**Rule**: Always use HTTPS in production. HTTP is only for local development.

---

## Part 9: Summary & Architecture Diagram

### 9.1 Authentication Methods Comparison

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         AUTHENTICATION METHODS                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  BASIC AUTH              TOKEN AUTH             JWT                    │
│  ───────────            ──────────            ─────                    │
│  User + Password    →   Server creates    →  Server creates           │
│  sent with each         token               self-signed token          │
│  request               (opaque string)      (contains data +           │
│                                             signature)                  │
│  ✅ Simple              ✅ Stateless          ✅ Scalable               │
│  ❌ Insecure            ✅ Can expire         ❌ Can't revoke           │
│  ❌ Prone to leaks      ✅ Can revoke         ✅ Self-contained        │
│                                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  OAUTH 2.0                              OpenID Connect                │
│  ──────────                             ──────────────                │
│  Delegated authorization                OAuth + Identity               │
│  (Login with Google/Facebook)           (Login + Profile info)         │
│  ✅ User never shares password          ✅ Proves who you are         │
│  ✅ Can revoke access                   ✅ Industry standard           │
│  ❌ Complex implementation               ✅ Works with OAuth           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### 9.2 When to Use What

| Scenario | Use |
|---|---|
| Internal APIs, trusted users | Basic Auth (over HTTPS) |
| Simple REST API with user sessions | JWT tokens |
| Multiple servers/microservices | JWT tokens |
| "Login with Google" feature | OAuth 2.0 + OpenID Connect |
| Complex data fetching | GraphQL |
| Legacy enterprise systems | SOAP |

---

### 9.3 Complete Authentication Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          USER WANTS TO LOGIN                            │
└────────────────────────────┬────────────────────────────────────────────┘
                             │
                    ┌────────↓─────────┐
                    │ OPTIONS:         │
                    │ 1. Username/PW   │
                    │ 2. Social login  │
                    └────┬────────────┘
                         │
        ┌────────────────┴────────────────┐
        │                                 │
        ↓ (Option 1)                      ↓ (Option 2)
┌───────────────────┐          ┌──────────────────────┐
│ Basic/Token Auth  │          │  OAuth 2.0 Flow      │
├───────────────────┤          ├──────────────────────┤
│ 1. Send username  │          │ 1. Redirect to       │
│    & password     │          │    Google/Facebook   │
│ 2. Server hashes  │          │ 2. User approves     │
│    & verifies     │          │ 3. Get auth code     │
│ 3. Create token   │          │ 4. Exchange code     │
│ (JWT/opaque)      │          │    for token         │
│ 4. Send to client │          │ 5. Get user info     │
└─────────┬─────────┘          │ 6. Create session    │
          │                    └──────────┬───────────┘
          │                               │
          └───────────────┬───────────────┘
                          │
                 ┌────────↓────────┐
                 │ CLIENT STORES:  │
                 │ - Token         │
                 │ - Session info  │
                 └────────┬────────┘
                          │
                ┌─────────↓──────────┐
                │ FOR FUTURE:        │
                │ Add token to       │
                │ Authorization      │
                │ header             │
                │ Bearer token_here  │
                └────────┬───────────┘
                         │
            ┌────────────↓────────────┐
            │ SERVER VERIFIES:        │
            │ - Token not expired     │
            │ - Token signature valid │
            │ - User has permissions  │
            └────────────┬────────────┘
                         │
            ┌────────────↓────────────┐
            │ SERVE REQUEST OR        │
            │ DENY WITH 401/403       │
            └────────────────────────┘
```

---



## Part 10: Key Takeaways

1. **APIs** let applications talk to each other
2. **REST** is the most common API style (GET, POST, PUT, DELETE)
3. **Authentication** proves who you are
4. **Authorization** controls what you can do
5. **JWT** is a modern way to handle stateless authentication
6. **OAuth 2.0** lets users login with Google/Facebook safely
7. **OpenID Connect** adds identity on top of OAuth
8. **GraphQL** is an alternative to REST with more flexibility
9. **Always use HTTPS** in production
10. **Never trust user input** - always validate and sanitize

---

## Quick Reference

### HTTP Status Codes
```
200 OK              ✅ Request successful
201 Created         ✅ Resource created
400 Bad Request     ❌ Invalid request
401 Unauthorized    ❌ No valid auth
403 Forbidden       ❌ Auth OK, no permission
404 Not Found       ❌ Resource doesn't exist
500 Server Error    ❌ Server problem
```

### HTTP Methods
```
GET     Read data
POST    Create data
PUT     Replace data
PATCH   Update data
DELETE  Remove data
```

### JWT Anatomy
```
Header.Payload.Signature
  ↓        ↓         ↓
Algorithm  Data    Proof
```


