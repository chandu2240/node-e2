# Day 02 - Part 01: File System Module Deep Dive

## 🎯 Learning Objectives
By the end of this lesson, you will understand:
- File System (fs) module fundamentals
- Synchronous vs Asynchronous file operations
- Reading and writing files in multiple ways
- Working with directories and file metadata
- File streams for large files
- File permissions and security
- Error handling in file operations
- Best practices for file system operations

---

## 📁 Introduction to File System Module

### What is the File System Module?
The File System (fs) module is Node.js's built-in module that allows you to interact with the file system on your computer. Think of it as your **digital assistant** that can:
- Read files (like opening a book)
- Write files (like writing in a notebook)
- Create folders (like organizing your desk)
- Delete files (like throwing away papers)
- Get file information (like checking file size)

### Real-World Analogy
```
File System Module = Digital File Cabinet
┌─────────────────────────────────────────────────────┐
│                File Cabinet                         │
│                                                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │   Drawer 1  │  │   Drawer 2  │  │   Drawer 3  │ │
│  │   (Folder)  │  │   (Folder)  │  │   (Folder)  │ │
│  │             │  │             │  │             │ │
│  │ 📄 File1.txt │  │ 📄 File2.js │  │ 📄 File3.md │ │
│  │ 📄 File2.txt │  │ 📄 File3.js │  │ 📄 File4.md │ │
│  └─────────────┘  └─────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────┘
```

---

## 🔄 Synchronous vs Asynchronous Operations

### Understanding the Difference

#### Synchronous (Blocking)
```javascript
// Synchronous - Waits for completion
console.log('Starting...');
const data = fs.readFileSync('file.txt', 'utf8');
console.log('File read:', data);
console.log('Finished!');
```

#### Asynchronous (Non-Blocking)
```javascript
// Asynchronous - Doesn't wait
console.log('Starting...');
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log('File read:', data);
});
console.log('Finished!');
```

### When to Use Each

| Use Synchronous When | Use Asynchronous When |
|---------------------|----------------------|
| Application startup | During normal operation |
| Configuration loading | User requests |
| Simple scripts | Web servers |
| One-time operations | Multiple concurrent operations |

---

## 📖 Reading Files

### Method 1: Basic File Reading

Create `read-file-demo.js`:
```javascript
// read-file-demo.js - Various ways to read files
const fs = require('fs');
const path = require('path');

console.log('=== File Reading Demo ===\n');

// First, let's create a sample file to read
const sampleContent = `Hello, Node.js!
This is a sample file for reading demonstration.
Line 3: Current time is ${new Date().toISOString()}
Line 4: This file contains multiple lines.
Line 5: Perfect for testing file operations!`;

const sampleFile = 'sample.txt';

// Create the sample file
fs.writeFileSync(sampleFile, sampleContent);
console.log('Sample file created:', sampleFile);

// 1. Synchronous reading
console.log('\n1. Synchronous Reading:');
try {
  const data = fs.readFileSync(sampleFile, 'utf8');
  console.log('File content:');
  console.log(data);
} catch (error) {
  console.error('Error reading file:', error.message);
}

// 2. Asynchronous reading with callback
console.log('\n2. Asynchronous Reading (Callback):');
fs.readFile(sampleFile, 'utf8', (err, data) => {
  if (err) {
    console.error('Error reading file:', err.message);
    return;
  }
  console.log('File content (async):');
  console.log(data);
});

// 3. Asynchronous reading with Promises
console.log('\n3. Asynchronous Reading (Promises):');
const fsPromises = require('fs').promises;

fsPromises.readFile(sampleFile, 'utf8')
  .then(data => {
    console.log('File content (Promise):');
    console.log(data);
  })
  .catch(error => {
    console.error('Error reading file:', error.message);
  });

// 4. Asynchronous reading with async/await
console.log('\n4. Asynchronous Reading (async/await):');
async function readFileAsync() {
  try {
    const data = await fsPromises.readFile(sampleFile, 'utf8');
    console.log('File content (async/await):');
    console.log(data);
  } catch (error) {
    console.error('Error reading file:', error.message);
  }
}

readFileAsync();
```

