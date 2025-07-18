# Day 05 - Part 02: Advanced Testing, Performance, and Deployment

## 🎯 Learning Objectives
By the end of this lesson, you will understand:
- Advanced testing strategies and patterns
- Performance optimization techniques
- Memory management and monitoring
- Load testing and stress testing
- Application deployment strategies
- Docker containerization
- Environment configuration
- Production monitoring and logging

---

## 🧪 Advanced Testing Strategies

### Simple Explanation (for 10-year-olds!)

Imagine you're building a **super cool robot**:

#### **Testing is like Quality Control:**
```
🤖 Your Robot (Node.js App)
├── 🔧 Unit Tests = "Does each part work?"
├── 🔗 Integration Tests = "Do parts work together?"
├── 🏃‍♂️ Performance Tests = "Is it fast enough?"
├── 🌍 Load Tests = "Can it handle many users?"
└── 🚀 End-to-End Tests = "Does everything work perfectly?"
```

### Real-World Analogy
```
Testing is like a CAR INSPECTION:
┌─────────────────────────────────────────────────────┐
│  🔍 Engine Check (Unit Tests)                      │
│  🔧 Parts Connection (Integration Tests)           │
│  🏎️ Speed Test (Performance Tests)                 │
│  🚗 Full Road Test (End-to-End Tests)              │
│  🛣️ Heavy Traffic Test (Load Tests)                │
└─────────────────────────────────────────────────────┘
```

---

## 🔗 Integration Testing

### Testing Database Operations

