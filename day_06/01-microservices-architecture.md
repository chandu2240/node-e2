# Day 06 - Part 01: Microservices Architecture

## 🎯 Learning Objectives
By the end of this lesson, you will understand:
- What microservices are and why they're important
- Differences between monolithic and microservices architecture
- How to design and build microservices
- Service communication patterns
- Data management in microservices
- Service discovery and load balancing

---

## 🏗️ What are Microservices?

### Simple Explanation (for 10-year-olds!)

Imagine building with **LEGO blocks**:

#### **Monolithic App - Like a Giant Castle:**
```
🏰 ONE HUGE CASTLE
┌─────────────────────────────────────────────────────┐
│  🏠 User Management + 🛒 Shopping Cart +            │
│  💳 Payment + 📦 Shipping + 📊 Analytics           │
│  All stuck together in one piece!                  │
└─────────────────────────────────────────────────────┘

Problems:
❌ Hard to change one part without breaking others
❌ If one part breaks, everything stops working
❌ Need to rebuild entire castle for small changes
```

#### **Microservices - Like Separate LEGO Sets:**
```
🏠 User Service    🛒 Cart Service    💳 Payment Service
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│  👤 Login   │   │  🛍️ Add     │   │  💰 Process │
│  📝 Profile │   │  ❌ Remove  │   │  🔒 Secure  │
│  🔐 Auth    │   │  📊 Count   │   │  📄 Receipt │
└─────────────┘   └─────────────┘   └─────────────┘

Benefits:
✅ Change one service without affecting others
✅ If one breaks, others keep working
✅ Different teams can work on different services
✅ Use different technologies for different services
```

### Real-World Analogy
```
Microservices are like a RESTAURANT:
┌─────────────────────────────────────────────────────┐
│  🍳 Kitchen (Cooking Service)                       │
│  🍽️ Waiter (Order Service)                         │
│  💰 Cashier (Payment Service)                       │
│  📋 Manager (Inventory Service)                     │
│  Each person has ONE specific job!                  │
└─────────────────────────────────────────────────────┘
```

---

## 🔧 Building Your First Microservice

### User Service Microservice

