# Module 4: Web Security Fundamentals

## OWASP Top 10 Security Risks

### Injection Attacks

**SQL Injection**

SQL injection occurs when untrusted user input is concatenated into SQL queries without proper sanitization, allowing attackers to manipulate database queries. Attackers can bypass authentication, extract sensitive data, or modify/delete database records. Prevention involves using parameterized queries, prepared statements, or ORM libraries that automatically handle escaping. Input validation and sanitization provide additional layers of defense.

```javascript
// ❌ Vulnerable: Direct string concatenation
app.get('/users/:id', (req, res) => {
  const query = `SELECT * FROM users WHERE id = ${req.params.id}`;
  db.execute(query); // Vulnerable to injection
});

// ✅ Secure: Parameterized queries
app.get('/users/:id', (req, res) => {
  const query = 'SELECT * FROM users WHERE id = ?';
  db.execute(query, [req.params.id]); // Safe from injection
});

// ✅ Secure: Using ORM with automatic escaping
const { Sequelize } = require('sequelize');
const User = Sequelize.define('User', {
  name: Sequelize.STRING,
  email: Sequelize.STRING
});

// ORM automatically parameterizes queries
const user = await User.findByPk(userId); // Safe
```

**NoSQL Injection**

NoSQL injection targets NoSQL databases like MongoDB by manipulating query objects instead of SQL strings. Attackers inject operators like `$ne`, `$gt`, or `$regex` to bypass authentication or access unauthorized data. Prevention includes strict input validation, type checking, and using safe query builders that sanitize operators.

```javascript
// ❌ Vulnerable: Operator injection in MongoDB
app.get('/users', (req, res) => {
  const { username, password } = req.query;
  const query = { username, password };
  db.collection('users').findOne(query); // Vulnerable if username contains operators
});

// ✅ Secure: Type checking and validation
const { Schema } = require('mongoose');
const userSchema = new Schema({
  username: { type: String, validate: { validator: v => /^[a-zA-Z0-9]+$/ } },
  password: { type: String, minlength: 8 }
});

const User = mongoose.model('User', userSchema);
const user = await User.findOne({ username, password }); // Safe with validation
```

**Command Injection**

Command injection occurs when application executes system commands based on untrusted user input without proper validation. Attackers can execute arbitrary commands, access file systems, or escalate privileges. Prevention involves avoiding shell execution of user input, using allowlists of permitted commands, and employing proper input sanitization.

```javascript
// ❌ Vulnerable: Direct command execution
const { exec } = require('child_process');
app.post('/ping', (req, res) => {
  const { host } = req.body;
  exec(`ping -c 1 ${host}`, (error, stdout, stderr) => {
    // Vulnerable to command injection
    res.json({ output: stdout });
  });
});

// ✅ Secure: Input validation and allowlisting
const allowedHosts = ['localhost', '127.0.0.1'];

app.post('/ping', (req, res) => {
  const { host } = req.body;
  
  if (!allowedHosts.includes(host)) {
    return res.status(400).json({ error: 'Invalid host' });
  }
  
  // Use safe command construction
  const { spawn } = require('child_process');
  const ping = spawn('ping', ['-c', '1', host]);
  
  ping.on('close', (code) => {
    res.json({ output: `Exit code: ${code}` });
  });
});
```

---

### Broken Authentication

**Session Management**

Broken authentication vulnerabilities include weak session management, session fixation attacks, and improper session invalidation. Session fixation allows attackers to set a known session ID before authentication, then hijack the authenticated session. Prevention involves regenerating session IDs after login, using secure cookies with HttpOnly and Secure flags, and implementing proper session timeouts.

```javascript
// ❌ Vulnerable: Session fixation
app.post('/login', (req, res) => {
  const sessionId = req.query.sessionId;
  
  // Attacker provides session ID
  req.session.id = sessionId;
  
  req.session.user = { id: 1 };
  res.json({ message: 'Logged in' });
});

// ✅ Secure: Regenerate session after login
const session = require('express-session');

app.use(session({
  secret: process.env.SESSION_SECRET,
  name: 'sessionId',
  cookie: {
    secure: process.env.NODE_ENV === 'production', // HTTPS only
    httpOnly: true, // Prevent XSS access
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
    sameSite: 'strict' // CSRF protection
  }
}));

app.post('/login', (req, res) => {
  const { username, password } = req.body;
  const user = authenticate(username, password);
  
  if (user) {
    // Regenerate session ID to prevent fixation
    req.session.regenerate((err) => {
      if (err) {
        return res.status(500).json({ error: 'Session error' });
      }
      
      req.session.userId = user.id;
      res.json({ message: 'Logged in' });
    });
  } else {
    res.status(401).json({ error: 'Invalid credentials' });
  }
});
```

**Credential Stuffing**

Credential stuffing attacks use automated scripts to test stolen credentials across multiple websites. Attackers exploit password reuse by users who use the same password across services. Prevention involves implementing rate limiting, account lockout policies, multi-factor authentication, and monitoring for suspicious login patterns.

```javascript
// ✅ Secure: Rate limiting for login attempts
const rateLimit = require('express-rate-limit');

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts per window
  message: 'Too many login attempts, please try again later',
  skipSuccessfulRequests: true // Don't count successful logins
});

app.post('/login', loginLimiter, (req, res) => {
  const { username, password } = req.body;
  const user = authenticate(username, password);
  
  if (user) {
    res.json({ message: 'Logged in' });
  } else {
    res.status(401).json({ error: 'Invalid credentials' });
  }
});

// ✅ Secure: Account lockout after failed attempts
const failedAttempts = new Map();

app.post('/login', (req, res) => {
  const { username, password } = req.body;
  const user = authenticate(username, password);
  
  if (!user) {
    const attempts = failedAttempts.get(username) || 0;
    
    if (attempts >= 5) {
      // Lock account for 30 minutes
      failedAttempts.set(username, { count: attempts, lockedUntil: Date.now() + 30 * 60 * 1000 });
      
      return res.status(429).json({ 
        error: 'Account locked due to too many failed attempts. Try again in 30 minutes.',
        retryAfter: 1800 // 30 minutes in seconds
      });
    }
    
    failedAttempts.set(username, { count: attempts + 1, lockedUntil: null });
    res.status(401).json({ error: 'Invalid credentials' });
  } else {
    failedAttempts.delete(username);
    res.json({ message: 'Logged in' });
  }
});
```

---

### Sensitive Data Exposure

**Data at Rest**

Data at rest refers to sensitive information stored in databases, files, or memory when not actively used. Encryption at rest protects data if storage media is compromised or accessed without authorization. Best practices include using strong encryption algorithms (AES-256), proper key management, and encrypting sensitive fields individually rather than entire databases.

```javascript
// ❌ Vulnerable: Storing passwords in plaintext
const crypto = require('crypto');

function hashPassword(password) {
  return crypto.createHash('sha256').update(password).digest('hex');
}

// Store password hash instead of plaintext
const user = {
  username: 'john',
  password: hashPassword('secret123') // Better than plaintext
};

// ❌ Vulnerable: Weak encryption
const encrypted = crypto.createCipheriv('aes-128-cbc', 'weak-key', iv);
let encryptedData = encrypted.update(data, 'utf8').final();

// ✅ Secure: Strong encryption with AES-256-GCM
const crypto = require('crypto');

function encryptData(data, key) {
  const algorithm = 'aes-256-gcm';
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv(algorithm, key, iv);
  
  let encrypted = cipher.update(data, 'utf8');
  encrypted = Buffer.concat([encrypted, cipher.final()]);
  
  return {
    encryptedData: encrypted,
    iv: iv,
    authTag: cipher.getAuthTag() // GCM provides integrity
  };
}

// ✅ Secure: Use environment variables for encryption keys
const algorithm = 'aes-256-gcm';
const key = Buffer.from(process.env.ENCRYPTION_KEY, 'hex'); // Load from environment
const iv = crypto.randomBytes(16);
const cipher = crypto.createCipheriv(algorithm, key, iv);
```

**Data in Transit**

Data in transit protection ensures sensitive data is encrypted during transmission between client and server. HTTPS/TLS provides encryption, authentication, and integrity verification. Best practices include using strong TLS configurations (TLS 1.2+), enforcing HTTPS, implementing HSTS headers, and properly configuring certificates.

