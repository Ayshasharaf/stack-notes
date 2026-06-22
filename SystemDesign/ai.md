

## Part 1: AI Agents Fundamentals (Beginner Level)

### 1.1 What is an AI Agent?

**AI Agent** = **An AI system that can perceive, think, and act independently**

Think of it like an **employee**:
- **Traditional software**: Does exactly what you tell it (calculator)
- **AI Agent**: Understands the goal and figures out how to achieve it (personal assistant)

```
Traditional Software:
You: "Calculate 5 + 5"
Software: 10
(No thinking, just follows exact instructions)

AI Agent:
You: "I need a report on Q3 sales"
Agent:
  1. Understands: "Report on sales in Q3"
  2. Plans: "I need to query database, analyze data, format results"
  3. Acts: Executes each step
  4. Delivers: Complete report with insights
```

---

### 1.2 AI Agent vs AI Chat Bot

| Aspect | Chatbot | AI Agent |
|---|---|---|
| **Goal** | Answer questions | Solve problems |
| **Interaction** | Back and forth | Can work independently |
| **Tools** | Just talks | Can use tools (APIs, databases, files) |
| **Complexity** | Single turn | Multi-step planning |
| **Example** | ChatGPT | Claude + Tools (can write code, fetch data) |

---

### 1.3 Key Components of an AI Agent

```
┌─────────────────────────────────────────────────────────┐
│                    AI AGENT STRUCTURE                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. PERCEPTION (Input)                                 │
│     └─ Receives user request, data, environment info   │
│                                                         │
│  2. REASONING (Thinking)                               │
│     └─ Plans steps, decides what to do                 │
│                                                         │
│  3. ACTION (Execution)                                 │
│     └─ Calls tools, runs code, fetches data            │
│                                                         │
│  4. FEEDBACK LOOP (Learning)                           │
│     └─ Gets results, adjusts strategy if needed        │
│                                                         │
│  5. RESPONSE (Output)                                  │
│     └─ Delivers final result to user                   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### 1.4 Types of AI Agents

#### Reactive Agents (Simple)
```
Input → Immediate Response (no planning)

Example: Spam filter
Email arrives → Check if spam → Filter/Allow
(No complex thinking, pattern matching only)
```

#### Deliberative Agents (Planning)
```
Input → Plan → Execute → Monitor → Adjust → Output

Example: Travel booking agent
"Book a flight to NYC"
  1. Search flights
  2. Check budget
  3. Compare prices
  4. Book if affordable
  5. Send confirmation
```

#### Learning Agents (Adaptive)
```
Input → Execute → Get feedback → Learn → Adjust strategy

Example: Recommendation engine
  1. Recommend movies
  2. User watches/rates
  3. Learn preferences
  4. Improve future recommendations
```

---

### 1.5 AI Agent in Real Life

**Claude as an AI Agent:**

```
User: "Analyze this code and fix any security issues"

Agent Steps:
1. PERCEIVE: Read the code file
2. REASON: "This has SQL injection vulnerability, hardcoded password"
3. ACT: 
   - Calls code editor
   - Writes secure version
   - Tests it
4. FEEDBACK: "Found 2 issues, fixed them"
5. RESPOND: "Here's the secure code..."
```

---

## Part 2: Prompt Engineering Fundamentals (Beginner Level)

### 2.1 What is Prompt Engineering?

**Prompt Engineering** = **Art of writing instructions to get better AI responses**

Just like managing people:
- **Bad instruction**: "Make the report good"
- **Good instruction**: "Create a 5-page report with Q3 sales figures, comparisons to Q2, and recommendations"

**Result**: Better output with less back-and-forth.

---

### 2.2 Why Prompts Matter

```
Same AI, Different Prompts = Different Quality

Prompt 1 (Vague):
"Tell me about Python"

Response:
"Python is a programming language."
(Too brief, not useful)

Prompt 2 (Clear):
"Explain Python for someone learning backend web development.
Include: syntax basics, common frameworks (Django/FastAPI),
and one practical example of building an API."

Response:
"Python is popular for web backends because... [detailed explanation
with code examples]"
(Exactly what we need!)
```

---

### 2.3 Golden Rules of Prompt Engineering

#### Rule 1: Be Specific & Clear

```
❌ "Write code"
✅ "Write a Node.js Express API endpoint that accepts a POST request
   with a user's email, validates it, and stores it in MongoDB"

