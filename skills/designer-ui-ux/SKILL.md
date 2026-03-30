---
name: designer-ui-ux
trigger: design this page, create a new UI component, new page, form, interactive element, review the UI, check UX, test the flow
description: Designs UI components and pages for Nadistudio using shadcn/ui + Tailwind, following the LEGO GRANDE rules. Includes UX review capabilities (Miche test, clarity, flow, accessibility, mobile-first).
origin: Nadistudio
---

# Designer UI/UX

## 1. When to Use

Activate this skill when:
- User says "design this page" or "create a new UI component"
- Need a new page, form, or interactive element
- shadcn/ui components are available for reuse
- User says "review the UI", "check UX", or "test the flow"
- Before launching a new feature
- When users report confusion about the interface

## 2. Guidelines

### Stack Rules

| Rule | Why |
|------|-----|
| shadcn/ui components | Consistent, accessible |
| Tailwind inline classes | LEGO GRANDE |
| No separate CSS files | Keep it simple |
| Mobile-first | 375px first, scale up |
| Server Components when possible | Performance |

### LEGO GRANDE Rules

- **LEGO GRANDE**: inline Tailwind, no CSS files
- **shadcn/ui**: reuse components
- **Mobile-first**: start small, scale up
- **Spanish labels**: all UI text

### Component Patterns

#### Card with Hover
```tsx
<Card className="hover:shadow-md transition cursor-pointer">
  <CardContent>...</CardContent>
</Card>
```

#### Form with Labels
```tsx
<div className="space-y-4">
  <div>
    <Label htmlFor="name">Nombre del producto</Label>
    <Input id="name" placeholder="Ej: Jeans Azul" />
  </div>
</div>
```

#### Loading State
```tsx
{isLoading ? (
  <div className="animate-pulse space-y-4">
    <div className="h-4 bg-gray-200 rounded w-3/4" />
    <div className="h-4 bg-gray-200 rounded w-1/2" />
  </div>
) : content}
```

#### Empty State
```tsx
<div className="text-center py-12">
  <p className="text-gray-500">No hay productos todavía</p>
  <Button asChild className="mt-4">
    <Link href="/admin/catalogo/nuevo">Subir fotos</Link>
  </Button>
</div>
```

### Color System

Use theme variables:
- `primary` — brand color
- `muted` — backgrounds
- `destructive` — errors, delete
- `success` — confirmations (custom green)

### UX Review Checklist

#### The Miche Test

**Can a non-technical shop owner do this without help?**

If not, simplify.

#### Clarity
- [ ] Labels are in Spanish, clear, no jargon
- [ ] Buttons say what they do ("Publicar", not "Submit")
- [ ] Error messages explain the problem + solution
- [ ] Empty states guide the user ("No hay productos, sube fotos")

#### Error States
- [ ] Use `templates/components/error-boundary.tsx` — React error boundary with Spanish UI for global error handling

#### Flow
- [ ] User knows what to do next
- [ ] No dead ends (every action has a next step)
- [ ] Confirmation before destructive actions
- [ ] Progress indicators for async operations

#### Accessibility
- [ ] Touch targets ≥ 44px
- [ ] Sufficient color contrast
- [ ] Labels on all inputs
- [ ] Loading states visible

#### Mobile-First
- [ ] Works on 375px width
- [ ] No horizontal scroll
- [ ] Thumb-friendly navigation
- [ ] No tiny text

#### Common Issues

| Issue | Fix |
|-------|-----|
| "No entendí" in bot | Add inline buttons |
| Empty page with no guidance | Add empty state with CTA |
| Button says "Submit" | Change to "Publicar producto" |
| Loading without indicator | Add spinner or skeleton |
| Form too long | Split into steps |

## 3. Information Gaps — Catastro

**STOP. Do not proceed if any of the following is missing:**

| Required Info | Why It Matters | Where to Find It |
|---------------|----------------|------------------|
| Target user profile (shop owner vs admin) | Affects the Miche test threshold and complexity tolerance | Ask user, reference nadi-strategic skill |
| Device context (mobile-only vs desktop) | Determines responsive breakpoints and interaction patterns | User requirements, analytics |
| Brand color values | Required for `primary` theme variable | `tailwind.config.ts`, CSS variables, or brand guidelines |
| Content language scope (Spanish-only vs bilingual) | All UI labels must be consistent | Project README, nadi-operational skill |
| Existing shadcn/ui components in project | Avoid duplicating functionality | Check `components/ui/` directory |
| Accessibility requirements (WCAG level) | Legal and ethical compliance requirements | Project specs, client requirements |
| Performance constraints (slow connections) | Determines loading state complexity | Target market infrastructure |

**If gaps detected:** Ask the user for clarification before generating any code.