Create `user-service.js`:
```javascript
// user-service.js - User Management Microservice
const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

console.log('👤 Building User Service Microservice!');

const app = express();
app.use(express.json());

// In-memory user storage (use database in production)
const users = new Map();
const sessions = new Map();

// Service configuration
const SERVICE_CONFIG = {
  name: 'user-service',
  version: '1.0.0',
  port: 3001,
  health: '/health',
  metrics: '/metrics'
};

// User data model
class User {
  constructor(id, email, password, profile = {}) {
    this.id = id;
    this.email = email;
    this.passwordHash = password;
    this.profile = profile;
    this.createdAt = new Date().toISOString();
    this.updatedAt = new Date().toISOString();
    this.isActive = true;
  }
  
  toJSON() {
    // Don't expose password hash
    const { passwordHash, ...userWithoutPassword } = this;
    return userWithoutPassword;
  }
}

// Middleware for authentication
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({
      success: false,
      message: 'Access token required!'
    });
  }
  
  jwt.verify(token, 'your-secret-key', (err, user) => {
    if (err) {
      return res.status(403).json({
        success: false,
        message: 'Invalid or expired token!'
      });
    }
    
    req.user = user;
    next();
  });
};

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({
    service: SERVICE_CONFIG.name,
    version: SERVICE_CONFIG.version,
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    users: users.size,
    activeSessions: sessions.size
  });
});

// Service info endpoint
app.get('/info', (req, res) => {
  res.json({
    service: SERVICE_CONFIG.name,
    version: SERVICE_CONFIG.version,
    description: 'User management microservice',
    endpoints: {
      'POST /register': 'Register new user',
      'POST /login': 'User login',
      'GET /profile': 'Get user profile',
      'PUT /profile': 'Update user profile',
      'POST /logout': 'User logout',
      'GET /users': 'List all users (admin)',
      'GET /health': 'Health check'
    }
  });
});

// Register new user
app.post('/register', async (req, res) => {
  try {
    const { email, password, profile } = req.body;
    
    // Validation
    if (!email || !password) {
      return res.status(400).json({
        success: false,
        message: 'Email and password are required!'
      });
    }
    
    // Check if user already exists
    const existingUser = Array.from(users.values()).find(u => u.email === email);
    if (existingUser) {
      return res.status(409).json({
        success: false,
        message: 'User already exists!'
      });
    }
    
    // Create new user
    const userId = Date.now().toString();
    const passwordHash = await bcrypt.hash(password, 10);
    const user = new User(userId, email, passwordHash, profile);
    
    users.set(userId, user);
    
    console.log(`✅ New user registered: ${email}`);
    
    res.status(201).json({
      success: true,
      message: 'User registered successfully!',
      user: user.toJSON()
    });
    
  } catch (error) {
    console.error('❌ Registration error:', error);
    res.status(500).json({
      success: false,
      message: 'Internal server error'
    });
  }
});

// User login
app.post('/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    
    if (!email || !password) {
      return res.status(400).json({
        success: false,
        message: 'Email and password are required!'
      });
    }
    
    // Find user
    const user = Array.from(users.values()).find(u => u.email === email);
    if (!user) {
      return res.status(401).json({
        success: false,
        message: 'Invalid credentials!'
      });
    }
    
    // Verify password
    const isValidPassword = await bcrypt.compare(password, user.passwordHash);
    if (!isValidPassword) {
      return res.status(401).json({
        success: false,
        message: 'Invalid credentials!'
      });
    }
    
    // Create JWT token
    const token = jwt.sign(
      { userId: user.id, email: user.email },
      'your-secret-key',
      { expiresIn: '24h' }
    );
    
    // Store session
    const sessionId = Date.now().toString();
    sessions.set(sessionId, {
      userId: user.id,
      token,
      createdAt: new Date().toISOString()
    });
    
    console.log(`✅ User logged in: ${email}`);
    
    res.json({
      success: true,
      message: 'Login successful!',
      token,
      user: user.toJSON()
    });
    
  } catch (error) {
    console.error('❌ Login error:', error);
    res.status(500).json({
      success: false,
      message: 'Internal server error'
    });
  }
});

// Get user profile
app.get('/profile', authenticateToken, (req, res) => {
  const user = users.get(req.user.userId);
  
  if (!user) {
    return res.status(404).json({
      success: false,
      message: 'User not found!'
    });
  }
  
  res.json({
    success: true,
    user: user.toJSON()
  });
});

// Update user profile
app.put('/profile', authenticateToken, (req, res) => {
  const user = users.get(req.user.userId);
  
  if (!user) {
    return res.status(404).json({
      success: false,
      message: 'User not found!'
    });
  }
  
  const { profile } = req.body;
  
  // Update profile
  user.profile = { ...user.profile, ...profile };
  user.updatedAt = new Date().toISOString();
  
  users.set(user.id, user);
  
  console.log(`✅ Profile updated for user: ${user.email}`);
  
  res.json({
    success: true,
    message: 'Profile updated successfully!',
    user: user.toJSON()
  });
});

// Logout user
app.post('/logout', authenticateToken, (req, res) => {
  // Find and remove session
  for (const [sessionId, session] of sessions) {
    if (session.userId === req.user.userId) {
      sessions.delete(sessionId);
      break;
    }
  }
  
  console.log(`✅ User logged out: ${req.user.email}`);
  
  res.json({
    success: true,
    message: 'Logged out successfully!'
  });
});

// List all users (admin endpoint)
app.get('/users', authenticateToken, (req, res) => {
  const allUsers = Array.from(users.values()).map(user => user.toJSON());
  
  res.json({
    success: true,
    users: allUsers,
    count: allUsers.length
  });
});

// Service-to-service endpoint (for other microservices)
app.get('/internal/user/:userId', (req, res) => {
  const { userId } = req.params;
  const user = users.get(userId);
  
  if (!user) {
    return res.status(404).json({
      success: false,
      message: 'User not found!'
    });
  }
  
  res.json({
    success: true,
    user: user.toJSON()
  });
});

// Error handling middleware
app.use((error, req, res, next) => {
  console.error('❌ Service error:', error);
  res.status(500).json({
    success: false,
    message: 'Internal server error',
    service: SERVICE_CONFIG.name
  });
});

// Start service
const PORT = SERVICE_CONFIG.port;
app.listen(PORT, () => {
  console.log(`👤 User Service running on port ${PORT}`);
  console.log(`🔍 Health check: http://localhost:${PORT}/health`);
  console.log(`📊 Service info: http://localhost:${PORT}/info`);
});

