---
description: Згенерувати або оновити design-system.md + синхронізувати Tailwind, globals.css і компоненти
argument-hint: (без аргументів)
---

# /design — дизайн-система

> **Коли викликати**: під час `/new-site` ця команда не потрібна — вона викликається автоматично на Кроці 2. Викликай `/design` окремо тільки щоб **оновити** існуючу систему, **перегенерувати з нуля** або створити її вручну якщо бриф був зроблений без `/new-site`.

## Конвенції (застосовуються у всіх кроках)

- **Формат кольорів у файлах**: OKLCH (`oklch(0.7 0.15 250)`). Для shadcn-проєктів — той формат, який використовує поточна версія shadcn (HSL tuple у старих, OKLCH у нових). У `design-system.md` додатково вказуй HEX-еквіваленти для читабельності.
- **Контраст**: кожна пара text/background з палітри ДОЛЖНА давати ≥4.5:1 (WCAG AA для body). Для великого тексту (≥24px) — ≥3:1. Розраховуй через APCA або WCAG formula перед пропозицією.
- **Темна тема**: завжди генеруй парну темну палітру (не просто інверсія — свідомий набір з тим же контрастом).

## Крок 0: Передумови
- Має існувати `brief.md`. Якщо немає — СТОП.
- Якщо у `brief.md` немає секції `## Референси` — створи її (пусту), щоб було куди писати результат витягування.
- Стан `design-system.md`:
  - Немає → режим **create**.
  - Є → запитай `update` (точкові правки) або `regenerate` (з нуля; стара версія у git-історії).

## Крок 1: Аналіз брифу і референсів
Прочитай `brief.md`, витягни нішу, тон, URL-референси.

**Каскад витягування з URL** (чесні можливості):

1. **Playwright MCP** (якщо підключений) — скриншот + аналіз dominant colors.
2. **WebFetch HTML**: `<meta name="theme-color">`, `<link href=".../fonts.googleapis.com/...">`, `<link rel="stylesheet" href="...">` (зберегти URL CSS).
3. **WebFetch CSS** за знайденими URL — парсинг CSS-змінних у `:root` (`--color-*`, `--font-*`). Працює лише якщо сайт їх справді має.
4. Якщо даних недостатньо / суперечливі → **НЕ вгадуй**. Запитай словами: «З X витяг primary=#.., шрифт Inter. Accent і spacing не визначив. Опиши словами: настрій, контраст, що подобається».

Запиши результат у `brief.md` → `## Референси`: що витягнуто, яким методом, що лишилось невизначеним.

## Крок 2: Генерація палітри і шрифтів

**Режим create**: якщо референсів нема або мало даних — запропонуй 2-3 варіанти у фіксованому форматі (інакше вибрати неможливо):

```
### Варіант A — «Мінімалізм з холодним синім»
Настрій: спокій, довіра, tech
Primary:   #3b82f6   (oklch 0.65 0.18 250)
Accent:    #f59e0b   (oklch 0.78 0.17 70)
Neutral-0: #ffffff
Neutral-950: #0a0a0a
Heading: Inter 600
Body:    Inter 400
Контраст text/bg: 15.3:1 ✅
```

**Генерація шкал** (не вручну):
- Палітра 50-950 — через Radix Colors або OKLCH lightness ramp від `L=0.98` до `L=0.15` крок `~0.08`, фіксований hue/chroma.
- Семантичні (success/warning/error/info) — фіксовані eta-значення, перевірені на контраст з обома фонами (світлим і темним).

**Brand archetypes** (якщо ніша тяжіє до одного — використай як основу):
- `tech`: Inter/Geist, холодні нейтралі, синій/фіолетовий primary
- `luxury`: serif (Playfair/Cormorant) + sans-serif, теплі нейтралі, золотисті/бордові
- `editorial`: serif display + humanist sans, високий контраст
- `playful`: geometric sans, насичені кольори, rounded
- `neutral-pro`: Inter/Manrope, сіра шкала, один акцент
- `technical`: mono + sans, монохром + 1 accent

**Режим update**: питай що саме міняємо, варіанти НЕ показуємо.

**Режим regenerate**: як create, попередня — в git.

**Обов'язкова перевірка перед показом**: розрахуй контраст для всіх text/bg пар (включно з темною темою). Якщо хоч одна <4.5:1 — ВИПРАВ автоматично або помітити червоним у варіанті.

## Крок 3: Заповнення `design-system.md`

Скопіюй `.claude/templates/design-system.md` у `./design-system.md` (create/regenerate) і заповни ВСІ секції:

