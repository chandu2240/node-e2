# Day 07 - Part 01: Docker Containerization Basics

## 🎯 Learning Objectives
By the end of this lesson, you will understand:
- What Docker is and why it's amazing
- How containers work (simple explanations!)
- Creating your first Docker container
- Writing Dockerfiles for Node.js applications
- Running and managing containers
- Basic Docker commands every developer needs

---

## 🐳 What is Docker?

### Simple Explanation (for 10-year-olds!)

Imagine you have a **magic lunchbox** that can pack anything perfectly:

#### **Without Docker - The Messy Way:**
```
🏠 YOUR COMPUTER
┌─────────────────────────────────────────────────────┐
│  📦 App 1 needs Node.js v16                        │
│  📦 App 2 needs Node.js v18                        │
│  📦 App 3 needs Python 3.9                        │
│  📦 App 4 needs different libraries                │
│  Everything mixed together! 😵‍💫                    │
│  Sometimes they fight with each other!             │
└─────────────────────────────────────────────────────┘

Problems:
❌ Apps conflict with each other
❌ Hard to move apps to other computers
❌ "It works on my computer" but not yours
❌ Different versions cause problems
```

#### **With Docker - The Magic Lunchbox:**
```
🏠 YOUR COMPUTER
┌─────────────────────────────────────────────────────┐
│  📦 Lunchbox 1: [App + Node.js v16 + Libraries]    │
│  📦 Lunchbox 2: [App + Node.js v18 + Libraries]    │
│  📦 Lunchbox 3: [App + Python 3.9 + Libraries]    │
│  📦 Lunchbox 4: [App + Everything it needs]        │
│  Each app has its own complete lunchbox! 🍱        │
└─────────────────────────────────────────────────────┘

Benefits:
✅ Each app has everything it needs
✅ No conflicts between apps
✅ Easy to move to any computer
✅ Same behavior everywhere
```

### Real-World Analogy
```
Docker is like a SHIPPING CONTAINER:
┌─────────────────────────────────────────────────────┐
│  🚢 Shipping Container                              │
│  📦 Contains everything needed                      │
│  🔒 Sealed and protected                           │
│  📏 Standard size (fits any ship/truck)            │
│  🌍 Can go anywhere in the world                   │
│  📋 Label tells you what's inside                  │
└─────────────────────────────────────────────────────┘
```

### Why Kids Love Docker
- **Like LEGO blocks**: Each container is a complete piece
- **Like a backpack**: Pack everything you need for school
- **Like a game cartridge**: Insert and play anywhere
- **Like a recipe box**: Same result every time

---

## 🏗️ Your First Docker Container

### Simple Node.js App to Containerize

Create `simple-app.js`:
```javascript
// simple-app.js - Simple Node.js app for Docker
const express = require('express');

console.log('🚀 Starting our Simple App for Docker!');

const app = express();
app.use(express.json());

// Simple greeting endpoint
app.get('/', (req, res) => {
  res.json({
    message: 'Hello from Docker! 🐳',
    timestamp: new Date().toISOString(),
    container: process.env.HOSTNAME || 'unknown',
    version: '1.0.0'
  });
});

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({
    status: 'healthy',
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    timestamp: new Date().toISOString()
  });
});

// Fun facts endpoint
app.get('/fun-facts', (req, res) => {
  const facts = [
    'Docker was created in 2013! 🎂',
    'Containers are like magical boxes! 📦',
    'Docker makes apps portable! 🚀',
    'Containers share the same OS kernel! 🧠',
    'Docker Hub has millions of images! 🌟'
  ];
  
  const randomFact = facts[Math.floor(Math.random() * facts.length)];
  
  res.json({
    fact: randomFact,
    totalFacts: facts.length,
    timestamp: new Date().toISOString()
  });
});

// Environment info endpoint
app.get('/env', (req, res) => {
  res.json({
    nodeVersion: process.version,
    platform: process.platform,
    architecture: process.arch,
    hostname: process.env.HOSTNAME,
    port: process.env.PORT || 3000,
    timestamp: new Date().toISOString()
  });
});

// Kid-friendly endpoint
app.get('/kids', (req, res) => {
  res.json({
    message: 'Docker is like a magic lunchbox! 🍱',
    explanation: 'It packs your app with everything it needs!',
    benefits: [
      'Your app works the same everywhere! 🌍',
      'No more "it works on my computer" problems! 💻',
      'Easy to share with friends! 👫',
      'Apps don\'t fight with each other! ✌️'
    ],
    funEmoji: '🐳🎉🚀✨'
  });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`🌟 Simple App running on port ${PORT}`);
  console.log(`🔍 Health check: http://localhost:${PORT}/health`);
  console.log(`🎯 Main endpoint: http://localhost:${PORT}/`);
  console.log(`🎉 Fun facts: http://localhost:${PORT}/fun-facts`);
  console.log(`👶 Kids info: http://localhost:${PORT}/kids`);
});
```

### Package.json for Docker

Create `package.json`:
```json
{
  "name": "simple-docker-app",
  "version": "1.0.0",
  "description": "Simple Node.js app for Docker learning",
  "main": "simple-app.js",
  "scripts": {
    "start": "node simple-app.js",
    "dev": "nodemon simple-app.js",
    "test": "echo \"No tests yet! Add some! 🧪\" && exit 1"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^2.0.20"
  },
  "keywords": ["docker", "nodejs", "container", "beginner"],
  "author": "Docker Learning Student",
  "license": "MIT"
}
```

---

## 📝 Writing Your First Dockerfile

### Simple Dockerfile Explanation

Create `Dockerfile`:
```dockerfile
# Dockerfile - Instructions to build our container
# Think of this like a recipe for making a cake! 🍰

