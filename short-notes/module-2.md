# Module 2: Node.js Backend Development & API Design

## Express.js Framework

### Express Fundamentals

**Application Initialization**

Express.js is a minimal and flexible Node.js web application framework that provides a robust set of features for web and mobile applications. It simplifies routing, middleware handling, and HTTP request/response processing, making it the de facto standard for building Node.js APIs. The framework is built on top of Node.js's HTTP module, providing a higher-level abstraction that handles common web server tasks like parsing request bodies, managing cookies, and handling file uploads.

```javascript
const express = require('express');
const app = express();

// Middleware for parsing JSON and URL-encoded data
app.use(express.json()); // Parse JSON request bodies
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded data

// Basic server setup
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

**Routing**

Express routing defines how an application responds to client requests to specific endpoints. Routes are defined using HTTP methods (GET, POST, PUT, PATCH, DELETE) and URL paths, which can include dynamic parameters. Route parameters capture values from the URL, while query strings provide optional data. Express supports both string-based paths and regular expressions for flexible route matching, enabling powerful URL patterns for complex applications.

```javascript
// Basic routes with different HTTP methods
app.get('/', (req, res) => {
  res.send('GET request to homepage');
});

app.post('/users', (req, res) => {
  res.send('POST request to create user');
});

app.put('/users/:id', (req, res) => {
  const userId = req.params.id; // Route parameter
  res.send(`PUT request to update user ${userId}`);
});

