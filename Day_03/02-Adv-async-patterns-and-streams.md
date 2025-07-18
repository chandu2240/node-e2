# Day 03 - Part 02: Advanced Async Patterns and Streams

## 🎯 Learning Objectives
By the end of this lesson, you will understand:
- Advanced async patterns and best practices
- What streams are and why they're important
- Different types of streams in Node.js
- How to work with readable and writable streams
- Stream transformations and piping
- Backpressure and flow control
- Event emitters and custom events
- Real-world streaming applications

---

## 🌊 What Are Streams?

### Simple Explanation (for 10-year-olds!)

Imagine you want to move water from a big swimming pool to your garden:

#### **Without Streams (Bad Way):**
```
🏊‍♂️ Swimming Pool → 🪣 Giant Bucket → 🌱 Garden
```
You'd need a **HUGE** bucket to carry all the water at once! 💪😰

#### **With Streams (Smart Way):**
```
🏊‍♂️ Swimming Pool → 🚰 Hose → 🌱 Garden
```
Water flows through the hose **bit by bit**, continuously! 💧💧💧

### Real-World Analogy
```
Streams are like a conveyor belt at a factory:
┌─────────────────────────────────────────────────────┐
│  📦 → 📦 → 📦 → 📦 → 📦 → 📦 → 📦 → 📦 → 📦      │
│  Items move one by one, continuously                │
│  Workers can process items as they arrive           │
│  No need to wait for ALL items at once!            │
└─────────────────────────────────────────────────────┘
```

### Why Streams Are Amazing
- **Memory Efficient**: Don't load everything at once
- **Fast**: Start processing immediately
- **Scalable**: Handle huge amounts of data
- **Flexible**: Can transform data on the fly

---

## 🎪 Advanced Async Patterns

### Pattern 1: Rate Limiting

Create `rate-limiting.js`:
```javascript
// rate-limiting.js - Rate limiting async operations
console.log('🚦 Learning rate limiting!');

// Simple rate limiter class
class RateLimiter {
  constructor(maxRequests, timeWindow) {
    this.maxRequests = maxRequests;
    this.timeWindow = timeWindow;
    this.requests = [];
  }
  
  async execute(fn) {
    // Clean old requests
    const now = Date.now();
    this.requests = this.requests.filter(time => now - time < this.timeWindow);
    
    // Check if we can make a request
    if (this.requests.length >= this.maxRequests) {
      const waitTime = this.timeWindow - (now - this.requests[0]);
      console.log(`⏰ Rate limit reached! Waiting ${waitTime}ms...`);
      await new Promise(resolve => setTimeout(resolve, waitTime));
      return this.execute(fn);
    }
    
    // Record this request
    this.requests.push(now);
    
    // Execute the function
    return await fn();
  }
}

// Example API call function
async function apiCall(id) {
  console.log(`📡 Making API call ${id}...`);
  await new Promise(resolve => setTimeout(resolve, 500));
  console.log(`✅ API call ${id} completed!`);
  return `Result ${id}`;
}

// Create rate limiter: max 3 requests per 2 seconds
const rateLimiter = new RateLimiter(3, 2000);

async function demonstrateRateLimit() {
  console.log('🚀 Starting rate limiting demo...');
  
  // Make 6 API calls - should be rate limited
  const promises = [];
  for (let i = 1; i <= 6; i++) {
    promises.push(rateLimiter.execute(() => apiCall(i)));
  }
  
  const results = await Promise.all(promises);
  console.log('🎉 All API calls completed!');
  console.log('📊 Results:', results);
}

demonstrateRateLimit();
```

### Pattern 2: Circuit Breaker