```javascript
// ✅ Secure: Enforce HTTPS in Express
const helmet = require('helmet');

app.use(helmet.hsts({
  maxAge: 31536000, // 1 year in seconds
  includeSubDomains: true,
  preload: true
}));

// ✅ Secure: Configure TLS with strong ciphers
const https = require('https');

const server = https.createServer({
  key: fs.readFileSync('private-key.pem'),
  cert: fs.readFileSync('certificate.pem'),
  ciphers: 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256' // Strong ciphers
});

// ✅ Secure: Proper certificate management
const options = {
  key: fs.readFileSync('private-key.pem'),
  cert: fs.readFileSync('certificate.pem'),
  ca: fs.readFileSync('ca-bundle.crt'), // Certificate chain
  minVersion: 'TLSv1.2', // Minimum TLS version
  ciphers: 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384'
};
```

---

### Cross-Site Scripting (XSS)

**Stored XSS**

Stored XSS occurs when malicious scripts are permanently stored on a server and executed when users access affected pages. Attackers inject scripts through user-generated content like comments, posts, or profiles. Prevention involves output encoding, input sanitization, and Content Security Policy (CSP) headers.

```javascript
// ❌ Vulnerable: Rendering unsanitized user content
app.get('/comments', async (req, res) => {
  const comments = await db.query('SELECT * FROM comments');
  
  res.send(`
    <html>
      <body>
        <h1>Comments</h1>
        ${comments.map(c => `<div>${c.text}</div>`).join('')}
      </body>
    </html>
  `); // Vulnerable to XSS
});

// ✅ Secure: Output encoding
const escapeHtml = (unsafe) => {
  return unsafe
    .replace(/&/g, '&')
    .replace(/</g, '<')
    .replace(/>/g, '>')
    .replace(/"/g, '"')
    .replace(/'/g, ''');
};

app.get('/comments', async (req, res) => {
  const comments = await db.query('SELECT * FROM comments');
  
  res.send(`
    <html>
      <body>
        <h1>Comments</h1>
        ${comments.map(c => `<div>${escapeHtml(c.text)}</div>`).join('')}
      </body>
    </html>
  `); // Safe from XSS
});

// ✅ Secure: Using DOMPurify library
const createDOMPurify = require('isomorphic-dompurify');
const dompurify = createDOMPurify();

app.get('/comments', async (req, res) => {
  const comments = await db.query('SELECT * FROM comments');
  
  res.send(`
    <html>
      <body>
        <h1>Comments</h1>
        ${comments.map(c => `<div>${dompurify.sanitize(c.text)}</div>`).join('')}
      </body>
    </html>
  `); // Safe from XSS
});
```

**Reflected XSS**

Reflected XSS occurs when malicious scripts are reflected back to users in HTTP responses without proper encoding. Attackers craft malicious URLs containing scripts that execute when clicked. Prevention involves URL encoding, output encoding, and validating and sanitizing all input parameters.

```javascript
// ❌ Vulnerable: Reflecting unsanitized input
app.get('/search', (req, res) => {
  const query = req.query.q;
  
  res.send(`
    <html>
      <body>
        <h1>Search Results</h1>
        <p>You searched for: ${query}</p>
      </body>
    </html>
  `); // Vulnerable to XSS if query contains scripts
});

// ✅ Secure: URL encoding and output encoding
const querystring = require('querystring');

app.get('/search', (req, res) => {
  const query = req.query.q;
  
  // Encode URL parameter
  const encodedQuery = encodeURIComponent(query);
  
  // Output encode
  const safeQuery = query
    .replace(/</g, '<')
    .replace(/>/g, '>')
    .replace(/"/g, '"');
  
  res.send(`
    <html>
      <body>
        <h1>Search Results</h1>
        <p>You searched for: ${safeQuery}</p>
      </body>
    </html>
  `); // Safe from XSS
});
```

**DOM-based XSS**

DOM-based XSS occurs when malicious scripts manipulate the DOM directly in the browser, typically through unsafe JavaScript APIs like `innerHTML`, `eval()`, or `document.write()`. Prevention involves avoiding unsafe APIs, using safe alternatives like `textContent`, and validating data sources.

```javascript
// ❌ Vulnerable: Using innerHTML with user input
function renderUserInput(input) {
  const container = document.getElementById('container');
  container.innerHTML = `<div>User input: ${input}</div>`; // XSS vulnerability
}

// ✅ Secure: Using textContent instead
function renderUserInput(input) {
  const container = document.getElementById('container');
  container.textContent = `User input: ${input}`; // Safe, no script execution
}

// ❌ Vulnerable: Using eval() with user input
function evaluateInput(input) {
  eval(`console.log('${input}')`); // Dangerous! Executes arbitrary code
}

// ✅ Secure: Using JSON.parse or safe alternatives
function parseUserInput(input) {
  try {
    return JSON.parse(input); // Safe if input is valid JSON
  } catch {
    console.error('Invalid input');
  }
}
```

---

### Cross-Site Request Forgery (CSRF)

**CSRF Attack Mechanics**

CSRF attacks trick authenticated users into performing unwanted actions on a website where they're already authenticated. Attackers create malicious pages that submit forms to target sites using user's browser session cookies. Prevention involves CSRF tokens, SameSite cookie attributes, and verifying request origins.

```
┌─────────────────────────────────────────────┐
│                                             │
│  Attacker's Site                            │
│  ┌───────────────────────────────────────┐ │
│  │ <form action="https://bank.com/transfer">│
│  │   <input type="hidden"           │
│  │         name="amount"            │
│  │         value="10000">           │
│  │   <input type="hidden"           │
│  │         name="to"               │
│  │         value="attacker">         │
│  │   <button>Transfer</button>    │
│  └───────────────────────────────────────┘ │
│                                             │
└─────────────────────────────────────────────┘
                    │
                    │
                    ↓
        User's Browser
                    │
                    │
              ┌───────────────────────────────┐
              │                             │
              │  Victim visits attacker's page │
              │  Form submits automatically     │
              │  with user's cookies          │
              │                             │
              └───────────────────────────────┘
```

**CSRF Prevention**

CSRF tokens are unpredictable values generated by the server and included in forms, then validated on submission. SameSite cookies restrict when cookies are sent with cross-site requests, preventing CSRF attacks. Double-submit cookies provide additional protection by ensuring forms can only be submitted once.

```javascript
// ✅ Secure: CSRF token generation and validation
const crypto = require('crypto');

function generateCSRFToken() {
  return crypto.randomBytes(32).toString('hex');
}

// Store token in session
app.use((req, res, next) => {
  req.session.csrfToken = generateCSRFToken();
  next();
});

// Include token in forms
app.get('/transfer', (req, res) => {
  res.send(`
    <form action="/transfer" method="POST">
      <input type="hidden" name="csrf_token" value="${req.session.csrfToken}">
      <input type="text" name="amount">
      <input type="text" name="to">
      <button>Transfer</button>
    </form>
  `);
});

// Validate token on submission
app.post('/transfer', (req, res) => {
  const { csrf_token, amount, to } = req.body;
  
  if (!req.session.csrfToken || csrf_token !== req.session.csrfToken) {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }
  
  // Process transfer
  res.json({ message: 'Transfer complete' });
});

// ✅ Secure: SameSite cookie configuration
const session = require('express-session');

app.use(session({
  secret: process.env.SESSION_SECRET,
  cookie: {
    sameSite: 'strict', // Prevent CSRF
    secure: true, // HTTPS only
    httpOnly: true
  }
}));

// ✅ Secure: Double-submit cookie pattern
app.use((req, res, next) => {
  const token = generateCSRFToken();
  
  res.cookie('csrf_token', token, {
    httpOnly: true,
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000
  });
  
  req.session.csrfToken = token;
  next();
});
```

---

## Authentication & Authorization

### JWT Internals

**JWT Structure**

JSON Web Tokens (JWT) are compact, URL-safe means of representing claims between two parties. A JWT consists of three parts separated by dots: header, payload, and signature. The header specifies the algorithm used, the payload contains claims (user ID, expiration), and the signature ensures the token hasn't been tampered with. JWTs are stateless, making them ideal for distributed systems and microservices.

```
JWT Structure:
┌─────────────────────────────────────────────────────────┐
│  Header (base64url encoded JSON)              │
│  ┌───────────────────────────────────────────────┐ │
│  │ {"alg":"HS256","typ":"JWT"}          │ │
│  └───────────────────────────────────────────────┘ │
│                       .                             │
│                       Payload (base64url encoded JSON)  │
│  ┌───────────────────────────────────────────────┐ │
│  │ {"sub":"123","exp":1234567890,"iat":...│ │
│  │ {"iss":"myapp","role":"admin"}          │ │
│  └───────────────────────────────────────────────┘ │
│                       .                             │
│                       Signature (base64url encoded)        │
│  ┌───────────────────────────────────────────────┐ │
│  │ HMACSHA256(key, header + "." + payload)  │ │
│  └───────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘

Complete Token:
header.payload.signature
```

