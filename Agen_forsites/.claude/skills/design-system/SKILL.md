---
name: design-system
description: Створює або оновлює дизайн-систему сайту (палітра, типографіка, spacing, radius, motion, component overrides) на основі юзерських референсів або brand archetype з брифу. Тригериться ЗАВЖДИ коли користувач каже «зроби дизайн», «підбери кольори», «заміни шрифт», «онови палітру», «зроби primary темнішим», «додай референс» (з URL / скріншотом / кодом / описом), «ось Hero якого хочу», «ось форма що подобається», «застосуй стиль з цього сайту», «переглянь design-refs», або коли треба синхронізувати design-system.md з tailwind.config.ts і globals.css. Сам керує трьома режимами — init (створення з нуля), intake (користувач підкинув референс), revise (точкова правка). НЕ вибирає структуру сайту, НЕ пише копірайт, НЕ деплоїть.
---

# design-system

Скіл керує дизайн-системою сайту. Три режими: **init** (створити з нуля), **intake** (прийняти юзерський референс), **revise** (точкова правка). Завжди модифікує код безпосередньо і показує diff — git дає rollback. Юзерські референси зберігаються у `./design-refs/` пер-сайт.

## Коли запускатись

Автоматично при:
- Створенні нового сайту (відсутній `design-system.md`).
- Коли юзер кидає референс: URL, скріншот, код, опис елемента.
- Коли просить правку токена/шрифту/радіуса/motion.
- Коли просить перевірити чи синхронізовані `design-system.md` ↔ `tailwind.config.ts` ↔ `globals.css`.

НЕ запускатись якщо:
- Питання про структуру сайту → це скіл `site-structure`.
- Питання про контент/копірайт → це `/new-site` Крок 4.
- Питання про деплой → `/deploy`.

## Що на вході

Обов'язкове:
- `brief.md` (ніша, аудиторія, тон; блок `## Design Direction` від site-structure скіла якщо є).

Опційне, але корисне:
- `./design-refs/` (юзерські референси).
- `design-system.md` (поточний стан якщо існує).
- `package.json`, `components.json` (для детекту shadcn-версії → формат кольорів OKLCH vs HSL — див. [references/design-defaults.md](../../../references/design-defaults.md)).
- `tailwind.config.ts`, `src/app/globals.css`, `src/app/layout.tsx` (для sync-check).

## Спільна передмова для всіх режимів

1. Прочитай `brief.md`. Знайди `## Design Direction` — одне речення з site-structure скіла. Воно задає «відчуття» сайту: два прикметники + ЦА + емоція. Використовуй як якір для всіх рішень.
2. Визначи формат кольорів:
   - Є `components.json` → shadcn → перевір версію за `package.json` (tailwind v4 = OKLCH tuple, v3 = HSL tuple).
   - Нема `components.json` → наш формат OKLCH у `--color-*`.
3. Фіксовані дефолти (spacing, breakpoints, shadows, font-size, motion, focus, iconography, CVA Button/Card/Section) — **не вигадуй**, бери з [references/design-defaults.md](../../../references/design-defaults.md).

---

## Режим 1: init

**Тригер**: `design-system.md` не існує і треба створити з нуля.

### Крок 1. Логотип як якір №1

З `brief.md` візьми блок **Логотип** (Крок 4 брифу). Єдине що має бути у брифі — файл/URL логотипу або `нема`.

**Якщо логотип є** (це **якір №1** для всіх дизайн-рішень):
1. Попроси юзера скинути файл/URL якщо ще не в `./design-refs/brand/`.
2. Проаналізуй за секцією «Логотип» у [references/reference-intake.md](references/reference-intake.md) — витягни:
   - Dominant кольори → кандидат у `primary` (найнасиченіший, не нейтральний).
   - Secondary кольори → кандидат у `accent`.
   - Тип wordmark/icon/combination → напрямок типографіки (serif wordmark → serif heading; sans wordmark → sans heading).
   - Геометрія (гостра / округлена / pill) → `radius style` (sharp / slightly-rounded / rounded).
