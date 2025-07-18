# Day 04 - Part 01: REST API Development

## 🎯 Learning Objectives
By the end of this lesson, you will understand:
- What REST APIs are and why they're important
- How to build REST APIs with Express.js
- HTTP methods and status codes
- Request and response handling
- Data validation and error handling
- API testing and documentation
- Real-world REST API patterns

---

## 🌍 What is a REST API?

### Simple Explanation (for 10-year-olds!)

Imagine you have a **magical restaurant** where you can order food by sending messages:

#### **Traditional Restaurant:**
```
👦 You: "I want a pizza"
👨‍🍳 Chef: "What size? What toppings?"
👦 You: "Medium with pepperoni"
👨‍🍳 Chef: "Coming right up!"
```

#### **REST API Restaurant:**
```
📱 App: POST /orders {"food": "pizza", "size": "medium", "toppings": ["pepperoni"]}
🖥️ Server: 201 Created {"orderId": 123, "status": "preparing"}
📱 App: GET /orders/123
🖥️ Server: 200 OK {"orderId": 123, "status": "ready", "price": 15.99}
```

### Real-World Analogy
```
REST API is like a DRIVE-THROUGH WINDOW:
┌─────────────────────────────────────────────────────┐
│  🚗 You drive up (make a request)                   │
│  📢 You place an order (send data)                  │
│  🍟 You get your food (receive response)            │
│  💰 You pay (handle the transaction)                │
│  🚗 You drive away (connection closes)              │
└─────────────────────────────────────────────────────┘
```

### Why REST APIs Are Amazing
- **Universal Language**: Any app can talk to any server
- **Simple Rules**: Easy to understand and use
- **Scalable**: Can handle millions of users
- **Flexible**: Works with web, mobile, and desktop apps

---

## 🛠️ Building Your First REST API

### Setting Up Express Server

Create `basic-rest-api.js`:
```javascript
// basic-rest-api.js - Your first REST API
const express = require('express');
const app = express();
const PORT = 3000;

console.log('🚀 Building your first REST API!');

// Middleware to parse JSON
app.use(express.json());

// Simple data store (in real apps, this would be a database)
let books = [
  { id: 1, title: 'Harry Potter', author: 'J.K. Rowling', year: 1997 },
  { id: 2, title: 'The Hobbit', author: 'J.R.R. Tolkien', year: 1937 },
  { id: 3, title: 'Wonder', author: 'R.J. Palacio', year: 2012 }
];

// GET /books - Get all books
app.get('/books', (req, res) => {
  console.log('📚 Someone wants to see all books!');
  res.json({
    success: true,
    message: 'Books retrieved successfully',
    data: books
  });
});

// GET /books/:id - Get a specific book
app.get('/books/:id', (req, res) => {
  const bookId = parseInt(req.params.id);
  console.log(`📖 Someone wants to see book with ID: ${bookId}`);
  
  const book = books.find(b => b.id === bookId);
  
  if (!book) {
    return res.status(404).json({
      success: false,
      message: 'Book not found! 😢'
    });
  }
  
  res.json({
    success: true,
    message: 'Book found!',
    data: book
  });
});

// POST /books - Add a new book
app.post('/books', (req, res) => {
  console.log('📝 Someone wants to add a new book!');
  
  const { title, author, year } = req.body;
  
  // Simple validation
  if (!title || !author || !year) {
    return res.status(400).json({
      success: false,
      message: 'Please provide title, author, and year! 📝'
    });
  }
  
  // Create new book
  const newBook = {
    id: books.length + 1,
    title,
    author,
    year: parseInt(year)
  };
  
  books.push(newBook);
  
  res.status(201).json({
    success: true,
    message: 'Book added successfully! 🎉',
    data: newBook
  });
});

// PUT /books/:id - Update a book
app.put('/books/:id', (req, res) => {
  const bookId = parseInt(req.params.id);
  console.log(`✏️ Someone wants to update book with ID: ${bookId}`);
  
  const bookIndex = books.findIndex(b => b.id === bookId);
  
  if (bookIndex === -1) {
    return res.status(404).json({
      success: false,
      message: 'Book not found! 😢'
    });
  }
  
  const { title, author, year } = req.body;
  
  // Update book
  books[bookIndex] = {
    ...books[bookIndex],
    title: title || books[bookIndex].title,
    author: author || books[bookIndex].author,
    year: year ? parseInt(year) : books[bookIndex].year
  };
  
  res.json({
    success: true,
    message: 'Book updated successfully! ✨',
    data: books[bookIndex]
  });
});

// DELETE /books/:id - Delete a book
app.delete('/books/:id', (req, res) => {
  const bookId = parseInt(req.params.id);
  console.log(`🗑️ Someone wants to delete book with ID: ${bookId}`);
  
  const bookIndex = books.findIndex(b => b.id === bookId);
  
  if (bookIndex === -1) {
    return res.status(404).json({
      success: false,
      message: 'Book not found! 😢'
    });
  }
  
  const deletedBook = books.splice(bookIndex, 1)[0];
  
  res.json({
    success: true,
    message: 'Book deleted successfully! 🗑️',
    data: deletedBook
  });
});

// Start server
app.listen(PORT, () => {
  console.log(`🌟 REST API server running on http://localhost:${PORT}`);
  console.log('Try these endpoints:');
  console.log('  GET    /books      - Get all books');
  console.log('  GET    /books/:id  - Get specific book');
  console.log('  POST   /books      - Add new book');
  console.log('  PUT    /books/:id  - Update book');
  console.log('  DELETE /books/:id  - Delete book');
});
```

### Testing Your API

Create `test-api.js`:
```javascript
// test-api.js - Testing your REST API
console.log('🧪 Testing our REST API!');