### Method 2: Reading Different File Types

Create `read-different-types.js`:
```javascript
// read-different-types.js - Reading different file types
const fs = require('fs');

console.log('=== Reading Different File Types ===\n');

// 1. Reading as Buffer (binary data)
console.log('1. Reading as Buffer:');
fs.readFile('sample.txt', (err, buffer) => {
  if (err) {
    console.error('Error:', err.message);
    return;
  }
  console.log('Buffer:', buffer);
  console.log('Buffer length:', buffer.length);
  console.log('Buffer to string:', buffer.toString());
});

// 2. Reading JSON files
console.log('\n2. Reading JSON files:');
const jsonData = {
  name: 'John Doe',
  age: 30,
  city: 'New York',
  hobbies: ['reading', 'coding', 'gaming'],
  isActive: true
};

// Create JSON file
fs.writeFileSync('data.json', JSON.stringify(jsonData, null, 2));

// Read JSON file
fs.readFile('data.json', 'utf8', (err, data) => {
  if (err) {
    console.error('Error reading JSON:', err.message);
    return;
  }
  
  try {
    const parsedData = JSON.parse(data);
    console.log('Parsed JSON data:');
    console.log(parsedData);
    console.log('Name:', parsedData.name);
    console.log('Hobbies:', parsedData.hobbies);
  } catch (parseError) {
    console.error('Error parsing JSON:', parseError.message);
  }
});

// 3. Reading with specific encoding
console.log('\n3. Reading with different encodings:');
const encodings = ['utf8', 'ascii', 'base64', 'hex'];

encodings.forEach(encoding => {
  fs.readFile('sample.txt', encoding, (err, data) => {
    if (err) {
      console.error(`Error with ${encoding}:`, err.message);
      return;
    }
    console.log(`${encoding.toUpperCase()}:`, data.substring(0, 50) + '...');
  });
});
```

### Method 3: Reading Large Files Line by Line

Create `read-large-files.js`:
```javascript
// read-large-files.js - Reading large files efficiently
const fs = require('fs');
const readline = require('readline');

console.log('=== Reading Large Files ===\n');

// Create a large sample file
const largeContent = [];
for (let i = 1; i <= 1000; i++) {
  largeContent.push(`Line ${i}: This is line number ${i} with some sample content.`);
}

fs.writeFileSync('large-file.txt', largeContent.join('\n'));
console.log('Large file created with 1000 lines');

// Method 1: Reading line by line using readline
console.log('\n1. Reading line by line:');
async function readLineByLine() {
  const fileStream = fs.createReadStream('large-file.txt');
  
  const rl = readline.createInterface({
    input: fileStream,
    crlfDelay: Infinity // Handle Windows line endings
  });
  
  let lineNumber = 0;
  for await (const line of rl) {
    lineNumber++;
    if (lineNumber <= 5 || lineNumber % 100 === 0) {
      console.log(`Line ${lineNumber}: ${line}`);
    }
  }
  
  console.log(`Total lines processed: ${lineNumber}`);
}

readLineByLine();

// Method 2: Reading chunks
console.log('\n2. Reading in chunks:');
const stream = fs.createReadStream('large-file.txt', { 
  encoding: 'utf8',
  highWaterMark: 1024 // 1KB chunks
});

let chunkCount = 0;
stream.on('data', (chunk) => {
  chunkCount++;
  console.log(`Chunk ${chunkCount}: ${chunk.length} bytes`);
});

stream.on('end', () => {
  console.log(`File reading completed. Total chunks: ${chunkCount}`);
});

stream.on('error', (error) => {
  console.error('Error reading file:', error.message);
});
```

---

## ✏️ Writing Files

### Method 1: Basic File Writing

