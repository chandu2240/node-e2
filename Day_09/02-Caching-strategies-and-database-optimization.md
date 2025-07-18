# Day 09 - Part 02: Caching Strategies and Database Optimization

## 🎯 Learning Objectives
By the end of this lesson, you will understand:
- Different caching strategies and when to use them
- In-memory caching with Redis
- Database query optimization techniques
- Connection pooling and database best practices
- CDN and static asset optimization
- Cache invalidation strategies

---

## 💾 Understanding Caching

### Simple Explanation (for 10-year-olds!)

#### **Caching - Like Having a Super Fast Snack Drawer:**
```
🍪 SLOW WAY (No Cache)
┌─────────────────────────────────────────────────────┐
│  When you want cookies:                             │
│  1. 🚶 Walk to the kitchen                         │
│  2. 🔍 Look through all cabinets                   │
│  3. 📦 Open cookie jar                             │
│  4. 🍪 Get cookies                                 │
│  5. 🚶 Walk back to room                           │
│  ⏱️ Takes 5 minutes every time!                    │
└─────────────────────────────────────────────────────┘
```

```
🍪 FAST WAY (With Cache)
┌─────────────────────────────────────────────────────┐
│  When you want cookies:                             │
│  1. 🏃 Open desk drawer (cache)                     │
│  2. 🍪 Grab cookies instantly!                      │
│  ⏱️ Takes 5 seconds!                               │
│                                                     │
│  🔄 Refill drawer when empty                       │
│  (Cache refresh when needed)                        │
└─────────────────────────────────────────────────────┘
```

#### **Real-World Cache Analogy:**
```
📚 LIBRARY CACHE SYSTEM
┌─────────────────────────────────────────────────────┐
│  🏃 Quick Access Shelf (RAM Cache)                  │
│  • Most popular books                               │
│  • Super fast to get                               │
│  • Limited space                                   │
│                                                     │
│  🚶 Regular Shelves (Database)                      │
│  • All books                                       │
│  • Slower to find                                  │
│  • Unlimited space                                 │
│                                                     │
│  🏠 Warehouse (Cold Storage)                        │
│  • Rarely used books                               │
│  • Very slow to get                                │
│  • Huge space                                      │
└─────────────────────────────────────────────────────┘
```

---

## 🚀 In-Memory Caching

### Basic In-Memory Cache Implementation

