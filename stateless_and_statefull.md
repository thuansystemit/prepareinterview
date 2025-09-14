# Stateless vs Stateful Requests

## 1. Stateless Requests

### Definition
A **stateless request** is a request where the server does **not store any information about the client’s previous requests**. Every request is **independent** and contains all the information needed for the server to process it.

### Key Points
- No session or memory of previous requests on the server.
- Each request must include all necessary data (authentication, parameters, etc.).
- Easier to scale horizontally because servers don’t need to share session data.
- Commonly used in REST APIs.

### Example
**HTTP GET for user profile:**
```http
GET /profile
Authorization: Bearer <token>
```
- The server checks the token, retrieves user info, and responds.
- The server doesn’t remember what the user requested before; every request is standalone.

**Benefits:**
- Easy to scale (load balancers can route requests to any server).
- Simpler architecture.
- Reduced memory usage on the server.

**Drawbacks:**

- Clients need to send all required info in each request (like tokens, IDs).
- More repetitive data may increase bandwidth usage.

## 2. Stateful Requests
**Definition:**

A stateful request is a request where the server remembers the client’s previous requests or session state. The server keeps track of context between multiple requests from the same client.

**Key Points:**

- Server maintains session or memory (often via session ID or cookies).
- Requests are not independent; server uses previous data.
- Commonly used in traditional web apps or applications requiring session management.

**Example:**
- Online shopping cart:
    - User adds an item to the cart.
    - Server stores the cart info in a session.
    - Next request (checkout) uses the session data to know what’s in the cart.
- Benefits:
    - Convenient for maintaining context (shopping carts, user sessions, games).
    - Less data needs to be sent by the client repeatedly.
- Drawbacks:
    - Harder to scale (session state must be shared or sticky sessions must be used).
    - Higher memory usage on the server.
    - Server failure can cause session loss unless stored in a persistent/shared store.
  
## 3. Quick Comparison Table

| Feature              | Stateless                        | Stateful                                    |
|-----------------------|----------------------------------|---------------------------------------------|
| Server remembers state | ❌ No                           | ✅ Yes                                      |
| Request dependency    | Independent                     | Dependent on previous requests              |
| Scaling               | Easy (horizontal)               | Harder (sticky sessions / shared session storage) |
| Examples              | REST API, HTTP GET/POST         | Shopping cart, banking session, online games |
