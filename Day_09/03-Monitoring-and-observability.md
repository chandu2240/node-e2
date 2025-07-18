# Day 09 - Part 03: Monitoring and Observability

## 🎯 Learning Objectives
By the end of this lesson, you will understand:
- Application monitoring and health checks
- Logging strategies and structured logging
- Metrics collection and visualization
- Error tracking and alerting
- Performance monitoring and profiling
- Distributed tracing fundamentals

---

## 🔍 Understanding Monitoring

### Simple Explanation (for 10-year-olds!)

#### **Monitoring - Like Being a Super Detective:**
```
🕵️ THE DETECTIVE'S TOOLKIT
┌─────────────────────────────────────────────────────┐
│  🔍 MAGNIFYING GLASS (Logs)                        │
│  • Look at clues (what happened)                   │
│  • Read the story step by step                     │
│  • Find problems and mistakes                      │
│                                                     │
│  📊 MEASURING TAPE (Metrics)                       │
│  • How fast things are going                       │
│  • How much memory is being used                   │
│  • Count how many people visited                   │
│                                                     │
│  🚨 ALARM SYSTEM (Alerts)                          │
│  • Ring when something goes wrong                  │
│  • Wake you up if the house is on fire             │
│  • Tell you when to fix things                     │
│                                                     │
│  🗺️ TREASURE MAP (Tracing)                         │
│  • Follow the path of requests                     │
│  • See where the treasure hunt went wrong          │
│  • Find which part of the journey was slow         │
└─────────────────────────────────────────────────────┘
```

#### **Health Check - Like a Doctor's Checkup:**
```
🏥 REGULAR HEALTH CHECKUP
┌─────────────────────────────────────────────────────┐
│  👨‍⚕️ Doctor: "How are you feeling today?"            │
│  🤖 App: "I'm feeling great! All systems working!"   │
│                                                     │
│  👨‍⚕️ Doctor: "Can you run and jump?"                │
│  🤖 App: "Yes! Database connected, APIs working!"    │
│                                                     │
│  👨‍⚕️ Doctor: "Any pain or problems?"               │
│  🤖 App: "Nope! Memory usage is normal!"            │
│                                                     │
│  👨‍⚕️ Doctor: "Great! Come back tomorrow!"          │
│  🤖 App: "Will do! Thanks for checking!"            │
└─────────────────────────────────────────────────────┘
```

---

## 📊 Health Checks and Monitoring

### Basic Health Check Implementation