Create `write-file-demo.js`:
```javascript
// write-file-demo.js - Various ways to write files
const fs = require('fs');

console.log('=== File Writing Demo ===\n');

// 1. Synchronous writing
console.log('1. Synchronous Writing:');
const syncContent = `Synchronous Content
Written at: ${new Date().toISOString()}
This file was created synchronously.`;

try {
  fs.writeFileSync('sync-file.txt', syncContent);
  console.log('Synchronous file created successfully');
} catch (error) {
  console.error('Error writing sync file:', error.message);
}

// 2. Asynchronous writing with callback
console.log('\n2. Asynchronous Writing (Callback):');
const asyncContent = `Asynchronous Content
Written at: ${new Date().toISOString()}
This file was created asynchronously with callback.`;

fs.writeFile('async-file.txt', asyncContent, (err) => {
  if (err) {
    console.error('Error writing async file:', err.message);
    return;
  }
  console.log('Asynchronous file created successfully');
});

// 3. Asynchronous writing with Promises
console.log('\n3. Asynchronous Writing (Promises):');
const fsPromises = require('fs').promises;
const promiseContent = `Promise Content
Written at: ${new Date().toISOString()}
This file was created using Promises.`;

fsPromises.writeFile('promise-file.txt', promiseContent)
  .then(() => {
    console.log('Promise file created successfully');
  })
  .catch(error => {
    console.error('Error writing promise file:', error.message);
  });

// 4. Asynchronous writing with async/await
console.log('\n4. Asynchronous Writing (async/await):');
async function writeFileAsync() {
  const awaitContent = `Async/Await Content
Written at: ${new Date().toISOString()}
This file was created using async/await.`;

  try {
    await fsPromises.writeFile('await-file.txt', awaitContent);
    console.log('Async/await file created successfully');
  } catch (error) {
    console.error('Error writing await file:', error.message);
  }
}

writeFileAsync();

// 5. Writing with different options
console.log('\n5. Writing with options:');
const optionsContent = 'File with custom options';

fs.writeFile('options-file.txt', optionsContent, {
  encoding: 'utf8',
  flag: 'w',
  mode: 0o666
}, (err) => {
  if (err) {
    console.error('Error writing options file:', err.message);
    return;
  }
  console.log('Options file created successfully');
});
```

### Method 2: Appending to Files

Create `append-file-demo.js`:
```javascript
// append-file-demo.js - Appending content to files
const fs = require('fs');

console.log('=== File Appending Demo ===\n');

const logFile = 'app.log';

// 1. Create initial log file
const initialContent = `Application Log
Started at: ${new Date().toISOString()}
=====================================
`;

fs.writeFileSync(logFile, initialContent);
console.log('Initial log file created');

// 2. Append using appendFile
function logMessage(message) {
  const timestamp = new Date().toISOString();
  const logEntry = `[${timestamp}] ${message}\n`;
  
  fs.appendFile(logFile, logEntry, (err) => {
    if (err) {
      console.error('Error appending to log:', err.message);
      return;
    }
    console.log('Log entry added:', message);
  });
}

// 3. Append synchronously
function logMessageSync(message) {
  const timestamp = new Date().toISOString();
  const logEntry = `[${timestamp}] ${message}\n`;
  
  try {
    fs.appendFileSync(logFile, logEntry);
    console.log('Sync log entry added:', message);
  } catch (error) {
    console.error('Error appending sync log:', error.message);
  }
}

// 4. Append with async/await
async function logMessageAsync(message) {
  const timestamp = new Date().toISOString();
  const logEntry = `[${timestamp}] ${message}\n`;
  
  try {
    await fs.promises.appendFile(logFile, logEntry);
    console.log('Async log entry added:', message);
  } catch (error) {
    console.error('Error appending async log:', error.message);
  }
}

// Demo the logging functions
logMessage('Application started');
logMessageSync('User logged in');
logMessageAsync('Database connected');
logMessage('Processing user request');
logMessageSync('Request completed');

// 5. Alternative: Using writeFile with 'a' flag
setTimeout(() => {
  const finalEntry = `[${new Date().toISOString()}] Application shutting down\n`;
  fs.writeFile(logFile, finalEntry, { flag: 'a' }, (err) => {
    if (err) {
      console.error('Error writing final entry:', err.message);
      return;
    }
    console.log('Final entry added using writeFile with append flag');
    
    // Read the entire log file
    fs.readFile(logFile, 'utf8', (err, data) => {
      if (err) {
        console.error('Error reading log file:', err.message);
        return;
      }
      console.log('\nComplete log file:');
      console.log(data);
    });
  });
}, 1000);
```

