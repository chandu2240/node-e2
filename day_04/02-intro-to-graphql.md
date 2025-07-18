# Day 04 - Part 02: GraphQL Introduction and Database Integration

## 🎯 Learning Objectives
By the end of this lesson, you will understand:
- What GraphQL is and how it differs from REST
- Setting up a GraphQL server with Node.js
- Creating schemas, types, and resolvers
- Querying and mutating data with GraphQL
- Integrating with databases (MongoDB and SQLite)
- Real-world GraphQL patterns and best practices

---

## 🎨 What is GraphQL?

### Simple Explanation (for 10-year-olds!)

Imagine you're at a **magical restaurant** where you can order EXACTLY what you want:

#### **REST API Restaurant (Traditional):**
```
👦 You: "I want a burger meal"
👨‍🍳 Chef: "Here's a burger, fries, drink, napkins, ketchup, mustard, and pickles"
👦 You: "I only wanted the burger and fries... 😕"
```

#### **GraphQL Restaurant (Magical):**
```
👦 You: "I want a burger (with cheese, no pickles) and fries (large size)"
👨‍🍳 Chef: "Here's exactly what you asked for! ✨"
👦 You: "Perfect! Exactly what I wanted! 😊"
```

### Real-World Analogy
```
GraphQL is like a SMART WAITER:
┌─────────────────────────────────────────────────────┐
│  📝 Takes your EXACT order (what fields you want)   │
│  🍽️ Brings you ONLY what you asked for             │
│  ⚡ Makes ONE trip instead of multiple trips        │
│  🎯 No waste, no extra stuff you don't need        │
└─────────────────────────────────────────────────────┘
```

### Why GraphQL is Amazing
- **Get Exactly What You Need**: No more, no less
- **Single Request**: One query gets all your data
- **Flexible**: Frontend can ask for different data without backend changes
- **Self-Documenting**: Built-in documentation and type system

---

## 🛠️ Setting Up GraphQL Server

### Basic GraphQL Server

Create `graphql-basic.js`:
```javascript
// graphql-basic.js - Your first GraphQL server
const { graphql, buildSchema } = require('graphql');
const express = require('express');
const { graphqlHTTP } = require('express-graphql');

console.log('🎨 Building your first GraphQL server!');

// 1. Define your schema (like a menu at a restaurant)
const schema = buildSchema(`
  type Book {
    id: ID!
    title: String!
    author: String!
    year: Int!
    genre: String!
    rating: Float
  }
  
  type Query {
    books: [Book]
    book(id: ID!): Book
    booksByGenre(genre: String!): [Book]
  }
  
  type Mutation {
    addBook(title: String!, author: String!, year: Int!, genre: String!): Book
    updateBook(id: ID!, title: String, author: String, year: Int, genre: String): Book
    deleteBook(id: ID!): Book
  }
