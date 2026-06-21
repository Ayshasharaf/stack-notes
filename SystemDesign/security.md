**Cryptography, Hashing, HTTPS, CORS, and Server Security**

---

## Part 1: Cryptography Basics (Beginner Level)

### 1.1 What is Cryptography?

**Cryptography** = **Art of protecting information using codes**

**Two main purposes:**
1. **Encryption** - Scrambling data so only authorized people can read it
2. **Hashing** - Creating a fingerprint of data to detect changes

```
Original Data: "Hello World"
         │
         ├─→ ENCRYPTION (reversible) → "X#$@%^&" → (decrypt) → "Hello World"
         │
         └─→ HASHING (one-way) → "a591a6d40bf420404a011733cfb7b190" → (can't reverse)
```

---

### 1.2 Two Types of Cryptography

#### Symmetric Encryption (Same Key)
```
┌─────────────────────────────────────┐
│ ENCRYPTION & DECRYPTION             │
├─────────────────────────────────────┤
│ Message: "Secret"                   │
│ Key: "mypassword"                   │
│         │                           │
│         ├─→ ENCRYPT → "X#$@%^"     │
│         │              │            │
│         └─ DECRYPT ←───┘            │
│         (same key)                  │
└─────────────────────────────────────┘

Example: AES, DES
Use case: File encryption, database encryption
```

#### Asymmetric Encryption (Public + Private Key)
```
┌──────────────────────────────────────┐
│ TWO KEYS: Public (share) & Private   │
├──────────────────────────────────────┤
│                                      │
│  Someone sends encrypted message     │
│  with YOUR public key                │
│         │                            │
│         ├─→ ENCRYPT (public key)    │
│         │   → "X#$@%^"              │
│         │                            │
│  Only YOU can decrypt it             │
│  with YOUR private key               │
│         │                            │
│         └─→ DECRYPT (private key)   │
│             → "Secret"              │
│                                      │
└──────────────────────────────────────┘

Example: RSA, ECC
Use case: HTTPS, SSL certificates, digital signatures
```

**Real analogy:**
- **Symmetric**: One key to your house - everyone with it can enter/exit
- **Asymmetric**: Mailbox - anyone can drop mail in (public key), only you have the key to open it (private key)

---

## Part 2: Hashing - The One-Way Function (Beginner to Intermediate)

### 2.1 What is Hashing?

**Hashing** = **Converting data to a fixed-size fingerprint**

**Key properties:**
- ✅ **One-way**: Can't reverse (can't get original from hash)
- ✅ **Fixed output**: Always same length (MD5 = 32 chars, SHA256 = 64 chars)
- ✅ **Deterministic**: Same input = same hash
- ✅ **Collision-resistant**: Nearly impossible to find 2 inputs with same hash

```
Input: "password123"
    │
    ├─→ MD5:    5f4dcc3b5aa765d61d8327deb882cf99
    ├─→ SHA1:   482c811da5d5b4bc6d497ffa98491e38
    ├─→ SHA256: ef92b778bafe771e89245d171bafecca6c12e941f5ff985be2e3e953d5438b26
    └─→ (Cannot reverse - one-way!)
```

---

### 2.2 Hashing Algorithms (Evolution & Comparison)

#### MD5 (❌ Don't Use - Broken)

```javascript
// Example
Input: "password"
Output: 5f4dcc3b5aa765d61d8327deb882cf99

Input: "password" (same)
Output: 5f4dcc3b5aa765d61d8327deb882cf99 (same)
```

**Problems:**
- ❌ Collision vulnerable (can create two different inputs with same hash)
- ❌ Fast to compute (easy to brute force)
- ❌ Deprecated since 2004
- ❌ **DO NOT USE FOR PASSWORDS**

**When MD5 is OK**: File integrity checks (not security-critical)

---

#### SHA-1 (⚠️ Legacy - Avoid)

**SHA** = **Secure Hash Algorithm**

```
Input: "password"
Output: 5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8 (40 characters)
```

**Problems:**
- ⚠️ Theoretically broken (collisions found in 2017)
- ⚠️ Still used in older systems
- ❌ **Not recommended for new projects**