**HS256 vs RS256**

HS256 uses HMAC SHA-256 with a shared secret key for both signing and verification. It's simpler to implement and suitable for internal services where secret can be securely shared. RS256 uses RSA asymmetric encryption with private key for signing and public key for verification. It's more complex but provides better security for public-facing APIs and enables key rotation without sharing secrets.

```javascript
// HS256: Shared secret (simpler, faster)
const jwt = require('jsonwebtoken');
const secret = process.env.JWT_SECRET; // Shared secret

function generateHS256Token(payload) {
  return jwt.sign(payload, secret, { algorithm: 'HS256', expiresIn: '1h' });
}

function verifyHS256Token(token) {
  return jwt.verify(token, secret, { algorithms: ['HS256'] });
}

// RS256: Asymmetric keys (more secure, supports rotation)
const crypto = require('crypto');

// Generate RSA key pair
const { publicKey, privateKey } = crypto.generateKeyPairSync('rsa', {
  modulusLength: 2048,
  publicKeyEncoding: 'pem',
  privateKeyEncoding: 'pem'
});

function generateRS256Token(payload) {
  return jwt.sign(payload, privateKey, { algorithm: 'RS256', expiresIn: '1h' });
}

function verifyRS256Token(token) {
  return jwt.verify(token, publicKey, { algorithms: ['RS256'] });
}

// Key rotation with RS256
const keys = {
  'key-2024-01': publicKey1,
  'key-2024-06': publicKey2
};

function verifyTokenWithRotation(token) {
  try {
    const decoded = jwt.verify(token, publicKey1, { algorithms: ['RS256'] });
    return decoded;
  } catch (error) {
    // Try with rotated key
    const decoded = jwt.verify(token, publicKey2, { algorithms: ['RS256'] });
    return decoded;
  }
}
```

**Base64url Encoding**

Base64url encoding is a variant of Base64 designed for safe use in URLs and JWTs. It replaces `+` with `-` and `/` with `_`, and removes padding `=` characters. This encoding ensures JWTs can be safely transmitted in URLs without special character issues. Decoding requires reversing these transformations before applying standard Base64 decoding.

```javascript
// Base64url encoding
function base64urlEncode(str) {
  return Buffer.from(str)
    .toString('base64')
    .replace(/\+/g, '-')
    .replace(/\//g, '_')
    .replace(/=+$/, '');
}

// Base64url decoding
function base64urlDecode(str) {
  // Add padding
  str += '='.repeat((4 - str.length % 4) % 4);
  
  // Reverse replacements
  str = str.replace(/-/g, '+').replace(/_/g, '/');
  
  return Buffer.from(str, 'base64').toString('utf8');
}

// JWT header encoding
const header = { alg: 'HS256', typ: 'JWT' };
const encodedHeader = base64urlEncode(JSON.stringify(header));
// "eyJhbGciOiJIUzI1NiIsInR5cCJ9"

// JWT payload encoding
const payload = { sub: '123', exp: Date.now() / 1000 + 3600 };
const encodedPayload = base64urlEncode(JSON.stringify(payload));
// "eyJzdWIiOiIxMjM..."
```

**Signature Computation**

JWT signature computation varies by algorithm. For HS256, it computes HMAC SHA-256 hash of the encoded header and payload, using a shared secret key. For RS256, it signs the SHA-256 hash using RSA private key. The signature ensures token integrity and authenticity, preventing tampering. Verification recomputes the signature using the appropriate key and compares it with the provided signature.

```javascript
// HS256 signature computation
const crypto = require('crypto');

function computeHS256Signature(header, payload, secret) {
  const data = `${header}.${payload}`;
  return crypto.createHmac('sha256', secret).update(data).digest('base64');
}

// RS256 signature computation
function computeRS256Signature(header, payload, privateKey) {
  const data = `${header}.${payload}`;
  const hash = crypto.createHash('sha256').update(data).digest();
  
  const sign = crypto.createSign('sha256', privateKey);
  sign.update(hash);
  
  return sign.sign(null, 'base64');
}

// Complete JWT construction
function createJWT(header, payload, signature) {
  return `${header}.${payload}.${signature}`;
}

// Example: eyJhbGciOiJIUzI1NiIsInR5cCJ9.eyJzdWIiOiIxMjM...
```

---

### Password Security

**Hashing (bcrypt, scrypt, argon2)**

Password hashing is a one-way cryptographic transformation that securely stores passwords by converting them into fixed-length hashes. bcrypt is designed specifically for password hashing, incorporating a salt and configurable work factor to slow down brute force attacks. scrypt and argon2 are memory-hard algorithms that are resistant to GPU-based attacks. Never store passwords in plaintext or use reversible encryption.

```javascript
// ✅ Secure: Password hashing with bcrypt
const bcrypt = require('bcrypt');

async function hashPassword(password) {
  const salt = await bcrypt.genSalt(10); // 10 rounds
  const hash = await bcrypt.hash(password, salt, 10);
  return hash;
}

async function verifyPassword(password, hash) {
  const match = await bcrypt.compare(password, hash);
  return match;
}

// ✅ Secure: Using argon2 (memory-hard)
const argon2 = require('argon2');

async function hashWithArgon2(password) {
  return await argon2.hash(password, {
    type: argon2.argon2id,
    memoryCost: 65536, // High memory cost
    timeCost: 3,
    parallelism: 4
  });
}

// ❌ Vulnerable: Storing plaintext passwords
const users = [
  { username: 'john', password: 'secret123' } // Never do this!
];
```

**Salting**

Salting adds random data to passwords before hashing, ensuring that identical passwords produce different hashes. This prevents rainbow table attacks and ensures that cracking one password doesn't reveal information about other passwords. Modern hashing libraries like bcrypt automatically include salt generation, but understanding salting is important for security architecture.

```javascript
// Manual salting (not recommended, use bcrypt instead)
const crypto = require('crypto');

function hashPasswordWithSalt(password) {
  const salt = crypto.randomBytes(16).toString('hex');
  const hash = crypto.pbkdf2Sync(password, salt, 100000);
  return { hash, salt };
}

// ✅ Secure: bcrypt includes automatic salting
const bcrypt = require('bcrypt');

// bcrypt generates and manages salt internally
const hash = await bcrypt.hash(password, 10); // Salt included automatically
```

**Password Policies**

Password policies enforce strong password requirements to reduce vulnerability to brute force and dictionary attacks. Policies should include minimum length, complexity requirements (uppercase, lowercase, numbers, symbols), and prevent common weak passwords. However, overly strict policies can lead to user frustration and password reuse patterns.

```javascript
// ✅ Secure: Password validation with zod
const { z } = require('zod');

const passwordSchema = z.object({
  password: z.string()
    .min(12, 'Password must be at least 12 characters')
    .max(128, 'Password must not exceed 128 characters')
    .regex(/[A-Z]/, 'Password must contain at least one uppercase letter')
    .regex(/[a-z]/, 'Password must contain at least one lowercase letter')
    .regex(/[0-9]/, 'Password must contain at least one number')
    .regex(/[^A-Za-z0-9]/, 'Password must contain at least one special character')
});

function validatePassword(password) {
  return passwordSchema.safeParse(password);
}

// ✅ Secure: Check against common weak passwords
const commonPasswords = ['password', '123456', 'qwerty', 'admin'];

function isCommonPassword(password) {
  return commonPasswords.includes(password.toLowerCase());
}

function registerUser(username, password) {
  if (isCommonPassword(password)) {
    return { error: 'Please choose a stronger password' };
  }
  
  const validation = validatePassword(password);
  if (!validation.success) {
    return { error: validation.error };
  }
  
  // Create user
  return createUser(username, password);
}
```

---

### Authorization Patterns

**Role-Based Access Control (RBAC)**

RBAC assigns permissions to users based on their roles, simplifying authorization management. Roles represent job functions (admin, user, moderator), and permissions define what actions each role can perform. This pattern is easy to understand and implement but can become complex with many roles and fine-grained permissions.