Create `integration-testing.js`:
```javascript
// integration-testing.js - Integration testing examples
const express = require('express');
const sqlite3 = require('sqlite3').verbose();
const request = require('supertest');

console.log('🔗 Learning Integration Testing!');

// Test database setup
class TestDatabase {
  constructor() {
    this.db = new sqlite3.Database(':memory:');
    this.initializeDatabase();
  }
  
  initializeDatabase() {
    console.log('🔧 Setting up test database...');
    
    this.db.serialize(() => {
      this.db.run(`
        CREATE TABLE users (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          name TEXT NOT NULL,
          email TEXT UNIQUE NOT NULL,
          created_at DATETIME DEFAULT CURRENT_TIMESTAMP
        )
      `);
      
      this.db.run(`
        CREATE TABLE posts (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          title TEXT NOT NULL,
          content TEXT NOT NULL,
          user_id INTEGER,
          created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
          FOREIGN KEY (user_id) REFERENCES users (id)
        )
      `);
    });
  }
  
  async addUser(name, email) {
    return new Promise((resolve, reject) => {
      this.db.run(
        'INSERT INTO users (name, email) VALUES (?, ?)',
        [name, email],
        function(err) {
          if (err) reject(err);
          else resolve({ id: this.lastID, name, email });
        }
      );
    });
  }
  
  async getUser(id) {
    return new Promise((resolve, reject) => {
      this.db.get(
        'SELECT * FROM users WHERE id = ?',
        [id],
        (err, row) => {
          if (err) reject(err);
          else resolve(row);
        }
      );
    });
  }
  
  async getAllUsers() {
    return new Promise((resolve, reject) => {
      this.db.all('SELECT * FROM users', (err, rows) => {
        if (err) reject(err);
        else resolve(rows);
      });
    });
  }
  
  async addPost(title, content, userId) {
    return new Promise((resolve, reject) => {
      this.db.run(
        'INSERT INTO posts (title, content, user_id) VALUES (?, ?, ?)',
        [title, content, userId],
        function(err) {
          if (err) reject(err);
          else resolve({ id: this.lastID, title, content, user_id: userId });
        }
      );
    });
  }
  
  async getPostsByUser(userId) {
    return new Promise((resolve, reject) => {
      this.db.all(
        'SELECT * FROM posts WHERE user_id = ?',
        [userId],
        (err, rows) => {
          if (err) reject(err);
          else resolve(rows);
        }
      );
    });
  }
  
  close() {
    this.db.close();
  }
}

// Create Express app with database
function createApp(database) {
  const app = express();
  app.use(express.json());
  
  // Routes
  app.post('/api/users', async (req, res) => {
    try {
      const { name, email } = req.body;
      
      if (!name || !email) {
        return res.status(400).json({
          success: false,
          message: 'Name and email are required'
        });
      }
      
      const user = await database.addUser(name, email);
      
      res.status(201).json({
        success: true,
        data: user
      });
    } catch (error) {
      res.status(500).json({
        success: false,
        message: error.message
      });
    }
  });
  
  app.get('/api/users', async (req, res) => {
    try {
      const users = await database.getAllUsers();
      
      res.json({
        success: true,
        data: users
      });
    } catch (error) {
      res.status(500).json({
        success: false,
        message: error.message
      });
    }
  });
  
  app.get('/api/users/:id', async (req, res) => {
    try {
      const user = await database.getUser(req.params.id);
      
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
    } catch (error) {
      res.status(500).json({
        success: false,
        message: error.message
      });
    }
  });
  
  app.post('/api/posts', async (req, res) => {
    try {
      const { title, content, userId } = req.body;
      
      if (!title || !content || !userId) {
        return res.status(400).json({
          success: false,
          message: 'Title, content, and userId are required'
        });
      }
      
      // Check if user exists
      const user = await database.getUser(userId);
      if (!user) {
        return res.status(404).json({
          success: false,
          message: 'User not found'
        });
      }
      
      const post = await database.addPost(title, content, userId);
      
      res.status(201).json({
        success: true,
        data: post
      });
    } catch (error) {
      res.status(500).json({
        success: false,
        message: error.message
      });
    }
  });
  
  app.get('/api/users/:id/posts', async (req, res) => {
    try {
      const posts = await database.getPostsByUser(req.params.id);
      
      res.json({
        success: true,
        data: posts
      });
    } catch (error) {
      res.status(500).json({
        success: false,
        message: error.message
      });
    }
  });
  
  return app;
}

// Integration test functions
async function runIntegrationTests() {
  console.log('🧪 Running Integration Tests...');
  
  const database = new TestDatabase();
  const app = createApp(database);
  
  try {
    // Test 1: Create user and verify database storage
    console.log('\n1. Testing user creation integration...');
    const createUserResponse = await request(app)
      .post('/api/users')
      .send({ name: 'Alice', email: 'alice@example.com' })
      .expect(201);
    
    const userId = createUserResponse.body.data.id;
    console.log('✅ User created:', createUserResponse.body.data);
    
    // Verify user was actually stored in database
    const storedUser = await database.getUser(userId);
    console.log('✅ User stored in database:', storedUser);
    
    // Test 2: Create post and verify relationship
    console.log('\n2. Testing post creation with user relationship...');
    const createPostResponse = await request(app)
      .post('/api/posts')
      .send({ 
        title: 'My First Post', 
        content: 'This is my first post!',
        userId: userId 
      })
      .expect(201);
    
    console.log('✅ Post created:', createPostResponse.body.data);
    
    // Verify relationship works
    const userPostsResponse = await request(app)
      .get(`/api/users/${userId}/posts`)
      .expect(200);
    
    console.log('✅ User posts retrieved:', userPostsResponse.body.data);
    
    // Test 3: Error handling integration
    console.log('\n3. Testing error handling integration...');
    
    // Try to create post for non-existent user
    await request(app)
      .post('/api/posts')
      .send({
        title: 'Invalid Post',
        content: 'This should fail',
        userId: 999
      })
      .expect(404);
    
    console.log('✅ Error handling works correctly');
    
    // Test 4: Data validation integration
    console.log('\n4. Testing data validation integration...');
    
    // Try to create user with missing email
    await request(app)
      .post('/api/users')
      .send({ name: 'Invalid User' })
      .expect(400);
    
    console.log('✅ Data validation works correctly');
    
    console.log('\n🎉 All integration tests passed!');
    
  } catch (error) {
    console.log('❌ Integration test failed:', error.message);
  } finally {
    database.close();
  }
}

// Run the integration tests
runIntegrationTests();
```

---

## 🚀 Performance Testing and Optimization