`);

// 2. Sample data (in real apps, this would come from a database)
let books = [
  { id: '1', title: 'Harry Potter', author: 'J.K. Rowling', year: 1997, genre: 'Fantasy', rating: 4.8 },
  { id: '2', title: 'The Hobbit', author: 'J.R.R. Tolkien', year: 1937, genre: 'Fantasy', rating: 4.7 },
  { id: '3', title: '1984', author: 'George Orwell', year: 1949, genre: 'Dystopian', rating: 4.6 },
  { id: '4', title: 'To Kill a Mockingbird', author: 'Harper Lee', year: 1960, genre: 'Classic', rating: 4.5 }
];

// 3. Define resolvers (like chefs who prepare your order)
const rootValue = {
  // Query resolvers (READ operations)
  books: () => {
    console.log('📚 Someone wants to see all books!');
    return books;
  },
  
  book: ({ id }) => {
    console.log(`📖 Someone wants to see book with ID: ${id}`);
    return books.find(book => book.id === id);
  },
  
  booksByGenre: ({ genre }) => {
    console.log(`🔍 Someone wants to see ${genre} books!`);
    return books.filter(book => book.genre.toLowerCase() === genre.toLowerCase());
  },
  
  // Mutation resolvers (CREATE, UPDATE, DELETE operations)
  addBook: ({ title, author, year, genre }) => {
    console.log(`➕ Adding new book: ${title}`);
    const newBook = {
      id: String(books.length + 1),
      title,
      author,
      year,
      genre,
      rating: 0
    };
    books.push(newBook);
    return newBook;
  },
  
  updateBook: ({ id, title, author, year, genre }) => {
    console.log(`✏️ Updating book with ID: ${id}`);
    const bookIndex = books.findIndex(book => book.id === id);
    if (bookIndex === -1) return null;
    
    const book = books[bookIndex];
    if (title) book.title = title;
    if (author) book.author = author;
    if (year) book.year = year;
    if (genre) book.genre = genre;
    
    return book;
  },
  
  deleteBook: ({ id }) => {
    console.log(`🗑️ Deleting book with ID: ${id}`);
    const bookIndex = books.findIndex(book => book.id === id);
    if (bookIndex === -1) return null;
    
    const deletedBook = books[bookIndex];
    books.splice(bookIndex, 1);
    return deletedBook;
  }
};

// 4. Create Express server with GraphQL
const app = express();
const PORT = 4000;

app.use('/graphql', graphqlHTTP({
  schema: schema,
  rootValue: rootValue,
  graphiql: true, // Enable GraphiQL interface for testing
}));

app.listen(PORT, () => {
  console.log(`🚀 GraphQL server running on http://localhost:${PORT}/graphql`);
  console.log('🎨 GraphiQL interface available for testing!');
  console.log('\nTry these queries:');
  console.log('📚 Get all books: { books { id title author } }');
  console.log('📖 Get one book: { book(id: "1") { title author year } }');
  console.log('🔍 Get by genre: { booksByGenre(genre: "Fantasy") { title author } }');
});
```

### GraphQL Queries and Mutations

Create `graphql-queries.js`:
```javascript
// graphql-queries.js - Understanding GraphQL queries
console.log('🔍 Learning GraphQL Queries!');

// This file shows example queries you can run in GraphiQL
// Just copy and paste these into the GraphiQL interface!

console.log(`
🎯 BASIC QUERIES:

1. Get all books with specific fields:
{
  books {
    id
    title
    author
  }
}

2. Get a specific book:
{
  book(id: "1") {
    title
    author
    year
    genre
    rating
  }
}

3. Get books by genre:
{
  booksByGenre(genre: "Fantasy") {
    title
    author
    rating
  }
}

4. Get different data in one query:
{
  allBooks: books {
    title
    author
  }
  fantasyBooks: booksByGenre(genre: "Fantasy") {
    title
    rating
  }
  specificBook: book(id: "1") {
    title
    year
  }
}

🔧 MUTATIONS:

1. Add a new book:
mutation {
  addBook(
    title: "The Great Gatsby"
    author: "F. Scott Fitzgerald"
    year: 1925
    genre: "Classic"
  ) {
    id
    title
    author
  }
}

2. Update a book:
mutation {
  updateBook(
    id: "1"
    title: "Harry Potter and the Philosopher's Stone"
    rating: 4.9
  ) {
    id
    title
    rating
  }
}

3. Delete a book:
mutation {
  deleteBook(id: "2") {
    id
    title
    author
  }
}

🚀 ADVANCED QUERIES:

1. Query with variables:
query GetBooksByGenre($genre: String!) {
  booksByGenre(genre: $genre) {
    title
    author
    rating
  }
}

Variables: { "genre": "Fantasy" }

2. Query with aliases:
{
  oldBooks: booksByGenre(genre: "Classic") {
    title
    year
  }
  newBooks: booksByGenre(genre: "Fantasy") {
    title
    year
  }
}
`);

// Example of using GraphQL programmatically
const { graphql, buildSchema } = require('graphql');

// You can also run queries programmatically like this:
async function runExampleQuery() {
  console.log('\n🧪 Running programmatic GraphQL query...');
  
  const schema = buildSchema(`
    type Query {
      hello: String
      number: Int
    }
  `);
  
  const rootValue = {
    hello: () => 'Hello GraphQL World! 🌍',
    number: () => 42
  };
  
  const query = '{ hello number }';
  
  try {
    const result = await graphql(schema, query, rootValue);
    console.log('📊 Query result:', result);
  } catch (error) {
    console.log('❌ Query error:', error.message);
  }
}

