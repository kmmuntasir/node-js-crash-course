# Module 9: Integration Projects & Interview Preparation

## Full-Stack Integration Projects

### Complete Task Manager Application

**Backend (Node.js + Express)**

Build a RESTful API with authentication, CRUD operations, and security measures. The backend should handle task management with proper validation, error handling, and rate limiting.

- **RESTful API Design**: Use proper HTTP methods (GET, POST, PUT, DELETE) and status codes
- **JWT Authentication**: Token-based authentication with refresh mechanism
- **CRUD Operations**: Create, read, update, delete tasks with proper validation
- **Input Validation**: Use Zod or Joi for request validation
- **Error Handling**: Centralized error handling middleware
- **Rate Limiting**: Express-rate-limit to prevent abuse
- **Security Headers**: Helmet.js for security headers

```javascript
// Express server setup with security
const express = require('express');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

const app = express();

// Security middleware
app.use(helmet());
app.use(rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
}));

// JWT authentication middleware
const authenticateToken = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};

// Protected route
app.get('/api/tasks', authenticateToken, async (req, res) => {
  const tasks = await Task.find({ userId: req.user.id });
  res.json(tasks);
});

// Input validation with Zod
const taskSchema = z.object({
  title: z.string().min(1).max(100),
  description: z.string().max(500),
  completed: z.boolean()
});

app.post('/api/tasks', authenticateToken, async (req, res) => {
  try {
    const validated = taskSchema.parse(req.body);
    const task = await Task.create({ ...validated, userId: req.user.id });
    res.status(201).json(task);
  } catch (error) {
    res.status(400).json({ error: error.errors });
  }
});
```

**Frontend (React)**

Build a React application with component architecture, state management, routing, and API integration. The frontend should handle authentication, task management, and provide good user experience with loading and error states.

- **Component Architecture**: Organized components (presentational vs container)
- **State Management**: Context API for global state (auth, tasks)
- **Routing**: React Router for navigation and protected routes
- **Protected Routes**: Route guards for authenticated users
- **API Integration**: Axios or fetch for backend communication
- **Loading States**: Show loading indicators during API calls
- **Error States**: Display user-friendly error messages

```javascript
// Custom useAuth hook
import { createContext, useContext, useState, useEffect } from 'react';

const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const token = localStorage.getItem('token');
    if (token) {
      // Verify token with backend
      fetch('/api/auth/verify', {
        headers: { Authorization: `Bearer ${token}` }
      })
      .then(res => res.json())
      .then(data => {
        setUser(data.user);
        setLoading(false);
      })
      .catch(() => {
        localStorage.removeItem('token');
        setLoading(false);
      });
    } else {
      setLoading(false);
    }
  }, []);

  const login = async (email, password) => {
    const res = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password })
    });
    const data = await res.json();
    localStorage.setItem('token', data.token);
    setUser(data.user);
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => useContext(AuthContext);

// Protected route component
import { Navigate } from 'react-router-dom';
import { useAuth } from './AuthContext';

const ProtectedRoute = ({ children }) => {
  const { user, loading } = useAuth();

  if (loading) {
    return <div>Loading...</div>;
  }

  if (!user) {
    return <Navigate to="/login" />;
  }

  return children;
};

// Custom useTasks hook
import { useState, useEffect } from 'react';

export const useTasks = () => {
  const [tasks, setTasks] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const { user } = useAuth();

  useEffect(() => {
    const fetchTasks = async () => {
      setLoading(true);
      setError(null);
      try {
        const token = localStorage.getItem('token');
        const res = await fetch('/api/tasks', {
          headers: { Authorization: `Bearer ${token}` }
        });
        const data = await res.json();
        setTasks(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    if (user) {
      fetchTasks();
    }
  }, [user]);

  const addTask = async (taskData) => {
    try {
      const token = localStorage.getItem('token');
      const res = await fetch('/api/tasks', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Bearer ${token}`
        },
        body: JSON.stringify(taskData)
      });
      const newTask = await res.json();
      setTasks([...tasks, newTask]);
    } catch (err) {
      setError(err.message);
    }
  };

  return { tasks, loading, error, addTask };
};
```

**Security Implementation**

Implement comprehensive security measures across the full-stack application to protect against common vulnerabilities.

- **XSS Prevention**: Input sanitization, output encoding
- **CSRF Protection**: CSRF tokens, SameSite cookies
- **Input Sanitization**: DOMPurify for user-generated content
- **Secure Headers**: Content-Security-Policy, X-Frame-Options
- **HTTPS Only**: Enforce HTTPS in production
- **Environment Variables**: Never commit secrets, use .env files

```javascript
// XSS prevention in React
import DOMPurify from 'dompurify';

function TaskItem({ task }) {
  return (
    <div>
      <h3>{task.title}</h3>
      {/* Sanitize user-generated content */}
      <p 
        dangerouslySetInnerHTML={{ 
          __html: DOMPurify.sanitize(task.description) 
        }}
      />
    </div>
  );
}

// CSRF protection in Express
const csrf = require('csurf');
const cookieParser = require('cookie-parser');

app.use(cookieParser());
app.use(csrf({ cookie: true }));

app.get('/api/csrf-token', (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// Frontend includes CSRF token in requests
const token = await fetch('/api/csrf-token').then(res => res.json());

fetch('/api/tasks', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': token.csrfToken
  },
  body: JSON.stringify(taskData)
});

// Security headers with Helmet
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  }
}));
```

**Deployment**

Deploy the full-stack application to production with proper configuration, monitoring, and scalability.

- **Backend Deployment**: EC2 or Lambda with API Gateway
- **Frontend Deployment**: S3 + CloudFront for static hosting
- **HTTPS Configuration**: SSL/TLS certificates with ACM
- **Environment Configuration**: Separate configs for dev/staging/prod
- **Process Management**: PM2 for Node.js process management
- **Monitoring**: CloudWatch for logs and metrics

```bash
# Backend deployment to EC2
# 1. Launch EC2 instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.micro \
  --key-name my-key-pair \
  --security-group-ids sg-12345678

# 2. SSH and setup
ssh -i my-key-pair.pem ec2-user@<public-ip>

# 3. Install dependencies
sudo yum update -y
sudo yum install -y nodejs npm
sudo npm install -g pm2

# 4. Deploy application
git clone https://github.com/user/task-manager-api.git
cd task-manager-api
npm install --production

# 5. Configure environment variables
cat > .env <<EOF
NODE_ENV=production
PORT=3000
DB_HOST=production-db.example.com
JWT_SECRET=$(openssl rand -base64 32)
EOF

# 6. Start with PM2
pm2 start server.js --name task-manager-api
pm2 save
pm2 startup systemd -u ec2-user --hp /home/ec2-user

# Frontend deployment to S3
# 1. Build React app
cd task-manager-frontend
npm run build

# 2. Create S3 bucket
aws s3 mb s3://task-manager-frontend

# 3. Upload build files
aws s3 sync build/ s3://task-manager-frontend/ --delete

# 4. Configure static website hosting
aws s3api put-bucket-website \
  --bucket task-manager-frontend \
  --website-configuration '{
    "IndexDocument": "index.html",
    "ErrorDocument": "index.html"
  }'