### Memory Management and Monitoring

Create `performance-monitoring.js`:
```javascript
// performance-monitoring.js - Performance monitoring and optimization
const express = require('express');
const cluster = require('cluster');
const os = require('os');

console.log('🚀 Learning Performance Monitoring!');

// Performance monitoring utilities
class PerformanceMonitor {
  constructor() {
    this.startTime = Date.now();
    this.requestCount = 0;
    this.errorCount = 0;
    this.responseTimeSum = 0;
  }
  
  // Memory usage monitoring
  getMemoryUsage() {
    const usage = process.memoryUsage();
    
    return {
      rss: this.formatBytes(usage.rss), // Resident Set Size
      heapTotal: this.formatBytes(usage.heapTotal),
      heapUsed: this.formatBytes(usage.heapUsed),
      external: this.formatBytes(usage.external),
      arrayBuffers: this.formatBytes(usage.arrayBuffers)
    };
  }
  
  // CPU usage monitoring
  getCPUUsage() {
    const cpus = os.cpus();
    const load = os.loadavg();
    
    return {
      cores: cpus.length,
      model: cpus[0].model,
      loadAverage: {
        '1min': load[0].toFixed(2),
        '5min': load[1].toFixed(2),
        '15min': load[2].toFixed(2)
      }
    };
  }
  
  // Performance metrics
  getMetrics() {
    const uptime = Date.now() - this.startTime;
    const avgResponseTime = this.requestCount > 0 ? 
      (this.responseTimeSum / this.requestCount).toFixed(2) : 0;
    
    return {
      uptime: this.formatTime(uptime),
      requestCount: this.requestCount,
      errorCount: this.errorCount,
      errorRate: this.requestCount > 0 ? 
        ((this.errorCount / this.requestCount) * 100).toFixed(2) + '%' : '0%',
      averageResponseTime: avgResponseTime + 'ms',
      memory: this.getMemoryUsage(),
      cpu: this.getCPUUsage()
    };
  }
  
  // Middleware to track request performance
  middleware() {
    return (req, res, next) => {
      const startTime = Date.now();
      
      // Track request
      this.requestCount++;
      
      // Track response
      const originalSend = res.send;
      res.send = function(data) {
        const endTime = Date.now();
        const responseTime = endTime - startTime;
        
        // Add to response time sum
        this.responseTimeSum += responseTime;
        
        // Track errors
        if (res.statusCode >= 400) {
          this.errorCount++;
        }
        
        console.log(`📊 ${req.method} ${req.path} - ${res.statusCode} - ${responseTime}ms`);
        
        return originalSend.call(this, data);
      }.bind(this);
      
      next();
    };
  }
  
  // Utility functions
  formatBytes(bytes) {
    if (bytes === 0) return '0 Bytes';
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  }
  
  formatTime(ms) {
    const seconds = Math.floor(ms / 1000);
    const minutes = Math.floor(seconds / 60);
    const hours = Math.floor(minutes / 60);
    
    if (hours > 0) return `${hours}h ${minutes % 60}m ${seconds % 60}s`;
    if (minutes > 0) return `${minutes}m ${seconds % 60}s`;
    return `${seconds}s`;
  }
}

// Create Express app with performance monitoring
function createMonitoredApp() {
  const app = express();
  const monitor = new PerformanceMonitor();
  
  app.use(express.json());
  app.use(monitor.middleware());
  
  // Sample data for testing
  const users = [];
  for (let i = 1; i <= 1000; i++) {
    users.push({
      id: i,
      name: `User ${i}`,
      email: `user${i}@example.com`,
      data: 'x'.repeat(1000) // Add some data to test memory
    });
  }
  
  // Routes
  app.get('/api/users', (req, res) => {
    // Simulate some processing time
    setTimeout(() => {
      res.json({
        success: true,
        data: users,
        count: users.length
      });
    }, Math.random() * 100); // Random delay 0-100ms
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
  
  // Memory-intensive route (for testing)
  app.get('/api/memory-test', (req, res) => {
    console.log('🧠 Running memory test...');
    
    // Create large array (careful - this uses memory!)
    const largeArray = new Array(100000).fill(0).map((_, i) => ({
      id: i,
      data: 'x'.repeat(100)
    }));
    
    res.json({
      success: true,
      message: 'Memory test completed',
      arraySize: largeArray.length,
      memory: monitor.getMemoryUsage()
    });
  });
  
  // CPU-intensive route (for testing)
  app.get('/api/cpu-test', (req, res) => {
    console.log('⚡ Running CPU test...');
    
    const startTime = Date.now();
    
    // CPU-intensive task
    let result = 0;
    for (let i = 0; i < 1000000; i++) {
      result += Math.sqrt(i);
    }
    
    const endTime = Date.now();
    
    res.json({
      success: true,
      message: 'CPU test completed',
      result: result,
      executionTime: `${endTime - startTime}ms`,
      cpu: monitor.getCPUUsage()
    });
  });
  
  // Performance metrics endpoint
  app.get('/api/metrics', (req, res) => {
    res.json({
      success: true,
      metrics: monitor.getMetrics()
    });
  });
  
  // Error simulation route
  app.get('/api/error-test', (req, res) => {
    console.log('❌ Simulating error...');
    res.status(500).json({
      success: false,
      message: 'Simulated error for testing'
    });
  });
  
  return app;
}

// Load testing function
async function runLoadTest() {
  console.log('🏋️‍♂️ Running Load Test...');
  
  const app = createMonitoredApp();
  const PORT = 5003;
  
  const server = app.listen(PORT, () => {
    console.log(`🚀 Performance test server running on http://localhost:${PORT}`);
  });
  
  // Simulate load
  const testDuration = 10000; // 10 seconds
  const requestsPerSecond = 10;
  const interval = 1000 / requestsPerSecond;
  
  console.log(`⚡ Generating ${requestsPerSecond} requests per second for ${testDuration/1000} seconds...`);
  
  let requestCount = 0;
  const startTime = Date.now();
  
  const loadInterval = setInterval(async () => {
    // Make concurrent requests
    const promises = [];
    for (let i = 0; i < requestsPerSecond; i++) {
      promises.push(
        fetch(`http://localhost:${PORT}/api/users`)
          .then(response => response.json())
          .catch(error => ({ error: error.message }))
      );
    }
    
    try {
      await Promise.all(promises);
      requestCount += requestsPerSecond;
      
      if (requestCount % 50 === 0) {
        console.log(`📊 Sent ${requestCount} requests...`);
      }
    } catch (error) {
      console.log('❌ Load test error:', error.message);
    }
    
    // Check if test duration is complete
    if (Date.now() - startTime >= testDuration) {
      clearInterval(loadInterval);
      
      // Get final metrics
      setTimeout(async () => {
        try {
          const response = await fetch(`http://localhost:${PORT}/api/metrics`);
          const metrics = await response.json();
          
          console.log('\n📊 LOAD TEST RESULTS:');
          console.log('Total requests sent:', requestCount);
          console.log('Server metrics:', metrics.metrics);
          
          server.close();
        } catch (error) {
          console.log('❌ Error getting metrics:', error.message);
          server.close();
        }
      }, 2000);
    }
  }, interval);
}