Create `circuit-breaker.js`:
```javascript
// circuit-breaker.js - Circuit breaker pattern
console.log('🔌 Learning circuit breaker pattern!');

class CircuitBreaker {
  constructor(threshold, timeout) {
    this.threshold = threshold; // Number of failures before opening
    this.timeout = timeout; // Time to wait before trying again
    this.failureCount = 0;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.nextAttempt = 0;
  }
  
  async execute(fn) {
    console.log(`🔌 Circuit breaker state: ${this.state}`);
    
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('🚫 Circuit breaker is OPEN - request blocked!');
      }
      this.state = 'HALF_OPEN';
      console.log('🔄 Circuit breaker is now HALF_OPEN - trying again...');
    }
    
    try {
      const result = await fn();
      
      if (this.state === 'HALF_OPEN') {
        this.state = 'CLOSED';
        this.failureCount = 0;
        console.log('✅ Circuit breaker is now CLOSED - service recovered!');
      }
      
      return result;
    } catch (error) {
      this.failureCount++;
      console.log(`❌ Failure ${this.failureCount}/${this.threshold}`);
      
      if (this.failureCount >= this.threshold) {
        this.state = 'OPEN';
        this.nextAttempt = Date.now() + this.timeout;
        console.log(`🚫 Circuit breaker is now OPEN for ${this.timeout}ms`);
      }
      
      throw error;
    }
  }
}

// Simulate unreliable service
let callCount = 0;
async function unreliableService() {
  callCount++;
  console.log(`🌐 Service call ${callCount}`);
  
  // Fail for first 3 calls, then succeed
  if (callCount <= 3) {
    throw new Error('💥 Service is down!');
  }
  
  return `✅ Service call ${callCount} succeeded!`;
}

async function demonstrateCircuitBreaker() {
  const circuitBreaker = new CircuitBreaker(2, 3000); // 2 failures, 3 second timeout
  
  // Make multiple calls
  for (let i = 1; i <= 8; i++) {
    try {
      console.log(`\n--- Call ${i} ---`);
      const result = await circuitBreaker.execute(unreliableService);
      console.log('📝 Result:', result);
    } catch (error) {
      console.log('❌ Error:', error.message);
    }
    
    // Wait between calls
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
}

demonstrateCircuitBreaker();
```

### Pattern 3: Async Queue

Create `async-queue.js`:
```javascript
// async-queue.js - Async queue pattern
console.log('🚶‍♂️ Learning async queue pattern!');

class AsyncQueue {
  constructor(concurrency = 1) {
    this.concurrency = concurrency;
    this.running = 0;
    this.queue = [];
  }
  
  async add(fn) {
    return new Promise((resolve, reject) => {
      this.queue.push({
        fn,
        resolve,
        reject
      });
      
      this.process();
    });
  }
  
  async process() {
    if (this.running >= this.concurrency || this.queue.length === 0) {
      return;
    }
    
    this.running++;
    const { fn, resolve, reject } = this.queue.shift();
    
    try {
      const result = await fn();
      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      this.running--;
      this.process(); // Process next item
    }
  }
  
  get size() {
    return this.queue.length;
  }
  
  get pending() {
    return this.running;
  }
}

// Example tasks
async function createTask(name, duration) {
  console.log(`🏃‍♂️ Starting task: ${name}`);
  await new Promise(resolve => setTimeout(resolve, duration));
  console.log(`✅ Completed task: ${name}`);
  return `Result from ${name}`;
}

async function demonstrateAsyncQueue() {
  console.log('🚀 Starting async queue demo...');
  
  // Create queue with concurrency of 2
  const queue = new AsyncQueue(2);
  
  // Add tasks to queue
  const tasks = [
    () => createTask('Task 1', 2000),
    () => createTask('Task 2', 1500),
    () => createTask('Task 3', 1000),
    () => createTask('Task 4', 2500),
    () => createTask('Task 5', 1200),
    () => createTask('Task 6', 1800)
  ];
  
  console.log('📋 Adding tasks to queue...');
  const promises = tasks.map(task => queue.add(task));
  
  // Monitor queue progress
  const monitor = setInterval(() => {
    console.log(`📊 Queue size: ${queue.size}, Running: ${queue.pending}`);
  }, 500);
  
  // Wait for all tasks to complete
  const results = await Promise.all(promises);
  clearInterval(monitor);
  
  console.log('🎉 All tasks completed!');
  console.log('📊 Results:', results);
}

demonstrateAsyncQueue();
```