app.delete('/users/:id', (req, res) => {
  res.send(`DELETE request to delete user');
});

// Query string parameters
// GET /users?page=2&limit=10
app.get('/users', (req, res) => {
  const page = req.query.page || 1;
  const limit = req.query.limit || 10;
  res.json({ page, limit });
});
```

**Request Object**

The Express request object (`req`) represents the HTTP request and contains properties for the request query string, parameters, body, HTTP headers, and more. `req.body` contains parsed request body data (requires body-parsing middleware), `req.params` contains route parameters, and `req.query` contains URL query string parameters. The request object also provides methods like `req.get()` for retrieving header values and `req.accepts()` for content negotiation, enabling sophisticated request handling logic.

```javascript
app.post('/users', (req, res) => {
  // Access request body (requires express.json() middleware)
  const { name, email } = req.body;
  
  // Access route parameters
  const userId = req.params.id;
  
  // Access query string parameters
  const page = req.query.page;
  
  // Access headers
  const contentType = req.get('Content-Type');
  const authToken = req.get('Authorization');
  
  // Content negotiation
  const acceptsJson = req.accepts('json');
  
  res.json({ name, email, userId, page, contentType, acceptsJson });
});
```

**Response Object**

The Express response object (`res`) represents the HTTP response that an Express app sends when it receives an HTTP request. It provides methods like `res.send()`, `res.json()`, `res.status()`, and `res.redirect()` for crafting responses. The response object supports chaining methods for fluent API usage, allowing you to set status codes, headers, and send data in a single statement. Understanding response methods is crucial for building RESTful APIs with proper HTTP semantics.

```javascript
app.get('/users/:id', (req, res) => {
  const user = { id: req.params.id, name: 'John Doe' };
  
  // Send JSON response
  res.json(user);
  // Equivalent to: res.status(200).json(user);
});

app.post('/users', (req, res) => {
  const newUser = { id: 1, ...req.body };
  
  // Set status code and send response
  res.status(201).json(newUser);
});

app.get('/redirect', (req, res) => {
  // Redirect to another URL
  res.redirect('/users');
});

app.get('/not-found', (req, res) => {
  // Set status and send custom message
  res.status(404).send('Resource not found');
});

// Chaining methods
app.get('/api/data', (req, res) => {
  res
    .status(200)
    .set('X-Custom-Header', 'value')
    .json({ data: 'response' });
});
```

---

### Middleware Architecture

**Middleware Fundamentals**

Middleware functions are functions that have access to the request object (`req`), the response object (`res`), and the next middleware function in the application's request-response cycle. They can execute any code, make changes to the request and response objects, end the request-response cycle, or call the next middleware function. Middleware is executed in the order it's defined, making the order of middleware registration critical for proper application behavior.

```javascript
// Custom middleware function
function loggerMiddleware(req, res, next) {
  console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
  next(); // Pass control to next middleware
}

// Middleware that ends the request-response cycle
function authMiddleware(req, res, next) {
  const token = req.get('Authorization');
  if (!token) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  req.user = { id: 1, name: 'John' }; // Attach user to request
  next(); // Pass control to next middleware
}

// Apply middleware to all routes
app.use(loggerMiddleware);
app.use(authMiddleware);

// Apply middleware to specific routes
app.get('/protected', authMiddleware, (req, res) => {
  res.json({ user: req.user });
});
```

**Application-Level Middleware**

Application-level middleware is bound to an instance of `express()` using `app.use()` or `app.METHOD()`. It runs for every request to the application, making it ideal for cross-cutting concerns like logging, authentication, and error handling. You can mount middleware at specific paths to limit its scope, and multiple middleware functions can be applied in sequence. This level of middleware is perfect for functionality that needs to run before or after all route handlers.

```javascript
// Apply middleware to all routes
app.use(express.json()); // Parse JSON bodies
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded bodies
app.use(express.static('public')); // Serve static files

// Custom logging middleware
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next();
});

// Apply middleware to specific path
app.use('/api', (req, res, next) => {
  req.startTime = Date.now();
  next();
});

// Multiple middleware functions
app.get('/data', 
  (req, res, next) => {
    console.log('First middleware');
    next();
  },
  (req, res, next) => {
    console.log('Second middleware');
    next();
  },
  (req, res) => {
    res.json({ message: 'Response after middleware' });
  }
);
```

**Router-Level Middleware**

Router-level middleware works similarly to application-level middleware but is bound to an instance of `express.Router()`. This allows you to create modular, mountable route handlers that can be composed together. Routers can have their own middleware and can be mounted on specific paths in the main application. This pattern is essential for organizing large applications into logical modules, such as separating user routes, product routes, and admin routes into different router files.

```javascript
// Create a router instance
const userRouter = express.Router();

// Router-level middleware
userRouter.use((req, res, next) => {
  console.log('User router middleware');
  next();
});

// Define routes on the router
userRouter.get('/', (req, res) => {
  res.json({ users: [] });
});

userRouter.get('/:id', (req, res) => {
  res.json({ user: { id: req.params.id } });
});

userRouter.post('/', (req, res) => {
  res.status(201).json({ message: 'User created' });
});

// Mount the router on a specific path
app.use('/users', userRouter);

// Another router for products
const productRouter = express.Router();
productRouter.get('/', (req, res) => {
  res.json({ products: [] });
});
app.use('/products', productRouter);
```

**Error-Handling Middleware**

Error-handling middleware in Express is defined with four arguments instead of three: `(err, req, res, next)`. This middleware is only called when an error is passed to `next(err)` or when an error occurs in the application. Error-handling middleware should be defined after all other middleware and routes to catch any errors that occur during request processing. This centralized approach to error handling ensures consistent error responses across the entire application.

```javascript
// Regular middleware that can throw errors
app.get('/error', (req, res, next) => {
  const error = new Error('Something went wrong');
  error.status = 400;
  next(error); // Pass error to error-handling middleware
});

// Async error wrapper for async route handlers
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// Async route with error handling
app.get('/async-error', asyncHandler(async (req, res) => {
  const data = await someAsyncOperation();
  res.json(data);
}));

// Error-handling middleware (must have 4 arguments)
app.use((err, req, res, next) => {
  console.error(err.stack);
  
  const status = err.status || 500;
  const message = err.message || 'Internal Server Error';
  
  res.status(status).json({
    error: {
      message,
      status,
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    }
  });
});

// 404 handler (must be after all routes)
app.use((req, res) => {
  res.status(404).json({ error: 'Not Found' });
});
```

**Built-in Middleware**

Express provides several built-in middleware functions for common tasks. `express.json()` parses incoming requests with JSON payloads, `express.urlencoded()` parses incoming requests with URL-encoded payloads, and `express.static()` serves static files such as images, CSS, and JavaScript. These middleware functions are essential for handling different types of request data and serving static assets. Understanding when and how to use each built-in middleware is fundamental for building functional Express applications.

```javascript
// Parse JSON request bodies (Content-Type: application/json)
app.use(express.json());

// Parse URL-encoded request bodies (Content-Type: application/x-www-form-urlencoded)
app.use(express.urlencoded({ extended: true }));

// Serve static files from 'public' directory
app.use(express.static('public'));

// Serve static files from multiple directories
app.use('/static', express.static('public'));
app.use('/assets', express.static('assets'));

// Configure JSON parser with options
app.use(express.json({
  limit: '10mb', // Limit request body size
  strict: true, // Only accept objects and arrays
  inflate: true // Inflate deflated bodies
}));

// Configure URL-encoded parser with options
app.use(express.urlencoded({
  extended: true, // Use qs library for parsing
  limit: '10mb',
  parameterLimit: 100
}));
```

**Third-Party Middleware**

The Express ecosystem includes numerous third-party middleware packages that extend functionality. Popular middleware includes `morgan` for HTTP request logging, `cors` for enabling Cross-Origin Resource Sharing, `helmet` for security headers, and `cookie-parser` for parsing cookies. These middleware packages provide production-ready solutions for common web application concerns, allowing developers to focus on business logic rather than infrastructure code.

```javascript
const morgan = require('morgan');
const cors = require('cors');
const helmet = require('helmet');
const cookieParser = require('cookie-parser');

// HTTP request logger
app.use(morgan('combined')); // 'dev', 'combined', 'common', 'short', 'tiny'

// Enable CORS for all routes
app.use(cors());

// Enable CORS with options
app.use(cors({
  origin: 'https://example.com',
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true
}));

// Security headers
app.use(helmet());

// Parse cookies
app.use(cookieParser());

// Rate limiting
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
});
app.use('/api', limiter);
```

**Middleware Execution Order**

Middleware executes in the order it's registered, which is critical for proper application behavior. Request processing flows through middleware from top to bottom, with each middleware either calling `next()` to pass control or sending a response to end the cycle. Error-handling middleware is skipped unless an error is passed to `next(err)`. Understanding this execution order helps prevent issues like authentication running after route handlers or error handlers being unreachable.

```
Request → Middleware 1 → Middleware 2 → Route Handler → Response
            ↓ next()         ↓ next()         ↓ res.send()
           (or res.send)   (or res.send)   (or next(err))

Error Flow:
Route Handler → next(err) → Error Middleware → Error Response
```

```javascript
// Correct order: middleware before routes
app.use(express.json()); // 1. Parse JSON
app.use(loggerMiddleware); // 2. Log requests
app.use(authMiddleware); // 3. Authenticate

app.get('/protected', (req, res) => { // 4. Route handler
  res.json({ data: 'protected' });
});

app.use((err, req, res, next) => { // 5. Error handler
  res.status(500).json({ error: err.message });
});

// Incorrect order: route before middleware
app.get('/data', (req, res) => { // Route runs first
  res.json({ data: req.body }); // req.body is undefined
});
app.use(express.json()); // Middleware never runs for /data
```

---

### REST API Design Principles

**Richardson Maturity Model**

The Richardson Maturity Model defines four levels of REST API maturity, from Level 0 (the Swamp of POX) to Level 3 (Hypermedia Controls). Level 0 uses a single URI and HTTP methods like POST for all operations. Level 1 introduces multiple URIs for resources. Level 2 uses HTTP methods properly (GET, POST, PUT, DELETE) and HTTP status codes. Level 3 adds HATEOAS (Hypermedia as the Engine of Application State), where responses include links to related resources, enabling clients to discover actions dynamically.

| Level | Description | Characteristics |
|-------|-------------|------------------|
| Level 0 | Swamp of POX | Single URI, POST for everything |
| Level 1 | Resources | Multiple URIs for different resources |
| Level 2 | HTTP Verbs | Proper use of GET, POST, PUT, DELETE |
| Level 3 | Hypermedia Controls | HATEOAS, discoverable actions |

```javascript
// Level 0: Single URI, POST for everything
app.post('/api', (req, res) => {
  const { action, resource, data } = req.body;
  if (action === 'getUsers') {
    res.json({ users: [] });
  } else if (action === 'createUser') {
    res.json({ user: data });
  }
});

// Level 1: Multiple URIs for resources
app.get('/users', (req, res) => res.json({ users: [] }));
app.get('/products', (req, res) => res.json({ products: [] }));

// Level 2: Proper HTTP methods and status codes
app.get('/users', (req, res) => res.status(200).json({ users: [] }));
app.get('/users/:id', (req, res) => res.status(200).json({ user: {} }));
app.post('/users', (req, res) => res.status(201).json({ user: {} }));
app.put('/users/:id', (req, res) => res.status(200).json({ user: {} }));
app.delete('/users/:id', (req, res) => res.status(204).send());

// Level 3: HATEOAS with hypermedia controls
app.get('/users/:id', (req, res) => {
  const user = { id: 1, name: 'John Doe' };
  res.status(200).json({
    ...user,
    _links: {
      self: { href: `/users/${user.id}` },
      update: { href: `/users/${user.id}`, method: 'PUT' },
      delete: { href: `/users/${user.id}`, method: 'DELETE' }
    }
  });
});
```

**Resource Naming Conventions**

RESTful resource naming follows specific conventions to create predictable and intuitive APIs. Resources should be nouns (not verbs), pluralized, and use lowercase letters with hyphens for multi-word names. Hierarchical relationships are represented with nested paths. Query parameters should be used for filtering, sorting, and pagination. Following these conventions makes APIs self-documenting and easier for developers to understand and consume.

```javascript
// Good resource naming (nouns, plural, lowercase)
app.get('/users'); // Get all users
app.get('/users/:id'); // Get specific user
app.get('/users/:id/posts'); // Get posts by user
app.get('/posts/:id/comments'); // Get comments on a post

// Bad resource naming (verbs, singular, inconsistent)
app.get('/getUser'); // Verb instead of noun
app.get('/User'); // Singular instead of plural
app.get('/get_all_users'); // Underscores instead of hyphens

// Query parameters for filtering, sorting, pagination
app.get('/users', (req, res) => {
  const { role, status, sort, page, limit } = req.query;
  // /users?role=admin&status=active&sort=name&page=1&limit=10
  res.json({ users: [] });
});

// Sub-resources for relationships
app.get('/users/:userId/orders'); // Orders belonging to a user
app.get('/orders/:orderId/items'); // Items in an order
```

**HTTP Status Codes**

Proper use of HTTP status codes is essential for RESTful API design. Status codes indicate the outcome of an HTTP request and help clients understand how to handle responses. Success codes (2xx) indicate successful requests, client error codes (4xx) indicate client-side errors, and server error codes (5xx) indicate server-side errors. Using the correct status codes makes APIs more predictable and easier to integrate with client applications.

| Code | Category | Description |
|------|----------|-------------|
| 200 | Success | OK - Request succeeded |
| 201 | Success | Created - Resource created successfully |
| 204 | Success | No Content - Request succeeded, no content returned |
| 400 | Client Error | Bad Request - Invalid request data |
| 401 | Client Error | Unauthorized - Authentication required |
| 403 | Client Error | Forbidden - Insufficient permissions |
| 404 | Client Error | Not Found - Resource doesn't exist |
| 409 | Client Error | Conflict - Resource already exists |
| 422 | Client Error | Unprocessable Entity - Validation failed |
| 500 | Server Error | Internal Server Error - Server error |

```javascript
// 200 OK - Successful GET, PUT, PATCH
app.get('/users/:id', (req, res) => {
  const user = { id: req.params.id, name: 'John' };
  res.status(200).json(user);
});

// 201 Created - Successful POST
app.post('/users', (req, res) => {
  const newUser = { id: 1, ...req.body };
  res.status(201).json(newUser);
});

// 204 No Content - Successful DELETE
app.delete('/users/:id', (req, res) => {
  // Delete user logic
  res.status(204).send();
});

// 400 Bad Request - Invalid input
app.post('/users', (req, res) => {
  if (!req.body.name) {
    return res.status(400).json({ error: 'Name is required' });
  }
  res.status(201).json({ user: req.body });
});

// 401 Unauthorized - Missing or invalid authentication
app.get('/protected', (req, res) => {
  const token = req.get('Authorization');
  if (!token) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  res.json({ data: 'protected' });
});

// 403 Forbidden - Valid authentication but insufficient permissions
app.delete('/users/:id', (req, res) => {
  if (!req.user.isAdmin) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  // Delete user
  res.status(204).send();
});

// 404 Not Found - Resource doesn't exist
app.get('/users/:id', (req, res) => {
  const user = findUser(req.params.id);
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  res.json(user);
});

// 409 Conflict - Resource already exists
app.post('/users', (req, res) => {
  const existingUser = findUserByEmail(req.body.email);
  if (existingUser) {
    return res.status(409).json({ error: 'User already exists' });
  }
  const newUser = createUser(req.body);
  res.status(201).json(newUser);
});

// 422 Unprocessable Entity - Validation failed
app.post('/users', (req, res) => {
  const validation = validateUser(req.body);
  if (!validation.valid) {
    return res.status(422).json({ errors: validation.errors });
  }
  const newUser = createUser(req.body);
  res.status(201).json(newUser);
});

// 500 Internal Server Error - Server error
app.get('/users', async (req, res) => {
  try {
    const users = await getAllUsers();
    res.json(users);
  } catch (error) {
    res.status(500).json({ error: 'Internal Server Error' });
  }
});
```

**Idempotency in HTTP Methods**

Idempotency is a property of HTTP methods where making the same request multiple times has the same effect as making it once. GET, HEAD, PUT, and DELETE are idempotent, while POST is not. This distinction is crucial for API design, especially for handling network failures and retries. Understanding idempotency helps clients safely retry operations without causing unintended side effects, making APIs more robust and reliable.

```javascript
// Idempotent methods: GET, PUT, DELETE
// GET - Safe and idempotent (no side effects)
app.get('/users/:id', (req, res) => {
  const user = findUser(req.params.id);
  res.json(user);
  // Calling this multiple times has no side effects
});

// PUT - Idempotent (replacing entire resource)
app.put('/users/:id', (req, res) => {
  const user = updateUser(req.params.id, req.body);
  res.json(user);
  // Calling this multiple times with same data results in same state
});

// DELETE - Idempotent (resource is deleted)
app.delete('/users/:id', (req, res) => {
  deleteUser(req.params.id);
  res.status(204).send();
  // Calling this multiple times: first call deletes, subsequent calls do nothing
});

// Non-idempotent method: POST
// POST - Not idempotent (creates new resource each time)
app.post('/users', (req, res) => {
  const newUser = createUser(req.body);
  res.status(201).json(newUser);
  // Calling this multiple times creates multiple users
});

// PATCH - Not necessarily idempotent (partial updates)
app.patch('/users/:id', (req, res) => {
  const user = partialUpdateUser(req.params.id, req.body);
  res.json(user);
  // Calling this multiple times may have different effects depending on the update
});
```

**Versioning Strategies**

API versioning allows you to evolve your API without breaking existing clients. Common strategies include URL versioning (`/v1/users`), header versioning (`Accept: application/vnd.api.v1+json`), and query parameter versioning (`/users?version=1`). URL versioning is the most straightforward and widely adopted approach. Choosing a versioning strategy early in API development helps manage breaking changes and maintain backward compatibility as your API grows.

```javascript
// URL versioning (most common)
const v1Router = express.Router();
v1Router.get('/users', (req, res) => {
  res.json({ version: 'v1', users: [] });
});

const v2Router = express.Router();
v2Router.get('/users', (req, res) => {
  res.json({ version: 'v2', users: [], metadata: {} });
});

app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);

// Header versioning
app.get('/users', (req, res) => {
  const version = req.get('API-Version') || 'v1';
  if (version === 'v2') {
    return res.json({ version: 'v2', users: [] });
  }
  res.json({ version: 'v1', users: [] });
});

// Query parameter versioning
app.get('/users', (req, res) => {
  const version = req.query.version || 'v1';
  if (version === 'v2') {
    return res.json({ version: 'v2', users: [] });
  }
  res.json({ version: 'v1', users: [] });
});
```

**Pagination Patterns**

Pagination is essential for APIs that return large datasets to prevent overwhelming clients and servers. Common patterns include offset-based pagination (using `page` and `limit` parameters) and cursor-based pagination (using a cursor to mark position). Offset-based pagination is simpler to implement but can have performance issues with large offsets. Cursor-based pagination is more efficient for large datasets and provides consistent results when data changes during pagination.

```javascript
// Offset-based pagination
app.get('/users', (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const offset = (page - 1) * limit;
  
  const users = getUsers({ limit, offset });
  const total = getTotalUsers();
  const totalPages = Math.ceil(total / limit);
  
  res.json({
    data: users,
    pagination: {
      page,
      limit,
      total,
      totalPages,
      hasNext: page < totalPages,
      hasPrev: page > 1
    },
    _links: {
      self: `/users?page=${page}&limit=${limit}`,
      next: page < totalPages ? `/users?page=${page + 1}&limit=${limit}` : null,
      prev: page > 1 ? `/users?page=${page - 1}&limit=${limit}` : null
    }
  });
});

// Cursor-based pagination
app.get('/users', (req, res) => {
  const cursor = req.query.cursor;
  const limit = parseInt(req.query.limit) || 10;
  
  const users = getUsersByCursor({ cursor, limit });
  const nextCursor = users.length > 0 ? users[users.length - 1].id : null;
  
  res.json({
    data: users,
    pagination: {
      cursor,
      nextCursor,
      limit,
      hasMore: users.length === limit
    },
    _links: {
      self: `/users?cursor=${cursor}&limit=${limit}`,
      next: nextCursor ? `/users?cursor=${nextCursor}&limit=${limit}` : null
    }
  });
});
```

---

### Error Handling Patterns

**Centralized Error Handling**

Centralized error handling in Express uses error-handling middleware to catch and process errors consistently across the application. This pattern involves creating a custom error class hierarchy, wrapping async route handlers to catch errors, and using a single error-handling middleware to format and send error responses. Centralized error handling ensures consistent error responses, simplifies error logging, and makes it easier to implement features like error monitoring and alerting.

```javascript
// Custom error class
class AppError extends Error {
  constructor(message, statusCode = 500) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

// Specific error types
class NotFoundError extends AppError {
  constructor(message = 'Resource not found') {
    super(message, 404);
  }
}

class ValidationError extends AppError {
  constructor(message = 'Validation failed', errors = []) {
    super(message, 422);
    this.errors = errors;
  }
}

class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 401);
  }
}

// Async error wrapper
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// Usage in routes
app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await findUser(req.params.id);
  if (!user) {
    throw new NotFoundError('User not found');
  }
  res.json(user);
}));

app.post('/users', asyncHandler(async (req, res) => {
  const validation = validateUser(req.body);
  if (!validation.valid) {
    throw new ValidationError('Validation failed', validation.errors);
  }
  const user = await createUser(req.body);
  res.status(201).json(user);
}));

// Centralized error-handling middleware
app.use((err, req, res, next) => {
  // Log error
  console.error('Error:', err);
  
  // Default to 500 if status code is not set
  const statusCode = err.statusCode || 500;
  const message = err.isOperational ? err.message : 'Internal Server Error';
  
  // Send error response
  res.status(statusCode).json({
    error: {
      message,
      status: statusCode,
      ...(err.errors && { errors: err.errors }),
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    }
  });
});
```

**Async Error Wrapper**

Async route handlers in Express don't automatically catch errors, which can lead to unhandled promise rejections. An async error wrapper is a higher-order function that wraps async route handlers and catches any errors, passing them to Express's error-handling middleware. This pattern eliminates the need for try/catch blocks in every async route handler and ensures consistent error handling across the application.

```javascript
// Async error wrapper implementation
const asyncHandler = (fn) => {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

// Usage without asyncHandler (requires try/catch)
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await findUser(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    next(error);
  }
});

// Usage with asyncHandler (cleaner)
app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await findUser(req.params.id);
  if (!user) {
    throw new NotFoundError('User not found');
  }
  res.json(user);
}));

// Multiple async handlers in sequence
app.get('/users/:id/posts',
  asyncHandler(async (req, res, next) => {
    const user = await findUser(req.params.id);
    if (!user) throw new NotFoundError('User not found');
    req.user = user;
    next();
  }),
  asyncHandler(async (req, res) => {
    const posts = await getPostsByUser(req.user.id);
    res.json(posts);
  })
);
```

**Custom Error Classes**

Custom error classes extend the built-in Error class to provide additional context and structure for different types of errors. Each error type can include specific properties like status codes, error codes, and validation errors. Using custom error classes makes error handling more explicit and allows for granular error responses. This pattern is particularly useful for validation errors, authentication errors, and other domain-specific error scenarios.

```javascript
// Base error class
class AppError extends Error {
  constructor(message, statusCode = 500, code = 'INTERNAL_ERROR') {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

// Validation error
class ValidationError extends AppError {
  constructor(message = 'Validation failed', errors = []) {
    super(message, 422, 'VALIDATION_ERROR');
    this.errors = errors;
  }
}

// Not found error
class NotFoundError extends AppError {
  constructor(message = 'Resource not found') {
    super(message, 404, 'NOT_FOUND');
  }
}

// Unauthorized error
class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 401, 'UNAUTHORIZED');
  }
}

// Forbidden error
class ForbiddenError extends AppError {
  constructor(message = 'Forbidden') {
    super(message, 403, 'FORBIDDEN');
  }
}

// Conflict error
class ConflictError extends AppError {
  constructor(message = 'Resource already exists') {
    super(message, 409, 'CONFLICT');
  }
}

// Usage in route handlers
app.post('/users', asyncHandler(async (req, res) => {
  const { email, password } = req.body;
  
  // Validate input
  if (!email || !password) {
    throw new ValidationError('Email and password are required', [
      { field: 'email', message: 'Email is required' },
      { field: 'password', message: 'Password is required' }
    ]);
  }
  
  // Check if user exists
  const existingUser = await findUserByEmail(email);
  if (existingUser) {
    throw new ConflictError('User with this email already exists');
  }
  
  // Create user
  const user = await createUser({ email, password });
  res.status(201).json(user);
}));
```

**Error Logging and Monitoring**

Error logging and monitoring are critical for maintaining production applications. Logging errors with context (request details, user information, timestamps) helps diagnose issues quickly. Monitoring tools like Sentry, LogRocket, or custom solutions can alert teams to errors in real-time. Structured logging with consistent formats makes it easier to search and analyze error patterns, enabling proactive bug fixing and performance optimization.

```javascript
const winston = require('winston');

// Configure logger
const logger = winston.createLogger({
  level: 'error',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.Console({ format: winston.format.simple() })
  ]
});

// Error-handling middleware with logging
app.use((err, req, res, next) => {
  // Log error with context
  logger.error({
    message: err.message,
    stack: err.stack,
    statusCode: err.statusCode,
    code: err.code,
    method: req.method,
    url: req.url,
    ip: req.ip,
    userAgent: req.get('user-agent'),
    userId: req.user?.id,
    timestamp: new Date().toISOString()
  });
  
  // Send error response
  const statusCode = err.statusCode || 500;
  const message = err.isOperational ? err.message : 'Internal Server Error';
  
  res.status(statusCode).json({
    error: {
      message,
      status: statusCode,
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    }
  });
});

// Integration with monitoring service (e.g., Sentry)
const Sentry = require('@sentry/node');

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV
});

app.use(Sentry.Handlers.errorHandler());

// Error-handling middleware with Sentry
app.use((err, req, res, next) => {
  Sentry.captureException(err);
  // ... rest of error handling
});
```

**Graceful Degradation**

Graceful degradation ensures that your application continues to function, albeit with reduced functionality, when certain services or features fail. This pattern involves implementing fallback mechanisms, caching strategies, and default values to maintain core functionality during outages. Graceful degradation improves user experience and system resilience by preventing complete failures and providing meaningful feedback when services are unavailable.

```javascript
// Graceful degradation with fallback data
app.get('/users/:id', asyncHandler(async (req, res) => {
  try {
    const user = await findUser(req.params.id);
    if (!user) {
      throw new NotFoundError('User not found');
    }
    res.json(user);
  } catch (error) {
    // Fallback to cached data if database is unavailable
    if (error.code === 'ECONNREFUSED') {
      const cachedUser = await cache.get(`user:${req.params.id}`);
      if (cachedUser) {
        res.set('X-Cache', 'HIT');
        return res.json(cachedUser);
      }
    }
    throw error;
  }
}));

// Graceful degradation with default values
app.get('/config', asyncHandler(async (req, res) => {
  try {
    const config = await getConfig();
    res.json(config);
  } catch (error) {
    // Return default configuration if config service is unavailable
    res.json({
      features: {
        darkMode: false,
        notifications: true
      },
      _warning: 'Using default configuration'
    });
  }
}));

// Graceful degradation with circuit breaker
const CircuitBreaker = require('opossum');

const dbBreaker = new CircuitBreaker(findUser, {
  timeout: 3000,
  errorThresholdPercentage: 50,
  resetTimeout: 30000
});

dbBreaker.on('open', () => {
  console.log('Database circuit breaker opened');
});

dbBreaker.on('halfOpen', () => {
  console.log('Database circuit breaker half-open');
});

app.get('/users/:id', asyncHandler(async (req, res) => {
  try {
    const user = await dbBreaker.fire(req.params.id);
    res.json(user);
  } catch (error) {
    if (dbBreaker.opened) {
      // Circuit is open, use fallback
      const cachedUser = await cache.get(`user:${req.params.id}`);
      if (cachedUser) {
        return res.json(cachedUser);
      }
    }
    throw error;
  }
}));
```

---

## Backend Architecture Patterns

### Layered Architecture

**Controller Layer**

The controller layer handles HTTP requests and responses, acting as the entry point for incoming requests. Controllers are responsible for parsing request data, validating inputs, calling service layer methods, and formatting responses. They should be thin and focused solely on request/response handling, delegating business logic to the service layer. This separation of concerns makes the codebase more maintainable and testable, as controllers can be tested independently of business logic.

```javascript
// controllers/userController.js
const userService = require('../services/userService');

class UserController {
  async getUsers(req, res, next) {
    try {
      const { page, limit, role } = req.query;
      const users = await userService.getUsers({ page, limit, role });
      res.json(users);
    } catch (error) {
      next(error);
    }
  }

  async getUserById(req, res, next) {
    try {
      const user = await userService.getUserById(req.params.id);
      res.json(user);
    } catch (error) {
      next(error);
    }
  }

  async createUser(req, res, next) {
    try {
      const user = await userService.createUser(req.body);
      res.status(201).json(user);
    } catch (error) {
      next(error);
    }
  }

  async updateUser(req, res, next) {
    try {
      const user = await userService.updateUser(req.params.id, req.body);
      res.json(user);
    } catch (error) {
      next(error);
    }
  }

  async deleteUser(req, res, next) {
    try {
      await userService.deleteUser(req.params.id);
      res.status(204).send();
    } catch (error) {
      next(error);
    }
  }
}

module.exports = new UserController();
```

**Service Layer**

The service layer contains business logic and acts as an intermediary between controllers and repositories. Services orchestrate operations, implement business rules, and coordinate multiple repository calls. They should be independent of HTTP concerns and can be reused across different controllers. This layer is crucial for maintaining clean separation of concerns, as it isolates business logic from both the presentation (controllers) and data access (repositories) layers.

```javascript
// services/userService.js
const userRepository = require('../repositories/userRepository');
const { ValidationError, NotFoundError } = require('../utils/errors');

class UserService {
  async getUsers(filters = {}) {
    const { page = 1, limit = 10, role } = filters;
    const offset = (page - 1) * limit;
    const users = await userRepository.findAll({ limit, offset, role });
    const total = await userRepository.count({ role });
    
    return {
      data: users,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit)
      }
    };
  }

  async getUserById(id) {
    const user = await userRepository.findById(id);
    if (!user) {
      throw new NotFoundError('User not found');
    }
    return user;
  }

  async createUser(userData) {
    // Business logic: validate email uniqueness
    const existingUser = await userRepository.findByEmail(userData.email);
    if (existingUser) {
      throw new ValidationError('Email already exists', [
        { field: 'email', message: 'Email already exists' }
      ]);
    }
    
    // Business logic: hash password
    const hashedPassword = await hashPassword(userData.password);
    const user = await userRepository.create({
      ...userData,
      password: hashedPassword
    });
    
    return user;
  }

  async updateUser(id, updates) {
    const user = await this.getUserById(id);
    
    // Business logic: prevent email change to existing email
    if (updates.email && updates.email !== user.email) {
      const existingUser = await userRepository.findByEmail(updates.email);
      if (existingUser) {
        throw new ValidationError('Email already exists', [
          { field: 'email', message: 'Email already exists' }
        ]);
      }
    }
    
    const updatedUser = await userRepository.update(id, updates);
    return updatedUser;
  }

  async deleteUser(id) {
    await this.getUserById(id);
    await userRepository.delete(id);
  }
}

module.exports = new UserService();
```

**Repository/DAO Layer**

The repository (or Data Access Object) layer handles all database operations, providing an abstraction over the data store. Repositories encapsulate SQL queries, database connections, and data mapping logic, making the rest of the application independent of the database implementation. This layer should only contain data access logic, with no business rules or HTTP concerns. Repositories make it easy to switch databases or mock data for testing, as they provide a consistent interface for data operations.

```javascript
// repositories/userRepository.js
const db = require('../config/database');

class UserRepository {
  async findAll({ limit = 10, offset = 0, role } = {}) {
    let query = 'SELECT * FROM users';
    const params = [];
    
    if (role) {
      query += ' WHERE role = ?';
      params.push(role);
    }
    
    query += ' LIMIT ? OFFSET ?';
    params.push(limit, offset);
    
    const [rows] = await db.execute(query, params);
    return rows;
  }

  async findById(id) {
    const [rows] = await db.execute(
      'SELECT * FROM users WHERE id = ?',
      [id]
    );
    return rows[0] || null;
  }

  async findByEmail(email) {
    const [rows] = await db.execute(
      'SELECT * FROM users WHERE email = ?',
      [email]
    );
    return rows[0] || null;
  }

  async count({ role } = {}) {
    let query = 'SELECT COUNT(*) as count FROM users';
    const params = [];
    
    if (role) {
      query += ' WHERE role = ?';
      params.push(role);
    }
    
    const [rows] = await db.execute(query, params);
    return rows[0].count;
  }

  async create(userData) {
    const [result] = await db.execute(
      'INSERT INTO users (name, email, password, role) VALUES (?, ?, ?, ?)',
      [userData.name, userData.email, userData.password, userData.role || 'user']
    );
    return this.findById(result.insertId);
  }

  async update(id, updates) {
    const fields = [];
    const values = [];
    
    Object.keys(updates).forEach(key => {
      if (updates[key] !== undefined) {
        fields.push(`${key} = ?`);
        values.push(updates[key]);
      }
    });
    
    if (fields.length === 0) {
      return this.findById(id);
    }
    
    values.push(id);
    await db.execute(
      `UPDATE users SET ${fields.join(', ')} WHERE id = ?`,
      values
    );
    
    return this.findById(id);
  }

  async delete(id) {
    await db.execute('DELETE FROM users WHERE id = ?', [id]);
  }
}

module.exports = new UserRepository();
```

**Separation of Concerns**

Separation of concerns is a design principle that divides a computer program into distinct sections, each addressing a separate concern. In layered architecture, this means controllers handle HTTP, services handle business logic, and repositories handle data access. This separation makes the codebase more maintainable, testable, and scalable, as changes to one layer don't affect others. It also enables parallel development, as different developers can work on different layers independently.

```
┌─────────────────────────────────────────┐
│           HTTP Request                 │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│      Controller Layer                   │
│  - Parse request data                   │
│  - Validate inputs                      │
│  - Call service methods                 │
│  - Format responses                    │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│       Service Layer                     │
│  - Business logic                       │
│  - Orchestrate operations               │
│  - Implement rules                      │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│    Repository/DAO Layer                 │
│  - Database operations                  │
│  - Data mapping                         │
│  - Query execution                      │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│         Database                        │
└─────────────────────────────────────────┘
```

---

### Dependency Injection

**Manual Dependency Injection**

Dependency injection (DI) is a technique where a class receives its dependencies from external sources rather than creating them internally. In Node.js, manual DI involves passing dependencies as constructor parameters or function arguments. This approach makes code more testable, as dependencies can be easily mocked, and more flexible, as implementations can be swapped without modifying the dependent code. Manual DI is simple to implement and doesn't require additional libraries.

```javascript
// Without dependency injection (tightly coupled)
class UserService {
  constructor() {
    this.userRepository = new UserRepository(); // Hard dependency
  }
  
  async getUser(id) {
    return this.userRepository.findById(id);
  }
}

// With manual dependency injection (loosely coupled)
class UserService {
  constructor(userRepository) {
    this.userRepository = userRepository; // Injected dependency
  }
  
  async getUser(id) {
    return this.userRepository.findById(id);
  }
}

// Usage
const userRepository = new UserRepository();
const userService = new UserService(userRepository);

// Testing with mock dependency
const mockUserRepository = {
  findById: (id) => Promise.resolve({ id, name: 'Test User' })
};
const testUserService = new UserService(mockUserRepository);
```

**Inversion of Control (IoC) Principles**

Inversion of Control is a design principle where the flow of control is inverted compared to traditional procedural programming. Instead of the application controlling the creation and lifecycle of objects, a framework or container manages these responsibilities. IoC enables loose coupling, as components don't need to know how their dependencies are created or managed. This principle is the foundation for dependency injection containers and frameworks that automate dependency management.

```javascript
// Traditional control (application creates dependencies)
class UserController {
  constructor() {
    this.userService = new UserService(); // Creates dependency
    this.userRepository = new UserRepository(); // Creates dependency
  }
}

// Inversion of Control (container manages dependencies)
class ServiceContainer {
  constructor() {
    this.services = new Map();
    this.singletons = new Map();
  }
  
  register(name, factory, isSingleton = true) {
    this.services.set(name, { factory, isSingleton });
  }
  
  get(name) {
    const service = this.services.get(name);
    if (!service) {
      throw new Error(`Service ${name} not registered`);
    }
    
    if (service.isSingleton) {
      if (!this.singletons.has(name)) {
        this.singletons.set(name, service.factory(this));
      }
      return this.singletons.get(name);
    }
    
    return service.factory(this);
  }
}

// Register services
const container = new ServiceContainer();

container.register('userRepository', (c) => new UserRepository());
container.register('userService', (c) => new UserService(c.get('userRepository')));
container.register('userController', (c) => new UserController(c.get('userService')));

// Use services
const userController = container.get('userController');
```

**Service Container Pattern**

A service container (also called a dependency injection container) is a class that manages the creation and lifecycle of services and their dependencies. The container resolves dependencies automatically, creating instances as needed and managing singletons. This pattern reduces boilerplate code for dependency injection and provides a centralized place to configure and manage all application services. Service containers are particularly useful in large applications with many interdependent components.

```javascript
// services/container.js
class ServiceContainer {
  constructor() {
    this.services = new Map();
    this.instances = new Map();
  }
  
  register(name, factory, options = {}) {
    this.services.set(name, {
      factory,
      singleton: options.singleton !== false,
      lazy: options.lazy !== false
    });
  }
  
  get(name) {
    if (this.instances.has(name)) {
      return this.instances.get(name);
    }
    
    const service = this.services.get(name);
    if (!service) {
      throw new Error(`Service '${name}' not registered`);
    }
    
    const instance = service.factory(this);
    
    if (service.singleton) {
      this.instances.set(name, instance);
    }
    
    return instance;
  }
  
  has(name) {
    return this.services.has(name);
  }
}

module.exports = new ServiceContainer();

// services/index.js (configure container)
const container = require('./container');
const UserRepository = require('../repositories/userRepository');
const UserService = require('../services/userService');
const UserController = require('../controllers/userController');

// Register services
container.register('userRepository', () => new UserRepository());
container.register('userService', (c) => new UserService(c.get('userRepository')));
container.register('userController', (c) => new UserController(c.get('userService')));

module.exports = container;

// Usage in routes
const container = require('../services');
const userController = container.get('userController');

app.get('/users', (req, res, next) => userController.getUsers(req, res, next));
```

**Testing Benefits of DI**

Dependency injection significantly improves testability by allowing dependencies to be easily replaced with mocks or stubs. Tests can isolate individual components without needing to set up real databases, external APIs, or other heavyweight dependencies. This leads to faster, more reliable tests that are easier to maintain. DI also enables integration tests to use test doubles for external services, ensuring tests are deterministic and don't have side effects.

```javascript
// Testing with dependency injection
const assert = require('assert');
const UserService = require('../../services/userService');

describe('UserService', () => {
  let userService;
  let mockUserRepository;
  
  beforeEach(() => {
    // Create mock repository
    mockUserRepository = {
      findById: async (id) => ({ id, name: 'Test User' }),
      findByEmail: async () => null,
      create: async (data) => ({ id: 1, ...data })
    };
    
    // Inject mock repository
    userService = new UserService(mockUserRepository);
  });
  
  describe('getUserById', () => {
    it('should return user when found', async () => {
      const user = await userService.getUserById(1);
      assert.strictEqual(user.id, 1);
      assert.strictEqual(user.name, 'Test User');
    });
    
    it('should throw NotFoundError when user not found', async () => {
      mockUserRepository.findById = async () => null;
      
      try {
        await userService.getUserById(999);
        assert.fail('Should have thrown error');
      } catch (error) {
        assert.strictEqual(error.message, 'User not found');
        assert.strictEqual(error.statusCode, 404);
      }
    });
  });
  
  describe('createUser', () => {
    it('should create user with hashed password', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'password123'
      };
      
      const user = await userService.createUser(userData);
      
      assert.strictEqual(user.name, 'John Doe');
      assert.strictEqual(user.email, 'john@example.com');
      assert.notStrictEqual(user.password, 'password123'); // Should be hashed
    });
  });
});
```

---

### Configuration Management

**Environment-Based Configuration**

Environment-based configuration allows applications to behave differently in different environments (development, staging, production) without code changes. Configuration values are loaded from environment variables or environment-specific configuration files. This approach separates configuration from code, making it easier to deploy across different environments and keeping sensitive data out of version control. Environment variables are the standard way to manage configuration in Node.js applications.

```javascript
// config/index.js
require('dotenv').config();

const config = {
  env: process.env.NODE_ENV || 'development',
  port: parseInt(process.env.PORT) || 3000,
  database: {
    host: process.env.DB_HOST || 'localhost',
    port: parseInt(process.env.DB_PORT) || 3306,
    user: process.env.DB_USER || 'root',
    password: process.env.DB_PASSWORD || '',
    name: process.env.DB_NAME || 'myapp'
  },
  jwt: {
    secret: process.env.JWT_SECRET || 'secret',
    expiresIn: process.env.JWT_EXPIRES_IN || '1d'
  },
  redis: {
    host: process.env.REDIS_HOST || 'localhost',
    port: parseInt(process.env.REDIS_PORT) || 6379
  }
};

// Environment-specific overrides
if (config.env === 'test') {
  config.database.name = 'myapp_test';
}

module.exports = config;

// .env file (not committed to version control)
NODE_ENV=development
PORT=3000
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=password
DB_NAME=myapp
JWT_SECRET=your-secret-key
JWT_EXPIRES_IN=1d
REDIS_HOST=localhost
REDIS_PORT=6379
```

**dotenv Library Usage**

The `dotenv` library loads environment variables from a `.env` file into `process.env`, making configuration management straightforward. It reads key-value pairs from the `.env` file and adds them to the environment, accessible via `process.env`. The library should be loaded as early as possible in the application, typically at the entry point. Using `.env` files keeps sensitive configuration out of version control while providing a simple way to manage environment-specific settings.

```javascript
// Install dotenv
// npm install dotenv

// Load environment variables (at the top of your entry file)
require('dotenv').config();

// Access environment variables
const dbConfig = {
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME
};

// Use in application
const express = require('express');
const app = express();

app.listen(process.env.PORT || 3000, () => {
  console.log(`Server running on port ${process.env.PORT || 3000}`);
  console.log(`Environment: ${process.env.NODE_ENV || 'development'}`);
});

// .env.example (committed to version control as template)
NODE_ENV=development
PORT=3000
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=your_password
DB_NAME=myapp
JWT_SECRET=your_jwt_secret
JWT_EXPIRES_IN=1d
```

**Configuration Validation**

Configuration validation ensures that all required environment variables are present and have valid values before the application starts. This prevents runtime errors caused by missing or invalid configuration. Validation can be performed using libraries like `joi`, `zod`, or custom validation logic. Validating configuration at startup provides early feedback to developers and operators, reducing the risk of deployment failures.

```javascript
// Using Zod for configuration validation
const { z } = require('zod');

// Define configuration schema
const configSchema = z.object({
  NODE_ENV: z.enum(['development', 'staging', 'production']).default('development'),
  PORT: z.string().regex(/^\d+$/).transform(Number).default('3000'),
  DB_HOST: z.string().min(1),
  DB_PORT: z.string().regex(/^\d+$/).transform(Number).default('3306'),
  DB_USER: z.string().min(1),
  DB_PASSWORD: z.string().min(1),
  DB_NAME: z.string().min(1),
  JWT_SECRET: z.string().min(32),
  JWT_EXPIRES_IN: z.string().default('1d'),
  REDIS_HOST: z.string().default('localhost'),
  REDIS_PORT: z.string().regex(/^\d+$/).transform(Number).default('6379')
});

// Validate and parse configuration
function loadConfig() {
  try {
    const config = configSchema.parse(process.env);
    console.log('Configuration loaded successfully');
    return config;
  } catch (error) {
    console.error('Configuration validation failed:');
    error.errors.forEach(err => {
      console.error(`  ${err.path.join('.')}: ${err.message}`);
    });
    process.exit(1);
  }
}

const config = loadConfig();
module.exports = config;

// Custom validation without external libraries
function validateConfig() {
  const required = [
    'DB_HOST',
    'DB_USER',
    'DB_PASSWORD',
    'DB_NAME',
    'JWT_SECRET'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    console.error('Missing required environment variables:');
    missing.forEach(key => console.error(`  - ${key}`));
    process.exit(1);
  }
  
  if (process.env.JWT_SECRET && process.env.JWT_SECRET.length < 32) {
    console.error('JWT_SECRET must be at least 32 characters');
    process.exit(1);
  }
}
```

**Secrets Management**

Secrets management involves securely storing and accessing sensitive information like API keys, database passwords, and JWT secrets. Best practices include never committing secrets to version control, using environment variables for secrets, and employing secrets management services like AWS Secrets Manager, HashiCorp Vault, or Azure Key Vault for production environments. Secrets should be encrypted at rest and rotated regularly to minimize the impact of potential breaches.

```javascript
// Using environment variables for secrets (basic approach)
const secrets = {
  jwtSecret: process.env.JWT_SECRET,
  dbPassword: process.env.DB_PASSWORD,
  apiKey: process.env.API_KEY
};

// Validate secrets are present
Object.entries(secrets).forEach(([key, value]) => {
  if (!value) {
    throw new Error(`Missing secret: ${key}`);
  }
});

// Using AWS Secrets Manager (production approach)
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager();

async function getSecret(secretName) {
  const data = await secretsManager.getSecretValue({ SecretId: secretName }).promise();
  if (data.SecretString) {
    return JSON.parse(data.SecretString);
  }
  return Buffer.from(data.SecretBinary, 'base64').toString('ascii');
}

async function loadSecrets() {
  try {
    const dbSecret = await getSecret('prod/db-credentials');
    const jwtSecret = await getSecret('prod/jwt-secret');
    
    return {
      db: {
        host: dbSecret.host,
        user: dbSecret.username,
        password: dbSecret.password
      },
      jwt: {
        secret: jwtSecret.secret
      }
    };
  } catch (error) {
    console.error('Failed to load secrets:', error);
    process.exit(1);
  }
}

// Secrets rotation (example)
async function rotateSecret(secretName) {
  const newSecret = generateRandomSecret();
  
  await secretsManager.putSecretValue({
    SecretId: secretName,
    SecretString: JSON.stringify({ secret: newSecret })
  }).promise();
  
  console.log(`Secret ${secretName} rotated successfully`);
}
```

---

### State Management

**Stateless vs Stateful Services**

Stateless services don't store any client context between requests, making them easier to scale and more resilient to failures. Each request contains all necessary information, and any server can handle any request. Stateful services maintain client context across requests, which can improve performance but complicates scaling and fault tolerance. REST APIs are typically designed to be stateless, with state managed by the client (e.g., via JWTs) or external services (e.g., Redis).

```javascript
// Stateless service (preferred for REST APIs)
app.get('/users/:id', async (req, res) => {
  const user = await userRepository.findById(req.params.id);
  res.json(user);
});

// Stateful service (maintains session state)
const sessions = new Map();

app.post('/login', (req, res) => {
  const { username, password } = req.body;
  const user = authenticate(username, password);
  
  const sessionId = generateSessionId();
  sessions.set(sessionId, { userId: user.id, createdAt: Date.now() });
  
  res.json({ sessionId });
});

app.get('/profile', (req, res) => {
  const sessionId = req.get('X-Session-ID');
  const session = sessions.get(sessionId);
  
  if (!session) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  const user = userRepository.findById(session.userId);
  res.json(user);
});
```

**Session Management Strategies**

Session management tracks user state across multiple requests, enabling features like authentication, shopping carts, and user preferences. Strategies include server-side sessions (stored in memory, database, or Redis), client-side sessions (JWTs), and hybrid approaches. Server-side sessions provide better security and can be invalidated, but require server storage. Client-side sessions reduce server storage requirements but are harder to invalidate and require careful security measures.

```javascript
// Server-side session with express-session
const session = require('express-session');
const RedisStore = require('connect-redis').default;
const redis = require('redis');

const redisClient = redis.createClient({
  host: process.env.REDIS_HOST,
  port: process.env.REDIS_PORT
});

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  }
}));

app.post('/login', (req, res) => {
  const { username, password } = req.body;
  const user = authenticate(username, password);
  
  req.session.userId = user.id;
  res.json({ message: 'Logged in' });
});

app.get('/profile', (req, res) => {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  const user = userRepository.findById(req.session.userId);
  res.json(user);
});

app.post('/logout', (req, res) => {
  req.session.destroy();
  res.json({ message: 'Logged out' });
});

// Client-side session with JWT
const jwt = require('jsonwebtoken');

app.post('/login', (req, res) => {
  const { username, password } = req.body;
  const user = authenticate(username, password);
  
  const token = jwt.sign(
    { userId: user.id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: '1d' }
  );
  
  res.json({ token });
});

app.get('/profile', (req, res) => {
  const token = req.get('Authorization')?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    const user = userRepository.findById(decoded.userId);
    res.json(user);
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
});
```

**Caching Strategies**

Caching improves application performance by storing frequently accessed data in fast storage like memory or Redis. Common patterns include cache-aside (lazy loading), write-through, and write-back caching. Cache-aside is the most common pattern for web applications, where the application checks the cache first and only queries the database on cache misses. Proper cache invalidation is crucial to prevent serving stale data. Redis is a popular choice for distributed caching in Node.js applications.

```javascript
// Cache-aside pattern with Redis
const redis = require('redis');
const client = redis.createClient();

async function getUserWithCache(userId) {
  const cacheKey = `user:${userId}`;
  
  // Try to get from cache
  const cachedUser = await client.get(cacheKey);
  if (cachedUser) {
    console.log('Cache hit');
    return JSON.parse(cachedUser);
  }
  
  // Cache miss - get from database
  console.log('Cache miss');
  const user = await userRepository.findById(userId);
  
  // Store in cache for 1 hour
  await client.setex(cacheKey, 3600, JSON.stringify(user));
  
  return user;
}

// Write-through cache (update cache when updating database)
async function updateUserWithCache(userId, updates) {
  const user = await userRepository.update(userId, updates);
  
  // Update cache
  const cacheKey = `user:${userId}`;
  await client.setex(cacheKey, 3600, JSON.stringify(user));
  
  return user;
}

// Cache invalidation
async function deleteUserWithCache(userId) {
  await userRepository.delete(userId);
  
  // Invalidate cache
  const cacheKey = `user:${userId}`;
  await client.del(cacheKey);
}

// Using caching in Express routes
app.get('/users/:id', async (req, res) => {
  const user = await getUserWithCache(req.params.id);
  res.json(user);
});

app.put('/users/:id', async (req, res) => {
  const user = await updateUserWithCache(req.params.id, req.body);
  res.json(user);
});

app.delete('/users/:id', async (req, res) => {
  await deleteUserWithCache(req.params.id);
  res.status(204).send();
});
```

---

## Hands-on Projects: Node.js

### Build Simple CLI Task Manager

**File I/O Operations**

The CLI task manager uses Node.js's file system module to persist tasks in a JSON file. File I/O operations include reading the existing tasks file, parsing JSON, modifying the task list, and writing back to the file. Asynchronous file operations are preferred to avoid blocking the event loop. Understanding file I/O is essential for building CLI tools that need to persist data between runs.

```javascript
const fs = require('fs').promises;
const path = require('path');

const TASKS_FILE = path.join(__dirname, 'tasks.json');

// Read tasks from file
async function readTasks() {
  try {
    const data = await fs.readFile(TASKS_FILE, 'utf8');
    return JSON.parse(data);
  } catch (error) {
    if (error.code === 'ENOENT') {
      return []; // File doesn't exist, return empty array
    }
    throw error;
  }
}

// Write tasks to file
async function writeTasks(tasks) {
  await fs.writeFile(TASKS_FILE, JSON.stringify(tasks, null, 2));
}

// Ensure tasks file exists
async function initTasksFile() {
  try {
    await fs.access(TASKS_FILE);
  } catch {
    await writeTasks([]);
  }
}
```

**Command Parsing**

Command parsing involves interpreting command-line arguments to determine which operation to perform. The task manager supports commands like `add`, `list`, `complete`, and `delete`, each with their own arguments. Using `process.argv` provides access to command-line arguments, which can be parsed to extract commands and parameters. Proper command parsing makes CLI tools intuitive and user-friendly.

```javascript
// Parse command-line arguments
function parseCommand(args) {
  const [command, ...params] = args.slice(2); // Skip node and script path
  
  return {
    command,
    params
  };
}

// Usage
const { command, params } = parseCommand(process.argv);

// Commands:
// node task-manager.js add "Buy groceries"
// node task-manager.js list
// node task-manager.js complete 1
// node task-manager.js delete 1
```

**Task CRUD Operations**

The task manager implements full CRUD (Create, Read, Update, Delete) operations for tasks. Create adds a new task, Read lists all tasks, Update marks tasks as complete, and Delete removes tasks. Each operation reads the current tasks from the file, performs the modification, and writes the updated list back. This pattern is fundamental for building data-driven applications and understanding how to manage persistent state.

```javascript
// Create task
async function addTask(description) {
  const tasks = await readTasks();
  const newTask = {
    id: Date.now(),
    description,
    completed: false,
    createdAt: new Date().toISOString()
  };
  tasks.push(newTask);
  await writeTasks(tasks);
  return newTask;
}

// Read all tasks
async function listTasks() {
  const tasks = await readTasks();
  return tasks;
}

// Update task (mark as complete)
async function completeTask(taskId) {
  const tasks = await readTasks();
  const task = tasks.find(t => t.id === taskId);
  
  if (!task) {
    throw new Error('Task not found');
  }
  
  task.completed = true;
  task.completedAt = new Date().toISOString();
  await writeTasks(tasks);
  return task;
}

// Delete task
async function deleteTask(taskId) {
  const tasks = await readTasks();
  const index = tasks.findIndex(t => t.id === taskId);
  
  if (index === -1) {
    throw new Error('Task not found');
  }
  
  const deletedTask = tasks.splice(index, 1)[0];
  await writeTasks(tasks);
  return deletedTask;
}

// Main CLI logic
async function main() {
  await initTasksFile();
  
  const { command, params } = parseCommand(process.argv);
  
  try {
    switch (command) {
      case 'add':
        const newTask = await addTask(params[0]);
        console.log(`✓ Added task: ${newTask.description}`);
        break;
        
      case 'list':
        const tasks = await listTasks();
        console.log('Tasks:');
        tasks.forEach(task => {
          const status = task.completed ? '✓' : '○';
          console.log(`  ${status} ${task.id}: ${task.description}`);
        });
        break;
        
      case 'complete':
        const completedTask = await completeTask(parseInt(params[0]));
        console.log(`✓ Completed task: ${completedTask.description}`);
        break;
        
      case 'delete':
        const deletedTask = await deleteTask(parseInt(params[0]));
        console.log(`✓ Deleted task: ${deletedTask.description}`);
        break;
        
      default:
        console.log('Usage: node task-manager.js <command> [args]');
        console.log('Commands: add, list, complete, delete');
    }
  } catch (error) {
    console.error(`Error: ${error.message}`);
    process.exit(1);
  }
}

main();
```

---

### Build REST API Server

**Express Server Setup**

Setting up an Express server involves creating an Express application, configuring middleware, and defining routes. Basic setup includes body parsing middleware for JSON and URL-encoded data, error handling middleware, and a root route. The server listens on a specified port, typically from an environment variable. A proper Express setup provides a solid foundation for building REST APIs.

```javascript
const express = require('express');
const app = express();

// Middleware
app.use(express.json()); // Parse JSON request bodies
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded bodies

// Routes
app.get('/', (req, res) => {
  res.json({ message: 'Welcome to the API' });
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

**User Endpoints**

User endpoints implement CRUD operations for user resources, following RESTful conventions. GET `/users` retrieves all users with pagination, GET `/users/:id` retrieves a specific user, POST `/users` creates a new user, PUT `/users/:id` updates a user, and DELETE `/users/:id` deletes a user. Each endpoint uses appropriate HTTP methods and status codes, making the API predictable and easy to consume.

```javascript
// In-memory user storage (for demo purposes)
const users = [];
let nextId = 1;

// GET /users - Get all users
app.get('/users', (req, res) => {
  const { page = 1, limit = 10 } = req.query;
  const offset = (page - 1) * limit;
  
  const paginatedUsers = users.slice(offset, offset + parseInt(limit));
  
  res.json({
    data: paginatedUsers,
    pagination: {
      page: parseInt(page),
      limit: parseInt(limit),
      total: users.length,
      totalPages: Math.ceil(users.length / limit)
    }
  });
});

// GET /users/:id - Get specific user
app.get('/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  res.json(user);
});

// POST /users - Create new user
app.post('/users', (req, res) => {
  const { name, email } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({ error: 'Name and email are required' });
  }
  
  const newUser = {
    id: nextId++,
    name,
    email,
    createdAt: new Date().toISOString()
  };
  
  users.push(newUser);
  res.status(201).json(newUser);
});

// PUT /users/:id - Update user
app.put('/users/:id', (req, res) => {
  const userIndex = users.findIndex(u => u.id === parseInt(req.params.id));
  
  if (userIndex === -1) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  const { name, email } = req.body;
  
  users[userIndex] = {
    ...users[userIndex],
    name: name || users[userIndex].name,
    email: email || users[userIndex].email,
    updatedAt: new Date().toISOString()
  };
  
  res.json(users[userIndex]);
});

// DELETE /users/:id - Delete user
app.delete('/users/:id', (req, res) => {
  const userIndex = users.findIndex(u => u.id === parseInt(req.params.id));
  
  if (userIndex === -1) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  users.splice(userIndex, 1);
  res.status(204).send();
});
```

**Health Check Endpoint**

A health check endpoint provides a simple way to monitor the API's status. It typically returns a 200 OK status with information about the API's health, including database connectivity, cache status, and other dependencies. Health checks are essential for load balancers, container orchestrators, and monitoring systems to determine if the API is functioning correctly.

```javascript
// GET /health - Health check endpoint
app.get('/health', async (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    environment: process.env.NODE_ENV || 'development'
  };
  
  try {
    // Check database connectivity
    await db.execute('SELECT 1');
    health.database = 'connected';
  } catch (error) {
    health.status = 'unhealthy';
    health.database = 'disconnected';
    health.error = error.message;
  }
  
  // Check Redis connectivity (if using)
  try {
    await redis.ping();
    health.redis = 'connected';
  } catch (error) {
    health.redis = 'disconnected';
  }
  
  const statusCode = health.status === 'healthy' ? 200 : 503;
  res.status(statusCode).json(health);
});
```

**Centralized Error Handling**

Centralized error handling uses error-handling middleware to catch and process errors consistently. This includes creating custom error classes, wrapping async route handlers, and formatting error responses. Centralized error handling ensures that all errors are logged and returned in a consistent format, making the API more predictable and easier to debug.

```javascript
// Custom error classes
class AppError extends Error {
  constructor(message, statusCode = 500) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

class NotFoundError extends AppError {
  constructor(message = 'Resource not found') {
    super(message, 404);
  }
}

class ValidationError extends AppError {
  constructor(message = 'Validation failed', errors = []) {
    super(message, 400);
    this.errors = errors;
  }
}

// Async error wrapper
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// Error-handling middleware
app.use((err, req, res, next) => {
  console.error('Error:', err);
  
  const statusCode = err.statusCode || 500;
  const message = err.isOperational ? err.message : 'Internal Server Error';
  
  res.status(statusCode).json({
    error: {
      message,
      status: statusCode,
      ...(err.errors && { errors: err.errors }),
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    }
  });
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({ error: 'Not Found' });
});
```

**Logging Middleware**

Logging middleware records information about incoming requests and outgoing responses, which is crucial for debugging and monitoring. Popular logging libraries include `morgan` for HTTP request logging and `winston` for structured logging. Logs should include timestamps, request methods, URLs, status codes, response times, and relevant metadata. Proper logging helps identify issues, track usage patterns, and maintain application health.

```javascript
const morgan = require('morgan');
const winston = require('winston');

// Configure winston logger
const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'combined.log' }),
    new winston.transports.File({ filename: 'error.log', level: 'error' })
  ]
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