// Note: This is a simple test using fetch
// In real projects, you'd use tools like Postman or automated tests

async function testAPI() {
  const baseURL = 'http://localhost:3000';
  
  try {
    console.log('1. Testing GET /books...');
    let response = await fetch(`${baseURL}/books`);
    let data = await response.json();
    console.log('📚 All books:', data);
    
    console.log('\n2. Testing GET /books/1...');
    response = await fetch(`${baseURL}/books/1`);
    data = await response.json();
    console.log('📖 Book 1:', data);
    
    console.log('\n3. Testing POST /books...');
    response = await fetch(`${baseURL}/books`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        title: 'Charlotte\'s Web',
        author: 'E.B. White',
        year: 1952
      })
    });
    data = await response.json();
    console.log('➕ New book added:', data);
    
    console.log('\n4. Testing PUT /books/1...');
    response = await fetch(`${baseURL}/books/1`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        year: 1998  // Update publication year
      })
    });
    data = await response.json();
    console.log('✏️ Book updated:', data);
    
    console.log('\n5. Testing DELETE /books/2...');
    response = await fetch(`${baseURL}/books/2`, {
      method: 'DELETE'
    });
    data = await response.json();
    console.log('🗑️ Book deleted:', data);
    
    console.log('\n6. Final GET /books...');
    response = await fetch(`${baseURL}/books`);
    data = await response.json();
    console.log('📚 Remaining books:', data);
    
  } catch (error) {
    console.log('❌ Error testing API:', error.message);
  }
}

// Run tests after a delay to let server start
setTimeout(testAPI, 2000);
```

---

## 🎯 HTTP Methods and Status Codes

### HTTP Methods Explained

Create `http-methods-demo.js`:
```javascript
// http-methods-demo.js - Understanding HTTP methods
const express = require('express');
const app = express();
const PORT = 3001;

console.log('🔤 Learning HTTP Methods!');

app.use(express.json());