### Method 3: Writing JSON and Structured Data

Create `write-json-demo.js`:
```javascript
// write-json-demo.js - Writing JSON and structured data
const fs = require('fs');

console.log('=== JSON Writing Demo ===\n');

// 1. Writing JSON data
const userData = {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  age: 30,
  address: {
    street: '123 Main St',
    city: 'New York',
    zipCode: '10001'
  },
  hobbies: ['reading', 'coding', 'gaming'],
  isActive: true,
  createdAt: new Date().toISOString()
};

// Write JSON with formatting
fs.writeFileSync('user-data.json', JSON.stringify(userData, null, 2));
console.log('Formatted JSON file created');

// Write JSON compact
fs.writeFileSync('user-data-compact.json', JSON.stringify(userData));
console.log('Compact JSON file created');

// 2. Writing CSV data
const csvData = [
  ['Name', 'Age', 'City', 'Email'],
  ['John Doe', 30, 'New York', 'john@example.com'],
  ['Jane Smith', 25, 'Los Angeles', 'jane@example.com'],
  ['Bob Johnson', 35, 'Chicago', 'bob@example.com']
];

const csvContent = csvData.map(row => row.join(',')).join('\n');
fs.writeFileSync('users.csv', csvContent);
console.log('CSV file created');

// 3. Writing XML data
const xmlData = `<?xml version="1.0" encoding="UTF-8"?>
<users>
  <user id="1">
    <name>John Doe</name>
    <email>john@example.com</email>
    <age>30</age>
  </user>
  <user id="2">
    <name>Jane Smith</name>
    <email>jane@example.com</email>
    <age>25</age>
  </user>
</users>`;

fs.writeFileSync('users.xml', xmlData);
console.log('XML file created');

// 4. Writing configuration files
const configData = {
  server: {
    port: 3000,
    host: 'localhost'
  },
  database: {
    host: 'localhost',
    port: 5432,
    name: 'myapp'
  },
  features: {
    enableLogging: true,
    enableCaching: false
  }
};

fs.writeFileSync('config.json', JSON.stringify(configData, null, 2));
console.log('Configuration file created');

// 5. Writing to multiple files
const users = [
  { id: 1, name: 'John', department: 'IT' },
  { id: 2, name: 'Jane', department: 'HR' },
  { id: 3, name: 'Bob', department: 'IT' }
];

users.forEach(user => {
  const fileName = `user-${user.id}.json`;
  fs.writeFileSync(fileName, JSON.stringify(user, null, 2));
  console.log(`Individual user file created: ${fileName}`);
});
```

---

## 📂 Working with Directories

### Directory Operations Demo

