# Node.js Interview Questions & Answers

Complete Q&A guide for backend developer interviews - from fresher to experienced level.

---

## Table of Contents
- [Basic Level (Fresher)](#basic-level-fresher)
- [Intermediate Level (1-2 years)](#intermediate-level-1-2-years)
- [Advanced Level (2+ years)](#advanced-level-2-years)
- [Coding Challenges](#coding-challenges)
- [System Design Questions](#system-design-questions)
- [Practical Scenarios](#practical-scenarios)

---

## Basic Level (Fresher)

### 1. What is Node.js and why is it popular?

**Answer:**
Node.js is a JavaScript runtime built on Chrome's V8 engine that allows you to run JavaScript on the server-side.

**Key Features:**
- **Event-driven architecture** - Non-blocking I/O operations
- **Asynchronous** - Handles multiple requests simultaneously
- **Single-threaded** - Uses event loop for concurrency
- **V8 Engine** - Google's fast JavaScript engine
- **NPM ecosystem** - Largest package registry (1M+ packages)

**Use Cases:**
- REST APIs & GraphQL
- Real-time applications (chat, notifications)
- Microservices
- Streaming services
- Serverless functions

---

### 2. What is the Event Loop in Node.js?

**Answer:**
The Event Loop is what allows Node.js to perform non-blocking I/O operations despite being single-threaded.

**How it works:**
```javascript
// 1. Call Stack executes synchronous code
console.log('Start');

// 2. Async operations go to Web APIs / C++ APIs
setTimeout(() => {
  console.log('Timeout');
}, 0);

// 3. Callbacks go to Callback Queue
Promise.resolve().then(() => {
  console.log('Promise');
});

// 4. Microtasks (Promises) have priority
console.log('End');

// Output: Start, End, Promise, Timeout
```

**Event Loop Phases:**
1. **Timers** - setTimeout, setInterval callbacks
2. **Pending callbacks** - I/O callbacks deferred to next loop
3. **Idle, prepare** - Internal use
4. **Poll** - Retrieve new I/O events, execute I/O callbacks
5. **Check** - setImmediate callbacks
6. **Close callbacks** - socket.on('close', ...)

---

### 3. Difference between `process.nextTick()`, `setImmediate()`, and `setTimeout()`?

**Answer:**

```javascript
// 1. process.nextTick() - Runs immediately after current operation
process.nextTick(() => {
  console.log('Runs before any other I/O events');
});

// 2. setImmediate() - Runs in the Check phase
setImmediate(() => {
  console.log('Runs in Check phase');
});

// 3. setTimeout() - Runs in Timers phase
setTimeout(() => {
  console.log('Runs in Timers phase');
}, 0);

// Order: nextTick -> setImmediate -> setTimeout (usually)
```

| Method | Phase | When it runs |
|--------|-------|--------------|
| `process.nextTick()` | Not a phase | Immediately after current operation |
| `setImmediate()` | Check phase | After I/O events |
| `setTimeout()` | Timers phase | After specified delay |

**Pro Tip:** `process.nextTick()` can starve I/O if used recursively!

---

### 4. What are Promises and async/await?

**Answer:**

**Promises:**
```javascript
// Promise creation
const fetchData = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('Data fetched');
    // reject('Error occurred');
  }, 1000);
});

// Using promise
fetchData
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

**Async/Await (Syntactic sugar over Promises):**
```javascript
const fetchData = async () => {
  try {
    const data = await someAsyncOperation();
    console.log(data);
  } catch (error) {
    console.error(error);
  }
};

fetchData();
```

---

### 5. What is middleware in Express.js?

**Answer:**
Middleware functions are functions that have access to the request object (req), response object (res), and the next middleware function in the application's request-response cycle.

```javascript
const express = require('express');
const app = express();

// Built-in middleware
app.use(express.json()); // Parse JSON bodies
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded bodies

// Custom middleware
const logger = (req, res, next) => {
  console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
  next(); // Pass control to next middleware
};

app.use(logger);

// Authentication middleware
const authenticate = (req, res, next) => {
  const token = req.headers.authorization;
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  // Verify token
  req.user = { id: 1, role: 'user' };
  next();
};

// Route-specific middleware
app.get('/profile', authenticate, (req, res) => {
  res.json({ user: req.user });
});

app.listen(3000);
```

---

### 6. Explain REST API and HTTP methods.

**Answer:**
REST (Representational State Transfer) is an architectural style for designing networked applications.

**HTTP Methods:**

| Method | Purpose | Example | Idempotent |
|--------|---------|---------|------------|
| GET | Retrieve data | `GET /api/users` | Yes |
| POST | Create new resource | `POST /api/users` | No |
| PUT | Update/Replace resource | `PUT /api/users/1` | Yes |
| PATCH | Partially update | `PATCH /api/users/1` | No |
| DELETE | Remove resource | `DELETE /api/users/1` | Yes |

**RESTful Routes Example:**
```javascript
// GET all users
app.get('/api/users', getAllUsers);

// GET single user
app.get('/api/users/:id', getUser);

// POST create user
app.post('/api/users', createUser);

// PUT update user
app.put('/api/users/:id', updateUser);

// DELETE user
app.delete('/api/users/:id', deleteUser);
```

---

### 7. What is the difference between `==` and `===`?

**Answer:**

```javascript
// == (Loose equality) - Performs type coercion
console.log(5 == '5');        // true (converts string to number)
console.log(0 == false);      // true
console.log(null == undefined); // true

// === (Strict equality) - No type coercion
console.log(5 === '5');       // false (different types)
console.log(0 === false);     // false
console.log(null === undefined); // false

// Best Practice: Always use === for predictable comparisons
```

---

### 8. What are Streams in Node.js?

**Answer:**
Streams are objects that let you read data from a source or write data to a destination in a continuous manner.

**Types of Streams:**

```javascript
const fs = require('fs');

// 1. Readable Stream
const readable = fs.createReadStream('input.txt', { encoding: 'utf8' });
readable.on('data', (chunk) => {
  console.log('Received chunk:', chunk);
});

// 2. Writable Stream
const writable = fs.createWriteStream('output.txt');
writable.write('Hello ');
writable.write('World!');
writable.end();

// 3. Piping (connect readable to writable)
readable.pipe(writable);

// 4. Duplex Stream (both readable and writable)
// Example: net.Socket

// 5. Transform Stream (modify data while reading/writing)
const { Transform } = require('stream');
const upperCase = new Transform({
  transform(chunk, encoding, callback) {
    callback(null, chunk.toString().toUpperCase());
  }
});

fs.createReadStream('input.txt')
  .pipe(upperCase)
  .pipe(fs.createWriteStream('output.txt'));
```

**Why use Streams?**
- Memory efficient (don't load entire file into memory)
- Time efficient (start processing immediately)
- Handle large files gracefully

---

### 9. What is `package.json`?

**Answer:**
`package.json` is the configuration file for Node.js projects that contains metadata and dependencies.

```json
{
  "name": "my-api",
  "version": "1.0.0",
  "description": "My REST API",
  "main": "src/app.js",
  "scripts": {
    "start": "node src/app.js",
    "dev": "nodemon src/app.js",
    "test": "jest",
    "build": "webpack --mode production"
  },
  "keywords": ["api", "nodejs", "express"],
  "author": "Your Name",
  "license": "MIT",
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.0.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.22",
    "jest": "^29.5.0"
  }
}
```

**Key Sections:**
- `dependencies` - Production packages
- `devDependencies` - Development-only packages
- `scripts` - Command shortcuts
- `main` - Entry point

---

### 10. What is Callback Hell and how to avoid it?

**Answer:**
Callback Hell occurs when callbacks are nested within callbacks, making code hard to read and maintain.

**Bad (Callback Hell):**
```javascript
getUser(userId, (err, user) => {
  if (err) return handleError(err);
  
  getPosts(user.id, (err, posts) => {
    if (err) return handleError(err);
    
    getComments(posts[0].id, (err, comments) => {
      if (err) return handleError(err);
      
      getLikes(comments[0].id, (err, likes) => {
        if (err) return handleError(err);
        console.log(likes);
      });
    });
  });
});
```

**Good (Using async/await):**
```javascript
try {
  const user = await getUser(userId);
  const posts = await getPosts(user.id);
  const comments = await getComments(posts[0].id);
  const likes = await getLikes(comments[0].id);
  console.log(likes);
} catch (error) {
  handleError(error);
}
```

**Good (Using Promises):**
```javascript
getUser(userId)
  .then(user => getPosts(user.id))
  .then(posts => getComments(posts[0].id))
  .then(comments => getLikes(comments[0].id))
  .then(likes => console.log(likes))
  .catch(handleError);
```

---

## Intermediate Level (1-2 years)

### 11. How does authentication work in Node.js?

**Answer:**

**JWT Authentication Flow:**

```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

// Login
app.post('/api/login', async (req, res) => {
  const { email, password } = req.body;
  
  // 1. Find user
  const user = await User.findOne({ email });
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // 2. Verify password
  const isPasswordValid = await bcrypt.compare(password, user.password);
  if (!isPasswordValid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // 3. Generate JWT token
  const token = jwt.sign(
    { id: user._id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: '7d' }
  );
  
  res.json({ token, user: { id: user._id, email: user.email } });
});

// Authentication middleware
const authenticateToken = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'Access denied' });
  }
  
  try {
    const verified = jwt.verify(token, process.env.JWT_SECRET);
    req.user = verified;
    next();
  } catch (error) {
    res.status(400).json({ error: 'Invalid token' });
  }
};

// Protected route
app.get('/api/profile', authenticateToken, (req, res) => {
  res.json({ userId: req.user.id });
});
```

---

### 12. What is the difference between SQL and NoSQL databases?

**Answer:**

| Feature | SQL (PostgreSQL, MySQL) | NoSQL (MongoDB, Redis) |
|---------|------------------------|------------------------|
| Data Model | Tables with rows/columns | Documents, Key-Value, Graph |
| Schema | Fixed schema | Flexible/Dynamic schema |
| Scaling | Vertical (bigger server) | Horizontal (more servers) |
| ACID | Strong ACID compliance | BASE (eventual consistency) |
| Joins | Supports complex joins | Embedded documents or references |
| Best For | Complex queries, transactions | Flexible data, rapid iteration |

**When to use SQL:**
- Complex transactions (banking, e-commerce)
- Structured data with relationships
- Need ACID compliance

**When to use NoSQL:**
- Rapid prototyping
- Unstructured/semi-structured data
- Need horizontal scaling
- Real-time analytics

---

### 13. How do you handle errors in Express.js?

**Answer:**

```javascript
// 1. Try-catch in async routes
app.get('/api/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    next(error); // Pass to error handler
  }
});