```javascript
// ✅ Secure: RBAC implementation
const roles = {
  admin: ['create', 'read', 'update', 'delete', 'manage_users'],
  moderator: ['read', 'update', 'delete'],
  user: ['read']
};

function hasPermission(user, permission) {
  return user.roles.some(role => roles[role].includes(permission));
}

// Authorization middleware
function requirePermission(permission) {
  return (req, res, next) => {
    const user = req.user;
    
    if (!hasPermission(user, permission)) {
      return res.status(403).json({ 
        error: 'Forbidden: Insufficient permissions' 
      });
    }
    
    req.permission = permission;
    next();
  };
}

// Usage in routes
app.get('/admin/users', 
  requirePermission('manage_users'), // Admin only
  async (req, res) => {
    const users = await getUsers();
    res.json(users);
  }
);

app.get('/users', 
  requirePermission('read'), // All authenticated users
  async (req, res) => {
    const users = await getUsers();
    res.json(users);
  }
);
```

**Attribute-Based Access Control (ABAC)**

ABAC evaluates access based on user attributes (department, clearance level, time of day) and resource attributes (classification, owner). This provides more flexible and fine-grained control than RBAC but is more complex to implement and evaluate. ABAC is suitable for organizations with complex security requirements and dynamic access policies.

```javascript
// ✅ Secure: ABAC implementation
const policies = [
  {
    id: 'policy-1',
    name: 'Financial Data Access',
    rules: [
      {
        subject: { attribute: 'department', operator: 'equals', value: 'finance' },
        action: { attribute: 'clearance', operator: 'gte', value: 'secret' },
        resource: { attribute: 'classification', operator: 'equals', value: 'confidential' }
      }
    ]
  }
];

function evaluateAccess(user, resource, action) {
  for (const policy of policies) {
    const subjectMatch = evaluateRule(user, policy.rules[0].subject);
    const actionMatch = evaluateRule(user, policy.rules[1].action);
    const resourceMatch = evaluateRule(resource, policy.rules[2].resource);
    
    if (subjectMatch && actionMatch && resourceMatch) {
      return true; // Access granted
    }
  }
  
  return false; // Access denied
}

function evaluateRule(entity, rule) {
  const entityValue = entity[rule.attribute];
  
  switch (rule.operator) {
    case 'equals': return entityValue === rule.value;
    case 'gte': return entityValue >= rule.value;
    case 'lte': return entityValue <= rule.value;
    case 'in': return rule.value.includes(entityValue);
    default: return false;
  }
}

// Usage
app.get('/financial-reports', async (req, res) => {
  const user = req.user;
  const resource = { classification: 'confidential' };
  
  if (!evaluateAccess(user, resource, 'read')) {
    return res.status(403).json({ error: 'Access denied' });
  }
  
  const reports = await getFinancialReports(user);
  res.json(reports);
});
```

---

## Frontend Security

### XSS Prevention

**Input Sanitization**

Input sanitization removes or neutralizes malicious content from user input before it's stored or displayed. Libraries like DOMPurify provide comprehensive sanitization by stripping dangerous HTML tags, attributes, and JavaScript. Sanitization should be applied on both client-side (before rendering) and server-side (before storage).

```javascript
// ✅ Secure: Using DOMPurify for input sanitization
const createDOMPurify = require('isomorphic-dompurify');
const dompurify = createDOMPurify({
  ALLOWED_TAGS: ['p', 'br', 'strong', 'em'],
  ALLOWED_ATTR: ['class', 'id']
});

function sanitizeUserInput(input) {
  return dompurify.sanitize(input);
}

// React component with sanitization
function Comment({ text }) {
  return (
    <div>
      {dompurify.sanitize(text)}
    </div>
  );
}

// ❌ Vulnerable: Using dangerouslySetInnerHTML without sanitization
function Comment({ text }) {
  return (
    <div dangerouslySetInnerHTML={{ __html: text }} />
  ); // XSS vulnerability if text contains scripts
}
```

**Content Security Policy (CSP)**

CSP is an HTTP header that restricts what resources (scripts, styles, images) a page can load, mitigating XSS and data injection attacks. CSP directives include `default-src`, `script-src`, `style-src`, `img-src`, and `connect-src`. Report-only mode allows testing policies without blocking resources.

```javascript
// ✅ Secure: CSP header configuration
const helmet = require('helmet');

app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"], // Only load from same origin
    scriptSrc: ["'self'", "'unsafe-inline'"], // Allow inline scripts if needed
    styleSrc: ["'self'", "'unsafe-inline'"], // Allow inline styles
    imgSrc: ["'self'", 'data:', 'https:'], // Allow images from specific sources
    connectSrc: ["'self'"], // Only connect to same origin
    fontSrc: ["'self'", 'https://fonts.googleapis.com'], // Allow fonts from specific CDN
    objectSrc: ["'none'"], // Block plugins
    baseUri: 'self',
    formAction: ["'self'"],
    frameAncestors: ["'none'"],
    reportUri: '/csp-violation-report' // Report violations
  },
  reportOnly: false // Enforce in production
}));

// ✅ Secure: CSP with nonce for inline scripts
const crypto = require('crypto');

function generateNonce() {
  return crypto.randomBytes(16).toString('base64');
}

app.use((req, res, next) => {
  const nonce = generateNonce();
  res.locals.nonce = nonce;
  next();
});

app.get('/page', (req, res) => {
  res.send(`
    <html>
      <head>
        <meta http-equiv="Content-Security-Policy" 
              content="script-src 'self' 'nonce-${req.locals.nonce}';">
      </head>
      <body>
        <script nonce="${req.locals.nonce}">
          console.log('Safe inline script');
        </script>
      </body>
    </html>
  `);
});
```

**Output Encoding**

Output encoding converts special characters to their HTML entity equivalents before rendering user content. This prevents browsers from interpreting user input as HTML or JavaScript. Common encodings include HTML entities (`<` becomes `<`), URL encoding, and JavaScript encoding.

```javascript
// ✅ Secure: Output encoding function
function escapeHtml(unsafe) {
  const htmlEscapes = {
    '&': '&',
    '<': '<',
    '>': '>',
    '"': '"',
    "'": ''',
    '/': '&#47;'
  };
  
  return unsafe.replace(/[&<>"'/]/g, char => htmlEscapes[char]);
}

// React automatic escaping (built-in protection)
function UserComment({ text }) {
  return <div>{text}</div>; // React automatically escapes
}

// Manual escaping for non-React contexts
const template = `
  <div>User: {{ username }}</div>
`;

function renderTemplate(username) {
  return template.replace('{{ username }}', escapeHtml(username));
}
```

---

### CSRF Protection

**SameSite Cookies**

SameSite cookie attribute controls when cookies are sent with cross-site requests. `Strict` mode prevents cookies from being sent in cross-site requests, effectively blocking CSRF attacks. `Lax` mode allows cookies with top-level navigations. `None` mode blocks all cross-site cookie sending.

```javascript
// ✅ Secure: SameSite cookie configuration
const session = require('express-session');

app.use(session({
  secret: process.env.SESSION_SECRET,
  cookie: {
    sameSite: 'strict', // Best for CSRF protection
    secure: true, // HTTPS only
    httpOnly: true, // Prevent XSS access
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  }
}));

// ✅ Secure: Different SameSite settings for different use cases
app.use(session({
  cookie: {
    sameSite: 'lax', // Allow top-level navigation
    secure: true,
    httpOnly: true
  }
}));

// ✅ Secure: CSRF token with SameSite strict
app.use((req, res, next) => {
  const token = generateCSRFToken();
  
  res.cookie('csrf_token', token, {
    httpOnly: true,
    sameSite: 'strict',
    secure: true
  });
  
  req.session.csrfToken = token;
  next();
});
```

**CSRF Tokens**

CSRF tokens are cryptographically random values generated by the server and validated on form submission. They should be unique per session, unpredictable, and bound to the session. Tokens can be stored in session, cookies, or custom headers. Double-submit cookie pattern provides additional protection.

```javascript
// ✅ Secure: CSRF token generation and validation
const crypto = require('crypto');

function generateCSRFToken() {
  return crypto.randomBytes(32).toString('hex');
}

// Store in session
app.use((req, res, next) => {
  req.session.csrfToken = generateCSRFToken();
  next();
});

// Include in all state-changing forms
app.get('/transfer', (req, res) => {
  res.send(`
    <form action="/transfer" method="POST">
      <input type="hidden" name="csrf_token" value="${req.session.csrfToken}">
      <input type="text" name="amount">
      <button>Transfer</button>
    </form>
  `);
});

// Validate on submission
app.post('/transfer', (req, res) => {
  const { csrf_token, amount, to } = req.body;
  
  if (!req.session.csrfToken || csrf_token !== req.session.csrfToken) {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }
  
  // Process transfer
  res.json({ message: 'Transfer complete' });
});

// ✅ Secure: Double-submit cookie pattern
app.use((req, res, next) => {
  const token = generateCSRFToken();
  
  res.cookie('csrf_token', token, {
    httpOnly: true,
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000
  });
  
  req.session.csrfToken = token;
  next();
});
```