// Sample data
let students = [
  { id: 1, name: 'Alice', grade: 'A', subject: 'Math' },
  { id: 2, name: 'Bob', grade: 'B', subject: 'Science' },
  { id: 3, name: 'Charlie', grade: 'A', subject: 'History' }
];

// GET - Read/Retrieve data (like asking "What's there?")
app.get('/students', (req, res) => {
  console.log('👀 GET request - Someone wants to READ student data');
  res.json({
    message: 'Here are all the students!',
    data: students
  });
});

// POST - Create new data (like saying "Add this new thing!")
app.post('/students', (req, res) => {
  console.log('➕ POST request - Someone wants to CREATE a new student');
  
  const newStudent = {
    id: students.length + 1,
    name: req.body.name,
    grade: req.body.grade,
    subject: req.body.subject
  };
  
  students.push(newStudent);
  
  res.status(201).json({
    message: 'New student created!',
    data: newStudent
  });
});

// PUT - Update/Replace entire data (like saying "Replace this completely!")
app.put('/students/:id', (req, res) => {
  console.log('🔄 PUT request - Someone wants to UPDATE entire student');
  
  const studentId = parseInt(req.params.id);
  const studentIndex = students.findIndex(s => s.id === studentId);
  
  if (studentIndex === -1) {
    return res.status(404).json({ message: 'Student not found!' });
  }
  
  // Replace entire student data
  students[studentIndex] = {
    id: studentId,
    name: req.body.name,
    grade: req.body.grade,
    subject: req.body.subject
  };
  
  res.json({
    message: 'Student updated completely!',
    data: students[studentIndex]
  });
});

// PATCH - Update part of data (like saying "Just change this one thing!")
app.patch('/students/:id', (req, res) => {
  console.log('🔧 PATCH request - Someone wants to UPDATE part of student');
  
  const studentId = parseInt(req.params.id);
  const studentIndex = students.findIndex(s => s.id === studentId);
  
  if (studentIndex === -1) {
    return res.status(404).json({ message: 'Student not found!' });
  }
  
  // Update only provided fields
  Object.keys(req.body).forEach(key => {
    students[studentIndex][key] = req.body[key];
  });
  
  res.json({
    message: 'Student updated partially!',
    data: students[studentIndex]
  });
});

// DELETE - Remove data (like saying "Take this away!")
app.delete('/students/:id', (req, res) => {
  console.log('🗑️ DELETE request - Someone wants to REMOVE a student');
  
  const studentId = parseInt(req.params.id);
  const studentIndex = students.findIndex(s => s.id === studentId);
  
  if (studentIndex === -1) {
    return res.status(404).json({ message: 'Student not found!' });
  }
  
  const deletedStudent = students.splice(studentIndex, 1)[0];
  
  res.json({
    message: 'Student deleted!',
    data: deletedStudent
  });
});

// START SERVER
app.listen(PORT, () => {
  console.log(`🌟 HTTP Methods Demo running on http://localhost:${PORT}`);
  console.log('\nHTTP Methods Summary:');
  console.log('  GET    = Read/View data    (👀 "Show me!")');
  console.log('  POST   = Create new data   (➕ "Add this!")');
  console.log('  PUT    = Replace all data  (🔄 "Replace everything!")');
  console.log('  PATCH  = Update part       (🔧 "Change just this!")');
  console.log('  DELETE = Remove data       (🗑️ "Take this away!")');
});
```

### Status Codes Explained

Create `status-codes-demo.js`:
```javascript
// status-codes-demo.js - Understanding HTTP status codes
const express = require('express');
const app = express();
const PORT = 3002;

console.log('🚦 Learning HTTP Status Codes!');

app.use(express.json());

// Status codes are like traffic lights for your API!
// 🟢 Green (200s) = Success! Everything is good!
// 🟡 Yellow (300s) = Redirect! Go somewhere else!
// 🔴 Red (400s) = Client Error! You did something wrong!
// 🚨 Red (500s) = Server Error! We messed up!