// Use morgan for HTTP request logging
app.use(morgan('combined', {
  stream: {
    write: (message) => logger.info(message.trim())
  }
}));

// Custom logging middleware
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    logger.info({
      method: req.method,
      url: req.url,
      status: res.statusCode,
      duration: `${duration}ms`,
      ip: req.ip
    });
  });
  
  next();
});
```

**Input Validation with Zod/Joi**

Input validation ensures that incoming data meets expected formats and constraints, preventing invalid data from reaching business logic. Libraries like Zod and Joi provide schema-based validation with detailed error messages. Validation should occur early in the request pipeline, typically in controllers or dedicated validation middleware. Proper input validation is a critical security measure that prevents injection attacks and data integrity issues.

```javascript
// Using Zod for validation
const { z } = require('zod');

// Define validation schemas
const createUserSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
  password: z.string().min(8)
});

const updateUserSchema = z.object({
  name: z.string().min(2).max(100).optional(),
  email: z.string().email().optional()
}).partial();

// Validation middleware
function validateBody(schema) {
  return (req, res, next) => {
    try {
      req.body = schema.parse(req.body);
      next();
    } catch (error) {
      const errors = error.errors.map(err => ({
        field: err.path.join('.'),
        message: err.message
      }));
      
      res.status(400).json({
        error: 'Validation failed',
        errors
      });
    }
  };
}

