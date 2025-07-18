# Day 06 - Part 02: API Gateways and Service Discovery

## 🎯 Learning Objectives
By the end of this lesson, you will understand:
- What API Gateways are and why they're essential
- How to build a simple API Gateway
- Service discovery patterns and implementation
- Load balancing and routing strategies
- Authentication and authorization at the gateway level
- Monitoring and logging in distributed systems

---

## 🚪 What is an API Gateway?

### Simple Explanation (for 10-year-olds!)

Imagine your house has **many different rooms** (microservices):

#### **Without API Gateway - Chaos!**
```
🏠 YOUR HOUSE (Microservices)
┌─────────────────────────────────────────────────────┐
│  🚪 Front Door → Kitchen (direct)                   │
│  🚪 Back Door → Living Room (direct)                │
│  🚪 Side Door → Bedroom (direct)                    │
│  🚪 Window → Bathroom (direct)                      │
│  Everyone enters from different places! 😵‍💫         │
└─────────────────────────────────────────────────────┘

Problems:
❌ Visitors get confused about which door to use
❌ No security - anyone can enter anywhere
❌ No way to know who's visiting
❌ Each room needs its own doorbell
```

#### **With API Gateway - Organized!**
```
🏠 YOUR HOUSE (Microservices)
┌─────────────────────────────────────────────────────┐
│           🚪 MAIN ENTRANCE (API Gateway)            │
│                      ↓                             │
│  🏠 Receptionist checks who you are                 │
│  📋 Gives you directions to the right room          │
│  🔐 Makes sure you're allowed to go there          │
│                      ↓                             │
│  🍳 Kitchen   🛋️ Living   🛏️ Bedroom   🚿 Bathroom  │
└─────────────────────────────────────────────────────┘

Benefits:
✅ One entrance for everyone (simple!)
✅ Security guard checks everyone
✅ Visitors get proper directions
✅ Know who's visiting when
```

### Real-World Analogy
```
API Gateway is like a HOTEL RECEPTION:
┌─────────────────────────────────────────────────────┐
│  🏨 Hotel Reception Desk                            │
│  👨‍💼 "Welcome! How can I help you?"                   │
│  📝 Checks your booking (authentication)            │
│  🗝️ Gives you room key (authorization)              │
│  🗺️ Shows you directions to your room               │
│  📞 Handles all guest requests                      │
└─────────────────────────────────────────────────────┘
```

---

## 🛠️ Building an API Gateway

### Simple API Gateway Implementation