// 200 - OK (Everything is perfect!)
app.get('/success', (req, res) => {
  console.log('✅ 200 OK - Everything is perfect!');
  res.status(200).json({
    message: 'Success! Everything worked great! 🎉',
    statusCode: 200,
    meaning: 'OK - Request successful'
  });
});

// 201 - Created (New thing was made!)
app.post('/create', (req, res) => {
  console.log('➕ 201 Created - New thing was made!');
  res.status(201).json({
    message: 'Created! New item was added! 🆕',
    statusCode: 201,
    meaning: 'Created - New resource was created'
  });
});

// 400 - Bad Request (You sent wrong data!)
app.post('/bad-request', (req, res) => {
  console.log('❌ 400 Bad Request - Wrong data sent!');
  res.status(400).json({
    message: 'Bad Request! You sent wrong data! 📝',
    statusCode: 400,
    meaning: 'Bad Request - Check your data and try again'
  });
});

// 401 - Unauthorized (You need to login!)
app.get('/unauthorized', (req, res) => {
  console.log('🔐 401 Unauthorized - Need to login!');
  res.status(401).json({
    message: 'Unauthorized! Please login first! 🔐',
    statusCode: 401,
    meaning: 'Unauthorized - You need to authenticate'
  });
});

// 403 - Forbidden (You can\'t do this!)
app.get('/forbidden', (req, res) => {
  console.log('🚫 403 Forbidden - Access denied!');
  res.status(403).json({
    message: 'Forbidden! You don\'t have permission! 🚫',
    statusCode: 403,
    meaning: 'Forbidden - You don\'t have permission'
  });
});

// 404 - Not Found (Thing doesn\'t exist!)
app.get('/not-found', (req, res) => {
  console.log('🔍 404 Not Found - Thing doesn\'t exist!');
  res.status(404).json({
    message: 'Not Found! The thing you\'re looking for doesn\'t exist! 🔍',
    statusCode: 404,
    meaning: 'Not Found - Resource doesn\'t exist'
  });
});

// 500 - Internal Server Error (We messed up!)
app.get('/server-error', (req, res) => {
  console.log('💥 500 Internal Server Error - We messed up!');
  res.status(500).json({
    message: 'Internal Server Error! We messed up! 💥',
    statusCode: 500,
    meaning: 'Internal Server Error - Something went wrong on our end'
  });
});

// Catch all other routes (404)
app.get('*', (req, res) => {
  console.log('🔍 404 - Route not found!');
  res.status(404).json({
    message: 'Page not found! Try a different URL! 🔍',
    statusCode: 404,
    availableRoutes: [
      '/success (200)',
      '/create (201)',
      '/bad-request (400)',
      '/unauthorized (401)',
      '/forbidden (403)',
      '/not-found (404)',
      '/server-error (500)'
    ]
  });
});

app.listen(PORT, () => {
  console.log(`🚦 Status Codes Demo running on http://localhost:${PORT}`);
  console.log('\nStatus Code Summary:');
  console.log('  2xx = Success     (🟢 "Great job!")');
  console.log('  3xx = Redirect    (🟡 "Go here instead!")');
  console.log('  4xx = Client Error (🔴 "You did something wrong!")');
  console.log('  5xx = Server Error (🚨 "We messed up!")');
});
```

---

## 🛡️ Data Validation and Error Handling

### Input Validation

Create `validation-demo.js`:
```javascript
// validation-demo.js - Data validation and error handling
const express = require('express');
const app = express();
const PORT = 3003;

console.log('🛡️ Learning Data Validation!');

app.use(express.json());

// Simple validation helper functions
const validators = {
  isString: (value) => typeof value === 'string' && value.trim().length > 0,
  isNumber: (value) => typeof value === 'number' && !isNaN(value),
  isEmail: (value) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(value);
  },
  isAge: (value) => {
    return validators.isNumber(value) && value >= 0 && value <= 120;
  },
  isGrade: (value) => {
    const validGrades = ['A', 'B', 'C', 'D', 'F'];
    return validGrades.includes(value);
  }
};

