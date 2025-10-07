---
tags: ["testing", "api", "tutorial"]
date: 2025-10-07 00:00:00 +0700
description: When dealing with legacy systems, sometimes the best integration is the one that doesn't require actually integrating at all 😄
published: true
collection: posts
category: blog
id: 2858e9d5-00d5-808b-966d-e97c1c98ff62
title: Creating Mock Servers Using json-server
created_time: 2025-10-07T03:17:00.000Z
cover: 
icon: 
last_edited_time: 2025-10-07T03:24:00.000Z
archived: false
created_by_object: user
created_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
last_edited_by_object: user
last_edited_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
---

![](https://images.unsplash.com/photo-1528869294317-c5cc1ebacaab?ixlib=rb-4.1.0&q=85&fm=jpg&crop=entropy&cs=srgb)

When you inherit a legacy project, the first thing you want to do is get it running locally. Clone the repo, install dependencies, maybe spin up some Docker containers, and you're good to go. But legacy systems have their own ideas about cooperation.

I recently had to work on improving a process that integrated with an old internal system. This wasn't just "a few versions behind" old - we're talking Windows Server 2012, SQL Server 2008, and a an app that I never heard before. The system communicated via HTTP, which should've made things simple. Instead, I spent two days trying to get the environment working on my Mac. Between incompatible SQL Server drivers, Windows-specific dependencies, framework versions that refused to run outside their natural habitat, and my laptop's very limited RAM struggling with the resource-hungry legacy stack, I realized I was fighting a losing battle.

## Why json-server Solves This Problem

**json-server** is a full fake REST API with zero coding required. It's perfect when you need to mock an external service quickly without dealing with complex infrastructure. In my case, instead of wrestling with outdated dependencies and database versions, I could simulate the legacy API's responses and continue development.

The benefits for legacy system integration:

- **No dependencies on the actual service** - develop offline or when the service is down
- **Predictable responses** - test edge cases without manipulating production data
- **Fast iteration** - modify responses instantly without database changes
- **Team collaboration** - share the mock server configuration with your team

## Setting Up json-server

First, install json-server as a development dependency:

```bash
npm install --save-dev json-server
```

Create a `db.json` file with your mock data structure:

```json
{
  "users": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com",
      "status": "active"
    },
    {
      "id": 2,
      "name": "Jane Smith",
      "email": "jane@example.com",
      "status": "inactive"
    }
  ],
  "transactions": [
    {
      "id": 1,
      "userId": 1,
      "amount": 150.00,
      "status": "completed",
      "createdAt": "2024-01-15T10:30:00Z"
    }
  ]
}
```

Add a script to your `package.json`:

```json
{
  "scripts": {
    "mock-server": "json-server --watch db.json --port 3001"
  }
}
```

Now you can run:

```bash
npm run mock-server
```

This gives you a fully functional REST API at `http://localhost:3001` with GET, POST, PUT, PATCH, and DELETE operations on your resources.

## Custom Server for Dynamic Endpoints

The basic setup works great for simple cases, but legacy systems often have quirky endpoints and custom logic. Here's how to create a custom server with dynamic routing:

Create a `server.js` file:

```javascript
// server.js
const jsonServer = require('json-server');
const server = jsonServer.create();
const router = jsonServer.router('db.json');
const middlewares = jsonServer.defaults();

// Add custom middleware
server.use(middlewares);

// Parse JSON bodies
server.use(jsonServer.bodyParser);

// Custom routes before json-server router
server.get('/api/v1/users/:id/full-profile', (req, res) => {
  const userId = parseInt(req.params.id);
  const db = router.db; // Access lowdb instance

  const user = db.get('users')
    .find({ id: userId })
    .value();

  const transactions = db.get('transactions')
    .filter({ userId: userId })
    .value();

  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  res.json({
    user: user,
    transactions: transactions,
    transactionCount: transactions.length,
    totalAmount: transactions.reduce((sum, t) => sum + t.amount, 0)
  });
});

// Simulate legacy system's weird authentication endpoint
server.post('/legacy/auth/validate', (req, res) => {
  const { token } = req.body;

  // Mock validation logic
  if (token && token.startsWith('valid_')) {
    res.json({
      success: true,
      userId: parseInt(token.split('_')[1]) || 1,
      permissions: ['read', 'write']
    });
  } else {
    res.status(401).json({
      success: false,
      error: 'Invalid token'
    });
  }
});

// Add delay to simulate network latency
server.use((req, res, next) => {
  setTimeout(next, 500); // 500ms delay
});

// Custom error responses to match legacy system
server.use((req, res, next) => {
  if (req.method === 'POST') {
    req.body.createdAt = new Date().toISOString();
  }
  next();
});

// Use default router
server.use('/api', router);

// Custom 404 handler
server.use((req, res) => {
  res.status(404).json({
    error: 'Endpoint not found',
    path: req.path,
    timestamp: new Date().toISOString()
  });
});

const PORT = process.env.MOCK_SERVER_PORT || 3001;
server.listen(PORT, () => {
  console.log(`JSON Server is running on port ${PORT}`);
  console.log(`http://localhost:${PORT}`);
});
```

For more complex scenarios with middleware chains:

```javascript
// middleware.js
module.exports = (req, res, next) => {
  // Add custom headers like the legacy system
  res.header('X-Legacy-System', 'v1.0');
  res.header('X-Response-Time', Date.now());

  // Log all requests
  console.log(`${new Date().toISOString()} - ${req.method} ${req.path}`);

  // Validate API key for certain endpoints
  if (req.path.startsWith('/api/secured')) {
    const apiKey = req.headers['x-api-key'];
    if (!apiKey || apiKey !== 'test-key-123') {
      return res.status(403).json({
        error: 'API key required',
        code: 'MISSING_API_KEY'
      });
    }
  }

  next();
};
```

Update your `server.js` to use the middleware:

```javascript
const customMiddleware = require('./middleware');
server.use(customMiddleware);
```

## Running and Testing the Mock Server

Create a `routes.json` file for URL rewriting to match your legacy system's patterns:

```json
{
  "/old-api/get-user/:id": "/users/:id",
  "/legacy/transactions/user/:userId": "/transactions?userId=:userId",
  "/v1/users\\?id=:id": "/users/:id"
}
```

Update your package.json with different configurations:

```json
{
  "scripts": {
    "mock:basic": "json-server --watch db.json --port 3001",
    "mock:custom": "node server.js",
    "mock:routes": "json-server --watch db.json --routes routes.json --port 3001",
    "mock:dev": "nodemon server.js"
  }
}
```

Test your endpoints using curl or httpie:

```bash
# Test basic CRUD
curl http://localhost:3001/api/users

# Test custom endpoint
curl http://localhost:3001/api/v1/users/1/full-profile

# Test with custom headers
curl -H "X-API-Key: test-key-123" \
     http://localhost:3001/api/secured/users

# Test POST with authentication
curl -X POST \
     -H "Content-Type: application/json" \
     -d '{"token":"valid_123"}' \
     http://localhost:3001/legacy/auth/validate
```

## Wrapping Up

json-server transformed what could have been days of environment setup into a 30-minute solution. Instead of fighting with legacy dependencies, you can focus on building your integration. The mock server is also invaluable for:

- **Frontend development** when the backend isn't ready
- **Integration testing** without hitting production APIs
- **Demo environments** that need predictable data
- **Documentation** by showing example responses

You might also want to explore **WireMock** if you need more sophisticated request matching. But for quickly mocking REST APIs, especially legacy systems, json-server hits the sweet spot between simplicity and functionality.

**When dealing with legacy systems, sometimes the best integration is the one that doesn't require actually integrating at all **😄