// 2. Async error wrapper
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get('/api/users/:id', asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  res.json(user);
}));

// 3. Global error handler middleware
const errorHandler = (err, req, res, next) => {
  console.error(err.stack);
  
  // Default error
  const statusCode = err.statusCode || 500;
  const message = err.message || 'Internal Server Error';
  
  res.status(statusCode).json({
    error: message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
};

app.use(errorHandler);

// 4. Custom error class
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

// Usage
app.get('/api/users/:id', asyncHandler(async (req, res, next) => {
  const user = await User.findById(req.params.id);
  if (!user) {
    return next(new AppError('User not found', 404));
  }
  res.json(user);
}));
```

---

### 14. What is CORS and how to configure it?

**Answer:**
CORS (Cross-Origin Resource Sharing) controls which domains can access your API.

```javascript
const cors = require('cors');

// Allow all origins (development only)
app.use(cors());

// Production configuration
app.use(cors({
  origin: ['https://yourdomain.com', 'https://www.yourdomain.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true, // Allow cookies
  maxAge: 86400 // Cache preflight for 24 hours
}));

// Dynamic origin
app.use(cors({
  origin: (origin, callback) => {
    const allowed = process.env.ALLOWED_ORIGINS.split(',');
    if (!origin || allowed.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  }
}));
```

---

### 15. Explain connection pooling in databases.

**Answer:**
Connection pooling reuses database connections instead of creating new ones for each request.

**Why it matters:**
- Creating connections is expensive (TCP handshake, authentication)
- Databases have connection limits
- Improves performance significantly

**Example with PostgreSQL:**
```javascript
const { Pool } = require('pg');

const pool = new Pool({
  host: 'localhost',
  port: 5432,
  database: 'mydb',
  user: 'admin',
  password: process.env.DB_PASSWORD,
  max: 20, // Maximum connections in pool
  idleTimeoutMillis: 30000, // Close idle connections after 30s
  connectionTimeoutMillis: 2000 // Fail if can't connect in 2s
});

// Using pool
app.get('/api/users', async (req, res) => {
  const client = await pool.connect();
  try {
    const result = await client.query('SELECT * FROM users');
    res.json(result.rows);
  } finally {
    client.release(); // Return connection to pool
  }
});

// Or simpler
const result = await pool.query('SELECT * FROM users');
```

**MongoDB connection pooling:**
```javascript
mongoose.connect(process.env.MONGODB_URI, {
  maxPoolSize: 10, // Default is 100
  minPoolSize: 5,
  serverSelectionTimeoutMS: 5000
});
```

---

### 16. What are environment variables and why use them?

**Answer:**
Environment variables store configuration values outside your code (secrets, API keys, database URLs).

```javascript
// .env file (NEVER commit to Git)
DATABASE_URL=mongodb://localhost:27017/mydb
JWT_SECRET=my-super-secret-key
PORT=3000
NODE_ENV=production

// Load with dotenv
require('dotenv').config();

// Access variables
const dbUrl = process.env.DATABASE_URL;
const port = process.env.PORT || 3000;

// Validate required variables
const required = ['DATABASE_URL', 'JWT_SECRET', 'NODE_ENV'];
required.forEach(varName => {
  if (!process.env[varName]) {
    throw new Error(`Missing required env variable: ${varName}`);
  }
});
```

---

### 17. How do you validate user input?

**Answer:**

**Using Joi:**
```javascript
const Joi = require('joi');

const userSchema = Joi.object({
  name: Joi.string().min(3).max(50).required(),
  email: Joi.string().email().required(),
  password: Joi.string()
    .min(8)
    .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/)
    .required()
    .messages({
      'string.pattern.base': 'Password must contain uppercase, lowercase, number, and special character'
    }),
  age: Joi.number().integer().min(18).max(120).optional()
});

app.post('/api/users', async (req, res) => {
  const { error, value } = userSchema.validate(req.body);
  
  if (error) {
    return res.status(400).json({ 
      error: error.details.map(d => d.message).join(', ') 
    });
  }
  
  // Create user with validated data
  const user = await User.create(value);
  res.status(201).json(user);
});
```

**Using Zod:**
```javascript
const z = require('zod');

const userSchema = z.object({
  name: z.string().min(3).max(50),
  email: z.string().email(),
  password: z.string().min(8),
  age: z.number().int().min(18).optional()
});

app.post('/api/users', async (req, res) => {
  const result = userSchema.safeParse(req.body);
  
  if (!result.success) {
    return res.status(400).json({ error: result.error.errors });
  }
  
  const user = await User.create(result.data);
  res.status(201).json(user);
});
```

---

### 18. What is the purpose of `index.js` and how to structure a Node.js project?

**Answer:**

**Standard Project Structure:**
```
project/
├── src/
│   ├── controllers/      # Request handlers
│   │   └── userController.js
│   ├── routes/           # API routes
│   │   └── userRoutes.js
│   ├── models/           # Database models
│   │   └── User.js
│   ├── services/         # Business logic
│   │   └── userService.js
│   ├── middleware/       # Custom middleware
│   │   ├── auth.js
│   │   └── validate.js
│   ├── utils/            # Helper functions
│   │   └── logger.js
│   ├── config/           # Configuration
│   │   └── database.js
│   └── app.js            # Express setup
├── tests/                # Test files
├── .env                  # Environment variables
├── .gitignore
├── package.json
└── README.md
```

---

## Advanced Level (2+ years)

### 19. How do you scale a Node.js application?

**Answer:**

**1. Vertical Scaling (Bigger Server):**
```javascript
// Increase memory limit
node --max-old-space-size=4096 app.js  // 4GB
```

**2. Horizontal Scaling (Multiple Instances):**
```javascript
// Using Cluster module
const cluster = require('cluster');
const os = require('os');

if (cluster.isMaster) {
  const numCPUs = os.cpus().length;
  console.log(`Master ${process.pid} is running`);
  
  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // Replace dead worker
  });
} else {
  // Workers share the TCP connection
  require('./app.js');
  console.log(`Worker ${process.pid} started`);
}
```

**3. PM2 Clustering:**
```bash
pm2 start app.js -i max  # Auto-detect CPU cores
pm2 scale api +2         # Add 2 more instances
```

**4. Microservices Architecture:**
```
api-gateway:3000
├── user-service:3001
├── product-service:3002
├── order-service:3003
└── notification-service:3004
```

**5. Load Balancing (NGINX):**
```nginx
upstream nodejs {
  server localhost:3001;
  server localhost:3002;
  server localhost:3003;
}

server {
  listen 80;
  location / {
    proxy_pass http://nodejs;
  }
}
```

---

### 20. How do you implement caching in Node.js?

**Answer:**

```javascript
const Redis = require('ioredis');
const redis = new Redis();

// Cache middleware
const cache = (key, ttl = 3600) => {
  return async (req, res, next) => {
    try {
      const cached = await redis.get(key);
      if (cached) {
        return res.json(JSON.parse(cached));
      }
      
      // Override res.json to cache response
      const originalJson = res.json.bind(res);
      res.json = (body) => {
        redis.setex(key, ttl, JSON.stringify(body));
        originalJson(body);
      };
      
      next();
    } catch (error) {
      next(); // Continue without caching if Redis fails
    }
  };
};

app.get('/api/products', cache('products', 1800), async (req, res) => {
  const products = await Product.find();
  res.json(products);
});
```

---

### 21. What is the difference between `fork`, `spawn`, and `exec`?

**Answer:**

```javascript
const { fork, spawn, exec } = require('child_process');

// 1. spawn - Creates new process, returns stream
const ls = spawn('ls', ['-lh', '/usr']);
ls.stdout.on('data', (data) => console.log(`stdout: ${data}`));
ls.stderr.on('data', (data) => console.error(`stderr: ${data}`));

// Best for: Large output, streaming data

// 2. exec - Creates shell process, buffers output
exec('ls -lh /usr', (error, stdout, stderr) => {
  if (error) return console.error(error);
  console.log(stdout);
});

// Best for: Small output, shell commands

// 3. fork - Special spawn for Node.js modules, IPC
const child = fork('./worker.js');
child.send({ type: 'START', data: 'process this' });
child.on('message', (msg) => {
  console.log('From child:', msg);
});

// Best for: Running Node.js files, communication between processes
```

| Method | Returns | Communication | Use Case |
|--------|---------|---------------|----------|
| `spawn()` | Stream | No | Large output, binary files |
| `exec()` | Buffer | No | Shell commands, small output |
| `fork()` | ChildProcess | Yes (IPC) | Node.js modules |

---

### 22. How do you handle file uploads?

**Answer:**

```javascript
const multer = require('multer');
const path = require('path');

// Configure storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    const uniqueName = `${Date.now()}-${file.originalname}`;
    cb(null, uniqueName);
  }
});