// Demo usage
console.log(`
👤 USER SERVICE DEMO:

1. Register user:
   POST http://localhost:3001/register
   {
     "email": "john@example.com",
     "password": "password123",
     "profile": { "name": "John Doe", "age": 25 }
   }

2. Login:
   POST http://localhost:3001/login
   {
     "email": "john@example.com",
     "password": "password123"
   }

3. Get profile:
   GET http://localhost:3001/profile
   Authorization: Bearer <token>

4. Update profile:
   PUT http://localhost:3001/profile
   Authorization: Bearer <token>
   {
     "profile": { "age": 26, "city": "New York" }
   }
`);
```

### Product Service Microservice

Create `product-service.js`:
```javascript
// product-service.js - Product Management Microservice
const express = require('express');
const axios = require('axios');

console.log('📦 Building Product Service Microservice!');

const app = express();
app.use(express.json());

// In-memory product storage
const products = new Map();
const categories = new Map();

// Service configuration
const SERVICE_CONFIG = {
  name: 'product-service',
  version: '1.0.0',
  port: 3002,
  dependencies: {
    userService: 'http://localhost:3001'
  }
};

// Product data model
class Product {
  constructor(id, name, description, price, category, stock = 0) {
    this.id = id;
    this.name = name;
    this.description = description;
    this.price = price;
    this.category = category;
    this.stock = stock;
    this.createdAt = new Date().toISOString();
    this.updatedAt = new Date().toISOString();
    this.isActive = true;
  }
}

// Category data model
class Category {
  constructor(id, name, description) {
    this.id = id;
    this.name = name;
    this.description = description;
    this.createdAt = new Date().toISOString();
  }
}

// Initialize demo data
function initializeDemoData() {
  // Create categories
  const electronics = new Category('1', 'Electronics', 'Electronic devices and gadgets');
  const books = new Category('2', 'Books', 'Physical and digital books');
  const clothing = new Category('3', 'Clothing', 'Apparel and accessories');
  
  categories.set('1', electronics);
  categories.set('2', books);
  categories.set('3', clothing);
  
  // Create products
  const laptop = new Product('1', 'Gaming Laptop', 'High-performance gaming laptop', 1299.99, '1', 10);
  const smartphone = new Product('2', 'Smartphone', 'Latest flagship smartphone', 899.99, '1', 25);
  const book = new Product('3', 'Node.js Guide', 'Complete Node.js development guide', 29.99, '2', 50);
  const tshirt = new Product('4', 'Cotton T-Shirt', 'Comfortable cotton t-shirt', 19.99, '3', 100);
  
  products.set('1', laptop);
  products.set('2', smartphone);
  products.set('3', book);
  products.set('4', tshirt);
  
  console.log('✅ Demo data initialized!');
}

// Middleware to verify user (calls User Service)
const verifyUser = async (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({
      success: false,
      message: 'Access token required!'
    });
  }
  
  try {
    // Call User Service to verify token
    const userResponse = await axios.get(`${SERVICE_CONFIG.dependencies.userService}/profile`, {
      headers: { Authorization: `Bearer ${token}` }
    });
    
    req.user = userResponse.data.user;
    next();
  } catch (error) {
    return res.status(401).json({
      success: false,
      message: 'Invalid token or user service unavailable!'
    });
  }
};

// Health check
app.get('/health', (req, res) => {
  res.json({
    service: SERVICE_CONFIG.name,
    version: SERVICE_CONFIG.version,
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    products: products.size,
    categories: categories.size,
    dependencies: SERVICE_CONFIG.dependencies
  });
});

// Service info
app.get('/info', (req, res) => {
  res.json({
    service: SERVICE_CONFIG.name,
    version: SERVICE_CONFIG.version,
    description: 'Product management microservice',
    endpoints: {
      'GET /products': 'List all products',
      'GET /products/:id': 'Get product details',
      'POST /products': 'Create new product (auth required)',
      'PUT /products/:id': 'Update product (auth required)',
      'DELETE /products/:id': 'Delete product (auth required)',
      'GET /categories': 'List all categories',
      'GET /search': 'Search products',
      'GET /health': 'Health check'
    }
  });
});

// Get all products
app.get('/products', (req, res) => {
  const { category, minPrice, maxPrice, inStock } = req.query;
  let productList = Array.from(products.values());
  
  // Apply filters
  if (category) {
    productList = productList.filter(p => p.category === category);
  }
  
  if (minPrice) {
    productList = productList.filter(p => p.price >= parseFloat(minPrice));
  }
  
  if (maxPrice) {
    productList = productList.filter(p => p.price <= parseFloat(maxPrice));
  }
  
  if (inStock === 'true') {
    productList = productList.filter(p => p.stock > 0);
  }
  
  res.json({
    success: true,
    products: productList,
    count: productList.length,
    filters: { category, minPrice, maxPrice, inStock }
  });
});