Create `directory-operations.js`:
```javascript
// directory-operations.js - Working with directories
const fs = require('fs');
const path = require('path');

console.log('=== Directory Operations Demo ===\n');

// 1. Creating directories
console.log('1. Creating directories:');

// Create single directory
const singleDir = 'test-directory';
if (!fs.existsSync(singleDir)) {
  fs.mkdirSync(singleDir);
  console.log('Single directory created:', singleDir);
}

// Create nested directories
const nestedDir = 'parent/child/grandchild';
if (!fs.existsSync(nestedDir)) {
  fs.mkdirSync(nestedDir, { recursive: true });
  console.log('Nested directories created:', nestedDir);
}

// 2. Reading directory contents
console.log('\n2. Reading directory contents:');

// List files in current directory
fs.readdir('.', (err, files) => {
  if (err) {
    console.error('Error reading directory:', err.message);
    return;
  }
  
  console.log('Files in current directory:');
  files.forEach(file => {
    const stats = fs.statSync(file);
    const type = stats.isDirectory() ? 'DIR' : 'FILE';
    const size = stats.isFile() ? stats.size : '-';
    console.log(`  ${type}: ${file} (${size} bytes)`);
  });
});

// 3. Directory statistics
console.log('\n3. Directory statistics:');

function analyzeDirectory(dirPath) {
  try {
    const stats = fs.statSync(dirPath);
    console.log(`Directory: ${dirPath}`);
    console.log(`  Created: ${stats.birthtime}`);
    console.log(`  Modified: ${stats.mtime}`);
    console.log(`  Is Directory: ${stats.isDirectory()}`);
    console.log(`  Is File: ${stats.isFile()}`);
  } catch (error) {
    console.error('Error analyzing directory:', error.message);
  }
}

analyzeDirectory('.');
analyzeDirectory('test-directory');

// 4. Recursive directory listing
console.log('\n4. Recursive directory listing:');

function listDirectoryRecursive(dirPath, indent = '') {
  try {
    const files = fs.readdirSync(dirPath);
    
    files.forEach(file => {
      const filePath = path.join(dirPath, file);
      const stats = fs.statSync(filePath);
      
      if (stats.isDirectory()) {
        console.log(`${indent}📁 ${file}/`);
        listDirectoryRecursive(filePath, indent + '  ');
      } else {
        console.log(`${indent}📄 ${file} (${stats.size} bytes)`);
      }
    });
  } catch (error) {
    console.error('Error listing directory:', error.message);
  }
}

listDirectoryRecursive('.');

// 5. Create directory structure
console.log('\n5. Creating project structure:');

const projectStructure = [
  'my-project',
  'my-project/src',
  'my-project/src/components',
  'my-project/src/utils',
  'my-project/tests',
  'my-project/docs'
];

projectStructure.forEach(dir => {
  if (!fs.existsSync(dir)) {
    fs.mkdirSync(dir, { recursive: true });
    console.log('Created:', dir);
  }
});

// Create some sample files
const sampleFiles = [
  'my-project/README.md',
  'my-project/package.json',
  'my-project/src/index.js',
  'my-project/src/components/Component.js',
  'my-project/src/utils/helpers.js'
];

sampleFiles.forEach(file => {
  const content = `// ${path.basename(file)}
// Created at: ${new Date().toISOString()}
console.log('Hello from ${path.basename(file)}');
`;
  fs.writeFileSync(file, content);
  console.log('Created file:', file);
});
```

### Directory Watching Demo