// Use validation middleware
app.post('/users', validateBody(createUserSchema), (req, res) => {
  const user = createUser(req.body);
  res.status(201).json(user);
});

app.put('/users/:id', validateBody(updateUserSchema), (req, res) => {
  const user = updateUser(req.params.id, req.body);
  res.json(user);
});

// Using Joi for validation
const Joi = require('joi');

const createUserSchemaJoi = Joi.object({
  name: Joi.string().min(2).max(100).required(),
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required()
});

function validateBodyJoi(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, { abortEarly: false });
    
    if (error) {
      const errors = error.details.map(detail => ({
        field: detail.path.join('.'),
        message: detail.message
      }));
      
      return res.status(400).json({
        error: 'Validation failed',
        errors
      });
    }
    
    req.body = value;
    next();
  };
}

app.post('/users', validateBodyJoi(createUserSchemaJoi), (req, res) => {
  const user = createUser(req.body);
  res.status(201).json(user);
});
```

---

### Build CRUD Todo REST API

**Complete CRUD Operations**

The Todo API implements full CRUD operations for managing todo items. Create adds a new todo, Read retrieves todos (all or by ID), Update modifies existing todos, and Delete removes todos. Each operation includes validation, error handling, and appropriate HTTP status codes. The API supports filtering by completion status and sorting by creation date, providing a flexible interface for todo management.

```javascript
const express = require('express');
const app = express();