// Get product by ID
app.get('/products/:id', (req, res) => {
  const { id } = req.params;
  const product = products.get(id);
  
  if (!product) {
    return res.status(404).json({
      success: false,
      message: 'Product not found!'
    });
  }
  
  res.json({
    success: true,
    product
  });
});

// Create new product
app.post('/products', verifyUser, (req, res) => {
  const { name, description, price, category, stock } = req.body;
  
  if (!name || !price || !category) {
    return res.status(400).json({
      success: false,
      message: 'Name, price, and category are required!'
    });
  }
  
  // Verify category exists
  if (!categories.has(category)) {
    return res.status(400).json({
      success: false,
      message: 'Invalid category!'
    });
  }
  
  const productId = Date.now().toString();
  const product = new Product(productId, name, description, price, category, stock);
  
  products.set(productId, product);
  
  console.log(`✅ Product created: ${name} by ${req.user.email}`);
  
  res.status(201).json({
    success: true,
    message: 'Product created successfully!',
    product
  });
});

// Update product
app.put('/products/:id', verifyUser, (req, res) => {
  const { id } = req.params;
  const product = products.get(id);
  
  if (!product) {
    return res.status(404).json({
      success: false,
      message: 'Product not found!'
    });
  }
  
  const { name, description, price, category, stock } = req.body;
  
  // Update fields
  if (name) product.name = name;
  if (description) product.description = description;
  if (price) product.price = price;
  if (category && categories.has(category)) product.category = category;
  if (stock !== undefined) product.stock = stock;
  
  product.updatedAt = new Date().toISOString();
  products.set(id, product);
  
  console.log(`✅ Product updated: ${product.name} by ${req.user.email}`);
  
  res.json({
    success: true,
    message: 'Product updated successfully!',
    product
  });
});

// Delete product
app.delete('/products/:id', verifyUser, (req, res) => {
  const { id } = req.params;
  const product = products.get(id);
  
  if (!product) {
    return res.status(404).json({
      success: false,
      message: 'Product not found!'
    });
  }
  
  products.delete(id);
  
  console.log(`✅ Product deleted: ${product.name} by ${req.user.email}`);
  
  res.json({
    success: true,
    message: 'Product deleted successfully!'
  });
});

// Get all categories
app.get('/categories', (req, res) => {
  const categoryList = Array.from(categories.values());
  
  res.json({
    success: true,
    categories: categoryList,
    count: categoryList.length
  });
});

// Search products
app.get('/search', (req, res) => {
  const { q } = req.query;
  
  if (!q) {
    return res.status(400).json({
      success: false,
      message: 'Search query is required!'
    });
  }
  
  const searchResults = Array.from(products.values()).filter(product => 
    product.name.toLowerCase().includes(q.toLowerCase()) ||
    product.description.toLowerCase().includes(q.toLowerCase())
  );
  
  res.json({
    success: true,
    results: searchResults,
    count: searchResults.length,
    query: q
  });
});

// Service-to-service endpoint (for other microservices)
app.get('/internal/product/:productId', (req, res) => {
  const { productId } = req.params;
  const product = products.get(productId);
  
  if (!product) {
    return res.status(404).json({
      success: false,
      message: 'Product not found!'
    });
  }
  
  res.json({
    success: true,
    product
  });
});

// Update stock (for order service)
app.patch('/internal/stock/:productId', express.json(), (req, res) => {
  const { productId } = req.params;
  const { quantity } = req.body;
  
  const product = products.get(productId);
  if (!product) {
    return res.status(404).json({
      success: false,
      message: 'Product not found!'
    });
  }
  
  if (product.stock < quantity) {
    return res.status(400).json({
      success: false,
      message: 'Insufficient stock!'
    });
  }
  
  product.stock -= quantity;
  product.updatedAt = new Date().toISOString();
  products.set(productId, product);
  
  res.json({
    success: true,
    message: 'Stock updated successfully!',
    product
  });
});

// Initialize demo data
initializeDemoData();

// Start service
const PORT = SERVICE_CONFIG.port;
app.listen(PORT, () => {
  console.log(`📦 Product Service running on port ${PORT}`);
  console.log(`🔍 Health check: http://localhost:${PORT}/health`);
  console.log(`📊 Service info: http://localhost:${PORT}/info`);
});