runExampleQuery();
```

---

## 🗄️ Database Integration

### MongoDB Integration

Create `graphql-mongodb.js`:
```javascript
// graphql-mongodb.js - GraphQL with MongoDB
const { graphql, buildSchema } = require('graphql');
const express = require('express');
const { graphqlHTTP } = require('express-graphql');
const { MongoClient } = require('mongodb');

console.log('🗄️ Connecting GraphQL with MongoDB!');

// MongoDB connection (using in-memory mock for demonstration)
class MockMongoDB {
  constructor() {
    this.users = [
      { _id: '1', name: 'Alice', email: 'alice@example.com', age: 25 },
      { _id: '2', name: 'Bob', email: 'bob@example.com', age: 30 },
      { _id: '3', name: 'Charlie', email: 'charlie@example.com', age: 35 }
    ];
    this.posts = [
      { _id: '1', title: 'Hello World', content: 'This is my first post!', authorId: '1' },
      { _id: '2', title: 'GraphQL is Cool', content: 'Learning GraphQL with Node.js', authorId: '1' },
      { _id: '3', title: 'MongoDB Tips', content: 'Some useful MongoDB tips', authorId: '2' }
    ];
  }
  
  async findUsers() {
    return this.users;
  }
  
  async findUser(id) {
    return this.users.find(user => user._id === id);
  }
  
  async findPosts() {
    return this.posts;
  }
  
  async findPost(id) {
    return this.posts.find(post => post._id === id);
  }
  
  async findPostsByAuthor(authorId) {
    return this.posts.filter(post => post.authorId === authorId);
  }
  
  async insertUser(user) {
    const newUser = { _id: String(this.users.length + 1), ...user };
    this.users.push(newUser);
    return newUser;
  }
  
  async insertPost(post) {
    const newPost = { _id: String(this.posts.length + 1), ...post };
    this.posts.push(newPost);
    return newPost;
  }
}

const db = new MockMongoDB();

// GraphQL Schema with relationships
const schema = buildSchema(`
  type User {
    id: ID!
    name: String!
    email: String!
    age: Int!
    posts: [Post]
  }
  
  type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
  }
  
  type Query {
    users: [User]
    user(id: ID!): User
    posts: [Post]
    post(id: ID!): Post
  }
  
  type Mutation {
    addUser(name: String!, email: String!, age: Int!): User
    addPost(title: String!, content: String!, authorId: ID!): Post
  }
`);

// Resolvers with database operations
const rootValue = {
  // Query resolvers
  users: async () => {
    console.log('👥 Fetching all users from database...');
    return await db.findUsers();
  },
  
  user: async ({ id }) => {
    console.log(`👤 Fetching user ${id} from database...`);
    return await db.findUser(id);
  },
  
  posts: async () => {
    console.log('📝 Fetching all posts from database...');
    return await db.findPosts();
  },
  
  post: async ({ id }) => {
    console.log(`📄 Fetching post ${id} from database...`);
    return await db.findPost(id);
  },
  
  // Mutation resolvers
  addUser: async ({ name, email, age }) => {
    console.log(`➕ Adding new user: ${name}`);
    return await db.insertUser({ name, email, age });
  },
  
  addPost: async ({ title, content, authorId }) => {
    console.log(`➕ Adding new post: ${title}`);
    return await db.insertPost({ title, content, authorId });
  }
};

// Field resolvers for relationships
const User = {
  posts: async (user) => {
    console.log(`📝 Fetching posts for user ${user.name}...`);
    return await db.findPostsByAuthor(user._id);
  }
};

const Post = {
  author: async (post) => {
    console.log(`👤 Fetching author for post ${post.title}...`);
    return await db.findUser(post.authorId);
  }
};

// Enhanced resolvers with field resolvers
const enhancedResolvers = {
  ...rootValue,
  User,
  Post
};

