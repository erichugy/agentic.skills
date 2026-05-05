# Imports and Validation

Detailed import organization and runtime validation rules.

## Import Ordering

Group imports with blank lines between groups, alphabetized within each:

```typescript
// 1. Builtin / external
import { readFile } from 'node:fs/promises'
import { z } from 'zod'

// 2. Parent directories
import { AppConfig } from '../config'

// 3. Sibling / index
import { helpers } from './helpers'
```

Use `import type` for type-only imports:

```typescript
import type React from 'react'
import type { FC, ReactNode } from 'react'
```

## Zod Validation — All External Data

All external API responses, user inputs, and deserialized data MUST be validated with Zod:

```typescript
const UserSchema = z.object({
  id: z.string(),
  name: z.string().trim().min(1).max(200),
  email: z.string().email().max(320),
})

type User = z.infer<typeof UserSchema>

const user = UserSchema.parse(data)
```

Use `.parse()` when invalid data should fail fast. Use `.safeParse()` when you need a fallback path.

For boundary-owned shapes, let the schema name reflect the domain behavior instead of the transport history. Prefer `InitMessageRequestBodySchema` over a stale name like `BootstrapMessageRequestBodySchema` once the meaning is clearly "init payload".

## No `as` Type Assertions

`as` bypasses type checking. Use Zod or type guards instead:

```typescript
// BAD
const user = response.data as User

// GOOD
const user = UserSchema.parse(response.data)
```