Create `api-gateway.js`:
```javascript
// api-gateway.js - Simple API Gateway implementation
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');
const jwt = require('jsonwebtoken');
const axios = require('axios');

console.log('🚪 Building API Gateway!');

const app = express();
app.use(express.json());

// Gateway configuration
const GATEWAY_CONFIG = {
  name: 'api-gateway',
  version: '1.0.0',
  port: 3000
};

// Service registry (in production, this would be dynamic)
const SERVICE_REGISTRY = {
  'user-service': {
    url: 'http://localhost:3001',
    healthCheck: '/health',
    routes: ['/users', '/register', '/login', '/profile', '/logout']
  },
  'product-service': {
    url: 'http://localhost:3002',
    healthCheck: '/health',
    routes: ['/products', '/categories', '/search']
  },
  'order-service': {
    url: 'http://localhost:3003',
    healthCheck: '/health',
    routes: ['/orders']
  }
};

// Rate limiting storage
const rateLimitStore = new Map();

// Request logging middleware
const requestLogger = (req, res, next) => {
  const startTime = Date.now();
  
  console.log(`📝 ${new Date().toISOString()} - ${req.method} ${req.path} from ${req.ip}`);
  
  // Log response time
  res.on('finish', () => {
    const duration = Date.now() - startTime;
    console.log(`⏱️ ${req.method} ${req.path} - ${res.statusCode} (${duration}ms)`);
  });
  
  next();
};

// Rate limiting middleware
const rateLimit = (limit = 100, windowMs = 15 * 60 * 1000) => {
  return (req, res, next) => {
    const key = req.ip;
    const now = Date.now();
    
    if (!rateLimitStore.has(key)) {
      rateLimitStore.set(key, {
        count: 1,
        resetTime: now + windowMs
      });
      return next();
    }
    
    const rateLimitData = rateLimitStore.get(key);
    
    if (now > rateLimitData.resetTime) {
      // Reset window
      rateLimitStore.set(key, {
        count: 1,
        resetTime: now + windowMs
      });
      return next();
    }
    
    if (rateLimitData.count >= limit) {
      return res.status(429).json({
        success: false,
        message: 'Too many requests! Please try again later.',
        retryAfter: Math.ceil((rateLimitData.resetTime - now) / 1000)
      });
    }
    
    rateLimitData.count++;
    rateLimitStore.set(key, rateLimitData);
    
    // Add rate limit headers
    res.set({
      'X-RateLimit-Limit': limit,
      'X-RateLimit-Remaining': limit - rateLimitData.count,
      'X-RateLimit-Reset': new Date(rateLimitData.resetTime).toISOString()
    });
    
    next();
  };
};

// Authentication middleware
const authenticate = (req, res, next) => {
  const authHeader = req.headers.authorization;
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({
      success: false,
      message: 'Access token required!'
    });
  }
  
  try {
    const decoded = jwt.verify(token, 'your-secret-key');
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(403).json({
      success: false,
      message: 'Invalid or expired token!'
    });
  }
};

// Service health checker
class ServiceHealthChecker {
  constructor() {
    this.serviceStatus = new Map();
    this.checkInterval = 30000; // 30 seconds
    this.startHealthChecks();
  }
  
  async checkServiceHealth(serviceName, serviceConfig) {
    try {
      const response = await axios.get(`${serviceConfig.url}${serviceConfig.healthCheck}`, {
        timeout: 5000
      });
      
      this.serviceStatus.set(serviceName, {
        healthy: true,
        lastCheck: new Date().toISOString(),
        responseTime: response.headers['response-time'] || 'N/A',
        status: response.status
      });
      
      return true;
    } catch (error) {
      this.serviceStatus.set(serviceName, {
        healthy: false,
        lastCheck: new Date().toISOString(),
        error: error.message,
        status: error.response?.status || 'N/A'
      });
      
      console.log(`❌ Service ${serviceName} is unhealthy: ${error.message}`);
      return false;
    }
  }
  
  startHealthChecks() {
    console.log('🔍 Starting service health checks...');
    
    // Initial check
    this.checkAllServices();
    
    // Regular checks
    setInterval(() => {
      this.checkAllServices();
    }, this.checkInterval);
  }
  
  async checkAllServices() {
    for (const [serviceName, serviceConfig] of Object.entries(SERVICE_REGISTRY)) {
      await this.checkServiceHealth(serviceName, serviceConfig);
    }
  }
  
  getServiceStatus(serviceName) {
    return this.serviceStatus.get(serviceName);
  }
  
  getAllServiceStatus() {
    return Object.fromEntries(this.serviceStatus);
  }
}

// Initialize health checker
const healthChecker = new ServiceHealthChecker();

// Apply global middleware
app.use(requestLogger);
app.use(rateLimit(100, 15 * 60 * 1000)); // 100 requests per 15 minutes

// Gateway health check
app.get('/health', (req, res) => {
  const serviceStatuses = healthChecker.getAllServiceStatus();
  const healthyServices = Object.values(serviceStatuses).filter(s => s.healthy).length;
  const totalServices = Object.keys(SERVICE_REGISTRY).length;
  
  res.json({
    gateway: {
      name: GATEWAY_CONFIG.name,
      version: GATEWAY_CONFIG.version,
      status: 'healthy',
      timestamp: new Date().toISOString(),
      uptime: process.uptime()
    },
    services: serviceStatuses,
    summary: {
      healthy: healthyServices,
      total: totalServices,
      healthPercentage: ((healthyServices / totalServices) * 100).toFixed(2)
    }
  });
});

// Gateway info endpoint
app.get('/info', (req, res) => {
  res.json({
    gateway: GATEWAY_CONFIG,
    services: Object.keys(SERVICE_REGISTRY),
    routes: {
      'GET /health': 'Gateway health check',
      'GET /info': 'Gateway information',
      'GET /services': 'List all services',
      'POST /auth/*': 'Authentication routes',
      'GET /api/*': 'Proxied service routes'
    },
    features: [
      'Request logging',
      'Rate limiting',
      'Authentication',
      'Service discovery',
      'Load balancing',
      'Health monitoring'
    ]
  });
});

// List all services
app.get('/services', (req, res) => {
  const services = Object.entries(SERVICE_REGISTRY).map(([name, config]) => ({
    name,
    url: config.url,
    routes: config.routes,
    status: healthChecker.getServiceStatus(name)
  }));
  
  res.json({
    success: true,
    services,
    count: services.length
  });
});

// Authentication routes (public)
app.use('/auth', createProxyMiddleware({
  target: SERVICE_REGISTRY['user-service'].url,
  changeOrigin: true,
  pathRewrite: {
    '^/auth': ''
  },
  onProxyReq: (proxyReq, req, res) => {
    console.log(`🔐 Proxying auth request: ${req.method} ${req.path}`);
  }
}));

// Protected API routes
app.use('/api/users', authenticate, createProxyMiddleware({
  target: SERVICE_REGISTRY['user-service'].url,
  changeOrigin: true,
  pathRewrite: {
    '^/api/users': ''
  },
  onProxyReq: (proxyReq, req, res) => {
    console.log(`👤 Proxying user request: ${req.method} ${req.path}`);
  }
}));

app.use('/api/products', createProxyMiddleware({
  target: SERVICE_REGISTRY['product-service'].url,
  changeOrigin: true,
  pathRewrite: {
    '^/api/products': ''
  },
  onProxyReq: (proxyReq, req, res) => {
    console.log(`📦 Proxying product request: ${req.method} ${req.path}`);
  }
}));

app.use('/api/orders', authenticate, createProxyMiddleware({
  target: SERVICE_REGISTRY['order-service'].url,
  changeOrigin: true,
  pathRewrite: {
    '^/api/orders': ''
  },
  onProxyReq: (proxyReq, req, res) => {
    console.log(`🛍️ Proxying order request: ${req.method} ${req.path}`);
  }
}));

// Service discovery endpoint
app.get('/discover/:serviceName', (req, res) => {
  const { serviceName } = req.params;
  const service = SERVICE_REGISTRY[serviceName];
  
  if (!service) {
    return res.status(404).json({
      success: false,
      message: 'Service not found!'
    });
  }
  
  const status = healthChecker.getServiceStatus(serviceName);
  
  res.json({
    success: true,
    service: {
      name: serviceName,
      url: service.url,
      routes: service.routes,
      status: status || { healthy: false, error: 'Not checked yet' }
    }
  });
});

// Error handling middleware
app.use((error, req, res, next) => {
  console.error('❌ Gateway error:', error);
  
  res.status(500).json({
    success: false,
    message: 'Gateway error occurred',
    error: process.env.NODE_ENV === 'development' ? error.message : 'Internal server error'
  });
});

// Handle 404 for undefined routes
app.use('*', (req, res) => {
  res.status(404).json({
    success: false,
    message: 'Route not found!',
    availableRoutes: [
      'GET /health',
      'GET /info',
      'GET /services',
      'POST /auth/register',
      'POST /auth/login',
      'GET /api/products',
      'GET /api/orders (auth required)',
      'GET /api/users (auth required)'
    ]
  });
});

// Start gateway
const PORT = GATEWAY_CONFIG.port;
app.listen(PORT, () => {
  console.log(`🚪 API Gateway running on port ${PORT}`);
  console.log(`🔍 Health check: http://localhost:${PORT}/health`);
  console.log(`📊 Gateway info: http://localhost:${PORT}/info`);
  console.log(`🔍 Service discovery: http://localhost:${PORT}/services`);
  console.log('\n🎯 Available Routes:');
  console.log('   📝 POST /auth/register - Register new user');
  console.log('   🔐 POST /auth/login - User login');
  console.log('   📦 GET /api/products - Get products');
  console.log('   🛍️ GET /api/orders - Get orders (auth required)');
  console.log('   👤 GET /api/users - User operations (auth required)');
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('🔄 Shutting down API Gateway...');
  server.close(() => {
    console.log('✅ API Gateway closed');
    process.exit(0);
  });
});
```

---

## 🔍 Service Discovery

### Dynamic Service Discovery Implementation

Create `service-discovery.js`:
```javascript
// service-discovery.js - Service Discovery implementation
const express = require('express');
const axios = require('axios');