---

## 🌊 Understanding Streams

### Basic Stream Concepts

Create `stream-basics.js`:
```javascript
// stream-basics.js - Understanding streams
const { Readable, Writable, Transform } = require('stream');
const fs = require('fs');

console.log('🌊 Learning about streams!');

// 1. Creating a simple readable stream
console.log('1. Creating a readable stream:');

class NumberStream extends Readable {
  constructor(options) {
    super(options);
    this.current = 1;
    this.max = 5;
  }
  
  _read() {
    if (this.current <= this.max) {
      console.log(`📤 Producing number: ${this.current}`);
      this.push(`Number ${this.current}\n`);
      this.current++;
    } else {
      console.log('📤 Stream finished!');
      this.push(null); // End the stream
    }
  }
}

const numberStream = new NumberStream();

// 2. Creating a simple writable stream
console.log('2. Creating a writable stream:');

class LogStream extends Writable {
  _write(chunk, encoding, callback) {
    console.log(`📝 Received: ${chunk.toString().trim()}`);
    callback(); // Signal that we're done processing this chunk
  }
}

const logStream = new LogStream();

// 3. Connecting streams with pipe
console.log('3. Connecting streams with pipe:');
numberStream.pipe(logStream);

// 4. Creating a transform stream
setTimeout(() => {
  console.log('\n4. Creating a transform stream:');
  
  class UpperCaseTransform extends Transform {
    _transform(chunk, encoding, callback) {
      const transformed = chunk.toString().toUpperCase();
      console.log(`🔄 Transforming: ${chunk.toString().trim()} → ${transformed.trim()}`);
      callback(null, transformed);
    }
  }
  
  const upperCaseStream = new UpperCaseTransform();
  const anotherNumberStream = new NumberStream();
  const anotherLogStream = new LogStream();
  
  // Chain streams: Numbers → UpperCase → Log
  anotherNumberStream
    .pipe(upperCaseStream)
    .pipe(anotherLogStream);
    
}, 3000);
```

### File Streaming

Create `file-streaming.js`:
```javascript
// file-streaming.js - Working with file streams
const fs = require('fs');
const path = require('path');

console.log('📁 Learning file streaming!');

// 1. Create a large file to work with
console.log('1. Creating a large file...');

const largeFile = 'large-file.txt';
const writeStream = fs.createWriteStream(largeFile);

// Write 1000 lines
for (let i = 1; i <= 1000; i++) {
  writeStream.write(`Line ${i}: This is some sample content for line ${i}\n`);
}

writeStream.end();

writeStream.on('finish', () => {
  console.log('✅ Large file created!');
  
  // 2. Read file using streams (memory efficient)
  console.log('2. Reading file using streams...');
  
  const readStream = fs.createReadStream(largeFile, { encoding: 'utf8' });
  let lineCount = 0;
  let buffer = '';
  
  readStream.on('data', (chunk) => {
    buffer += chunk;
    const lines = buffer.split('\n');
    
    // Process complete lines
    for (let i = 0; i < lines.length - 1; i++) {
      lineCount++;
      if (lineCount <= 5 || lineCount % 100 === 0) {
        console.log(`📖 Line ${lineCount}: ${lines[i]}`);
      }
    }
    
    // Keep the last incomplete line in buffer
    buffer = lines[lines.length - 1];
  });
  
  readStream.on('end', () => {
    console.log(`✅ File reading completed! Total lines: ${lineCount}`);
    
    // 3. Copy file using streams
    console.log('3. Copying file using streams...');
    
    const sourceStream = fs.createReadStream(largeFile);
    const destinationStream = fs.createWriteStream('copied-file.txt');
    
    sourceStream.pipe(destinationStream);
    
    destinationStream.on('finish', () => {
      console.log('✅ File copied successfully!');
      
      // 4. Transform file content while copying
      console.log('4. Transforming file content...');
      
      const { Transform } = require('stream');
      
      class LineNumberTransform extends Transform {
        constructor() {
          super({ objectMode: false });
          this.lineNumber = 0;
        }
        
        _transform(chunk, encoding, callback) {
          const lines = chunk.toString().split('\n');
          const transformedLines = lines.map(line => {
            if (line.trim()) {
              this.lineNumber++;
              return `[${this.lineNumber}] ${line}`;
            }
            return line;
          });
          
          callback(null, transformedLines.join('\n'));
        }
      }
      
      const sourceStream2 = fs.createReadStream(largeFile);
      const transformStream = new LineNumberTransform();
      const destinationStream2 = fs.createWriteStream('transformed-file.txt');
      
      sourceStream2
        .pipe(transformStream)
        .pipe(destinationStream2);
      
      destinationStream2.on('finish', () => {
        console.log('✅ File transformation completed!');
        
        // Cleanup
        setTimeout(() => {
          fs.unlinkSync(largeFile);
          fs.unlinkSync('copied-file.txt');
          fs.unlinkSync('transformed-file.txt');
          console.log('🧹 Cleanup completed!');
        }, 2000);
      });
    });
  });
  
  readStream.on('error', (error) => {
    console.log('❌ Error reading file:', error.message);
  });
});

writeStream.on('error', (error) => {
  console.log('❌ Error writing file:', error.message);
});
```