// File filter
const fileFilter = (req, file, cb) => {
  const allowedTypes = /jpeg|jpg|png|gif/;
  const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
  const mimetype = allowedTypes.test(file.mimetype);
  
  if (mimetype && extname) {
    cb(null, true);
  } else {
    cb(new Error('Only images are allowed'));
  }
};

const upload = multer({
  storage,
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB
  fileFilter
});

// Single file upload
app.post('/api/upload', upload.single('avatar'), (req, res) => {
  res.json({ file: req.file });
});

// Multiple files
app.post('/api/uploads', upload.array('photos', 5), (req, res) => {
  res.json({ files: req.files });
});

// Multiple fields
app.post('/api/profile', upload.fields([
  { name: 'avatar', maxCount: 1 },
  { name: 'gallery', maxCount: 10 }
]), (req, res) => {
  res.json({ files: req.files, body: req.body });
});
```

---

### 23. Explain the difference between authentication and authorization.

**Answer:**

**Authentication (Who are you?):**
- Verifies user identity
- Login with username/password
- OAuth, JWT tokens

**Authorization (What can you do?):**
- Checks user permissions
- Role-based access control
- Resource permissions

```javascript
// Authentication middleware
const authenticate = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'Not authenticated' });
  }
  
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch (error) {
    res.status(403).json({ error: 'Invalid token' });
  }
};