// Validation middleware
function validateStudent(req, res, next) {
  console.log('🔍 Validating student data...');
  
  const { name, age, email, grade } = req.body;
  const errors = [];
  
  // Check required fields
  if (!name) {
    errors.push('Name is required! 📝');
  } else if (!validators.isString(name)) {
    errors.push('Name must be a non-empty string! 📝');
  }
  
  if (!age) {
    errors.push('Age is required! 🎂');
  } else if (!validators.isAge(age)) {
    errors.push('Age must be a number between 0 and 120! 🎂');
  }
  
  if (!email) {
    errors.push('Email is required! 📧');
  } else if (!validators.isEmail(email)) {
    errors.push('Email must be a valid email address! 📧');
  }
  
  if (!grade) {
    errors.push('Grade is required! 📊');
  } else if (!validators.isGrade(grade)) {
    errors.push('Grade must be A, B, C, D, or F! 📊');
  }
  
  if (errors.length > 0) {
    return res.status(400).json({
      success: false,
      message: 'Validation failed! Please fix these errors:',
      errors: errors
    });
  }
  
  console.log('✅ Validation passed!');
  next();
}

// Error handling middleware
function errorHandler(err, req, res, next) {
  console.log('💥 Error occurred:', err.message);
  
  res.status(500).json({
    success: false,
    message: 'Something went wrong! 💥',
    error: err.message
  });
}

// Students array
let students = [];

// Create student with validation
app.post('/students', validateStudent, (req, res) => {
  try {
    const { name, age, email, grade } = req.body;
    
    // Check if email already exists
    const existingStudent = students.find(s => s.email === email);
    if (existingStudent) {
      return res.status(409).json({
        success: false,
        message: 'Student with this email already exists! 📧'
      });
    }
    
    const newStudent = {
      id: students.length + 1,
      name,
      age,
      email,
      grade,
      createdAt: new Date().toISOString()
    };
    
    students.push(newStudent);
    
    res.status(201).json({
      success: true,
      message: 'Student created successfully! 🎉',
      data: newStudent
    });
    
  } catch (error) {
    console.log('❌ Error creating student:', error.message);
    res.status(500).json({
      success: false,
      message: 'Failed to create student! 💥'
    });
  }
});

// Get all students
app.get('/students', (req, res) => {
  res.json({
    success: true,
    message: 'Students retrieved successfully!',
    data: students,
    count: students.length
  });
});

// Get student by ID with validation
app.get('/students/:id', (req, res) => {
  try {
    const studentId = parseInt(req.params.id);
    
    if (isNaN(studentId)) {
      return res.status(400).json({
        success: false,
        message: 'Invalid student ID! Must be a number! 🔢'
      });
    }
    
    const student = students.find(s => s.id === studentId);
    
    if (!student) {
      return res.status(404).json({
        success: false,
        message: 'Student not found! 🔍'
      });
    }
    
    res.json({
      success: true,
      message: 'Student found!',
      data: student
    });
    
  } catch (error) {
    console.log('❌ Error finding student:', error.message);
    res.status(500).json({
      success: false,
      message: 'Failed to find student! 💥'
    });
  }
});

// Use error handling middleware
app.use(errorHandler);

app.listen(PORT, () => {
  console.log(`🛡️ Validation Demo running on http://localhost:${PORT}`);
  console.log('\nTry sending POST to /students with:');
  console.log('✅ Valid: {"name": "Alice", "age": 15, "email": "alice@school.com", "grade": "A"}');
  console.log('❌ Invalid: {"name": "", "age": 150, "email": "invalid", "grade": "Z"}');
});
```

---

## 📊 Advanced REST API Features

### Pagination and Filtering

Create `advanced-features.js`:
```javascript
// advanced-features.js - Advanced REST API features
const express = require('express');
const app = express();
const PORT = 3004;