### Stream Processing Pipeline

Create `stream-pipeline.js`:
```javascript
// stream-pipeline.js - Stream processing pipeline
const { Readable, Transform, Writable, pipeline } = require('stream');
const { promisify } = require('util');

console.log('🏭 Learning stream processing pipeline!');

// Create a data source
class DataSource extends Readable {
  constructor(options) {
    super({ objectMode: true, ...options });
    this.current = 1;
    this.max = 10;
  }
  
  _read() {
    if (this.current <= this.max) {
      const data = {
        id: this.current,
        name: `Item ${this.current}`,
        value: Math.random() * 100
      };
      
      console.log(`📊 Generating data: ${JSON.stringify(data)}`);
      this.push(data);
      this.current++;
    } else {
      this.push(null); // End stream
    }
  }
}

// Create data transformers
class ValidateTransform extends Transform {
  constructor(options) {
    super({ objectMode: true, ...options });
  }
  
  _transform(data, encoding, callback) {
    console.log(`🔍 Validating: ${data.name}`);
    
    if (data.value > 50) {
      console.log(`✅ ${data.name} passed validation (value: ${data.value.toFixed(2)})`);
      callback(null, data);
    } else {
      console.log(`❌ ${data.name} failed validation (value: ${data.value.toFixed(2)})`);
      callback(); // Skip this item
    }
  }
}

class EnrichTransform extends Transform {
  constructor(options) {
    super({ objectMode: true, ...options });
  }
  
  _transform(data, encoding, callback) {
    console.log(`🔧 Enriching: ${data.name}`);
    
    const enrichedData = {
      ...data,
      category: data.value > 75 ? 'premium' : 'standard',
      processed: true,
      timestamp: new Date().toISOString()
    };
    
    console.log(`✨ Enriched: ${enrichedData.name} (category: ${enrichedData.category})`);
    callback(null, enrichedData);
  }
}

// Create data sink
class DataSink extends Writable {
  constructor(options) {
    super({ objectMode: true, ...options });
    this.processedCount = 0;
  }
  
  _write(data, encoding, callback) {
    this.processedCount++;
    console.log(`💾 Storing: ${data.name} (${this.processedCount} items processed)`);
    callback();
  }
  
  _final(callback) {
    console.log(`🎉 Processing complete! Total items: ${this.processedCount}`);
    callback();
  }
}

// Create pipeline using promisified pipeline
const pipelineAsync = promisify(pipeline);

async function runPipeline() {
  console.log('🚀 Starting data processing pipeline...');
  
  try {
    await pipelineAsync(
      new DataSource(),
      new ValidateTransform(),
      new EnrichTransform(),
      new DataSink()
    );
    
    console.log('✅ Pipeline completed successfully!');
  } catch (error) {
    console.log('❌ Pipeline error:', error.message);
  }
}

runPipeline();
```