console.log('🔍 Building Service Discovery!');

const app = express();
app.use(express.json());

// Service registry storage
const serviceRegistry = new Map();

// Service discovery configuration
const DISCOVERY_CONFIG = {
  name: 'service-discovery',
  version: '1.0.0',
  port: 3004,
  healthCheckInterval: 30000, // 30 seconds
  serviceTimeout: 60000 // 60 seconds
};

// Service model
class ServiceInstance {
  constructor(serviceName, instanceId, host, port, metadata = {}) {
    this.serviceName = serviceName;
    this.instanceId = instanceId;
    this.host = host;
    this.port = port;
    this.url = `http://${host}:${port}`;
    this.metadata = metadata;
    this.registeredAt = new Date().toISOString();
    this.lastHeartbeat = new Date().toISOString();
    this.healthy = true;
    this.healthCheckPath = metadata.healthCheckPath || '/health';
  }
  
  updateHeartbeat() {
    this.lastHeartbeat = new Date().toISOString();
    this.healthy = true;
  }
  
  markUnhealthy() {
    this.healthy = false;
  }
  
  isExpired() {
    const now = Date.now();
    const lastHeartbeat = new Date(this.lastHeartbeat).getTime();
    return (now - lastHeartbeat) > DISCOVERY_CONFIG.serviceTimeout;
  }
}

