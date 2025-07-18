# Day 01 - Part 02: Setting up Node.js Development Environment & Your First Program

## 🎯 Learning Objectives
By the end of this lesson, you will:
- Install and configure Node.js on your system
- Understand the Node.js installation components
- Set up a proper development environment
- Write and run your first Node.js programs
- Understand the Node.js REPL (Read-Eval-Print-Loop)
- Learn about package.json and project structure

---

## 🛠️ Installing Node.js

### Step 1: Download Node.js

#### For Windows:
1. Go to [nodejs.org](https://nodejs.org)
2. Download the **LTS (Long Term Support)** version
3. Choose the Windows Installer (.msi) - 64-bit
4. Run the installer as Administrator

#### For macOS:
1. Download the macOS Installer (.pkg)
2. Or use Homebrew: `brew install node`

#### For Linux (Ubuntu/Debian):
```bash
# Using NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# Or using snap
sudo snap install node --classic
```

### Step 2: Verify Installation

Open your terminal/command prompt and run:

```bash
# Check Node.js version
node --version
# or
node -v

# Check NPM version
npm --version
# or
npm -v
```

Expected output:
```
v18.17.0  (or similar)
9.6.7     (or similar)
```

### 🔍 What Gets Installed?

When you install Node.js, you get:

1. **Node.js Runtime** - The JavaScript engine
2. **NPM (Node Package Manager)** - Package manager
3. **Node.js Documentation** - Offline help
4. **NPX** - Package runner tool

---

## 🏗️ Setting Up Your Development Environment

### 1. **Choose Your Code Editor**

#### Recommended: Visual Studio Code
- **Free** and **powerful**
- **Built-in terminal**
- **Excellent Node.js support**
- **Extensions marketplace**

#### Essential VS Code Extensions for Node.js:
```
1. Node.js Extension Pack
2. ESLint
3. Prettier - Code formatter
4. Auto Import - ES6, TS, JSX, TSX
5. Thunder Client (for API testing)
6. GitLens
7. Bracket Pair Colorizer
```

### 2. **Create Your First Node.js Project**

#### Create Project Directory:
```bash
# Create and navigate to project folder
mkdir my-first-node-app
cd my-first-node-app

# Initialize project
npm init -y
```

#### Project Structure:
```
my-first-node-app/
├── package.json          # Project configuration
├── index.js              # Main application file
├── node_modules/         # Dependencies (auto-created)
├── .gitignore            # Files to ignore in git
└── README.md             # Project documentation
```

---

## 📝 Understanding package.json

### What is package.json?
Think of `package.json` as your project's **ID card** - it contains all the important information about your project.

### Sample package.json:
```json
{
  "name": "my-first-node-app",
  "version": "1.0.0",
  "description": "My first Node.js application",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "node index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": ["nodejs", "javascript", "beginner"],
  "author": "Your Name",
  "license": "MIT",
  "dependencies": {},
  "devDependencies": {}
}
```

### Key Fields Explained:

| Field | Purpose | Example |
|-------|---------|---------|
| `name` | Project name | "my-first-node-app" |
| `version` | Project version | "1.0.0" |
| `description` | What your project does | "My first Node.js app" |
| `main` | Entry point file | "index.js" |
| `scripts` | Commands you can run | `npm start`, `npm test` |
| `dependencies` | Required packages | Express, lodash |
| `devDependencies` | Development-only packages | nodemon, jest |

---

## 🎮 Node.js REPL (Interactive Shell)

### What is REPL?
**R**ead-**E**val-**P**rint-**L**oop - An interactive shell for testing JavaScript code.

### Starting REPL:
```bash
node
```

### REPL Examples:
```javascript
// Simple calculations
> 2 + 3
5

// Variables
> let name = "Node.js"
undefined
> name
'Node.js'

// Functions
> function greet(name) {
    return `Hello, ${name}!`;
  }
undefined
> greet("World")
'Hello, World!'

// Built-in objects
> console.log("Hello REPL!")
Hello REPL!
undefined

// Exit REPL
> .exit
```

### REPL Commands:
```javascript
.help       // Show help
.exit       // Exit REPL
.clear      // Clear context
.save       // Save session to file
.load       // Load file into session
```

---

## 🚀 Your First Node.js Programs

### Program 1: Hello World

Create `hello.js`:
```javascript
// hello.js
console.log("Hello, World!");
console.log("Welcome to Node.js!");
console.log("Today is:", new Date().toDateString());
```

Run it:
```bash
node hello.js
```

Expected output:
```
Hello, World!
Welcome to Node.js!
Today is: Fri Jul 18 2025
```

### Program 2: Interactive Input

Create `interactive.js`:
```javascript
// interactive.js
const readline = require('readline');

// Create interface for input/output
const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

// Ask for user's name
rl.question('What is your name? ', (name) => {
  console.log(`Hello, ${name}!`);
  
  // Ask for age
  rl.question('How old are you? ', (age) => {
    console.log(`Wow! You are ${age} years old.`);
    
    // Calculate birth year
    const currentYear = new Date().getFullYear();
    const birthYear = currentYear - parseInt(age);
    console.log(`You were born in approximately ${birthYear}.`);
    
    // Close the interface
    rl.close();
  });
});
```

Run it:
```bash
node interactive.js
```

### Program 3: Simple Calculator

Create `calculator.js`:
```javascript
// calculator.js
const readline = require('readline');

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

console.log("=== Simple Calculator ===");

// Get first number
rl.question('Enter first number: ', (num1) => {
  // Get operator
  rl.question('Enter operator (+, -, *, /): ', (operator) => {
    // Get second number
    rl.question('Enter second number: ', (num2) => {
      
      // Convert strings to numbers
      const n1 = parseFloat(num1);
      const n2 = parseFloat(num2);
      let result;
      
      // Perform calculation
      switch(operator) {
        case '+':
          result = n1 + n2;
          break;
        case '-':
          result = n1 - n2;
          break;
        case '*':
          result = n1 * n2;
          break;
        case '/':
          if (n2 === 0) {
            console.log("Error: Cannot divide by zero!");
            rl.close();
            return;
          }
          result = n1 / n2;
          break;
        default:
          console.log("Error: Invalid operator!");
          rl.close();
          return;
      }
      
      console.log(`Result: ${n1} ${operator} ${n2} = ${result}`);
      rl.close();
    });
  });
});
```

### Program 4: File Operations

Create `file-ops.js`:
```javascript
// file-ops.js
const fs = require('fs');
const path = require('path');

// Create a simple text file
const data = `Hello from Node.js!
This file was created on: ${new Date().toISOString()}
Node.js version: ${process.version}
Platform: ${process.platform}
`;

const fileName = 'my-file.txt';

// Write to file
fs.writeFile(fileName, data, (err) => {
  if (err) {
    console.error('Error writing file:', err);
    return;
  }
  
  console.log(`File '${fileName}' created successfully!`);
  
  // Read the file back
  fs.readFile(fileName, 'utf8', (err, content) => {
    if (err) {
      console.error('Error reading file:', err);
      return;
    }
    
    console.log('\nFile contents:');
    console.log('='.repeat(40));
    console.log(content);
    console.log('='.repeat(40));
  });
});
```

---

## 🔧 Command Line Arguments

### Program 5: Command Line Arguments

Create `args.js`:
```javascript
// args.js
console.log('Command line arguments:');
console.log('process.argv:', process.argv);
console.log('');

// process.argv[0] = node executable path
// process.argv[1] = script file path
// process.argv[2+] = actual arguments

const args = process.argv.slice(2);

if (args.length === 0) {
  console.log('No arguments provided!');
  console.log('Usage: node args.js <name> <age>');
  process.exit(1);
}

const name = args[0];
const age = args[1];

console.log(`Hello, ${name}!`);
if (age) {
  console.log(`You are ${age} years old.`);
}
```

Run with arguments:
```bash
node args.js John 25
```

Output:
```
Command line arguments:
process.argv: [ 'C:\\Program Files\\nodejs\\node.exe', 'C:\\path\\to\\args.js', 'John', '25' ]

Hello, John!
You are 25 years old.
```

---

## 🌍 Environment Variables

### Program 6: Environment Variables

Create `env.js`:
```javascript
// env.js
console.log('Environment Variables:');
console.log('='.repeat(50));

// Common environment variables
console.log('Node.js version:', process.version);
console.log('Platform:', process.platform);
console.log('Architecture:', process.arch);
console.log('Current working directory:', process.cwd());
console.log('Home directory:', require('os').homedir());
console.log('Username:', require('os').userInfo().username);

// Custom environment variables
console.log('\nCustom Environment Variables:');
console.log('NODE_ENV:', process.env.NODE_ENV || 'not set');
console.log('PORT:', process.env.PORT || 'not set');

// Set a custom environment variable
process.env.MY_APP_NAME = 'My First Node App';
console.log('MY_APP_NAME:', process.env.MY_APP_NAME);
```

Run with environment variables:
```bash
# Windows
set NODE_ENV=development && node env.js

# macOS/Linux
NODE_ENV=development node env.js
```

---

## 🎯 Project Structure Best Practices

### Recommended Project Structure:
```
my-node-project/
├── package.json              # Project configuration
├── package-lock.json         # Dependency lock file
├── .gitignore               # Git ignore file
├── .env                     # Environment variables
├── README.md                # Project documentation
├── index.js                 # Main entry point
├── src/                     # Source code
│   ├── controllers/         # Route handlers
│   ├── models/             # Data models
│   ├── routes/             # Route definitions
│   ├── middleware/         # Custom middleware
│   ├── utils/              # Utility functions
│   └── config/             # Configuration files
├── tests/                   # Test files
├── docs/                    # Documentation
└── node_modules/            # Dependencies (auto-generated)
```

### Create .gitignore:
```gitignore
# Node.js
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Environment variables
.env
.env.local
.env.production

# Logs
logs/
*.log

# Runtime data
pids/
*.pid
*.seed

# Coverage directory used by tools like istanbul
coverage/

# OS generated files
.DS_Store
Thumbs.db

# IDE files
.vscode/
.idea/
*.swp
*.swo
```

---

## 🚀 NPM Scripts

### Enhanced package.json with Scripts:
```json
{
  "name": "my-first-node-app",
  "version": "1.0.0",
  "description": "My first Node.js application",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "node index.js",
    "hello": "node hello.js",
    "calc": "node calculator.js",
    "interactive": "node interactive.js",
    "file-ops": "node file-ops.js",
    "args": "node args.js",
    "env": "node env.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": ["nodejs", "javascript", "beginner"],
  "author": "Your Name",
  "license": "MIT"
}
```

### Running Scripts:
```bash
npm run hello
npm run calc
npm run interactive
npm run file-ops
npm run args John 25
npm run env
```

---

## 🔍 Debugging Node.js

### 1. Basic Debugging with console.log:
```javascript
// debug.js
function addNumbers(a, b) {
  console.log('Input a:', a, 'type:', typeof a);
  console.log('Input b:', b, 'type:', typeof b);
  
  const result = a + b;
  console.log('Result:', result);
  
  return result;
}

addNumbers(5, 3);
addNumbers('5', '3');  // This will concatenate strings!
```

### 2. Node.js Debugger:
```javascript
// Add debugger statement
function calculate(x, y) {
  debugger;  // Execution will pause here
  const sum = x + y;
  return sum;
}

const result = calculate(10, 20);
console.log('Result:', result);
```

Run with debugger:
```bash
node inspect debug.js
```

### 3. VS Code Debugging:
Create `.vscode/launch.json`:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Node.js",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/index.js",
      "console": "integratedTerminal"
    }
  ]
}
```

---

## 🎯 Practical Exercises

### Exercise 1: Personal Info App
Create a program that:
1. Asks for name, age, and favorite color
2. Calculates birth year
3. Saves the information to a file
4. Reads and displays the file content

### Exercise 2: Simple Math Quiz
Create a program that:
1. Generates two random numbers
2. Asks user to add them
3. Checks if the answer is correct
4. Keeps score

### Exercise 3: File Explorer
Create a program that:
1. Lists files in current directory
2. Shows file sizes
3. Identifies file types
4. Counts total files

---

## 🏆 Advanced Tips

### 1. **Using ES6+ Features:**
```javascript
// Modern JavaScript in Node.js
const greet = (name = 'World') => {
  return `Hello, ${name}!`;
};