Create `src/health-monitoring.ts`:
```typescript
// src/health-monitoring.ts - Health checks and monitoring
console.log('🏥 Learning Health Checks and Monitoring!');

import { performance } from 'perf_hooks';

// 🏥 HEALTH CHECK SYSTEM
class HealthChecker {
  private healthChecks: Map<string, () => Promise<HealthStatus>> = new Map();
  private lastCheckResults: Map<string, HealthResult> = new Map();
  
  // Register a health check
  registerHealthCheck(name: string, check: () => Promise<HealthStatus>): void {
    this.healthChecks.set(name, check);
    console.log(`✅ Registered health check: ${name}`);
  }
  
  // Run a single health check
  async runHealthCheck(name: string): Promise<HealthResult> {
    const check = this.healthChecks.get(name);
    if (!check) {
      throw new Error(`Health check '${name}' not found`);
    }
    
    const startTime = performance.now();
    
    try {
      const status = await check();
      const endTime = performance.now();
      const duration = endTime - startTime;
      
      const result: HealthResult = {
        name,
        status: status.healthy ? 'healthy' : 'unhealthy',
        message: status.message,
        duration: Math.round(duration),
        timestamp: new Date().toISOString(),
        details: status.details
      };
      
      this.lastCheckResults.set(name, result);
      
      if (status.healthy) {
        console.log(`✅ ${name}: ${status.message} (${Math.round(duration)}ms)`);
      } else {
        console.log(`❌ ${name}: ${status.message} (${Math.round(duration)}ms)`);
      }
      
      return result;
    } catch (error) {
      const endTime = performance.now();
      const duration = endTime - startTime;
      
      const result: HealthResult = {
        name,
        status: 'unhealthy',
        message: `Health check failed: ${error.message}`,
        duration: Math.round(duration),
        timestamp: new Date().toISOString(),
        error: error.message
      };
      
      this.lastCheckResults.set(name, result);
      console.log(`❌ ${name}: Health check failed - ${error.message}`);
      
      return result;
    }
  }
  
  // Run all health checks
  async runAllHealthChecks(): Promise<OverallHealth> {
    console.log('🏥 Running all health checks...');
    
    const results: HealthResult[] = [];
    let healthyCount = 0;
    
    for (const [name] of this.healthChecks) {
      const result = await this.runHealthCheck(name);
      results.push(result);
      
      if (result.status === 'healthy') {
        healthyCount++;
      }
    }
    
    const overallStatus = healthyCount === results.length ? 'healthy' : 'unhealthy';
    
    const overall: OverallHealth = {
      status: overallStatus,
      timestamp: new Date().toISOString(),
      totalChecks: results.length,
      healthyChecks: healthyCount,
      unhealthyChecks: results.length - healthyCount,
      checks: results
    };
    
    console.log(`🏥 Overall health: ${overallStatus} (${healthyCount}/${results.length} healthy)`);
    
    return overall;
  }
  
  // Get health check summary
  getHealthSummary(): HealthSummary {
    const summary: HealthSummary = {
      totalChecks: this.healthChecks.size,
      lastResults: Array.from(this.lastCheckResults.values()),
      timestamp: new Date().toISOString()
    };
    
    return summary;
  }
}

// 🔍 SYSTEM MONITORS
class SystemMonitor {
  private metrics: Map<string, number[]> = new Map();
  private alerts: Alert[] = [];
  
  // Record a metric
  recordMetric(name: string, value: number): void {
    if (!this.metrics.has(name)) {
      this.metrics.set(name, []);
    }
    
    const values = this.metrics.get(name)!;
    values.push(value);
    
    // Keep only last 100 values
    if (values.length > 100) {
      values.shift();
    }
    
    console.log(`📊 Metric recorded: ${name} = ${value}`);
    
    // Check for alerts
    this.checkAlerts(name, value);
  }
  
  // Get metric statistics
  getMetricStats(name: string): MetricStats | undefined {
    const values = this.metrics.get(name);
    if (!values || values.length === 0) {
      return undefined;
    }
    
    const sorted = [...values].sort((a, b) => a - b);
    const sum = values.reduce((a, b) => a + b, 0);
    
    return {
      name,
      count: values.length,
      min: sorted[0],
      max: sorted[sorted.length - 1],
      avg: sum / values.length,
      median: sorted[Math.floor(sorted.length / 2)],
      latest: values[values.length - 1],
      timestamp: new Date().toISOString()
    };
  }
  
  // Set up alerts
  addAlert(name: string, condition: 'above' | 'below', threshold: number): void {
    const alert: Alert = {
      id: `alert_${Date.now()}`,
      metricName: name,
      condition,
      threshold,
      active: true,
      created: new Date().toISOString()
    };
    
    this.alerts.push(alert);
    console.log(`🚨 Alert configured: ${name} ${condition} ${threshold}`);
  }
  
  // Check if metrics trigger alerts
  private checkAlerts(metricName: string, value: number): void {
    const relevantAlerts = this.alerts.filter(a => 
      a.metricName === metricName && a.active
    );
    
    for (const alert of relevantAlerts) {
      let triggered = false;
      
      if (alert.condition === 'above' && value > alert.threshold) {
        triggered = true;
      } else if (alert.condition === 'below' && value < alert.threshold) {
        triggered = true;
      }
      
      if (triggered) {
        console.log(`🚨 ALERT TRIGGERED: ${metricName} is ${value} (${alert.condition} ${alert.threshold})`);
        
        // In a real app, you would send notifications here
        this.sendAlert(alert, metricName, value);
      }
    }
  }
  
  private sendAlert(alert: Alert, metricName: string, value: number): void {
    // Simulate sending alert notification
    console.log(`📧 Sending alert notification:`);
    console.log(`   Alert ID: ${alert.id}`);
    console.log(`   Metric: ${metricName} = ${value}`);
    console.log(`   Condition: ${alert.condition} ${alert.threshold}`);
    console.log(`   Time: ${new Date().toISOString()}`);
  }
  
  // Get all metrics summary
  getAllMetrics(): Record<string, MetricStats> {
    const summary: Record<string, MetricStats> = {};
    
    for (const [name] of this.metrics) {
      const stats = this.getMetricStats(name);
      if (stats) {
        summary[name] = stats;
      }
    }
    
    return summary;
  }
}

// 📝 STRUCTURED LOGGER
class StructuredLogger {
  private logLevel: LogLevel = 'info';
  private logs: LogEntry[] = [];
  private maxLogs = 1000;
  
  setLogLevel(level: LogLevel): void {
    this.logLevel = level;
    console.log(`📝 Log level set to: ${level}`);
  }
  
  // Log with different levels
  debug(message: string, meta?: Record<string, any>): void {
    this.log('debug', message, meta);
  }
  
  info(message: string, meta?: Record<string, any>): void {
    this.log('info', message, meta);
  }
  
  warn(message: string, meta?: Record<string, any>): void {
    this.log('warn', message, meta);
  }
  
  error(message: string, meta?: Record<string, any>): void {
    this.log('error', message, meta);
  }
  
  // Core logging method
  private log(level: LogLevel, message: string, meta?: Record<string, any>): void {
    const levels = ['debug', 'info', 'warn', 'error'];
    const currentLevelIndex = levels.indexOf(this.logLevel);
    const logLevelIndex = levels.indexOf(level);
    
    // Skip if log level is below current threshold
    if (logLevelIndex < currentLevelIndex) {
      return;
    }
    
    const logEntry: LogEntry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      meta,
      correlationId: this.generateCorrelationId()
    };
    
    this.logs.push(logEntry);
    
    // Keep only recent logs
    if (this.logs.length > this.maxLogs) {
      this.logs.shift();
    }
    
    // Output to console with emoji
    const emoji = this.getLevelEmoji(level);
    const metaStr = meta ? ` ${JSON.stringify(meta)}` : '';
    console.log(`${emoji} [${level.toUpperCase()}] ${message}${metaStr}`);
  }
  
  private getLevelEmoji(level: LogLevel): string {
    switch (level) {
      case 'debug': return '🔍';
      case 'info': return 'ℹ️';
      case 'warn': return '⚠️';
      case 'error': return '❌';
      default: return '📝';
    }
  }
  
  private generateCorrelationId(): string {
    return `req_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
  
  // Search logs
  searchLogs(query: string, level?: LogLevel): LogEntry[] {
    let results = this.logs;
    
    if (level) {
      results = results.filter(log => log.level === level);
    }
    
    if (query) {
      results = results.filter(log => 
        log.message.toLowerCase().includes(query.toLowerCase()) ||
        JSON.stringify(log.meta).toLowerCase().includes(query.toLowerCase())
      );
    }
    
    return results;
  }
  
  // Get log statistics
  getLogStats(): LogStats {
    const stats: LogStats = {
      total: this.logs.length,
      byLevel: {
        debug: 0,
        info: 0,
        warn: 0,
        error: 0
      },
      timeRange: {
        oldest: this.logs[0]?.timestamp,
        newest: this.logs[this.logs.length - 1]?.timestamp
      }
    };
    
    for (const log of this.logs) {
      stats.byLevel[log.level]++;
    }
    
    return stats;
  }
}

