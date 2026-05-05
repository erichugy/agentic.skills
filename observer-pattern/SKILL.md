---
name: observer-pattern
description: Observer/pub-sub design pattern. Apply when building event systems, reactive state, notification systems, or when you need to decouple producers from consumers. Covers event emitters, subscriptions, and cleanup.
---

# Observer Pattern (Pub/Sub)

Decouple producers from consumers by allowing objects to subscribe to events without the producer knowing who's listening.

## When to Apply

- Multiple parts of the system need to react to the same event
- The producer shouldn't know or care about its consumers
- You need loose coupling between modules (e.g., "user signed up" triggers welcome email, analytics, onboarding — none should know about each other)
- Building plugin systems or middleware chains
- Emitting structured operational events such as script failures, persistence warnings, or report-generation failures without coupling workflow code to one telemetry backend

## TypeScript Implementation

### Simple Event Emitter

```typescript
type Listener<T> = (data: T) => void
type Unsubscribe = () => void

function createEventEmitter<Events extends Record<string, unknown>>() {
  const listeners = new Map<keyof Events, Set<Listener<any>>>()

  return {
    on<K extends keyof Events>(event: K, listener: Listener<Events[K]>): Unsubscribe {
      if (!listeners.has(event)) listeners.set(event, new Set())
      listeners.get(event)!.add(listener)
      return () => { listeners.get(event)?.delete(listener) }
    },

    emit<K extends keyof Events>(event: K, data: Events[K]) {
      listeners.get(event)?.forEach(fn => fn(data))
    },
  }
}

// Usage — fully typed
type AppEvents = {
  'user:created': { id: string; email: string }
  'order:completed': { orderId: string; total: number }
}

const bus = createEventEmitter<AppEvents>()

const unsub = bus.on('user:created', (user) => {
  sendWelcomeEmail(user.email)
})

bus.emit('user:created', { id: '1', email: 'user@example.com' })
unsub() // cleanup
```

### React: Custom Hook with Subscription

```typescript
function useEventListener<K extends keyof WindowEventMap>(
  event: K,
  handler: (e: WindowEventMap[K]) => void,
) {
  const savedHandler = useRef(handler)
  savedHandler.current = handler

  useEffect(() => {
    const listener = (e: WindowEventMap[K]) => savedHandler.current(e)
    window.addEventListener(event, listener)
    return () => window.removeEventListener(event, listener)
  }, [event])
}
```

## Rust Implementation

```rust
use std::collections::HashMap;

type Callback<T> = Box<dyn Fn(&T) + Send>;

struct EventBus<T> {
    listeners: HashMap<String, Vec<Callback<T>>>,
}

impl<T> EventBus<T> {
    fn new() -> Self {
        Self { listeners: HashMap::new() }
    }

    fn on(&mut self, event: &str, callback: impl Fn(&T) + Send + 'static) {
        self.listeners
            .entry(event.to_string())
            .or_default()
            .push(Box::new(callback));
    }

    fn emit(&self, event: &str, data: &T) {
        if let Some(listeners) = self.listeners.get(event) {
            for listener in listeners {
                listener(data);
            }
        }
    }
}
```

## Critical Rules

### Type Operational Events Like Domain Events

If the event is important enough to drive alerts, dashboards, or review feedback, give it the same rigor as product-domain events:
- define event names and scopes once
- prefer typed payloads over loosely assembled tag maps
- promote repeated boundaries like `top_level` or `persistence_sink` into shared constants or schema-backed enums
- keep one helper responsible for mapping domain failures into telemetry events

This avoids repeated string drift across scripts, sinks, and observability adapters.

### Always Clean Up Subscriptions

Every `subscribe`/`on` call must have a corresponding unsubscribe, especially in:
- React `useEffect` cleanup returns
- Component unmount handlers
- Test teardown

```typescript
// BAD — memory leak
useEffect(() => {
  bus.on('update', handler)  // never cleaned up
}, [])

// GOOD
useEffect(() => {
  const unsub = bus.on('update', handler)
  return unsub
}, [])
```

### Avoid Circular Events

If event A triggers event B which triggers event A, you have an infinite loop. Guard against this with:
- A `processing` flag that prevents re-entry
- Clearly defined event flow direction (unidirectional)

### Size Caps on Listener Collections

In long-running processes, cap the number of listeners to prevent memory leaks:

```typescript
const MAX_LISTENERS = 100
if (listeners.size >= MAX_LISTENERS) {
  console.warn(`Max listeners (${MAX_LISTENERS}) reached for event: ${event}`)
}
```

## Anti-Patterns to Avoid

- **God event bus** — one bus for everything. Scope buses by domain.
- **Events for synchronous control flow** — if you need a return value, use a function call, not an event.
- **String-typed events without a type map** — leads to typos and no type safety.
- **Hand-built telemetry tags in every caller** — centralize event shaping so producers emit one domain event instead of rebuilding observability payloads ad hoc.
