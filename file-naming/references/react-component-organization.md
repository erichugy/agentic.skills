# React Component & Route Organization

Use this reference when organizing React pages, route-local components, or component groups.

## Core Rules

- A non-trivial component should generally live in its own file
- A page-level file should usually compose smaller components instead of containing all markup and logic inline
- If several components exist only to build one composed component, group them in a folder named after that component
- The top-level file for a component group should usually share the folder name
- Keep route-specific UI near the route; keep truly shared UI in shared component areas

## Default Pattern

```text
app/
  home/
    page.tsx
    components/
      hero.tsx
      featured-projects.tsx
      contact-cta.tsx
```

Here, `page.tsx` is the composition root for the route. The smaller sections live beside it because they are owned by that route.

## Component Group Pattern

If several subcomponents exist only for one larger component, group them:

```text
components/
  navbar/
    index.ts
    navbar.tsx
    nav-link.tsx
    mobile-menu.tsx
    theme-toggle.tsx
```

Guidelines:

- `navbar/` carries the domain
- `navbar.tsx` is the top-level composed component
- `nav-link.tsx`, `mobile-menu.tsx`, and `theme-toggle.tsx` are supporting pieces
- `index.ts` is optional but, if present, should only barrel-export

Avoid this:

```text
components/
  Navbar.tsx
  NavbarLink.tsx
  NavbarMobileMenu.tsx
  NavbarThemeToggle.tsx
```

That flattens a component family and forces filenames to carry too much repeated context.

## Route-Local vs Shared

Use route-local placement when:

- the component is only used by one route
- the component exists to assemble one page or one feature
- the component name becomes much cleaner when the route folder provides context

Use shared placement when:

- the component is reused across multiple routes
- it is part of app chrome, shared primitives, or stable shared UI

## Good React Naming Examples

```text
app/experience/components/intro.tsx
app/experience/components/work-experience.tsx
app/requests/page-client.tsx
app/requests/components/filter-bar.tsx
components/navbar/navbar.tsx
components/navbar/nav-link.tsx
```

## Exceptions

You do not need to extract every tiny fragment into its own file. Keep tiny one-off fragments inline when extraction would add ceremony without improving clarity.

The rule is "generally one component per file," not "every JSX block deserves a file."