app.use(express.json());

// In-memory storage
const todos = [];
let nextId = 1;

// POST /todos - Create todo
app.post('/todos', (req, res) => {
  const { title, description } = req.body;
  
  if (!title) {
    return res.status(400).json({ error: 'Title is required' });
  }
  
  const todo = {
    id: nextId++,
    title,
    description: description || '',
    completed: false,
    createdAt: new Date().toISOString()
  };
  
  todos.push(todo);
  res.status(201).json(todo);
});

// GET /todos - Get all todos
app.get('/todos', (req, res) => {
  const { completed, sort } = req.query;
  
  let filteredTodos = [...todos];
  
  // Filter by completion status
  if (completed !== undefined) {
    const isCompleted = completed === 'true';
    filteredTodos = filteredTodos.filter(t => t.completed === isCompleted);
  }
  
  // Sort by creation date
  if (sort === 'asc') {
    filteredTodos.sort((a, b) => new Date(a.createdAt) - new Date(b.createdAt));
  } else if (sort === 'desc') {
    filteredTodos.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
  }
  
  res.json({
    data: filteredTodos,
    total: filteredTodos.length
  });
});

// GET /todos/:id - Get specific todo
app.get('/todos/:id', (req, res) => {
  const todo = todos.find(t => t.id === parseInt(req.params.id));
  
  if (!todo) {
    return res.status(404).json({ error: 'Todo not found' });
  }
  
  res.json(todo);
});

