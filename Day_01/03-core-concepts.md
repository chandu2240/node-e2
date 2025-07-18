# Day 01 - Part 03: Node.js Core Concepts - Modules, Event Loop, and Global Objects

## 🎯 Learning Objectives
By the end of this lesson, you will understand:
- The Node.js module system (CommonJS and ES6)
- How to create and use custom modules
- The Node.js Event Loop in detail
- Global objects and their purposes
- Built-in modules and their uses
- Module resolution and caching
- Best practices for module organization

---

## 🧩 Understanding Modules

### What Are Modules?
Think of modules like **LEGO blocks** - each block has a specific purpose, and you can combine them to build amazing things!

In Node.js, modules are:
- **Reusable pieces of code**
- **Separate files** that export functionality
- **Building blocks** for larger applications
- **Encapsulated code** that doesn't pollute the global scope

### Real-World Analogy
```
Kitchen Analogy:
┌─────────────────────────────────────────────────────┐
│                 Your Kitchen                        │
│                                                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │    Oven     │  │ Refrigerator│  │  Microwave  │ │
│  │  (Module)   │  │  (Module)   │  │  (Module)   │ │
│  └─────────────┘  └─────────────┘  └─────────────┘ │
│                                                     │
│  Each appliance has specific functions,             │
│  but they all work together to help you cook!      │
└─────────────────────────────────────────────────────┘
```

---

## 📦 CommonJS Modules (Traditional Node.js)

### Exporting from a Module

Create `math-utils.js`:
```javascript
// math-utils.js - A utility module for mathematical operations

// Simple function exports
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

function multiply(a, b) {
  return a * b;
}

function divide(a, b) {
  if (b === 0) {
    throw new Error('Division by zero is not allowed!');
  }
  return a / b;
}

// Advanced mathematical functions
const mathOperations = {
  power: (base, exponent) => Math.pow(base, exponent),
  squareRoot: (number) => Math.sqrt(number),
  absolute: (number) => Math.abs(number),
  round: (number, decimals = 0) => {
    const factor = Math.pow(10, decimals);
    return Math.round(number * factor) / factor;
  }
};

// Constants
const PI = Math.PI;
const E = Math.E;

// Export individual functions
module.exports.add = add;
module.exports.subtract = subtract;
module.exports.multiply = multiply;
module.exports.divide = divide;

// Export objects and constants
module.exports.advanced = mathOperations;
module.exports.constants = { PI, E };

// Alternative export syntax
module.exports.factorial = function(n) {
  if (n < 0) return undefined;
  if (n === 0 || n === 1) return 1;
  
  let result = 1;
  for (let i = 2; i <= n; i++) {
    result *= i;
  }
  return result;
};

// Export metadata
module.exports.info = {
  version: '1.0.0',
  author: 'Node.js Student',
  description: 'Basic mathematical operations'
};
```

### Importing and Using Modules

Create `app.js`:
```javascript
// app.js - Main application file

// Import the entire module
const mathUtils = require('./math-utils');

// Import specific functions using destructuring
const { add, subtract, multiply, divide } = require('./math-utils');

console.log('=== Math Utils Demo ===');

// Using individual functions
console.log('Addition: 5 + 3 =', add(5, 3));
console.log('Subtraction: 10 - 4 =', subtract(10, 4));
console.log('Multiplication: 6 * 7 =', multiply(6, 7));
console.log('Division: 15 / 3 =', divide(15, 3));

// Using advanced operations
console.log('\n=== Advanced Operations ===');
console.log('Power: 2^8 =', mathUtils.advanced.power(2, 8));
console.log('Square Root of 16 =', mathUtils.advanced.squareRoot(16));
console.log('Absolute value of -42 =', mathUtils.advanced.absolute(-42));
console.log('Round 3.14159 to 2 decimals =', mathUtils.advanced.round(3.14159, 2));

// Using constants
console.log('\n=== Constants ===');
console.log('PI =', mathUtils.constants.PI);
console.log('E =', mathUtils.constants.E);

// Using factorial
console.log('\n=== Factorial ===');
console.log('5! =', mathUtils.factorial(5));
console.log('0! =', mathUtils.factorial(0));

// Module information
console.log('\n=== Module Info ===');
console.log('Version:', mathUtils.info.version);
console.log('Author:', mathUtils.info.author);
console.log('Description:', mathUtils.info.description);
```