// Service Discovery Manager
class ServiceDiscoveryManager {
  constructor() {
    this.services = new Map();
    this.startHealthChecks();
    this.startCleanupJob();
  }
  
  // Register a service instance
  register(serviceName, instanceId, host, port, metadata = {}) {
    const instance = new ServiceInstance(serviceName, instanceId, host, port, metadata);
    
    if (!this.services.has(serviceName)) {
      this.services.set(serviceName, new Map());
    }
    
    this.services.get(serviceName).set(instanceId, instance);
    
    console.log(`✅ Service registered: ${serviceName}/${instanceId} at ${host}:${port}`);
    return instance;
  }
  
  // Deregister a service instance
  deregister(serviceName, instanceId) {
    if (this.services.has(serviceName)) {
      const serviceInstances = this.services.get(serviceName);
      const removed = serviceInstances.delete(instanceId);
      
      if (serviceInstances.size === 0) {
        this.services.delete(serviceName);
      }
      
      if (removed) {
        console.log(`✅ Service deregistered: ${serviceName}/${instanceId}`);
      }
      
      return removed;
    }
    
    return false;
  }
  
  // Update service heartbeat
  heartbeat(serviceName, instanceId) {
    if (this.services.has(serviceName)) {
      const serviceInstances = this.services.get(serviceName);
      const instance = serviceInstances.get(instanceId);
      
      if (instance) {
        instance.updateHeartbeat();
        console.log(`💓 Heartbeat received: ${serviceName}/${instanceId}`);
        return true;
      }
    }
    
    return false;
  }
  
  // Discover service instances
  discover(serviceName, onlyHealthy = true) {
    if (!this.services.has(serviceName)) {
      return [];
    }
    
    const serviceInstances = this.services.get(serviceName);
    const instances = Array.from(serviceInstances.values());
    
    if (onlyHealthy) {
      return instances.filter(instance => instance.healthy && !instance.isExpired());
    }
    
    return instances;
  }
  
  // Get all services
  getAllServices() {
    const result = {};
    
    for (const [serviceName, instances] of this.services) {
      result[serviceName] = Array.from(instances.values());
    }
    
    return result;
  }
  
  // Load balancer - round robin
  getServiceInstance(serviceName, strategy = 'round-robin') {
    const instances = this.discover(serviceName, true);
    
    if (instances.length === 0) {
      return null;
    }
    
    switch (strategy) {
      case 'round-robin':
        // Simple round-robin (in production, use more sophisticated approach)
        const index = Date.now() % instances.length;
        return instances[index];
      
      case 'random':
        const randomIndex = Math.floor(Math.random() * instances.length);
        return instances[randomIndex];
      
      default:
        return instances[0];
    }
  }
  
  // Health check all services
  async startHealthChecks() {
    console.log('🔍 Starting health checks...');
    
    setInterval(async () => {
      for (const [serviceName, serviceInstances] of this.services) {
        for (const [instanceId, instance] of serviceInstances) {
          await this.checkInstanceHealth(instance);
        }
      }
    }, DISCOVERY_CONFIG.healthCheckInterval);
  }
  
  async checkInstanceHealth(instance) {
    try {
      const response = await axios.get(
        `${instance.url}${instance.healthCheckPath}`,
        { timeout: 5000 }
      );
      
      if (response.status === 200) {
        instance.updateHeartbeat();
      } else {
        instance.markUnhealthy();
      }
    } catch (error) {
      instance.markUnhealthy();
      console.log(`❌ Health check failed: ${instance.serviceName}/${instance.instanceId}`);
    }
  }
  