// Demo usage
console.log(`
📦 PRODUCT SERVICE DEMO:

1. Get all products:
   GET http://localhost:3002/products

2. Get product by ID:
   GET http://localhost:3002/products/1

3. Search products:
   GET http://localhost:3002/search?q=laptop

4. Create product (requires auth):
   POST http://localhost:3002/products
   Authorization: Bearer <token>
   {
     "name": "New Product",
     "description": "Amazing product",
     "price": 99.99,
     "category": "1",
     "stock": 20
   }

5. Get categories:
   GET http://localhost:3002/categories
`);
```

---

## 🔄 Service Communication

### Service Communication Example

Create `service-communication.js`:
```javascript
// service-communication.js - Inter-service communication example
const express = require('express');
const axios = require('axios');

console.log('🔄 Building Service Communication Example!');

const app = express();
app.use(express.json());

// Service registry (in production, use actual service discovery)
const SERVICE_REGISTRY = {
  user: 'http://localhost:3001',
  product: 'http://localhost:3002',
  order: 'http://localhost:3003'
};

// Service configuration
const SERVICE_CONFIG = {
  name: 'order-service',
  version: '1.0.0',
  port: 3003
};

// In-memory order storage
const orders = new Map();

// Order data model
class Order {
  constructor(id, userId, items, totalAmount) {
    this.id = id;
    this.userId = userId;
    this.items = items;
    this.totalAmount = totalAmount;
    this.status = 'pending';
    this.createdAt = new Date().toISOString();
    this.updatedAt = new Date().toISOString();
  }
}

// Helper function to make service calls
async function callService(serviceName, endpoint, options = {}) {
  const baseUrl = SERVICE_REGISTRY[serviceName];
  if (!baseUrl) {
    throw new Error(`Service ${serviceName} not found in registry`);
  }
  
  const url = `${baseUrl}${endpoint}`;
  
  try {
    const response = await axios({
      url,
      method: options.method || 'GET',
      headers: options.headers || {},
      data: options.data
    });
    
    return response.data;
  } catch (error) {
    console.error(`❌ Service call failed: ${serviceName}${endpoint}`, error.message);
    throw error;
  }
}

// Middleware to verify user through User Service
const verifyUser = async (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({
      success: false,
      message: 'Access token required!'
    });
  }
  
  try {
    const userResponse = await callService('user', '/profile', {
      headers: { Authorization: `Bearer ${token}` }
    });
    
    req.user = userResponse.user;
    next();
  } catch (error) {
    return res.status(401).json({
      success: false,
      message: 'Invalid token or user service unavailable!'
    });
  }
};

// Health check
app.get('/health', async (req, res) => {
  const healthStatus = {
    service: SERVICE_CONFIG.name,
    version: SERVICE_CONFIG.version,
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    orders: orders.size,
    dependencies: {}
  };
  
  // Check dependency health
  for (const [serviceName, serviceUrl] of Object.entries(SERVICE_REGISTRY)) {
    try {
      const response = await callService(serviceName, '/health');
      healthStatus.dependencies[serviceName] = {
        status: 'healthy',
        response: response.status
      };
    } catch (error) {
      healthStatus.dependencies[serviceName] = {
        status: 'unhealthy',
        error: error.message
      };
    }
  }
  
  res.json(healthStatus);
});