console.log('🚀 Learning Advanced API Features!');

app.use(express.json());

// Sample data - more books for pagination demo
let books = [
  { id: 1, title: 'Harry Potter', author: 'J.K. Rowling', genre: 'Fantasy', year: 1997, rating: 4.8 },
  { id: 2, title: 'The Hobbit', author: 'J.R.R. Tolkien', genre: 'Fantasy', year: 1937, rating: 4.7 },
  { id: 3, title: 'To Kill a Mockingbird', author: 'Harper Lee', genre: 'Classic', year: 1960, rating: 4.5 },
  { id: 4, title: '1984', author: 'George Orwell', genre: 'Dystopian', year: 1949, rating: 4.6 },
  { id: 5, title: 'Pride and Prejudice', author: 'Jane Austen', genre: 'Romance', year: 1813, rating: 4.4 },
  { id: 6, title: 'The Great Gatsby', author: 'F. Scott Fitzgerald', genre: 'Classic', year: 1925, rating: 4.2 },
  { id: 7, title: 'Lord of the Rings', author: 'J.R.R. Tolkien', genre: 'Fantasy', year: 1954, rating: 4.9 },
  { id: 8, title: 'The Catcher in the Rye', author: 'J.D. Salinger', genre: 'Coming-of-age', year: 1951, rating: 4.0 },
  { id: 9, title: 'Animal Farm', author: 'George Orwell', genre: 'Political Fiction', year: 1945, rating: 4.3 },
  { id: 10, title: 'Brave New World', author: 'Aldous Huxley', genre: 'Dystopian', year: 1932, rating: 4.1 }
];

// GET /books with pagination, filtering, and sorting
app.get('/books', (req, res) => {
  try {
    console.log('📚 Advanced book search request received!');
    
    let filteredBooks = [...books];
    
    // 1. FILTERING
    const { genre, author, minRating, maxRating, year } = req.query;
    
    if (genre) {
      filteredBooks = filteredBooks.filter(book => 
        book.genre.toLowerCase().includes(genre.toLowerCase())
      );
      console.log(`🔍 Filtered by genre: ${genre}`);
    }
    
    if (author) {
      filteredBooks = filteredBooks.filter(book => 
        book.author.toLowerCase().includes(author.toLowerCase())
      );
      console.log(`🔍 Filtered by author: ${author}`);
    }
    
    if (minRating) {
      filteredBooks = filteredBooks.filter(book => book.rating >= parseFloat(minRating));
      console.log(`⭐ Filtered by minimum rating: ${minRating}`);
    }
    
    if (maxRating) {
      filteredBooks = filteredBooks.filter(book => book.rating <= parseFloat(maxRating));
      console.log(`⭐ Filtered by maximum rating: ${maxRating}`);
    }
    
    if (year) {
      filteredBooks = filteredBooks.filter(book => book.year === parseInt(year));
      console.log(`📅 Filtered by year: ${year}`);
    }
    
    // 2. SORTING
    const { sortBy, sortOrder } = req.query;
    
    if (sortBy) {
      filteredBooks.sort((a, b) => {
        let aValue = a[sortBy];
        let bValue = b[sortBy];
        
        if (typeof aValue === 'string') {
          aValue = aValue.toLowerCase();
          bValue = bValue.toLowerCase();
        }
        
        if (sortOrder === 'desc') {
          return bValue > aValue ? 1 : -1;
        } else {
          return aValue > bValue ? 1 : -1;
        }
      });
      console.log(`📊 Sorted by ${sortBy} (${sortOrder || 'asc'})`);
    }
    
    // 3. PAGINATION
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 5;
    const startIndex = (page - 1) * limit;
    const endIndex = startIndex + limit;
    
    const paginatedBooks = filteredBooks.slice(startIndex, endIndex);
    
    // 4. RESPONSE WITH METADATA
    const totalBooks = filteredBooks.length;
    const totalPages = Math.ceil(totalBooks / limit);
    const hasNextPage = endIndex < totalBooks;
    const hasPrevPage = startIndex > 0;
    
    console.log(`📄 Page ${page} of ${totalPages} (${paginatedBooks.length} books)`);
    
    res.json({
      success: true,
      message: 'Books retrieved successfully!',
      data: paginatedBooks,
      pagination: {
        currentPage: page,
        totalPages: totalPages,
        totalBooks: totalBooks,
        booksPerPage: limit,
        hasNextPage: hasNextPage,
        hasPrevPage: hasPrevPage
      },
      filters: {
        genre: genre || null,
        author: author || null,
        minRating: minRating || null,
        maxRating: maxRating || null,
        year: year || null
      },
      sorting: {
        sortBy: sortBy || null,
        sortOrder: sortOrder || 'asc'
      }
    });
    
  } catch (error) {
    console.log('❌ Error in advanced search:', error.message);
    res.status(500).json({
      success: false,
      message: 'Search failed! 💥',
      error: error.message
    });
  }
});

