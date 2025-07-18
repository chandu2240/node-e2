# Day 10 - Part 02: Security and DevOps Best Practices

## 🎯 Learning Objectives
By the end of this lesson, you will understand:
- Security best practices and threat prevention
- Authentication and authorization systems
- DevOps workflows and CI/CD pipelines
- Automated testing and quality assurance
- Deployment strategies and infrastructure
- Monitoring and incident response

---

## 🔐 Security Best Practices

### Simple Explanation (for 10-year-olds!)

#### **Security - Like Protecting Your House:**
```
🏠 HOME SECURITY SYSTEM
┌─────────────────────────────────────────────────────┐
│  🚪 FRONT DOOR (Authentication)                     │
│  • Only family members have keys                   │
│  • Visitors need to ring doorbell                  │
│  • Special codes for delivery people               │
│                                                     │
│  🏠 ROOM LOCKS (Authorization)                      │
│  • Parents' bedroom has extra lock                 │
│  • Kids can't access dangerous areas               │
│  • Each room has different access rules            │
│                                                     │
│  📷 SECURITY CAMERAS (Monitoring)                  │
│  • Record everything that happens                  │
│  • Alert when something unusual occurs             │
│  • Help find problems later                        │
│                                                     │
│  🚨 ALARM SYSTEM (Alerts)                          │
│  • Loud noise when danger detected                 │
│  • Calls police automatically                      │
│  • Sends message to parents' phones                │
│                                                     │
│  🔒 SAFE (Encryption)                              │
│  • Important documents locked away                 │
│  • Only parents know the combination               │
│  • Even if thieves get in, they can't open it     │
└─────────────────────────────────────────────────────┘
```

#### **DevOps - Like a School Assembly Line:**
```
🏫 SCHOOL ASSEMBLY LINE
┌─────────────────────────────────────────────────────┐
│  📝 HOMEWORK (Code)                                 │
│  • Students write their work                       │
│  • Each student works on different parts           │
│  • Everyone follows the same rules                 │
│                                                     │
│  ✅ TEACHER CHECK (Testing)                         │
│  • Teacher reviews all homework                    │
│  • Marks mistakes and gives feedback               │
│  • Students fix errors before submission           │
│                                                     │
│  📋 GRADE BOOK (CI/CD)                             │
│  • Automatically records good grades               │
│  • Sends report cards to parents                   │
│  • Tracks progress over time                       │
│                                                     │
│  🏆 AWARD CEREMONY (Deployment)                    │
│  • Best work gets displayed                        │
│  • Everyone sees the final results                 │
│  • Celebration of completed project                │
└─────────────────────────────────────────────────────┘
```

---

## 🔒 Security Implementation

### Authentication and Authorization

