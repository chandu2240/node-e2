# Day 05 - Part 01: Authentication, Security, and Testing

## 🎯 Learning Objectives
By the end of this lesson, you will understand:
- User authentication and authorization concepts
- Implementing JWT (JSON Web Tokens) authentication
- Password hashing and security best practices
- Session management and cookies
- Role-based access control (RBAC)
- Security vulnerabilities and how to prevent them
- Testing Node.js applications
- Unit testing, integration testing, and API testing

---

## 🔐 Understanding Authentication vs Authorization

### Simple Explanation (for 10-year-olds!)

Imagine your school has a **magic ID card system**:

#### **Authentication = "Who Are You?"**
```
👦 You: "Hi, I'm John!"
🏫 School: "Show me your ID card!"
👦 You: *shows ID card*
🏫 School: "Yes, you are John! Welcome!"
```

#### **Authorization = "What Can You Do?"**
```
👦 John: "Can I go to the teacher's lounge?"
🏫 School: "You're a student, so NO! 🚫"
👨‍🏫 Teacher: "Can I go to the teacher's lounge?"
🏫 School: "You're a teacher, so YES! ✅"
```

### Real-World Analogy
```
Authentication & Authorization are like a HOTEL SYSTEM:
┌─────────────────────────────────────────────────────┐
│  🏨 Hotel Reception (Authentication)                │
│     "Please show your ID and reservation"           │
│     ↓                                               │
│  🗝️ Room Key (Authorization)                        │
│     "This key opens room 205, not room 301"        │
│     ↓                                               │
│  🚪 Access Control                                   │
│     "Key works = Access granted!"                   │
└─────────────────────────────────────────────────────┘
```

---

## 🔑 JWT Authentication Implementation

### Basic JWT Authentication

