---
name: strategy-pattern
description: Strategy design pattern. Apply when you need swappable algorithms, conditional behavior based on type/mode, or want to eliminate growing switch/if chains. Covers implementation in TypeScript and Rust.
---

# Strategy Pattern

Encapsulate a family of algorithms behind a common interface, making them interchangeable at runtime. Use this to eliminate growing switch/if chains and make behavior pluggable.

## When to Apply

- A function has a growing `switch` or `if/else` chain based on a type or mode
- You need to swap behavior at runtime (e.g., different export formats, pricing strategies, auth methods)
- You're adding a new variant and don't want to modify existing code (Open/Closed principle)

## TypeScript Implementation

### Function-Based (Preferred)

```typescript
// Define the strategy interface
type PricingStrategy = (basePrice: number, quantity: number) => number

// Implement strategies as functions
const regularPricing: PricingStrategy = (price, qty) => price * qty
const bulkPricing: PricingStrategy = (price, qty) => price * qty * 0.9
const premiumPricing: PricingStrategy = (price, qty) => price * qty * 0.8

// Registry — add new strategies without modifying existing code
const strategies: Record<string, PricingStrategy> = {
  regular: regularPricing,
  bulk: bulkPricing,
  premium: premiumPricing,
}

// Usage
function calculateTotal(tier: string, price: number, qty: number): number {
  const strategy = strategies[tier]
  if (!strategy) throw new Error(`Unknown pricing tier: ${tier}`)
  return strategy(price, qty)
}
```

### Class-Based (When State is Needed)

```typescript
type Exporter = {
  export(data: Data): string
  mimeType: string
}

const csvExporter: Exporter = {
  export: (data) => data.rows.map(r => r.join(',')).join('\n'),
  mimeType: 'text/csv',
}

const jsonExporter: Exporter = {
  export: (data) => JSON.stringify(data, null, 2),
  mimeType: 'application/json',
}

function exportData(data: Data, exporter: Exporter): Response {
  const body = exporter.export(data)
  return new Response(body, {
    headers: { 'Content-Type': exporter.mimeType },
  })
}
```

## Rust Implementation

```rust
// Trait as strategy interface
trait Compressor {
    fn compress(&self, data: &[u8]) -> Vec<u8>;
    fn name(&self) -> &str;
}

struct GzipCompressor;
impl Compressor for GzipCompressor {
    fn compress(&self, data: &[u8]) -> Vec<u8> { /* ... */ }
    fn name(&self) -> &str { "gzip" }
}

struct ZstdCompressor { level: i32 }
impl Compressor for ZstdCompressor {
    fn compress(&self, data: &[u8]) -> Vec<u8> { /* ... */ }
    fn name(&self) -> &str { "zstd" }
}

// Accept any strategy via trait object or generics
fn process(data: &[u8], compressor: &dyn Compressor) -> Vec<u8> {
    compressor.compress(data)
}
```

## Anti-Patterns to Avoid

- **Strategy with one implementation** — premature abstraction. Wait until you have 2+.
- **Passing strategy name as string then switching** — defeats the purpose. Pass the strategy directly.
- **Deep inheritance for strategies** — use composition. Strategies should be flat.
