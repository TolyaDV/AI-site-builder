# Design defaults (фіксовані для всіх сайтів)

Ці значення не змінюються між проєктами. Не генеруй їх у `design-system.md`, просто використовуй.

## Spacing
Стандартна шкала Tailwind: `0, 0.5, 1, 1.5, 2, 3, 4, 6, 8, 10, 12, 16, 20, 24, 32` (rem).

## Breakpoints
`sm=640, md=768, lg=1024, xl=1280, 2xl=1536` (px).

## Shadows
Стандартні Tailwind `shadow-xs/sm/md/lg/xl/2xl`.

## Font-size scale
`text-xs` (0.75rem) → `text-7xl` (4.5rem), стандарт Tailwind.

## Motion
- Duration: `75/150/300/500/700 ms`
- Easing: `ease-in / ease-out / linear / ease-in-out`
- ОБОВ'ЯЗКОВО додавай `prefers-reduced-motion` override у `globals.css` (є в `globals.reference.css`).

## Focus / Accessibility
- Focus ring: `focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2`.
- Ніколи не прибирай `outline` без заміни.
- Tap targets мінімум 44×44px.
- WCAG AA: body ≥4.5:1, large (≥24px або bold ≥18.66px) ≥3:1.

## Iconography
`lucide-react`, розмір за замовчуванням 20px, stroke 1.5.

## Формат кольорів залежно від shadcn

- **shadcn v2+ / Tailwind v4** → OKLCH tuple (`--primary: 0.65 0.18 250`).
- **shadcn v1 / Tailwind v3** → HSL tuple без функції (`--primary: 222 47% 11%`).

Детектуй версію через `package.json` і `components.json`. Якщо не ясно — запитай.

## CVA defaults

### Button

```ts
const button = cva(
  "inline-flex items-center justify-center rounded-md font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:opacity-50 disabled:pointer-events-none",
  {
    variants: {
      variant: {
        primary:     "bg-primary text-primary-foreground hover:bg-primary/90",
        secondary:   "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost:       "hover:bg-accent hover:text-accent-foreground",
        destructive: "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        link:        "text-primary underline-offset-4 hover:underline",
      },
      size: {
        sm: "h-9 px-3 text-sm",
        md: "h-10 px-4",
        lg: "h-11 px-6 text-lg",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: { variant: "primary", size: "md" },
  }
);
```

### Card

```ts
const card = cva(
  "rounded-lg border border-border bg-card text-card-foreground",
  {
    variants: {
      padding: { sm: "p-4", md: "p-6", lg: "p-8" },
      interactive: {
        true: "transition-shadow hover:shadow-md cursor-pointer",
        false: "",
      },
    },
    defaultVariants: { padding: "md", interactive: false },
  }
);
```

### Section

```ts
const section = cva("w-full", {
  variants: {
    spacing: {
      sm: "py-12",
      md: "py-16 md:py-24",
      lg: "py-24 md:py-32 lg:py-40",
    },
    container: {
      true: "max-w-7xl mx-auto px-4 sm:px-6 lg:px-8",
      false: "",
    },
  },
  defaultVariants: { spacing: "md", container: true },
});
```

Відхилення → тільки в `design-system.md` → `Component overrides`.