// Create Express server
const app = express();
const PORT = 4001;

app.use('/graphql', graphqlHTTP({
  schema: schema,
  rootValue: enhancedResolvers,
  graphiql: true,
}));

app.listen(PORT, () => {
  console.log(`🗄️ GraphQL + MongoDB server running on http://localhost:${PORT}/graphql`);
  console.log('\nTry these queries:');
  console.log('👥 Users with posts: { users { name email posts { title } } }');
  console.log('📝 Posts with authors: { posts { title author { name } } }');
});
```

### SQLite Integration

Create `graphql-sqlite.js`:
```javascript
// graphql-sqlite.js - GraphQL with SQLite
const { graphql, buildSchema } = require('graphql');
const express = require('express');
const { graphqlHTTP } = require('express-graphql');
const sqlite3 = require('sqlite3').verbose();

console.log('💾 Connecting GraphQL with SQLite!');

// SQLite Database Setup
class SQLiteDB {
  constructor() {
    this.db = new sqlite3.Database(':memory:');
    this.initializeDatabase();
  }
  
  initializeDatabase() {
    console.log('🔧 Initializing SQLite database...');
    
    // Create tables
    this.db.serialize(() => {
      this.db.run(`
        CREATE TABLE users (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          name TEXT NOT NULL,
          email TEXT UNIQUE NOT NULL,
          age INTEGER NOT NULL
        )
      `);
      
      this.db.run(`
        CREATE TABLE posts (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          title TEXT NOT NULL,
          content TEXT NOT NULL,
          author_id INTEGER NOT NULL,
          FOREIGN KEY (author_id) REFERENCES users (id)
        )
      `);
      
      // Insert sample data
      this.db.run(`
        INSERT INTO users (name, email, age) VALUES 
        ('Alice', 'alice@example.com', 25),
        ('Bob', 'bob@example.com', 30),
        ('Charlie', 'charlie@example.com', 35)
      `);
      
      this.db.run(`
        INSERT INTO posts (title, content, author_id) VALUES 
        ('Hello World', 'This is my first post!', 1),
        ('GraphQL is Cool', 'Learning GraphQL with Node.js', 1),
        ('SQLite Tips', 'Some useful SQLite tips', 2)
      `);
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
  
  async getUser(id) {
    return new Promise((resolve, reject) => {
      this.db.get('SELECT * FROM users WHERE id = ?', [id], (err, row) => {
        if (err) reject(err);
        else resolve(row);
      });
    });
  }
  
  async getAllPosts() {
    return new Promise((resolve, reject) => {
      this.db.all('SELECT * FROM posts', (err, rows) => {
        if (err) reject(err);
        else resolve(rows);
      });
    });
  }
  
  async getPost(id) {
    return new Promise((resolve, reject) => {
      this.db.get('SELECT * FROM posts WHERE id = ?', [id], (err, row) => {
        if (err) reject(err);
        else resolve(row);
      });
    });
  }
  
  async getPostsByAuthor(authorId) {
    return new Promise((resolve, reject) => {
      this.db.all('SELECT * FROM posts WHERE author_id = ?', [authorId], (err, rows) => {
        if (err) reject(err);
        else resolve(rows);
      });
    });
  }
  
  async insertUser(name, email, age) {
    return new Promise((resolve, reject) => {
      this.db.run('INSERT INTO users (name, email, age) VALUES (?, ?, ?)', 
        [name, email, age], 
        function(err) {
          if (err) reject(err);
          else resolve({ id: this.lastID, name, email, age });
        }
      );
    });
  }
  
  async insertPost(title, content, authorId) {
    return new Promise((resolve, reject) => {
      this.db.run('INSERT INTO posts (title, content, author_id) VALUES (?, ?, ?)', 
        [title, content, authorId], 
        function(err) {
          if (err) reject(err);
          else resolve({ id: this.lastID, title, content, author_id: authorId });
        }
      );
    });
  }
}

const database = new SQLiteDB();

// GraphQL Schema
const schema = buildSchema(`
  type User {
    id: ID!
    name: String!
    email: String!
    age: Int!
    posts: [Post]
  }
  
  type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
  }
  
  type Query {
    users: [User]
    user(id: ID!): User
    posts: [Post]
    post(id: ID!): Post
  }
  
  type Mutation {
    addUser(name: String!, email: String!, age: Int!): User
    addPost(title: String!, content: String!, authorId: ID!): Post
  }
`);

// Resolvers
const rootValue = {
  // Query resolvers
  users: async () => {
    console.log('👥 Fetching all users from SQLite...');
    return await database.getAllUsers();
  },
  
  user: async ({ id }) => {
    console.log(`👤 Fetching user ${id} from SQLite...`);
    return await database.getUser(id);
  },
  
  posts: async () => {
    console.log('📝 Fetching all posts from SQLite...');
    return await database.getAllPosts();
  },
  
  post: async ({ id }) => {
    console.log(`📄 Fetching post ${id} from SQLite...`);
    return await database.getPost(id);
  },
  
  // Mutation resolvers
  addUser: async ({ name, email, age }) => {
    console.log(`➕ Adding new user: ${name}`);
    return await database.insertUser(name, email, age);
  },
  
  addPost: async ({ title, content, authorId }) => {
    console.log(`➕ Adding new post: ${title}`);
    return await database.insertPost(title, content, authorId);
  }
};

// Field resolvers for relationships
const User = {
  posts: async (user) => {
    console.log(`📝 Fetching posts for user ${user.name}...`);
    return await database.getPostsByAuthor(user.id);
  }
};

const Post = {
  author: async (post) => {
    console.log(`👤 Fetching author for post ${post.title}...`);
    return await database.getUser(post.author_id);
  }
};

// Enhanced resolvers
const enhancedResolvers = {
  ...rootValue,
  User,
  Post
};

// Create Express server
const app = express();
const PORT = 4002;

app.use('/graphql', graphqlHTTP({
  schema: schema,
  rootValue: enhancedResolvers,
  graphiql: true,
}));

app.listen(PORT, () => {
  console.log(`💾 GraphQL + SQLite server running on http://localhost:${PORT}/graphql`);
  console.log('\nTry these relationship queries:');
  console.log('👥 Users with posts: { users { name posts { title } } }');
  console.log('📝 Posts with authors: { posts { title author { name email } } }');
});
```

---

## 🔧 Advanced GraphQL Features

### GraphQL with Validation and Authentication

Create `graphql-advanced.js`:
```javascript
// graphql-advanced.js - Advanced GraphQL features
const { graphql, buildSchema } = require('graphql');
const express = require('express');
const { graphqlHTTP } = require('express-graphql');
const jwt = require('jsonwebtoken');

console.log('🔧 Learning Advanced GraphQL Features!');

// Mock user database
const users = [
  { id: '1', username: 'alice', email: 'alice@example.com', role: 'admin' },
  { id: '2', username: 'bob', email: 'bob@example.com', role: 'user' },
  { id: '3', username: 'charlie', email: 'charlie@example.com', role: 'user' }
];

// JWT secret (in real apps, use environment variables)
const JWT_SECRET = 'your-secret-key';

// Advanced GraphQL Schema
const schema = buildSchema(`
  type User {
    id: ID!
    username: String!
    email: String!
    role: String!
  }
  
  type AuthPayload {
    token: String!
    user: User!
  }
  
  type Query {
    me: User
    users: [User]
    user(id: ID!): User
  }
  
  type Mutation {
    login(username: String!, password: String!): AuthPayload
    register(username: String!, email: String!, password: String!): AuthPayload
  }
`);

// Authentication middleware
function authenticateToken(req) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) return null;
  
  try {
    const decoded = jwt.verify(token, JWT_SECRET);
    return users.find(user => user.id === decoded.userId);
  } catch (error) {
    return null;
  }
}

// Resolvers with authentication
const rootValue = {
  // Query resolvers
  me: (args, context) => {
    console.log('👤 Getting current user...');
    
    if (!context.user) {
      throw new Error('Authentication required! Please login first. 🔐');
    }
    
    return context.user;
  },
  
  users: (args, context) => {
    console.log('👥 Getting all users...');
    
    if (!context.user) {
      throw new Error('Authentication required! Please login first. 🔐');
    }
    
    if (context.user.role !== 'admin') {
      throw new Error('Admin access required! 🚫');
    }
    
    return users;
  },
  
  user: ({ id }, context) => {
    console.log(`👤 Getting user ${id}...`);
    
    if (!context.user) {
      throw new Error('Authentication required! Please login first. 🔐');
    }
    
    // Users can only see their own profile, admins can see all
    if (context.user.id !== id && context.user.role !== 'admin') {
      throw new Error('Access denied! You can only view your own profile. 🚫');
    }
    
    return users.find(user => user.id === id);
  },
  
  // Mutation resolvers
  login: ({ username, password }) => {
    console.log(`🔑 Login attempt for: ${username}`);
    
    // Simple password validation (in real apps, use proper hashing)
    if (password !== 'password123') {
      throw new Error('Invalid credentials! 🔐');
    }
    
    const user = users.find(u => u.username === username);
    if (!user) {
      throw new Error('User not found! 🔍');
    }
    
    // Generate JWT token
    const token = jwt.sign(
      { userId: user.id, username: user.username },
      JWT_SECRET,
      { expiresIn: '24h' }
    );
    
    console.log(`✅ Login successful for: ${username}`);
    
    return {
      token,
      user
    };
  },
  
  register: ({ username, email, password }) => {
    console.log(`📝 Registration attempt for: ${username}`);
    
    // Validation
    if (!username || !email || !password) {
      throw new Error('All fields are required! 📝');
    }
    
    if (password.length < 6) {
      throw new Error('Password must be at least 6 characters! 🔐');
    }
    
    // Check if user already exists
    const existingUser = users.find(u => u.username === username || u.email === email);
    if (existingUser) {
      throw new Error('User already exists! 👥');
    }
    
    // Create new user
    const newUser = {
      id: String(users.length + 1),
      username,
      email,
      role: 'user'
    };
    
    users.push(newUser);
    
    // Generate JWT token
    const token = jwt.sign(
      { userId: newUser.id, username: newUser.username },
      JWT_SECRET,
      { expiresIn: '24h' }
    );
    
    console.log(`✅ Registration successful for: ${username}`);
    
    return {
      token,
      user: newUser
    };
  }
};

// Create Express server with authentication context
const app = express();
const PORT = 4003;

app.use('/graphql', graphqlHTTP((req) => ({
  schema: schema,
  rootValue: rootValue,
  context: {
    user: authenticateToken(req)
  },
  graphiql: true,
  formatError: (error) => {
    console.log('❌ GraphQL Error:', error.message);
    return {
      message: error.message,
      locations: error.locations,
      path: error.path
    };
  }
})));

app.listen(PORT, () => {
  console.log(`🔧 Advanced GraphQL server running on http://localhost:${PORT}/graphql`);
  console.log('\nTry these operations:');
  console.log('1. Register: mutation { register(username: "test", email: "test@example.com", password: "password123") { token user { username } } }');
  console.log('2. Login: mutation { login(username: "alice", password: "password123") { token user { username role } } }');
  console.log('3. Get profile: query { me { username email role } }');
  console.log('4. Set Authorization header: { "Authorization": "Bearer YOUR_TOKEN_HERE" }');
});
```

---

## 🎯 GraphQL vs REST Comparison

### Side-by-Side Comparison

Create `graphql-vs-rest.js`:
```javascript
// graphql-vs-rest.js - GraphQL vs REST comparison
console.log('⚖️ GraphQL vs REST Comparison!');

const examples = {
  // REST API Examples
  rest: {
    description: 'REST API - Multiple requests needed',
    examples: [
      {
        scenario: 'Get user profile with posts and comments',
        requests: [
          'GET /api/users/1',
          'GET /api/users/1/posts',
          'GET /api/posts/1/comments',
          'GET /api/posts/2/comments'
        ],
        issues: [
          'Multiple HTTP requests (N+1 problem)',
          'Over-fetching (getting unused data)',
          'Under-fetching (need more requests)',
          'Waterfall requests (wait for each response)'
        ]
      }
    ]
  },
  
  // GraphQL Examples
  graphql: {
    description: 'GraphQL - Single request gets everything',
    examples: [
      {
        scenario: 'Get user profile with posts and comments',
        query: `
          {
            user(id: "1") {
              name
              email
              posts {
                title
                content
                comments {
                  text
                  author {
                    name
                  }
                }
              }
            }
          }
        `,
        benefits: [
          'Single HTTP request',
          'Get exactly what you need',
          'No over-fetching or under-fetching',
          'Parallel data fetching'
        ]
      }
    ]
  }
};

// Print comparison
console.log('\n📊 COMPARISON SUMMARY:');
console.log('┌─────────────────┬─────────────────┬─────────────────┐');
console.log('│     Feature     │      REST       │     GraphQL     │');
console.log('├─────────────────┼─────────────────┼─────────────────┤');
console.log('│   Requests      │     Multiple    │      Single     │');
console.log('│   Data Fetching │   Over/Under    │     Precise     │');
console.log('│   Flexibility   │      Low        │      High       │');
console.log('│   Caching       │      Easy       │    Complex      │');
console.log('│   Learning      │      Easy       │     Moderate    │');
console.log('│   Tooling       │     Mature      │     Growing     │');
console.log('└─────────────────┴─────────────────┴─────────────────┘');

console.log('\n🎯 WHEN TO USE WHAT:');
console.log('📱 Use REST when:');
console.log('   • Simple CRUD operations');
console.log('   • Heavy caching requirements');
console.log('   • Team is familiar with REST');
console.log('   • File uploads/downloads');

console.log('\n🎨 Use GraphQL when:');
console.log('   • Complex data relationships');
console.log('   • Multiple client types (web, mobile, etc.)');
console.log('   • Rapid frontend development');
console.log('   • Data fetching optimization needed');

// Demo function showing the difference
function demonstrateDifference() {
  console.log('\n🔍 PRACTICAL EXAMPLE:');
  console.log('Scenario: Mobile app showing user profile');
  
  console.log('\n📱 REST Approach:');
  console.log('   1. GET /users/1 → Gets ALL user data (including unused fields)');
  console.log('   2. GET /users/1/posts → Gets ALL post data');
  console.log('   3. Mobile app only uses: name, avatar, post titles');
  console.log('   ❌ Wasted bandwidth, multiple requests');
  
  console.log('\n🎨 GraphQL Approach:');
  console.log('   1. Single query: { user(id: "1") { name avatar posts { title } } }');
  console.log('   2. Gets EXACTLY what mobile app needs');
  console.log('   ✅ Efficient, single request, precise data');
}

demonstrateDifference();
```

---

## 🎉 Summary

Today you learned:
- ✅ What GraphQL is (magical restaurant analogy!)
- ✅ Building GraphQL servers with Node.js
- ✅ Creating schemas, types, and resolvers
- ✅ Querying and mutating data
- ✅ Database integration (MongoDB & SQLite)
- ✅ Advanced features (authentication, validation)
- ✅ GraphQL vs REST comparison
- ✅ When to use each approach

**Next Up**: Day 05 - Part 01: Authentication, Security, and Testing

---

## 🎯 Practice Exercises

### Exercise 1: Build a Blog GraphQL API
Create a GraphQL API for a blog with:
1. Posts, comments, and authors
2. Nested queries (posts with comments and authors)
3. Mutations for creating/updating content
4. Database integration

### Exercise 2: E-commerce GraphQL API
Build a GraphQL API for an online store with:
1. Products, categories, and orders
2. Search and filtering capabilities
3. User authentication
4. Order management

### Exercise 3: Social Media GraphQL API
Create a social media API with:
1. Users, posts, likes, and follows
2. Complex relationships and nested queries
3. Real-time subscriptions (if time permits)
4. Permission-based access control

---

*Remember: GraphQL is like having a smart assistant who brings you exactly what you ask for, nothing more, nothing less! 🎯*