3. Збережи аналіз у `./design-refs/brand/logo-analysis.md`.
4. Ці витягнуті значення — **базова лінія** палітри і типографіки. Усі подальші рішення (інші brand assets, референси, ручні правки) мусять з ними гармоніювати.

**Якщо логотипу нема** — рухайся далі без anchor'а.

### Крок 2. Запитай про інші brand assets (якщо є)

Дослівно:
> «Є ще готові brand assets які треба використати на сайті?
> - готові шрифти (назва / файл / Google Fonts?)
> - готова палітра / hex-коди (список чи скріншот зразків)
> - готовий макет від дизайнера (Figma URL / PDF / картинка)
>
> Якщо нема — я підберу сам на основі [логотипу / брифу]. Скажи що з цього є.»

Якщо юзер дав шрифти/палітру/макет:
- Шрифти → записуй як hard constraint для typography.
- Палітра → записуй як hard constraint для colors.
- Макет → проаналізуй як референс (intake-mode), витягни spacing/radius/layout патерни.

**Конфлікт**: якщо логотип і надані assets суперечать — логотип виграє за замовчуванням, але покажи конфлікт юзеру і запитай.

Якщо нічого додаткового нема — рухайся далі.

### Крок 3. Запитай про inspiration-референси

Дослівно:
> «Є референси дизайну — сайти, Hero, форми, шрифти що подобаються? Можеш кинути:
> - посилання на сайт(и)
> - скріншот елементу
> - готовий код компонента
> - або просто словами описати настрій / стиль.
>
> Якщо нічого немає — я підберу сам на основі [логотипу + брифу / тільки брифу]. Скажи як зручно.»

### Крок 4a. Якщо inspiration-референси є

Для КОЖНОГО референсу послідовно:
1. Перейди у **intake mode** (див. нижче) — витягни патерни.
2. Збережи у `./design-refs/<category>/<name>.md`.
3. Запам'ятай витягнуті токени (колір, шрифт, spacing, radius).

Коли всі референси оброблені:
1. Об'єднай патерни з **Кроку 1 (логотип/brand)** + референсів у єдину палітру і шрифтовий набір.
2. **Конфлікт-резолюція**: якщо референс каже одне, логотип — інше, питай юзера:
   > «Логотип дає primary ~#5b9bd5 (холодний синій). Референс — warmer orange. Що пріоритет: бренд чи референс?»
   За замовчуванням логотип виграє (він залишиться на сайті назавжди).
3. Згенеруй OKLCH-шкалу 50-950 для кожного кольору (L-ramp від 0.98 до 0.15, крок ≈0.08, hue/chroma зафіксовані).
4. Перевір WCAG-контраст усіх text/bg пар (≥4.5:1 body, ≥3:1 large). Якщо хоч одна <4.5 — виправ або помітити.
5. Згенеруй парну dark-палітру: `L_dark = 1 - L_light`, `C_dark = C_light * 0.85`, hue незмінний.

### Крок 4b. Якщо inspiration-референсів нема (fallback)

1. Обери archetype з 6-ти у [templates/design-system.md](../../../templates/design-system.md): tech / luxury / editorial / playful / neutral-pro / technical. Логіка вибору — ніша + тон + Design Direction з брифу.
2. **Якщо був логотип** — archetype має гармоніювати з ним (напр. логотип з serif wordmark → editorial або luxury, не playful). Палітра-варіанти базуй на hue логотипу.
3. **Якщо логотипу нема** — запропонуй 2-3 варіанти палітри під archetype + Design Direction.
4. Після вибору — згенеруй шкали, dark-пару, контраст-таблицю.

**Формат варіанту**:
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

### Крок 5. Заповни `design-system.md`

Скопіюй [templates/design-system.md](../../../templates/design-system.md) у `./design-system.md` і заповни ВСІ секції:
- Brand archetype
- Палітра Light (brand, semantic, surface)
- Палітра Dark
- Контраст-таблиця
- Типографіка (heading/body/mono)
- Radius style (rounded / slightly-rounded / sharp)
- Component overrides (тільки відхилення від дефолтів — якщо все стандартно, залиш порожнім)