Create `src/caching-strategies.ts`:
```typescript
// src/caching-strategies.ts - Caching strategies implementation
console.log('💾 Learning Caching Strategies!');

// 🧠 BASIC IN-MEMORY CACHE
class SimpleCache<T> {
  private cache: Map<string, { value: T; timestamp: number; ttl: number }> = new Map();
  
  set(key: string, value: T, ttlSeconds: number = 300): void {
    const ttl = ttlSeconds * 1000; // Convert to milliseconds
    const timestamp = Date.now();
    
    this.cache.set(key, { value, timestamp, ttl });
    console.log(`💾 Cached '${key}' for ${ttlSeconds} seconds`);
  }
  
  get(key: string): T | undefined {
    const item = this.cache.get(key);
    
    if (!item) {
      console.log(`❌ Cache miss for '${key}'`);
      return undefined;
    }
    
    // Check if expired
    const now = Date.now();
    if (now - item.timestamp > item.ttl) {
      this.cache.delete(key);
      console.log(`⏰ Cache expired for '${key}'`);
      return undefined;
    }
    
    console.log(`✅ Cache hit for '${key}'`);
    return item.value;
  }
  
  delete(key: string): boolean {
    const deleted = this.cache.delete(key);
    if (deleted) {
      console.log(`🗑️ Deleted '${key}' from cache`);
    }
    return deleted;
  }
  
  clear(): void {
    this.cache.clear();
    console.log('🧹 Cache cleared');
  }
  
  size(): number {
    return this.cache.size;
  }
  
  // Clean up expired entries
  cleanup(): void {
    const now = Date.now();
    let cleaned = 0;
    
    for (const [key, item] of this.cache.entries()) {
      if (now - item.timestamp > item.ttl) {
        this.cache.delete(key);
        cleaned++;
      }
    }
    
    console.log(`🧹 Cleaned ${cleaned} expired entries`);
  }
}

// 🎯 LRU (Least Recently Used) CACHE
class LRUCache<T> {
  private cache: Map<string, T> = new Map();
  private maxSize: number;
  
  constructor(maxSize: number) {
    this.maxSize = maxSize;
  }
  
  get(key: string): T | undefined {
    const value = this.cache.get(key);
    
    if (value !== undefined) {
      // Move to end (most recently used)
      this.cache.delete(key);
      this.cache.set(key, value);
      console.log(`✅ LRU cache hit for '${key}'`);
    } else {
      console.log(`❌ LRU cache miss for '${key}'`);
    }
    
    return value;
  }
  
  set(key: string, value: T): void {
    if (this.cache.has(key)) {
      // Update existing key
      this.cache.delete(key);
    } else if (this.cache.size >= this.maxSize) {
      // Remove least recently used (first item)
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
      console.log(`🗑️ LRU evicted '${firstKey}'`);
    }
    
    this.cache.set(key, value);
    console.log(`💾 LRU cached '${key}' (size: ${this.cache.size}/${this.maxSize})`);
  }
  
  size(): number {
    return this.cache.size;
  }
}

// 🔄 WRITE-THROUGH CACHE
class WriteThroughCache<T> {
  private cache: SimpleCache<T>;
  private database: Map<string, T>; // Simulated database
  
  constructor() {
    this.cache = new SimpleCache<T>();
    this.database = new Map<string, T>();
  }
  
  async get(key: string): Promise<T | undefined> {
    // Try cache first
    let value = this.cache.get(key);
    
    if (value === undefined) {
      // Cache miss - get from database
      console.log(`📀 Loading '${key}' from database...`);
      await this.simulateDbDelay();
      
      value = this.database.get(key);
      
      if (value !== undefined) {
        // Cache the value
        this.cache.set(key, value, 600); // 10 minutes
        console.log(`💾 Cached '${key}' from database`);
      }
    }
    
    return value;
  }
  
  async set(key: string, value: T): Promise<void> {
    console.log(`💾 Write-through: Setting '${key}'...`);
    
    // Write to database first
    await this.simulateDbDelay();
    this.database.set(key, value);
    console.log(`📀 Saved '${key}' to database`);
    
    // Update cache
    this.cache.set(key, value, 600);
    console.log(`💾 Updated cache for '${key}'`);
  }
  
  async delete(key: string): Promise<boolean> {
    console.log(`🗑️ Write-through: Deleting '${key}'...`);
    
    // Delete from database
    await this.simulateDbDelay();
    const dbDeleted = this.database.delete(key);
    
    // Delete from cache
    const cacheDeleted = this.cache.delete(key);
    
    console.log(`📀 Deleted '${key}' from database: ${dbDeleted}`);
    return dbDeleted;
  }
  
  private async simulateDbDelay(): Promise<void> {
    await new Promise(resolve => setTimeout(resolve, 50));
  }
}

// 🌊 WRITE-BACK CACHE
class WriteBackCache<T> {
  private cache: Map<string, { value: T; dirty: boolean }> = new Map();
  private database: Map<string, T> = new Map();
  private flushInterval: NodeJS.Timer;
  
  constructor(flushIntervalMs: number = 5000) {
    this.flushInterval = setInterval(() => {
      this.flush();
    }, flushIntervalMs);
  }
  
  async get(key: string): Promise<T | undefined> {
    const cached = this.cache.get(key);
    
    if (cached) {
      console.log(`✅ Write-back cache hit for '${key}'`);
      return cached.value;
    }
    
    // Load from database
    console.log(`📀 Loading '${key}' from database...`);
    await this.simulateDbDelay();
    const value = this.database.get(key);
    
    if (value !== undefined) {
      this.cache.set(key, { value, dirty: false });
      console.log(`💾 Cached '${key}' from database`);
    }
    
    return value;
  }
  
  set(key: string, value: T): void {
    console.log(`💾 Write-back: Setting '${key}' (dirty)`);
    this.cache.set(key, { value, dirty: true });
  }
  
  async flush(): Promise<void> {
    const dirtyEntries = Array.from(this.cache.entries())
      .filter(([key, item]) => item.dirty);
    
    if (dirtyEntries.length === 0) {
      return;
    }
    
    console.log(`🔄 Flushing ${dirtyEntries.length} dirty entries to database...`);
    
    for (const [key, item] of dirtyEntries) {
      await this.simulateDbDelay();
      this.database.set(key, item.value);
      this.cache.set(key, { value: item.value, dirty: false });
      console.log(`📀 Flushed '${key}' to database`);
    }
  }
  
  async close(): Promise<void> {
    clearInterval(this.flushInterval);
    await this.flush();
    console.log('🔒 Write-back cache closed');
  }
  
  private async simulateDbDelay(): Promise<void> {
    await new Promise(resolve => setTimeout(resolve, 30));
  }
}

// 🎯 CACHE PATTERNS DEMO
class CachePatternDemo {
  private simpleCache = new SimpleCache<string>();
  private lruCache = new LRUCache<string>(3);
  private writeThroughCache = new WriteThroughCache<string>();
  private writeBackCache = new WriteBackCache<string>();
  
  async demonstrateSimpleCache(): Promise<void> {
    console.log('\n🧠 Simple Cache Demo:');
    
    // Set values
    this.simpleCache.set('user:1', 'Alice', 2); // 2 seconds TTL
    this.simpleCache.set('user:2', 'Bob', 5);   // 5 seconds TTL
    
    // Get values
    console.log(`User 1: ${this.simpleCache.get('user:1')}`);
    console.log(`User 2: ${this.simpleCache.get('user:2')}`);
    
    // Wait and check expiration
    await new Promise(resolve => setTimeout(resolve, 3000));
    console.log('\n⏰ After 3 seconds:');
    console.log(`User 1: ${this.simpleCache.get('user:1')}`); // Should be expired
    console.log(`User 2: ${this.simpleCache.get('user:2')}`); // Should still exist
  }
  
  demonstrateLRUCache(): void {
    console.log('\n🎯 LRU Cache Demo (max size: 3):');
    
    // Fill cache
    this.lruCache.set('a', 'Value A');
    this.lruCache.set('b', 'Value B');
    this.lruCache.set('c', 'Value C');
    
    // Access 'a' to make it recently used
    this.lruCache.get('a');
    
    // Add another item - should evict 'b'
    this.lruCache.set('d', 'Value D');
    
    // Check what's still cached
    console.log(`A: ${this.lruCache.get('a')}`); // Should hit
    console.log(`B: ${this.lruCache.get('b')}`); // Should miss (evicted)
    console.log(`C: ${this.lruCache.get('c')}`); // Should hit
    console.log(`D: ${this.lruCache.get('d')}`); // Should hit
  }
  
  async demonstrateWriteThroughCache(): Promise<void> {
    console.log('\n🔄 Write-Through Cache Demo:');
    
    // Set data (writes to both cache and database)
    await this.writeThroughCache.set('product:1', 'Laptop');
    await this.writeThroughCache.set('product:2', 'Mouse');
    
    // Get data (should hit cache)
    console.log(`Product 1: ${await this.writeThroughCache.get('product:1')}`);
    console.log(`Product 2: ${await this.writeThroughCache.get('product:2')}`);
    
    // Get non-existent data
    console.log(`Product 3: ${await this.writeThroughCache.get('product:3')}`);
  }
  
  async demonstrateWriteBackCache(): Promise<void> {
    console.log('\n🌊 Write-Back Cache Demo:');
    
    // Set data (writes to cache only, database later)
    this.writeBackCache.set('order:1', 'Order A');
    this.writeBackCache.set('order:2', 'Order B');
    
    // Get data immediately
    console.log(`Order 1: ${await this.writeBackCache.get('order:1')}`);
    console.log(`Order 2: ${await this.writeBackCache.get('order:2')}`);
    
    // Wait for automatic flush
    console.log('\n⏳ Waiting for automatic flush...');
    await new Promise(resolve => setTimeout(resolve, 6000));
    
    // Manual flush
    await this.writeBackCache.flush();
    await this.writeBackCache.close();
  }
  
  async runAllDemos(): Promise<void> {
    console.log('🎮 Cache Patterns Demo Starting!\n');
    
    await this.demonstrateSimpleCache();
    this.demonstrateLRUCache();
    await this.demonstrateWriteThroughCache();
    await this.demonstrateWriteBackCache();
    
    console.log('\n🎉 All cache demos completed!');
  }
}

// 🎯 CACHE PERFORMANCE COMPARISON
class CachePerformanceTest {
  private cache = new SimpleCache<number>();
  private database = new Map<string, number>();
  
  async runPerformanceTest(): Promise<void> {
    console.log('\n📊 Cache Performance Test:');
    
    // Populate database
    for (let i = 0; i < 1000; i++) {
      this.database.set(`item:${i}`, i * 10);
    }
    
    // Test without cache
    console.log('\n❌ Without Cache:');
    const startWithoutCache = Date.now();
    
    for (let i = 0; i < 100; i++) {
      const key = `item:${Math.floor(Math.random() * 1000)}`;
      await this.getFromDatabase(key);
    }
    
    const endWithoutCache = Date.now();
    console.log(`   Time: ${endWithoutCache - startWithoutCache}ms`);
    
    // Test with cache
    console.log('\n✅ With Cache:');
    const startWithCache = Date.now();
    
    for (let i = 0; i < 100; i++) {
      const key = `item:${Math.floor(Math.random() * 1000)}`;
      await this.getFromCache(key);
    }
    
    const endWithCache = Date.now();
    console.log(`   Time: ${endWithCache - startWithCache}ms`);
    
    // Calculate improvement
    const improvement = ((endWithoutCache - startWithoutCache) / (endWithCache - startWithCache)).toFixed(2);
    console.log(`\n🚀 Cache made it ${improvement}x faster!`);
  }
  
  private async getFromDatabase(key: string): Promise<number | undefined> {
    // Simulate database delay
    await new Promise(resolve => setTimeout(resolve, 10));
    return this.database.get(key);
  }
  
  private async getFromCache(key: string): Promise<number | undefined> {
    // Try cache first
    let value = this.cache.get(key);
    
    if (value === undefined) {
      // Cache miss - get from database
      value = await this.getFromDatabase(key);
      if (value !== undefined) {
        this.cache.set(key, value, 30); // Cache for 30 seconds
      }
    }
    
    return value;
  }
}

// 🎮 RUN ALL DEMOS
async function runCachingDemo(): Promise<void> {
  console.log('🎮 Caching Strategies Demo Starting!\n');
  
  const demo = new CachePatternDemo();
  await demo.runAllDemos();
  
  const perfTest = new CachePerformanceTest();
  await perfTest.runPerformanceTest();
  
  console.log('\n📚 Key Takeaways:');
  console.log('   ✅ Cache frequently accessed data');
  console.log('   ✅ Use TTL to prevent stale data');
  console.log('   ✅ LRU cache for memory-limited scenarios');
  console.log('   ✅ Write-through for data consistency');
  console.log('   ✅ Write-back for write-heavy workloads');
  console.log('   ✅ Monitor cache hit rates');
  console.log('\n🎉 Caching demo complete!');
}

// Run the demo
runCachingDemo().catch(console.error);
```