Run the application:
```bash
node app.js
```

---

## 🎯 Different Export Patterns

### Pattern 1: Single Function Export

Create `greeting.js`:
```javascript
// greeting.js - Single function export

function createGreeting(name, timeOfDay = 'day') {
  const greetings = {
    morning: 'Good morning',
    afternoon: 'Good afternoon',
    evening: 'Good evening',
    night: 'Good night',
    day: 'Hello'
  };
  
  const greeting = greetings[timeOfDay] || greetings.day;
  return `${greeting}, ${name}! Welcome to Node.js!`;
}

// Export single function
module.exports = createGreeting;
```

### Pattern 2: Object Export

Create `user-manager.js`:
```javascript
// user-manager.js - Object-based export

const users = [];
let nextId = 1;

const userManager = {
  // Add a new user
  addUser(name, email) {
    const user = {
      id: nextId++,
      name,
      email,
      createdAt: new Date(),
      isActive: true
    };
    users.push(user);
    return user;
  },
  
  // Get all users
  getAllUsers() {
    return [...users]; // Return a copy
  },
  
  // Find user by ID
  findUserById(id) {
    return users.find(user => user.id === id);
  },
  
  // Find user by email
  findUserByEmail(email) {
    return users.find(user => user.email === email);
  },
  
  // Update user
  updateUser(id, updates) {
    const user = this.findUserById(id);
    if (user) {
      Object.assign(user, updates, { updatedAt: new Date() });
      return user;
    }
    return null;
  },
  
  // Delete user
  deleteUser(id) {
    const index = users.findIndex(user => user.id === id);
    if (index !== -1) {
      return users.splice(index, 1)[0];
    }
    return null;
  },
  
  // Get user statistics
  getStats() {
    return {
      totalUsers: users.length,
      activeUsers: users.filter(user => user.isActive).length,
      inactiveUsers: users.filter(user => !user.isActive).length
    };
  }
};

module.exports = userManager;
```

### Pattern 3: Class Export

Create `calculator.js`:
```javascript
// calculator.js - Class-based export

class Calculator {
  constructor() {
    this.history = [];
    this.memory = 0;
  }
  
  // Basic operations
  add(a, b) {
    const result = a + b;
    this.addToHistory('add', a, b, result);
    return result;
  }
  
  subtract(a, b) {
    const result = a - b;
    this.addToHistory('subtract', a, b, result);
    return result;
  }
  
  multiply(a, b) {
    const result = a * b;
    this.addToHistory('multiply', a, b, result);
    return result;
  }
  
  divide(a, b) {
    if (b === 0) {
      throw new Error('Division by zero!');
    }
    const result = a / b;
    this.addToHistory('divide', a, b, result);
    return result;
  }
  
  // Memory operations
  memoryStore(value) {
    this.memory = value;
    console.log(`Memory stored: ${value}`);
  }
  
  memoryRecall() {
    return this.memory;
  }
  
  memoryClear() {
    this.memory = 0;
    console.log('Memory cleared');
  }
  
  // History operations
  addToHistory(operation, a, b, result) {
    this.history.push({
      operation,
      operands: [a, b],
      result,
      timestamp: new Date()
    });
  }
  
  getHistory() {
    return [...this.history];
  }
  
  clearHistory() {
    this.history = [];
    console.log('History cleared');
  }
  
  // Display last calculation
  getLastCalculation() {
    return this.history[this.history.length - 1] || null;
  }
}

module.exports = Calculator;
```

### Using Different Export Patterns

