# Day 10 - Enterprise Architecture and Best Practices

## 🎯 Learning Objectives
By the end of this lesson, you will understand:
- Enterprise architecture patterns and design principles
- Scalability strategies and system design
- Security best practices for production applications
- DevOps practices and CI/CD pipelines
- Code quality and team collaboration
- Production deployment and maintenance

---

## 🏢 Understanding Enterprise Architecture

### Simple Explanation (for 10-year-olds!)

#### **Enterprise Architecture - Like Building a Super City:**
```
🏙️ BUILDING A MEGA CITY
┌─────────────────────────────────────────────────────┐
│  🏢 SKYSCRAPERS (Applications)                      │
│  • Each building has a purpose                      │
│  • Some are offices, some are shops                │
│  • All connected by elevators and stairs           │
│                                                     │
│  🛣️ ROADS & HIGHWAYS (APIs)                        │
│  • Connect all buildings together                  │
│  • Traffic flows smoothly                          │
│  • Emergency routes for problems                   │
│                                                     │
│  🏭 POWER PLANT (Database)                         │
│  • Powers the whole city                           │
│  • Backup generators for emergencies              │
│  • Distributes energy efficiently                 │
│                                                     │
│  🚨 SECURITY SYSTEM (Authentication)               │
│  • Guards at every entrance                        │
│  • ID cards for everyone                          │
│  • Security cameras everywhere                     │
│                                                     │
│  🏥 HOSPITAL (Monitoring)                          │
│  • Checks everyone's health                        │
│  • Ambulances for emergencies                     │
│  • Doctors always on call                         │
└─────────────────────────────────────────────────────┘
```

#### **Team Collaboration - Like a School Orchestra:**
```
🎼 SCHOOL ORCHESTRA
┌─────────────────────────────────────────────────────┐
│  🎭 CONDUCTOR (Team Lead)                           │
│  • Keeps everyone in sync                          │
│  • Decides the tempo                               │
│  • Makes sure everyone plays together             │
│                                                     │
│  🎻 VIOLINS (Frontend Developers)                  │
│  • Play the melody everyone hears                  │
│  • Make beautiful music                            │
│  • Work together as a section                      │
│                                                     │
│  🎺 TRUMPETS (Backend Developers)                  │
│  • Provide the strong foundation                   │
│  • Support the melody                              │
│  • Make powerful, reliable sounds                  │
│                                                     │
│  🥁 DRUMS (DevOps Engineers)                       │
│  • Keep the steady beat                            │
│  • Make sure timing is perfect                     │
│  • Support everyone else's rhythm                  │
│                                                     │
│  📚 SHEET MUSIC (Documentation)                    │
│  • Everyone knows what to play                     │
│  • Clear instructions for everyone                │
│  • Same music for the whole orchestra             │
└─────────────────────────────────────────────────────┘
```

---

## 🏗️ System Design Principles

### Design Patterns and Architecture