Create `directory-watcher.js`:
```javascript
// directory-watcher.js - Watching directory changes
const fs = require('fs');
const path = require('path');

console.log('=== Directory Watching Demo ===\n');

const watchDir = 'watch-me';

// Create directory to watch
if (!fs.existsSync(watchDir)) {
  fs.mkdirSync(watchDir);
  console.log('Created directory to watch:', watchDir);
}

// Watch for changes
console.log('Watching directory for changes...');
const watcher = fs.watch(watchDir, (eventType, filename) => {
  console.log(`Event: ${eventType}, File: ${filename}`);
  
  if (filename) {
    const filePath = path.join(watchDir, filename);
    
    // Check if file exists (might have been deleted)
    if (fs.existsSync(filePath)) {
      const stats = fs.statSync(filePath);
      console.log(`  File size: ${stats.size} bytes`);
      console.log(`  Modified: ${stats.mtime}`);
    } else {
      console.log('  File was deleted');
    }
  }
});

// Simulate file changes
console.log('\nSimulating file changes...');

setTimeout(() => {
  fs.writeFileSync(path.join(watchDir, 'test1.txt'), 'Initial content');
  console.log('Created test1.txt');
}, 1000);

setTimeout(() => {
  fs.appendFileSync(path.join(watchDir, 'test1.txt'), '\nAppended content');
  console.log('Modified test1.txt');
}, 2000);

setTimeout(() => {
  fs.writeFileSync(path.join(watchDir, 'test2.txt'), 'Another file');
  console.log('Created test2.txt');
}, 3000);

setTimeout(() => {
  fs.unlinkSync(path.join(watchDir, 'test1.txt'));
  console.log('Deleted test1.txt');
}, 4000);

// Clean up after 10 seconds
setTimeout(() => {
  console.log('\nCleaning up...');
  watcher.close();
  
  // Remove remaining files
  try {
    const files = fs.readdirSync(watchDir);
    files.forEach(file => {
      fs.unlinkSync(path.join(watchDir, file));
    });
    fs.rmdirSync(watchDir);
    console.log('Cleanup completed');
  } catch (error) {
    console.error('Cleanup error:', error.message);
  }
}, 10000);
```

---

## 🔒 File Permissions and Security

### File Permissions Demo

Create `file-permissions.js`:
```javascript
// file-permissions.js - Working with file permissions
const fs = require('fs');

console.log('=== File Permissions Demo ===\n');

// 1. Check file permissions
function checkPermissions(filePath) {
  try {
    // Check if file exists
    fs.accessSync(filePath, fs.constants.F_OK);
    console.log(`✅ ${filePath} exists`);
    
    // Check read permission
    fs.accessSync(filePath, fs.constants.R_OK);
    console.log(`✅ ${filePath} is readable`);
    
    // Check write permission
    fs.accessSync(filePath, fs.constants.W_OK);
    console.log(`✅ ${filePath} is writable`);
    
    // Check execute permission (if applicable)
    try {
      fs.accessSync(filePath, fs.constants.X_OK);
      console.log(`✅ ${filePath} is executable`);
    } catch (error) {
      console.log(`❌ ${filePath} is not executable`);
    }
    
  } catch (error) {
    console.error(`❌ Permission check failed for ${filePath}:`, error.message);
  }
}

// 2. Create test files with different permissions
const testFile = 'permission-test.txt';
fs.writeFileSync(testFile, 'Test content for permissions');

console.log('1. Initial file permissions:');
checkPermissions(testFile);

// 3. Change file permissions
console.log('\n2. Changing file permissions:');

// Make file read-only
fs.chmodSync(testFile, 0o444);
console.log('Changed to read-only (0o444)');
checkPermissions(testFile);

// Try to write to read-only file
try {
  fs.writeFileSync(testFile, 'This should fail');
} catch (error) {
  console.log('❌ Cannot write to read-only file:', error.code);
}

// Restore write permissions
fs.chmodSync(testFile, 0o666);
console.log('\n3. Restored write permissions (0o666):');
checkPermissions(testFile);

// 4. File statistics with permissions
console.log('\n4. Detailed file statistics:');
const stats = fs.statSync(testFile);
console.log('File stats:');
console.log(`  Size: ${stats.size} bytes`);
console.log(`  Mode: ${stats.mode.toString(8)}`);
console.log(`  UID: ${stats.uid}`);
console.log(`  GID: ${stats.gid}`);
console.log(`  Created: ${stats.birthtime}`);
console.log(`  Modified: ${stats.mtime}`);
console.log(`  Accessed: ${stats.atime}`);

// 5. Async permission checking
console.log('\n5. Async permission checking:');
fs.access(testFile, fs.constants.R_OK | fs.constants.W_OK, (err) => {
  if (err) {
    console.error('File is not readable/writable:', err.message);
  } else {
    console.log('File is readable and writable');
  }
});

// Cleanup
setTimeout(() => {
  fs.unlinkSync(testFile);
  console.log('Cleanup: Test file deleted');
}, 1000);
```