Create `module-patterns-demo.js`:
```javascript
// module-patterns-demo.js - Demonstrating different module patterns

// Pattern 1: Single function import
const createGreeting = require('./greeting');

// Pattern 2: Object import
const userManager = require('./user-manager');

// Pattern 3: Class import
const Calculator = require('./calculator');

console.log('=== Module Patterns Demo ===\n');

// 1. Single function usage
console.log('1. Single Function Module:');
console.log(createGreeting('Alice', 'morning'));
console.log(createGreeting('Bob', 'evening'));
console.log(createGreeting('Charlie'));

// 2. Object module usage
console.log('\n2. Object Module:');
const user1 = userManager.addUser('John Doe', 'john@example.com');
const user2 = userManager.addUser('Jane Smith', 'jane@example.com');

console.log('Added users:', userManager.getAllUsers());
console.log('User stats:', userManager.getStats());

const foundUser = userManager.findUserByEmail('john@example.com');
console.log('Found user by email:', foundUser);

// 3. Class module usage
console.log('\n3. Class Module:');
const calc = new Calculator();

console.log('Addition: 10 + 5 =', calc.add(10, 5));
console.log('Multiplication: 7 * 8 =', calc.multiply(7, 8));
console.log('Division: 20 / 4 =', calc.divide(20, 4));

// Store in memory
calc.memoryStore(100);
console.log('Memory recall:', calc.memoryRecall());

// Show history
console.log('\nCalculation history:');
calc.getHistory().forEach((entry, index) => {
  console.log(`${index + 1}. ${entry.operation}: ${entry.operands[0]} ${entry.operation} ${entry.operands[1]} = ${entry.result}`);
});
```

---

## 🌐 ES6 Modules (Modern JavaScript)

### To use ES6 modules in Node.js, update package.json:
```json
{
  "type": "module",
  "name": "my-first-node-app",
  "version": "1.0.0"
}
```

### ES6 Export Syntax

Create `es6-math.js`:
```javascript
// es6-math.js - ES6 module syntax

// Named exports
export const PI = Math.PI;
export const E = Math.E;

export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

// Export object
export const constants = {
  GOLDEN_RATIO: 1.618033988749,
  SQRT_2: Math.sqrt(2)
};

// Default export
export default class MathHelper {
  static factorial(n) {
    if (n < 0) return undefined;
    if (n === 0 || n === 1) return 1;
    return n * this.factorial(n - 1);
  }
  
  static fibonacci(n) {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }
  
  static isPrime(n) {
    if (n <= 1) return false;
    if (n <= 3) return true;
    if (n % 2 === 0 || n % 3 === 0) return false;
    
    for (let i = 5; i * i <= n; i += 6) {
      if (n % i === 0 || n % (i + 2) === 0) return false;
    }
    return true;
  }
}
```

### ES6 Import Syntax

Create `es6-demo.js`:
```javascript
// es6-demo.js - ES6 import syntax

// Named imports
import { add, subtract, PI, E, constants } from './es6-math.js';

// Default import
import MathHelper from './es6-math.js';

// Import all
import * as MathUtils from './es6-math.js';

console.log('=== ES6 Modules Demo ===\n');

// Using named imports
console.log('Addition: 15 + 25 =', add(15, 25));
console.log('Subtraction: 100 - 30 =', subtract(100, 30));
console.log('PI =', PI);
console.log('E =', E);
console.log('Golden Ratio =', constants.GOLDEN_RATIO);

// Using default import
console.log('\nFactorial of 6 =', MathHelper.factorial(6));
console.log('Fibonacci sequence (first 10):');
for (let i = 0; i < 10; i++) {
  console.log(`F(${i}) = ${MathHelper.fibonacci(i)}`);
}

console.log('\nPrime numbers from 1 to 20:');
for (let i = 1; i <= 20; i++) {
  if (MathHelper.isPrime(i)) {
    console.log(i);
  }
}

// Using import all
console.log('\nUsing import * syntax:');
console.log('Addition via namespace:', MathUtils.add(5, 3));
console.log('Default class via namespace:', MathUtils.default.factorial(5));
```

---

## 🔄 The Node.js Event Loop

### Understanding the Event Loop
The Event Loop is the **heart** of Node.js - it's what makes Node.js non-blocking and efficient.

### Event Loop Phases:
```
┌───────────────────────────┐
┌─>│           timers          │  ← setTimeout, setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  ← I/O callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  ← Internal use
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           poll            │  ← Fetching new I/O events
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │  ← setImmediate callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │  ← Close events
   └───────────────────────────┘
```

