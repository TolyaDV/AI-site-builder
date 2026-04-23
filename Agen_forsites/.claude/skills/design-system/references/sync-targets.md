# Sync targets — куди писати яку зміну

Коли скіл змінює щось у дизайн-системі, **усі пов'язані файли мусять оновитись у тому ж turn**. Якщо оновити тільки `design-system.md` — токени розсинхронізуються з кодом і сайт зламається (Tailwind не знає нового значення).

Ця матриця — довідка для всіх трьох режимів (init / intake / revise).

---

## Матриця «тип зміни → файли»

| Зміна | `design-system.md` | `globals.css` | `tailwind.config.ts` | `layout.tsx` | components |
|---|:---:|:---:|:---:|:---:|:---:|
| Primary color | ✅ | ✅ `:root` + `.dark` | ✅ `theme.extend.colors.primary` | — | — |
| Accent color | ✅ | ✅ `:root` + `.dark` | ✅ `theme.extend.colors.accent` | — | — |
| Neutral shades | ✅ | ✅ `background/foreground/muted/border` | ✅ `theme.extend.colors.neutral` | — | — |
| Semantic (success/warning/error) | ✅ | ✅ `--destructive` + custom | ✅ `theme.extend.colors` | — | — |
| Heading font | ✅ | — | ✅ `fontFamily.heading` | ✅ `next/font` import + variable | — |
| Body font | ✅ | — | ✅ `fontFamily.sans` | ✅ `next/font` import + variable | — |
| Mono font | ✅ | — | ✅ `fontFamily.mono` | ✅ `next/font` | — |
| Radius | ✅ | ✅ `--radius` | ✅ `borderRadius` | — | — |
| Shadow scale | ✅ | — | ✅ `boxShadow` (лише якщо кастом) | — | — |
| Motion (duration/easing) | ✅ | — | ✅ `transitionDuration/TimingFunction` | — | — |
| Spacing scale | ✅ | — | ✅ `spacing` (лише якщо кастом) | — | — |
| Breakpoints | ✅ | — | ✅ `screens` | — | — |
| Component override (Button/Card/Section) | ✅ `Component overrides` | — | — | — | ✅ cva у `src/components/ui/<name>.tsx` |
| Component новий (поза defaults) | ✅ нова секція | — | — | — | ✅ новий файл у `src/components/ui/` |

---

## Деталі по кожному файлу

### `design-system.md` (source of truth)

**Завжди оновлюється.** Без змін тут — решта sync втрачає сенс.

Структуру див. у [templates/design-system.md](../../../templates/design-system.md). Оновлюй конкретні рядки таблиць/секцій, не переписуй файл цілком (щоб diff був читабельний).

### `src/app/globals.css`

**Формат залежить від shadcn-версії** — перевір через `components.json` + `package.json`:
- shadcn v2+ / Tailwind v4 → OKLCH tuple: `--primary: 0.65 0.18 250;`
- shadcn v1 / Tailwind v3 → HSL tuple: `--primary: 222 47% 11%;`

**Секції які змінюються:**
```css
:root {
  --background: ...;
  --foreground: ...;
  --primary: ...;
  --primary-foreground: ...;
  --accent: ...;
  --radius: 0.5rem;
  /* ... */
}

.dark {
  /* парні значення за правилом L_dark = 1 - L_light, C_dark = C_light * 0.85 */
}
```

**Якщо файлу не існує** — скопіюй [references/globals.reference.css](../../../references/globals.reference.css) і підстав нові значення, не пиши з нуля.

**НЕ чіпай** блоки `@media (prefers-reduced-motion)`, `:focus-visible`, `html { scroll-behavior }` — це фіксовані правила з reference-шаблону.

### `tailwind.config.ts`

**Секція `theme.extend`** — додавай тільки відхилення від дефолтів:

```typescript
theme: {
  extend: {
    colors: {
      primary: "oklch(var(--primary) / <alpha-value>)",  // shadcn v2+
      // або HSL: primary: "hsl(var(--primary) / <alpha-value>)"
      "primary-foreground": "oklch(var(--primary-foreground) / <alpha-value>)",
      accent: "oklch(var(--accent) / <alpha-value>)",
      // ...
    },
    fontFamily: {
      sans: ["var(--font-sans)", "system-ui", "sans-serif"],
      heading: ["var(--font-heading)", "serif"],
      mono: ["var(--font-mono)", "monospace"],
    },
    borderRadius: {
      lg: "var(--radius)",
      md: "calc(var(--radius) - 2px)",
      sm: "calc(var(--radius) - 4px)",
    },
    transitionDuration: {
      // тільки якщо кастом
    },
  },
}
```

**НЕ перевизначай дефолти Tailwind** (spacing, breakpoints, font-size) — вони зафіксовані у [references/design-defaults.md](../../../references/design-defaults.md).

### `src/app/layout.tsx`

**Оновлюється тільки при зміні шрифту.**

```typescript
import { Inter, Playfair_Display } from "next/font/google";

const fontSans = Inter({
  subsets: ["latin"],
  variable: "--font-sans",
  display: "swap",
});

const fontHeading = Playfair_Display({
  subsets: ["latin"],
  variable: "--font-heading",
  display: "swap",
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="uk" className={`${fontSans.variable} ${fontHeading.variable}`}>
      <body className="font-sans antialiased">{children}</body>
    </html>
  );
}
```

**Старий шрифт видаляй** з import-ів, щоб bundle не розростався.

### Компоненти — `src/components/ui/<name>.tsx`

**Новий cva-варіант або override — тільки якщо є у `design-system.md` → `Component overrides`.**

Якщо компонент не існує — створи на основі CVA-defaults з [references/design-defaults.md](../../../references/design-defaults.md), потім додавай override.

Приклад override Button:

```typescript
// src/components/ui/button.tsx
import { cva } from "class-variance-authority";

export const button = cva(
  "inline-flex items-center justify-center rounded-md font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:opacity-50 disabled:pointer-events-none",
  {
    variants: {
      variant: {
        primary: "bg-primary text-primary-foreground hover:bg-primary/90",
        // override з design-system.md →
        glass: "bg-white/10 backdrop-blur border border-white/20 text-white hover:bg-white/20",
        // ...
      },
      // ...
    },
    defaultVariants: { variant: "primary", size: "md" },
  }
);
```

---

## Алгоритм sync

1. Визнач тип зміни (подивись у матрицю).
2. Склади список файлів які треба оновити.
3. Для кожного файлу:
   - Прочитай поточний стан.
   - Застосуй зміну (Edit tool з точним рядком).
4. **Sync-check**: прогрепай новий токен у всіх файлах → має бути присутній всюди де матриця вимагає.
5. Валідація:
   - `tsc --noEmit` (якщо зачеплений .tsx).
   - WCAG-контраст (якщо колір).
6. Покажи diff кожного файлу юзеру.

---

## Anti-patterns

- **Оновити тільки `design-system.md` і забути про код** — найчастіша помилка. Браузер покаже старі кольори, хоча у доку вже нові.
- **Використати `git add .`** після синхронізації — додасть зайві файли. Завжди `git add <specific files>`.
- **Переписати `globals.css` з нуля** замість точкового оновлення `:root/.dark` — втрачаються кастомні правила які юзер міг додати.
- **Додавати до `tailwind.config.ts` дефолти які вже зафіксовані** (spacing scale, breakpoints з design-defaults.md) — створює плутанину.
- **Пропустити sync-check** — після 3-4 правок гарантовано щось розсинхронізується.