// 🎯 PERFORMANCE TRACER
class PerformanceTracer {
  private activeTraces: Map<string, TraceSpan> = new Map();
  private completedTraces: TraceSpan[] = [];
  
  // Start a new trace
  startTrace(name: string, tags?: Record<string, string>): string {
    const traceId = `trace_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    
    const span: TraceSpan = {
      traceId,
      name,
      startTime: performance.now(),
      tags: tags || {},
      logs: []
    };
    
    this.activeTraces.set(traceId, span);
    console.log(`🎯 Started trace: ${name} (${traceId})`);
    
    return traceId;
  }
  
  // Add a log to a trace
  addTraceLog(traceId: string, message: string, data?: any): void {
    const span = this.activeTraces.get(traceId);
    if (!span) {
      console.warn(`⚠️ Trace ${traceId} not found`);
      return;
    }
    
    span.logs.push({
      timestamp: performance.now() - span.startTime,
      message,
      data
    });
    
    console.log(`📋 Trace log [${traceId}]: ${message}`);
  }
  
  // End a trace
  endTrace(traceId: string): TraceSpan | undefined {
    const span = this.activeTraces.get(traceId);
    if (!span) {
      console.warn(`⚠️ Trace ${traceId} not found`);
      return undefined;
    }
    
    span.endTime = performance.now();
    span.duration = span.endTime - span.startTime;
    
    this.activeTraces.delete(traceId);
    this.completedTraces.push(span);
    
    console.log(`🏁 Completed trace: ${span.name} (${Math.round(span.duration)}ms)`);
    
    return span;
  }
  
  // Get trace statistics
  getTraceStats(): TraceStats {
    const allTraces = [...this.completedTraces];
    if (allTraces.length === 0) {
      return {
        totalTraces: 0,
        avgDuration: 0,
        slowestTrace: undefined,
        fastestTrace: undefined
      };
    }
    
    const durations = allTraces.map(t => t.duration!);
    const totalDuration = durations.reduce((a, b) => a + b, 0);
    
    const slowest = allTraces.reduce((prev, current) => 
      (current.duration! > prev.duration!) ? current : prev
    );
    
    const fastest = allTraces.reduce((prev, current) => 
      (current.duration! < prev.duration!) ? current : prev
    );
    
    return {
      totalTraces: allTraces.length,
      avgDuration: totalDuration / allTraces.length,
      slowestTrace: slowest,
      fastestTrace: fastest
    };
  }
}

// 🎮 MONITORING DEMO
class MonitoringDemo {
  private healthChecker = new HealthChecker();
  private systemMonitor = new SystemMonitor();
  private logger = new StructuredLogger();
  private tracer = new PerformanceTracer();
  
  async setupHealthChecks(): Promise<void> {
    console.log('🏥 Setting up health checks...');
    
    // Database health check
    this.healthChecker.registerHealthCheck('database', async () => {
      // Simulate database check
      await new Promise(resolve => setTimeout(resolve, 50));
      const isHealthy = Math.random() > 0.1; // 90% chance healthy
      
      return {
        healthy: isHealthy,
        message: isHealthy ? 'Database connected' : 'Database connection failed',
        details: {
          connectionPool: isHealthy ? 'active' : 'inactive',
          responseTime: Math.round(Math.random() * 100)
        }
      };
    });
    
    // Memory health check
    this.healthChecker.registerHealthCheck('memory', async () => {
      const memoryUsage = Math.random() * 100;
      const isHealthy = memoryUsage < 80;
      
      return {
        healthy: isHealthy,
        message: `Memory usage: ${memoryUsage.toFixed(1)}%`,
        details: {
          usage: memoryUsage,
          threshold: 80
        }
      };
    });
    
    // API health check
    this.healthChecker.registerHealthCheck('api', async () => {
      // Simulate API check
      await new Promise(resolve => setTimeout(resolve, 30));
      const isHealthy = Math.random() > 0.05; // 95% chance healthy
      
      return {
        healthy: isHealthy,
        message: isHealthy ? 'API endpoints responding' : 'API endpoints failing',
        details: {
          endpointsChecked: 5,
          endpointsPassing: isHealthy ? 5 : 3
        }
      };
    });
  }
  
  setupMetricsAndAlerts(): void {
    console.log('📊 Setting up metrics and alerts...');
    
    // Set up alerts
    this.systemMonitor.addAlert('response_time', 'above', 1000);
    this.systemMonitor.addAlert('memory_usage', 'above', 80);
    this.systemMonitor.addAlert('cpu_usage', 'above', 90);
    this.systemMonitor.addAlert('error_rate', 'above', 5);
  }
  
  async simulateApplicationTraffic(): Promise<void> {
    console.log('🚗 Simulating application traffic...');
    
    // Simulate different scenarios
    for (let i = 0; i < 20; i++) {
      const scenario = Math.random();
      
      if (scenario < 0.1) {
        // 10% chance of slow request
        await this.simulateSlowRequest();
      } else if (scenario < 0.15) {
        // 5% chance of error
        await this.simulateErrorRequest();
      } else {
        // 85% chance of normal request
        await this.simulateNormalRequest();
      }
      
      // Record system metrics
      this.recordSystemMetrics();
      
      // Small delay between requests
      await new Promise(resolve => setTimeout(resolve, 100));
    }
  }
  
  private async simulateNormalRequest(): Promise<void> {
    const traceId = this.tracer.startTrace('user_request', {
      userId: 'user123',
      endpoint: '/api/users'
    });
    
    this.tracer.addTraceLog(traceId, 'Processing user request');
    
    // Simulate processing time
    const responseTime = 100 + Math.random() * 200;
    await new Promise(resolve => setTimeout(resolve, responseTime));
    
    this.tracer.addTraceLog(traceId, 'Database query completed');
    this.tracer.addTraceLog(traceId, 'Response sent to client');
    
    this.tracer.endTrace(traceId);
    
    // Record metrics
    this.systemMonitor.recordMetric('response_time', responseTime);
    this.systemMonitor.recordMetric('requests_per_second', 1);
    
    // Log success
    this.logger.info('Request processed successfully', {
      responseTime,
      endpoint: '/api/users'
    });
  }
  
  private async simulateSlowRequest(): Promise<void> {
    const traceId = this.tracer.startTrace('slow_request', {
      userId: 'user456',
      endpoint: '/api/reports'
    });
    
    this.tracer.addTraceLog(traceId, 'Processing complex report request');
    
    // Simulate slow processing
    const responseTime = 1000 + Math.random() * 2000;
    await new Promise(resolve => setTimeout(resolve, responseTime));
    
    this.tracer.addTraceLog(traceId, 'Heavy database query completed');
    this.tracer.endTrace(traceId);
    
    // Record metrics
    this.systemMonitor.recordMetric('response_time', responseTime);
    
    // Log warning
    this.logger.warn('Slow request detected', {
      responseTime,
      endpoint: '/api/reports'
    });
  }
  
  private async simulateErrorRequest(): Promise<void> {
    const traceId = this.tracer.startTrace('error_request', {
      userId: 'user789',
      endpoint: '/api/data'
    });
    
    this.tracer.addTraceLog(traceId, 'Processing data request');
    
    // Simulate error
    const responseTime = 50 + Math.random() * 100;
    await new Promise(resolve => setTimeout(resolve, responseTime));
    
    this.tracer.addTraceLog(traceId, 'Error occurred during processing');
    this.tracer.endTrace(traceId);
    
    // Record metrics
    this.systemMonitor.recordMetric('response_time', responseTime);
    this.systemMonitor.recordMetric('error_rate', 1);
    
    // Log error
    this.logger.error('Request failed', {
      responseTime,
      endpoint: '/api/data',
      error: 'Database connection timeout'
    });
  }
  
  private recordSystemMetrics(): void {
    // Simulate system metrics
    const memoryUsage = 50 + Math.random() * 40; // 50-90%
    const cpuUsage = 20 + Math.random() * 60; // 20-80%
    
    this.systemMonitor.recordMetric('memory_usage', memoryUsage);
    this.systemMonitor.recordMetric('cpu_usage', cpuUsage);
  }
  
  async runDemo(): Promise<void> {
    console.log('🎮 Starting Monitoring Demo!\n');
    
    // Setup
    await this.setupHealthChecks();
    this.setupMetricsAndAlerts();
    
    // Run health checks
    console.log('\n🏥 Initial Health Check:');
    await this.healthChecker.runAllHealthChecks();
    
    // Simulate traffic
    console.log('\n🚗 Simulating Application Traffic:');
    await this.simulateApplicationTraffic();
    
    // Show results
    console.log('\n📊 Final Results:');
    
    // Health check summary
    const healthSummary = this.healthChecker.getHealthSummary();
    console.log(`🏥 Health checks: ${healthSummary.totalChecks} registered`);
    
    // Metrics summary
    const metricsSummary = this.systemMonitor.getAllMetrics();
    console.log('📊 Metrics summary:');
    for (const [name, stats] of Object.entries(metricsSummary)) {
      console.log(`   ${name}: avg=${stats.avg.toFixed(2)}, max=${stats.max.toFixed(2)}`);
    }
    
    // Trace statistics
    const traceStats = this.tracer.getTraceStats();
    console.log(`🎯 Traces: ${traceStats.totalTraces} completed`);
    console.log(`   Average duration: ${traceStats.avgDuration.toFixed(2)}ms`);
    if (traceStats.slowestTrace) {
      console.log(`   Slowest trace: ${traceStats.slowestTrace.name} (${traceStats.slowestTrace.duration?.toFixed(2)}ms)`);
    }
    
    // Log statistics
    const logStats = this.logger.getLogStats();
    console.log(`📝 Logs: ${logStats.total} total`);
    console.log(`   Errors: ${logStats.byLevel.error}, Warnings: ${logStats.byLevel.warn}`);
    
    console.log('\n🎉 Monitoring demo complete!');
  }
}

// 🎯 TYPE DEFINITIONS
interface HealthStatus {
  healthy: boolean;
  message: string;
  details?: any;
}

interface HealthResult {
  name: string;
  status: 'healthy' | 'unhealthy';
  message: string;
  duration: number;
  timestamp: string;
  details?: any;
  error?: string;
}

interface OverallHealth {
  status: 'healthy' | 'unhealthy';
  timestamp: string;
  totalChecks: number;
  healthyChecks: number;
  unhealthyChecks: number;
  checks: HealthResult[];
}

interface HealthSummary {
  totalChecks: number;
  lastResults: HealthResult[];
  timestamp: string;
}

interface MetricStats {
  name: string;
  count: number;
  min: number;
  max: number;
  avg: number;
  median: number;
  latest: number;
  timestamp: string;
}

interface Alert {
  id: string;
  metricName: string;
  condition: 'above' | 'below';
  threshold: number;
  active: boolean;
  created: string;
  triggered?: string;
}

type LogLevel = 'debug' | 'info' | 'warn' | 'error';

interface LogEntry {
  timestamp: string;
  level: LogLevel;
  message: string;
  meta?: Record<string, any>;
  correlationId: string;
}

interface LogStats {
  total: number;
  byLevel: Record<LogLevel, number>;
  timeRange: {
    oldest?: string;
    newest?: string;
  };
}

interface TraceSpan {
  traceId: string;
  name: string;
  startTime: number;
  endTime?: number;
  duration?: number;
  tags: Record<string, string>;
  logs: Array<{
    timestamp: number;
    message: string;
    data?: any;
  }>;
}

interface TraceStats {
  totalTraces: number;
  avgDuration: number;
  slowestTrace?: TraceSpan;
  fastestTrace?: TraceSpan;
}

// 🎮 RUN MONITORING DEMO
async function runMonitoringDemo(): Promise<void> {
  console.log('🎮 Monitoring and Observability Demo Starting!\n');
  
  const demo = new MonitoringDemo();
  await demo.runDemo();
  
  console.log('\n📚 Key Takeaways:');
  console.log('   ✅ Monitor application health with regular checks');
  console.log('   ✅ Use structured logging for better debugging');
  console.log('   ✅ Collect metrics to understand performance');
  console.log('   ✅ Set up alerts for critical issues');
  console.log('   ✅ Use tracing to follow request journeys');
  console.log('   ✅ Monitor both technical and business metrics');
  console.log('\n🎉 Monitoring demo complete!');
}

// Run the demo
runMonitoringDemo().catch(console.error);