# 5. Set bucket policy for public access
cat > bucket-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::task-manager-frontend/*"
    }
  ]
}
EOF

aws s3api put-bucket-policy \
  --bucket task-manager-frontend \
  --policy file://bucket-policy.json

# 6. Configure CloudFront (optional)
aws cloudfront create-distribution \
  --default-root-object index.html \
  --origins DomainName=task-manager-frontend.s3-website-us-east-1.amazonaws.com,Id=origin1,CustomOriginConfig={OriginProtocolPolicy:http-only} \
  --default-cache-behavior TargetOriginId=origin1,ViewerProtocolPolicy=allow-all,ForwardedValues=QueryString,MinTTL=86400,Compress=true

# 7. Configure HTTPS with ACM
aws acm request-certificate \
  --domain-name api.taskmanager.com \
  --subject-alternative-names www.taskmanager.com \
  --validation-method DNS

# 8. Create CloudFront distribution for API
aws cloudfront create-distribution \
  --default-cache-behavior TargetOriginId=api-origin,ViewerProtocolPolicy=https-only,MinTTL=0,Compress=true \
  --origins Id=api-origin,DomainName=api.taskmanager.com,CustomOriginConfig={OriginProtocolPolicy:https-only,OriginSslProtocols:[TLSv1.2,TLSv1.1,TLSv1.0]} \
  --viewer-certificate <certificate-arn>
```

---

## Final Project Polish

### Code Quality

**Apply Design Patterns**

Refactor code to use appropriate design patterns for better maintainability and testability.

- **Repository Pattern**: Abstract data access logic
- **Factory Pattern**: Create objects dynamically
- **Strategy Pattern**: Interchangeable algorithms
- **Singleton Pattern**: Single instance for shared resources
- **Observer Pattern**: Event-driven updates
- **Dependency Injection**: Pass dependencies instead of hardcoding

```javascript
// Repository pattern for data access
class TaskRepository {
  constructor(db) {
    this.db = db;
  }

  async findAll(userId) {
    return await this.db.query('SELECT * FROM tasks WHERE user_id = ?', [userId]);
  }

  async findById(id) {
    const [task] = await this.db.query('SELECT * FROM tasks WHERE id = ?', [id]);
    return task;
  }

  async create(taskData) {
    const [result] = await this.db.query(
      'INSERT INTO tasks (title, description, user_id) VALUES (?, ?, ?)',
      [taskData.title, taskData.description, taskData.userId]
    );
    return { id: result.insertId, ...taskData };
  }

  async update(id, taskData) {
    await this.db.query(
      'UPDATE tasks SET title = ?, description = ? WHERE id = ?',
      [taskData.title, taskData.description, id]
    );
  }

  async delete(id) {
    await this.db.query('DELETE FROM tasks WHERE id = ?', [id]);
  }
}

// Factory pattern for task creation
class TaskFactory {
  static create(type, data) {
    switch (type) {
      case 'personal':
        return new PersonalTask(data);
      case 'work':
        return new WorkTask(data);
      case 'shopping':
        return new ShoppingTask(data);
      default:
        throw new Error('Unknown task type');
    }
  }
}

// Strategy pattern for task sorting
class SortStrategy {
  sort(tasks) {
    throw new Error('Must implement sort method');
  }
}

class ByDateStrategy extends SortStrategy {
  sort(tasks) {
    return [...tasks].sort((a, b) => new Date(a.dueDate) - new Date(b.dueDate));
  }
}

class ByPriorityStrategy extends SortStrategy {
  sort(tasks) {
    const priorityOrder = { high: 1, medium: 2, low: 3 };
    return [...tasks].sort((a, b) => priorityOrder[a.priority] - priorityOrder[b.priority]);
  }
}
```

**Refactor for SOLID Principles**

Apply SOLID principles to create maintainable, scalable code.

- **Single Responsibility**: Each class/function has one reason to change
- **Open/Closed**: Open for extension, closed for modification
- **Liskov Substitution**: Subtypes must be substitutable for base types
- **Interface Segregation**: Small, focused interfaces
- **Dependency Inversion**: Depend on abstractions, not concretions

```javascript
// Before: Violates SRP
class TaskController {
  async createTask(req, res) {
    // Validation logic
    if (!req.body.title || req.body.title.length > 100) {
      return res.status(400).json({ error: 'Invalid title' });
    }
    
    // Database logic
    const task = await db.insert('tasks', req.body);
    
    // Email logic
    await emailService.send(req.user.email, 'Task created');
    
    // Logging logic
    logger.info('Task created', { taskId: task.id });
    
    res.json(task);
  }
}

// After: Follows SRP
class TaskValidator {
  validate(taskData) {
    const errors = [];
    if (!taskData.title) errors.push('Title is required');
    if (taskData.title?.length > 100) errors.push('Title too long');
    return errors;
  }
}

class TaskRepository {
  constructor(db) {
    this.db = db;
  }
  
  async create(taskData) {
    return await this.db.insert('tasks', taskData);
  }
}

class NotificationService {
  constructor(emailService, logger) {
    this.emailService = emailService;
    this.logger = logger;
  }
  
  async notifyTaskCreated(user, task) {
    await this.emailService.send(user.email, 'Task created');
    this.logger.info('Task created', { taskId: task.id });
  }
}

class TaskController {
  constructor(validator, repository, notificationService) {
    this.validator = validator;
    this.repository = repository;
    this.notificationService = notificationService;
  }
  
  async createTask(req, res) {
    const errors = this.validator.validate(req.body);
    if (errors.length > 0) {
      return res.status(400).json({ errors });
    }
    
    const task = await this.repository.create(req.body);
    await this.notificationService.notifyTaskCreated(req.user, task);
    
    res.json(task);
  }
}

// Open/Closed Principle
interface TaskProcessor {
  process(task: Task): Promise<void>;
}

class EmailTaskProcessor implements TaskProcessor {
  async process(task) {
    await this.emailService.send(task.user.email, task.title);
  }
}

class NotificationTaskProcessor implements TaskProcessor {
  async process(task) {
    await this.notificationService.push(task.user.id, task.title);
  }
}

// Extend without modifying existing code
class SMSNotificationTaskProcessor implements TaskProcessor {
  async process(task) {
    await this.smsService.send(task.user.phone, task.title);
  }
}

// Liskov Substitution Principle
class Bird {
  fly() {
    console.log('Flying');
  }
}

class Penguin extends Bird {
  fly() {
    throw new Error('Penguins cannot fly');
  }
}

// BAD: Breaks LSP - cannot substitute Penguin for Bird
function makeBirdFly(bird: Bird) {
  bird.fly();
}

// GOOD: Follows LSP - separate flying and non-flying birds
class FlyingBird extends Bird {
  fly() {
    console.log('Flying');
  }
}

class NonFlyingBird extends Bird {
  walk() {
    console.log('Walking');
  }
}

function makeBirdFly(bird: FlyingBird) {
  bird.fly();
}

// Interface Segregation Principle
// BAD: Fat interface
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
}

// GOOD: Segregated interfaces
interface Workable {
  work(): void;
}

interface Eatable {
  eat(): void;
}

interface Sleepable {
  sleep(): void;
}

class Employee implements Workable, Eatable, Sleepable {
  work() { /* ... */ }
  eat() { /* ... */ }
  sleep() { /* ... */ }
}

// Dependency Inversion Principle
// BAD: Depends on concrete implementation
class TaskController {
  constructor() {
    this.db = new MySQLDatabase();
    this.emailService = new SendGridEmailService();
  }
}

// GOOD: Depends on abstractions
class TaskController {
  constructor(db, emailService) {
    this.db = db; // Database interface
    this.emailService = emailService; // Email service interface
  }
}

// Usage with dependency injection
const db = new PostgreSQLDatabase();
const emailService = new AWSSESEmailService();
const controller = new TaskController(db, emailService);
```

**Add Comprehensive Error Handling**

Implement robust error handling across the application.

- **Custom Error Classes**: Extend Error class for specific error types
- **Error Middleware**: Centralized error handling in Express
- **Error Boundaries**: React error boundaries for UI errors
- **Logging**: Structured error logging with context
- **User-Friendly Messages**: Map technical errors to user-friendly messages

```javascript
// Custom error classes
class AppError extends Error {
  constructor(message, statusCode, isOperational = true) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = isOperational;
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(message) {
    super(message, 400);
  }
}

class AuthenticationError extends AppError {
  constructor(message) {
    super(message, 401);
  }
}

class NotFoundError extends AppError {
  constructor(message) {
    super(message, 404);
  }
}

// Express error handling middleware
const errorHandler = (err, req, res, next) => {
  // Log error
  console.error({
    message: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method,
    ip: req.ip,
    userAgent: req.headers['user-agent']
  });

  // Operational errors (known errors)
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      error: err.message
    });
  }

  // Unknown errors
  console.error('Unexpected error:', err);
  return res.status(500).json({
    error: 'An unexpected error occurred'
  });
};

// 404 handler
const notFoundHandler = (req, res) => {
  res.status(404).json({
    error: 'Resource not found'
  });
};

// React Error Boundary
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => window.location.reload()}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <App />
</ErrorBoundary>
```

**Implement Logging**

Add structured logging throughout the application for debugging and monitoring.

- **Structured Logging**: JSON format with consistent fields
- **Log Levels**: Error, warn, info, debug
- **Context Information**: Include request ID, user ID, timestamp
- **Performance Logging**: Log slow operations
- **Error Tracking**: Log errors with stack traces

```javascript
// Structured logger
class Logger {
  constructor(service) {
    this.service = service;
  }