// Run load test
runLoadTest();
```

### Clustering for Performance

Create `clustering-example.js`:
```javascript
// clustering-example.js - Node.js clustering for performance
const cluster = require('cluster');
const os = require('os');
const express = require('express');

console.log('🏭 Learning Node.js Clustering!');

if (cluster.isMaster) {
  // Master process
  console.log('👑 Master process started');
  console.log(`💻 CPU cores available: ${os.cpus().length}`);
  
  // Fork workers
  const numWorkers = os.cpus().length;
  console.log(`🔄 Starting ${numWorkers} worker processes...`);
  
  for (let i = 0; i < numWorkers; i++) {
    const worker = cluster.fork();
    console.log(`👷 Worker ${worker.process.pid} started`);
  }
  
  // Handle worker exit
  cluster.on('exit', (worker, code, signal) => {
    console.log(`💀 Worker ${worker.process.pid} died`);
    console.log('🔄 Starting a new worker...');
    cluster.fork();
  });
  
  // Monitor cluster health
  setInterval(() => {
    const workers = Object.keys(cluster.workers);
    console.log(`📊 Active workers: ${workers.length}`);
  }, 10000);
  
} else {
  // Worker process
  const app = express();
  const PORT = 5004;
  
  app.use(express.json());
  
  // Sample data
  const users = [];
  for (let i = 1; i <= 10000; i++) {
    users.push({
      id: i,
      name: `User ${i}`,
      email: `user${i}@example.com`
    });
  }
  
  // Routes
  app.get('/api/users', (req, res) => {
    console.log(`👷 Worker ${process.pid} handling request`);
    
    // Simulate some processing
    setTimeout(() => {
      res.json({
        success: true,
        data: users.slice(0, 100), // Return first 100 users
        total: users.length,
        worker: process.pid
      });
    }, Math.random() * 100);
  });
  
  app.get('/api/heavy-task', (req, res) => {
    console.log(`⚡ Worker ${process.pid} handling heavy task`);
    
    // CPU-intensive task
    const startTime = Date.now();
    let result = 0;
    
    for (let i = 0; i < 1000000; i++) {
      result += Math.sqrt(i);
    }
    
    const endTime = Date.now();
    
    res.json({
      success: true,
      message: 'Heavy task completed',
      executionTime: `${endTime - startTime}ms`,
      result: result,
      worker: process.pid
    });
  });
  
  app.get('/api/worker-info', (req, res) => {
    res.json({
      success: true,
      worker: {
        pid: process.pid,
        memory: process.memoryUsage(),
        uptime: process.uptime(),
        cpu: process.cpuUsage()
      }
    });
  });
  
  app.listen(PORT, () => {
    console.log(`👷 Worker ${process.pid} listening on port ${PORT}`);
  });
}
```

---

## 🐳 Docker Containerization

### Basic Dockerfile

Create `Dockerfile`:
```dockerfile
# Dockerfile - Containerizing Node.js applications
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Change ownership of app directory
RUN chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Start the application
CMD ["node", "server.js"]
```

### Docker Compose Configuration

Create `docker-compose.yml`:
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - PORT=3000
    depends_on:
      - redis
      - postgres
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped
    
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped
    
  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped
    
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app
    restart: unless-stopped

volumes:
  redis_data:
  postgres_data:
```