---

### CORS & Security Headers

**CORS Configuration**

Cross-Origin Resource Sharing (CORS) controls which origins can access your API. Proper CORS configuration specifies allowed origins, methods, headers, and credentials. Misconfigured CORS can expose APIs to unauthorized access or enable CSRF attacks.

```javascript
// ✅ Secure: CORS with specific origins
const cors = require('cors');

app.use(cors({
  origin: ['https://trusted-site.com', 'https://another-trusted.com'], // Whitelist specific origins
  methods: ['GET', 'POST', 'PUT', 'DELETE'], // Allowed HTTP methods
  allowedHeaders: ['Content-Type', 'Authorization'], // Allowed request headers
  credentials: true, // Allow cookies
  maxAge: 86400 // 24 hours preflight cache
}));

// ❌ Vulnerable: Allow all origins
app.use(cors({
  origin: '*', // Dangerous in production
  methods: '*',
  allowedHeaders: '*'
}));

// ✅ Secure: Dynamic origin based on environment
const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'];

app.use(cors({
  origin: (origin, callback) => {
    if (allowedOrigins.includes(origin)) {
      callback(null, true); // Allow
    }
    callback(null, false); // Block
  }
}));
```

**Security Headers (HSTS, X-Frame-Options, etc.)**

Security headers provide additional layers of protection by instructing browsers on security policies. HSTS enforces HTTPS connections, X-Frame-Options prevents clickjacking, X-Content-Type-Options prevents MIME sniffing, and Referrer-Policy controls information leakage. Helmet.js provides easy configuration of these headers.

```javascript
// ✅ Secure: Comprehensive security headers with Helmet
const helmet = require('helmet');

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"]
    }
  },
  hsts: {
    maxAge: 31536000, // 1 year
    includeSubDomains: true,
    preload: true
  },
  frameguard: {
    action: 'deny' // Prevent clickjacking
  },
  noSniff: true, // Prevent MIME sniffing
  xContentTypeOptions: 'nosniff',
  referrerPolicy: {
    policy: 'no-referrer-when-downgrade'
  },
  permittedCrossDomainPolicies: false // Disable legacy policies
}));

// ✅ Secure: Custom security headers
app.use((req, res, next) => {
  // X-Frame-Options: Prevent embedding
  res.setHeader('X-Frame-Options', 'DENY');
  
  // X-Content-Type-Options: Prevent MIME sniffing
  res.setHeader('X-Content-Type-Options', 'nosniff');
  
  // X-XSS-Protection: Enable browser XSS filter
  res.setHeader('X-XSS-Protection', '1; mode=block');
  
  // Strict-Transport-Security: Enforce HTTPS
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  
  next();
});
```

---

## Backend Security

### Rate Limiting

**Fixed Window**

Fixed window rate limiting divides time into fixed windows (e.g., 15 minutes) and limits requests per window. When the window expires, the counter resets. This approach is simple to implement but can allow bursts of requests at window boundaries.

```javascript
// ✅ Secure: Fixed window rate limiting
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: 'Too many requests from this IP, please try again later',
  standardHeaders: true, // Return rate limit info in headers
  legacyHeaders: false
});

app.use('/api', limiter);

// Custom fixed window implementation
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
    
    if (now - entry.timestamp < windowMs) {
      entry.count++;
    }
    
    requestCounts.set(ip, entry);
    
    // Check limit
    if (entry.count > max) {
      return res.status(429).json({
        error: 'Too many requests',
        retryAfter: Math.ceil(windowMs / 1000)
      });
    }
    
    // Add rate limit headers
    res.set('X-RateLimit-Limit', max);
    res.set('X-RateLimit-Remaining', max - entry.count);
    res.set('X-RateLimit-Reset', new Date(entry.timestamp + windowMs).toISOString());
    
    next();
  };
}
```

**Sliding Window**

Sliding window rate limiting tracks requests within a rolling time window, providing more accurate rate limiting than fixed window. As time progresses, the window slides forward, and requests outside the window are no longer counted. This prevents burst attacks at window boundaries while allowing consistent request rates.

```javascript
// ✅ Secure: Sliding window rate limiting
const requestTimestamps = new Map();

function slidingWindowRateLimit(options) {
  const { windowMs = 60000, max = 100 } = options;
  const windowSize = 100; // Track last 100 requests
  
  return (req, res, next) => {
    const ip = req.ip;
    const now = Date.now();
    
    // Get or create timestamp list for this IP
    let timestamps = requestTimestamps.get(ip) || [];
    
    // Add current request timestamp
    timestamps.push(now);
    
    // Remove timestamps outside the window
    timestamps = timestamps.filter(ts => now - ts < windowMs);
    
    // Keep only windowSize most recent
    if (timestamps.length > windowSize) {
      timestamps = timestamps.slice(-windowSize);
    }
    
    requestTimestamps.set(ip, timestamps);
    
    // Count requests in window
    const countInWindow = timestamps.length;
    
    if (countInWindow >= max) {
      return res.status(429).json({
        error: 'Too many requests',
        retryAfter: Math.ceil(windowMs / 1000)
      });
    }
    
    // Add rate limit headers
    res.set('X-RateLimit-Limit', max);
    res.set('X-RateLimit-Remaining', max - countInWindow);
    res.set('X-RateLimit-Reset', new Date(now + windowMs).toISOString());
    
    next();
  };
}
```

**Token Bucket**

Token bucket rate limiting provides a more sophisticated approach using a bucket metaphor. Each request consumes one or more tokens, and tokens are replenished at a fixed rate. This allows bursts while maintaining long-term average rate, making it suitable for APIs with varying request patterns.

```javascript
// ✅ Secure: Token bucket rate limiting
class TokenBucket {
  constructor(capacity, refillRate) {
    this.capacity = capacity; // Max tokens
    this.tokens = capacity; // Current tokens
    this.refillRate = refillRate; // Tokens per second
    this.lastRefill = Date.now();
  }
  
  consume(tokens = 1) {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    
    // Refill tokens
    this.tokens = Math.min(this.capacity, this.tokens + elapsed * this.refillRate);
    this.lastRefill = now;
    
    return this.tokens >= tokens;
  }
  
  tryConsume(req, res, next) {
    const cost = 1; // Cost per request
    
    if (this.consume(cost)) {
      return next(); // Allow request
    }
    
    res.status(429).json({
      error: 'Rate limit exceeded',
      retryAfter: Math.ceil((this.capacity - this.tokens) / this.refillRate)
    });
  }
}

const bucket = new TokenBucket(100, 10); // 100 tokens, refill 10 per second

app.use('/api', bucket.tryConsume.bind(bucket));
```

---

### Input Validation

**Schema Validation**

Schema validation ensures incoming data conforms to expected structure, types, and constraints. Libraries like Zod, Joi, or Yup provide declarative schema definitions with automatic validation and detailed error messages. Validation should occur early in the request pipeline, before business logic or database operations.

```javascript
// ✅ Secure: Schema validation with Zod
const { z } = require('zod');

const userSchema = z.object({
  username: z.string()
    .min(3, 'Username must be at least 3 characters')
    .max(50, 'Username must not exceed 50 characters')
    .regex(/^[a-zA-Z0-9_]+$/, 'Username can only contain letters, numbers, and underscores'),
  
  email: z.string()
    .email('Invalid email address'),
  
  age: z.number()
    .min(18, 'Must be at least 18 years old')
    .max(120, 'Must not exceed 120 years old')
    .positive('Age must be positive')
});

function validateUserInput(input) {
  return userSchema.safeParse(input);
}

// Usage in Express route
app.post('/users', (req, res) => {
  const validation = validateUserInput(req.body);
  
  if (!validation.success) {
    return res.status(400).json({
      error: 'Validation failed',
      details: validation.error.errors
    });
  }
  
  // Process with validated data
  const user = await createUser(validation.data);
  res.status(201).json(user);
});

// ✅ Secure: Nested object validation
const addressSchema = z.object({
  street: z.string().min(1),
  city: z.string().min(1),
  country: z.string().min(1),
  postalCode: z.string().regex(/^\d{5}$/)
});

const profileSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  address: addressSchema
});

function validateProfile(input) {
  return profileSchema.safeParse(input);
}
```

**Whitelisting vs Blacklisting**