  log(level, message, meta = {}) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      level,
      service: this.service,
      message,
      ...meta
    };

    console.log(JSON.stringify(logEntry));
  }

  error(message, meta) {
    this.log('error', message, meta);
  }

  warn(message, meta) {
    this.log('warn', message, meta);
  }

  info(message, meta) {
    this.log('info', message, meta);
  }

  debug(message, meta) {
    this.log('debug', message, meta);
  }
}

// Usage with context
const logger = new Logger('task-service');

app.use((req, res, next) => {
  req.requestId = require('uuid').v4();
  logger.info('Request received', {
    requestId: req.requestId,
    method: req.method,
    url: req.url,
    userId: req.user?.id
  });
  next();
});

// Performance logging
const logPerformance = async (operationName, operation) => {
  const start = Date.now();
  const result = await operation();
  const duration = Date.now() - start;
  
  if (duration > 1000) {
    logger.warn('Slow operation', {
      operation: operationName,
      duration: `${duration}ms`
    });
  }
  
  return result;
};

// Usage
await logPerformance('database-query', () => 
  db.query('SELECT * FROM tasks WHERE user_id = ?', [userId])
);
```

### Documentation

**Create Comprehensive README**

Document the project thoroughly for others to understand and use it.

- **Project Overview**: What the project does and its purpose
- **Architecture Description**: High-level architecture diagram and explanation
- **Security Decisions**: Why specific security measures were chosen
- **Trade-offs and Alternatives**: Discuss architectural decisions and alternatives
- **Setup Instructions**: Step-by-step setup guide
- **API Documentation**: Endpoints, request/response formats
- **Environment Variables**: Required environment variables and their purposes

```markdown
# Task Manager Application

## Overview

A full-stack task management application built with Node.js, Express, React, and PostgreSQL. Users can create, read, update, and delete tasks with authentication and real-time updates.

## Architecture

```
┌─────────────────────────────────────────────────┐
│                   Users                        │
└────────────────────┬────────────────────────┘
                     │
              [CloudFront CDN]
                     │
              ┌────────┴────────┐
              │                 │
        [React Frontend]    [API Gateway]
              │                 │
              └────────┬────────┘
                       │
                [Express API]
                       │
                ┌───────┴───────┐
                │               │
           [PostgreSQL]     [Redis Cache]
```

### Components

- **Frontend**: React 18 with Context API for state management
- **Backend**: Express.js REST API with JWT authentication
- **Database**: PostgreSQL for persistent storage
- **Cache**: Redis for session management and caching
- **CDN**: CloudFront for static asset delivery
- **Deployment**: EC2 for backend, S3 + CloudFront for frontend

## Security Decisions

### Authentication
- **JWT Tokens**: Chosen over sessions for stateless scalability
- **Token Expiration**: 1 hour access token, 7 day refresh token
- **Refresh Mechanism**: Automatic token refresh before expiration

### Input Validation
- **Zod Schema**: Runtime type validation with clear error messages
- **Sanitization**: DOMPurify for XSS prevention on frontend

### Rate Limiting
- **Express-Rate-Limit**: 100 requests per 15 minutes per IP
- **Redis Storage**: Distributed rate limiting across multiple instances

## Trade-offs and Alternatives

### Database Choice
**Chosen**: PostgreSQL
**Reasons**: ACID compliance, complex queries, relational data model
**Alternatives Considered**: MongoDB (flexible schema), DynamoDB (serverless)

### State Management
**Chosen**: Context API
**Reasons**: Built-in, no additional dependencies, sufficient for app size
**Alternatives Considered**: Redux (overkill for this app), Zustand (lighter but less familiar)

### Deployment Strategy
**Chosen**: EC2 + S3 + CloudFront
**Reasons**: Full control, cost-effective for current scale, familiar stack
**Alternatives Considered**: Lambda + API Gateway (serverless, but cold starts), Elastic Beanstalk (managed but less control)

## Setup Instructions

### Prerequisites

- Node.js 18+
- PostgreSQL 13+
- Redis 6+
- npm or yarn

### Installation

```bash
# Clone repository
git clone https://github.com/username/task-manager.git
cd task-manager

# Install backend dependencies
cd backend
npm install

# Install frontend dependencies
cd ../frontend
npm install

# Configure environment variables
cp backend/.env.example backend/.env
# Edit backend/.env with your values

# Setup database
createdb task_manager
psql task_manager < backend/schema.sql

# Start backend
cd backend
npm run dev

# Start frontend (new terminal)
cd frontend
npm start
```

### Environment Variables

| Variable | Description | Required | Default |
|----------|-------------|-----------|---------|
| NODE_ENV | Environment (development/production) | Yes | development |
| PORT | Backend server port | Yes | 3000 |
| DB_HOST | PostgreSQL host | Yes | localhost |
| DB_PORT | PostgreSQL port | Yes | 5432 |
| DB_NAME | Database name | Yes | task_manager |
| DB_USER | Database user | Yes | postgres |
| DB_PASSWORD | Database password | Yes | - |
| JWT_SECRET | Secret for JWT signing | Yes | - |
| JWT_EXPIRATION | Token expiration in seconds | No | 3600 |
| REDIS_HOST | Redis host | Yes | localhost |
| REDIS_PORT | Redis port | Yes | 6379 |

## API Documentation

### Authentication

#### POST /api/auth/register
Register a new user account.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!",
  "name": "John Doe"
}
```

**Response (201 Created):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 123,
    "email": "user@example.com",
    "name": "John Doe"
  }
}
```

#### POST /api/auth/login
Authenticate with email and password.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!"
}
```

**Response (200 OK):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 123,
    "email": "user@example.com",
    "name": "John Doe"
  }
}
```

### Tasks

#### GET /api/tasks
Get all tasks for authenticated user.

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "title": "Complete project",
    "description": "Finish the task manager application",
    "completed": false,
    "createdAt": "2024-01-10T10:00:00.000Z",
    "updatedAt": "2024-01-10T10:00:00.000Z"
  }
]
```

#### POST /api/tasks
Create a new task.

**Headers:**
```
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "title": "New task",
  "description": "Task description",
  "completed": false
}
```

**Response (201 Created):**
```json
{
  "id": 2,
  "title": "New task",
  "description": "Task description",
  "completed": false,
  "createdAt": "2024-01-10T10:00:00.000Z",
  "updatedAt": "2024-01-10T10:00:00.000Z"
}
```

#### PUT /api/tasks/:id
Update an existing task.

**Headers:**
```
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "title": "Updated task",
  "description": "Updated description",
  "completed": true
}
```

**Response (200 OK):**
```json
{
  "id": 2,
  "title": "Updated task",
  "description": "Updated description",
  "completed": true,
  "createdAt": "2024-01-10T10:00:00.000Z",
  "updatedAt": "2024-01-10T10:05:00.000Z"
}
```

#### DELETE /api/tasks/:id
Delete a task.

**Headers:**
```
Authorization: Bearer <token>
```

**Response (204 No Content):**
```
(empty response body)
```

## Error Responses

All error responses follow this format:

```json
{
  "error": "Error message description"
}
```

**Common Error Codes:**
- `400 Bad Request`: Invalid input data
- `401 Unauthorized`: Missing or invalid authentication
- `403 Forbidden`: Insufficient permissions
- `404 Not Found`: Resource not found
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Unexpected server error
```

**Add Inline Code Comments**

Add meaningful comments to explain complex logic, algorithms, and design decisions.

```javascript
/**
 * TaskRepository handles all database operations for tasks.
 * Implements the Repository pattern for data access abstraction.
 */
class TaskRepository {
  /**
   * Find all tasks for a specific user.
   * @param {number} userId - The user ID to filter tasks by
   * @returns {Promise<Array>} Array of tasks
   */
  async findAll(userId) {
    // Use parameterized query to prevent SQL injection
    const query = 'SELECT * FROM tasks WHERE user_id = ? ORDER BY created_at DESC';
    return await this.db.execute(query, [userId]);
  }

  /**
   * Create a new task.
   * Uses transaction to ensure data consistency.
   * @param {Object} taskData - Task data to insert
   * @returns {Promise<Object>} Created task with ID
   */
  async create(taskData) {
    const connection = await this.db.getConnection();
    
    try {
      await connection.beginTransaction();
      
      const [result] = await connection.execute(
        'INSERT INTO tasks (title, description, user_id, completed) VALUES (?, ?, ?, ?)',
        [taskData.title, taskData.description, taskData.userId, taskData.completed]
      );
      
      await connection.commit();
      
      return {
        id: result.insertId,
        ...taskData,
        createdAt: new Date().toISOString()
      };
    } catch (error) {
      await connection.rollback();
      throw error;
    } finally {
      connection.release();
    }
  }
}

// Commenting design decisions
// Chose JWT over sessions because:
// 1. Stateless architecture scales better horizontally
// 2. No server-side session storage required
// 3. Works well with mobile and single-page applications
// Trade-off: Cannot revoke tokens without blacklist (adds complexity)

// Chose PostgreSQL over MongoDB because:
// 1. ACID compliance required for task data integrity
// 2. Complex queries needed (filtering, sorting, aggregation)
// 3. Strong typing with TypeScript integration
// Trade-off: Schema migrations required (vs flexible schema)
```