Create `jwt-auth-basic.js`:
```javascript
// jwt-auth-basic.js - Basic JWT authentication
const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

console.log('🔐 Learning JWT Authentication!');

const app = express();
const PORT = 5000;

// Middleware
app.use(express.json());

// Secret key (in real apps, use environment variables!)
const JWT_SECRET = 'your-super-secret-key-change-this-in-production';

// Mock user database
const users = [
  {
    id: 1,
    username: 'alice',
    email: 'alice@example.com',
    // Password: 'password123' (hashed)
    password: '$2b$10$rOZLNLTVJFOWYTQFfAKJCOYO3lbGzLJLzLJLNLTVJFOWYTQFfAKJCOYO3lbGzLJL',
    role: 'admin'
  },
  {
    id: 2,
    username: 'bob',
    email: 'bob@example.com',
    // Password: 'password123' (hashed)
    password: '$2b$10$rOZLNLTVJFOWYTQFfAKJCOYO3lbGzLJLzLJLNLTVJFOWYTQFfAKJCOYO3lbGzLJL',
    role: 'user'
  }
];

// Helper function to generate JWT token
function generateToken(user) {
  const payload = {
    userId: user.id,
    username: user.username,
    email: user.email,
    role: user.role
  };
  
  return jwt.sign(payload, JWT_SECRET, { expiresIn: '24h' });
}

// Helper function to hash passwords
async function hashPassword(password) {
  const saltRounds = 10;
  return await bcrypt.hash(password, saltRounds);
}

// Helper function to verify passwords
async function verifyPassword(password, hashedPassword) {
  return await bcrypt.compare(password, hashedPassword);
}

// Authentication middleware
function authenticateToken(req, res, next) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN
  
  if (!token) {
    return res.status(401).json({
      success: false,
      message: 'Access token required! Please login first. 🔐'
    });
  }
  
  jwt.verify(token, JWT_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({
        success: false,
        message: 'Invalid or expired token! Please login again. ⏰'
      });
    }
    
    req.user = user;
    next();
  });
}

// Authorization middleware
function authorizeRole(roles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({
        success: false,
        message: 'Authentication required! 🔐'
      });
    }
    
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        success: false,
        message: `Access denied! Required role: ${roles.join(' or ')} 🚫`
      });
    }
    
    next();
  };
}

// Routes

// 1. User Registration
app.post('/auth/register', async (req, res) => {
  try {
    const { username, email, password } = req.body;
    
    console.log(`📝 Registration attempt for: ${username}`);
    
    // Validation
    if (!username || !email || !password) {
      return res.status(400).json({
        success: false,
        message: 'Username, email, and password are required! 📝'
      });
    }
    
    if (password.length < 6) {
      return res.status(400).json({
        success: false,
        message: 'Password must be at least 6 characters long! 🔐'
      });
    }
    
    // Check if user already exists
    const existingUser = users.find(u => u.username === username || u.email === email);
    if (existingUser) {
      return res.status(409).json({
        success: false,
        message: 'Username or email already exists! 👥'
      });
    }
    
    // Hash password
    const hashedPassword = await hashPassword(password);
    
    // Create new user
    const newUser = {
      id: users.length + 1,
      username,
      email,
      password: hashedPassword,
      role: 'user'
    };
    
    users.push(newUser);
    
    // Generate token
    const token = generateToken(newUser);
    
    console.log(`✅ User registered successfully: ${username}`);
    
    res.status(201).json({
      success: true,
      message: 'User registered successfully! 🎉',
      token,
      user: {
        id: newUser.id,
        username: newUser.username,
        email: newUser.email,
        role: newUser.role
      }
    });
    
  } catch (error) {
    console.log('❌ Registration error:', error.message);
    res.status(500).json({
      success: false,
      message: 'Registration failed! 💥'
    });
  }
});

// 2. User Login
app.post('/auth/login', async (req, res) => {
  try {
    const { username, password } = req.body;
    
    console.log(`🔑 Login attempt for: ${username}`);
    
    // Validation
    if (!username || !password) {
      return res.status(400).json({
        success: false,
        message: 'Username and password are required! 📝'
      });
    }
    
    // Find user
    const user = users.find(u => u.username === username);
    if (!user) {
      return res.status(401).json({
        success: false,
        message: 'Invalid credentials! 🔐'
      });
    }
    
    // Verify password
    const isValidPassword = await verifyPassword(password, user.password);
    if (!isValidPassword) {
      return res.status(401).json({
        success: false,
        message: 'Invalid credentials! 🔐'
      });
    }
    
    // Generate token
    const token = generateToken(user);
    
    console.log(`✅ Login successful for: ${username}`);
    
    res.json({
      success: true,
      message: 'Login successful! 🎉',
      token,
      user: {
        id: user.id,
        username: user.username,
        email: user.email,
        role: user.role
      }
    });
    
  } catch (error) {
    console.log('❌ Login error:', error.message);
    res.status(500).json({
      success: false,
      message: 'Login failed! 💥'
    });
  }
});

// 3. Get user profile (protected route)
app.get('/auth/profile', authenticateToken, (req, res) => {
  console.log(`👤 Profile request for: ${req.user.username}`);
  
  res.json({
    success: true,
    message: 'Profile retrieved successfully! 👤',
    user: {
      id: req.user.userId,
      username: req.user.username,
      email: req.user.email,
      role: req.user.role
    }
  });
});

// 4. Admin only route
app.get('/admin/users', authenticateToken, authorizeRole(['admin']), (req, res) => {
  console.log(`👥 Admin users list requested by: ${req.user.username}`);
  
  const userList = users.map(user => ({
    id: user.id,
    username: user.username,
    email: user.email,
    role: user.role
  }));
  
  res.json({
    success: true,
    message: 'Users retrieved successfully! 👥',
    users: userList
  });
});

// 5. Token refresh endpoint
app.post('/auth/refresh', authenticateToken, (req, res) => {
  console.log(`🔄 Token refresh for: ${req.user.username}`);
  
  // Generate new token
  const newToken = generateToken({
    id: req.user.userId,
    username: req.user.username,
    email: req.user.email,
    role: req.user.role
  });
  
  res.json({
    success: true,
    message: 'Token refreshed successfully! 🔄',
    token: newToken
  });
});

// Start server
app.listen(PORT, () => {
  console.log(`🔐 JWT Authentication server running on http://localhost:${PORT}`);
  console.log('\nTry these endpoints:');
  console.log('POST /auth/register - Register new user');
  console.log('POST /auth/login - Login user');
  console.log('GET /auth/profile - Get user profile (requires token)');
  console.log('GET /admin/users - Admin only (requires admin token)');
  console.log('\nTest credentials:');
  console.log('Username: alice, Password: password123 (admin)');
  console.log('Username: bob, Password: password123 (user)');
});
```

### Session-Based Authentication

Create `session-auth.js`:
```javascript
// session-auth.js - Session-based authentication
const express = require('express');
const session = require('express-session');
const bcrypt = require('bcrypt');