// Create order (demonstrates service communication)
app.post('/orders', verifyUser, async (req, res) => {
  try {
    const { items } = req.body; // [{ productId, quantity }]
    
    if (!items || !Array.isArray(items) || items.length === 0) {
      return res.status(400).json({
        success: false,
        message: 'Order items are required!'
      });
    }
    
    let totalAmount = 0;
    const orderItems = [];
    
    // Process each item
    for (const item of items) {
      const { productId, quantity } = item;
      
      if (!productId || !quantity || quantity <= 0) {
        return res.status(400).json({
          success: false,
          message: 'Invalid item: productId and quantity are required!'
        });
      }
      
      // Get product details from Product Service
      try {
        const productResponse = await callService('product', `/products/${productId}`);
        const product = productResponse.product;
        
        // Check stock availability
        if (product.stock < quantity) {
          return res.status(400).json({
            success: false,
            message: `Insufficient stock for product: ${product.name}`
          });
        }
        
        const itemTotal = product.price * quantity;
        totalAmount += itemTotal;
        
        orderItems.push({
          productId,
          productName: product.name,
          price: product.price,
          quantity,
          total: itemTotal
        });
        
      } catch (error) {
        return res.status(400).json({
          success: false,
          message: `Product not found: ${productId}`
        });
      }
    }
    
    // Create order
    const orderId = Date.now().toString();
    const order = new Order(orderId, req.user.id, orderItems, totalAmount);
    orders.set(orderId, order);
    
    // Update stock in Product Service
    try {
      for (const item of items) {
        await callService('product', `/internal/stock/${item.productId}`, {
          method: 'PATCH',
          data: { quantity: item.quantity }
        });
      }
    } catch (error) {
      // Rollback: remove order if stock update fails
      orders.delete(orderId);
      return res.status(500).json({
        success: false,
        message: 'Failed to update stock. Order cancelled.'
      });
    }
    
    // Update order status
    order.status = 'confirmed';
    order.updatedAt = new Date().toISOString();
    orders.set(orderId, order);
    
    console.log(`✅ Order created: ${orderId} for user ${req.user.email}`);
    
    res.status(201).json({
      success: true,
      message: 'Order created successfully!',
      order
    });
    
  } catch (error) {
    console.error('❌ Order creation error:', error);
    res.status(500).json({
      success: false,
      message: 'Failed to create order'
    });
  }
});

// Get user orders
app.get('/orders', verifyUser, (req, res) => {
  const userOrders = Array.from(orders.values()).filter(order => order.userId === req.user.id);
  
  res.json({
    success: true,
    orders: userOrders,
    count: userOrders.length
  });
});

// Get order by ID
app.get('/orders/:id', verifyUser, (req, res) => {
  const { id } = req.params;
  const order = orders.get(id);
  
  if (!order) {
    return res.status(404).json({
      success: false,
      message: 'Order not found!'
    });
  }
  
  // Check if order belongs to user
  if (order.userId !== req.user.id) {
    return res.status(403).json({
      success: false,
      message: 'Access denied!'
    });
  }
  
  res.json({
    success: true,
    order
  });
});

// Get order details with product information
app.get('/orders/:id/details', verifyUser, async (req, res) => {
  const { id } = req.params;
  const order = orders.get(id);
  
  if (!order) {
    return res.status(404).json({
      success: false,
      message: 'Order not found!'
    });
  }
  
  if (order.userId !== req.user.id) {
    return res.status(403).json({
      success: false,
      message: 'Access denied!'
    });
  }
  
  try {
    // Get detailed product information
    const detailedItems = await Promise.all(
      order.items.map(async (item) => {
        try {
          const productResponse = await callService('product', `/products/${item.productId}`);
          return {
            ...item,
            productDetails: productResponse.product
          };
        } catch (error) {
          return {
            ...item,
            productDetails: { error: 'Product not found' }
          };
        }
      })
    );
    
    res.json({
      success: true,
      order: {
        ...order,
        items: detailedItems
      }
    });
    
  } catch (error) {
    console.error('❌ Error fetching order details:', error);
    res.status(500).json({
      success: false,
      message: 'Failed to fetch order details'
    });
  }
});

// Start service
const PORT = SERVICE_CONFIG.port;
app.listen(PORT, () => {
  console.log(`🛍️ Order Service running on port ${PORT}`);
  console.log(`🔍 Health check: http://localhost:${PORT}/health`);
  console.log('\n📞 Service Dependencies:');
  Object.entries(SERVICE_REGISTRY).forEach(([name, url]) => {
    console.log(`   ${name}: ${url}`);
  });
});

// Demo usage
console.log(`
🛍️ ORDER SERVICE DEMO:

1. Create order:
   POST http://localhost:3003/orders
   Authorization: Bearer <token>
   {
     "items": [
       { "productId": "1", "quantity": 2 },
       { "productId": "2", "quantity": 1 }
     ]
   }

2. Get user orders:
   GET http://localhost:3003/orders
   Authorization: Bearer <token>

3. Get order details:
   GET http://localhost:3003/orders/123/details
   Authorization: Bearer <token>

4. Check service health:
   GET http://localhost:3003/health
`);
```

---

## 🎯 Summary

Today you learned:
- ✅ Microservices architecture concepts (LEGO blocks analogy!)
- ✅ Building individual microservices
- ✅ Service-to-service communication
- ✅ Data management across services
- ✅ Health checks and monitoring
- ✅ Authentication across services

**Next up: API Gateways and Service Discovery!** 🚀

---

*Remember: Microservices are like building with LEGO blocks - each service is independent but works together to create something amazing! 🏗️*