**Document Design Pattern Usage**

Document which design patterns are used and why.

```javascript
/**
 * DESIGN PATTERNS USED
 * 
 * 1. Repository Pattern
 *    - Data access abstraction
 *    - Easier testing with mocks
 *    - Centralized query logic
 * 
 * 2. Factory Pattern
 *    - Task creation based on type
 *    - Extensible for new task types
 * 
 * 3. Strategy Pattern
 *    - Interchangeable sorting algorithms
 *    - Easy to add new sorting methods
 * 
 * 4. Singleton Pattern
 *    - Database connection pool
 *    - Logger instance
 *    - Redis client
 * 
 * 5. Observer Pattern
 *    - Real-time task updates via WebSocket
 *    - Event-driven architecture
 * 
 * 6. Dependency Injection
 *    - Pass dependencies via constructor
 *    - Easier testing and flexibility
 */

// Repository implementation
class TaskRepository { /* ... */ }

// Factory implementation
class TaskFactory {
  static create(type, data) { /* ... */ }
}

// Strategy implementation
class SortStrategy { /* ... */ }

// Singleton implementation
class Database {
  static instance = null;
  
  static getInstance() {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }
}

// Observer implementation
class TaskNotifier {
  constructor() {
    this.listeners = [];
  }
  
  subscribe(listener) {
    this.listeners.push(listener);
  }
  
  notify(task) {
    this.listeners.forEach(listener => listener(task));
  }
}

// Dependency injection
class TaskController {
  constructor(taskRepository, notifier) {
    this.taskRepository = taskRepository;
    this.notifier = notifier;
  }
}
```

### GitHub Presentation

**Clean Commit History**

Maintain a clean, meaningful commit history.

- **Meaningful Messages**: Describe what and why, not just what
- **Logical Commits**: One logical change per commit
- **Conventional Commits**: Use format (feat, fix, docs, refactor, test)
- **Atomic Changes**: Small, reviewable commits

```bash
# Conventional commit format
feat: add user authentication
fix: resolve race condition in task creation
docs: update API documentation
refactor: extract validation logic to separate module
test: add integration tests for task API

# Commit message examples
git commit -m "feat(auth): implement JWT-based authentication

- Add login and registration endpoints
- Implement token generation and validation
- Add refresh token mechanism
- Update middleware for protected routes

Closes #123"

git commit -m "fix(tasks): resolve race condition in task creation

- Add database transaction for task creation
- Implement retry logic for concurrent requests
- Add unit tests for edge cases

Fixes #456"
```

**Professional README**

Create a comprehensive README that sells the project.

- **Project Title**: Clear, descriptive name
- **Badges**: Build status, license, version
- **Screenshots/GIFs**: Visual demonstration of features
- **Features List**: Bullet points of key features
- **Tech Stack**: Technologies used with versions
- **Live Demo**: Link to deployed application
- **Getting Started**: Quick start guide
- **Contributing**: Guidelines for contributions

```markdown
# 📝 Task Manager

[![Build Status](https://img.shields.io/badge/build-passing-brightgreen)]
[![License](https://img.shields.io/badge/license-MIT-blue.svg)]
[![Version](https://img.shields.io/badge/version-1.0.0-orange.svg)]

A full-stack task management application with real-time updates, authentication, and a modern UI.

## ✨ Features

- ✅ User authentication with JWT
- ✅ Create, read, update, delete tasks
- ✅ Real-time task updates via WebSockets
- ✅ Task filtering and sorting
- ✅ Responsive design for mobile and desktop
- ✅ Dark mode support
- ✅ Input validation and sanitization
- ✅ Rate limiting and security headers
- ✅ Comprehensive error handling

## 🚀 Live Demo

[View Live Demo](https://task-manager.example.com)

## 📸 Screenshots

### Dashboard
![Dashboard](screenshots/dashboard.png)

### Task Creation
![Task Creation](screenshots/create-task.png)

### Task Management
![Task Management](screenshots/manage-tasks.png)

## 🛠️ Tech Stack

### Frontend
- **React 18** - UI framework
- **React Router 6** - Client-side routing
- **Tailwind CSS** - Styling
- **Axios** - HTTP client

### Backend
- **Node.js 18** - Runtime
- **Express 4** - Web framework
- **PostgreSQL 13** - Database
- **Redis 6** - Caching
- **JWT** - Authentication
- **Zod** - Validation

### DevOps
- **AWS EC2** - Backend hosting
- **AWS S3** - Frontend hosting
- **CloudFront** - CDN
- **GitHub Actions** - CI/CD
- **PM2** - Process management

## 📦 Installation

```bash
# Clone the repository
git clone https://github.com/username/task-manager.git
cd task-manager

# Install dependencies
npm install

# Configure environment variables
cp .env.example .env
# Edit .env with your configuration

# Setup database
npm run db:migrate

# Start development server
npm run dev
```

## 🏃 Running Tests

```bash
# Run unit tests
npm test

# Run integration tests
npm run test:integration

# Run e2e tests
npm run test:e2e

# Run with coverage
npm run test:coverage
```

## 📝 Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'feat: add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Commit Convention

We follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

- `feat:` A new feature
- `fix:` A bug fix
- `docs:` Documentation only changes
- `style:` Code style changes (formatting, etc.)
- `refactor:` Code refactoring
- `test:` Adding or updating tests
- `chore:` Maintenance tasks

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
```

---

## Interview Preparation

### Technical Narrative

**"Manager to Maker" Pivot Story**

Prepare a compelling narrative explaining your career transition from management to individual contribution.

- **Why Returning to IC**: Passion for hands-on work, desire for technical depth
- **Leadership Experience Value**: Communication, project management, mentoring skills transfer
- **Hands-On Impact Focus**: Recent projects, technical contributions, learning journey

```markdown
# My Journey: From Manager to Full-Stack Developer

## Background

For the past 5 years, I've managed teams of 5-15 developers, overseeing project delivery, code quality, and team growth. While I enjoyed mentoring and the strategic aspects of management, I found myself increasingly drawn to the technical work and missed the hands-on problem-solving that initially attracted me to software development.

## The Pivot Decision

### Why Return to Individual Contribution?

1. **Passion for Technology**: I want to stay current with rapidly evolving technologies. As a manager, I was overseeing but not deeply engaged with the day-to-day technical challenges.

2. **Desire for Technical Depth**: Management broadened my perspective but limited my technical depth. I want to specialize and become an expert in full-stack development.

3. **Hands-On Problem Solving**: I miss the satisfaction of solving complex technical problems and seeing my code in production.

4. **Market Demand**: The industry values deep technical expertise, and I want to position myself accordingly.

### Leadership Experience Value

My management experience isn't lost—it's an asset:

- **Communication Skills**: Translating technical concepts for stakeholders
- **Project Management**: Delivering projects on time and within budget
- **Team Development**: Mentoring junior developers, conducting code reviews
- **Strategic Thinking**: Making architectural decisions that balance business and technical needs

## Recent Technical Work

Since deciding to pivot, I've been intensively studying:

- **Full-Stack Development**: Built a task manager with React and Node.js
- **System Design**: Designed scalable architectures for web applications
- **Cloud Infrastructure**: Deployed applications on AWS (EC2, S3, Lambda)
- **Security**: Implemented JWT authentication, input validation, and security headers

## What I Bring

As a former manager returning to hands-on development, I offer:

1. **Communication**: Clear articulation of technical concepts
2. **Collaboration**: Experience working in and leading teams
3. **Business Context**: Understanding of how technology serves business goals
4. **Leadership**: Ability to mentor and guide when needed
5. **Fresh Perspective**: Eagerness to learn and adapt to new challenges

I'm excited to contribute as a senior full-stack engineer, combining my management experience with renewed technical expertise.
```