Перевір: усі `{{ PLACEHOLDER }}` замінені.

### Крок 6. Покажи юзеру

Виведи зміст `design-system.md`. Покажи контраст-таблицю окремо. Чекай підтвердження.

Якщо юзер каже «зроби primary більше warm» / «шрифт вузький» / тощо — **перейди у revise mode** (не перегенеровуй все).

### Крок 7. Синхронізація коду

**Детектуй стан проєкту:**
- Немає `package.json` з `next` → проєкт ще не ініціалізовано. Напиши тільки `design-system.md`. Код синхронізуєш після ініціалізації (повторний виклик скіла після `create-next-app`).
- Є `package.json` з `next` → виконай синхронізацію за матрицею у [references/sync-targets.md](references/sync-targets.md):
  - `globals.css` — копіюй з [references/globals.reference.css](../../../references/globals.reference.css), заміни CSS-змінні на актуальні. Якщо файл вже існує — оновлюй ТІЛЬКИ значення `:root` і `.dark`, не переписуй цілий файл.
  - `tailwind.config.ts` — `theme.extend.colors/fontFamily/borderRadius`.
  - `src/app/layout.tsx` — `next/font` imports + CSS-variable injection.

Після синхронізації — запусти sync-check: токени у `design-system.md` повинні бути присутні у `tailwind.config.ts`. Якщо розбіжність — зупинись, виправ.

### Крок 8. Завершення

- Онови `CHANGELOG.md`: «design-system: created» + що саме.
- Запитай юзера чи робити коміт (НЕ комить автоматично). Якщо так — `git add <specific files>` + `git commit`.
- Запропонуй юзеру `npm run dev` для візуальної перевірки.

---

## Режим 2: intake

**Тригер**: юзер кидає референс — URL / картинка / код / опис.

### Крок 1. Визнач тип референсу і витягни патерни

Деталі для кожного типу — [references/reference-intake.md](references/reference-intake.md). Коротко:
- **URL** → WebFetch → `<meta theme-color>`, `<link>` на CSS/fonts, парсинг `:root`-змінних.
- **Скріншот** → якщо vision-режим недоступний, запитай юзера описати dominant color, шрифт, настрій.
- **Код (HTML/CSS/React)** → парсинг className, CSS-змінних, inline-стилів.
- **Текст** → уточнюючі питання для витягнення конкретики (колір, шрифт, spacing).

**Завжди витягай**: кольори (primary/accent якщо видно), типографіка, spacing/padding патерн, radius, тінь, motion, layout-композиція (якщо hero — grid vs center vs split).

### Крок 2. Збережи аналіз

Створи `./design-refs/<category>/<name>.md` де:
- `<category>` — `hero` / `forms` / `cards` / `nav` / `footer` / `typography` / `colors` / `misc`.
- `<name>` — коротке описове ім'я (напр. `stripe-blue-hero`, `linear-dark-form`).

Формат файлу:
```markdown
# <Title>

**Джерело**: <URL або «скріншот від юзера» або «код наданий юзером»>
**Категорія**: hero / form / ...
**Додано**: YYYY-MM-DD

## Витягнуті патерни

- **Колір**: ...
- **Шрифт**: ...
- **Spacing**: ...
- **Radius**: ...
- **Motion**: ...
- **Layout**: ...

## Примітки юзера

<що саме сподобалось; те що явно сказав юзер>

## Застосовано

- [ ] Глобально (design-system.md токени)
- [ ] Тільки компонент <name>
- [ ] Поки ні
```

Онови `./design-refs/README.md` — простий індекс усіх референсів (якщо нема — створи).

### Крок 3. Запитай scope

Дослівно:
> «Витягнув [короткий підсумок: колір / шрифт / що ще]. Застосувати:
> - **глобально** (оновити токени у `design-system.md` — вплине на всі компоненти)
> - **тільки [Hero / Form / компонент]** (cva-override у `src/components/ui/<name>.tsx`)
> - **поки не застосовувати** (збережу у `design-refs/` на майбутнє)»