# Step 1: Choose the base (like choosing flour for cake)
# We use Node.js 18 on Alpine Linux (small and fast!)
FROM node:18-alpine

# Step 2: Set working directory (like setting up kitchen counter)
# This is where our app will live inside the container
WORKDIR /app

# Step 3: Copy package files first (like reading recipe ingredients)
# We copy package.json and package-lock.json
COPY package*.json ./

# Step 4: Install dependencies (like buying ingredients)
# This installs all the npm packages our app needs
RUN npm install

# Step 5: Copy our app code (like mixing ingredients)
# Copy all our source code into the container
COPY . .

# Step 6: Expose the port (like setting table for guests)
# Tell Docker our app uses port 3000
EXPOSE 3000

# Step 7: Start the app (like serving the cake!)
# This command runs when container starts
CMD ["npm", "start"]
```

### Kid-Friendly Dockerfile with Comments

Create `Dockerfile.explained`:
```dockerfile
# 🐳 DOCKERFILE = RECIPE FOR BUILDING A CONTAINER
# Think of this like instructions for building a LEGO set!

# 📦 STEP 1: Choose your base container (like choosing LEGO baseplate)
# node:18-alpine = A small Linux system with Node.js 18 already installed
# Alpine is tiny and fast - perfect for containers!
FROM node:18-alpine

# 👤 STEP 2: Add a label (like putting your name on your LEGO box)
LABEL maintainer="Your Name <your.email@example.com>"
LABEL description="Simple Node.js app for learning Docker"
LABEL version="1.0.0"

# 📁 STEP 3: Set working directory (like organizing your LEGO workspace)
# This creates a folder called /app inside the container
# All our app files will go here
WORKDIR /app

# 👥 STEP 4: Create a user (like having a special LEGO builder)
# This is for security - we don't want to run as root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# 📋 STEP 5: Copy package files (like reading LEGO instruction manual)
# We copy package.json first for better Docker caching
COPY package*.json ./

# 🛍️ STEP 6: Install dependencies (like getting all LEGO pieces)
# npm ci is faster and more reliable than npm install
RUN npm ci --only=production

# 📂 STEP 7: Copy app code (like putting LEGO pieces in place)
# Copy all source code into the container
COPY . .

# 🔧 STEP 8: Change ownership (like giving LEGO set to the builder)
RUN chown -R nextjs:nodejs /app

# 👤 STEP 9: Switch to non-root user (like having builder take over)
USER nextjs

# 🚪 STEP 10: Expose port (like marking entrance to LEGO house)
# This tells Docker which port our app will use
EXPOSE 3000

# 🏥 STEP 11: Health check (like checking if LEGO house is stable)
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# 🚀 STEP 12: Start the app (like turning on lights in LEGO house)
CMD ["npm", "start"]
```

### Multi-stage Dockerfile (Advanced)

Create `Dockerfile.multistage`:
```dockerfile
# 🏗️ MULTI-STAGE DOCKERFILE
# Like building a LEGO set, then packaging it nicely!

# 📦 STAGE 1: Build stage (like LEGO building workspace)
FROM node:18-alpine AS builder

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install ALL dependencies (including dev dependencies)
RUN npm ci

# Copy source code
COPY . .

# Build the app (if you have a build step)
# RUN npm run build