**Prepare 2-Minute Module Explanations**

Develop concise explanations for key technical concepts from each module.

- **Architecture Decisions**: Why you chose specific technologies or patterns
- **Security Implementations**: How you addressed security concerns
- **Design Pattern Choices**: When and why you used specific patterns
- **Performance Optimizations**: How you improved application performance

```markdown
# Module Explanations

## Module 1: JavaScript & Node.js Core

### Event Loop Architecture

The Node.js event loop enables non-blocking I/O operations, making it ideal for high-concurrency applications. I chose this architecture because it handles thousands of concurrent connections efficiently without thread overhead. The event loop processes tasks in phases: timers, pending callbacks, idle/prepare, poll, check, and close callbacks.

**Key Implementation:**
- Used `process.nextTick()` for high-priority operations
- Implemented `setImmediate()` for I/O callbacks
- Avoided blocking operations in the main thread
- Used worker threads for CPU-intensive tasks

### V8 Engine Optimization

Understanding V8's hidden classes and inline caching helped me optimize object creation. By maintaining consistent object shapes, I achieved 30% performance improvement in our data processing pipeline.

**Optimizations Applied:**
- Maintained consistent property order for hidden class reuse
- Used object pooling for frequently created objects
- Minimized property additions after object creation
- Used typed arrays for numeric data

## Module 2: Backend Development

### REST API Design

I implemented RESTful APIs following Richardson maturity level 2, using proper HTTP methods and status codes. This approach provides clear, predictable interfaces for clients and enables easy caching and proxying.

**Design Principles:**
- Resource-based URLs (`/api/tasks/:id` instead of `/api/getTask`)
- Proper HTTP verbs (GET, POST, PUT, DELETE)
- Meaningful status codes (200, 201, 204, 400, 401, 404, 500)
- Content negotiation (JSON responses with proper Content-Type headers)

### Middleware Architecture

Express middleware provides a clean way to implement cross-cutting concerns. I used middleware for authentication, logging, error handling, and rate limiting, keeping the route handlers focused on business logic.

**Middleware Stack:**
1. Request logging (request ID, method, URL)
2. Security headers (Helmet.js)
3. Rate limiting (100 req/15min per IP)
4. Body parsing (JSON, URL-encoded)
5. Authentication (JWT verification)
6. Route handler
7. Error handling (centralized)

## Module 3: React Fundamentals

### Virtual DOM and Reconciliation

React's virtual DOM enables efficient updates by batching DOM manipulations. I implemented reconciliation-aware components to minimize unnecessary re-renders, achieving 60fps animations.

**Optimizations:**
- Used `React.memo()` for expensive components
- Implemented `useMemo()` for computed values
- Used `useCallback()` for stable function references
- Added unique keys for list items
- Implemented code splitting with `React.lazy()`

### Custom Hooks

Created reusable custom hooks to extract stateful logic from components. This improved code reusability and testability.

**Custom Hooks Built:**
- `useAuth()`: Authentication state and login/logout actions
- `useTasks()`: Task CRUD operations with caching
- `useLocalStorage()`: Local storage synchronization
- `useDebounce()`: Debounced function calls
- `useWindowSize()`: Responsive design utilities

## Module 4: Security

### JWT Implementation

Implemented JWT-based authentication with access and refresh tokens. This stateless approach scales horizontally and works well with SPAs.

**Security Measures:**
- HS256 algorithm with strong secret (256+ bits)
- Short access token expiration (1 hour)
- Separate refresh token (7 days)
- Token rotation on refresh
- Blacklist for revoked tokens

### OWASP Top 10 Mitigation

Addressed OWASP Top 10 vulnerabilities:

1. **Injection**: Parameterized queries, input validation
2. **Broken Authentication**: Strong password policy, MFA support
3. **XSS**: Output encoding, CSP headers, DOMPurify
4. **CSRF**: CSRF tokens, SameSite cookies
5. **Security Misconfiguration**: Security headers, error sanitization

## Module 5: Data Structures & Algorithms

### Algorithm Selection

Chose algorithms based on time and space complexity for the problem constraints. For example, using hash maps for O(1) lookups instead of O(n) array searches.

**Patterns Applied:**
- Two-pointer technique for array problems
- Sliding window for subarray problems
- Hash maps for frequency counting
- Binary search for sorted data
- DFS/BFS for graph traversal

### LeetCode Practice

Completed 100+ LeetCode problems focusing on pattern recognition. Tracked time complexity and optimized solutions.

**Statistics:**
- Medium problems: 75 completed (avg 25 min)
- Hard problems: 25 completed (avg 45 min)
- Patterns mastered: 10 (sliding window, two pointers, BFS, DFS, etc.)

## Module 6: Design Patterns

### SOLID Principles

Refactored codebase to follow SOLID principles, improving maintainability and testability.

**Refactoring Examples:**
- Extracted validation logic (Single Responsibility)
- Created strategy pattern for sorting (Open/Closed)
- Implemented dependency injection (Dependency Inversion)
- Segregated interfaces (Interface Segregation)

### Pattern Applications

Applied appropriate design patterns for specific use cases:

- **Repository Pattern**: Data access abstraction
- **Factory Pattern**: Object creation based on type
- **Observer Pattern**: Event-driven updates
- **Singleton Pattern**: Database connection pool
- **Strategy Pattern**: Interchangeable algorithms

## Module 7: AWS Cloud

### Infrastructure as Code

Used CloudFormation for reproducible infrastructure deployment. This enables version control of infrastructure and easy rollback.

**Benefits:**
- Reproducible deployments across environments
- Version-controlled infrastructure changes
- Easy rollback to previous versions
- Reduced manual configuration errors

### Cost Optimization

Implemented cost optimization strategies:

- Reserved instances for steady workloads (40% savings)
- Spot instances for fault-tolerant batch jobs (70% savings)
- S3 lifecycle policies for data archival
- Auto-scaling to match capacity with demand

## Module 8: System Design

### Scalability Strategy

Designed systems for horizontal scalability using the scalability cube:

- **X-axis**: Cloning with load balancers
- **Y-axis**: Functional decomposition (microservices)
- **Z-axis**: Data partitioning (sharding)

### Caching Architecture

Implemented multi-layer caching:

- **CDN**: CloudFront for static assets
- **Application Cache**: Redis for API responses
- **Database Cache**: Query result caching
- **Browser Cache**: Cache-Control headers

## Module 9: Integration Projects

### Full-Stack Integration

Built a complete task manager application integrating all learned concepts:

- **Authentication**: JWT with refresh tokens
- **Authorization**: Role-based access control
- **API Design**: RESTful with proper status codes
- **Frontend**: React with Context API
- **Security**: OWASP mitigation, security headers
- **Deployment**: EC2 + S3 + CloudFront
- **Monitoring**: CloudWatch logging and metrics

### Trade-offs

**Database Choice**: PostgreSQL (ACID) vs MongoDB (flexible schema)
- Chose PostgreSQL for data integrity and complex queries
- Trade-off: Schema migrations required

**State Management**: Context API vs Redux
- Chose Context API for simplicity and built-in support
- Trade-off: Less powerful for complex state

**Deployment**: EC2 vs Lambda
- Chose EC2 for full control and familiarity
- Trade-off: Higher operational overhead vs serverless
```

### Common Interview Questions

**JavaScript/Node.js**

Prepare answers for common JavaScript and Node.js interview questions.

- **Event Loop**: Explain phases, microtasks vs macrotasks
- **Closures**: Lexical scoping, memory leaks, use cases
- **Async/Await vs Promises**: Syntax sugar, error handling, debugging
- **Node.js Scalability**: Event-driven architecture, worker threads, clustering