### Event Loop Demo

Create `event-loop-demo.js`:
```javascript
// event-loop-demo.js - Understanding the event loop

console.log('=== Event Loop Demo ===\n');

// 1. Synchronous code (executed immediately)
console.log('1. Start of script (synchronous)');

// 2. setTimeout (timers phase)
setTimeout(() => {
  console.log('4. setTimeout 0ms (timers phase)');
}, 0);

setTimeout(() => {
  console.log('6. setTimeout 100ms (timers phase)');
}, 100);

// 3. setImmediate (check phase)
setImmediate(() => {
  console.log('5. setImmediate (check phase)');
});

// 4. Process.nextTick (executed before each phase)
process.nextTick(() => {
  console.log('3. process.nextTick (highest priority)');
});

// 5. Promise (microtask queue)
Promise.resolve().then(() => {
  console.log('3.5. Promise resolved (microtask)');
});

// 6. More synchronous code
console.log('2. End of script (synchronous)');

// 7. Demonstrate non-blocking I/O
const fs = require('fs');
console.log('7. Starting file read (non-blocking)');

fs.readFile('./package.json', 'utf8', (err, data) => {
  if (err) {
    console.log('8. File read error:', err.message);
  } else {
    console.log('8. File read completed (I/O callback)');
  }
});

console.log('9. After file read call (synchronous)');
```

### Event Loop Order Demo

Create `event-order-demo.js`:
```javascript
// event-order-demo.js - Event loop execution order

console.log('=== Event Loop Order Demo ===\n');

// This demonstrates the order of execution
let order = 1;

// Synchronous
console.log(`${order++}. Synchronous code`);

// Microtasks (highest priority after synchronous)
Promise.resolve().then(() => {
  console.log(`${order++}. Promise 1`);
});

Promise.resolve().then(() => {
  console.log(`${order++}. Promise 2`);
});

// Process.nextTick (even higher priority than Promises)
process.nextTick(() => {
  console.log(`${order++}. nextTick 1`);
});

process.nextTick(() => {
  console.log(`${order++}. nextTick 2`);
});

// Timers
setTimeout(() => {
  console.log(`${order++}. setTimeout 0`);
}, 0);

// Immediate
setImmediate(() => {
  console.log(`${order++}. setImmediate 1`);
});

setImmediate(() => {
  console.log(`${order++}. setImmediate 2`);
});

// More synchronous
console.log(`${order++}. More synchronous code`);

// Nested event loop behavior
setTimeout(() => {
  console.log(`${order++}. setTimeout with nested operations`);
  
  process.nextTick(() => {
    console.log(`${order++}. nextTick inside setTimeout`);
  });
  
  setImmediate(() => {
    console.log(`${order++}. setImmediate inside setTimeout`);
  });
}, 0);
```

---

## 🌍 Global Objects in Node.js

### Node.js Global Objects