### Dockerized Application Example

Create `dockerized-app.js`:
```javascript
// dockerized-app.js - Production-ready Node.js application
const express = require('express');
const helmet = require('helmet');
const compression = require('compression');
const cors = require('cors');

console.log('🐳 Starting Dockerized Node.js Application!');

const app = express();
const PORT = process.env.PORT || 3000;
const NODE_ENV = process.env.NODE_ENV || 'development';

// Security middleware
app.use(helmet());
app.use(compression());
app.use(cors());
app.use(express.json({ limit: '10mb' }));

// Request logging
app.use((req, res, next) => {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.path}`);
  next();
});

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({
    success: true,
    status: 'healthy',
    timestamp: new Date().toISOString(),
    environment: NODE_ENV,
    uptime: process.uptime(),
    memory: process.memoryUsage()
  });
});

// API routes
app.get('/api/info', (req, res) => {
  res.json({
    success: true,
    message: 'Dockerized Node.js API is running! 🐳',
    environment: NODE_ENV,
    version: process.version,
    platform: process.platform,
    architecture: process.arch
  });
});

app.get('/api/users', (req, res) => {
  // Simulate user data
  const users = [
    { id: 1, name: 'Alice', email: 'alice@example.com' },
    { id: 2, name: 'Bob', email: 'bob@example.com' },
    { id: 3, name: 'Charlie', email: 'charlie@example.com' }
  ];
  
  res.json({
    success: true,
    data: users,
    container: {
      hostname: require('os').hostname(),
      pid: process.pid
    }
  });
});