// PUT /todos/:id - Update todo
app.put('/todos/:id', (req, res) => {
  const todoIndex = todos.findIndex(t => t.id === parseInt(req.params.id));
  
  if (todoIndex === -1) {
    return res.status(404).json({ error: 'Todo not found' });
  }
  
  const { title, description, completed } = req.body;
  
  todos[todoIndex] = {
    ...todos[todoIndex],
    title: title || todos[todoIndex].title,
    description: description !== undefined ? description : todos[todoIndex].description,
    completed: completed !== undefined ? completed : todos[todoIndex].completed,
    updatedAt: new Date().toISOString()
  };
  
  res.json(todos[todoIndex]);
});

// PATCH /todos/:id/complete - Mark todo as complete
app.patch('/todos/:id/complete', (req, res) => {
  const todo = todos.find(t => t.id === parseInt(req.params.id));
  
  if (!todo) {
    return res.status(404).json({ error: 'Todo not found' });
  }
  
  todo.completed = true;
  todo.completedAt = new Date().toISOString();
  
  res.json(todo);
});

// DELETE /todos/:id - Delete todo
app.delete('/todos/:id', (req, res) => {
  const todoIndex = todos.findIndex(t => t.id === parseInt(req.params.id));
  
  if (todoIndex === -1) {
    return res.status(404).json({ error: 'Todo not found' });
  }
  
  todos.splice(todoIndex, 1);
  res.status(204).send();
});
```

**Sorting and Filtering Endpoints**

Sorting and filtering endpoints allow clients to retrieve subsets of data based on specific criteria. The Todo API supports filtering by completion status (`?completed=true`) and sorting by creation date (`?sort=asc` or `?sort=desc`). These features improve API usability by allowing clients to retrieve exactly the data they need without additional client-side processing. Implementing sorting and filtering requires careful query parameter parsing and array manipulation.

```javascript
// GET /todos with filtering and sorting
app.get('/todos', (req, res) => {
  const { 
    completed, 
    sort = 'desc', // Default to descending
    page = 1,
    limit = 10
  } = req.query;
  
  let filteredTodos = [...todos];
  
  // Filter by completion status
  if (completed !== undefined) {
    const isCompleted = completed === 'true';
    filteredTodos = filteredTodos.filter(t => t.completed === isCompleted);
  }
  
  // Sort by creation date
  filteredTodos.sort((a, b) => {
    const dateA = new Date(a.createdAt);
    const dateB = new Date(b.createdAt);
    return sort === 'asc' ? dateA - dateB : dateB - dateA;
  });
  
  // Pagination
  const offset = (page - 1) * limit;
  const paginatedTodos = filteredTodos.slice(offset, offset + parseInt(limit));
  
  res.json({
    data: paginatedTodos,
    pagination: {
      page: parseInt(page),
      limit: parseInt(limit),
      total: filteredTodos.length,
      totalPages: Math.ceil(filteredTodos.length / limit)
    }
  });
});

