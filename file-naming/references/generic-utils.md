# Generic Utils Boundaries

Use this reference when deciding whether code belongs in `utils/`.

## What Belongs In `utils/`

`utils/` is for small helpers that are:

- project-agnostic
- domain-agnostic
- easy to imagine reusing in many codebases
- too small to deserve a service or feature module

Examples:

- `utils/datetime.ts`
- `utils/io.ts`
- `utils/string-formatting.ts`
- `utils/parse-json.ts`

## What Does Not Belong In `utils/`

Do not put code in `utils/` just because it is small.

Keep code out of `utils/` when it is:

- tied to one feature or route
- tied to one API or vendor
- part of business logic or workflow logic
- a React component or hook
- a hidden core module pretending to be a helper

Examples that should stay near their feature:

- request filtering
- feature-specific serialization
- API client wrappers
- route-specific date filtering rules

## Structure Inside `utils/`

Use purpose-named files:

```text
utils/
  datetime.ts
  io.ts
```

If one area grows, use a small subfolder:

```text
utils/
  datetime/
    format.ts
    parse.ts
    ranges.ts
```

Avoid:

```text
utils/
  helpers.ts
  misc.ts
  shared.ts
```

## Decision Rule

Ask:

1. Would this helper make sense in many projects?
2. Does it avoid product or feature vocabulary?
3. Is it small enough that making a service/module would be overkill?

If yes to all three, `utils/` is a good fit.
