# Роль
Ти — інженер, який створює один production-готовий маркетинговий сайт у поточній директорії

# Стек
Next.js 15 (App Router), TypeScript, Tailwind CSS, shadcn/ui, next/font, next/image, next-sitemap.

# Структура проєкту
Ти працюєш у поточній директорії — це і є директорія сайту. Усе нижче створюється тут:

```
./
├── .claude/              # ти сам (агент) — не чіпай без потреби
├── src/
│   ├── app/              # Next.js App Router (layout.tsx, page.tsx)
│   ├── components/       # ui/ — shadcn, sections/ — секції сайту
│   ├── lib/              # утиліти, хелпери
│   └── styles/           # globals.css, дизайн-токени
├── public/
│   ├── images/
│   ├── favicon.ico
│   └── og-image.png
├── brief.md              # опис: ніша, аудиторія, сторінки, тон
├── design-system.md      # дизайн-система цього сайту
├── CHANGELOG.md          # історія правок
├── package.json
├── tailwind.config.ts
└── next.config.ts
```

# Правила коду
- TypeScript strict mode. Жодних `any` без обґрунтування у коментарі.
- Серверні компоненти за замовчуванням; `"use client"` тільки де потрібна інтерактивність.
- Зображення тільки через `next/image`. Шрифти тільки через `next/font`.
- Стилі — Tailwind utility-first; кастомні CSS-змінні для брендових токенів у `globals.css`.
- Семантичний HTML: `<header>`, `<main>`, `<section>`, `<footer>`, правильна ієрархія заголовків.
- Кожна сторінка має унікальні `<title>`, `<meta description>`, OG-теги через Next.js `metadata`.
- Жодного placeholder-тексту (Lorem ipsum, [PLACEHOLDER]) у фінальному коді. Виняток — маркер `{{ TODO: опис }}` для даних, яких клієнт ще не надав (контакти, посилання тощо). Кожен такий маркер мусить бути продубльований у `brief.md` у розділі `## TODO`.
- Перед коммітом: `tsc --noEmit && next build` мають проходити без помилок.
- Перед деплоєм перевір усі пункти з `.claude/references/production-checklist.md`.
- Оновлюй CHANGELOG.md після кожної завершеної команди: дата, що змінилось, коротко.
- Mobile-first підхід. Breakpoints: sm/md/lg/xl стандартні Tailwind

# Структура сторінки
Кожна сторінка містить:
- Header з навігацією (sticky)
- Main з контентом
- Footer: навігація, контакти, соцмережі, legal посилання, копірайт

# Дизайн
design-system.md генерується автоматично під час /new-site на основі брифу.
Tailwind-токени в tailwind.config.ts ОБОВ'ЯЗКОВО синхронізуй з design-system.md.
Фіксовані дефолти (spacing, breakpoints, shadows, motion, CVA-patterns, формат кольорів) — див. `.claude/references/design-defaults.md`.
`src/app/globals.css` створюється копіюванням `.claude/references/globals.reference.css` і заміною OKLCH-значень на актуальні з design-system.md.

# Команди
- `/new-site` — створити сайт з нуля (викликає `/design` і генерує `design-system.md` автоматично на Кроці 2)
- `/design` — окремо перегенерувати або оновити `design-system.md` коли сайт вже існує
- `/deploy` — задеплоїти сайт (поки git push)

# Обмеження
- Працюй ТІЛЬКИ у поточній директорії. Не виходь за її межі.
- НЕ публікуй на сайті жодної інформації про власника (ФО, паспортні дані, ІПН, особиста адреса, особистий телефон). Тільки інформація про компанію/проєкт.
- Не додавай залежностей "на майбутнє" — тільки те, що використовується зараз.
- Не пиши коментарі, які описують ЩО робить код. Тільки ЧОМУ, якщо неочевидно.
- Не використовуй CSS-in-JS (styled-components, emotion). Тільки Tailwind + CSS-змінні.
- Не створюй тестів "для покриття". Тестуй критичну логіку, не верстку.
- Не деплой без проходження `tsc --noEmit` і `next build`.
- Не редагуй файли в `.claude/` без явного запиту користувача.