# 🚀 STAGE 2: Production stage (like the final LEGO display)
FROM node:18-alpine AS production

# Create app directory
WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# Copy package files
COPY package*.json ./

# Install only production dependencies
RUN npm ci --only=production && npm cache clean --force

# Copy app code from builder stage
COPY --from=builder --chown=nextjs:nodejs /app .

# Switch to non-root user
USER nextjs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Start the app
CMD ["npm", "start"]
```

---

## 🔧 Basic Docker Commands

### Building and Running Your First Container

Create `docker-commands.md`:
```markdown
# 🐳 DOCKER COMMANDS FOR BEGINNERS

## 📦 Building Your Container (like following a recipe)

### Build basic image
```bash
# Build image with tag "my-app"
docker build -t my-app .

# Build with specific tag
docker build -t my-app:v1.0.0 .

# Build with custom dockerfile
docker build -f Dockerfile.explained -t my-app:explained .
```

## 🚀 Running Your Container (like serving the cake)

### Run basic container
```bash
# Run container in foreground
docker run my-app

# Run container in background (detached mode)
docker run -d my-app

# Run with port mapping (your computer port 3000 -> container port 3000)
docker run -p 3000:3000 my-app

# Run with custom name
docker run -p 3000:3000 --name my-running-app my-app
```

## 🔍 Container Management (like checking on your LEGO creation)

### List containers
```bash
# List running containers
docker ps

# List all containers (running and stopped)
docker ps -a

# List with custom format
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

### Container logs
```bash
# View container logs
docker logs my-running-app

# Follow logs in real-time
docker logs -f my-running-app

# View last 50 lines
docker logs --tail 50 my-running-app
```

### Stop and remove containers
```bash
# Stop container gracefully
docker stop my-running-app

# Force stop container
docker kill my-running-app

# Remove stopped container
docker rm my-running-app

# Stop and remove in one command
docker rm -f my-running-app
```

## 🖼️ Image Management (like organizing your LEGO instruction manuals)

### List images
```bash
# List all images
docker images

# List images with custom format
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

### Remove images
```bash
# Remove specific image
docker rmi my-app

# Remove image by ID
docker rmi abc123def456

# Remove all unused images
docker image prune
```

## 🧹 Cleanup Commands (like cleaning up your LEGO workspace)

### Remove everything unused
```bash
# Remove unused containers, networks, images
docker system prune

# Remove everything unused including volumes
docker system prune -a

# Remove only stopped containers
docker container prune

# Remove only unused images
docker image prune
```

## 📊 Inspection Commands (like examining your LEGO creation)

### Inspect container
```bash
# Get detailed container info
docker inspect my-running-app

# Get specific info (like IP address)
docker inspect -f '{{.NetworkSettings.IPAddress}}' my-running-app
```

### Container stats
```bash
# View resource usage
docker stats

# View stats for specific container
docker stats my-running-app
```

## 🔧 Interactive Commands (like playing with your LEGO creation)

### Execute commands in running container
```bash
# Run bash shell in container
docker exec -it my-running-app /bin/sh

# Run specific command
docker exec my-running-app ls -la

# Run as specific user
docker exec -u nextjs my-running-app whoami
```
```

---

## 🎮 Hands-on Practice

### Docker Practice Script

Create `docker-practice.js`:
```javascript
// docker-practice.js - Practice Docker commands
const { exec } = require('child_process');
const util = require('util');

const execAsync = util.promisify(exec);

console.log('🎮 Docker Practice Helper!');
console.log('This script will help you practice Docker commands!\n');

// Practice exercises
const exercises = [
  {
    title: '🏗️ Build your first image',
    command: 'docker build -t my-first-app .',
    explanation: 'This builds a Docker image from your Dockerfile'
  },
  {
    title: '🚀 Run your container',
    command: 'docker run -p 3000:3000 --name my-app my-first-app',
    explanation: 'This runs your container and maps port 3000'
  },
  {
    title: '📋 List running containers',
    command: 'docker ps',
    explanation: 'This shows all currently running containers'
  },
  {
    title: '📝 View container logs',
    command: 'docker logs my-app',
    explanation: 'This shows the output from your container'
  },
  {
    title: '🔍 Inspect container',
    command: 'docker inspect my-app',
    explanation: 'This shows detailed information about your container'
  },
  {
    title: '🛑 Stop container',
    command: 'docker stop my-app',
    explanation: 'This gracefully stops your running container'
  },
  {
    title: '🗑️ Remove container',
    command: 'docker rm my-app',
    explanation: 'This removes the stopped container'
  }
];