// Advanced filtering with multiple criteria
app.get('/todos', (req, res) => {
  const { 
    completed, 
    title, // Search by title
    startDate, // Filter todos created after date
    endDate, // Filter todos created before date
    sort = 'desc',
    page = 1,
    limit = 10
  } = req.query;
  
  let filteredTodos = [...todos];
  
  // Filter by completion status
  if (completed !== undefined) {
    const isCompleted = completed === 'true';
    filteredTodos = filteredTodos.filter(t => t.completed === isCompleted);
  }
  
  // Filter by title (case-insensitive search)
  if (title) {
    const searchTerm = title.toLowerCase();
    filteredTodos = filteredTodos.filter(t => 
      t.title.toLowerCase().includes(searchTerm)
    );
  }
  
  // Filter by date range
  if (startDate) {
    const start = new Date(startDate);
    filteredTodos = filteredTodos.filter(t => new Date(t.createdAt) >= start);
  }
  
  if (endDate) {
    const end = new Date(endDate);
    filteredTodos = filteredTodos.filter(t => new Date(t.createdAt) <= end);
  }
  
  // Sort
  filteredTodos.sort((a, b) => {
    const dateA = new Date(a.createdAt);
    const dateB = new Date(b.createdAt);
    return sort === 'asc' ? dateA - dateB : dateB - dateA;
  });
  
  // Pagination
  const offset = (page - 1) * limit;
  const paginatedTodos = filteredTodos.slice(offset, offset + parseInt(limit));
  
  res.json({
    data: paginatedTodos,
    pagination: {
      page: parseInt(page),
      limit: parseInt(limit),
      total: filteredTodos.length,
      totalPages: Math.ceil(filteredTodos.length / limit)
    }
  });
});
```

**Rate Limiting Implementation**

Rate limiting protects APIs from abuse by limiting the number of requests a client can make within a time window. The `express-rate-limit` middleware provides an easy way to implement rate limiting in Express applications. Rate limiting can be applied globally or to specific routes, with different limits for different endpoints. Proper rate limiting prevents DDoS attacks, ensures fair resource usage, and protects against brute force attacks.

```javascript
const rateLimit = require('express-rate-limit');

// Global rate limiter
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP, please try again later',
  standardHeaders: true, // Return rate limit info in the `RateLimit-*` headers
  legacyHeaders: false // Disable the `X-RateLimit-*` headers
});

// Apply global rate limiting
app.use('/api', globalLimiter);