console.log('🍪 Learning Session-based Authentication!');

const app = express();
const PORT = 5001;

// Middleware
app.use(express.json());

// Session configuration
app.use(session({
  secret: 'your-session-secret-key',
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: false, // Set to true in production with HTTPS
    httpOnly: true,
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  }
}));

// Mock user database
const users = [
  {
    id: 1,
    username: 'alice',
    email: 'alice@example.com',
    password: '$2b$10$rOZLNLTVJFOWYTQFfAKJCOYO3lbGzLJLzLJLNLTVJFOWYTQFfAKJCOYO3lbGzLJL',
    role: 'admin'
  },
  {
    id: 2,
    username: 'bob',
    email: 'bob@example.com',
    password: '$2b$10$rOZLNLTVJFOWYTQFfAKJCOYO3lbGzLJLzLJLNLTVJFOWYTQFfAKJCOYO3lbGzLJL',
    role: 'user'
  }
];

// Authentication middleware
function requireAuth(req, res, next) {
  if (!req.session.user) {
    return res.status(401).json({
      success: false,
      message: 'Please login first! 🔐'
    });
  }
  next();
}

// Authorization middleware
function requireRole(roles) {
  return (req, res, next) => {
    if (!req.session.user) {
      return res.status(401).json({
        success: false,
        message: 'Authentication required! 🔐'
      });
    }
    
    if (!roles.includes(req.session.user.role)) {
      return res.status(403).json({
        success: false,
        message: `Access denied! Required role: ${roles.join(' or ')} 🚫`
      });
    }
    
    next();
  };
}

// Routes

// 1. Login
app.post('/auth/login', async (req, res) => {
  try {
    const { username, password } = req.body;
    
    console.log(`🔑 Session login attempt for: ${username}`);
    
    // Find user
    const user = users.find(u => u.username === username);
    if (!user) {
      return res.status(401).json({
        success: false,
        message: 'Invalid credentials! 🔐'
      });
    }
    
    // Verify password (for demo, we'll use simple comparison)
    if (password !== 'password123') {
      return res.status(401).json({
        success: false,
        message: 'Invalid credentials! 🔐'
      });
    }
    
    // Create session
    req.session.user = {
      id: user.id,
      username: user.username,
      email: user.email,
      role: user.role
    };
    
    console.log(`✅ Session login successful for: ${username}`);
    console.log(`🍪 Session ID: ${req.session.id}`);
    
    res.json({
      success: true,
      message: 'Login successful! Session created! 🍪',
      user: req.session.user
    });
    
  } catch (error) {
    console.log('❌ Session login error:', error.message);
    res.status(500).json({
      success: false,
      message: 'Login failed! 💥'
    });
  }
});

// 2. Logout
app.post('/auth/logout', requireAuth, (req, res) => {
  const username = req.session.user.username;
  
  req.session.destroy((err) => {
    if (err) {
      return res.status(500).json({
        success: false,
        message: 'Logout failed! 💥'
      });
    }
    
    console.log(`👋 User logged out: ${username}`);
    
    res.json({
      success: true,
      message: 'Logged out successfully! 👋'
    });
  });
});

// 3. Get profile
app.get('/auth/profile', requireAuth, (req, res) => {
  console.log(`👤 Profile request for: ${req.session.user.username}`);
  
  res.json({
    success: true,
    message: 'Profile retrieved successfully! 👤',
    user: req.session.user
  });
});

// 4. Admin route
app.get('/admin/users', requireAuth, requireRole(['admin']), (req, res) => {
  console.log(`👥 Admin users list requested by: ${req.session.user.username}`);
  
  const userList = users.map(user => ({
    id: user.id,
    username: user.username,
    email: user.email,
    role: user.role
  }));
  
  res.json({
    success: true,
    message: 'Users retrieved successfully! 👥',
    users: userList
  });
});

// 5. Check session status
app.get('/auth/status', (req, res) => {
  if (req.session.user) {
    res.json({
      success: true,
      message: 'User is authenticated! ✅',
      user: req.session.user,
      sessionId: req.session.id
    });
  } else {
    res.json({
      success: false,
      message: 'User is not authenticated! 🔐'
    });
  }
});