❌ "Explain APIs"
✅ "Explain REST APIs using the example of a Twitter-like social app.
   Include: what GET, POST, PUT, DELETE do with specific examples"
```

#### Rule 2: Provide Context

```
❌ "How do I optimize this?"
✅ "I have a Node.js API that queries MongoDB for 100,000 user records.
   The query takes 10 seconds. How can I optimize it?
   Context: We need results in under 1 second."

Without context: Generic optimization tips
With context: Specific solutions (indexing, pagination, caching)
```

#### Rule 3: Give Examples (Few-Shot Learning)

```
❌ "Format this data"
   [data]

✅ "Format this data as JSON with this structure:
   Example input: John Smith, age 30, NYC
   Example output: {\"name\": \"John Smith\", \"age\": 30, \"city\": \"NYC\"}
   
   Data to format: [actual data]"
```

#### Rule 4: Define the Role (System Prompt)

```
❌ "Explain databases"
✅ "You are a backend engineer interviewer.
   I'm a junior dev interviewing for a role.
   Ask me one medium-difficulty question about database optimization.
   Then evaluate my answer and give feedback."
```

#### Rule 5: Break Complex Tasks

```
❌ "Build a full social media app"
✅ "Step 1: Design the database schema for a social media app.
   Include: users, posts, comments, likes tables.
   
   Once done, we'll move to Step 2: API design"
   
(Easier for AI to handle, easier to review)
```

---

### 2.4 Prompt Structure (Template)

```
┌────────────────────────────────────────────────────┐
│ 1. ROLE (Optional but helpful)                     │
│    "You are a senior backend engineer..."          │
├────────────────────────────────────────────────────┤
│ 2. CONTEXT (Background info)                       │
│    "I'm building a real-time chat app with..."     │
├────────────────────────────────────────────────────┤
│ 3. TASK (What you want)                            │
│    "Design the database schema for..."             │
├────────────────────────────────────────────────────┤
│ 4. CONSTRAINTS (Limitations)                       │
│    "Must support 10,000 concurrent users..."       │
├────────────────────────────────────────────────────┤
│ 5. FORMAT (How you want it)                        │
│    "Provide: SQL schema, explanation, trade-offs"  │
├────────────────────────────────────────────────────┤
│ 6. EXAMPLES (Show what you want)                   │
│    "Like this table structure: ..."                │
└────────────────────────────────────────────────────┘
```

---

### 2.5 Prompt Examples

#### Example 1: Code Review

```
❌ Basic:
"Review this code"

✅ Good:
"You are a senior code reviewer.
Review this Node.js Express code for:
1. Security vulnerabilities
2. Performance issues
3. Code quality

Focus especially on: authentication, input validation, and database queries.

Format: List issues with: [Issue] → [Risk] → [Solution]

Code:
[paste code]"
```

#### Example 2: Learning

```
❌ Basic:
"Explain GraphQL"

✅ Good:
"I'm a backend engineer familiar with REST APIs.
Teach me GraphQL by:
1. Explaining how it differs from REST
2. Show a side-by-side comparison: REST endpoint vs GraphQL query
3. Give one real-world scenario where GraphQL is better

Use practical code examples for a Twitter-like app."
```

#### Example 3: Problem Solving

```
❌ Basic:
"How do I fix this error?"

✅ Good:
"I'm getting this error: [error message]
Context: Node.js app, PostgreSQL database, using Prisma ORM
What I did: [describe steps]
What I expected: [outcome]
What happened: [actual outcome]

Provide: Root cause + Solution + Prevention"
```

---

## Part 3: Advanced Prompt Engineering (Intermediate to Advanced)

### 3.1 Chain of Thought Prompting

Force the AI to **explain its thinking step-by-step**.

```
❌ Without Chain of Thought:
"Should we use SQL or NoSQL for our app?"
Response: "Use SQL" (no explanation)

✅ With Chain of Thought:
"Should we use SQL or NoSQL for our app?
Think step-by-step:
1. What are the trade-offs?
2. What are our data requirements?
3. What are our scaling needs?
4. Which fits best? Why?"

Response: Shows complete reasoning, better decision
```

---

### 3.2 Role-Playing (System Prompt)

Give the AI a persona to think like.

```
Prompt:
"You are Claude, an expert AI engineer at Anthropic who specializes
in building AI agents. You have shipped 10+ agent systems.

Design an AI agent architecture for: [task]

Think like someone with your experience would."

Result: More sophisticated, production-ready design
```

---

### 3.3 Constraint-Based Prompting

Add **real-world limitations** to get practical solutions.

```
Prompt:
"Design a microservices architecture for a fintech app with:
Constraints:
- Budget: $5,000/month
- Team: 3 engineers
- Latency: <100ms for transactions
- Uptime: 99.9%
- Learning curve: Must use technologies we know (Node.js, PostgreSQL)

Don't suggest: $50,000/month infrastructure, bleeding-edge tech"

Result: Realistic, implementable design
```

---

### 3.4 Persona-Based Prompting

Make the AI think like a specific person.

```
Prompt:
"You are a startup CTO with limited budget but need to move fast.
A junior dev asks: 'Should we build our own payment system or use Stripe?'

Answer as you would, considering: cost, time, risk, maintenance"

Result: Practical startup-focused advice, not over-engineered
```

---

### 3.5 Iterative Prompting (Refinement)

Start broad, then refine based on responses.

```
Round 1:
"Design a database for an e-commerce app"

[AI responds with basic design]

Round 2:
"Good start. Now optimize for:
- 1 million products
- 100k concurrent users
- Real-time inventory updates

What changes?"

[AI refines previous design with new constraints]

Round 3:
"How would you handle this specific edge case:
[describe edge case]"

[AI addresses specific concern]
```

---

### 3.6 Format Specification

Tell the AI exactly how to format output.

```
❌ Vague:
"Give me tips for learning React"

✅ Specific Format:
"Give me 5 tips for learning React.

Format each as:
**Tip Title**
Description: [1-2 sentences]
Example: [practical example]
Why it works: [explanation]"

Result: Consistent, easy-to-scan format
```

---

### 3.7 Error Correction Prompting

If the response is wrong, tell the AI what's wrong.

```
Round 1:
"Write SQL to get top 10 customers by spending"
[AI writes wrong query]

Round 2:
"That query is wrong because:
- It doesn't exclude refunds
- It doesn't group by customer correctly

Rewrite it considering:
- Refunds should be subtracted
- Only orders from last year"

[AI corrects it]
```

---

## Part 4: MCP (Model Context Protocol) Skills (Intermediate Level)

### 4.1 What is MCP?

**MCP** = **Model Context Protocol**

It's a **standard way for AI to talk to external tools and services**.

```
Traditional:
AI ↔ User (text only)

With MCP:
AI ↔ User (text)
  ↓
  ├─ Tools (Google Drive, Gmail, Slack)
  ├─ Databases
  ├─ APIs
  └─ Custom services

AI can now DO things, not just talk about them.
```

---

### 4.2 Real-World MCP Example

```
User: "Fetch my Q3 sales data and create a chart"

Without MCP:
- AI: "I can't access your data, describe it and I'll make a chart"
- User: Has to manually copy data

With MCP (Google Drive connected):
- AI: Connects to Google Drive
- AI: Reads sales spreadsheet
- AI: Creates chart in Artifact
- User: Instant visualization

(AI does the work, user just asks)
```

---

### 4.3 Common MCP Servers

These are tools AI can access if connected:

| MCP Server | What It Does | Example Use |
|---|---|---|
| **Google Drive** | Read/write files, spreadsheets | Fetch documents, create reports |
| **Gmail** | Read/send emails | Check email, draft responses |
| **Slack** | Send messages, read channels | Post updates, get channel info |
| **GitHub** | Read repos, create issues | Check code, file issues |
| **Asana** | Create/update tasks | Add todos, update projects |
| **Web Search** | Search the internet | Find current info, fact-check |

---

### 4.4 How MCP Works (Under the Hood)

```
┌─────────────────────────────────────────────────────┐
│ User: "Send a message to #sales in Slack"          │
└────────────┬────────────────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────────────────┐
│ Claude (AI) thinks:                                 │
│ "User wants to send Slack message"                  │
│ "I have access to Slack MCP"                        │
│ "Let me call the Slack tool"                        │
└────────────┬────────────────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────────────────┐
│ MCP Protocol (standardized):                        │
│ {                                                   │
│   "tool": "slack",                                  │
│   "action": "send_message",                         │
│   "params": {                                       │
│     "channel": "#sales",                            │
│     "message": "..."                                │
│   }                                                 │
│ }                                                   │
└────────────┬────────────────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────────────────┐
│ Slack API:                                          │
│ Sends message to #sales channel                     │
└────────────┬────────────────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────────────────┐
│ Response:                                           │
│ "Message sent successfully"                         │
└────────────┬────────────────────────────────────────┘
             │
             ↓
┌─────────────────────────────────────────────────────┐
│ Claude: "Done! Message sent to #sales"              │
└─────────────────────────────────────────────────────┘
```

---

### 4.5 MCP Skills Explained

An **MCP Skill** is a **set of connected tools grouped by purpose**.

```
Example MCP Skill: "Document Management"
├─ Google Drive (read/write files)
├─ Google Docs (edit documents)
└─ PDF tools (create/edit PDFs)

User can say: "Create a report and save it to Drive"
AI (with skill): Can access all 3 tools to complete the task
```

---

### 4.6 Using MCP Skills in Practice

```
Workflow:
1. You enable MCP skill (e.g., Google Drive)
2. You ask AI to do something with that tool
3. AI connects via MCP protocol
4. Tool executes
5. AI shows you results

Example:
User: "Read my sales spreadsheet and summarize Q3 numbers"

AI:
Step 1: (MCP) Connect to Google Drive
Step 2: (MCP) Find and read sales.xlsx
Step 3: (AI) Analyze data
Step 4: Return summary

User sees: Complete summary without manual data copy
```

---

## Part 5: Assisted Coding (Beginner to Advanced)

### 5.1 What is Assisted Coding?

**Assisted Coding** = **AI helps you write, debug, and improve code**

```
Traditional Coding:
You: Write all code yourself
Time: Longer
Quality: Depends on your skill

With Assisted Coding:
You: Describe what you want
AI: Writes code, catches bugs, optimizes
Time: Much faster
Quality: Often better (AI reviews itself)
```

---

### 5.2 Tools for Assisted Coding

| Tool | What It Does |
|---|---|
| **Claude Code (CLI)** | Run AI code in your terminal |
| **GitHub Copilot** | IDE extension (autocomplete) |
| **Claude in IDE** | ChatGPT-like in VS Code |
| **Cursor Editor** | IDE built for AI coding |

---

### 5.3 How Claude Code Works

**Claude Code** = AI agent for your terminal

```bash
# Install
npm install -g @anthropic/claude-code

# Use
claude-code "Create a REST API for a todo app"

# Claude automatically:
# 1. Plans the architecture
# 2. Creates files
# 3. Writes code
# 4. Tests it
# 5. Shows results
```

---

### 5.4 Real-World Coding Workflow

#### Scenario: You need a payment processing system

```
TRADITIONAL WAY:
1. You plan architecture (2 hours)
2. You research Stripe API (1 hour)
3. You write backend code (4 hours)
4. You test and debug (2 hours)
Total: 9 hours

WITH ASSISTED CODING:
1. You describe what you want (10 min)
2. AI writes complete implementation (2 min)
3. You review & make tweaks (30 min)
4. AI handles refactoring (5 min)
Total: 45 minutes
```

---

### 5.5 Coding Prompts: From Good to Excellent

#### Example 1: Basic Task

```
❌ Basic:
"Write a function to validate email"

✅ Better:
"Write a JavaScript function that validates email addresses.
Requirements:
- Accept valid formats like: user@domain.com, user+tag@domain.co.uk
- Reject: invalid formats, no @ sign, no domain
- Use regex or validator library

Export as ES6 module"
```

#### Example 2: Feature Implementation

```
❌ Basic:
"Add login to my app"

✅ Better:
"Add JWT-based authentication to my Node.js/Express app.

Current setup:
- Express server on port 3000
- MongoDB with User model
- Passwords stored as bcrypt hashes

Implement:
1. POST /auth/login - returns JWT token
2. POST /auth/register - creates new user
3. verifyToken middleware - checks JWT validity
4. Protected route example: GET /profile (needs token)

Use: jsonwebtoken library, 1-hour expiration

Also provide: curl commands to test each endpoint"
```

#### Example 3: Debugging

```
❌ Basic:
"Fix this code"

✅ Better:
"My Node.js API is returning 401 Unauthorized for all authenticated requests.

Code:
[paste code]

What I did:
- Sent token in Authorization header: Bearer token_here
- Token was generated from login endpoint
- Middleware checks: jwt.verify(token, SECRET)

What happens:
- Login works, returns token
- Next request with token: 401 error

Debug this and provide:
1. Root cause
2. Fix
3. Test to verify it works"
```

---

### 5.6 Prompting for Code Generation

**Structure for best results:**

```
ROLE: [You are a senior backend engineer]

CONTEXT: [I'm building a fintech app with...]

TASK: [Generate a payment processing module that...]

REQUIREMENTS:
- Must support: Stripe, PayPal, local testing
- Must handle: Retry logic, logging, error recovery
- Must validate: Amount, currency, user permissions

CONSTRAINTS:
- Must work with existing code (show examples)
- No external APIs except Stripe/PayPal
- Must be testable (include test examples)

DELIVERABLES:
- Source code file
- Unit tests
- Example usage
- API documentation
```

---

### 5.7 The Iterative Coding Loop

**Assisted coding works best when you iterate:**

```
Iteration 1: Get Basic Working Version
User: "Create a basic REST API for users"
AI: Writes functional code

Iteration 2: Add Features
User: "Add authentication with JWT"
AI: Integrates auth into existing code

Iteration 3: Optimize
User: "The queries are slow. Add database indexing."
AI: Adds indexes, optimizes queries

Iteration 4: Refactor
User: "Make it follow SOLID principles"
AI: Refactors for maintainability

Iteration 5: Test
User: "Add comprehensive unit tests"
AI: Writes test suite
```

---

### 5.8 Assisted Coding Best Practices

#### 1. Start Simple, Build Up

```
❌ "Build a full e-commerce app"
(Too vague, overwhelming)

✅ "Build the product listing API endpoint"
(Clear, achievable)

Then: "Add the cart API"
Then: "Add checkout API"
```

#### 2. Provide Your Existing Code

```
❌ "Write authentication"
(AI doesn't know your codebase)

✅ "Write authentication for my app.
My current user model looks like:
[show User schema]
My current project structure:
[show folder structure]
I use: Express, MongoDB, Typescript"

(AI adapts to your setup)
```

#### 3. Specify the "Why"

```
❌ "Use Redis"

✅ "Use Redis for session caching because:
- We have 10k concurrent users
- Session lookups are 30% of DB load
- Need <10ms response time"

(AI understands the real need)
```

#### 4. Ask for Explanations

```
❌ Just: "Write the code"

✅ "Write the code AND:
- Explain each function's purpose
- Highlight important design decisions
- Point out potential issues
- Show how to test it"
```

---

### 5.9 Debugging with AI

```
Code has a bug? Smart approach:

Step 1: Show the error
"Getting: Cannot read property 'name' of undefined"

Step 2: Show what you tried
"I tried: adding null checks, changing variable names"

Step 3: Show context
"This happens when user data is incomplete"

AI Response:
- Root cause identified
- Fix provided
- Prevents bug in future
```

---

## Part 6: Prompt Engineering for Coding (Advanced)

### 6.1 Code Review Prompts

```
"Review this code as a senior engineer would.

Code:
[paste code]

Evaluate:
1. Security - Are there vulnerabilities?
2. Performance - Any bottlenecks?
3. Maintainability - Is it easy to understand?
4. Testing - Is it testable?
5. Best practices - Following conventions?

For each issue, provide:
[Issue] → [Why it matters] → [How to fix]"
```

---

### 6.2 Architecture Design Prompts

```
"Design the backend architecture for a real-time chat app.

Requirements:
- 100,000 daily active users
- 500 concurrent users per room
- Instant message delivery
- Message history (last 100 messages per room)
- Read receipts

Constraints:
- Budget: $2,000/month
- Team: 2 engineers
- Timeline: 3 months

Provide:
1. Database schema
2. API design (REST endpoints)
3. Real-time communication approach (WebSocket, etc.)
4. Scaling strategy
5. Deployment architecture"
```

---

### 6.3 Documentation Generation Prompts

```
"Generate comprehensive documentation for this code.

Code:
[paste code]

Include:
1. Overview - What does this module do?
2. Installation - How to set up
3. API Reference - All functions/methods
4. Examples - Real usage
5. Troubleshooting - Common issues
6. Contributing - How to extend

Format as: Markdown with code blocks"
```

---

### 6.4 Testing Prompts

```
"Write comprehensive tests for this code.

Code:
[paste code]

Requirements:
- Unit tests for each function
- Integration tests
- Edge cases
- Error handling

Provide:
- Jest test suite
- >80% code coverage
- Clear test names explaining what's tested
- Sample data/mocks"
```

---

## Part 7: Best Practices & Tips (Advanced)

### 7.1 Claude (AI) Strengths & Limitations

**Claude is Great At:**
- ✅ Writing clean, readable code
- ✅ Explaining complex concepts
- ✅ Debugging and analyzing code
- ✅ Writing documentation
- ✅ Refactoring and optimization
- ✅ Architecture design
- ✅ Best practices

**Claude's Limitations:**
- ❌ Can't run code directly (needs your terminal)
- ❌ May hallucinate (make up functions that don't exist)
- ❌ Doesn't know your codebase unless you show it
- ❌ Can't learn from previous sessions (start fresh each time)

---

### 7.2 Avoiding Common Mistakes

#### Mistake 1: Too Vague Prompts

```
❌ "Write an API"
(What kind? What does it do?)

✅ "Write a REST API for blog posts with:
- GET /posts (list all)
- POST /posts (create)
- GET /posts/:id (get one)
- PUT /posts/:id (update)
- DELETE /posts/:id (delete)
Tech: Node.js/Express, MongoDB"
```

#### Mistake 2: Not Providing Context

```
❌ "Fix this function"
[code with no context]

✅ "Fix this function:
[code]

Context: Used to process payment transactions.
Currently: Returns error when amount has decimals.
Expected: Should handle amounts like $19.99"
```

#### Mistake 3: Not Specifying Constraints

```
❌ "Optimize this"

✅ "Optimize this query:
[query]

Constraint: Must stay under 100ms
Currently: Takes 2 seconds
Data: 10M records in users table"
```

#### Mistake 4: Trusting Without Verification

```
❌ Copy-paste AI code without review

✅ AI writes code → You review it → You test it → You use it

(AI can make mistakes!)
```

---

### 7.3 Iterative Refinement Workflow

```
Round 1: Get Working Version
"Create a user authentication endpoint"
→ AI produces basic version

Round 2: Add Features
"Add email verification during signup"
→ AI extends previous code

Round 3: Security
"Audit for security vulnerabilities"
→ AI reviews and fixes

Round 4: Performance
"Optimize database queries"
→ AI refactors for speed

Round 5: Testing
"Write unit tests"
→ AI adds comprehensive tests

Round 6: Documentation
"Generate API documentation"
→ AI creates docs
```

---

### 7.4 Prompt Engineering Checklist

Before asking Claude to code:

```
☐ Clearly state what you want
☐ Provide context (existing code, setup)
☐ List requirements explicitly
☐ State constraints (budget, time, tech)
☐ Specify output format
☐ Give examples if complex
☐ Ask for explanations, not just code
☐ Mention what you'll use it for
☐ Note any edge cases to handle
☐ Specify testing requirements
```

---

## Part 8: Real-World Examples

### 8.1 Example: Building a Rate Limiter

```
PROMPT:
"I'm building a fintech API and need a distributed rate limiter.

Context:
- Node.js/Express API
- Multiple servers (horizontal scaling)
- Using Redis for caching
- Need to limit: 100 requests per minute per user

Requirements:
1. Prevent brute force attacks
2. Return 429 Too Many Requests when exceeded
3. Include Retry-After header
4. Work across multiple servers

Provide:
- Complete middleware code
- Redis integration
- Example usage in route
- Unit tests"

RESULT: Complete, production-ready rate limiter
```

---

### 8.2 Example: Database Schema Design

```
PROMPT:
"Design a normalized SQL schema for a social media app.

Features:
- Users can follow other users
- Users can create posts
- Posts can have comments
- Users can like posts and comments
- User authentication with passwords

Requirements:
- Normalized database design
- Foreign keys and constraints
- Indexes for common queries
- Handle edge cases (duplicate follows, etc.)

Provide:
- CREATE TABLE statements
- Indexes
- Example queries for common operations"

RESULT: Professional, optimized schema
```

---

### 8.3 Example: API Documentation

```
PROMPT:
"Generate API documentation for my payment processing service.

Endpoints:
- POST /api/payments (create payment)
- GET /api/payments/:id (get payment status)
- POST /api/refunds (refund payment)
- GET /api/webhooks (webhook logs)

Include:
- Request/response examples
- Error codes and meanings
- Authentication requirements
- Rate limits
- Code snippets (JavaScript, Python)"

RESULT: Professional API docs ready to share
```

---

## Part 9: AI Agent Architecture Pattern

### 9.1 Agent Loop Pattern

```
┌─────────────────────────────────────────────────────┐
│                START AGENT LOOP                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  1. RECEIVE USER REQUEST                           │
│     Input: "Build a todo app backend"              │
│                                                     │
│  2. PLAN                                            │
│     "I need to:                                     │
│      - Design database schema                      │
│      - Write API endpoints                         │
│      - Add authentication"                         │
│                                                     │
│  3. EXECUTE STEP 1                                  │
│     Generate database schema                       │
│                                                     │
│  4. REFLECT                                         │
│     "Is this good? Does it match requirements?"    │
│                                                     │
│  5. DECIDE                                          │
│     "Yes, move to Step 2"                          │
│                                                     │
│  6. EXECUTE STEP 2                                  │
│     Write API endpoints                            │
│                                                     │
│  7. REFLECT                                         │
│     "API looks good, but need auth first"          │
│                                                     │
│  8. LOOP BACK                                       │
│     Execute Step 3 (authentication)                │
│                                                     │
│  9. FINAL CHECK                                     │
│     All parts done and integrated?                 │
│     Yes → Continue                                 │
│                                                     │
│  10. DELIVER RESULT                                 │
│      Present complete todo app backend             │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## Part 10: AI Agent vs Tools Comparison

```
┌────────────────────────────────────────────────────────┐
│       WHEN TO USE AI AGENTS vs TOOLS                   │
├────────────────────────────────────────────────────────┤
│                                                        │
│ USE SIMPLE TOOL if:                                   │
│ - Task is straightforward ("Summarize this")          │
│ - Don't need planning ("Get user list")               │
│ - Quick turnaround needed                             │
│ Example: Fetch data, format output                    │
│                                                        │
│ USE AI AGENT if:                                      │
│ - Task is complex ("Build an e-commerce site")        │
│ - Needs planning and steps                            │
│ - Needs to adapt based on feedback                    │
│ - Multiple sub-tasks to coordinate                    │
│ Example: Multi-step code generation, debugging        │
│                                                        │
└────────────────────────────────────────────────────────┘
```
=

## Part 12: Quick Reference

### Prompt Engineering Checklist

```
☐ Be specific (not vague)
☐ Provide context
☐ Give examples
☐ Define the role
☐ Break into steps
☐ Specify format
☐ Set constraints
☐ Ask for reasoning
☐ Request explanations
☐ Include edge cases
```

### Coding Prompt Template

```
ROLE: [Expert in X technology]

CONTEXT: [What I'm building, current setup]

TASK: [What I need]

REQUIREMENTS:
- Requirement 1
- Requirement 2
- Requirement 3

CONSTRAINTS:
- Constraint 1
- Constraint 2

DELIVERABLES:
- Code
- Tests
- Documentation

EXAMPLES:
[Show what you want]
```

### AI Agent Loop

```
1. PERCEIVE (Input)
2. REASON (Plan)
3. ACT (Execute)
4. REFLECT (Evaluate)
5. LOOP (Adjust & repeat)
6. RESPOND (Output)
```


=