---

#### SHA-256 (✅ Good - Use This)

```
Input: "password"
Output: 5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8
        (64 characters)
```

**Advantages:**
- ✅ Cryptographically secure
- ✅ No known practical attacks
- ✅ Used in Bitcoin, SSL certificates
- ✅ Industry standard

**Usage:**
- HTTPS/SSL certificates
- Digital signatures
- File integrity verification

---

#### Scrypt (✅ Best for Passwords - Slow by Design)

```javascript
const scrypt = require('scrypt-js');

// Hashing password with scrypt
const hashed = scrypt.encryptSync('mypassword', {
  N: 16384,  // CPU cost
  r: 8,      // Block size
  p: 1       // Parallelization
});
// Takes 100ms to compute (intentionally slow!)
```

**Why Scrypt is Best for Passwords:**

```
┌─────────────────────────────────┐
│ PASSWORD HASHING SHOWDOWN       │
├─────────────────────────────────┤
│                                 │
│ MD5:      0.000001ms per hash  │
│ SHA256:   0.000005ms per hash  │
│ Scrypt:   100ms per hash       │
│           ↑ INTENTIONALLY SLOW │
│                                 │
└─────────────────────────────────┘
```

**Why slow is good:**

Imagine an attacker with a list of 100,000 password hashes.

```
Attack with MD5:
- Try 1 million passwords per second
- Crack 100,000 hashes in 0.1 seconds
- ❌ Disaster!

Attack with Scrypt:
- Try 1 password per second (100ms each)
- Would need 100,000 seconds = 27+ hours to crack
- ✅ Attacker gives up!
```

---

#### bcrypt (✅ Recommended - Designed for Passwords)

**bcrypt** = **Blowfish + Salt + Work Factor**

It's the **most popular** way to hash passwords in modern applications.

```javascript
const bcrypt = require('bcrypt');

// Hashing
const password = "mySecurePassword123";
const saltRounds = 10;  // Work factor (higher = slower = safer)

const hashedPassword = await bcrypt.hash(password, saltRounds);
// Output: $2b$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcg7b3XeKeUxWdeS86E36jbMnu.
//         ↑   ↑  ↑
//         |   |  └─ Hash (different every time)
//         |   └─ Salt rounds
//         └─ Algorithm version

// Verifying
const isCorrect = await bcrypt.compare(password, hashedPassword);
// true or false
```

**Why bcrypt is better than SHA256 for passwords:**

```
SHA256:
const hash = sha256("password");
// 5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8

Problem: Same password = same hash
        User 1: password → 5e88...
        User 2: password → 5e88...  ← BOTH ARE THE SAME!
        
Attacker can see two users have the same password!

bcrypt:
const hash1 = await bcrypt.hash("password", 10);
// $2b$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcg7b3XeKeUxWdeS86E36jbMnu.

const hash2 = await bcrypt.hash("password", 10);
// $2b$10$abcdefghijklmnopqrstuvwxyzABCDEF123456789012345678901

Different every time because bcrypt adds SALT!
```

---

### 2.3 Salt - Making Hashes Unique

**Salt** = **Random data added to password before hashing**

```
Without salt:
password + MD5 = always same hash (attackers use rainbow tables)

┌─────────────────────────────────┐
│ RAINBOW TABLE (pre-computed)    │
├─────────────────────────────────┤
│ 5f4dcc3b5aa → password          │
│ 827ccb0eea8 → password123       │
│ 1bb9d6bda26 → qwerty            │
│ ... millions of them             │
└─────────────────────────────────┘

Attacker: Sees hash 5f4dcc3b5aa → Looks up in table → "password" ✗

With salt:
password + random_salt + MD5 = unique hash every time

┌──────────────────────────────────────────┐
│ User 1: password + salt_abc_xyz → HASH1  │
│ User 2: password + salt_def_uvw → HASH2  │
│                                          │
│ Same password, different hashes!         │
│ Rainbow table attack fails! ✓            │
└──────────────────────────────────────────┘
```