```javascript
// Event loop explanation
console.log('Start');
setTimeout(() => console.log('Timeout'), 0);
Promise.resolve().then(() => console.log('Promise'));
process.nextTick(() => console.log('NextTick'));
console.log('End');

// Output:
// Start
// End
// NextTick
// Promise
// Timeout

// Explanation:
// 1. Synchronous code runs first (Start, End)
// 2. process.nextTick() runs between phases (NextTick)
// 3. Promise callbacks run in microtask queue (Promise)
// 4. setTimeout callbacks run in macrotask queue (Timeout)

// Closure example
function createCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    getCount: () => count
  };
}

const counter = createCounter();
counter.increment();
console.log(counter.getCount()); // 1

// Closure maintains access to 'count' variable
// Used for data hiding, module patterns, function factories

// Memory leak example
// BAD: Event listener not cleaned up
function setupButton() {
  const button = document.getElementById('myButton');
  button.addEventListener('click', () => {
    console.log('Clicked');
  });
}
// When component unmounts, listener remains, causing memory leak

// GOOD: Clean up event listener
function setupButton() {
  const button = document.getElementById('myButton');
  const handler = () => console.log('Clicked');
  button.addEventListener('click', handler);
  
  // Return cleanup function
  return () => button.removeEventListener('click', handler);
}

// In React
useEffect(() => {
  const cleanup = setupButton();
  return cleanup; // Cleanup on unmount
}, []);
```

**React**

Prepare answers for React-specific questions.

- **Virtual DOM and Reconciliation**: Diffing algorithm, fiber architecture
- **Hooks Rules and Best Practices**: Call order, dependency arrays, custom hooks
- **Performance Optimization**: Memoization, code splitting, virtual scrolling
- **State Management Choices**: Context API vs Redux vs Zustand

```javascript
// Virtual DOM reconciliation
// React compares virtual DOM trees and updates only changed parts
// Example: List with 1000 items

function TaskList({ tasks }) {
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          {task.title}
        </li>
      ))}
    </ul>
  );
}

// Without keys: React recreates all 1000 items on any change
// With keys: React only updates changed items

// React.memo optimization
const TaskItem = React.memo(function TaskItem({ task }) {
  console.log('Rendering:', task.title);
  return <li>{task.title}</li>;
});

// Only re-renders if task prop changes
// Without React.memo: re-renders on every parent render

// Hooks rules
// BAD: Conditional hook call
if (someCondition) {
  const [state, setState] = useState(0);
}

// GOOD: Always call hooks in same order
function MyComponent({ someCondition }) {
  const [state, setState] = useState(0);
  
  if (someCondition) {
    // Use state conditionally
  return <div>{state}</div>;
  }
  
  return <div>Default</div>;
}

// Dependency arrays
// BAD: Missing dependency causes stale closure
useEffect(() => {
  fetchUser(userId).then(setUser);
}, []); // Missing userId dependency

// GOOD: Include all dependencies
useEffect(() => {
  fetchUser(userId).then(setUser);
}, [userId]); // Correct

// Custom hook rules
function useCustomHook() {
  const [value, setValue] = useState(0);
  
  // BAD: Conditional hook call
  if (value > 10) {
    const [extra, setExtra] = useState(0);
  }
  
  // GOOD: Always call hooks
  return { value, setValue };
}
```

**System Design**

Prepare for system design questions with structured approach.

- **Scalability Approaches**: Vertical vs horizontal, when to use each
- **Caching Strategies**: Cache-aside, write-through, CDN caching
- **Database Selection**: SQL vs NoSQL decision factors
- **Microservices vs Monolith**: Trade-offs and migration strategies

```
System Design Interview Framework:

1. Requirements Clarification (5-10 minutes)
   - Functional requirements (what to build)
   - Non-functional requirements (scale, availability, latency)
   - Constraints (traffic, data size)
   - Assumptions (document and confirm)

2. Capacity Estimation (5-10 minutes)
   - Requests per second (QPS)
   - Storage requirements (data × users × retention)
   - Bandwidth requirements (requests × data size)
   - Growth projections (1 year, 5 years)

3. High-Level Architecture (10-15 minutes)
   - Load balancers
   - Application servers
   - Caching layer
   - Databases
   - Message queues
   - CDN

4. Data Model Design (10-15 minutes)
   - Entities and relationships
   - Schema (SQL) or documents (NoSQL)
   - Indexes for query optimization
   - Partitioning strategy (if needed)

5. Detailed Component Design (15-20 minutes)
   - Choose 1-2 components to design deeply
   - APIs, algorithms, data structures
   - Scalability considerations
   - Trade-offs and alternatives

6. Bottleneck Identification (5-10 minutes)
   - Database: queries, connections, disk I/O
   - Cache: hit ratio, stampede
   - Network: bandwidth, latency
   - Application: CPU, memory, inefficient algorithms

7. Scalability Discussion (5-10 minutes)
   - Horizontal scaling strategy
   - Caching strategy
   - Database scaling (replicas, sharding)
   - CDN for static assets
   - Multi-region deployment
```

**Security**

Prepare for security-focused interview questions.

- **OWASP Top 10**: Explain each vulnerability and prevention
- **XSS/CSRF Prevention**: Implementation strategies
- **JWT Security**: Signing algorithms, key management, best practices
- **Authentication vs Authorization**: Difference and implementation

```javascript
// XSS prevention
// Input sanitization
const sanitize = (input) => {
  return input
    .replace(/&/g, '&')
    .replace(/</g, '<')
    .replace(/>/g, '>')
    .replace(/"/g, '"')
    .replace(/'/g, '&#x27;');
};

// Output encoding
const encode = (input) => {
  return input
    .replace(/&/g, '&')
    .replace(/</g, '<')
    .replace(/>/g, '>');
};

// Content Security Policy
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    styleSrc: ["'self'"],
    imgSrc: ["'self'", "data:"],
    connectSrc: ["'self'"],
    fontSrc: ["'self'"],
    objectSrc: ["'none'"],
    mediaSrc: ["'self'"],
    frameSrc: ["'none'"]
  }
}));

// CSRF protection
// Double submit cookie pattern
const csrfToken = crypto.randomBytes(32).toString('hex');
res.cookie('csrfToken', csrfToken, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict'
});

// Verify CSRF token
app.post('/api/tasks', (req, res) => {
  if (req.body.csrfToken !== req.cookies.csrfToken) {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }
  // Process request
});

// JWT security best practices
const jwt = require('jsonwebtoken');

// Use strong secret (256+ bits)
const secret = process.env.JWT_SECRET; // Must be cryptographically strong

// Set appropriate expiration
const token = jwt.sign(
  { userId: user.id },
  secret,
  { expiresIn: '1h' } // Short access token
);

// Verify algorithm in production
const decoded = jwt.verify(token, secret, {
  algorithms: ['HS256'] // Reject tokens with 'none' or weak algorithms
});

// Never include sensitive data in payload
const payload = { userId: user.id }; // Don't include: password, email, role

// Implement refresh token rotation
app.post('/api/auth/refresh', async (req, res) => {
  const { refreshToken } = req.body;
  
  // Verify refresh token
  const decoded = jwt.verify(refreshToken, REFRESH_SECRET);
  
  // Check if token is in blacklist
  if (await isTokenBlacklisted(refreshToken)) {
    return res.status(401).json({ error: 'Token revoked' });
  }
  
  // Generate new tokens
  const newAccessToken = jwt.sign({ userId: decoded.userId }, JWT_SECRET, { expiresIn: '1h' });
  const newRefreshToken = jwt.sign({ userId: decoded.userId }, REFRESH_SECRET, { expiresIn: '7d' });
  
  // Blacklist old refresh token
  await blacklistToken(refreshToken);
  
  res.json({ accessToken: newAccessToken, refreshToken: newRefreshToken });
});
```

**Design Patterns**

Prepare to explain 3 patterns with real-world use cases.

- **Explain 3 Patterns**: Choose patterns you've used
- **Use Cases**: When and why you used each pattern
- **SOLID Examples**: Show how you applied SOLID principles
- **When to Use/Not Use**: Discuss trade-offs