// Authorization middleware
const authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Not authorized' });
    }
    next();
  };
};

// Usage
app.delete('/api/users/:id', 
  authenticate,           // First: verify identity
  authorize('admin'),     // Then: check permissions
  deleteUser
);
```

---

### 24. What are WebSockets and when to use them?

**Answer:**
WebSockets provide full-duplex communication channels over a single TCP connection.

**Use Cases:**
- Real-time chat applications
- Live notifications
- Collaborative editing
- Live sports updates
- Online gaming

**Socket.io Implementation:**

```javascript
const http = require('http');
const { Server } = require('socket.io');
const express = require('express');

const app = express();
const server = http.createServer(app);
const io = new Server(server, {
  cors: { origin: '*' }
});

// Connection
io.on('connection', (socket) => {
  console.log('User connected:', socket.id);
  
  // Join room
  socket.on('join-room', (roomId) => {
    socket.join(roomId);
    socket.to(roomId).emit('user-joined', socket.id);
  });
  
  // Chat message
  socket.on('send-message', ({ roomId, message }) => {
    io.to(roomId).emit('receive-message', {
      sender: socket.id,
      message,
      timestamp: new Date()
    });
  });
  
  // Typing indicator
  socket.on('typing', (roomId) => {
    socket.to(roomId).emit('user-typing', socket.id);
  });
  
  // Disconnect
  socket.on('disconnect', () => {
    console.log('User disconnected:', socket.id);
  });
});