Whitelisting allows only known good values, while blacklisting blocks known bad values. Whitelisting is more secure because it defaults to denying access unless explicitly allowed. Blacklisting is less secure because new attack vectors can bypass the list. Always prefer whitelisting for security-critical operations.

```javascript
// ❌ Vulnerable: Blacklisting (can be bypassed)
const dangerousTags = ['script', 'iframe', 'object', 'embed'];

function sanitizeHtml(input) {
  return input.replace(new RegExp(dangerousTags.join('|'), 'gi'), '');
}

// ✅ Secure: Whitelisting (deny by default)
const allowedTags = ['p', 'br', 'strong', 'em', 'u', 'ol', 'ul', 'li', 'span', 'div'];
const allowedAttributes = ['class', 'id', 'data-*'];

function sanitizeHtmlStrict(input) {
  return createDOMPurify({
    ALLOWED_TAGS: allowedTags,
    ALLOWED_ATTR: allowedAttributes
  }).sanitize(input);
}

// ✅ Secure: Whitelisting file types
const allowedFileTypes = ['image/jpeg', 'image/png', 'image/gif'];
const allowedMimeTypes = ['application/pdf', 'text/plain'];

function validateFileType(file) {
  return allowedFileTypes.includes(file.mimetype) || allowedMimeTypes.includes(file.mimetype);
}
```

---

### Secure Communication

**HTTPS/TLS Implementation**

HTTPS/TLS encrypts all communication between client and server, preventing eavesdropping and man-in-the-middle attacks. TLS provides authentication through certificates, integrity through message authentication codes, and confidentiality through encryption. Proper configuration includes using strong cipher suites, disabling weak protocols, and implementing certificate pinning.

```javascript
// ✅ Secure: HTTPS server with proper TLS configuration
const https = require('https');
const fs = require('fs');

const options = {
  key: fs.readFileSync('private-key.pem'),
  cert: fs.readFileSync('certificate.pem'),
  ca: fs.readFileSync('ca-bundle.crt'),
  
  minVersion: 'TLSv1.2', // Minimum TLS version
  ciphers: 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384', // Strong ciphers only
  honorCipherOrder: true, // Use server cipher order
  rejectUnauthorized: false, // Allow self-signed certs in development
  
  // Disable weak protocols and ciphers
  secureOptions: {
    secureProtocol: true, // Only allow TLSv1.2+
    secureCiphers: true, // Only strong ciphers
    rejectUnauthorized: true
  }
};

const server = https.createServer(options, (req, res) => {
  // Handle request
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Secure response');
});

server.listen(443, () => {
  console.log('HTTPS server running on port 443');
});

// ✅ Secure: HTTP/2 with TLS
const spdy = require('spdy');

const options = {
  key: fs.readFileSync('private-key.pem'),
  cert: fs.readFileSync('certificate.pem'),
  ca: fs.readFileSync('ca-bundle.crt'),
  
  protocols: ['h2'], // Enable HTTP/2
  ciphers: 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384'
};

const server = spdy.createServer(options, (req, res) => {
  // Handle request with HTTP/2 benefits
  res.end('HTTP/2 secure response');
});
```

**Certificate Management**

Certificate management involves obtaining, renewing, and properly configuring SSL/TLS certificates. Let's Encrypt provides free, automated certificates with proper validation. Certificate pinning prevents man-in-the-middle attacks by specifying which certificates are trusted. Always use certificates from trusted Certificate Authorities in production.

```javascript
// ✅ Secure: Certificate pinning with HPKP header
app.use((req, res, next) => {
  // Public-Key-Pins header for certificate pinning
  const pins = 'pin-sha256="cUPc06ZUYdJq5b4RdIw="; max-age=5184000; includeSubDomains';
  
  res.setHeader('Public-Key-Pins', pins);
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  
  next();
});

// ✅ Secure: Dynamic certificate loading with validation
const crypto = require('crypto');
const tls = require('tls');

function createSecureServer(certPath, keyPath) {
  const options = {
    key: fs.readFileSync(keyPath),
    cert: fs.readFileSync(certPath),
    ca: fs.readFileSync('ca-bundle.crt'),
    
    minVersion: 'TLSv1.2',
    ciphers: tls.getCiphers().join(':'),
    
    // Certificate validation
    rejectUnauthorized: true, // Reject self-signed in production
    requestCert: true, // Request client certificates
    checkServerIdentity: true // Verify server hostname
  };
  
  return https.createServer(options);
}
```

---

### Security Middleware

**Helmet.js**

Helmet.js is a collection of middleware functions that set security-related HTTP headers. It provides sensible defaults while allowing customization. Helmet protects against well-known web vulnerabilities by setting headers like CSP, HSTS, X-Frame-Options, and X-Content-Type-Options.

```javascript
// ✅ Secure: Comprehensive Helmet configuration
const helmet = require('helmet');

app.use(helmet({
  // Content Security Policy
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'"],
      fontSrc: ["'self'", 'https://fonts.googleapis.com'],
      objectSrc: ["'none'"],
      baseUri: 'self',
      formAction: ["'self'"],
      frameAncestors: ["'none'"],
      reportUri: '/csp-violation-report'
    },
    reportOnly: process.env.NODE_ENV !== 'production'
  },
  
  // HTTP Strict Transport Security
  hsts: {
    maxAge: 31536000, // 1 year
    includeSubDomains: true,
    preload: true,
    force: true // Force HSTS even in development
  },
  
  // X-Frame-Options (prevent clickjacking)
  frameguard: {
    action: 'deny' // Block all framing
  },
  
  // X-Content-Type-Options (prevent MIME sniffing)
  noSniff: true,
  xContentTypeOptions: 'nosniff',
  
  // Referrer Policy
  referrerPolicy: {
    policy: 'no-referrer-when-downgrade'
  },
  
  // Additional protections
  xssFilter: true, // Enable browser XSS filter
  permittedCrossDomainPolicies: false, // Disable legacy policies
  ieNoOpen: true, // Prevent IE from opening files directly
  noCache: true // Disable caching for sensitive data
}));

// ✅ Secure: Custom security middleware
app.use((req, res, next) => {
  // Remove server information
  res.removeHeader('X-Powered-By');
  
  // Disable caching for sensitive endpoints
  if (req.path.startsWith('/api/')) {
    res.setHeader('Cache-Control', 'no-store, no-cache, must-revalidate');
  }
  
  next();
});
```

---

## Hands-on Security Projects

### Project 1: Secure Express Server

**Helmet.js Implementation**

Helmet.js provides security headers to protect against common web vulnerabilities. Implementing Helmet involves configuring Content Security Policy, HSTS, and frame protection. This project demonstrates how to add security middleware to an Express application and verify the headers are set correctly.

```javascript
const express = require('express');
const helmet = require('helmet');

// ✅ Secure: Configure Helmet with security headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:cdn.example.com']
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  },
  frameguard: {
    action: 'deny'
  },
  noSniff: true,
  xContentTypeOptions: 'nosniff'
}));

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

app.get('/', (req, res) => {
  res.json({ message: 'Secure server running' });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

**Rate Limiting**

Rate limiting protects APIs from abuse by limiting the number of requests from a single IP or user. This project implements rate limiting using the express-rate-limit middleware with different limits for different endpoints. Rate limiting prevents brute force attacks and ensures fair resource usage.

```javascript
const rateLimit = require('express-rate-limit');

// ✅ Secure: Global rate limiting
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: 'Too many requests from this IP, please try again later',
  standardHeaders: true
});

app.use('/api', globalLimiter);

// ✅ Secure: Stricter rate limiting for auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  message: 'Too many login attempts, please try again later',
  skipSuccessfulRequests: true
});

app.post('/login', authLimiter, (req, res) => {
  const { username, password } = req.body;
  
  // Validate credentials
  if (!username || !password) {
    return res.status(400).json({ error: 'Username and password are required' });
  }
  
  const user = await authenticate(username, password);
  
  if (user) {
    res.json({ message: 'Logged in successfully' });
  } else {
    res.status(401).json({ error: 'Invalid credentials' });
  }
});
```

**Input Validation with Zod**

Input validation ensures that incoming data meets expected requirements before processing. This project uses Zod schema validation to validate user input, preventing injection attacks and data integrity issues. Validation occurs early in the request pipeline, providing clear error messages to clients.

```javascript
const { z } = require('zod');