```javascript
// Pattern 1: Repository Pattern
// Use Case: Data access abstraction for testing and swapping databases

interface TaskRepository {
  findAll(userId): Promise<Task[]>;
  findById(id): Promise<Task>;
  create(taskData): Promise<Task>;
  update(id, taskData): Promise<void>;
  delete(id): Promise<void>;
}

class SQLTaskRepository implements TaskRepository {
  constructor(private db: Database) {}
  
  async findAll(userId) {
    return await this.db.query('SELECT * FROM tasks WHERE user_id = ?', [userId]);
  }
}

class MongoTaskRepository implements TaskRepository {
  constructor(private db: MongoClient) {}
  
  async findAll(userId) {
    return await this.db.collection('tasks').find({ userId }).toArray();
  }
}

// Benefits: Easy to swap databases, testable with mocks
// When to use: When you need to abstract data access
// When not to use: Simple CRUD with single database

// Pattern 2: Observer Pattern
// Use Case: Real-time updates across components

class TaskNotifier {
  private listeners: ((task: Task) => void)[] = [];
  
  subscribe(listener: (task: Task) => void) {
    this.listeners.push(listener);
  }
  
  unsubscribe(listener: (task: Task) => void) {
    this.listeners = this.listeners.filter(l => l !== listener);
  }
  
  notify(task: Task) {
    this.listeners.forEach(listener => listener(task));
  }
}

// React Context uses Observer pattern internally
const TaskContext = createContext();

function TaskProvider({ children }) {
  const [tasks, setTasks] = useState([]);
  const notifier = new TaskNotifier();
  
  useEffect(() => {
    const listener = (task) => {
      setTasks(prev => [...prev, task]);
    };
    
    notifier.subscribe(listener);
    
    return () => notifier.unsubscribe(listener);
  }, []);
  
  return (
    <TaskContext.Provider value={{ tasks }}>
      {children}
    </TaskContext.Provider>
  );
}

// Benefits: Loose coupling, event-driven, extensible
// When to use: Real-time updates, event systems, pub/sub
// When not to use: Simple synchronous operations

// Pattern 3: Strategy Pattern
// Use Case: Different sorting algorithms

interface SortStrategy {
  sort(tasks: Task[]): Task[];
}

class ByDateStrategy implements SortStrategy {
  sort(tasks: Task[]) {
    return [...tasks].sort((a, b) => 
      new Date(a.dueDate).getTime() - new Date(b.dueDate).getTime()
    );
  }
}

class ByPriorityStrategy implements SortStrategy {
  sort(tasks: Task[]) {
    const priority = { high: 3, medium: 2, low: 1 };
    return [...tasks].sort((a, b) => 
      priority[a.priority] - priority[b.priority]
    );
  }
}

// Usage
function sortTasks(tasks: Task[], strategy: SortStrategy) {
  return strategy.sort(tasks);
}

const byDate = new ByDateStrategy();
const byPriority = new ByPriorityStrategy();

sortTasks(tasks, byDate); // Sort by date
sortTasks(tasks, byPriority); // Sort by priority

// Benefits: Open/closed principle, interchangeable algorithms
// When to use: Multiple algorithms for same operation, runtime selection
// When not to use: Single, fixed algorithm

// SOLID Examples

// Single Responsibility Principle
// BAD: Controller does validation, database, email, logging
class BadController {
  async createTask(req, res) {
    // Validation
    if (!req.body.title) return res.status(400).send('Invalid');
    
    // Database
    const task = await db.insert(req.body);
    
    // Email
    await email.send(req.user.email, 'Task created');
    
    // Logging
    logger.info('Task created', task);
  }
}

// GOOD: Each class has single responsibility
class Validator {
  validate(data) { /* ... */ }
}

class Repository {
  async insert(data) { /* ... */ }
}

class EmailService {
  async send(to, subject) { /* ... */ }
}

class Logger {
  info(message, data) { /* ... */ }
}

class Controller {
  constructor(validator, repository, email, logger) {
    this.validator = validator;
    this.repository = repository;
    this.email = email;
    this.logger = logger;
  }
  
  async createTask(req, res) {
    const errors = this.validator.validate(req.body);
    if (errors) return res.status(400).json({ errors });
    
    const task = await this.repository.insert(req.body);
    await this.email.send(req.user.email, 'Task created');
    this.logger.info('Task created', task);
    
    res.json(task);
  }
}

// Open/Closed Principle
// BAD: Modify existing class for new feature
class TaskSorter {
  sort(tasks) {
    return tasks.sort((a, b) => a.title.localeCompare(b.title));
  }
}

// Add new sort method requires modifying class
class TaskSorter {
  sort(tasks) {
    return tasks.sort((a, b) => a.title.localeCompare(b.title));
  }
  
  sortByDate(tasks) {
    return tasks.sort((a, b) => new Date(a.dueDate) - new Date(b.dueDate));
  }
}

// GOOD: Extend without modification (Strategy pattern)
interface SortStrategy {
  sort(tasks): Task[];
}

class ByTitleStrategy implements SortStrategy {
  sort(tasks) { return tasks.sort((a, b) => a.title.localeCompare(b.title)); }
}

class ByDateStrategy implements SortStrategy {
  sort(tasks) { return tasks.sort((a, b) => new Date(a.dueDate) - new Date(b.dueDate)); }
}

// Add new sort by creating new strategy
class ByPriorityStrategy implements SortStrategy {
  sort(tasks) { /* ... */ }
}
```

### Mock Interview Practice

**System Design Whiteboarding**

Practice explaining system designs visually and verbally.

- **Draw Architecture**: Clear diagrams with labels
- **Explain Components**: Purpose of each component
- **Discuss Trade-offs**: Why you chose specific approaches
- **Identify Bottlenecks**: Potential limitations and mitigations

```
Whiteboarding Framework:

1. Clarify Requirements (2-3 min)
   "Before I design, I'd like to clarify a few things:
   - How many daily active users?
   - What's the read/write ratio?
   - Any specific latency requirements?
   - Expected growth rate?"

2. High-Level Design (3-5 min)
   [Draw diagram while explaining]
   "I'll design with a load balancer, application servers, cache layer, and database.
   The load balancer distributes traffic across 3 app servers for scalability.
   Redis cache sits between app servers and database to reduce load."

3. Data Model (2-3 min)
   [Draw ER diagram or document structure]
   "For the data model, I'll have users table and tasks table.
   Tasks table will have user_id foreign key, with indexes on created_at and status."

4. Deep Dive (5-8 min)
   [Choose 1-2 components]
   "Let me dive deeper into the cache invalidation strategy.
   I'll use cache-aside pattern with write-through for consistency.
   Cache keys will include user_id and task_id for fine-grained invalidation."

5. Bottlenecks & Scalability (2-3 min)
   "Potential bottlenecks:
   - Database at high write throughput
   - Cache stampede on popular tasks
   
   Mitigations:
   - Read replicas for scaling reads
   - Write-behind cache for high writes
   - Consistent hashing for cache distribution"

6. Follow-up Questions (2-3 min)
   "Do you have any questions about the architecture or trade-offs?"
```

**Coding Problem Verbalization**

Practice solving coding problems while explaining your thought process.

- **Think Aloud**: Explain approach before coding
- **Ask Questions**: Clarify constraints and edge cases
- **Start Simple**: Brute force, then optimize
- **Test Examples**: Walk through test cases manually

```javascript
// Example: Two Sum problem
// Verbalization process:

// Interviewer: "Given an array of integers and a target, find two numbers that add up to target."

// Candidate: "Let me think through this...
// 
// First, I'll clarify the requirements:
// - Are there duplicate numbers in the array?
// - Should I return all pairs or just one?
// - What if no solution exists?
// - Are the numbers sorted?
// 
// Assuming: Return one pair, unsorted array, return null if no solution.
// 
// Approach 1: Brute force O(n²)
// For each number, check every other number to see if they sum to target.
// This works but is slow for large arrays.
// 
// Approach 2: Hash map O(n)
// I'll iterate through the array once, storing each number in a hash map.
// For each number, I'll check if (target - number) exists in the map.
// If it does and it's not the same index, I found a pair.
// This is O(n) time and O(n) space.
// 
// Let me code the hash map approach:"

function twoSum(nums, target) {
  // Create hash map to store number -> index
  const map = new Map();
  
  // Iterate through array
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    
    // Check if complement exists in map
    if (map.has(complement)) {
      // Found pair
      return [map.get(complement), i];
    }
    
    // Store current number and index
    map.set(nums[i], i);
  }
  
  // No solution found
  return null;
}

// Test with examples:
// twoSum([2, 7, 11, 15], 9) → [0, 1] (2 + 7 = 9)
// twoSum([3, 2, 4], 6) → [1, 2] (2 + 4 = 6)
// twoSum([1, 2, 3], 7) → null (no pair sums to 7)
```

**Trade-off Discussions**

Practice discussing trade-offs for different approaches.

- **Pros and Cons**: List advantages and disadvantages
- **Context Matters**: When to use each approach
- **Be Decisive**: Choose approach and justify it
- **Admit Unknowns**: It's okay to say "I'm not sure, let me think"

