---
name: design-patterns
description: Index of all design pattern skills. Read this to understand which patterns exist and when to apply each one. Use when planning architecture or reviewing code structure.
---

# Design Patterns — Index

Read this to understand which pattern skills are available and when to reach for each one. Then assign the specific pattern skill(s) to the relevant agent.

## Available Pattern Skills

### Structural & Behavioral

| Pattern | Skill | When to Use | Key Signal |
|---------|-------|-------------|------------|
| SOLID Principles | `/solid` | Module design, class architecture, dependency management | "This class does too much", tight coupling, growing switch chains |
| Composition | `/composition` | Sharing behavior, code reuse, class hierarchies | Deep inheritance trees, "extends" chains, shared behavior across unrelated types |
| Strategy | `/strategy-pattern` | Swappable algorithms, mode-dependent behavior | Growing if/switch based on type, runtime algorithm selection |
| Factory | `/factory-pattern` | Object creation, config-driven instantiation | Complex constructors, type depends on runtime conditions |
| Observer | `/observer-pattern` | Event systems, reactive state, notifications | "When X happens, do Y and Z", decoupled reactions to state changes |

## Pattern Selection Guide

### "I have a growing switch/if chain"
→ Use **Strategy Pattern** to encapsulate each branch as a pluggable strategy

### "I need different objects based on config/environment"
→ Use **Factory Pattern** to centralize creation logic

### "Multiple things need to react to the same event"
→ Use **Observer Pattern** to decouple producers from consumers

### "I'm using inheritance for code reuse"
→ Use **Composition** — favor "has-a" over "is-a"

### "This module is getting too big"
→ Apply **SOLID** (Single Responsibility) to split it

### "I can't test this without standing up the whole system"
→ Apply **SOLID** (Dependency Inversion) to inject abstractions

### "Adding a feature requires changing 5 files"
→ Apply **SOLID** (Open/Closed) to make the system extensible

## Combining Patterns

Patterns work best together:

| Combination | Use Case |
|-------------|----------|
| Factory + Strategy | Create the right strategy based on config |
| Observer + Strategy | Different event handlers swappable at runtime |
| Composition + Factory | Build complex objects from composed parts |
| SOLID + all patterns | SOLID tells you WHEN to apply patterns; patterns tell you HOW |

## When NOT to Use Patterns

- **One-off scripts** — just write procedural code
- **Only one implementation** — don't abstract for hypothetical futures
- **Simple data transformations** — a function is enough, no pattern needed
- **Premature abstraction** — wait until the second or third instance before abstracting

**Rule of thumb**: If the pattern makes the code harder to follow than the problem it solves, don't use it.