**bcrypt does this automatically:**

```javascript
// bcrypt generates and stores salt inside the hash
const hash = await bcrypt.hash("password", 10);
// $2b$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcg7b3XeKeUxWdeS86E36jbMnu.
//         ↑ This is the salt (stored inside)
```

---

### 2.4 Comparison Table

| Algorithm | Speed | Security | Use Case | Recommendation |
|---|---|---|---|---|
| **MD5** | Very fast ⚡ | ❌ Broken | Not recommended | ❌ Don't use |
| **SHA-1** | Fast | ⚠️ Weak | Legacy systems | ⚠️ Avoid |
| **SHA-256** | Fast | ✅ Good | File verification, certificates | ✅ OK |
| **Scrypt** | Slow (intentional) | ✅ Excellent | Password hashing | ✅ Best |
| **bcrypt** | Slow (intentional) | ✅ Excellent | Password hashing | ✅ Recommended |

---

## Part 3: HTTPS, SSL, and TLS (Intermediate Level)

### 3.1 HTTP vs HTTPS

#### HTTP (Insecure)
```
Client: "I want to send: username=john, password=secret123"
    │
    │ (sent as plain text over the internet)
    │
    ↓
Attacker on the network: "Oh cool, his password is secret123!" ✗
```

#### HTTPS (Secure)
```
Client: "I want to send: username=john, password=secret123"
    │
    │ (encrypted with server's public key)
    │
    ├─→ "X#$@%^&*(#$@)*&@(#$"
    │
    ↓
Attacker on the network: "I see encrypted data... can't read it" ✓
    │
    ↓
Server: (decrypts with private key)
    │
    ↓
Server: "OK, username is john, password is secret123" ✓
```

**Key difference:**
- HTTP: All data visible on network
- HTTPS: Data encrypted end-to-end

---

### 3.2 How HTTPS Works (SSL/TLS Handshake)

**SSL/TLS** = **Secure Sockets Layer / Transport Layer Security**