Create `globals-demo.js`:
```javascript
// globals-demo.js - Node.js global objects

console.log('=== Node.js Global Objects Demo ===\n');

// 1. console - Global logging object
console.log('1. console.log() - Basic logging');
console.error('2. console.error() - Error logging');
console.warn('3. console.warn() - Warning logging');
console.info('4. console.info() - Info logging');
console.debug('5. console.debug() - Debug logging');

// 2. process - Current Node.js process
console.log('\n=== Process Object ===');
console.log('Node.js version:', process.version);
console.log('Platform:', process.platform);
console.log('Architecture:', process.arch);
console.log('Process ID:', process.pid);
console.log('Current working directory:', process.cwd());
console.log('Memory usage:', process.memoryUsage());
console.log('CPU usage:', process.cpuUsage());

// 3. global - Global namespace
console.log('\n=== Global Namespace ===');
global.myGlobalVariable = 'Hello from global scope!';
console.log('Global variable:', global.myGlobalVariable);

// 4. Buffer - Binary data handling
console.log('\n=== Buffer ===');
const buffer1 = Buffer.from('Hello World', 'utf8');
console.log('Buffer from string:', buffer1);
console.log('Buffer to string:', buffer1.toString());
console.log('Buffer length:', buffer1.length);

const buffer2 = Buffer.alloc(10, 'A');
console.log('Allocated buffer:', buffer2);
console.log('Buffer to string:', buffer2.toString());

// 5. __dirname and __filename (CommonJS only)
console.log('\n=== File/Directory Paths ===');
console.log('Current file:', __filename);
console.log('Current directory:', __dirname);

// 6. setTimeout/setInterval/setImmediate
console.log('\n=== Timers ===');
const timeoutId = setTimeout(() => {
  console.log('Timeout executed after 1 second');
}, 1000);

const intervalId = setInterval(() => {
  console.log('Interval tick');
}, 500);

// Clear interval after 3 seconds
setTimeout(() => {
  clearInterval(intervalId);
  console.log('Interval cleared');
}, 3000);

setImmediate(() => {
  console.log('Immediate executed');
});

// 7. require (CommonJS)
console.log('\n=== Module System ===');
console.log('require function:', typeof require);
console.log('require.cache keys:', Object.keys(require.cache).length);

// 8. module and exports
console.log('Current module:', module.filename);
console.log('Module exports:', typeof module.exports);

// 9. URL and URLSearchParams (Global in newer Node.js versions)
console.log('\n=== URL Globals ===');
const url = new URL('https://example.com/path?name=John&age=30');
console.log('URL object:', url.toString());
console.log('URL hostname:', url.hostname);
console.log('URL search params:', url.searchParams.get('name'));
```

### Process Events and Signals

Create `process-events-demo.js`:
```javascript
// process-events-demo.js - Process events and signals

console.log('=== Process Events Demo ===');
console.log('Process ID:', process.pid);
console.log('Press Ctrl+C to test SIGINT handling\n');

// 1. Exit event
process.on('exit', (code) => {
  console.log(`\nProcess exiting with code: ${code}`);
  // Synchronous cleanup only
});

// 2. SIGINT (Ctrl+C)
process.on('SIGINT', () => {
  console.log('\nReceived SIGINT (Ctrl+C)');
  console.log('Performing graceful shutdown...');
  
  // Simulate cleanup
  setTimeout(() => {
    console.log('Cleanup completed');
    process.exit(0);
  }, 1000);
});

// 3. SIGTERM (Termination signal)
process.on('SIGTERM', () => {
  console.log('\nReceived SIGTERM');
  console.log('Shutting down gracefully...');
  process.exit(0);
});

// 4. Uncaught Exception
process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error.message);
  console.error('Stack trace:', error.stack);
  process.exit(1);
});

// 5. Unhandled Promise Rejection
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Promise Rejection at:', promise);
  console.error('Reason:', reason);
  process.exit(1);
});

// 6. Warning events
process.on('warning', (warning) => {
  console.warn('Warning:', warning.name);
  console.warn('Message:', warning.message);
  console.warn('Stack:', warning.stack);
});

// Keep the process running
console.log('Process running... (Press Ctrl+C to exit)');
setInterval(() => {
  console.log('Process is alive at', new Date().toISOString());
}, 5000);

// Simulate some work
setTimeout(() => {
  console.log('Doing some work...');
}, 2000);
```

---

## 📚 Built-in Modules Overview

### Core Module Categories

