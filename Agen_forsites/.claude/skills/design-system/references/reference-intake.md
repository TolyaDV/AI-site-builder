# Reference intake — як аналізувати різні типи референсів

Скіл отримує референси у 4-х формах. Для кожної — свій спосіб витягнути дизайн-патерни.

**Завжди витягай по максимуму:** кольори (primary, accent, neutrals), типографіку (heading/body font, weight, scale), spacing pattern, radius, shadows, motion (якщо видно), layout-композицію.

Якщо частина інформації відсутня — НЕ вигадуй. Запиши «не визначено» і запитай юзера словами.

---

## 1. URL

### Каскад витягування

**Крок 1. WebFetch HTML** — шукай у `<head>`:
- `<meta name="theme-color" content="#...">`
- `<link rel="stylesheet" href="...">` (зберегти URL CSS)
- `<link href=".../fonts.googleapis.com/...">` (Google Fonts)
- Inline `<style>` блоки з `:root { --color-*: ... }`

**Крок 2. WebFetch CSS** за знайденими URL — парсинг CSS-змінних у `:root`:
- `--color-primary`, `--primary`, `--brand-*`, `--bg-*`, `--text-*`
- `--font-*`, `--font-family-*`
- `--radius`, `--border-radius-*`

Якщо CSS використовує Tailwind/shadcn формат — шукай `--primary: ... ... ...` (HSL tuple або OKLCH tuple).

**Крок 3. Якщо сайт не віддає даних (SPA, server-rendered з inline Tailwind)** — використай Playwright MCP якщо підключений:
- Screenshot всієї сторінки
- Eval JS: `getComputedStyle(document.body).backgroundColor`, `fontFamily`
- Dominant color через canvas-sampling

**Крок 4. Fallback** — запитай юзера словами:
> «З `[URL]` витягнув:
> - Primary: ~#3b82f6
> - Шрифт: Inter
> - Spacing/radius не визначив.
>
> Опиши своїми словами: настрій, контраст, що саме подобається в цьому сайті?»

### Що записати у `./design-refs/<category>/<name>.md`

```markdown
# <Title>

**Джерело**: <URL>
**Метод витягування**: WebFetch + CSS parsing (або Playwright, або словами)
**Додано**: YYYY-MM-DD

## Витягнуті патерни
- Primary: `#...` (oklch ... ... ...)
- Accent: ...
- Heading font: ...
- Body font: ...
- Radius: `0.5rem`
- Spacing: scale 1.0× Tailwind / відхилення: ...
- Layout hero: grid 2-col / center / split

## Що не визначено
- ...

## Примітки юзера
<чому сподобалось>
```

---

## 2. Скріншот / картинка

### Підхід

Claude Code не завжди має доступ до vision-режиму. Тому:

**Крок 1. Спробуй прочитати картинку через Read** — інструмент підтримує PNG/JPG. Якщо Claude це все ж побачить — аналізуй напряму:
- Dominant colors (3-5 штук)
- Шрифт heading vs body (serif/sans/mono, вузький/широкий)
- Spacing щільний vs повітряний
- Radius: sharp / rounded / pill
- Shadows: плоско / soft / dramatic
- Настрій однією фразою

**Крок 2. Якщо не вдалось прочитати або деталі нечіткі** — запитай юзера:
> «Не можу точно проаналізувати картинку. Допоможи словами:
> - Який там головний колір (приблизно)?
> - Шрифт: serif чи sans-serif? Тонкий чи товстий?
> - Кути елементів: гострі / округлені / повністю pill?
> - Загальний настрій одним-двома словами?»

**Ніколи не вигадуй** на основі «типового» дизайну. Якщо пропустив — краще залиш «не визначено» ніж наповни сміттям.

---

## 3. Код (HTML / CSS / React / Tailwind className)

### Парсинг

**CSS-змінні:**
- `:root { --color-*: ... }` → мапуй на свої токени.
- `@theme` блок Tailwind v4 — шукай `--color-primary`, `--font-sans`, `--radius` тощо.

**Inline CSS / styled components:**
- `background: #...`, `color: #...`, `font-family: ...`
- Конкретні hex/rgb значення → конвертуй у OKLCH для запису.

**Tailwind className:**
- `bg-blue-500 text-white rounded-lg p-6 shadow-md` → мапай у токени:
  - `bg-blue-500` → primary (hue 220-240, L ~0.55, C ~0.18)
  - `rounded-lg` → radius 0.5rem
  - `p-6` → spacing-6 (стандарт)
  - `shadow-md` → shadow-md (стандарт)
- Якщо є `bg-[#3b82f6]` (arbitrary) — витягни точне значення.

**React-компоненти:**
- Якщо є cva-variants — прочитай `variants` і `defaultVariants`. Мапай на свої CVA patterns у [../../../references/design-defaults.md](../../../references/design-defaults.md).

### Запис

У `./design-refs/` файл містить:
- Оригінальний код (зібрано у ```code block```).
- Витягнуті патерни.
- Мапінг «їхній токен → наш токен» (де конфлікт або невизначене — поміть).

---

## 4. Текстовий опис

### Стратегія: уточнюючі питання

Юзер каже «хочу щось спокійне і тепле». Цього недостатньо.

Задай конкретні питання (не всі одразу — 2-3 найрелевантніші):