// Emit to all connected clients
io.emit('announcement', 'Server will restart in 5 minutes');

// Emit to specific room
io.to('room123').emit('message', 'Hello room!');

server.listen(3000);
```

**Client-side:**
```javascript
const socket = io('http://localhost:3000');

socket.emit('join-room', 'room123');

socket.on('receive-message', (data) => {
  console.log(data.message);
});

socket.emit('send-message', {
  roomId: 'room123',
  message: 'Hello everyone!'
});
```

---

### 25. How do you implement rate limiting?

**Answer:**

```javascript
const rateLimit = require('express-rate-limit');

// Global rate limiter
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per window
  message: {
    error: 'Too many requests, please try again later',
    retryAfter: '15 minutes'
  },
  standardHeaders: true, // Return rate limit info in headers
  legacyHeaders: false
});

app.use(globalLimiter);

// Strict limiter for auth routes
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // Only 5 login attempts per 15 minutes
  message: { error: 'Too many login attempts' },
  skipSuccessfulRequests: true
});

app.use('/api/auth/login', authLimiter);

// Redis-based rate limiter (for distributed systems)
const RedisStore = require('rate-limit-redis');

const redisLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'ratelimit:'
  }),
  windowMs: 15 * 60 * 1000,
  max: 100
});
```

---

## Coding Challenges

### 26. Create a simple Express server with CRUD operations

```javascript
const express = require('express');
const app = express();