Create `builtin-modules-demo.js`:
```javascript
// builtin-modules-demo.js - Overview of built-in modules

console.log('=== Built-in Modules Demo ===\n');

// 1. File System
const fs = require('fs');
const path = require('path');

console.log('1. File System Module:');
console.log('Current directory files:');
try {
  const files = fs.readdirSync('.');
  files.forEach(file => {
    const stats = fs.statSync(file);
    console.log(`  ${file} (${stats.isDirectory() ? 'DIR' : 'FILE'})`);
  });
} catch (error) {
  console.error('Error reading directory:', error.message);
}

// 2. Path Module
console.log('\n2. Path Module:');
const filePath = '/users/john/documents/file.txt';
console.log('Full path:', filePath);
console.log('Directory name:', path.dirname(filePath));
console.log('Base name:', path.basename(filePath));
console.log('Extension:', path.extname(filePath));
console.log('Parsed path:', path.parse(filePath));

// 3. Operating System
const os = require('os');
console.log('\n3. Operating System Module:');
console.log('Platform:', os.platform());
console.log('Architecture:', os.arch());
console.log('CPU cores:', os.cpus().length);
console.log('Total memory:', Math.round(os.totalmem() / 1024 / 1024 / 1024) + ' GB');
console.log('Free memory:', Math.round(os.freemem() / 1024 / 1024 / 1024) + ' GB');
console.log('Home directory:', os.homedir());
console.log('Temp directory:', os.tmpdir());

// 4. Utilities
const util = require('util');
console.log('\n4. Utilities Module:');
const obj = { name: 'John', age: 30, hobbies: ['reading', 'coding'] };
console.log('Object inspection:', util.inspect(obj, { colors: true }));
console.log('Is Array:', util.isArray(obj.hobbies));

// 5. URL Module
const url = require('url');
console.log('\n5. URL Module:');
const urlString = 'https://example.com:8080/path/to/resource?name=John&age=30#section1';
const parsedUrl = url.parse(urlString, true);
console.log('Parsed URL:', parsedUrl);
console.log('Host:', parsedUrl.host);
console.log('Query parameters:', parsedUrl.query);

// 6. Query String
const querystring = require('querystring');
console.log('\n6. Query String Module:');
const qs = 'name=John&age=30&hobbies=reading&hobbies=coding';
console.log('Query string:', qs);
console.log('Parsed:', querystring.parse(qs));

// 7. Crypto (for basic operations)
const crypto = require('crypto');
console.log('\n7. Crypto Module:');
const hash = crypto.createHash('sha256').update('Hello World').digest('hex');
console.log('SHA256 hash of "Hello World":', hash);

const randomBytes = crypto.randomBytes(16).toString('hex');
console.log('Random bytes:', randomBytes);

// 8. Events
const EventEmitter = require('events');
console.log('\n8. Events Module:');
class MyEmitter extends EventEmitter {}
const myEmitter = new MyEmitter();

myEmitter.on('event', (message) => {
  console.log('Event received:', message);
});

myEmitter.emit('event', 'Hello from event emitter!');
```

---

## 🔧 Module Resolution and Caching

### Module Resolution Demo

Create `module-resolution-demo.js`:
```javascript
// module-resolution-demo.js - How Node.js resolves modules

console.log('=== Module Resolution Demo ===\n');

// 1. Module search paths
console.log('1. Module search paths:');
console.log('require.resolve.paths("express"):');
try {
  const paths = require.resolve.paths('express');
  paths.forEach((path, index) => {
    console.log(`  ${index + 1}. ${path}`);
  });
} catch (error) {
  console.log('  Express not found in search paths');
}

// 2. Resolving module paths
console.log('\n2. Module resolution:');
try {
  console.log('Path module location:', require.resolve('path'));
  console.log('FS module location:', require.resolve('fs'));
} catch (error) {
  console.error('Error resolving modules:', error.message);
}

// 3. Module cache
console.log('\n3. Module cache:');
console.log('Number of cached modules:', Object.keys(require.cache).length);

// Show some cached modules
const cachedModules = Object.keys(require.cache).slice(0, 5);
console.log('First 5 cached modules:');
cachedModules.forEach((module, index) => {
  console.log(`  ${index + 1}. ${module}`);
});

// 4. Creating a module that demonstrates caching
const moduleA = require('./math-utils');
const moduleB = require('./math-utils'); // Same module, from cache

console.log('\n4. Module caching:');
console.log('Module A === Module B:', moduleA === moduleB);
console.log('Both references point to the same cached module');

// 5. Module wrapper
console.log('\n5. Module wrapper:');
console.log('Every module is wrapped in a function like this:');
console.log(`
(function(exports, require, module, __filename, __dirname) {
  // Your module code here
});
`);
```

### Custom Module with Caching Demo