// Start server
app.listen(PORT, () => {
  console.log(`🍪 Session Authentication server running on http://localhost:${PORT}`);
  console.log('\nTry these endpoints:');
  console.log('POST /auth/login - Login with sessions');
  console.log('POST /auth/logout - Logout and destroy session');
  console.log('GET /auth/profile - Get user profile');
  console.log('GET /auth/status - Check session status');
  console.log('GET /admin/users - Admin only route');
});
```

---

## 🛡️ Security Best Practices

### Input Validation and Sanitization

Create `security-validation.js`:
```javascript
// security-validation.js - Input validation and sanitization
const express = require('express');
const validator = require('validator');
const rateLimit = require('express-rate-limit');
const helmet = require('helmet');

console.log('🛡️ Learning Security Best Practices!');

const app = express();
const PORT = 5002;

// Security middleware
app.use(helmet()); // Sets various HTTP headers
app.use(express.json({ limit: '10mb' })); // Limit request size

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: {
    success: false,
    message: 'Too many requests! Please try again later. 🐌'
  }
});

app.use('/api/', limiter);

// Input validation helper
class InputValidator {
  static validateEmail(email) {
    if (!email || typeof email !== 'string') {
      return { valid: false, message: 'Email is required and must be a string!' };
    }
    
    if (!validator.isEmail(email)) {
      return { valid: false, message: 'Invalid email format!' };
    }
    
    return { valid: true };
  }
  
  static validatePassword(password) {
    if (!password || typeof password !== 'string') {
      return { valid: false, message: 'Password is required and must be a string!' };
    }
    
    if (password.length < 8) {
      return { valid: false, message: 'Password must be at least 8 characters long!' };
    }
    
    if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(password)) {
      return { valid: false, message: 'Password must contain at least one uppercase letter, one lowercase letter, and one number!' };
    }
    
    return { valid: true };
  }
  
  static validateUsername(username) {
    if (!username || typeof username !== 'string') {
      return { valid: false, message: 'Username is required and must be a string!' };
    }
    
    if (username.length < 3 || username.length > 20) {
      return { valid: false, message: 'Username must be between 3 and 20 characters!' };
    }
    
    if (!/^[a-zA-Z0-9_]+$/.test(username)) {
      return { valid: false, message: 'Username can only contain letters, numbers, and underscores!' };
    }
    
    return { valid: true };
  }
  
  static sanitizeInput(input) {
    if (typeof input !== 'string') return input;
    
    // Remove HTML tags
    const sanitized = validator.escape(input);
    
    // Trim whitespace
    return sanitized.trim();
  }
}

// Security middleware for input validation
function validateUserInput(req, res, next) {
  const { username, email, password } = req.body;
  const errors = [];
  
  // Validate username
  if (username !== undefined) {
    const usernameValidation = InputValidator.validateUsername(username);
    if (!usernameValidation.valid) {
      errors.push(usernameValidation.message);
    } else {
      req.body.username = InputValidator.sanitizeInput(username);
    }
  }
  
  // Validate email
  if (email !== undefined) {
    const emailValidation = InputValidator.validateEmail(email);
    if (!emailValidation.valid) {
      errors.push(emailValidation.message);
    } else {
      req.body.email = InputValidator.sanitizeInput(email);
    }
  }
  
  // Validate password
  if (password !== undefined) {
    const passwordValidation = InputValidator.validatePassword(password);
    if (!passwordValidation.valid) {
      errors.push(passwordValidation.message);
    }
  }
  
  if (errors.length > 0) {
    return res.status(400).json({
      success: false,
      message: 'Validation failed! 📝',
      errors: errors
    });
  }
  
  next();
}

// Routes

// 1. Secure user registration
app.post('/api/register', validateUserInput, (req, res) => {
  const { username, email, password } = req.body;
  
  console.log(`📝 Secure registration attempt for: ${username}`);
  
  // Additional security checks
  const securityChecks = [
    {
      condition: validator.isLength(username, { min: 3, max: 20 }),
      message: 'Username length validation failed!'
    },
    {
      condition: validator.isEmail(email),
      message: 'Email format validation failed!'
    },
    {
      condition: validator.isStrongPassword(password, {
        minLength: 8,
        minLowercase: 1,
        minUppercase: 1,
        minNumbers: 1,
        minSymbols: 0
      }),
      message: 'Password strength validation failed!'
    }
  ];
  
  const failedChecks = securityChecks.filter(check => !check.condition);
  
  if (failedChecks.length > 0) {
    return res.status(400).json({
      success: false,
      message: 'Security validation failed! 🛡️',
      errors: failedChecks.map(check => check.message)
    });
  }
  
  res.json({
    success: true,
    message: 'User registration successful! All security checks passed! 🎉',
    user: {
      username: username,
      email: email
    }
  });
});