  // Cleanup expired services
  startCleanupJob() {
    setInterval(() => {
      for (const [serviceName, serviceInstances] of this.services) {
        for (const [instanceId, instance] of serviceInstances) {
          if (instance.isExpired()) {
            console.log(`🗑️ Removing expired service: ${serviceName}/${instanceId}`);
            this.deregister(serviceName, instanceId);
          }
        }
      }
    }, DISCOVERY_CONFIG.serviceTimeout);
  }
}

// Initialize service discovery manager
const discoveryManager = new ServiceDiscoveryManager();

// Health check endpoint
app.get('/health', (req, res) => {
  const totalServices = discoveryManager.services.size;
  let totalInstances = 0;
  let healthyInstances = 0;
  
  for (const [serviceName, instances] of discoveryManager.services) {
    totalInstances += instances.size;
    for (const [instanceId, instance] of instances) {
      if (instance.healthy && !instance.isExpired()) {
        healthyInstances++;
      }
    }
  }
  
  res.json({
    service: DISCOVERY_CONFIG.name,
    version: DISCOVERY_CONFIG.version,
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    registry: {
      totalServices,
      totalInstances,
      healthyInstances,
      healthPercentage: totalInstances > 0 ? ((healthyInstances / totalInstances) * 100).toFixed(2) : 0
    }
  });
});

// Register service endpoint
app.post('/register', (req, res) => {
  const { serviceName, instanceId, host, port, metadata } = req.body;
  
  if (!serviceName || !instanceId || !host || !port) {
    return res.status(400).json({
      success: false,
      message: 'serviceName, instanceId, host, and port are required!'
    });
  }
  
  const instance = discoveryManager.register(serviceName, instanceId, host, port, metadata);
  
  res.status(201).json({
    success: true,
    message: 'Service registered successfully!',
    instance: {
      serviceName: instance.serviceName,
      instanceId: instance.instanceId,
      url: instance.url,
      registeredAt: instance.registeredAt
    }
  });
});

// Deregister service endpoint
app.delete('/register/:serviceName/:instanceId', (req, res) => {
  const { serviceName, instanceId } = req.params;
  
  const removed = discoveryManager.deregister(serviceName, instanceId);
  
  if (removed) {
    res.json({
      success: true,
      message: 'Service deregistered successfully!'
    });
  } else {
    res.status(404).json({
      success: false,
      message: 'Service instance not found!'
    });
  }
});

// Heartbeat endpoint
app.post('/heartbeat/:serviceName/:instanceId', (req, res) => {
  const { serviceName, instanceId } = req.params;
  
  const updated = discoveryManager.heartbeat(serviceName, instanceId);
  
  if (updated) {
    res.json({
      success: true,
      message: 'Heartbeat updated successfully!'
    });
  } else {
    res.status(404).json({
      success: false,
      message: 'Service instance not found!'
    });
  }
});

// Discover service instances
app.get('/discover/:serviceName', (req, res) => {
  const { serviceName } = req.params;
  const { onlyHealthy = 'true' } = req.query;
  
  const instances = discoveryManager.discover(serviceName, onlyHealthy === 'true');
  
  res.json({
    success: true,
    serviceName,
    instances,
    count: instances.length
  });
});

// Get service instance (load balancing)
app.get('/instance/:serviceName', (req, res) => {
  const { serviceName } = req.params;
  const { strategy = 'round-robin' } = req.query;
  
  const instance = discoveryManager.getServiceInstance(serviceName, strategy);
  
  if (instance) {
    res.json({
      success: true,
      instance: {
        serviceName: instance.serviceName,
        instanceId: instance.instanceId,
        url: instance.url,
        healthy: instance.healthy,
        lastHeartbeat: instance.lastHeartbeat
      }
    });
  } else {
    res.status(404).json({
      success: false,
      message: 'No healthy instances found!'
    });
  }
});

// Get all services
app.get('/services', (req, res) => {
  const services = discoveryManager.getAllServices();
  
  res.json({
    success: true,
    services,
    serviceCount: Object.keys(services).length
  });
});