Create `counter-module.js`:
```javascript
// counter-module.js - Demonstrates module caching

console.log('Counter module loaded!');

let count = 0;

function increment() {
  count++;
  console.log(`Count incremented to: ${count}`);
  return count;
}

function decrement() {
  count--;
  console.log(`Count decremented to: ${count}`);
  return count;
}

function getCount() {
  return count;
}

function reset() {
  count = 0;
  console.log('Count reset to 0');
  return count;
}

module.exports = {
  increment,
  decrement,
  getCount,
  reset
};
```

Create `caching-demo.js`:
```javascript
// caching-demo.js - Demonstrates how modules are cached

console.log('=== Module Caching Demo ===\n');

// First import
console.log('1. First import:');
const counter1 = require('./counter-module');
counter1.increment();
counter1.increment();
console.log('Counter 1 value:', counter1.getCount());

// Second import (same module, cached)
console.log('\n2. Second import (cached):');
const counter2 = require('./counter-module');
console.log('Counter 2 value:', counter2.getCount());
counter2.increment();
console.log('Counter 1 value after counter2 increment:', counter1.getCount());

// They're the same object!
console.log('\n3. Are they the same object?', counter1 === counter2);

// Clear cache and reload
console.log('\n4. Clearing cache and reloading:');
delete require.cache[require.resolve('./counter-module')];
const counter3 = require('./counter-module');
console.log('Counter 3 value (fresh load):', counter3.getCount());
```

---

## 🎯 Best Practices for Modules

### 1. Module Organization

```javascript
// Good module structure
// user-service.js
const validator = require('./validators/user-validator');
const database = require('./database/connection');

class UserService {
  constructor() {
    this.users = [];
  }
  
  async createUser(userData) {
    // Validate input
    const validationResult = validator.validateUser(userData);
    if (!validationResult.isValid) {
      throw new Error(validationResult.errors.join(', '));
    }
    
    // Save to database
    const user = await database.users.create(userData);
    return user;
  }
}

module.exports = UserService;
```

### 2. Error Handling in Modules

```javascript
// error-handling-module.js
const fs = require('fs').promises;

async function readConfigFile(filePath) {
  try {
    const data = await fs.readFile(filePath, 'utf8');
    return JSON.parse(data);
  } catch (error) {
    if (error.code === 'ENOENT') {
      throw new Error(`Config file not found: ${filePath}`);
    } else if (error instanceof SyntaxError) {
      throw new Error(`Invalid JSON in config file: ${filePath}`);
    } else {
      throw error; // Re-throw unexpected errors
    }
  }
}

module.exports = { readConfigFile };
```

### 3. Module Dependencies

```javascript
// dependency-injection-example.js
class EmailService {
  constructor(emailProvider) {
    this.emailProvider = emailProvider;
  }
  
  async sendEmail(to, subject, body) {
    return await this.emailProvider.send(to, subject, body);
  }
}

// Factory function
function createEmailService(providerType = 'default') {
  const providers = {
    default: require('./providers/default-email'),
    gmail: require('./providers/gmail-provider'),
    sendgrid: require('./providers/sendgrid-provider')
  };
  
  const provider = providers[providerType];
  if (!provider) {
    throw new Error(`Unknown email provider: ${providerType}`);
  }
  
  return new EmailService(provider);
}

module.exports = { EmailService, createEmailService };
```

---

## 🎉 Summary

Today you learned:
- ✅ CommonJS and ES6 module systems
- ✅ Different export and import patterns
- ✅ The Node.js Event Loop and execution order
- ✅ Global objects and their purposes
- ✅ Built-in modules overview
- ✅ Module resolution and caching
- ✅ Best practices for module organization

**Next Up**: Day 02 - Part 01: File System Module Deep Dive

---

## 📚 Additional Resources

- [Node.js Module Documentation](https://nodejs.org/api/modules.html)
- [Event Loop Explained](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
- [CommonJS vs ES6 Modules](https://nodejs.org/api/esm.html)
- [Module Resolution Algorithm](https://nodejs.org/api/modules.html#modules_all_together)

---

*Understanding modules is like learning to organize your toolbox - once you master it, building complex applications becomes much easier!* 🔧