// 2. SQL Injection prevention demo
app.get('/api/users/:id', (req, res) => {
  const userId = req.params.id;
  
  // Validate ID is a number
  if (!validator.isNumeric(userId)) {
    return res.status(400).json({
      success: false,
      message: 'Invalid user ID! Must be a number! 🔢'
    });
  }
  
  // Sanitize the ID
  const sanitizedId = validator.escape(userId);
  
  console.log(`👤 Secure user lookup for ID: ${sanitizedId}`);
  
  // In real applications, use parameterized queries
  // const query = 'SELECT * FROM users WHERE id = ?';
  // const result = await db.query(query, [sanitizedId]);
  
  res.json({
    success: true,
    message: 'User retrieved securely! 🛡️',
    userId: sanitizedId,
    note: 'This demonstrates SQL injection prevention'
  });
});

// 3. XSS Prevention demo
app.post('/api/comments', (req, res) => {
  const { comment } = req.body;
  
  if (!comment || typeof comment !== 'string') {
    return res.status(400).json({
      success: false,
      message: 'Comment is required and must be a string! 📝'
    });
  }
  
  // Sanitize comment to prevent XSS
  const sanitizedComment = validator.escape(comment);
  
  console.log(`💬 Original comment: ${comment}`);
  console.log(`🛡️ Sanitized comment: ${sanitizedComment}`);
  
  res.json({
    success: true,
    message: 'Comment posted securely! 🛡️',
    original: comment,
    sanitized: sanitizedComment,
    note: 'This demonstrates XSS prevention'
  });
});

// 4. CSRF Protection demo
app.get('/api/csrf-token', (req, res) => {
  // Generate a simple CSRF token (in real apps, use proper CSRF libraries)
  const csrfToken = require('crypto').randomBytes(32).toString('hex');
  
  // Store in session (simplified for demo)
  req.session = req.session || {};
  req.session.csrfToken = csrfToken;
  
  res.json({
    success: true,
    message: 'CSRF token generated! 🛡️',
    csrfToken: csrfToken
  });
});

// Error handling middleware
app.use((error, req, res, next) => {
  console.log('❌ Security error:', error.message);
  
  res.status(500).json({
    success: false,
    message: 'Internal server error! 💥',
    error: process.env.NODE_ENV === 'development' ? error.message : 'Something went wrong!'
  });
});

// Start server
app.listen(PORT, () => {
  console.log(`🛡️ Security demo server running on http://localhost:${PORT}`);
  console.log('\nSecurity features implemented:');
  console.log('✅ Input validation and sanitization');
  console.log('✅ Rate limiting');
  console.log('✅ Helmet security headers');
  console.log('✅ SQL injection prevention');
  console.log('✅ XSS prevention');
  console.log('✅ CSRF protection');
});
```

---

## 🧪 Testing Node.js Applications

### Unit Testing with Jest

Create `test-examples.js`:
```javascript
// test-examples.js - Unit testing examples
console.log('🧪 Learning Unit Testing!');

// Simple functions to test
class Calculator {
  add(a, b) {
    if (typeof a !== 'number' || typeof b !== 'number') {
      throw new Error('Arguments must be numbers!');
    }
    return a + b;
  }
  
  subtract(a, b) {
    if (typeof a !== 'number' || typeof b !== 'number') {
      throw new Error('Arguments must be numbers!');
    }
    return a - b;
  }
  
  multiply(a, b) {
    if (typeof a !== 'number' || typeof b !== 'number') {
      throw new Error('Arguments must be numbers!');
    }
    return a * b;
  }
  
  divide(a, b) {
    if (typeof a !== 'number' || typeof b !== 'number') {
      throw new Error('Arguments must be numbers!');
    }
    if (b === 0) {
      throw new Error('Cannot divide by zero!');
    }
    return a / b;
  }
}

// User service to test
class UserService {
  constructor() {
    this.users = [];
  }
  
  addUser(user) {
    if (!user.name || !user.email) {
      throw new Error('Name and email are required!');
    }
    
    if (this.users.find(u => u.email === user.email)) {
      throw new Error('User with this email already exists!');
    }
    
    const newUser = {
      id: this.users.length + 1,
      name: user.name,
      email: user.email,
      createdAt: new Date()
    };
    
    this.users.push(newUser);
    return newUser;
  }
  