app.use(express.json());

// In-memory storage
let users = [];
let nextId = 1;

// CREATE
app.post('/api/users', (req, res) => {
  const { name, email } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({ error: 'Name and email required' });
  }
  
  const user = { id: nextId++, name, email };
  users.push(user);
  
  res.status(201).json(user);
});

// READ ALL
app.get('/api/users', (req, res) => {
  res.json(users);
});

// READ ONE
app.get('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  res.json(user);
});

// UPDATE
app.put('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  user.name = req.body.name || user.name;
  user.email = req.body.email || user.email;
  
  res.json(user);
});

// DELETE
app.delete('/api/users/:id', (req, res) => {
  const index = users.findIndex(u => u.id === parseInt(req.params.id));
  
  if (index === -1) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  users.splice(index, 1);
  res.status(204).send();
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

---

### 27. Implement a promise from scratch

```javascript
class MyPromise {
  constructor(executor) {
    this.state = 'pending';
    this.value = null;
    this.callbacks = [];
    
    const resolve = (value) => {
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
        this.callbacks.forEach(cb => cb.onFulfilled(value));
      }
    };
    
    const reject = (error) => {
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.value = error;
        this.callbacks.forEach(cb => cb.onRejected(error));
      }
    };
    
    try {
      executor(resolve, reject);
    } catch (error) {
      reject(error);
    }
  }
  
  then(onFulfilled) {
    return new MyPromise((resolve, reject) => {
      this.callbacks.push({
        onFulfilled: (value) => {
          try {
            const result = onFulfilled(value);
            resolve(result);
          } catch (error) {
            reject(error);
          }
        }
      });
    });
  }
  
  catch(onRejected) {
    return new MyPromise((resolve, reject) => {
      this.callbacks.push({
        onRejected: (error) => {
          try {
            const result = onRejected(error);
            reject(result);
          } catch (e) {
            reject(e);
          }
        }
      });
    });
  }
}
```

---

### 28. Create a custom middleware for logging

```javascript
const fs = require('fs');
const path = require('path');

const logger = (options = {}) => {
  const logFile = options.file || 'app.log';
  const logPath = path.join(__dirname, 'logs', logFile);
  
  return (req, res, next) => {
    const start = Date.now();
    
    // Log after response is sent
    res.on('finish', () => {
      const duration = Date.now() - start;
      const logEntry = `[${new Date().toISOString()}] ${req.method} ${req.url} ${res.statusCode} - ${duration}ms\n`;
      
      console.log(logEntry.trim());
      
      // Write to file
      fs.appendFile(logPath, logEntry, (err) => {
        if (err) console.error('Failed to write log');
      });
    });
    
    next();
  };
};

app.use(logger({ file: 'api.log' }));
```

---

## System Design Questions

### 29. How would you design a URL shortener?

**Answer:**

```javascript
// Key considerations:
// 1. Generate unique short codes (6-8 characters)
// 2. Fast redirect (301/302)
// 3. Analytics tracking
// 4. Custom aliases
// 5. Expiration dates

const crypto = require('crypto');

class URLShortener {
  constructor(redisClient, db) {
    this.redis = redisClient;
    this.db = db;
  }
  
  // Generate short URL
  async shorten(longUrl, customAlias = null) {
    const base62 = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    
    // Check for custom alias
    if (customAlias) {
      const exists = await this.redis.get(`url:${customAlias}`);
      if (exists) throw new Error('Alias already taken');
      
      await this.redis.set(`url:${customAlias}`, longUrl);
      await this.db.urls.create({ shortCode: customAlias, longUrl });
      return { shortCode: customAlias, longUrl };
    }
    
    // Generate unique code
    let shortCode;
    do {
      const hash = crypto.randomBytes(4).toString('base64');
      shortCode = hash.replace(/[^A-Za-z0-9]/g, '').slice(0, 6);
    } while (await this.redis.get(`url:${shortCode}`));
    
    // Store in cache and database
    await this.redis.set(`url:${shortCode}`, longUrl);
    await this.db.urls.create({ shortCode, longUrl });
    
    return { shortCode, longUrl };
  }
  
  // Redirect
  async redirect(shortCode) {
    // Try cache first
    let longUrl = await this.redis.get(`url:${shortCode}`);
    
    if (!longUrl) {
      // Fallback to database
      const record = await this.db.urls.findOne({ shortCode });
      if (!record) return null;
      
      longUrl = record.longUrl;
      // Update cache
      await this.redis.set(`url:${shortCode}`, longUrl, 86400);
    }
    
    // Track analytics
    await this.db.analytics.create({ shortCode, accessedAt: new Date() });
    
    return longUrl;
  }
}
```

---

### 30. How would you handle 10,000 requests per second?

**Answer:**

**Architecture:**
```
Load Balancer (NGINX/ALB)
├── API Gateway (Rate Limiting, Auth)
│   ├── Service 1 (PM2 Cluster - 4 instances)
│   ├── Service 2 (PM2 Cluster - 4 instances)
│   └── Service 3 (PM2 Cluster - 4 instances)
├── Redis Cluster (Caching, Sessions)
├── Message Queue (RabbitMQ/Kafka)
└── Database Cluster (Read Replicas)
```

**Key Strategies:**
1. **Load Balancing** - Distribute traffic
2. **Caching** - Redis for frequent queries
3. **Connection Pooling** - Reuse DB connections
4. **Async Processing** - Queue heavy tasks
5. **CDN** - Static assets
6. **Horizontal Scaling** - Multiple servers
7. **Database Optimization** - Indexes, read replicas
8. **Monitoring** - Real-time metrics

---

## Practical Scenarios

### 31. Your API is slow. How do you debug it?

**Answer:**

```javascript
// 1. Add timing middleware
const performanceMonitor = (req, res, next) => {
  const start = process.hrtime();
  
  res.on('finish', () => {
    const duration = process.hrtime(start);
    const ms = (duration[0] * 1000) + (duration[1] / 1e6);
    
    if (ms > 1000) { // Log slow requests
      console.warn(`SLOW: ${req.method} ${req.url} - ${ms.toFixed(2)}ms`);
    }
  });
  
  next();
};

// 2. Profile database queries
const getUsers = async () => {
  const start = Date.now();
  const users = await User.find();
  console.log(`DB Query: ${Date.now() - start}ms`);
  return users;
};

// 3. Use profiling
// node --prof app.js
// node --prof-process isolate-*.log > profile.txt

// 4. Memory profiling
// node --inspect app.js
// Check Chrome DevTools > Memory

// Solutions:
// - Add database indexes
// - Implement caching
// - Optimize queries (avoid N+1)
// - Use pagination
// - Compress responses
```

---

### 32. How do you secure a Node.js application?

**Answer:**

```javascript
const helmet = require('helmet');
const cors = require('cors');
const rateLimit = require('express-rate-limit');
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss-clean');
const hpp = require('hpp');

// Security middleware
app.use(helmet()); // Security headers
app.use(cors({ origin: process.env.ALLOWED_ORIGINS })); // CORS
app.use(rateLimit({ windowMs: 15*60*1000, max: 100 })); // Rate limit
app.use(mongoSanitize()); // Prevent NoSQL injection
app.use(xss()); // Prevent XSS
app.use(hpp()); // Prevent HTTP parameter pollution
app.use(express.json({ limit: '10kb' })); // Limit body size

// Additional measures:
// - Use HTTPS
// - Hash passwords (bcrypt)
// - Use parameterized queries
// - Validate all inputs
// - Implement authentication
// - Use JWT with short expiry
// - Set secure cookie flags
// - Regular dependency audits
// - Environment variables for secrets
```

---

### 33. Explain the difference between `module.exports` and `exports`

**Answer:**

```javascript
// module.exports - The actual object returned
// exports - Shorthand reference to module.exports

// Method 1: module.exports
module.exports = {
  name: 'John',
  greet: () => console.log('Hello')
};

// Method 2: exports (add properties)
exports.name = 'John';
exports.greet = () => console.log('Hello');

// WRONG - This breaks the reference
exports = { name: 'John' }; // Won't work!

// CORRECT - Use module.exports for objects
module.exports = { name: 'John' }; // Works!

// Rule: Use module.exports for assigning objects, exports for adding properties
```

---

## Quick Fire Questions

### 34-50. Rapid Fire

**Q: What is `__dirname` and `__filename`?**
A: `__dirname` - Current directory path, `__filename` - Current file path

**Q: Difference between `process.env` and `.env`?**
A: `process.env` - Environment variables object, `.env` - File to store them

**Q: What does `npm install` do?**
A: Reads package.json and installs all dependencies

**Q: What is a package.json lock file?**
A: Ensures consistent installs across environments (package-lock.json)

**Q: How to run Node.js in debug mode?**
A: `node --inspect app.js`

**Q: What is the default PORT for Node.js?**
A: No default, commonly 3000 or 8080

**Q: What is callback?**
A: Function passed as argument, executed after operation completes

**Q: What are buffers?**
A: Temporary memory storage for binary data

**Q: What is REPL?**
A: Read-Eval-Print-Loop (Node.js interactive shell)

**Q: How to exit from Node.js code?**
A: `process.exit()` or `process.exit(1)` for error

**Q: What is the use of `cluster` module?**
A: Create child processes to utilize multi-core systems

**Q: What is piping?**
A: Connecting readable stream to writable stream: `readable.pipe(writable)`

**Q: What is the V8 engine?**
A: Google's open-source JavaScript engine that Node.js uses

**Q: What is the difference between `readFile` and `createReadStream`?**
A: `readFile` loads entire file to memory, `createReadStream` reads in chunks

**Q: What is callback hell?**
A: Deeply nested callbacks making code unreadable

**Q: How to handle uncaught exceptions?**
A: `process.on('uncaughtException', callback)`

**Q: What is the use of `util` module?**
A: Utility functions for debugging, formatting, type checking

---

## Interview Tips

### Before the Interview
1. Review fundamentals (Event Loop, Promises, Closures)
2. Practice coding challenges
3. Build 2-3 complete projects
4. Understand your past projects deeply
5. Learn system design basics

### During the Interview
1. Think out loud
2. Ask clarifying questions
3. Start simple, then optimize
4. Write clean, readable code
5. Test your code
6. Admit what you don't know

### After Coding
1. Discuss time/space complexity
2. Mention edge cases
3. Suggest improvements
4. Ask for feedback

---

**Good luck with your interviews! 🚀**