---

## 📡 Event Emitters

### Basic Event Emitter

Create `event-emitter-basics.js`:
```javascript
// event-emitter-basics.js - Understanding Event Emitters
const EventEmitter = require('events');

console.log('📡 Learning Event Emitters!');

// 1. Basic event emitter
console.log('1. Basic event emitter:');

const myEmitter = new EventEmitter();

// Add event listeners
myEmitter.on('message', (data) => {
  console.log('📨 Received message:', data);
});

myEmitter.on('error', (error) => {
  console.log('❌ Error occurred:', error.message);
});

// Emit events
myEmitter.emit('message', 'Hello World!');
myEmitter.emit('message', 'This is another message');
myEmitter.emit('error', new Error('Something went wrong'));

// 2. Custom event emitter class
console.log('\n2. Custom event emitter class:');

class Restaurant extends EventEmitter {
  constructor(name) {
    super();
    this.name = name;
    this.orders = [];
  }
  
  takeOrder(customerName, dish) {
    const order = {
      id: this.orders.length + 1,
      customer: customerName,
      dish: dish,
      status: 'received'
    };
    
    this.orders.push(order);
    console.log(`📋 Order received: ${customerName} ordered ${dish}`);
    
    // Emit order received event
    this.emit('orderReceived', order);
    
    // Simulate cooking
    setTimeout(() => {
      this.cookOrder(order);
    }, 2000);
  }
  
  cookOrder(order) {
    console.log(`👨‍🍳 Cooking ${order.dish}...`);
    order.status = 'cooking';
    
    // Emit cooking started event
    this.emit('cookingStarted', order);
    
    // Simulate cooking time
    setTimeout(() => {
      order.status = 'ready';
      console.log(`🍽️ ${order.dish} is ready!`);
      
      // Emit order ready event
      this.emit('orderReady', order);
    }, 3000);
  }
}

const restaurant = new Restaurant('Node.js Cafe');

// Add event listeners
restaurant.on('orderReceived', (order) => {
  console.log(`✅ Order #${order.id} received and confirmed`);
});

restaurant.on('cookingStarted', (order) => {
  console.log(`🔥 Started cooking order #${order.id}`);
});

restaurant.on('orderReady', (order) => {
  console.log(`🎉 Order #${order.id} is ready for ${order.customer}!`);
});

// Take some orders
restaurant.takeOrder('Alice', 'Pizza');
restaurant.takeOrder('Bob', 'Burger');
```

### Advanced Event Patterns

Create `advanced-events.js`:
```javascript
// advanced-events.js - Advanced event patterns
const EventEmitter = require('events');

console.log('🚀 Advanced Event Patterns!');

// 1. Event emitter with error handling
console.log('1. Event emitter with error handling:');

class SafeEmitter extends EventEmitter {
  safeEmit(event, ...args) {
    try {
      this.emit(event, ...args);
    } catch (error) {
      console.log('❌ Error in event listener:', error.message);
      this.emit('error', error);
    }
  }
}

const safeEmitter = new SafeEmitter();

// Add error-prone listener
safeEmitter.on('test', (data) => {
  if (data.causeBug) {
    throw new Error('Bug in event listener!');
  }
  console.log('✅ Test event processed:', data.message);
});

// Add error handler
safeEmitter.on('error', (error) => {
  console.log('🛡️ Error handled gracefully:', error.message);
});

// Test safe emission
safeEmitter.safeEmit('test', { message: 'This works!' });
safeEmitter.safeEmit('test', { message: 'This will fail', causeBug: true });

// 2. Event emitter with async listeners
console.log('\n2. Event emitter with async listeners:');

class AsyncEmitter extends EventEmitter {
  async emitAsync(event, ...args) {
    const listeners = this.listeners(event);
    
    for (const listener of listeners) {
      try {
        await listener(...args);
      } catch (error) {
        console.log('❌ Error in async listener:', error.message);
      }
    }
  }
}