// Interactive practice function
async function runPractice() {
  console.log('🎯 Docker Practice Exercises:\n');
  
  for (let i = 0; i < exercises.length; i++) {
    const exercise = exercises[i];
    console.log(`${i + 1}. ${exercise.title}`);
    console.log(`   Command: ${exercise.command}`);
    console.log(`   What it does: ${exercise.explanation}\n`);
    
    // Wait for user input (in real practice, you'd run these commands)
    console.log('   💡 Try running this command in your terminal!');
    console.log('   ⏳ Practice this command, then continue...\n');
    
    // In a real scenario, you might want to pause here
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  
  console.log('🎉 Great job! You\'ve practiced basic Docker commands!');
  console.log('🚀 Now you can build and run containers like a pro!');
}

// Helper function to check if Docker is installed
async function checkDocker() {
  try {
    const { stdout } = await execAsync('docker --version');
    console.log('✅ Docker is installed:', stdout.trim());
    return true;
  } catch (error) {
    console.log('❌ Docker is not installed or not running');
    console.log('📥 Please install Docker from: https://www.docker.com/');
    return false;
  }
}

// Run the practice
async function main() {
  const dockerAvailable = await checkDocker();
  
  if (dockerAvailable) {
    await runPractice();
  } else {
    console.log('\n💡 Docker Installation Tips:');
    console.log('1. Go to https://www.docker.com/');
    console.log('2. Download Docker Desktop for your operating system');
    console.log('3. Install and start Docker Desktop');
    console.log('4. Run this script again!');
  }
}

// Start the practice
main().catch(console.error);
```

### Docker Cheat Sheet

Create `docker-cheat-sheet.md`:
```markdown
# 🐳 DOCKER CHEAT SHEET FOR KIDS

## 🏗️ Building (Like following a recipe)
- `docker build -t my-app .` → Build image called "my-app"
- `docker build -t my-app:v1 .` → Build with version tag
- `docker images` → List all images (like recipe books)

## 🚀 Running (Like serving the food)
- `docker run my-app` → Run container once
- `docker run -d my-app` → Run in background
- `docker run -p 3000:3000 my-app` → Run with port access
- `docker run --name my-container my-app` → Run with custom name

## 📋 Checking (Like checking on your food)
- `docker ps` → List running containers
- `docker ps -a` → List all containers (running and stopped)
- `docker logs my-container` → See what container is saying
- `docker logs -f my-container` → Watch logs in real-time

## 🛑 Stopping (Like turning off the oven)
- `docker stop my-container` → Stop container nicely
- `docker kill my-container` → Force stop container
- `docker rm my-container` → Remove stopped container
- `docker rm -f my-container` → Force remove container

## 🧹 Cleaning (Like washing dishes)
- `docker system prune` → Clean unused stuff
- `docker image prune` → Clean unused images
- `docker container prune` → Clean stopped containers

## 🔍 Investigating (Like taste testing)
- `docker inspect my-container` → Get detailed info
- `docker exec -it my-container /bin/sh` → Go inside container
- `docker stats` → See resource usage

## 🎯 Common Patterns
- Build: `docker build -t my-app .`
- Run: `docker run -p 3000:3000 --name my-app-container my-app`
- Check: `docker ps`
- Stop: `docker stop my-app-container`
- Clean: `docker rm my-app-container`

## 🆘 Emergency Commands
- `docker kill $(docker ps -q)` → Stop all running containers
- `docker rm $(docker ps -a -q)` → Remove all containers
- `docker system prune -a` → Clean everything
```

---

## 🎯 Summary

Today you learned:
- ✅ Docker concepts (magic lunchbox analogy!)
- ✅ What containers are and why they're amazing
- ✅ Writing Dockerfiles (like recipes)
- ✅ Building and running containers
- ✅ Basic Docker commands
- ✅ Container management and cleanup

**Next up: Docker Compose and Multi-Container Applications!** 🚀

---

## 🎉 Fun Docker Facts for Kids

- 🐳 Docker's mascot is a whale carrying containers!
- 📦 Docker containers are like LEGO blocks - you can combine them!
- 🚀 Docker makes it super easy to share your apps with friends!
- 🌍 Your app will work the same on any computer with Docker!
- 🎯 Docker was named after a person who loads cargo onto ships!

---

*Remember: Docker is like a magic lunchbox that packs your app with everything it needs to run anywhere! 🍱✨*