---

## 🚨 Error Handling Best Practices

### Error Handling Demo

Create `error-handling.js`:
```javascript
// error-handling.js - Error handling in file operations
const fs = require('fs');

console.log('=== Error Handling Demo ===\n');

// 1. Common error types
console.log('1. Common error types:');

// File not found
try {
  fs.readFileSync('nonexistent-file.txt');
} catch (error) {
  console.log('ENOENT Error:', error.code, '-', error.message);
}

// Permission denied
try {
  fs.accessSync('/etc/shadow', fs.constants.R_OK);
} catch (error) {
  console.log('EACCES Error:', error.code, '-', error.message);
}

// 2. Graceful error handling function
function safeFileRead(filePath) {
  return new Promise((resolve, reject) => {
    fs.readFile(filePath, 'utf8', (err, data) => {
      if (err) {
        switch (err.code) {
          case 'ENOENT':
            reject(new Error(`File not found: ${filePath}`));
            break;
          case 'EACCES':
            reject(new Error(`Permission denied: ${filePath}`));
            break;
          case 'EISDIR':
            reject(new Error(`Expected file, got directory: ${filePath}`));
            break;
          default:
            reject(new Error(`Unexpected error: ${err.message}`));
        }
        return;
      }
      resolve(data);
    });
  });
}

// 3. Test error handling
console.log('\n2. Testing error handling:');

async function testErrorHandling() {
  const testCases = [
    'existing-file.txt',
    'nonexistent-file.txt',
    '.',  // directory instead of file
  ];
  
  // Create test file
  fs.writeFileSync('existing-file.txt', 'This file exists');
  
  for (const testFile of testCases) {
    try {
      const data = await safeFileRead(testFile);
      console.log(`✅ Successfully read ${testFile}:`, data.substring(0, 50));
    } catch (error) {
      console.log(`❌ Error reading ${testFile}:`, error.message);
    }
  }
  
  // Cleanup
  fs.unlinkSync('existing-file.txt');
}

testErrorHandling();

// 4. Error handling with retry logic
console.log('\n3. Error handling with retry:');

function readFileWithRetry(filePath, maxRetries = 3) {
  return new Promise((resolve, reject) => {
    let attempts = 0;
    
    function attemptRead() {
      attempts++;
      fs.readFile(filePath, 'utf8', (err, data) => {
        if (err) {
          if (attempts < maxRetries && err.code !== 'ENOENT') {
            console.log(`Attempt ${attempts} failed, retrying...`);
            setTimeout(attemptRead, 1000);
          } else {
            reject(new Error(`Failed after ${attempts} attempts: ${err.message}`));
          }
          return;
        }
        resolve(data);
      });
    }
    
    attemptRead();
  });
}

// 5. Comprehensive error logging
function logFileOperation(operation, filePath, error = null) {
  const timestamp = new Date().toISOString();
  const logEntry = {
    timestamp,
    operation,
    filePath,
    success: !error,
    error: error ? {
      code: error.code,
      message: error.message
    } : null
  };
  
  console.log('Operation Log:', JSON.stringify(logEntry, null, 2));
}

// Demo logging
logFileOperation('read', 'test.txt');
logFileOperation('write', 'test.txt', new Error('Permission denied'));
```

---

## 🎯 Best Practices Summary

### File System Best Practices