Create `src/security-practices.ts`:
```typescript
// src/security-practices.ts - Security best practices
console.log('🔒 Learning Security Best Practices!');

import * as crypto from 'crypto';

// 🔐 PASSWORD SECURITY
class PasswordSecurity {
  // Generate salt for password hashing
  static generateSalt(): string {
    return crypto.randomBytes(16).toString('hex');
  }
  
  // Hash password with salt (like mixing secret ingredients)
  static hashPassword(password: string, salt: string): string {
    const hash = crypto.pbkdf2Sync(password, salt, 10000, 64, 'sha512');
    return hash.toString('hex');
  }
  
  // Verify password (like checking if the recipe matches)
  static verifyPassword(password: string, hash: string, salt: string): boolean {
    const testHash = this.hashPassword(password, salt);
    return testHash === hash;
  }
  
  // Check password strength (like checking if your bike lock is strong)
  static checkPasswordStrength(password: string): PasswordStrength {
    let score = 0;
    const feedback: string[] = [];
    
    // Length check
    if (password.length >= 8) {
      score += 25;
    } else {
      feedback.push('Password should be at least 8 characters long');
    }
    
    // Uppercase letters
    if (/[A-Z]/.test(password)) {
      score += 25;
    } else {
      feedback.push('Include uppercase letters (A-Z)');
    }
    
    // Lowercase letters
    if (/[a-z]/.test(password)) {
      score += 25;
    } else {
      feedback.push('Include lowercase letters (a-z)');
    }
    
    // Numbers
    if (/\d/.test(password)) {
      score += 25;
    } else {
      feedback.push('Include numbers (0-9)');
    }
    
    // Special characters
    if (/[!@#$%^&*(),.?":{}|<>]/.test(password)) {
      score += 25;
    } else {
      feedback.push('Include special characters (!@#$%^&*)');
    }
    
    // Bonus for length
    if (password.length >= 12) {
      score += 10;
    }
    
    let strength: 'weak' | 'medium' | 'strong' | 'very strong';
    if (score < 50) {
      strength = 'weak';
    } else if (score < 75) {
      strength = 'medium';
    } else if (score < 100) {
      strength = 'strong';
    } else {
      strength = 'very strong';
    }
    
    return {
      score,
      strength,
      feedback
    };
  }
}

// 🎫 JWT TOKEN MANAGER
class JWTTokenManager {
  private secret: string;
  
  constructor(secret: string) {
    this.secret = secret;
  }
  
  // Create a token (like getting a wristband at a theme park)
  generateToken(payload: any, expiresIn: string = '1h'): string {
    const header = {
      alg: 'HS256',
      typ: 'JWT'
    };
    
    const now = Date.now();
    const expirationTime = this.parseExpiration(expiresIn);
    
    const tokenPayload = {
      ...payload,
      iat: Math.floor(now / 1000),
      exp: Math.floor((now + expirationTime) / 1000)
    };
    
    const encodedHeader = this.base64UrlEncode(JSON.stringify(header));
    const encodedPayload = this.base64UrlEncode(JSON.stringify(tokenPayload));
    
    const signature = this.createSignature(encodedHeader, encodedPayload);
    
    const token = `${encodedHeader}.${encodedPayload}.${signature}`;
    console.log(`🎫 Generated JWT token for user: ${payload.userId}`);
    
    return token;
  }
  
  // Verify token (like checking if the wristband is real)
  verifyToken(token: string): any {
    try {
      const parts = token.split('.');
      if (parts.length !== 3) {
        throw new Error('Invalid token format');
      }
      
      const [encodedHeader, encodedPayload, providedSignature] = parts;
      
      // Verify signature
      const expectedSignature = this.createSignature(encodedHeader, encodedPayload);
      if (providedSignature !== expectedSignature) {
        throw new Error('Invalid token signature');
      }
      
      // Decode payload
      const payload = JSON.parse(this.base64UrlDecode(encodedPayload));
      
      // Check expiration
      const now = Math.floor(Date.now() / 1000);
      if (payload.exp && payload.exp < now) {
        throw new Error('Token expired');
      }
      
      console.log(`✅ Token verified for user: ${payload.userId}`);
      return payload;
      
    } catch (error) {
      console.log(`❌ Token verification failed: ${error.message}`);
      throw error;
    }
  }
  
  private createSignature(header: string, payload: string): string {
    const data = `${header}.${payload}`;
    const signature = crypto.createHmac('sha256', this.secret).update(data).digest('base64url');
    return signature;
  }
  
  private base64UrlEncode(str: string): string {
    return Buffer.from(str).toString('base64url');
  }
  
  private base64UrlDecode(str: string): string {
    return Buffer.from(str, 'base64url').toString();
  }
  
  private parseExpiration(expiresIn: string): number {
    const match = expiresIn.match(/^(\d+)([smhd])$/);
    if (!match) {
      throw new Error('Invalid expiration format');
    }
    
    const value = parseInt(match[1]);
    const unit = match[2];
    
    switch (unit) {
      case 's': return value * 1000;
      case 'm': return value * 60 * 1000;
      case 'h': return value * 60 * 60 * 1000;
      case 'd': return value * 24 * 60 * 60 * 1000;
      default: throw new Error('Invalid time unit');
    }
  }
}

// 🛡️ RATE LIMITER
class RateLimiter {
  private requests: Map<string, number[]> = new Map();
  private maxRequests: number;
  private windowMs: number;
  
  constructor(maxRequests: number, windowMs: number) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
  }
  
  // Check if request is allowed (like checking if you can have more cookies)
  isAllowed(identifier: string): boolean {
    const now = Date.now();
    const requestTimes = this.requests.get(identifier) || [];
    
    // Remove old requests outside the window
    const validRequests = requestTimes.filter(time => now - time < this.windowMs);
    
    if (validRequests.length >= this.maxRequests) {
      console.log(`🚫 Rate limit exceeded for ${identifier}`);
      return false;
    }
    
    // Add current request
    validRequests.push(now);
    this.requests.set(identifier, validRequests);
    
    console.log(`✅ Request allowed for ${identifier} (${validRequests.length}/${this.maxRequests})`);
    return true;
  }
  
  // Get remaining requests
  getRemainingRequests(identifier: string): number {
    const now = Date.now();
    const requestTimes = this.requests.get(identifier) || [];
    const validRequests = requestTimes.filter(time => now - time < this.windowMs);
    
    return Math.max(0, this.maxRequests - validRequests.length);
  }
}

// 🔍 INPUT VALIDATOR
class InputValidator {
  // Validate email (like checking if address is written correctly)
  static validateEmail(email: string): ValidationResult {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    const isValid = emailRegex.test(email);
    
    return {
      isValid,
      message: isValid ? 'Valid email address' : 'Invalid email format'
    };
  }
  
  // Validate phone number
  static validatePhoneNumber(phone: string): ValidationResult {
    const phoneRegex = /^\+?[\d\s\-\(\)]{10,}$/;
    const isValid = phoneRegex.test(phone);
    
    return {
      isValid,
      message: isValid ? 'Valid phone number' : 'Invalid phone number format'
    };
  }
  
  // Sanitize HTML (remove dangerous code)
  static sanitizeHTML(input: string): string {
    return input
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#x27;')
      .replace(/\//g, '&#x2F;');
  }
  
  // Validate SQL injection attempts
  static validateSQLSafety(input: string): ValidationResult {
    const sqlKeywords = ['SELECT', 'INSERT', 'UPDATE', 'DELETE', 'DROP', 'UNION', 'EXEC'];
    const upperInput = input.toUpperCase();
    
    for (const keyword of sqlKeywords) {
      if (upperInput.includes(keyword)) {
        return {
          isValid: false,
          message: `Potential SQL injection detected: ${keyword}`
        };
      }
    }
    
    return {
      isValid: true,
      message: 'Input appears safe'
    };
  }
}

// 🔒 ENCRYPTION SERVICE
class EncryptionService {
  private algorithm = 'aes-256-gcm';
  private key: Buffer;
  
  constructor(secretKey: string) {
    this.key = crypto.scryptSync(secretKey, 'salt', 32);
  }
  
  // Encrypt data (like putting it in a locked box)
  encrypt(text: string): EncryptedData {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipher(this.algorithm, this.key);
    
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const authTag = cipher.getAuthTag();
    
    console.log('🔐 Data encrypted successfully');
    
    return {
      iv: iv.toString('hex'),
      encryptedData: encrypted,
      authTag: authTag.toString('hex')
    };
  }
  
  // Decrypt data (like opening the locked box)
  decrypt(encryptedData: EncryptedData): string {
    const decipher = crypto.createDecipher(this.algorithm, this.key);
    decipher.setAuthTag(Buffer.from(encryptedData.authTag, 'hex'));
    
    let decrypted = decipher.update(encryptedData.encryptedData, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    console.log('🔓 Data decrypted successfully');
    return decrypted;
  }
}

// 🎮 SECURITY DEMO
class SecurityDemo {
  private passwordSecurity = new PasswordSecurity();
  private tokenManager = new JWTTokenManager('super-secret-key-for-demo');
  private rateLimiter = new RateLimiter(5, 60000); // 5 requests per minute
  private encryptionService = new EncryptionService('my-secret-encryption-key');
  
  demonstratePasswordSecurity(): void {
    console.log('\n🔐 Password Security Demo:');
    
    const passwords = ['weak', 'Better123', 'SuperSecure123!', 'MyV3ryStr0ng!P@ssw0rd'];
    
    passwords.forEach(password => {
      const strength = PasswordSecurity.checkPasswordStrength(password);
      console.log(`Password: "${password}"`);
      console.log(`   Strength: ${strength.strength} (${strength.score}%)`);
      
      if (strength.feedback.length > 0) {
        console.log(`   Suggestions: ${strength.feedback.join(', ')}`);
      }
      
      // Hash the password
      const salt = PasswordSecurity.generateSalt();
      const hash = PasswordSecurity.hashPassword(password, salt);
      const verified = PasswordSecurity.verifyPassword(password, hash, salt);
      
      console.log(`   Hash verification: ${verified ? 'Success' : 'Failed'}`);
      console.log('');
    });
  }
  
  demonstrateJWTTokens(): void {
    console.log('\n🎫 JWT Token Demo:');
    
    const userData = {
      userId: 'user123',
      email: 'user@example.com',
      role: 'admin'
    };
    
    // Generate token
    const token = this.tokenManager.generateToken(userData, '1h');
    console.log(`Generated token: ${token.substring(0, 50)}...`);
    
    // Verify token
    try {
      const decoded = this.tokenManager.verifyToken(token);
      console.log(`Decoded payload:`, decoded);
    } catch (error) {
      console.log(`Token verification failed: ${error.message}`);
    }
    
    // Test expired token
    console.log('\n🕐 Testing expired token:');
    const expiredToken = this.tokenManager.generateToken(userData, '0s');
    
    setTimeout(() => {
      try {
        this.tokenManager.verifyToken(expiredToken);
      } catch (error) {
        console.log(`Expected error: ${error.message}`);
      }
    }, 1000);
  }
  
  demonstrateRateLimiting(): void {
    console.log('\n🚫 Rate Limiting Demo:');
    
    const userId = 'user123';
    
    // Simulate multiple requests
    for (let i = 1; i <= 7; i++) {
      const allowed = this.rateLimiter.isAllowed(userId);
      const remaining = this.rateLimiter.getRemainingRequests(userId);
      
      console.log(`Request ${i}: ${allowed ? 'Allowed' : 'Blocked'} (${remaining} remaining)`);
    }
  }
  
  demonstrateInputValidation(): void {
    console.log('\n🔍 Input Validation Demo:');
    
    const testInputs = [
      { type: 'email', value: 'user@example.com' },
      { type: 'email', value: 'invalid-email' },
      { type: 'phone', value: '+1-555-123-4567' },
      { type: 'phone', value: '123' },
      { type: 'sql', value: 'SELECT * FROM users' },
      { type: 'sql', value: 'Hello World' },
      { type: 'html', value: '<script>alert("hack")</script>' }
    ];
    
    testInputs.forEach(input => {
      console.log(`\nTesting ${input.type}: "${input.value}"`);
      
      if (input.type === 'email') {
        const result = InputValidator.validateEmail(input.value);
        console.log(`   ${result.isValid ? '✅' : '❌'} ${result.message}`);
      } else if (input.type === 'phone') {
        const result = InputValidator.validatePhoneNumber(input.value);
        console.log(`   ${result.isValid ? '✅' : '❌'} ${result.message}`);
      } else if (input.type === 'sql') {
        const result = InputValidator.validateSQLSafety(input.value);
        console.log(`   ${result.isValid ? '✅' : '❌'} ${result.message}`);
      } else if (input.type === 'html') {
        const sanitized = InputValidator.sanitizeHTML(input.value);
        console.log(`   Sanitized: "${sanitized}"`);
      }
    });
  }
  
  demonstrateEncryption(): void {
    console.log('\n🔒 Encryption Demo:');
    
    const sensitiveData = 'This is super secret information!';
    console.log(`Original data: "${sensitiveData}"`);
    
    // Encrypt the data
    const encrypted = this.encryptionService.encrypt(sensitiveData);
    console.log(`Encrypted data: ${encrypted.encryptedData.substring(0, 30)}...`);
    
    // Decrypt the data
    const decrypted = this.encryptionService.decrypt(encrypted);
    console.log(`Decrypted data: "${decrypted}"`);
    
    console.log(`Data integrity: ${sensitiveData === decrypted ? 'Verified' : 'Compromised'}`);
  }
  
  async runSecurityDemo(): Promise<void> {
    console.log('🎮 Security Demo Starting!\n');
    
    this.demonstratePasswordSecurity();
    this.demonstrateJWTTokens();
    
    // Wait for JWT demo to complete
    await new Promise(resolve => setTimeout(resolve, 1500));
    
    this.demonstrateRateLimiting();
    this.demonstrateInputValidation();
    this.demonstrateEncryption();
    
    console.log('\n🎉 Security demo complete!');
  }
}

// 🚀 DEVOPS PIPELINE SIMULATOR
class DevOpsPipelineSimulator {
  private stages = [
    { name: 'Code Commit', duration: 1000, success: 0.98 },
    { name: 'Unit Tests', duration: 2000, success: 0.95 },
    { name: 'Integration Tests', duration: 3000, success: 0.90 },
    { name: 'Security Scan', duration: 1500, success: 0.92 },
    { name: 'Build Application', duration: 2500, success: 0.97 },
    { name: 'Deploy to Staging', duration: 1800, success: 0.94 },
    { name: 'Smoke Tests', duration: 1200, success: 0.88 },
    { name: 'Deploy to Production', duration: 2200, success: 0.96 }
  ];
  
  async runPipeline(): Promise<void> {
    console.log('\n🚀 DevOps Pipeline Demo:');
    console.log('📋 Starting CI/CD Pipeline...\n');
    
    let totalTime = 0;
    
    for (const stage of this.stages) {
      const startTime = Date.now();
      console.log(`⏳ Running: ${stage.name}...`);
      
      // Simulate stage execution
      await new Promise(resolve => setTimeout(resolve, stage.duration));
      
      const endTime = Date.now();
      const actualDuration = endTime - startTime;
      totalTime += actualDuration;
      
      // Simulate success/failure
      const success = Math.random() < stage.success;
      
      if (success) {
        console.log(`✅ ${stage.name} completed successfully (${actualDuration}ms)`);
      } else {
        console.log(`❌ ${stage.name} failed!`);
        console.log(`🔄 Pipeline stopped. Manual intervention required.`);
        console.log(`📧 Notification sent to development team.`);
        return;
      }
    }
    
    console.log(`\n🎉 Pipeline completed successfully!`);
    console.log(`⏱️ Total execution time: ${totalTime}ms`);
    console.log(`📊 All stages passed quality gates.`);
    console.log(`🚀 Application deployed to production!`);
  }
}

// 🎯 TYPE DEFINITIONS
interface PasswordStrength {
  score: number;
  strength: 'weak' | 'medium' | 'strong' | 'very strong';
  feedback: string[];
}

interface ValidationResult {
  isValid: boolean;
  message: string;
}

interface EncryptedData {
  iv: string;
  encryptedData: string;
  authTag: string;
}

// 🎮 RUN SECURITY AND DEVOPS DEMO
async function runSecurityAndDevOpsDemo(): Promise<void> {
  console.log('🎮 Security and DevOps Demo Starting!\n');
  
  const securityDemo = new SecurityDemo();
  await securityDemo.runSecurityDemo();
  
  const devopsDemo = new DevOpsPipelineSimulator();
  await devopsDemo.runPipeline();
  
  console.log('\n📚 Security and DevOps Key Takeaways:');
  console.log('   ✅ Use strong password policies');
  console.log('   ✅ Implement JWT tokens for authentication');
  console.log('   ✅ Add rate limiting to prevent abuse');
  console.log('   ✅ Validate and sanitize all inputs');
  console.log('   ✅ Encrypt sensitive data');
  console.log('   ✅ Automate testing and deployment');
  console.log('   ✅ Monitor security continuously');
  console.log('   ✅ Plan for failure scenarios');
  console.log('\n🎉 Security and DevOps demo complete!');
}

// Run the demo
runSecurityAndDevOpsDemo().catch(console.error);