- **Настрій**: спокійне / яскраве / строге / грайливе? Темне чи світле?
- **Колір**: теплий (червоний/оранж/жовтий) чи холодний (синій/фіолет/зелений)? Насичений чи приглушений?
- **Шрифт**: serif (класика) / sans-serif (сучасне) / mono (технологічне)? Тонкий елегантний чи товстий сміливий?
- **Кути**: гострі (sharp) / ледь округлені / сильно round / pill?
- **Щільність**: повітряне (багато простору) / щільне (мінімум проміжків)?
- **Рух**: статично / легкі переходи / виражена анімація?
- **Приклад у голові**: «як у X» — якщо є, попроси URL.

Записуй відповіді у `./design-refs/text-description-<date>.md`.

---

## 5. Логотип (спеціальний кейс — якір №1)

Логотип — не просто референс. Він залишиться на сайті назавжди, тож його стиль **диктує** дизайн-систему більше ніж будь-який inspiration-референс. Обробляється окремо перед усіма іншими.

### Що витягти з логотипу

1. **Тип логотипу** (визнач перше):
   - **Wordmark** — тільки текст (Google, Airbnb, Figma). Тип шрифту = сильна підказка напряму типографіки сайту.
   - **Icon / symbol** — тільки графіка (Apple, Target). Форма іконки → radius-style; колір → primary anchor.
   - **Combination** — текст + графіка (Adidas, Slack). Аналізуй обидва.
   - **Monogram / lettermark** — стилізована літера/абревіатура (HBO, IBM).

2. **Кольори**:
   - Dominant (найбільша площа) → кандидат у `primary`.
   - Secondary (присутні у менших елементах або градієнтах) → кандидат у `accent`.
   - Neutral (тло логотипу на самих-файлах — часто white/black) → напрямок нейтралей.
   - Якщо логотип монохромний — `primary` обирається з брифу / inspiration-референсів.

3. **Типографіка** (якщо wordmark або combination):
   - **Serif** (Playfair-like, Bodoni-like) → heading сайту теж serif (editorial / luxury archetype).
   - **Sans-serif geometric** (Futura-like, Avenir-like) → heading geometric sans (tech / playful).
   - **Sans-serif humanist** (Gill-like, Frutiger-like) → heading humanist (neutral-pro).
   - **Script / handwritten** → рідко копіюємо у heading, але задаємо м'якший tone.
   - **Display / custom** → неможливо повторити точно; обирай найближчий за настроєм.

4. **Геометрія**:
   - Гострі кути у літерах / іконці → `radius: sharp` (0 rem) або `slightly-rounded` (0.25 rem).
   - Округлені кути → `rounded` (0.5 rem).
   - Pill / фулл-раунд → `fully-rounded` (0.75-1 rem).

5. **Настрій** (одним словом-двома):
   - bold / тиха впевненість / грайливий / технічний / класичний / ексцентричний.
   - Це задає «tone of voice» візуалу: наскільки сміливі акценти, наскільки великі шрифти, наскільки контрастні кольори.

### Як працювати з файлом логотипу

- **SVG** — прочитай напряму (Read) → парсинг `fill`, `stroke`, кольорів.
- **PNG/JPG** — якщо vision доступний, аналізуй; якщо ні — запитай юзера описати.
- **URL** — WebFetch на сторінку бренду, витягни `<link rel="icon">` або скріншот через Playwright якщо є.

### Формат запису `./design-refs/brand/logo-analysis.md`

```markdown
# Logo analysis — <brand name>

**Файл**: <path or URL>
**Тип**: wordmark / icon / combination / monogram
**Додано**: YYYY-MM-DD

## Кольори
- Dominant: `#...` (oklch ... ... ...) → primary anchor
- Secondary: `#...` → accent anchor
- Нейтральний фон: `#...`

## Типографіка (якщо є текст у логотипі)
- Класифікація: serif / sans-geometric / sans-humanist / script / display
- Закрите до: Inter / Playfair / Geist / ... (найближчий системний/Google-шрифт)
- Напрям heading сайту: serif / sans / mono

## Геометрія
- Radius style: sharp / slightly-rounded / rounded / fully-rounded

## Настрій
<1-2 слова>

## Якорі для дизайн-системи
- `primary`: <hue/chroma hint>
- `accent`: <hue/chroma hint>
- `heading font`: <напрям>
- `radius`: <rem>
```

### Якщо логотип суперечить брифу

Наприклад, бриф каже «tech, мінімалістичний» але логотип — serif display з теплими кольорами (luxury vibes). Це сигнал що бренд вже існує і має свою ДНК. **Пріоритет логотипу**, але покажи юзеру конфлікт:
> «Логотип дає luxury-вайб (serif, теплий бордо), а бриф каже tech. Я поведу сайт у luxury-напрямку щоб не розривати бренд. Якщо хочеш tech — треба перепроектувати логотип або свідомо йти на дисонанс. Що обираємо?»

---

## Anti-patterns

- Писати патерн «tech» бо юзер сказав «сучасний» — «сучасний» нічого не означає, уточнюй.
- Витягувати з одного референсу 20 токенів і намагатись застосувати все — обирай 3-5 найсуттєвіших (зазвичай: primary, accent, heading font, body font, radius).
- Пропускати крок «запитай юзера словами» якщо автоматичні методи не дали даних — не вигадуй.
- Записувати witcheverything у `./design-refs/` у форматі вільного тексту — структурований markdown за шаблоном вище, інакше файл нечитабельний для майбутнього re-use.