---

## 🗄️ Database Optimization

### Database Query Optimization

Create `src/database-optimization.ts`:
```typescript
// src/database-optimization.ts - Database optimization techniques
console.log('🗄️ Learning Database Optimization!');

// 🎯 DATABASE QUERY OPTIMIZER
class DatabaseOptimizer {
  private queryCache = new Map<string, any>();
  private connectionPool: Connection[] = [];
  private readonly maxConnections = 10;
  
  constructor() {
    this.initializeConnectionPool();
  }
  
  // 🔌 CONNECTION POOLING
  private initializeConnectionPool(): void {
    console.log('🔌 Initializing connection pool...');
    
    for (let i = 0; i < this.maxConnections; i++) {
      this.connectionPool.push(new Connection(i));
    }
    
    console.log(`✅ Connection pool initialized with ${this.maxConnections} connections`);
  }
  
  private async getConnection(): Promise<Connection> {
    // Wait for available connection
    while (this.connectionPool.length === 0) {
      await new Promise(resolve => setTimeout(resolve, 10));
    }
    
    const connection = this.connectionPool.pop()!;
    console.log(`📞 Got connection ${connection.id} (${this.connectionPool.length} remaining)`);
    return connection;
  }
  
  private releaseConnection(connection: Connection): void {
    this.connectionPool.push(connection);
    console.log(`📞 Released connection ${connection.id} (${this.connectionPool.length} available)`);
  }
  
  // 📊 QUERY OPTIMIZATION EXAMPLES
  
  // BAD: N+1 Query Problem
  async getBooksWithAuthorsBad(bookIds: number[]): Promise<any[]> {
    console.log('❌ Bad: N+1 Query Problem');
    const connection = await this.getConnection();
    
    try {
      const books = [];
      
      // 1 query to get books
      const bookQuery = `SELECT * FROM books WHERE id IN (${bookIds.join(',')})`;
      const bookResults = await connection.query(bookQuery);
      
      // N queries to get authors (BAD!)
      for (const book of bookResults) {
        const authorQuery = `SELECT * FROM authors WHERE id = ${book.author_id}`;
        const authorResult = await connection.query(authorQuery);
        books.push({ ...book, author: authorResult[0] });
      }
      
      return books;
    } finally {
      this.releaseConnection(connection);
    }
  }
  
  // GOOD: Single JOIN Query
  async getBooksWithAuthorsGood(bookIds: number[]): Promise<any[]> {
    console.log('✅ Good: Single JOIN Query');
    const connection = await this.getConnection();
    
    try {
      const query = `
        SELECT b.*, a.name as author_name, a.email as author_email
        FROM books b
        JOIN authors a ON b.author_id = a.id
        WHERE b.id IN (${bookIds.join(',')})
      `;
      
      const results = await connection.query(query);
      return results;
    } finally {
      this.releaseConnection(connection);
    }
  }
  
  // 🎯 QUERY CACHING
  async getCachedQuery(query: string): Promise<any[]> {
    const cacheKey = `query:${Buffer.from(query).toString('base64')}`;
    
    // Check cache first
    if (this.queryCache.has(cacheKey)) {
      console.log('✅ Query cache hit');
      return this.queryCache.get(cacheKey);
    }
    
    console.log('❌ Query cache miss - executing query');
    const connection = await this.getConnection();
    
    try {
      const results = await connection.query(query);
      
      // Cache the results
      this.queryCache.set(cacheKey, results);
      
      // Auto-expire cache after 5 minutes
      setTimeout(() => {
        this.queryCache.delete(cacheKey);
        console.log('⏰ Query cache expired');
      }, 5 * 60 * 1000);
      
      return results;
    } finally {
      this.releaseConnection(connection);
    }
  }
  
  // 📈 BATCH OPERATIONS
  async batchInsertUsers(users: Array<{name: string, email: string}>): Promise<void> {
    console.log('📈 Batch insert users');
    const connection = await this.getConnection();
    
    try {
      // BAD: Multiple INSERT statements
      console.log('❌ Bad way: Multiple inserts');
      const startBad = Date.now();
      
      for (const user of users.slice(0, 3)) { // Just test with first 3
        const query = `INSERT INTO users (name, email) VALUES ('${user.name}', '${user.email}')`;
        await connection.query(query);
      }
      
      const endBad = Date.now();
      console.log(`   Multiple inserts took: ${endBad - startBad}ms`);
      
      // GOOD: Single batch INSERT
      console.log('✅ Good way: Batch insert');
      const startGood = Date.now();
      
      const values = users.map(user => `('${user.name}', '${user.email}')`).join(', ');
      const batchQuery = `INSERT INTO users (name, email) VALUES ${values}`;
      await connection.query(batchQuery);
      
      const endGood = Date.now();
      console.log(`   Batch insert took: ${endGood - startGood}ms`);
      
    } finally {
      this.releaseConnection(connection);
    }
  }
  
  // 🔍 INDEX OPTIMIZATION
  async demonstrateIndexing(): Promise<void> {
    console.log('🔍 Index Optimization Demo');
    const connection = await this.getConnection();
    
    try {
      // Create test table
      await connection.query(`
        CREATE TABLE IF NOT EXISTS products (
          id INTEGER PRIMARY KEY,
          name TEXT,
          category TEXT,
          price REAL,
          created_at TIMESTAMP
        )
      `);
      
      // Insert test data
      console.log('📝 Inserting test data...');
      for (let i = 0; i < 1000; i++) {
        await connection.query(`
          INSERT INTO products (name, category, price, created_at)
          VALUES ('Product ${i}', 'Category ${i % 10}', ${Math.random() * 100}, datetime('now'))
        `);
      }
      
      // Query without index
      console.log('❌ Query without index:');
      const start1 = Date.now();
      await connection.query(`SELECT * FROM products WHERE category = 'Category 5'`);
      const end1 = Date.now();
      console.log(`   Time: ${end1 - start1}ms`);
      
      // Create index
      console.log('🔍 Creating index...');
      await connection.query(`CREATE INDEX IF NOT EXISTS idx_category ON products(category)`);
      
      // Query with index
      console.log('✅ Query with index:');
      const start2 = Date.now();
      await connection.query(`SELECT * FROM products WHERE category = 'Category 5'`);
      const end2 = Date.now();
      console.log(`   Time: ${end2 - start2}ms`);
      
      const improvement = ((end1 - start1) / (end2 - start2)).toFixed(2);
      console.log(`🚀 Index made it ${improvement}x faster!`);
      
    } finally {
      this.releaseConnection(connection);
    }
  }
  
  // 🎯 QUERY OPTIMIZATION TIPS
  async demonstrateQueryOptimization(): Promise<void> {
    console.log('🎯 Query Optimization Tips');
    const connection = await this.getConnection();
    
    try {
      // 1. Use LIMIT for pagination
      console.log('📄 Pagination with LIMIT:');
      const page = 2;
      const pageSize = 10;
      const offset = (page - 1) * pageSize;
      
      const paginatedQuery = `
        SELECT * FROM products 
        ORDER BY created_at DESC 
        LIMIT ${pageSize} OFFSET ${offset}
      `;
      
      const results = await connection.query(paginatedQuery);
      console.log(`   Retrieved ${results.length} products for page ${page}`);
      
      // 2. Use specific columns instead of SELECT *
      console.log('🎯 Select specific columns:');
      const specificQuery = `
        SELECT name, price FROM products 
        WHERE category = 'Category 1' 
        LIMIT 5
      `;
      
      const specificResults = await connection.query(specificQuery);
      console.log(`   Retrieved ${specificResults.length} products with specific columns`);
      
      // 3. Use WHERE clauses effectively
      console.log('🔍 Effective WHERE clauses:');
      const whereQuery = `
        SELECT * FROM products 
        WHERE price BETWEEN 10 AND 50 
        AND category IN ('Category 1', 'Category 2')
        ORDER BY price ASC
        LIMIT 10
      `;
      
      const whereResults = await connection.query(whereQuery);
      console.log(`   Found ${whereResults.length} products in price range`);
      
    } finally {
      this.releaseConnection(connection);
    }
  }
  
  // 📊 PERFORMANCE MONITORING
  async monitorQueryPerformance(): Promise<void> {
    console.log('📊 Query Performance Monitoring');
    const connection = await this.getConnection();
    
    try {
      const queries = [
        'SELECT COUNT(*) FROM products',
        'SELECT AVG(price) FROM products',
        'SELECT category, COUNT(*) FROM products GROUP BY category',
        'SELECT * FROM products WHERE price > 50 ORDER BY price DESC LIMIT 10'
      ];
      
      for (const query of queries) {
        const start = Date.now();
        const results = await connection.query(query);
        const end = Date.now();
        
        console.log(`📊 Query: ${query.substring(0, 50)}...`);
        console.log(`   Time: ${end - start}ms, Rows: ${results.length}`);
      }
      
    } finally {
      this.releaseConnection(connection);
    }
  }
}

// 🔌 SIMULATED DATABASE CONNECTION
class Connection {
  public id: number;
  
  constructor(id: number) {
    this.id = id;
  }
  
  async query(sql: string): Promise<any[]> {
    // Simulate database query delay
    await new Promise(resolve => setTimeout(resolve, Math.random() * 50));
    
    // Simulate different query results
    if (sql.includes('SELECT COUNT(*)')) {
      return [{ 'COUNT(*)': 1000 }];
    } else if (sql.includes('SELECT AVG(price)')) {
      return [{ 'AVG(price)': 45.67 }];
    } else if (sql.includes('GROUP BY category')) {
      return [
        { category: 'Category 1', 'COUNT(*)': 100 },
        { category: 'Category 2', 'COUNT(*)': 150 }
      ];
    } else if (sql.includes('books') && sql.includes('JOIN')) {
      return [
        { id: 1, title: 'Book 1', author_name: 'Author 1' },
        { id: 2, title: 'Book 2', author_name: 'Author 2' }
      ];
    } else {
      // Generic result
      return Array.from({ length: Math.floor(Math.random() * 20) + 1 }, (_, i) => ({
        id: i + 1,
        name: `Item ${i + 1}`,
        value: Math.random() * 100
      }));
    }
  }
}

// 🎮 RUN DATABASE OPTIMIZATION DEMO
async function runDatabaseOptimizationDemo(): Promise<void> {
  console.log('🎮 Database Optimization Demo Starting!\n');
  
  const optimizer = new DatabaseOptimizer();
  
  // Wait for connection pool to initialize
  await new Promise(resolve => setTimeout(resolve, 100));
  
  // Demo N+1 problem
  console.log('📚 Books with Authors Demo:');
  await optimizer.getBooksWithAuthorsBad([1, 2, 3]);
  await optimizer.getBooksWithAuthorsGood([1, 2, 3]);
  
  // Demo query caching
  console.log('\n💾 Query Caching Demo:');
  const testQuery = 'SELECT * FROM products LIMIT 10';
  await optimizer.getCachedQuery(testQuery);
  await optimizer.getCachedQuery(testQuery); // Should hit cache
  
  // Demo batch operations
  console.log('\n📈 Batch Operations Demo:');
  const testUsers = [
    { name: 'Alice', email: 'alice@example.com' },
    { name: 'Bob', email: 'bob@example.com' },
    { name: 'Charlie', email: 'charlie@example.com' }
  ];
  await optimizer.batchInsertUsers(testUsers);
  
  // Demo indexing
  console.log('\n🔍 Indexing Demo:');
  await optimizer.demonstrateIndexing();
  
  // Demo query optimization
  console.log('\n🎯 Query Optimization Demo:');
  await optimizer.demonstrateQueryOptimization();
  
  // Demo performance monitoring
  console.log('\n📊 Performance Monitoring Demo:');
  await optimizer.monitorQueryPerformance();
  
  console.log('\n📚 Database Optimization Key Takeaways:');
  console.log('   ✅ Use connection pooling');
  console.log('   ✅ Avoid N+1 query problems');
  console.log('   ✅ Cache frequently used queries');
  console.log('   ✅ Use batch operations for multiple inserts');
  console.log('   ✅ Create indexes on frequently queried columns');
  console.log('   ✅ Use specific columns instead of SELECT *');
  console.log('   ✅ Monitor query performance');
  console.log('\n🎉 Database optimization demo complete!');
}

// Run the demo
runDatabaseOptimizationDemo().catch(console.error);
```

---

## 🎯 Summary

In Part 02, you learned:
- ✅ Different caching strategies (like having a super fast snack drawer!)
- ✅ In-memory caching with TTL and LRU eviction
- ✅ Write-through and write-back caching patterns
- ✅ Database connection pooling and optimization
- ✅ Query optimization techniques and indexing
- ✅ Performance monitoring and measurement

**Next up: Part 03 - Monitoring and Observability!** 🚀

---

## 🎉 Fun Caching and Database Facts for Kids

- 💾 Caching is like having a magic drawer that gives you things instantly!
- 🔍 Database indexes are like the index in the back of a book!
- 🔌 Connection pooling is like sharing toys instead of everyone having their own!
- 📊 Query optimization is like finding the shortest path to your destination!
- 🎯 Batch operations are like carrying multiple things at once!

---

*Remember: Good caching and database optimization is like having a well-organized room - everything is easy to find and use! 🏠✨*