Create `best-practices.js`:
```javascript
// best-practices.js - File system best practices
const fs = require('fs').promises;
const path = require('path');

console.log('=== File System Best Practices ===\n');

// 1. Always use path.join for file paths
const goodPath = path.join(__dirname, 'data', 'user.json');
const badPath = __dirname + '/data/user.json'; // Don't do this

console.log('Good path:', goodPath);
console.log('Bad path:', badPath);

// 2. Use async operations for better performance
async function bestPracticesDemo() {
  
  // 3. Check if file exists before operations
  async function safeFileOperation(filePath) {
    try {
      await fs.access(filePath);
      console.log('✅ File exists:', filePath);
      return true;
    } catch (error) {
      console.log('❌ File does not exist:', filePath);
      return false;
    }
  }
  
  // 4. Use proper error handling
  async function robustFileRead(filePath) {
    try {
      const data = await fs.readFile(filePath, 'utf8');
      return { success: true, data };
    } catch (error) {
      return { 
        success: false, 
        error: {
          code: error.code,
          message: error.message
        }
      };
    }
  }
  
  // 5. Use streams for large files
  const { createReadStream } = require('fs');
  
  function readLargeFile(filePath) {
    return new Promise((resolve, reject) => {
      const stream = createReadStream(filePath, { encoding: 'utf8' });
      let data = '';
      
      stream.on('data', chunk => {
        data += chunk;
      });
      
      stream.on('end', () => {
        resolve(data);
      });
      
      stream.on('error', reject);
    });
  }
  
  // 6. Validate file paths
  function validateFilePath(filePath) {
    // Check for path traversal attacks
    if (filePath.includes('..') || filePath.includes('~')) {
      throw new Error('Invalid file path');
    }
    
    // Ensure it's within allowed directory
    const allowedDir = path.resolve(__dirname);
    const resolvedPath = path.resolve(filePath);
    
    if (!resolvedPath.startsWith(allowedDir)) {
      throw new Error('File path outside allowed directory');
    }
    
    return resolvedPath;
  }
  
  // 7. Use atomic operations for critical files
  async function atomicWrite(filePath, data) {
    const tempPath = filePath + '.tmp';
    
    try {
      await fs.writeFile(tempPath, data);
      await fs.rename(tempPath, filePath);
      console.log('✅ Atomic write successful');
    } catch (error) {
      // Cleanup temp file if it exists
      try {
        await fs.unlink(tempPath);
      } catch (cleanupError) {
        // Ignore cleanup errors
      }
      throw error;
    }
  }
  
  // Demo the best practices
  console.log('\n1. Demonstrating best practices:');
  
  // Create test file
  const testFile = path.join(__dirname, 'best-practices-test.txt');
  await fs.writeFile(testFile, 'Test content');
  
  // Check existence
  await safeFileOperation(testFile);
  
  // Robust read
  const result = await robustFileRead(testFile);
  console.log('Read result:', result);
  
  // Validate path
  try {
    const validPath = validateFilePath(testFile);
    console.log('Valid path:', validPath);
  } catch (error) {
    console.error('Path validation error:', error.message);
  }
  
  // Atomic write
  await atomicWrite(testFile, 'Updated content atomically');
  
  // Cleanup
  await fs.unlink(testFile);
  console.log('Cleanup completed');
}

bestPracticesDemo();
```

---

## 🎉 Summary

Today you learned:
- ✅ File System module fundamentals
- ✅ Synchronous vs Asynchronous operations
- ✅ Reading files in multiple ways
- ✅ Writing and appending to files
- ✅ Working with directories
- ✅ File permissions and security
- ✅ Error handling best practices
- ✅ Performance optimization techniques

**Next Up**: Day 02 - Part 02: Introduction to Express.js Framework

---

## 🎯 Practice Exercises

### Exercise 1: Log File Manager
Create a program that:
1. Creates a daily log file
2. Appends timestamped entries
3. Rotates logs when they get too large
4. Provides search functionality

### Exercise 2: File Backup System
Create a program that:
1. Monitors a directory for changes
2. Creates backups of modified files
3. Maintains version history
4. Provides restore functionality

### Exercise 3: Configuration Manager
Create a program that:
1. Reads configuration from multiple formats (JSON, XML, CSV)
2. Validates configuration data
3. Provides a unified interface
4. Handles configuration updates

---

*Remember: The file system is your application's connection to persistent storage. Handle it with care and always plan for errors!* 💾