// Search books by title
app.get('/books/search', (req, res) => {
  try {
    const { q } = req.query;
    
    if (!q) {
      return res.status(400).json({
        success: false,
        message: 'Search query is required! Use ?q=search_term 🔍'
      });
    }
    
    const searchResults = books.filter(book => 
      book.title.toLowerCase().includes(q.toLowerCase()) ||
      book.author.toLowerCase().includes(q.toLowerCase())
    );
    
    console.log(`🔍 Search for "${q}" found ${searchResults.length} results`);
    
    res.json({
      success: true,
      message: `Search results for "${q}"`,
      data: searchResults,
      count: searchResults.length
    });
    
  } catch (error) {
    console.log('❌ Error in search:', error.message);
    res.status(500).json({
      success: false,
      message: 'Search failed! 💥'
    });
  }
});

app.listen(PORT, () => {
  console.log(`🚀 Advanced Features Demo running on http://localhost:${PORT}`);
  console.log('\nTry these advanced features:');
  console.log('  📄 Pagination: /books?page=2&limit=3');
  console.log('  🔍 Filtering: /books?genre=Fantasy&minRating=4.5');
  console.log('  📊 Sorting: /books?sortBy=rating&sortOrder=desc');
  console.log('  🔍 Search: /books/search?q=Harry');
  console.log('  🎯 Combined: /books?genre=Fantasy&sortBy=year&page=1&limit=2');
});
```

---

## 🧪 API Testing Made Easy

### Simple API Testing

Create `api-testing.js`:
```javascript
// api-testing.js - Simple API testing
console.log('🧪 Learning API Testing!');

// Simple test framework
class SimpleTest {
  constructor() {
    this.tests = [];
    this.passed = 0;
    this.failed = 0;
  }
  
  async test(name, testFunction) {
    try {
      console.log(`\n🧪 Testing: ${name}`);
      await testFunction();
      console.log(`✅ PASSED: ${name}`);
      this.passed++;
    } catch (error) {
      console.log(`❌ FAILED: ${name}`);
      console.log(`   Error: ${error.message}`);
      this.failed++;
    }
  }
  