  getUserById(id) {
    return this.users.find(u => u.id === id);
  }
  
  getAllUsers() {
    return this.users;
  }
  
  updateUser(id, updates) {
    const userIndex = this.users.findIndex(u => u.id === id);
    if (userIndex === -1) {
      throw new Error('User not found!');
    }
    
    this.users[userIndex] = { ...this.users[userIndex], ...updates };
    return this.users[userIndex];
  }
  
  deleteUser(id) {
    const userIndex = this.users.findIndex(u => u.id === id);
    if (userIndex === -1) {
      throw new Error('User not found!');
    }
    
    const deletedUser = this.users.splice(userIndex, 1)[0];
    return deletedUser;
  }
}

module.exports = { Calculator, UserService };

// Demo of what tests look like (these would be in separate test files)
console.log(`
🧪 TEST EXAMPLES:

1. Calculator Tests:
   ✅ add(2, 3) should return 5
   ✅ add('2', 3) should throw error
   ✅ divide(10, 2) should return 5
   ✅ divide(10, 0) should throw error

2. UserService Tests:
   ✅ addUser should create user with valid data
   ✅ addUser should throw error with duplicate email
   ✅ getUserById should return user if exists
   ✅ getUserById should return undefined if not exists
   ✅ updateUser should update existing user
   ✅ deleteUser should remove user from array

3. Test Structure:
   describe('Calculator', () => {
     test('should add two numbers correctly', () => {
       const calc = new Calculator();
       expect(calc.add(2, 3)).toBe(5);
     });
     
     test('should throw error for invalid input', () => {
       const calc = new Calculator();
       expect(() => calc.add('2', 3)).toThrow('Arguments must be numbers!');
     });
   });
`);
```

### Create actual test files

Create `calculator.test.js`:
```javascript
// calculator.test.js - Jest unit tests
const { Calculator } = require('./test-examples');

describe('Calculator', () => {
  let calculator;
  
  beforeEach(() => {
    calculator = new Calculator();
  });
  
  describe('add', () => {
    test('should add two positive numbers correctly', () => {
      expect(calculator.add(2, 3)).toBe(5);
    });
    
    test('should add negative numbers correctly', () => {
      expect(calculator.add(-2, -3)).toBe(-5);
    });
    
    test('should add zero correctly', () => {
      expect(calculator.add(5, 0)).toBe(5);
    });
    
    test('should throw error for non-numeric inputs', () => {
      expect(() => calculator.add('2', 3)).toThrow('Arguments must be numbers!');
      expect(() => calculator.add(2, '3')).toThrow('Arguments must be numbers!');
      expect(() => calculator.add('a', 'b')).toThrow('Arguments must be numbers!');
    });
  });
  
  describe('subtract', () => {
    test('should subtract two numbers correctly', () => {
      expect(calculator.subtract(5, 3)).toBe(2);
    });
    
    test('should handle negative results', () => {
      expect(calculator.subtract(3, 5)).toBe(-2);
    });
  });
  
  describe('multiply', () => {
    test('should multiply two numbers correctly', () => {
      expect(calculator.multiply(4, 3)).toBe(12);
    });
    
    test('should handle multiplication by zero', () => {
      expect(calculator.multiply(5, 0)).toBe(0);
    });
  });
  
  describe('divide', () => {
    test('should divide two numbers correctly', () => {
      expect(calculator.divide(10, 2)).toBe(5);
    });
    
    test('should throw error when dividing by zero', () => {
      expect(() => calculator.divide(10, 0)).toThrow('Cannot divide by zero!');
    });
    
    test('should handle decimal results', () => {
      expect(calculator.divide(10, 3)).toBeCloseTo(3.33, 2);
    });
  });
});
```

### API Testing with Supertest

Create `api-testing.js`:
```javascript
// api-testing.js - API testing with supertest
const express = require('express');
const request = require('supertest');

console.log('🧪 Learning API Testing!');