// ✅ Secure: User registration validation schema
const userRegistrationSchema = z.object({
  username: z.string()
    .min(3, 'Username must be at least 3 characters')
    .max(50, 'Username must not exceed 50 characters')
    .regex(/^[a-zA-Z0-9_]+$/, 'Username can only contain letters, numbers, and underscores'),
  
  email: z.string()
    .email('Invalid email address'),
  
  password: z.string()
    .min(12, 'Password must be at least 12 characters')
    .max(128, 'Password must not exceed 128 characters')
    .regex(/[A-Z]/, 'Password must contain at least one uppercase letter')
    .regex(/[a-z]/, 'Password must contain at least one lowercase letter')
    .regex(/[0-9]/, 'Password must contain at least one number')
    .regex(/[^A-Za-z0-9]/, 'Password must contain at least one special character')
});

// Validation middleware
function validateBody(schema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    
    if (!result.success) {
      return res.status(400).json({
        error: 'Validation failed',
        details: result.error.errors
      });
    }
    
    req.body = result.data;
    next();
  };
}

// Usage in routes
app.post('/users', validateBody(userRegistrationSchema), async (req, res) => {
  const user = await createUser(req.body);
  res.status(201).json(user);
});
```

**Secure Error Handling**

Centralized error handling ensures consistent error responses across the application. This project implements custom error classes, an async error wrapper, and error-handling middleware. Errors are logged with context and returned in a standardized format to clients.

```javascript
// ✅ Secure: Custom error classes
class AppError extends Error {
  constructor(message, statusCode = 500, isOperational = true) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = isOperational;
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(message = 'Validation failed', errors = []) {
    super(message, 400, 'VALIDATION_ERROR');
    this.errors = errors;
  }
}

class NotFoundError extends AppError {
  constructor(message = 'Resource not found') {
    super(message, 404, 'NOT_FOUND');
  }
}

// ✅ Secure: Async error wrapper
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
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    }
  });
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({ error: 'Not Found' });
});
```

---

### Project 2: JWT Authentication System

**User Registration with Password Hashing**

User registration requires secure password storage using bcrypt hashing. Passwords should never be stored in plaintext and must be hashed with a strong algorithm. This project demonstrates secure user creation, password hashing, and JWT token generation for authentication.

```javascript
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');

// ✅ Secure: Password hashing with bcrypt
async function hashPassword(password) {
  const salt = await bcrypt.genSalt(12);
  const hash = await bcrypt.hash(password, salt, 12);
  return hash;
}

// ✅ Secure: User registration
app.post('/register', async (req, res) => {
  const { username, email, password } = req.body;
  
  // Validate input
  if (!username || !email || !password) {
    return res.status(400).json({ error: 'All fields are required' });
  }
  
  // Check if user exists
  const existingUser = await findUserByEmail(email);
  if (existingUser) {
    return res.status(409).json({ error: 'User already exists' });
  }
  
  // Hash password
  const hashedPassword = await hashPassword(password);
  
  // Create user
  const user = await createUser({
    username,
    email,
    password: hashedPassword
  });
  
  res.status(201).json({
    id: user.id,
    username: user.username,
    email: user.email
  });
});
```

**JWT Login Endpoint**

JWT login endpoint generates tokens upon successful authentication and returns them to clients. Tokens should include appropriate claims (user ID, roles, expiration) and be signed with a strong algorithm. This project demonstrates secure token generation, verification, and protected route implementation.

```javascript
const jwt = require('jsonwebtoken');
const SECRET = process.env.JWT_SECRET;

// ✅ Secure: JWT login endpoint
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  
  // Authenticate user
  const user = await authenticateUser(username, password);
  
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Generate JWT token
  const token = jwt.sign(
    {
      sub: user.id,
      username: user.username,
      role: user.role
    },
    SECRET,
    { 
      algorithm: 'HS256',
      expiresIn: '1h'
    }
  );
  
  res.json({
    token,
    user: {
      id: user.id,
      username: user.username
    }
  });
});
```

**Protected Routes with JWT Verification**

Protected routes verify JWT tokens on each request to ensure only authenticated users can access sensitive endpoints. The verification middleware extracts the token from the Authorization header, validates it, and attaches the user to the request object.

```javascript
const jwt = require('jsonwebtoken');
const SECRET = process.env.JWT_SECRET;

// ✅ Secure: JWT verification middleware
function authenticateJWT(req, res, next) {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Missing or invalid authorization header' });
  }
  
  const token = authHeader.substring(7); // Remove 'Bearer '
  
  try {
    const decoded = jwt.verify(token, SECRET, { algorithms: ['HS256'] });
    
    // Attach user to request
    req.user = decoded;
    next();
    
  } catch (error) {
    return res.status(401).json({ error: 'Invalid or expired token' });
  }
}

// ✅ Secure: Protected route
app.get('/profile', authenticateJWT, (req, res) => {
  res.json({
    user: req.user,
    message: 'Protected profile data'
  });
});

app.get('/admin', authenticateJWT, (req, res) => {
  // Check role
  if (req.user.role !== 'admin') {
    return res.status(403).json({ error: 'Admin access required' });
  }
  
  res.json({
    message: 'Admin dashboard'
  });
});
```

**Token Refresh Mechanism**

Token refresh allows long-lived sessions without requiring users to re-authenticate frequently. Access tokens have short expiration (15 minutes) while refresh tokens have longer expiration (7 days). Refresh tokens are single-use and must be stored securely on the server.

```javascript
// ✅ Secure: Token refresh implementation
const jwt = require('jsonwebtoken');
const SECRET = process.env.JWT_SECRET;

// Generate access and refresh tokens
function generateTokens(userId) {
  const accessToken = jwt.sign(
    { sub: userId, type: 'access' },
    SECRET,
    { algorithm: 'HS256', expiresIn: '15m' }
  );
  
  const refreshToken = jwt.sign(
    { sub: userId, type: 'refresh' },
    SECRET,
    { algorithm: 'HS256', expiresIn: '7d' }
  );
  
  return { accessToken, refreshToken };
}

// Refresh endpoint
app.post('/refresh', async (req, res) => {
  const { refreshToken } = req.body;
  
  if (!refreshToken) {
    return res.status(400).json({ error: 'Refresh token is required' });
  }
  
  try {
    const decoded = jwt.verify(refreshToken, SECRET, { algorithms: ['HS256'] });
    
    if (decoded.type !== 'refresh') {
      return res.status(400).json({ error: 'Invalid token type' });
    }
    
    // Generate new tokens
    const tokens = generateTokens(decoded.sub);
    
    // Store refresh token (in a real app, store in database)
    await storeRefreshToken(decoded.sub, tokens.refreshToken);
    
    res.json({
      accessToken: tokens.accessToken,
      refreshToken: tokens.refreshToken
    });
    
  } catch (error) {
    return res.status(401).json({ error: 'Invalid or expired refresh token' });
  }
});
```

---

### Project 3: CSRF Prevention

**CSRF Token Implementation**

CSRF token implementation generates unique tokens for each session and includes them in forms. Tokens are validated on submission to prevent cross-site request forgery. This project demonstrates the complete CSRF protection flow including token generation, form inclusion, and validation.

```javascript
const crypto = require('crypto');
const session = require('express-session');

// ✅ Secure: CSRF token generation
function generateCSRFToken() {
  return crypto.randomBytes(32).toString('hex');
}

// Store token in session
app.use((req, res, next) => {
  req.session.csrfToken = generateCSRFToken();
  next();
});

// ✅ Secure: Form with CSRF token
app.get('/transfer', (req, res) => {
  res.send(`
    <html>
      <body>
        <h1>Transfer Money</h1>
        <form action="/transfer" method="POST">
          <input type="hidden" name="csrf_token" value="${req.session.csrfToken}">
          <input type="text" name="amount" placeholder="Amount">
          <input type="text" name="to" placeholder="Recipient">
          <button>Transfer</button>
        </form>
      </body>
    </html>
  `);
});

// ✅ Secure: Token validation
app.post('/transfer', (req, res) => {
  const { csrf_token, amount, to } = req.body;
  
  if (!req.session.csrfToken || csrf_token !== req.session.csrfToken) {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }
  
  // Process transfer
  res.json({ message: 'Transfer successful' });
});
```

**SameSite Cookie Configuration**

SameSite cookies prevent CSRF attacks by controlling when cookies are sent with cross-site requests. This project configures session cookies with `sameSite: 'strict' to prevent cookies from being sent in cross-site contexts.

```javascript
const session = require('express-session');

// ✅ Secure: SameSite cookie configuration
app.use(session({
  secret: process.env.SESSION_SECRET,
  cookie: {
    sameSite: 'strict', // Best for CSRF protection
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    maxAge: 24 * 60 * 60 * 1000
  }
}));
```

**CSRF Attack Testing**