- **Палітра** (light + dark): primary/accent/neutral (50-950) + semantic. Кожен токен: OKLCH + HEX.
- **Типографіка**: heading/body (через `next/font`), масштаб xs-7xl (line-height, letter-spacing).
- **Spacing**: 0, 0.5, 1, 1.5, 2, 3, 4, 6, 8, 10, 12, 16, 20, 24, 32 (rem-based).
- **Radius**: none/sm/md/lg/xl/2xl/full.
- **Shadows**: xs/sm/md/lg/xl.
- **Breakpoints**: sm=640, md=768, lg=1024, xl=1280, 2xl=1536 (або кастомні з обґрунтуванням).
- **Motion**: duration (75/150/300/500 ms), easing (linear, ease-in, ease-out, ease-in-out, spring), `prefers-reduced-motion` fallback.
- **Component patterns**: buttons (primary/secondary/ghost/destructive з `cva` variants), cards, sections (container widths + padding), forms (inputs, labels, errors), nav, focus rings (`ring-2 ring-offset-2`), мінімальний tap target 44×44.
- **Принципи**: 60-30-10, mobile-first, semantic tokens перед raw.

**Валідація згенерованого файлу**: перевір що всі перелічені секції присутні. Якщо щось пропущено — додай.

## Крок 4: Показати користувачу
Виведи `design-system.md`. Покажи результати contrast-check окремо (таблиця пар → ratio → pass/fail). Чекай підтвердження.

## Крок 5: Синхронізація коду

Якщо `package.json` містить `next`:

**Визнач тип стилізації**:
- `components.json` є → shadcn → використовуй його канонічні імена (`--primary`, `--primary-foreground`, `--background`, `--foreground`, `--muted`, `--card`, `--border`, `--ring`) і формат поточної версії shadcn (HSL tuple або OKLCH).
- Нема → наш власний стиль з `--color-*` у `globals.css`.

**Оновлення**:
- `tailwind.config.ts`: `theme.extend.colors/fontFamily/borderRadius/spacing/boxShadow/fontSize/screens/transitionDuration/transitionTimingFunction`.
- `src/app/globals.css`: якщо файл ще не створено — скопіюй з `.claude/references/globals.reference.css`. Якщо вже є — онови тільки OKLCH/HSL-значення CSS-змінних (`:root` і `.dark`), не переписуй файл цілком. Формат (OKLCH vs HSL) визнач за версією shadcn (див. `.claude/references/design-defaults.md`).
- `src/app/layout.tsx`: `next/font` imports + CSS variables.

**Unit-перевірка**: після оновлення запусти скрипт/перевірку, що всі токени з `design-system.md` присутні у `tailwind.config.ts` (простий парсинг обох файлів і порівняння ключів). Якщо розбіжність — зупинись, виправ, повтори.

Якщо проєкт НЕ ініціалізований — пропусти Крок 5, йди до Кроку 7.

## Крок 6: Застосування до існуючих компонентів

Якщо є `src/components/`:

1. **Grep-пошук** кольорів поза токенами:
   - Regex: `#[0-9a-fA-F]{3,8}\b`, `rgb\(`, `rgba\(`, `hsl\(`, `hsla\(`, `oklch\(` у `.tsx`/`.ts`/`.css` (виняток — `design-system.md` і `globals.css`).
   - Показати список файлів з рядками.
2. **Покажи користувачу** знайдене + запропонуй мапу заміни (old → new token).
3. Запитай: `так / ні / вибірково`.
4. За згодою онови ТІЛЬКИ `className` і стильові літерали — логіка (state, props, умови рендерингу) залишається недоторканою.
5. Після правок запусти:
   - `npx tsc --noEmit` → 0 помилок.
   - `npm run build` → успіх.
   - Якщо є Playwright MCP → зроби скриншоти до/після для візуального порівняння.
6. Якщо будь-яка перевірка червона — відкати зміни через `git checkout -- <file>` і попроси користувача переглянути вручну.

## Крок 7: Завершення

- `CHANGELOG.md`: дата, `design-system.md: created|updated|regenerated`, список синхронізованих файлів, результати тестів (contrast ✅, build ✅).
- **Коміт з точним списком файлів** (не `git add .`, не папками):
  ```
  git status              # покажи користувачу що зміниться
  git add design-system.md tailwind.config.ts src/app/globals.css src/app/layout.tsx <specific-component-files>
  git commit -m "feat(design): <короткий опис>"
  ```
- Якщо у `git status` є несуміжні зміни — запитай чи включати їх, не коміть автоматично.
- Повідом користувача: що зроблено, тести, next step (`npm run dev`).

## Тести які ЗАВЖДИ виконуються

| Коли | Що перевіряємо | Критерій |
|---|---|---|
| Крок 2 перед показом | WCAG-контраст для всіх text/bg пар (light+dark) | ≥4.5:1 body, ≥3:1 large |
| Крок 3 після заповнення | Валідність `design-system.md` — всі обов'язкові секції | 100% присутні |
| Крок 5 після синхронізації | Збіг токенів design-system.md ↔ tailwind.config.ts | 0 розбіжностей |
| Крок 6 після змін компонентів | `tsc --noEmit` + `next build` | 0 помилок, успіх |
| Крок 6 якщо є Playwright | Visual screenshots до/після | Готово для людського ревʼю |