// Error handling middleware
app.use((error, req, res, next) => {
  console.error('❌ Error:', error.message);
  
  res.status(500).json({
    success: false,
    message: NODE_ENV === 'production' ? 'Internal Server Error' : error.message
  });
});

// 404 handler
app.use('*', (req, res) => {
  res.status(404).json({
    success: false,
    message: 'Route not found'
  });
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('🔄 SIGTERM received, shutting down gracefully...');
  server.close(() => {
    console.log('✅ Process terminated');
    process.exit(0);
  });
});

process.on('SIGINT', () => {
  console.log('🔄 SIGINT received, shutting down gracefully...');
  server.close(() => {
    console.log('✅ Process terminated');
    process.exit(0);
  });
});

// Start server
const server = app.listen(PORT, () => {
  console.log(`🚀 Server running on port ${PORT}`);
  console.log(`🌍 Environment: ${NODE_ENV}`);
  console.log(`📊 Health check: http://localhost:${PORT}/health`);
});

// Demo Docker commands
console.log(`
🐳 DOCKER COMMANDS:

1. Build image:
   docker build -t my-node-app .

2. Run container:
   docker run -p 3000:3000 my-node-app

3. Run with environment variables:
   docker run -p 3000:3000 -e NODE_ENV=production my-node-app

4. Run with Docker Compose:
   docker-compose up -d

5. View logs:
   docker logs container-name

6. Stop container:
   docker stop container-name

7. Remove container:
   docker rm container-name

8. Remove image:
   docker rmi my-node-app
`);
```

---

## 📋 Environment Configuration

### Environment Management

Create `env-config.js`:
```javascript
// env-config.js - Environment configuration management
const path = require('path');

console.log('⚙️ Learning Environment Configuration!');

// Environment configuration class
class EnvConfig {
  constructor() {
    this.env = process.env.NODE_ENV || 'development';
    this.loadConfig();
  }
  
  loadConfig() {
    console.log(`🔧 Loading configuration for: ${this.env}`);
    
    // Base configuration
    this.config = {
      // Server settings
      port: process.env.PORT || 3000,
      host: process.env.HOST || 'localhost',
      
      // Database settings
      database: {
        host: process.env.DB_HOST || 'localhost',
        port: process.env.DB_PORT || 5432,
        name: process.env.DB_NAME || 'myapp',
        user: process.env.DB_USER || 'admin',
        password: process.env.DB_PASSWORD || 'password',
        ssl: process.env.DB_SSL === 'true'
      },
      
      // Redis settings
      redis: {
        host: process.env.REDIS_HOST || 'localhost',
        port: process.env.REDIS_PORT || 6379,
        password: process.env.REDIS_PASSWORD || null
      },
      
      // Security settings
      security: {
        jwtSecret: process.env.JWT_SECRET || 'default-secret-change-me',
        sessionSecret: process.env.SESSION_SECRET || 'default-session-secret',
        saltRounds: parseInt(process.env.SALT_ROUNDS) || 10
      },
      
      // Logging settings
      logging: {
        level: process.env.LOG_LEVEL || 'info',
        file: process.env.LOG_FILE || 'app.log'
      },
      
      // API settings
      api: {
        rateLimit: {
          windowMs: parseInt(process.env.RATE_LIMIT_WINDOW) || 900000, // 15 minutes
          max: parseInt(process.env.RATE_LIMIT_MAX) || 100
        },
        corsOrigins: process.env.CORS_ORIGINS ? 
          process.env.CORS_ORIGINS.split(',') : ['http://localhost:3000']
      }
    };
    
    // Environment-specific overrides
    this.applyEnvironmentOverrides();
  }
  
  applyEnvironmentOverrides() {
    switch (this.env) {
      case 'development':
        this.config.logging.level = 'debug';
        this.config.api.corsOrigins = ['*'];
        break;
        
      case 'test':
        this.config.port = 0; // Random port for testing
        this.config.database.name = 'myapp_test';
        this.config.logging.level = 'error';
        break;
        
      case 'production':
        this.config.logging.level = 'warn';
        this.config.database.ssl = true;
        // Remove default secrets in production
        if (this.config.security.jwtSecret === 'default-secret-change-me') {
          throw new Error('JWT_SECRET must be set in production!');
        }
        break;
    }
  }
  