```
Trade-off Discussion Framework:

1. State the Options
   "For this problem, I see two main approaches:
   Option A: SQL database with ACID transactions
   Option B: NoSQL database with eventual consistency"

2. Compare Trade-offs
   "Option A pros:
   - Strong consistency guarantees
   - Complex queries with joins
   - Mature tooling
   
   Option A cons:
   - Vertical scaling limits
   - Schema migrations required
   - Higher cost at scale
   
   Option B pros:
   - Horizontal scaling built-in
   - Flexible schema
   - Lower cost at scale
   
   Option B cons:
   - Eventual consistency
   - Limited query capabilities
   - Less mature tooling"

3. Make Recommendation
   "Given the requirements:
   - Need for complex queries (filtering, sorting, aggregation)
   - Data consistency is critical (tasks must not be lost)
   - Expected moderate scale (not millions of users initially)
   
   I recommend Option A: SQL database.
   The strong consistency and query capabilities outweigh the scaling limitations for our current use case.
   We can migrate to NoSQL if we hit scale limits."

4. Mention Mitigations
   "To address the scaling concerns with SQL:
   - Implement read replicas for scaling reads
   - Use caching layer (Redis) to reduce database load
   - Implement connection pooling
   - Plan for sharding if needed"
```

**Behavioral Questions (STAR Method)**

Use the STAR method for behavioral questions.

- **Situation**: Context and background
- **Task**: What you needed to accomplish
- **Action**: What you did (specific steps)
- **Result**: Outcome and what you learned

```markdown
# STAR Method Examples

## Question: Tell me about a time you had to deal with a difficult team member.

### Situation
"In my previous role as a tech lead, I had a team member who consistently missed deadlines and delivered low-quality code that caused production issues."

### Task
"My task was to ensure the team delivered high-quality features on time while maintaining team morale. I needed to address the performance issue without demotivating the team member."

### Action
"I took a multi-step approach:
1. **Private Conversation**: Scheduled a 1-on-1 meeting to understand their challenges
2. **Root Cause Analysis**: Discovered they were overwhelmed with personal issues and lacked proper onboarding
3. **Support Plan**: 
   - Reduced their workload by 50% for 2 weeks
   - Paired them with a senior developer for code reviews
   - Provided additional documentation and examples
   - Scheduled daily stand-ups to identify blockers early
4. **Performance Improvement Process**:
   - Created a checklist for PR reviews
   - Implemented automated testing to catch issues early
   - Set up weekly 1-on-1s to track progress"

### Result
"Within 3 weeks, their code quality improved significantly:
- PR review time reduced from 2 hours to 30 minutes
- Production bugs from their code dropped from 5 per sprint to 0
- They expressed appreciation for the support and became a strong contributor
- Team velocity increased by 40% as blockers were identified earlier

I learned that performance issues often stem from personal challenges or lack of clarity, and that supportive leadership can transform struggling team members into strong contributors."

---

## Question: Describe a time you made a mistake and how you handled it.

### Situation
"While working on a critical payment processing feature, I implemented a caching layer to improve performance."

### Task
"The task was to reduce API response time from 500ms to under 100ms for payment confirmation endpoints."

### Action
"I implemented Redis caching with a 5-minute TTL. However, I made a critical error: I cached payment status without invalidating it when the status changed.

When the bug was discovered:
1. **Immediate Mitigation**: Disabled caching for payment endpoints
2. **Root Cause Analysis**: Found that cached stale status caused incorrect payment confirmations
3. **Fix Implementation**: 
   - Implemented cache invalidation on status changes
   - Added versioning to cache keys
   - Added monitoring for cache hit/miss ratios
4. **Process Improvement**:
   - Added integration tests for cache behavior
   - Created code review checklist for caching logic
   - Documented caching best practices for the team"

### Result
"The fix was deployed within 4 hours:
- No further stale cache issues reported
- Response time maintained at 80ms (met the goal)
- Added monitoring showed 95% cache hit rate

I learned the importance of:
- Cache invalidation strategy is as important as caching itself
- Integration tests should cover cache behavior
- Always consider data consistency when implementing performance optimizations"
```

---

## Final Review Checklist

### Technical Skills

**Node.js: Event Loop, Express, Streams, Middleware**

- **Event Loop**: Understand phases, microtasks vs macrotasks, async patterns
- **Express**: Routing, middleware, error handling, security
- **Streams**: Readable, writable, transform streams, backpressure
- **Middleware**: Authentication, logging, validation, rate limiting

**React: Components, Hooks, Context API, Routing**

- **Components**: Functional components, lifecycle, memoization
- **Hooks**: useState, useEffect, useCallback, useMemo, custom hooks
- **Context API**: Provider/consumer pattern, performance optimization
- **Routing**: React Router, protected routes, lazy loading

**DS/Algo: Arrays, Strings, Trees, Graphs, Sorting**

- **Arrays**: Two-pointer, sliding window, prefix sums
- **Strings**: Pattern matching, substring search, character counting
- **Trees**: Traversals (DFS, BFS), BST operations
- **Graphs**: Shortest path, topological sort, connected components
- **Sorting**: Merge sort, quick sort, heap sort

**Security: OWASP Top 10, XSS, CSRF, JWT**

- **OWASP**: Injection, broken auth, XSS, broken access control
- **XSS**: Input sanitization, output encoding, CSP
- **CSRF**: Tokens, SameSite cookies, double submit
- **JWT**: Signing algorithms, key management, token rotation

**Design Patterns: Singleton, Factory, Strategy, Observer**

- **Singleton**: Database pools, logger instances, configuration
- **Factory**: Object creation, service instantiation
- **Strategy**: Interchangeable algorithms, runtime selection
- **Observer**: Event systems, React state, pub/sub

**AWS: EC2, S3, Lambda, IAM, Deployment**

- **EC2**: Instance types, security groups, Auto Scaling
- **S3**: Storage classes, lifecycle policies, static hosting
- **Lambda**: Serverless functions, triggers, cold starts
- **IAM**: Users, roles, policies, least privilege
- **Deployment**: CI/CD pipelines, blue-green, canary

**System Design: Scalability, Caching, Load Balancing**

- **Scalability**: Horizontal vs vertical, consistency models
- **Caching**: Cache-aside, CDN, Redis, invalidation strategies
- **Load Balancing**: Algorithms, health checks, sticky sessions
- **Database**: SQL vs NoSQL, sharding, replication

### Practical Projects

**REST API with Authentication**

- Implemented JWT-based authentication with refresh tokens
- Created RESTful endpoints with proper status codes
- Added input validation with Zod
- Implemented rate limiting and security headers
- Added comprehensive error handling and logging

**React Application with Routing**

- Built React app with Context API for state management
- Implemented protected routes with route guards
- Created custom hooks (useAuth, useTasks)
- Added loading and error states
- Implemented code splitting with React.lazy()

**Secure Full-Stack Application**

- Integrated OWASP Top 10 mitigations
- Implemented XSS and CSRF prevention
- Added security headers with Helmet.js
- Implemented secure password hashing with bcrypt
- Added input sanitization and validation

**Deployed Application on AWS**

- Deployed backend to EC2 with PM2
- Deployed frontend to S3 + CloudFront
- Configured HTTPS with ACM certificates
- Set up CloudWatch logging and metrics
- Implemented CI/CD with GitHub Actions

**Well-Documented GitHub Repository**

- Comprehensive README with setup instructions
- API documentation with examples
- Clean commit history with conventional commits
- Screenshots and live demo links
- Contributing guidelines and license

### Interview Readiness

**Prepared Technical Narrative**

- Drafted "Manager to Maker" pivot story
- Created 2-minute explanations for each module
- Documented architecture decisions and trade-offs
- Prepared examples of design pattern usage

**Practice Explaining Architecture Decisions**

- System design whiteboarding practice
- Scalability discussions with trade-offs
- Database selection rationale
- Caching strategy explanations
- Microservices vs monolith comparisons

**Mock Interview Sessions Completed**

- System design practice (URL shortener, chat system)
- Coding problem verbalization
- Behavioral questions with STAR method
- Trade-off discussions for technical choices
- Time management during practice sessions

**Confidence in Discussing Trade-offs**

- Understand pros and cons of different approaches
- Can justify technical decisions with reasoning
- Aware of limitations and when to scale
- Comfortable admitting unknowns and proposing research
- Can discuss both technical and business implications