CSRF attack testing involves simulating malicious requests to verify that CSRF protection is working correctly. This includes testing with missing tokens, invalid tokens, and cross-origin requests. Proper testing ensures the application is resilient against CSRF attacks.

```javascript
// ✅ Secure: CSRF protection testing
const axios = require('axios');

async function testCSRFProtection() {
  // Get CSRF token
  const { data: { csrfToken } } = await axios.get('/transfer');
  
  // Test with missing token
  const response1 = await axios.post('/transfer', {
    amount: 100,
    to: 'attacker'
    // Missing CSRF token
  });
  
  console.log('Without token:', response1.status); // Should be 403
  
  // Test with invalid token
  const response2 = await axios.post('/transfer', {
    amount: 100,
    to: 'attacker',
    csrf_token: 'invalid-token'
  });
  
  console.log('With invalid token:', response2.status); // Should be 403
  
  // Test with valid token
  const response3 = await axios.post('/transfer', {
    amount: 100,
    to: 'attacker',
    csrf_token: csrfToken
  });
  
  console.log('With valid token:', response3.status); // Should be 200
}
```

---

### Project 4: XSS Prevention Demo

**XSS Vulnerability Simulation**

XSS vulnerability simulation demonstrates how malicious scripts can be injected and executed in web applications. This project shows both vulnerable and secure implementations, highlighting the importance of input sanitization, output encoding, and Content Security Policy.

```javascript
const express = require('express');

// ❌ Vulnerable: XSS through unsanitized input
app.get('/comment', (req, res) => {
  const { comment } = req.query;
  
  res.send(`
    <html>
      <body>
        <h1>Comments</h1>
        <div>${comment}</div>
      </body>
    </html>
  `); // Vulnerable to XSS
});

// ✅ Secure: Output encoding
function escapeHtml(unsafe) {
  const htmlEscapes = {
    '&': '&',
    '<': '<',
    '>': '>',
    '"': '"',
    "'": '''
  };
  
  return unsafe.replace(/[&<>"'/]/g, char => htmlEscapes[char]);
}

app.get('/comment-safe', (req, res) => {
  const { comment } = req.query;
  
  res.send(`
    <html>
      <body>
        <h1>Comments</h1>
        <div>${escapeHtml(comment)}</div>
      </body>
    </html>
  `); // Safe from XSS
});
```

**Input Sanitization with DOMPurify**

DOMPurify removes dangerous HTML tags and attributes from user input, preventing XSS attacks. This project demonstrates comprehensive input sanitization using DOMPurify, including configuration of allowed tags and attributes for fine-grained control.

```javascript
const createDOMPurify = require('isomorphic-dompurify');

// ✅ Secure: DOMPurify configuration
const dompurify = createDOMPurify({
  ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'ol', 'ul', 'li', 'span', 'div', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6'],
  ALLOWED_ATTR: ['class', 'id', 'data-*'],
  ALLOW_DATA_ATTR: true, // Allow data-* attributes
  FORBID_TAGS: ['script', 'style', 'iframe', 'object', 'embed'],
  FORBID_ATTR: ['onclick', 'onload', 'onerror', 'onmouseover']
});

function sanitizeInput(input) {
  return dompurify.sanitize(input);
}

// Usage in Express route
app.post('/comment', (req, res) => {
  const { comment } = req.body;
  
  // Sanitize input
  const safeComment = sanitizeInput(comment);
  
  // Store in database
  await saveComment(safeComment);
  
  res.json({ message: 'Comment saved' });
});
```

**Content Security Policy Implementation**

CSP restricts the sources from which resources can be loaded, providing a powerful defense against XSS and data injection attacks. This project demonstrates CSP configuration with various directives, including script sources, style sources, and report endpoints for violation monitoring.

```javascript
const helmet = require('helmet');

// ✅ Secure: Comprehensive CSP configuration
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", "'unsafe-inline'", "'unsafe-eval'"],
    styleSrc: ["'self'", "'unsafe-inline'"],
    imgSrc: ["'self'", 'data:', 'https:cdn.example.com'],
    connectSrc: ["'self'"],
    fontSrc: ["'self'", 'https://fonts.googleapis.com'],
    objectSrc: ["'none'"],
    baseUri: 'self',
    formAction: ["'self'"],
    frameAncestors: ["'none'"],
    reportUri: '/csp-violation-report'
  },
  reportOnly: process.env.NODE_ENV !== 'production' // Enforce in production
}));

// CSP violation reporting endpoint
app.post('/csp-violation-report', express.json(), (req, res) => {
  const { cspReport } = req.body;
  
  // Log violations for monitoring
  console.log('CSP Violation:', cspReport);
  
  res.status(204).send();
});
```

---

## Quick Reference

**Security Headers Reference Table**

| Header | Purpose | Recommended Value | Example |
|--------|---------|-------------------|---------|
| Content-Security-Policy | Prevent XSS, control resource loading | `default-src 'self'; script-src 'self'` |
| Strict-Transport-Security | Enforce HTTPS | `max-age=31536000; includeSubDomains; preload` |
| X-Frame-Options | Prevent clickjacking | `DENY` |
| X-Content-Type-Options | Prevent MIME sniffing | `nosniff` |
| X-XSS-Protection | Enable browser XSS filter | `1; mode=block` |
| Referrer-Policy | Control referrer information | `no-referrer-when-downgrade` |
| Cache-Control | Control caching behavior | `no-store, no-cache` (sensitive data) |
| X-Content-Type-Options | Prevent content type sniffing | `nosniff` |

**OWASP Top 10 Quick Reference**

| Risk | Description | Prevention |
|-------|-------------|------------|
| Injection | Use parameterized queries, prepared statements, ORM | |
| Broken Authentication | Implement MFA, secure session management, rate limiting | |
| Sensitive Data Exposure | Encrypt data at rest and in transit, use strong TLS | |
| XXE | Disable XML external entities in parsers | |
| Broken Access Control | Implement proper authorization checks, use RBAC/ABAC | |
| Security Misconfiguration | Remove default credentials, disable unnecessary features | |
| XSS | Use output encoding, CSP, input sanitization | |
| Insecure Deserialization | Use safe deserialization libraries, validate input | |
| Known Vulnerabilities | Keep dependencies updated, use npm audit | |
| Insufficient Logging | Log security events, implement monitoring | |

**Security Checklist**

- [ ] Use HTTPS/TLS for all connections
- [ ] Implement rate limiting on all public endpoints
- [ ] Validate and sanitize all user input
- [ ] Use parameterized queries or ORM for database access
- [ ] Hash passwords with bcrypt, scrypt, or argon2
- [ ] Implement JWT with strong algorithms (HS256/RS256)
- [ ] Configure Content Security Policy headers
- [ ] Use Helmet.js for security headers
- [ ] Implement CSRF protection (tokens + SameSite cookies)
- [ ] Enable HSTS with long max-age
- [ ] Store secrets in environment variables, never in code
- [ ] Implement proper error handling and logging
- [ ] Regularly audit dependencies for vulnerabilities
- [ ] Use output encoding to prevent XSS
- [ ] Implement account lockout after failed login attempts

**Decision Tree: Choosing Authentication Method**

```
Need Authentication?
    ↓
Is it for internal services?
    ↓ YES → Use HS256 with shared secret
    ↓ NO → Use RS256 with public/private keys
    ↓
Do you need key rotation?
    ↓ YES → Use RS256 (supports rotation)
    ↓ NO → HS256 is sufficient
    ↓
Do you need distributed system?
    ↓ YES → Use RS256 with JWKS endpoint
    ↓ NO → HS256 works fine
    ↓
Decision: HS256 or RS256
```

**Decision Tree: Choosing Password Hashing Algorithm**

```
Password Hashing Algorithm Selection
    ↓
What's your priority?
    ↓ Security → Use Argon2 (most secure)
    ↓ Performance → Use bcrypt (well-optimized)
    ↓ Balance → Use scrypt
    ↓
What's your hardware?
    ↓ High memory → Argon2 or bcrypt
    ↓ Standard → scrypt
    ↓
Decision: Choose algorithm based on requirements
```

**Decision Tree: Choosing Rate Limiting Strategy**

```
Rate Limiting Strategy Selection
    ↓
What's your traffic pattern?
    ↓ Consistent → Use Fixed Window (simpler)
    ↓ Bursty → Use Sliding Window (smoother)
    ↓ Variable → Use Token Bucket (flexible)
    ↓
Do you need per-user limits?
    ↓ YES → Use Token Bucket with user keys
    ↓ NO → IP-based limiting is sufficient
    ↓
Decision: Choose strategy based on traffic pattern
```