  get(key) {
    return this.getNestedValue(this.config, key);
  }
  
  getNestedValue(obj, path) {
    return path.split('.').reduce((current, key) => {
      return current && current[key] !== undefined ? current[key] : undefined;
    }, obj);
  }
  
  validate() {
    const errors = [];
    
    // Validate required settings
    const required = [
      'port',
      'database.host',
      'database.name',
      'security.jwtSecret'
    ];
    
    required.forEach(key => {
      if (!this.get(key)) {
        errors.push(`Missing required configuration: ${key}`);
      }
    });
    
    // Validate production settings
    if (this.env === 'production') {
      if (this.get('security.jwtSecret') === 'default-secret-change-me') {
        errors.push('JWT_SECRET must be changed in production');
      }
      
      if (this.get('security.sessionSecret') === 'default-session-secret') {
        errors.push('SESSION_SECRET must be changed in production');
      }
    }
    
    return errors;
  }
  
  printConfig() {
    console.log('📋 Current Configuration:');
    console.log('Environment:', this.env);
    console.log('Port:', this.config.port);
    console.log('Database:', this.config.database.host + ':' + this.config.database.port);
    console.log('Redis:', this.config.redis.host + ':' + this.config.redis.port);
    console.log('Log Level:', this.config.logging.level);
    console.log('CORS Origins:', this.config.api.corsOrigins);
  }
}

// Create configuration instance
const config = new EnvConfig();

// Validate configuration
const errors = config.validate();
if (errors.length > 0) {
  console.log('❌ Configuration errors:');
  errors.forEach(error => console.log('  -', error));
  process.exit(1);
}

// Print configuration
config.printConfig();

// Example usage
console.log(`
📝 CONFIGURATION EXAMPLES:

1. Access nested values:
   config.get('database.host') // Returns database host
   config.get('api.rateLimit.max') // Returns rate limit max

2. Environment variables:
   NODE_ENV=production npm start
   PORT=8080 npm start
   DB_HOST=localhost DB_PORT=5432 npm start

3. .env file example:
   NODE_ENV=development
   PORT=3000
   DB_HOST=localhost
   DB_PORT=5432
   DB_NAME=myapp
   JWT_SECRET=your-secret-key-here
   REDIS_HOST=localhost
   REDIS_PORT=6379
   LOG_LEVEL=debug
   CORS_ORIGINS=http://localhost:3000,http://localhost:3001

4. Production checklist:
   ✅ Set JWT_SECRET
   ✅ Set SESSION_SECRET
   ✅ Configure database connection
   ✅ Set appropriate log level
   ✅ Configure CORS origins
   ✅ Enable SSL for database
`);

module.exports = config;
```

---

## 🎉 Summary

Today you learned:
- ✅ Advanced testing strategies (integration, load, stress)
- ✅ Performance monitoring and optimization
- ✅ Memory management and CPU monitoring
- ✅ Node.js clustering for scalability
- ✅ Docker containerization
- ✅ Environment configuration management
- ✅ Production deployment best practices
- ✅ Health monitoring and logging

**Next Up**: Day 05 - Part 03: Real-Time Applications and WebSockets

---

## 🎯 Practice Exercises

### Exercise 1: Performance Testing Suite
Create a comprehensive testing suite with:
1. Unit tests for all functions
2. Integration tests for database operations
3. Load tests for API endpoints
4. Memory and CPU monitoring
5. Performance benchmarks

### Exercise 2: Production Deployment
Set up a production environment with:
1. Docker containerization
2. Environment configuration
3. Health monitoring
4. Load balancing
5. Logging and alerting

### Exercise 3: Scalable Architecture
Build a scalable application with:
1. Clustering implementation
2. Database connection pooling
3. Redis caching
4. Performance monitoring
5. Graceful shutdown handling

---

*Remember: Performance optimization is like tuning a race car - every little improvement adds up to make a big difference! 🏎️*