const asyncEmitter = new AsyncEmitter();

// Add async listeners
asyncEmitter.on('process', async (data) => {
  console.log('🔄 Processing data:', data.id);
  await new Promise(resolve => setTimeout(resolve, 1000));
  console.log('✅ Processing complete:', data.id);
});

asyncEmitter.on('process', async (data) => {
  console.log('📊 Analyzing data:', data.id);
  await new Promise(resolve => setTimeout(resolve, 500));
  console.log('✅ Analysis complete:', data.id);
});

// Test async emission
async function testAsyncEmission() {
  console.log('🚀 Starting async emission...');
  await asyncEmitter.emitAsync('process', { id: 'task-1' });
  console.log('🎉 All async listeners completed!');
}

testAsyncEmission();

// 3. Event emitter with middleware
console.log('\n3. Event emitter with middleware:');

class MiddlewareEmitter extends EventEmitter {
  constructor() {
    super();
    this.middleware = [];
  }
  
  use(fn) {
    this.middleware.push(fn);
  }
  
  async emitWithMiddleware(event, data) {
    let processedData = data;
    
    // Run middleware
    for (const middleware of this.middleware) {
      try {
        processedData = await middleware(processedData);
        if (processedData === null) {
          console.log('🚫 Event cancelled by middleware');
          return;
        }
      } catch (error) {
        console.log('❌ Middleware error:', error.message);
        return;
      }
    }
    
    // Emit with processed data
    this.emit(event, processedData);
  }
}

const middlewareEmitter = new MiddlewareEmitter();

// Add middleware
middlewareEmitter.use(async (data) => {
  console.log('🔍 Middleware 1: Validating data...');
  if (!data.name) {
    throw new Error('Name is required');
  }
  return data;
});

middlewareEmitter.use(async (data) => {
  console.log('🔧 Middleware 2: Enriching data...');
  return {
    ...data,
    timestamp: new Date().toISOString(),
    processed: true
  };
});

// Add event listener
middlewareEmitter.on('userCreated', (user) => {
  console.log('👤 User created:', user);
});

// Test middleware emission
setTimeout(async () => {
  console.log('🧪 Testing middleware emission...');
  await middlewareEmitter.emitWithMiddleware('userCreated', { name: 'John Doe' });
  
  console.log('🧪 Testing middleware with invalid data...');
  await middlewareEmitter.emitWithMiddleware('userCreated', { age: 25 });
}, 6000);
```

---

## 🎯 Real-World Streaming Application

### Log Processing System

Create `log-processor.js`:
```javascript
// log-processor.js - Real-world log processing system
const { Transform, Writable } = require('stream');
const fs = require('fs');

console.log('📊 Building a log processing system!');

// 1. Create sample log file
console.log('1. Creating sample log file...');

const sampleLogs = [
  '2024-01-01 10:00:00 INFO User login successful: user123',
  '2024-01-01 10:01:00 ERROR Database connection failed',
  '2024-01-01 10:02:00 INFO User logout: user123',
  '2024-01-01 10:03:00 WARN Memory usage high: 85%',
  '2024-01-01 10:04:00 ERROR File not found: /path/to/file.txt',
  '2024-01-01 10:05:00 INFO New user registered: user456',
  '2024-01-01 10:06:00 DEBUG Query executed in 50ms',
  '2024-01-01 10:07:00 ERROR Network timeout',
  '2024-01-01 10:08:00 INFO User login successful: user456',
  '2024-01-01 10:09:00 WARN Disk space low: 15% remaining'
];

fs.writeFileSync('sample.log', sampleLogs.join('\n'));

// 2. Log parser transform
class LogParser extends Transform {
  constructor() {
    super({ objectMode: true });
  }
  
  _transform(chunk, encoding, callback) {
    const lines = chunk.toString().split('\n');
    
    for (const line of lines) {
      if (line.trim()) {
        const logEntry = this.parseLine(line);
        if (logEntry) {
          this.push(logEntry);
        }
      }
    }
    
    callback();
  }
  