  async run() {
    console.log('🚀 Starting API Tests...\n');
    
    // Test 1: GET all books
    await this.test('GET /books should return all books', async () => {
      const response = await fetch('http://localhost:3000/books');
      const data = await response.json();
      
      if (response.status !== 200) {
        throw new Error(`Expected status 200, got ${response.status}`);
      }
      
      if (!data.success) {
        throw new Error('Expected success to be true');
      }
      
      if (!Array.isArray(data.data)) {
        throw new Error('Expected data to be an array');
      }
      
      console.log(`   📊 Found ${data.data.length} books`);
    });
    
    // Test 2: GET specific book
    await this.test('GET /books/1 should return specific book', async () => {
      const response = await fetch('http://localhost:3000/books/1');
      const data = await response.json();
      
      if (response.status !== 200) {
        throw new Error(`Expected status 200, got ${response.status}`);
      }
      
      if (!data.success) {
        throw new Error('Expected success to be true');
      }
      
      if (!data.data || !data.data.id) {
        throw new Error('Expected book data with id');
      }
      
      console.log(`   📖 Found book: ${data.data.title}`);
    });
    
    // Test 3: POST new book
    await this.test('POST /books should create new book', async () => {
      const newBook = {
        title: 'Test Book',
        author: 'Test Author',
        year: 2024
      };
      
      const response = await fetch('http://localhost:3000/books', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newBook)
      });
      
      const data = await response.json();
      
      if (response.status !== 201) {
        throw new Error(`Expected status 201, got ${response.status}`);
      }
      
      if (!data.success) {
        throw new Error('Expected success to be true');
      }
      
      if (data.data.title !== newBook.title) {
        throw new Error('Book title doesn\'t match');
      }
      
      console.log(`   📚 Created book: ${data.data.title}`);
    });
    
    // Test 4: GET non-existent book
    await this.test('GET /books/999 should return 404', async () => {
      const response = await fetch('http://localhost:3000/books/999');
      const data = await response.json();
      
      if (response.status !== 404) {
        throw new Error(`Expected status 404, got ${response.status}`);
      }
      
      if (data.success !== false) {
        throw new Error('Expected success to be false');
      }
      
      console.log(`   🔍 Correctly returned 404 for non-existent book`);
    });
    
    // Test 5: POST with invalid data
    await this.test('POST /books with invalid data should return 400', async () => {
      const invalidBook = { title: 'Only Title' }; // Missing required fields
      
      const response = await fetch('http://localhost:3000/books', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(invalidBook)
      });
      
      const data = await response.json();
      
      if (response.status !== 400) {
        throw new Error(`Expected status 400, got ${response.status}`);
      }
      
      if (data.success !== false) {
        throw new Error('Expected success to be false');
      }
      
      console.log(`   ❌ Correctly returned 400 for invalid data`);
    });
    
    // Print results
    console.log('\n🏁 Test Results:');
    console.log(`   ✅ Passed: ${this.passed}`);
    console.log(`   ❌ Failed: ${this.failed}`);
    console.log(`   📊 Total: ${this.passed + this.failed}`);
    
    if (this.failed === 0) {
      console.log('   🎉 All tests passed!');
    } else {
      console.log('   🔧 Some tests failed. Check the errors above.');
    }
  }
}

// Run tests
const tester = new SimpleTest();
setTimeout(() => tester.run(), 3000); // Wait for server to start
```

---

## 🎉 Summary

Today you learned:
- ✅ What REST APIs are (drive-through window analogy!)
- ✅ How to build REST APIs with Express.js
- ✅ HTTP methods (GET, POST, PUT, PATCH, DELETE)
- ✅ Status codes (traffic lights for APIs!)
- ✅ Data validation and error handling
- ✅ Advanced features (pagination, filtering, sorting)
- ✅ API testing techniques
- ✅ Real-world REST API patterns

**Next Up**: Day 04 - Part 02: GraphQL Introduction and Database Integration

---

## 🎯 Practice Exercises

### Exercise 1: Build a Todo API
Create a REST API for managing todos with:
1. CRUD operations (Create, Read, Update, Delete)
2. Status filtering (completed/pending)
3. Due date sorting
4. Data validation

### Exercise 2: User Management System
Build an API for users with:
1. User registration and profiles
2. Input validation (email, age, etc.)
3. Search and pagination
4. Error handling

### Exercise 3: Simple Blog API
Create a blog API with:
1. Posts and comments
2. Category filtering
3. Search functionality
4. Proper HTTP status codes

---

*Remember: REST APIs are like well-organized libraries - everything has its place, and there are clear rules for finding and managing information! 📚*