// Start service discovery
const PORT = DISCOVERY_CONFIG.port;
app.listen(PORT, () => {
  console.log(`🔍 Service Discovery running on port ${PORT}`);
  console.log(`🔍 Health check: http://localhost:${PORT}/health`);
  console.log('\n📋 Available Endpoints:');
  console.log('   POST /register - Register service');
  console.log('   DELETE /register/:service/:instance - Deregister');
  console.log('   POST /heartbeat/:service/:instance - Send heartbeat');
  console.log('   GET /discover/:service - Discover instances');
  console.log('   GET /instance/:service - Get load-balanced instance');
  console.log('   GET /services - List all services');
});

// Demo usage
console.log(`
🔍 SERVICE DISCOVERY DEMO:

1. Register a service:
   POST http://localhost:3004/register
   {
     "serviceName": "user-service",
     "instanceId": "user-1",
     "host": "localhost",
     "port": 3001,
     "metadata": {
       "version": "1.0.0",
       "healthCheckPath": "/health"
     }
   }

2. Discover service instances:
   GET http://localhost:3004/discover/user-service

3. Get load-balanced instance:
   GET http://localhost:3004/instance/user-service?strategy=round-robin

4. Send heartbeat:
   POST http://localhost:3004/heartbeat/user-service/user-1

5. View all services:
   GET http://localhost:3004/services
`);
```

---

## ⚡ Advanced Gateway Features

### Enhanced API Gateway with Circuit Breaker

Create `advanced-gateway.js`:
```javascript
// advanced-gateway.js - Advanced API Gateway with Circuit Breaker
const express = require('express');
const axios = require('axios');

console.log('⚡ Building Advanced API Gateway!');

const app = express();
app.use(express.json());

// Circuit breaker states
const CIRCUIT_STATES = {
  CLOSED: 'CLOSED',     // Normal operation
  OPEN: 'OPEN',         // Failing, reject requests
  HALF_OPEN: 'HALF_OPEN' // Testing if service recovered
};

// Circuit breaker class
class CircuitBreaker {
  constructor(serviceName, options = {}) {
    this.serviceName = serviceName;
    this.state = CIRCUIT_STATES.CLOSED;
    this.failureCount = 0;
    this.successCount = 0;
    this.lastFailureTime = null;
    
    // Configuration
    this.failureThreshold = options.failureThreshold || 5;
    this.timeout = options.timeout || 60000; // 60 seconds
    this.monitoringPeriod = options.monitoringPeriod || 10000; // 10 seconds
  }
  
  async call(fn) {
    if (this.state === CIRCUIT_STATES.OPEN) {
      if (this.shouldAttemptReset()) {
        this.state = CIRCUIT_STATES.HALF_OPEN;
        console.log(`🔄 Circuit breaker HALF_OPEN for ${this.serviceName}`);
      } else {
        throw new Error(`Circuit breaker OPEN for ${this.serviceName}`);
      }
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  onSuccess() {
    this.failureCount = 0;
    this.successCount++;
    
    if (this.state === CIRCUIT_STATES.HALF_OPEN) {
      this.state = CIRCUIT_STATES.CLOSED;
      console.log(`✅ Circuit breaker CLOSED for ${this.serviceName}`);
    }
  }
  
  onFailure() {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    
    if (this.failureCount >= this.failureThreshold) {
      this.state = CIRCUIT_STATES.OPEN;
      console.log(`❌ Circuit breaker OPEN for ${this.serviceName}`);
    }
  }
  
  shouldAttemptReset() {
    return (Date.now() - this.lastFailureTime) > this.timeout;
  }
  
  getStatus() {
    return {
      serviceName: this.serviceName,
      state: this.state,
      failureCount: this.failureCount,
      successCount: this.successCount,
      lastFailureTime: this.lastFailureTime
    };
  }
}

// Service registry with circuit breakers
const serviceRegistry = {
  'user-service': {
    url: 'http://localhost:3001',
    circuitBreaker: new CircuitBreaker('user-service')
  },
  'product-service': {
    url: 'http://localhost:3002',
    circuitBreaker: new CircuitBreaker('product-service')
  },
  'order-service': {
    url: 'http://localhost:3003',
    circuitBreaker: new CircuitBreaker('order-service')
  }
};

// Request retry logic
async function retryRequest(fn, maxRetries = 3, delay = 1000) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxRetries) {
        throw error;
      }
      
      console.log(`🔄 Retry attempt ${attempt} failed, retrying in ${delay}ms...`);
      await new Promise(resolve => setTimeout(resolve, delay));
      delay *= 2; // Exponential backoff
    }
  }
}