```
┌──────────────────────────────────────────────────────────┐
│              HTTPS CONNECTION SETUP                      │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  1. CLIENT → SERVER: "Hi, let's establish secure conn"  │
│                                                          │
│  2. SERVER → CLIENT: "Here's my certificate with        │
│                       my public key"                     │
│                       (signed by trusted CA)             │
│                                                          │
│  3. CLIENT: Verifies certificate is real                │
│            (checks digital signature)                    │
│                                                          │
│  4. CLIENT → SERVER: Sends session key encrypted        │
│                      with server's public key            │
│                                                          │
│  5. SERVER: Decrypts with private key, gets session key  │
│                                                          │
│  6. Both: Now use session key to encrypt all data        │
│          (symmetric encryption - faster)                 │
│                                                          │
│  ✅ Secure connection established!                       │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

### 3.3 SSL Certificates

**What is an SSL Certificate?**

A digital document that:
- ✅ Proves the website is who they claim to be
- ✅ Contains the server's public key
- ✅ Is digitally signed by a trusted Certificate Authority (CA)

**Certificate Contents:**
```
Certificate: example.com
├─ Domain Name: example.com
├─ Public Key: MIIBIjANBgkqhkiG9w0BAQE...
├─ Valid From: Jan 1, 2024
├─ Valid Until: Jan 1, 2025
├─ Issued By: "Let's Encrypt"
├─ Serial Number: 123456789
└─ Digital Signature: X#$@%^&*()_+{}|:"<>?
   (proof it wasn't modified)
```

**Certificate Types:**

| Type | Cost | Validation | Best For |
|---|---|---|---|
| **Self-signed** | Free ❌ | None | Testing only |
| **DV (Domain Validation)** | $5-20/year | Check domain ownership | Personal blogs |
| **OV (Organization Validation)** | $20-100/year | Check business exists | Business websites |
| **EV (Extended Validation)** | $100-500/year | Thorough business check | E-commerce sites |

**Where to Get:**
- Let's Encrypt (free) - Recommended for most
- Cloudflare, DigiCert, GoDaddy (paid)

---

### 3.4 The Certificate Chain

```
┌──────────────────────────────────┐
│    ROOT CA CERTIFICATE           │ ← Trusted by browsers
│    (Verisign, Comodo, etc.)      │   Built-in
└──────────────┬───────────────────┘
               │
               │ Signs
               ↓
┌──────────────────────────────────┐
│    INTERMEDIATE CA CERTIFICATE   │
│    (Let's Encrypt, Cloudflare)   │
└──────────────┬───────────────────┘
               │
               │ Signs
               ↓
┌──────────────────────────────────┐
│    YOUR WEBSITE CERTIFICATE      │
│    (example.com)                 │
└──────────────────────────────────┘

Browser verification:
1. Gets your certificate
2. Checks: Who signed it? (Intermediate CA)
3. Checks: Who signed the Intermediate? (Root CA)
4. Root CA is in browser's trusted store → ✅ Valid!
```

---

### 3.5 HTTPS in Action

**Browser URL bar clues:**
```
🔒 https://example.com   ← Green lock = valid certificate
🔓 http://example.com    ← No lock = not encrypted
⚠️  https://[warning]    ← Yellow warning = certificate issue
```

---

## Part 4: AJAX (Asynchronous JavaScript) (Intermediate Level)

### 4.1 What is AJAX?

**AJAX** = **Asynchronous JavaScript and XML**

**Traditional way (Synchronous):**
```
User clicks button
    ↓
Page reloads completely
    ↓
Server sends whole new HTML page
    ↓
Page shows new content
    ↓
User waits during entire process ⌛
```

**AJAX way (Asynchronous):**
```
User clicks button
    ↓
JavaScript sends request in background
    ↓
Page stays responsive (user can keep clicking)
    ↓
Server sends just the data (JSON)
    ↓
JavaScript updates only the part that changed
    ↓
User sees instant response ⚡
```

---

### 4.2 Why AJAX Matters

```
Before AJAX (Web 1.0):
├─ Click a button
├─ Entire page reloads
├─ Flash of white screen ⌛
└─ Frustrating experience

After AJAX (Web 2.0):
├─ Click a button
├─ Only relevant part updates
├─ Page stays responsive ⚡
└─ Feels like desktop app
```

**Real examples:**
- Google Maps (smooth pan/zoom)
- Gmail (instant search, load messages without refresh)
- Facebook (infinite scroll without page reloads)
- Twitter (load tweets as you scroll)

---
### 4.3 AJAX vs Traditional Flow Diagram

```
TRADITIONAL (Synchronous):
┌─────────────┐
│ User Action │
└──────┬──────┘
       │ Click → Full page request
       ↓
┌──────────────────┐
│ Server processes │ ⌛ User waits
└──────┬───────────┘
       │ Full HTML response
       ↓
┌──────────────────┐
│ Entire page      │
│ reloads          │ Flash of white
└──────────────────┘

AJAX (Asynchronous):
┌─────────────┐
│ User Action │
└──────┬──────┘
       │ Click → Background request (fetch/XMLHttpRequest)
       │         Page stays responsive
       ↓
┌──────────────────┐
│ Server processes │ ⚡ User keeps working
└──────┬───────────┘
       │ JSON data (only needed info)
       ↓
┌──────────────────┐
│ JavaScript       │
│ updates DOM      │ Smooth update
│ (only changed    │
│  parts)          │
└──────────────────┘
```

---



## Part 5: CORS - Cross-Origin Resource Sharing (Intermediate to Advanced)

### 5.1 The Problem: Same-Origin Policy

**What is "Origin"?**
```
URL: https://example.com:8080/api/users
     └─ Protocol: https
     └─ Domain: example.com
     └─ Port: 8080

Two origins are SAME if protocol + domain + port match.
```

**Examples:**
```
https://example.com/api        } SAME ORIGIN
https://example.com/users      } (same protocol, domain, port)

https://example.com            } DIFFERENT
https://example.com:8080       } (different port)

https://example.com            } DIFFERENT
http://example.com             } (different protocol)

https://example.com            } DIFFERENT
https://api.example.com        } (different domain)
```

---

### 5.2 Why Same-Origin Policy Exists

**The Security Problem:**

```
You're logged into: facebook.com (session stored)
    │
    ├─ Evil website: hacker.com opens
    │
    ├─ Hacker.com runs JavaScript:
    │  fetch('https://facebook.com/api/friends/delete?id=123')
    │
    ├─ Your browser has active Facebook session
    │
    └─ ❌ CSRF Attack: Hacker deletes your friend without permission!
```

**Same-Origin Policy prevents this:**
```
Hacker.com tries to request Facebook API
    │
    ├─ Browser blocks it
    │
    └─ ✅ "Not allowed - different origin!"
```

---

### 5.3 The Problem: Legitimate Cross-Origin Requests

But sometimes you NEED to access APIs from different domains:

```
Your website: example.com
Needs data from: api.example.com (different domain = different origin)
                └─ Could be your own API!

JavaScript fetch from example.com:
fetch('https://api.example.com/users')
    │
    ├─ Browser blocks it
    │
    └─ ❌ CORS Error!
```

**CORS** solves this!

---

### 5.4 How CORS Works

**CORS** = **Cross-Origin Resource Sharing**

It's a **permission system**. The server says: "I allow requests from these origins."

```
┌──────────────────────────────────────────────────────────┐
│              CORS REQUEST FLOW                           │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ 1. Browser makes request from example.com               │
│    to api.example.com                                   │
│                                                          │
│    Request includes:                                    │
│    Origin: https://example.com                          │
│                                                          │
│ 2. Server (api.example.com) receives request            │
│                                                          │
│ 3. Server checks:                                       │
│    "Is https://example.com in my allowed list?"         │
│                                                          │
│ 4a. If YES:                                             │
│     Response headers include:                           │
│     Access-Control-Allow-Origin: https://example.com    │
│                                                          │
│ 4b. If NO:                                              │
│     Response headers include:                           │
│     (No Access-Control headers)                         │
│                                                          │
│ 5. Browser checks response headers                      │
│                                                          │
│ 5a. If allowed: ✅ JavaScript gets the data            │
│ 5b. If blocked: ❌ CORS Error, no access to data       │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

### 5.5 CORS Headers Explained

**Request Headers (Browser sends):**
```
GET /api/users HTTP/1.1
Host: api.example.com
Origin: https://example.com
Access-Control-Request-Method: GET
Access-Control-Request-Headers: authorization
```

**Response Headers (Server sends):**
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: authorization, content-type
Access-Control-Max-Age: 86400
Access-Control-Allow-Credentials: true

[Response body]
```

**Key headers:**

| Header | Meaning | Example |
|---|---|---|
| `Access-Control-Allow-Origin` | Which origins are allowed | `*` or `https://example.com` |
| `Access-Control-Allow-Methods` | Which HTTP methods are allowed | `GET, POST, PUT, DELETE` |
| `Access-Control-Allow-Headers` | Which headers are allowed | `authorization, content-type` |
| `Access-Control-Max-Age` | Cache preflight for X seconds | `86400` (1 day) |
| `Access-Control-Allow-Credentials` | Allow cookies/auth? | `true` or `false` |

---


### 5.6 CORS Errors & Solutions

**Error:** `Access to XMLHttpRequest has been blocked by CORS policy`

**Causes & Solutions:**

```
1. Server doesn't have CORS headers
   └─ Solution: Add CORS middleware to server

2. Origin not in Allow-Origin list
   └─ Solution: Add your domain to allowed list

3. Credentials not allowed
   └─ Solution: Set Access-Control-Allow-Credentials: true

4. Method not allowed (POST when only GET allowed)
   └─ Solution: Add method to Access-Control-Allow-Methods

5. Custom header not allowed (e.g., Authorization)
   └─ Solution: Add header to Access-Control-Allow-Headers
```

---

### 5.8 Preflight Requests

For complex requests, browser sends a **preflight** request first:

```
Complex request (POST with custom header):

PREFLIGHT (automatic):
OPTIONS /api/users HTTP/1.1
Origin: https://example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: authorization

Server responds:
Access-Control-Allow-Methods: POST
Access-Control-Allow-Headers: authorization

IF OK, browser sends actual request:
POST /api/users HTTP/1.1
Authorization: Bearer token
Content: {...}
```

**Simple requests** (no preflight):
```
GET, HEAD, POST (with simple Content-Type)
No custom headers
└─ Skips preflight, goes straight to request
```

**Complex requests** (need preflight):
```
PUT, DELETE, PATCH
Custom headers (Authorization, X-Custom-Header)
Content-Type: application/json
└─ Browser first sends OPTIONS preflight
```

---

## Part 6: OWASP & Web Security (Advanced)

### 6.1 What is OWASP?

**OWASP** = **Open Worldwide Application Security Project**

It's an **open-source community** that:
- ✅ Identifies common security vulnerabilities
- ✅ Provides tools and resources
- ✅ Publishes best practices
- ✅ **Free** and widely used

---


## Part 7: CSP - Content Security Policy (Advanced)

### 7.1 What is CSP?

**CSP** = **Content Security Policy**

It's a **security header** that controls what resources (scripts, images, etc.) can load on your page.

**Why?** Prevent XSS attacks and data theft.

---

## Part 8: Server Hardening (Advanced)

### 8.1 What is Server Hardening?

**Server Hardening** = **Reducing vulnerabilities and attack surface**

It involves securing:
- **OS hardening** (operating system)
- **Application hardening** (your code)
- **Database hardening** (data protection)
- **Network hardening** (firewalls, etc.)

---

### 8.2 OS Hardening

**1. Keep OS Updated**
```bash
# Regular updates patch security vulnerabilities
sudo apt-get update
sudo apt-get upgrade

# Set automatic updates
sudo apt-get install unattended-upgrades
```

**2. Remove Unnecessary Services**
```bash
# Don't run FTP, Telnet, etc. if not needed
sudo systemctl disable vsftpd
sudo systemctl disable telnet

# Only run what you need: SSH, your web server, etc.
```

**3. Configure Firewall**
```bash
# Only allow specific ports
sudo ufw enable
sudo ufw allow 22/tcp  # SSH
sudo ufw allow 80/tcp  # HTTP
sudo ufw allow 443/tcp # HTTPS

# Block all else
sudo ufw default deny incoming
```

**4. Disable Unnecessary Accounts**
```bash
# Remove test/demo accounts
sudo userdel testuser

# Disable root login via SSH
sudo nano /etc/ssh/sshd_config
# Set: PermitRootLogin no
```

**5. Use Strong SSH Keys**
```bash
# Generate 4096-bit key (not default 2048)
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa

# Disable password auth (keys only)
sudo nano /etc/ssh/sshd_config
# PasswordAuthentication no
```

---



## Part 10: Quick Reference

### Algorithms Cheat Sheet
```
HASHING (One-way):
MD5      → Don't use ❌
SHA1     → Avoid ⚠️
SHA256   → OK for non-password use
bcrypt   → Best for passwords ✅
Scrypt   → Also good for passwords ✅

ENCRYPTION:
AES-256  → Symmetric (fast, one key)
RSA-2048 → Asymmetric (slow, two keys)

PROTOCOLS:
HTTP     → Unencrypted ❌
HTTPS    → Encrypted ✅
```

### Status Codes for Auth
```
200 OK               ✅ Success
201 Created          ✅ Resource created
400 Bad Request      ❌ Invalid input
401 Unauthorized     ❌ Not authenticated
403 Forbidden        ❌ No permission
404 Not Found        ❌ Resource missing
429 Too Many         ❌ Rate limited
500 Server Error     ❌ Server problem
```

---

## Part 11: Additional Resources

### Tools
- **JWT debugger**: jwt.io
- **OWASP ZAP**: Free vulnerability scanner
- **npm audit**: Check dependencies
- **Snyk**: Continuous security monitoring
- **Burp Suite**: Penetration testing
- **OWASP Top 10**: owasp.org

---