const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);
const sum = numbers.reduce((acc, n) => acc + n, 0);

console.log(greet());
console.log('Doubled:', doubled);
console.log('Sum:', sum);
```

### 2. **Error Handling:**
```javascript
// Proper error handling
try {
  const result = riskyOperation();
  console.log('Success:', result);
} catch (error) {
  console.error('Error:', error.message);
  process.exit(1);
}

function riskyOperation() {
  if (Math.random() > 0.5) {
    throw new Error('Something went wrong!');
  }
  return 'Success!';
}
```

### 3. **Process Management:**
```javascript
// Graceful shutdown
process.on('SIGINT', () => {
  console.log('\nReceived SIGINT. Graceful shutdown...');
  // Clean up code here
  process.exit(0);
});

process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error);
  process.exit(1);
});
```

---

## 🎉 Summary

Today you learned:
- ✅ How to install and verify Node.js
- ✅ Setting up a professional development environment
- ✅ Understanding package.json and project structure
- ✅ Using the Node.js REPL for testing
- ✅ Writing and running various Node.js programs
- ✅ Working with command line arguments and environment variables
- ✅ Basic debugging techniques
- ✅ Best practices for project organization

**Next Up**: Day 01 - Part 03: Node.js Core Concepts (Modules, Event Loop, and Global Objects)

---

## 📚 Additional Resources

- [Node.js Official Documentation](https://nodejs.org/en/docs/)
- [NPM Documentation](https://docs.npmjs.com/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [VS Code Node.js Tutorial](https://code.visualstudio.com/docs/nodejs/nodejs-tutorial)

---

*Remember: The best way to learn Node.js is by doing! Practice writing small programs every day.* 💪
