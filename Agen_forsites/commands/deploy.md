---
description: Задеплоїти сайт на платформу, вказану в brief.md
argument-hint: (без аргументів)
---

# /deploy — деплой сайту

> **Коли викликати**: коли `/new-site` завершив цикл і сайт локально працює (`npm run dev` без помилок, `next build` зелений). Команда не запускається на напів-готовому сайті.

## Крок 0: Прочитай бриф і визнач ціль

- Прочитай `brief.md` → секція `## Deploy`.
- Якщо немає секції або значення `ще не знаю` / у `## TODO` — запитай користувача: `Vercel / Netlify / Cloudflare Pages / інше`. Запиши відповідь у `brief.md` (і прибери з TODO).
- Визнач поточний стан:
  - `git init` виконано?
  - Є `origin` remote?
  - `npm run build` успішно збирається?

## Крок 1: Pre-flight перевірки (обов'язково, не пропускай)

Запусти послідовно. Якщо хоч одна червона — СТОП, виведи помилку, не деплой.

1. `git status` → немає незакомічених змін. Якщо є — запитай: закомітити? які файли?
2. `npx tsc --noEmit` → 0 помилок.
3. `npm run lint` → 0 помилок.
4. `npm run build` → успіх.
5. Перевір що `brief.md` не містить незакритих `{{ TODO: ... }}` маркерів у критичних місцях (контакти, основний контент). Якщо є — попередь користувача і запитай чи деплоїти зараз чи спочатку заповнити.
6. Пройди `.claude/references/production-checklist.md`. Якщо є незакриті пункти, критичні для prod (legal, аналітика, форми-handler) — СТОП, покажи список, запитай що робити: зафіксити зараз, відкласти як `## TODO` у `brief.md`, чи деплой як є (лише за явною згодою).

## Крок 2: Деплой відповідно до цілі

### Якщо target = Vercel

1. Перевір чи встановлений Vercel CLI: `vercel --version`. Якщо ні — запропонуй `npm i -g vercel` або `npx vercel`.
2. Перший раз: `vercel link` (користувач обирає scope і проєкт у браузері).
3. Деплой: `vercel --prod` (або `vercel` для preview).
4. Витягни production URL з виводу, запиши у `brief.md` → `## Deploy` як `url:`.

### Якщо target = Netlify

1. `netlify --version` (встановлення через `npm i -g netlify-cli` якщо нема).
2. Перший раз: `netlify init` → обери existing або створи новий site.
3. `netlify deploy --prod` (якщо є `netlify.toml`) або `netlify deploy --prod --dir=.next` для Next.js з `@netlify/plugin-nextjs`.
4. Витягни URL, запиши у `brief.md`.

### Якщо target = Cloudflare Pages

1. `npx wrangler --version`.
2. Для Next.js App Router потрібен adapter: `@cloudflare/next-on-pages`. Додай у `package.json` якщо нема.
3. Збірка: `npx @cloudflare/next-on-pages`.
4. Деплой: `npx wrangler pages deploy .vercel/output/static`.
5. Витягни URL, запиши у `brief.md`.

### Якщо target = інше

- Запитай користувача деталі (VPS з Docker? Git push на власний хостинг?).
- Виконай вказане.
- Зафіксуй кроки, які використовувались, у `brief.md` → `## Deploy` → `steps:`.

## Крок 3: Git — зафіксувати стан деплою

- Якщо є не закомічені зміни, створені під час Кроку 2 (наприклад, `.vercel/`, `netlify.toml`, `wrangler.toml`) — закомітити:
  ```
  git add <конкретні файли>
  git commit -m "chore(deploy): configure {{ target }}"
  ```
- Якщо `origin` GitHub ще не налаштований і користувач хоче бекап на GitHub — запитай: створити `gh repo create <name> --private --source=. --push`?

## Крок 4: Пост-деплой перевірка

- Якщо отримали production URL:
  - `curl -I <URL>` → статус `200`.
  - Перевір OG-теги: `curl -s <URL> | grep "og:"` — `og:title`, `og:image`, `og:description` присутні.
  - (Опційно, якщо є Playwright MCP) — зайди на головну, зроби скриншот, додай у повідомлення.
- Якщо щось некоректне — попередь користувача конкретно про що.

## Крок 5: Завершення

- Онови `CHANGELOG.md`:
  ```
  YYYY-MM-DD — Deploy ({{ target }})
  URL: {{ production-url }}
  Commit: {{ short-sha }}
  ```
- Повідом користувача:
  - URL задеплоєного сайту.
  - Що саме перевірено (pre-flight + post-deploy).
  - Що робити далі (налаштувати кастомний домен, аналітику, форми — якщо ще є в `brief.md` → `## TODO`).

## Правила безпеки

- НЕ зберігай токени/ключі в git-треке файлах. Якщо платформа потребує — у локальні `.env.local` (gitignored).
- НЕ перезаписуй існуючі production-URL без явного підтвердження користувача.
- НЕ роби `vercel --prod` / `netlify deploy --prod` якщо хоч один pre-flight тест червоний.
