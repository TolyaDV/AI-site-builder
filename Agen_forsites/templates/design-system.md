# Design System — {{ PROJECT_NAME }}

> Згенеровано: {{ DATE }}
> Джерело: `brief.md`
> Фіксовані дефолти (spacing, breakpoints, shadows, motion, CVA-patterns) — див. `.claude/references/design-defaults.md`.

## Brand archetype

Обери один (впливає на шрифти, кольори, radius-стиль):

- **tech** — геометричний sans-serif (Inter/Geist), холодні нейтралі, blue/violet primary, середні radius.
- **luxury** — serif display (Playfair/Cormorant) + sans-serif body, теплі приглушені кольори, малі radius.
- **editorial** — serif display + humanist sans, високий контраст, великі заголовки, середні radius.
- **playful** — geometric rounded sans (Nunito/Poppins), насичені кольори, великі radius.
- **neutral-pro** — Inter/Manrope, сіра шкала + 1 accent, середні radius.
- **technical** — mono (JetBrains Mono/Fira Code) для акцентів + sans для body, монохром + 1 accent, малі radius.

Обрано: `{{ ARCHETYPE }}`

## Палітра — Light

### Brand
| Token | OKLCH | HEX |
|---|---|---|
| `primary-50`…`950` | `oklch({{ L }} {{ C }} {{ H }})` | `{{ HEX }}` |
| `accent-50`…`950`  | `oklch({{ ... }})` | `{{ ... }}` |
| `neutral-50`…`950` | `oklch({{ ... }})` | `{{ ... }}` |

### Semantic
| Token | OKLCH | HEX |
|---|---|---|
| `success` | `{{ ... }}` | `{{ ... }}` |
| `warning` | `{{ ... }}` | `{{ ... }}` |
| `error`   | `{{ ... }}` | `{{ ... }}` |
| `info`    | `{{ ... }}` | `{{ ... }}` |

### Surface
| Token | OKLCH | HEX |
|---|---|---|
| `background` / `foreground`   | … | … |
| `card` / `card-foreground`    | … | … |
| `muted` / `muted-foreground`  | … | … |
| `border` / `ring`             | … | … |

## Палітра — Dark

**Правило генерації**: `L_dark = 1 - L_light`, `C_dark = C_light * 0.85`, hue незмінний. Surface: dark `background` L~0.12, dark `foreground` L~0.95.

(таблиці структурою як Light)

## Перевірка контрасту

| Pair | Light | Dark |
|---|---|---|
| `foreground/background`      | `{{ X.X }}:1` | `{{ X.X }}:1` |
| `primary-foreground/primary` | `{{ X.X }}:1` | `{{ X.X }}:1` |
| `muted-foreground/muted`     | `{{ X.X }}:1` | `{{ X.X }}:1` |

Усі ≥4.5:1 (body). <4.5 — переглянь палітру.

## Типографіка

- **Heading**: `{{ FONT }}` via `next/font/google`, weight `{{ 600/700 }}`, letter-spacing `-0.02em`.
- **Body**: `{{ FONT }}`, weight `400/500`.
- **Mono** (опц.): `{{ FONT }}`.

## Radius style

`{{ rounded | slightly-rounded | sharp }}` — визначає глобальний `--radius`.

## Component overrides

Тільки патерни, що ВІДХИЛЯЮТЬСЯ від дефолтів у `.claude/references/design-defaults.md`. Якщо все стандартно — залиш порожнім.

---

## Checklist перед затвердженням

- [ ] Усі `{{ PLACEHOLDER }}` замінені.
- [ ] Archetype обраний.
- [ ] Контраст-таблиця ≥4.5:1 всюди.
- [ ] Dark-палітра згенерована за правилом.
- [ ] `tailwind.config.ts` і `globals.css` синхронізовані.