// Stricter rate limiting for auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // Limit each IP to 5 requests per windowMs
  message: 'Too many login attempts, please try again later',
  skipSuccessfulRequests: true // Don't count successful requests
});

app.post('/login', authLimiter, (req, res) => {
  // Login logic
});

// Rate limiting with Redis for distributed systems
const RedisStore = require('rate-limit-redis');
const redis = require('redis');

const redisClient = redis.createClient({
  host: process.env.REDIS_HOST,
  port: process.env.REDIS_PORT
});

const distributedLimiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rate-limit:'
  }),
  windowMs: 15 * 60 * 1000,
  max: 100
});

app.use('/api', distributedLimiter);

// Custom rate limiting logic
const requestCounts = new Map();

function customRateLimit(options) {
  const { windowMs = 60000, max = 100 } = options;
  
  return (req, res, next) => {
    const ip = req.ip;
    const now = Date.now();
    
    // Clean old entries
    for (const [key, value] of requestCounts.entries()) {
      if (now - value.timestamp > windowMs) {
        requestCounts.delete(key);
      }
    }
    
    // Get or create entry
    const entry = requestCounts.get(ip) || { count: 0, timestamp: now };
    
    if (now - entry.timestamp > windowMs) {
      entry.count = 0;
      entry.timestamp = now;
    }
    
    entry.count++;
    requestCounts.set(ip, entry);
    
    // Check limit
    if (entry.count > max) {
      return res.status(429).json({
        error: 'Too many requests',
        retryAfter: Math.ceil((windowMs - (now - entry.timestamp)) / 1000)
      });
    }
    
    // Add rate limit headers
    res.set('X-RateLimit-Limit', max);
    res.set('X-RateLimit-Remaining', max - entry.count);
    res.set('X-RateLimit-Reset', new Date(entry.timestamp + windowMs).toISOString());
    
    next();
  };
}

app.use('/api', customRateLimit({ windowMs: 60000, max: 100 }));
```

**File Upload Handling**

File upload handling allows clients to upload files to the server, which is essential for features like profile pictures, document uploads, and media management. The `multer` middleware handles `multipart/form-data` requests, storing files on disk or in memory. File uploads should include validation for file type, size, and count to prevent abuse. Uploaded files can be stored locally, on cloud storage like S3, or processed immediately.

```javascript
const multer = require('multer');
const path = require('path');
const fs = require('fs').promises;

// Configure storage
const storage = multer.diskStorage({
  destination: async (req, file, cb) => {
    const uploadDir = path.join(__dirname, 'uploads');
    await fs.mkdir(uploadDir, { recursive: true });
    cb(null, uploadDir);
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
  }
});

// Configure file filter
const fileFilter = (req, file, cb) => {
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
  
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error('Invalid file type. Only JPEG, PNG, and GIF are allowed.'), false);
  }
};

// Configure multer
const upload = multer({
  storage: storage,
  fileFilter: fileFilter,
  limits: {
    fileSize: 5 * 1024 * 1024 // 5MB limit
  }
});

// Single file upload
app.post('/upload', upload.single('file'), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }
  
  res.json({
    message: 'File uploaded successfully',
    file: {
      originalName: req.file.originalname,
      filename: req.file.filename,
      mimetype: req.file.mimetype,
      size: req.file.size,
      path: req.file.path
    }
  });
});

// Multiple file upload
app.post('/upload-multiple', upload.array('files', 5), (req, res) => {
  if (!req.files || req.files.length === 0) {
    return res.status(400).json({ error: 'No files uploaded' });
  }
  
  const files = req.files.map(file => ({
    originalName: file.originalname,
    filename: file.filename,
    mimetype: file.mimetype,
    size: file.size,
    path: file.path
  }));
  
  res.json({
    message: `${files.length} files uploaded successfully`,
    files
  });
});

// Upload with fields
app.post('/upload-with-fields', 
  upload.fields([
    { name: 'avatar', maxCount: 1 },
    { name: 'documents', maxCount: 3 }
  ]),
  (req, res) => {
    const avatar = req.files['avatar'] ? req.files['avatar'][0] : null;
    const documents = req.files['documents'] || [];
    
    res.json({
      avatar,
      documents,
      fields: req.body
    });
  }
);

// Serve uploaded files
app.use('/uploads', express.static(path.join(__dirname, 'uploads')));

// Delete uploaded file
app.delete('/uploads/:filename', async (req, res) => {
  const filePath = path.join(__dirname, 'uploads', req.params.filename);
  
  try {
    await fs.unlink(filePath);
    res.json({ message: 'File deleted successfully' });
  } catch (error) {
    if (error.code === 'ENOENT') {
      return res.status(404).json({ error: 'File not found' });
    }
    throw error;
  }
});

// Error handling for file uploads
app.use((error, req, res, next) => {
  if (error instanceof multer.MulterError) {
    if (error.code === 'LIMIT_FILE_SIZE') {
      return res.status(400).json({ error: 'File size exceeds the limit' });
    }
    if (error.code === 'LIMIT_FILE_COUNT') {
      return res.status(400).json({ error: 'Too many files uploaded' });
    }
    return res.status(400).json({ error: error.message });
  }
  
  if (error.message === 'Invalid file type') {
    return res.status(400).json({ error: error.message });
  }
  
  next(error);
});
```

---

## Quick Reference

**Express Middleware Order**

Middleware execution order is critical for proper application behavior. Request parsing middleware must come before route handlers. Error-handling middleware must come after all other middleware and routes. Understanding this order helps prevent issues like authentication running after route handlers or error handlers being unreachable.

```javascript
// Correct middleware order
app.use(helmet()); // Security headers (first)
app.use(cors()); // CORS
app.use(express.json()); // Parse JSON bodies
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded bodies
app.use(morgan('combined')); // Logging
app.use('/api', rateLimit()); // Rate limiting
app.use(authMiddleware); // Authentication

// Routes
app.get('/users', userController.getUsers);
app.post('/users', userController.createUser);

// Error handling (last)
app.use(notFoundHandler);
app.use(errorHandler);
```

**HTTP Status Codes Quick Reference**

Using the correct HTTP status codes makes APIs more predictable and easier to integrate with. Success codes indicate successful requests, client error codes indicate client-side errors, and server error codes indicate server-side errors.

| Code | Category | Use Case |
|------|----------|----------|
| 200 | Success | OK - Request succeeded |
| 201 | Success | Created - Resource created |
| 204 | Success | No Content - Successful DELETE |
| 400 | Client Error | Bad Request - Invalid input |
| 401 | Client Error | Unauthorized - Missing auth |
| 403 | Client Error | Forbidden - Insufficient permissions |
| 404 | Client Error | Not Found - Resource missing |
| 409 | Client Error | Conflict - Duplicate resource |
| 422 | Client Error | Unprocessable - Validation failed |
| 429 | Client Error | Too Many Requests - Rate limit exceeded |
| 500 | Server Error | Internal Server Error - Server error |

**REST API Design Checklist**

Following REST API design best practices ensures your API is intuitive, predictable, and easy to consume. Use nouns for resource names, pluralize collections, and use proper HTTP methods. Implement proper status codes, pagination, and error handling.

- Use nouns, not verbs, for resource names
- Use plural nouns for collections (`/users`, not `/user`)
- Use proper HTTP methods (GET, POST, PUT, DELETE)
- Return appropriate status codes
- Implement pagination for large datasets
- Use query parameters for filtering and sorting
- Version your API (`/api/v1/users`)
- Implement consistent error responses
- Use HATEOAS for discoverable actions
- Document your API with OpenAPI/Swagger

**Error Handling Pattern**

Centralized error handling ensures consistent error responses across the application. Create custom error classes, wrap async handlers, and use error-handling middleware to format responses.

```javascript
// Custom error class
class AppError extends Error {
  constructor(message, statusCode = 500) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

// Async wrapper
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// Error handler
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const message = err.isOperational ? err.message : 'Internal Server Error';
  
  res.status(statusCode).json({
    error: {
      message,
      status: statusCode
    }
  });
});
```

**Layered Architecture Pattern**

Layered architecture separates concerns into distinct layers: controllers handle HTTP, services handle business logic, and repositories handle data access. This separation makes the codebase more maintainable and testable.

```
Controller Layer (HTTP)
    ↓
Service Layer (Business Logic)
    ↓
Repository Layer (Data Access)
    ↓
Database
```

**Configuration Best Practices**

Proper configuration management keeps sensitive data out of version control and makes applications easier to deploy across environments. Use environment variables, validate configuration at startup, and never commit secrets to version control.

- Use `.env` files for local development
- Load environment variables with `dotenv`
- Validate configuration at startup
- Never commit `.env` files to version control
- Use `.env.example` as a template
- Store secrets in environment variables
- Use secrets management services in production
- Rotate secrets regularly

**Validation with Zod**

Zod provides type-safe schema validation for JavaScript. Define schemas, validate request bodies, and return detailed error messages for invalid data.

```javascript
const { z } = require('zod');

const schema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
  age: z.number().min(18).optional()
});

function validateBody(schema) {
  return (req, res, next) => {
    try {
      req.body = schema.parse(req.body);
      next();
    } catch (error) {
      res.status(400).json({ errors: error.errors });
    }
  };
}

app.post('/users', validateBody(schema), (req, res) => {
  // req.body is validated and typed
});
```

**Rate Limiting Configuration**

Rate limiting protects APIs from abuse by limiting request rates. Configure different limits for different endpoints, use Redis for distributed systems, and provide clear error messages.

```javascript
const rateLimit = require('express-rate-limit');

// Global limiter
app.use(rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: 'Too many requests'
}));

// Strict limiter for auth
app.post('/login', rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  message: 'Too many login attempts'
}), loginHandler);
```

**File Upload with Multer**

Multer handles multipart/form-data uploads. Configure storage, file filters, and size limits to prevent abuse.

```javascript
const multer = require('multer');

const upload = multer({
  storage: multer.diskStorage({
    destination: 'uploads/',
    filename: (req, file, cb) => {
      cb(null, Date.now() + '-' + file.originalname);
    }
  }),
  fileFilter: (req, file, cb) => {
    if (file.mimetype.startsWith('image/')) {
      cb(null, true);
    } else {
      cb(new Error('Only images allowed'), false);
    }
  },
  limits: { fileSize: 5 * 1024 * 1024 } // 5MB
});

app.post('/upload', upload.single('file'), (req, res) => {
  res.json({ file: req.file });
});
```