  parseLine(line) {
    const logPattern = /^(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) (\w+) (.+)$/;
    const match = line.match(logPattern);
    
    if (match) {
      return {
        timestamp: match[1],
        level: match[2],
        message: match[3],
        raw: line
      };
    }
    
    return null;
  }
}

// 3. Log filter transform
class LogFilter extends Transform {
  constructor(levelFilter) {
    super({ objectMode: true });
    this.levelFilter = levelFilter;
  }
  
  _transform(logEntry, encoding, callback) {
    if (this.levelFilter.includes(logEntry.level)) {
      console.log(`🔍 Filtered: ${logEntry.level} - ${logEntry.message}`);
      this.push(logEntry);
    }
    callback();
  }
}

// 4. Log aggregator
class LogAggregator extends Transform {
  constructor() {
    super({ objectMode: true });
    this.stats = {
      INFO: 0,
      ERROR: 0,
      WARN: 0,
      DEBUG: 0
    };
  }
  
  _transform(logEntry, encoding, callback) {
    this.stats[logEntry.level] = (this.stats[logEntry.level] || 0) + 1;
    this.push(logEntry);
    callback();
  }
  
  _flush(callback) {
    console.log('\n📊 Log Statistics:');
    Object.entries(this.stats).forEach(([level, count]) => {
      console.log(`   ${level}: ${count}`);
    });
    callback();
  }
}

// 5. Alert system
class AlertSystem extends Writable {
  constructor() {
    super({ objectMode: true });
    this.errorCount = 0;
  }
  
  _write(logEntry, encoding, callback) {
    if (logEntry.level === 'ERROR') {
      this.errorCount++;
      console.log(`🚨 ALERT: Error detected - ${logEntry.message}`);
      
      if (this.errorCount >= 3) {
        console.log('🚨 CRITICAL: Multiple errors detected! Notifying admin...');
      }
    }
    
    callback();
  }
}

// 6. Process logs
async function processLogs() {
  console.log('\n2. Processing logs...');
  
  const readStream = fs.createReadStream('sample.log');
  const parser = new LogParser();
  const filter = new LogFilter(['ERROR', 'WARN']); // Only show errors and warnings
  const aggregator = new LogAggregator();
  const alertSystem = new AlertSystem();
  
  // Create processing pipeline
  readStream
    .pipe(parser)
    .pipe(filter)
    .pipe(aggregator)
    .pipe(alertSystem);
  
  alertSystem.on('finish', () => {
    console.log('✅ Log processing completed!');
    
    // Cleanup
    fs.unlinkSync('sample.log');
    console.log('🧹 Cleanup completed!');
  });
}

processLogs();
```

---

## 🎉 Summary

Today you learned:
- ✅ Advanced async patterns (rate limiting, circuit breaker, async queue)
- ✅ What streams are and why they're important (conveyor belt analogy!)
- ✅ Different types of streams (readable, writable, transform)
- ✅ Stream processing pipelines
- ✅ Event emitters and custom events
- ✅ Real-world streaming applications
- ✅ Error handling in async and stream operations
- ✅ Performance optimization techniques

**Next Up**: Day 04 - Part 01: REST API Development and GraphQL Introduction

---

## 🎯 Practice Exercises

### Exercise 1: Data Processing Pipeline
Create a system that:
1. Reads CSV files using streams
2. Validates and transforms data
3. Aggregates statistics
4. Outputs results to multiple formats

### Exercise 2: Real-time Chat System
Build a chat system that:
1. Uses event emitters for messages
2. Implements rate limiting
3. Handles user connections/disconnections
4. Supports multiple rooms

### Exercise 3: File Upload System
Create a system that:
1. Accepts large file uploads using streams
2. Validates file types and sizes
3. Processes files in chunks
4. Provides progress updates

---

*Remember: Streams are like water flowing through pipes - they're powerful, efficient, and once you understand the flow, you can build amazing things! 🌊*