Create `src/enterprise-architecture.ts`:
```typescript
// src/enterprise-architecture.ts - Enterprise architecture patterns
console.log('🏢 Learning Enterprise Architecture!');

// 🏗️ SINGLETON PATTERN - Like Having One School Principal
class DatabaseConnection {
  private static instance: DatabaseConnection;
  private connections: number = 0;
  
  private constructor() {
    console.log('🏢 Creating database connection (Principal hired!)');
  }
  
  static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }
    return DatabaseConnection.instance;
  }
  
  connect(): void {
    this.connections++;
    console.log(`🔌 Connected to database (${this.connections} active connections)`);
  }
  
  disconnect(): void {
    if (this.connections > 0) {
      this.connections--;
      console.log(`📱 Disconnected from database (${this.connections} active connections)`);
    }
  }
  
  getConnectionCount(): number {
    return this.connections;
  }
}

// 🏭 FACTORY PATTERN - Like a Toy Factory
abstract class Vehicle {
  abstract type: string;
  abstract start(): void;
  abstract stop(): void;
}

class Car extends Vehicle {
  type = 'car';
  
  start(): void {
    console.log('🚗 Car engine started: Vroom vroom!');
  }
  
  stop(): void {
    console.log('🚗 Car engine stopped');
  }
}

class Truck extends Vehicle {
  type = 'truck';
  
  start(): void {
    console.log('🚛 Truck engine started: RUMBLE RUMBLE!');
  }
  
  stop(): void {
    console.log('🚛 Truck engine stopped');
  }
}

class Motorcycle extends Vehicle {
  type = 'motorcycle';
  
  start(): void {
    console.log('🏍️ Motorcycle engine started: VROOOOM!');
  }
  
  stop(): void {
    console.log('🏍️ Motorcycle engine stopped');
  }
}

class VehicleFactory {
  static createVehicle(type: 'car' | 'truck' | 'motorcycle'): Vehicle {
    console.log(`🏭 Factory creating a ${type}...`);
    
    switch (type) {
      case 'car':
        return new Car();
      case 'truck':
        return new Truck();
      case 'motorcycle':
        return new Motorcycle();
      default:
        throw new Error(`Unknown vehicle type: ${type}`);
    }
  }
}

// 👁️ OBSERVER PATTERN - Like a News Reporter
interface Observer {
  update(news: string): void;
}

class NewsReporter implements Observer {
  private name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  update(news: string): void {
    console.log(`📰 Reporter ${this.name} got breaking news: ${news}`);
  }
}

class NewsAgency {
  private observers: Observer[] = [];
  private latestNews: string = '';
  
  subscribe(observer: Observer): void {
    this.observers.push(observer);
    console.log('📺 New reporter subscribed to news agency');
  }
  
  unsubscribe(observer: Observer): void {
    const index = this.observers.indexOf(observer);
    if (index > -1) {
      this.observers.splice(index, 1);
      console.log('📺 Reporter unsubscribed from news agency');
    }
  }
  
  notifyObservers(): void {
    console.log(`📢 Notifying ${this.observers.length} reporters...`);
    for (const observer of this.observers) {
      observer.update(this.latestNews);
    }
  }
  
  setNews(news: string): void {
    console.log(`📝 News agency received: ${news}`);
    this.latestNews = news;
    this.notifyObservers();
  }
}

// 🔌 DEPENDENCY INJECTION - Like Plug and Play Toys
interface EmailService {
  sendEmail(to: string, subject: string, body: string): void;
}

class GmailService implements EmailService {
  sendEmail(to: string, subject: string, body: string): void {
    console.log(`📧 Gmail: Sending email to ${to} - ${subject}`);
  }
}

class OutlookService implements EmailService {
  sendEmail(to: string, subject: string, body: string): void {
    console.log(`📧 Outlook: Sending email to ${to} - ${subject}`);
  }
}

class NotificationService {
  private emailService: EmailService;
  
  constructor(emailService: EmailService) {
    this.emailService = emailService;
    console.log('📬 Notification service created with email provider');
  }
  
  sendWelcomeEmail(userEmail: string): void {
    this.emailService.sendEmail(
      userEmail,
      'Welcome!',
      'Welcome to our amazing app!'
    );
  }
  
  sendAlertEmail(userEmail: string, alert: string): void {
    this.emailService.sendEmail(
      userEmail,
      'Alert!',
      `Important alert: ${alert}`
    );
  }
}

// 🎯 STRATEGY PATTERN - Like Different Game Strategies
interface PaymentStrategy {
  pay(amount: number): void;
}

class CreditCardPayment implements PaymentStrategy {
  private cardNumber: string;
  
  constructor(cardNumber: string) {
    this.cardNumber = cardNumber;
  }
  
  pay(amount: number): void {
    console.log(`💳 Paid $${amount} using Credit Card ending in ${this.cardNumber.slice(-4)}`);
  }
}

class PayPalPayment implements PaymentStrategy {
  private email: string;
  
  constructor(email: string) {
    this.email = email;
  }
  
  pay(amount: number): void {
    console.log(`🏦 Paid $${amount} using PayPal account ${this.email}`);
  }
}

class BankTransferPayment implements PaymentStrategy {
  private accountNumber: string;
  
  constructor(accountNumber: string) {
    this.accountNumber = accountNumber;
  }
  
  pay(amount: number): void {
    console.log(`🏛️ Paid $${amount} using Bank Transfer from account ${this.accountNumber}`);
  }
}

class ShoppingCart {
  private items: Array<{name: string, price: number}> = [];
  private paymentStrategy: PaymentStrategy;
  
  constructor(paymentStrategy: PaymentStrategy) {
    this.paymentStrategy = paymentStrategy;
  }
  
  addItem(name: string, price: number): void {
    this.items.push({ name, price });
    console.log(`🛒 Added ${name} ($${price}) to cart`);
  }
  
  setPaymentStrategy(strategy: PaymentStrategy): void {
    this.paymentStrategy = strategy;
    console.log('💰 Payment method changed');
  }
  
  checkout(): void {
    const total = this.items.reduce((sum, item) => sum + item.price, 0);
    console.log(`🛍️ Cart total: $${total}`);
    
    if (this.items.length === 0) {
      console.log('🛒 Cart is empty!');
      return;
    }
    
    this.paymentStrategy.pay(total);
    this.items = [];
    console.log('✅ Checkout complete! Cart emptied.');
  }
}

// 🌉 ADAPTER PATTERN - Like a Universal Translator
interface ModernAPI {
  getData(): Promise<any>;
}

class OldSystemAPI {
  getOldData(): string {
    return 'Old system data in weird format';
  }
}

class LegacyAPIAdapter implements ModernAPI {
  private oldSystem: OldSystemAPI;
  
  constructor(oldSystem: OldSystemAPI) {
    this.oldSystem = oldSystem;
  }
  
  async getData(): Promise<any> {
    console.log('🔄 Adapter converting old data to new format...');
    const oldData = this.oldSystem.getOldData();
    
    // Convert old format to new format
    const newData = {
      data: oldData,
      timestamp: new Date().toISOString(),
      formatted: true
    };
    
    console.log('✅ Data successfully converted!');
    return newData;
  }
}

// 🎮 DESIGN PATTERNS DEMO
class DesignPatternsDemo {
  async demonstrateSingleton(): Promise<void> {
    console.log('\n🏢 Singleton Pattern Demo:');
    
    // Try to create multiple instances
    const db1 = DatabaseConnection.getInstance();
    const db2 = DatabaseConnection.getInstance();
    const db3 = DatabaseConnection.getInstance();
    
    console.log(`Same instance? ${db1 === db2 && db2 === db3}`);
    
    db1.connect();
    db2.connect();
    db3.connect();
    
    console.log(`Total connections: ${db1.getConnectionCount()}`);
  }
  
  demonstrateFactory(): void {
    console.log('\n🏭 Factory Pattern Demo:');
    
    const vehicles = [
      VehicleFactory.createVehicle('car'),
      VehicleFactory.createVehicle('truck'),
      VehicleFactory.createVehicle('motorcycle')
    ];
    
    vehicles.forEach(vehicle => {
      vehicle.start();
      vehicle.stop();
    });
  }
  
  demonstrateObserver(): void {
    console.log('\n👁️ Observer Pattern Demo:');
    
    const newsAgency = new NewsAgency();
    
    const reporter1 = new NewsReporter('Alice');
    const reporter2 = new NewsReporter('Bob');
    const reporter3 = new NewsReporter('Charlie');
    
    newsAgency.subscribe(reporter1);
    newsAgency.subscribe(reporter2);
    newsAgency.subscribe(reporter3);
    
    newsAgency.setNews('Breaking: New JavaScript framework released!');
    newsAgency.setNews('Update: Node.js gets major performance boost!');
    
    newsAgency.unsubscribe(reporter2);
    newsAgency.setNews('Final: Enterprise architecture seminar complete!');
  }
  
  demonstrateDependencyInjection(): void {
    console.log('\n🔌 Dependency Injection Demo:');
    
    // Use Gmail service
    const gmailService = new GmailService();
    const notificationWithGmail = new NotificationService(gmailService);
    notificationWithGmail.sendWelcomeEmail('user@example.com');
    
    // Switch to Outlook service
    const outlookService = new OutlookService();
    const notificationWithOutlook = new NotificationService(outlookService);
    notificationWithOutlook.sendAlertEmail('user@example.com', 'Your account was accessed');
  }
  
  demonstrateStrategy(): void {
    console.log('\n🎯 Strategy Pattern Demo:');
    
    // Create shopping cart with credit card payment
    const cart = new ShoppingCart(new CreditCardPayment('1234567890123456'));
    
    cart.addItem('Laptop', 999.99);
    cart.addItem('Mouse', 29.99);
    cart.addItem('Keyboard', 79.99);
    
    cart.checkout();
    
    // Add more items and use different payment method
    cart.addItem('Monitor', 299.99);
    cart.setPaymentStrategy(new PayPalPayment('user@example.com'));
    cart.checkout();
    
    // Try bank transfer
    cart.addItem('Webcam', 89.99);
    cart.setPaymentStrategy(new BankTransferPayment('9876543210'));
    cart.checkout();
  }
  
  async demonstrateAdapter(): Promise<void> {
    console.log('\n🌉 Adapter Pattern Demo:');
    
    const oldSystem = new OldSystemAPI();
    const adapter = new LegacyAPIAdapter(oldSystem);
    
    const modernData = await adapter.getData();
    console.log('📦 Modern data:', modernData);
  }
  
  async runAllDemos(): Promise<void> {
    console.log('🎮 Design Patterns Demo Starting!\n');
    
    await this.demonstrateSingleton();
    this.demonstrateFactory();
    this.demonstrateObserver();
    this.demonstrateDependencyInjection();
    this.demonstrateStrategy();
    await this.demonstrateAdapter();
    
    console.log('\n🎉 All design patterns demonstrated!');
  }
}

// 🏢 ENTERPRISE ARCHITECTURE CONCEPTS
class EnterpriseArchitectureDemo {
  demonstrateLayeredArchitecture(): void {
    console.log('\n🏢 Layered Architecture Demo:');
    
    // Presentation Layer
    console.log('📱 Presentation Layer: User clicked "Get Users" button');
    
    // Business Logic Layer
    console.log('⚙️ Business Logic Layer: Processing user request');
    console.log('   - Validating user permissions');
    console.log('   - Applying business rules');
    console.log('   - Formatting data for display');
    
    // Data Access Layer
    console.log('🗄️ Data Access Layer: Fetching data from database');
    console.log('   - SQL query executed');
    console.log('   - Results retrieved');
    
    // Database Layer
    console.log('💾 Database Layer: Data returned');
    
    // Response flows back up
    console.log('⬆️ Response flows back through layers');
    console.log('📱 User sees the results!');
  }
  
  demonstrateMicroservicesArchitecture(): void {
    console.log('\n🔧 Microservices Architecture Demo:');
    
    const services = [
      { name: 'User Service', port: 3001, function: 'Manages user accounts' },
      { name: 'Product Service', port: 3002, function: 'Manages products' },
      { name: 'Order Service', port: 3003, function: 'Processes orders' },
      { name: 'Payment Service', port: 3004, function: 'Handles payments' },
      { name: 'Notification Service', port: 3005, function: 'Sends emails/SMS' }
    ];
    
    console.log('🏢 Starting microservices...');
    services.forEach(service => {
      console.log(`   🚀 ${service.name} running on port ${service.port} - ${service.function}`);
    });
    
    console.log('\n🔄 Simulating e-commerce order flow:');
    console.log('   1. 👤 User Service: Validates user');
    console.log('   2. 📦 Product Service: Checks product availability');
    console.log('   3. 🛒 Order Service: Creates order');
    console.log('   4. 💳 Payment Service: Processes payment');
    console.log('   5. 📧 Notification Service: Sends confirmation email');
    console.log('   ✅ Order completed successfully!');
  }
  
  demonstrateEventDrivenArchitecture(): void {
    console.log('\n⚡ Event-Driven Architecture Demo:');
    
    const events = [
      { name: 'UserRegistered', data: { userId: 'user123', email: 'user@example.com' } },
      { name: 'OrderPlaced', data: { orderId: 'order456', userId: 'user123', total: 99.99 } },
      { name: 'PaymentProcessed', data: { orderId: 'order456', amount: 99.99 } },
      { name: 'OrderShipped', data: { orderId: 'order456', trackingNumber: 'TRACK789' } }
    ];
    
    console.log('📡 Event Bus is listening for events...');
    
    events.forEach((event, index) => {
      setTimeout(() => {
        console.log(`📢 Event Published: ${event.name}`);
        console.log(`   📦 Data: ${JSON.stringify(event.data)}`);
        
        // Simulate services reacting to events
        if (event.name === 'UserRegistered') {
          console.log('   🎯 Email Service: Sending welcome email');
          console.log('   🎯 Analytics Service: Recording new user');
        } else if (event.name === 'OrderPlaced') {
          console.log('   🎯 Inventory Service: Updating stock');
          console.log('   🎯 Payment Service: Charging customer');
        } else if (event.name === 'PaymentProcessed') {
          console.log('   🎯 Order Service: Updating order status');
          console.log('   🎯 Shipping Service: Preparing shipment');
        } else if (event.name === 'OrderShipped') {
          console.log('   🎯 Notification Service: Sending tracking info');
          console.log('   🎯 Analytics Service: Recording shipment');
        }
        
        console.log('');
      }, index * 1000);
    });
  }
  
  async runArchitectureDemo(): Promise<void> {
    console.log('🏗️ Enterprise Architecture Demo Starting!\n');
    
    this.demonstrateLayeredArchitecture();
    this.demonstrateMicroservicesArchitecture();
    this.demonstrateEventDrivenArchitecture();
    
    // Wait for events to complete
    await new Promise(resolve => setTimeout(resolve, 5000));
    
    console.log('\n🎉 Architecture patterns demonstrated!');
  }
}

// 🎮 RUN ENTERPRISE ARCHITECTURE DEMO
async function runEnterpriseArchitectureDemo(): Promise<void> {
  console.log('🎮 Enterprise Architecture Demo Starting!\n');
  
  const patternsDemo = new DesignPatternsDemo();
  await patternsDemo.runAllDemos();
  
  const architectureDemo = new EnterpriseArchitectureDemo();
  await architectureDemo.runArchitectureDemo();
  
  console.log('\n📚 Enterprise Architecture Key Takeaways:');
  console.log('   ✅ Use design patterns to solve common problems');
  console.log('   ✅ Singleton for shared resources');
  console.log('   ✅ Factory for creating objects');
  console.log('   ✅ Observer for event notifications');
  console.log('   ✅ Dependency injection for flexibility');
  console.log('   ✅ Strategy for different algorithms');
  console.log('   ✅ Adapter for legacy system integration');
  console.log('   ✅ Layer architecture for organization');
  console.log('   ✅ Microservices for scalability');
  console.log('   ✅ Event-driven for loose coupling');
  console.log('\n🎉 Enterprise architecture demo complete!');
}

// Run the demo
runEnterpriseArchitectureDemo().catch(console.error);