// Advanced proxy middleware
const createAdvancedProxy = (serviceName) => {
  return async (req, res, next) => {
    const service = serviceRegistry[serviceName];
    
    if (!service) {
      return res.status(404).json({
        success: false,
        message: `Service ${serviceName} not found`
      });
    }
    
    try {
      const result = await service.circuitBreaker.call(async () => {
        return await retryRequest(async () => {
          const response = await axios({
            method: req.method,
            url: `${service.url}${req.path}`,
            data: req.body,
            headers: {
              ...req.headers,
              'host': undefined, // Remove host header
              'x-forwarded-for': req.ip,
              'x-forwarded-proto': req.protocol
            },
            timeout: 5000
          });
          
          return response;
        });
      });
      
      // Forward response
      res.status(result.status).json(result.data);
      
    } catch (error) {
      console.error(`❌ Proxy error for ${serviceName}:`, error.message);
      
      // Return appropriate error response
      if (error.message.includes('Circuit breaker OPEN')) {
        res.status(503).json({
          success: false,
          message: 'Service temporarily unavailable',
          error: 'Circuit breaker open'
        });
      } else if (error.code === 'ECONNREFUSED' || error.code === 'ENOTFOUND') {
        res.status(503).json({
          success: false,
          message: 'Service unavailable',
          error: 'Connection refused'
        });
      } else if (error.code === 'ECONNABORTED') {
        res.status(504).json({
          success: false,
          message: 'Service timeout',
          error: 'Request timeout'
        });
      } else {
        res.status(500).json({
          success: false,
          message: 'Internal server error',
          error: error.message
        });
      }
    }
  };
};

// Gateway health check
app.get('/health', (req, res) => {
  const circuitBreakerStatus = {};
  
  for (const [serviceName, service] of Object.entries(serviceRegistry)) {
    circuitBreakerStatus[serviceName] = service.circuitBreaker.getStatus();
  }
  
  res.json({
    gateway: {
      name: 'advanced-api-gateway',
      status: 'healthy',
      timestamp: new Date().toISOString(),
      uptime: process.uptime()
    },
    circuitBreakers: circuitBreakerStatus
  });
});

// Service routes with circuit breakers
app.use('/api/users', createAdvancedProxy('user-service'));
app.use('/api/products', createAdvancedProxy('product-service'));
app.use('/api/orders', createAdvancedProxy('order-service'));

// Circuit breaker status endpoint
app.get('/circuit-breakers', (req, res) => {
  const status = {};
  
  for (const [serviceName, service] of Object.entries(serviceRegistry)) {
    status[serviceName] = service.circuitBreaker.getStatus();
  }
  
  res.json({
    success: true,
    circuitBreakers: status
  });
});

// Reset circuit breaker endpoint
app.post('/circuit-breakers/:serviceName/reset', (req, res) => {
  const { serviceName } = req.params;
  const service = serviceRegistry[serviceName];
  
  if (!service) {
    return res.status(404).json({
      success: false,
      message: 'Service not found'
    });
  }
  
  service.circuitBreaker.state = CIRCUIT_STATES.CLOSED;
  service.circuitBreaker.failureCount = 0;
  service.circuitBreaker.successCount = 0;
  service.circuitBreaker.lastFailureTime = null;
  
  res.json({
    success: true,
    message: `Circuit breaker reset for ${serviceName}`
  });
});

// Start advanced gateway
const PORT = 3000;
app.listen(PORT, () => {
  console.log(`⚡ Advanced API Gateway running on port ${PORT}`);
  console.log(`🔍 Health check: http://localhost:${PORT}/health`);
  console.log(`🔄 Circuit breakers: http://localhost:${PORT}/circuit-breakers`);
  console.log('\n⚡ Advanced Features:');
  console.log('   🔄 Circuit breakers for fault tolerance');
  console.log('   🔁 Automatic retry with exponential backoff');
  console.log('   📊 Service health monitoring');
  console.log('   🚦 Request routing and load balancing');
});
```

---

## 🎯 Summary

Today you learned:
- ✅ API Gateway concepts (hotel reception analogy!)
- ✅ Building a complete API Gateway
- ✅ Service discovery and registration
- ✅ Load balancing strategies
- ✅ Circuit breaker pattern
- ✅ Health monitoring and fault tolerance

**Next up: Cloud Deployment and Docker!** 🚀

---

*Remember: API Gateway is like a hotel reception - one entrance, proper directions, security checks, and helpful service for all visitors! 🏨*
