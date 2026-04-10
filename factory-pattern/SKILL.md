---
name: factory-pattern
description: Factory design pattern. Apply when you need to centralize object creation, decouple consumers from concrete types, or build config-driven instantiation. Covers factory functions, abstract factories, and builder patterns.
---

# Factory Pattern

Centralize object creation logic so consumers don't need to know about concrete types or complex construction steps.

## When to Apply

- Object creation requires complex setup or configuration
- The concrete type depends on runtime conditions (config, environment, user input)
- You want to decouple consumers from specific implementations
- You need to enforce invariants during construction (validation, defaults)

## TypeScript Implementation

### Simple Factory Function (Most Common)

```typescript
type Logger = {
  info(msg: string): void
  error(msg: string): void
}

function createLogger(env: 'production' | 'development'): Logger {
  if (env === 'production') {
    return {
      info: (msg) => externalService.log('info', msg),
      error: (msg) => externalService.log('error', msg),
    }
  }
  return {
    info: (msg) => console.log(`[INFO] ${msg}`),
    error: (msg) => console.error(`[ERROR] ${msg}`),
  }
}

// Consumer doesn't know or care about the concrete type
const logger = createLogger(process.env.NODE_ENV)
logger.info('Started')
```

### Registry Factory (Open/Closed)

```typescript
type NotificationChannel = {
  send(to: string, message: string): Promise<void>
}

// Registry — new channels added without modifying existing code
const channels: Record<string, () => NotificationChannel> = {
  email: () => new EmailChannel(smtpConfig),
  slack: () => new SlackChannel(webhookUrl),
  sms: () => new SmsChannel(twilioClient),
}

function createChannel(type: string): NotificationChannel {
  const factory = channels[type]
  if (!factory) throw new Error(`Unknown channel: ${type}`)
  return factory()
}
```

### Builder Pattern (Many Optional Fields)

```typescript
type QueryConfig = {
  table: string
  where: string[]
  orderBy: string | null
  limit: number | null
}

function queryBuilder(table: string) {
  const config: QueryConfig = { table, where: [], orderBy: null, limit: null }

  return {
    where(condition: string) { config.where.push(condition); return this },
    orderBy(field: string) { config.orderBy = field; return this },
    limit(n: number) { config.limit = n; return this },
    build(): QueryConfig { return { ...config } },
  }
}

const query = queryBuilder('users')
  .where('active = true')
  .orderBy('created_at')
  .limit(50)
  .build()
```

## Rust Implementation

```rust
// Factory function
fn create_storage(config: &Config) -> Box<dyn Storage> {
    match config.storage_type.as_str() {
        "s3" => Box::new(S3Storage::new(&config.s3_bucket)),
        "local" => Box::new(LocalStorage::new(&config.data_dir)),
        _ => Box::new(MemoryStorage::new()),
    }
}

// Builder pattern (common in Rust)
struct ServerBuilder {
    port: u16,
    workers: usize,
    tls: Option<TlsConfig>,
}

impl ServerBuilder {
    fn new(port: u16) -> Self {
        Self { port, workers: 4, tls: None }
    }
    fn workers(mut self, n: usize) -> Self { self.workers = n; self }
    fn tls(mut self, config: TlsConfig) -> Self { self.tls = Some(config); self }
    fn build(self) -> Result<Server, ConfigError> { /* validate and construct */ }
}
```

## Anti-Patterns to Avoid

- **Factory that always returns the same type** — just use a constructor
- **Factory with boolean flags** — use separate factory functions instead of `createWidget(true, false, true)`
- **Abstract factory when you only have one product family** — wait until you need it