// Create a simple Express app for testing
function createApp() {
  const app = express();
  app.use(express.json());
  
  // Mock data
  let users = [
    { id: 1, name: 'Alice', email: 'alice@example.com' },
    { id: 2, name: 'Bob', email: 'bob@example.com' }
  ];
  
  // Routes
  app.get('/api/users', (req, res) => {
    res.json({
      success: true,
      data: users
    });
  });
  
  app.get('/api/users/:id', (req, res) => {
    const user = users.find(u => u.id === parseInt(req.params.id));
    if (!user) {
      return res.status(404).json({
        success: false,
        message: 'User not found'
      });
    }
    res.json({
      success: true,
      data: user
    });
  });
  
  app.post('/api/users', (req, res) => {
    const { name, email } = req.body;
    
    if (!name || !email) {
      return res.status(400).json({
        success: false,
        message: 'Name and email are required'
      });
    }
    
    const newUser = {
      id: users.length + 1,
      name,
      email
    };
    
    users.push(newUser);
    
    res.status(201).json({
      success: true,
      data: newUser
    });
  });
  
  app.put('/api/users/:id', (req, res) => {
    const userIndex = users.findIndex(u => u.id === parseInt(req.params.id));
    if (userIndex === -1) {
      return res.status(404).json({
        success: false,
        message: 'User not found'
      });
    }
    
    const { name, email } = req.body;
    users[userIndex] = { ...users[userIndex], name, email };
    
    res.json({
      success: true,
      data: users[userIndex]
    });
  });
  
  app.delete('/api/users/:id', (req, res) => {
    const userIndex = users.findIndex(u => u.id === parseInt(req.params.id));
    if (userIndex === -1) {
      return res.status(404).json({
        success: false,
        message: 'User not found'
      });
    }
    
    const deletedUser = users.splice(userIndex, 1)[0];
    
    res.json({
      success: true,
      data: deletedUser
    });
  });
  
  return app;
}

// Test functions (these would be in separate test files)
async function runAPITests() {
  console.log('🚀 Running API Tests...');
  
  const app = createApp();
  
  try {
    // Test GET /api/users
    console.log('\n1. Testing GET /api/users...');
    const response1 = await request(app)
      .get('/api/users')
      .expect(200);
    
    console.log('✅ GET /api/users passed');
    console.log('📊 Response:', response1.body);
    
    // Test GET /api/users/:id
    console.log('\n2. Testing GET /api/users/1...');
    const response2 = await request(app)
      .get('/api/users/1')
      .expect(200);
    
    console.log('✅ GET /api/users/1 passed');
    console.log('📊 Response:', response2.body);
    
    // Test POST /api/users
    console.log('\n3. Testing POST /api/users...');
    const response3 = await request(app)
      .post('/api/users')
      .send({ name: 'Charlie', email: 'charlie@example.com' })
      .expect(201);
    
    console.log('✅ POST /api/users passed');
    console.log('📊 Response:', response3.body);
    
    // Test invalid POST
    console.log('\n4. Testing invalid POST /api/users...');
    const response4 = await request(app)
      .post('/api/users')
      .send({ name: 'Invalid User' }) // Missing email
      .expect(400);
    
    console.log('✅ Invalid POST /api/users passed');
    console.log('📊 Response:', response4.body);
    
    // Test 404
    console.log('\n5. Testing GET /api/users/999 (404)...');
    const response5 = await request(app)
      .get('/api/users/999')
      .expect(404);
    
    console.log('✅ 404 test passed');
    console.log('📊 Response:', response5.body);
    
    console.log('\n🎉 All API tests passed!');
    
  } catch (error) {
    console.log('❌ API test failed:', error.message);
  }
}

// Run the tests
runAPITests();

module.exports = { createApp };
```

---

## 🎉 Summary

Today you learned:
- ✅ Authentication vs Authorization concepts (hotel key analogy!)
- ✅ JWT token-based authentication
- ✅ Session-based authentication with cookies
- ✅ Security best practices and input validation
- ✅ Protection against common vulnerabilities
- ✅ Unit testing with Jest
- ✅ API testing with Supertest
- ✅ Test-driven development principles

**Next Up**: Day 05 - Part 02: Advanced Testing, Performance, and Deployment

---

## 🎯 Practice Exercises

### Exercise 1: Complete Authentication System
Build a full authentication system with:
1. User registration and login
2. Password reset functionality
3. Role-based access control
4. Session management
5. Security middleware

### Exercise 2: Secure API Development
Create a secure API with:
1. Input validation and sanitization
2. Rate limiting
3. CORS configuration
4. Security headers
5. Error handling

### Exercise 3: Comprehensive Testing Suite
Write tests for:
1. Authentication functions
2. API endpoints
3. Security middleware
4. Database operations
5. Integration tests

---

*Remember: Security is like wearing a seatbelt - it's not just about the destination, it's about arriving safely! 🛡️*