### Крок 4. Застосуй

**Глобально** → перейди у revise mode з новими значеннями. Онови `design-system.md`, потім sync код за [references/sync-targets.md](references/sync-targets.md).

**Компонент** → онови `Component overrides` у `design-system.md`, додай cva-варіант у `src/components/ui/<name>.tsx` (якщо немає компонента — створи з шаблону CVA-default у [references/design-defaults.md](../../../references/design-defaults.md)).

**Поки ні** → нічого не чіпай, тільки онови чекбокс у файлі референсу.

У всіх випадках покажи diff.

### Крок 5. Валідація

- WCAG-контраст якщо зачіпав кольори.
- Sync-check якщо зачіпав токени.
- `tsc --noEmit` + `npm run build` якщо зачіпав компонент.

Якщо валідація червона — відкоти через `git checkout -- <files>` і поясни юзеру що не вийшло.

---

## Режим 3: revise

**Тригер**: юзер каже «зроби X темніший», «заміни Y на Z», «збільш radius», «прибери тіні».

### Крок 1. Розпарсь інтент

Якими токенами/файлами зачіпається зміна? Використай матрицю у [references/sync-targets.md](references/sync-targets.md).

Приклади мапінгу:
- «зроби primary темніший» → токен `primary` + `primary-foreground` → `design-system.md` + `globals.css` light+dark + `tailwind.config.ts`.
- «заміни шрифт на serif» → `Heading` + `Body` → `design-system.md` + `tailwind.config.ts` + `src/app/layout.tsx` (next/font).
- «збільш radius» → `--radius` → `design-system.md` + `globals.css` + `tailwind.config.ts`.

### Крок 2. Знайди поточні значення

Прочитай усі зачеплені файли, зафіксуй before-стан.

### Крок 3. Порахуй новий стан

Якщо юзер дав конкретне значення («primary=#6366f1») — використовуй.
Якщо відносне («темніший», «на крок менший») — застосуй:
- Темніше: `L -= 0.08`.
- Світліше: `L += 0.08`.
- Menshиy radius: з 0.5 → 0.375 (rem), або 0.75 → 0.5.
- Теплiший: зсув `hue` у бік 30-60 (warmth).
- Холодніший: зсув у бік 220-260.

### Крок 4. Застосуй синхронно

Онови ВСІ зачеплені файли в одному turn, не одному за одним. Важливо: якщо зміниш тільки `design-system.md`, синхронізація буде розбита.

### Крок 5. Валідація + diff

- WCAG-контраст (якщо колір).
- Sync-check (токени в `design-system.md` = в `tailwind.config.ts`).
- Покажи diff кожного файлу.

Якщо контраст зламався — **відкоти** і запропонуй юзеру:
> «Новий primary дає контраст 3.2:1 з foreground — нижче WCAG AA. Можу:
> - зсунути L глибше (темніше на ще 0.05)
> - змінити foreground на чисто-білий
> - залишити і проігнорувати WCAG (не раджу)»

---

## Що скіл НЕ робить

- Не вибирає структуру сайту (`site-structure` скіл).
- Не пише контент сторінок.
- Не деплоїть.
- Не створює нові CVA-компоненти поза дефолтами без явного запиту.
- Не чіпає `brief.md` окрім читання блоку `## Design Direction`.

## Reference файли

- [references/reference-intake.md](references/reference-intake.md) — як аналізувати різні типи референсів.
- [references/sync-targets.md](references/sync-targets.md) — матриця «тип зміни → які файли оновити».

Зовнішні (з кореня проєкту):
- [../../../templates/design-system.md](../../../templates/design-system.md) — шаблон output-файлу.
- [../../../references/design-defaults.md](../../../references/design-defaults.md) — фіксовані дефолти.
- [../../../references/globals.reference.css](../../../references/globals.reference.css) — шаблон `globals.css`.

Завантажуй reference-файли **лише коли потрібно у відповідному кроці**, не тримай усе в контексті.
