# Production checklist

Застосовується під час `/new-site` (Крок 4) і обов'язково у `/deploy` pre-flight.

**Три рівні важливості:**
- 🔴 **Blocker** — без цього в prod не йдемо. Агент має СТОП і виправити.
- 🟡 **Required** — для повноцінного prod. Можна відкласти у перший деплой, потім закрити hotfix-ом. Фіксується як `## TODO` у `brief.md`.
- 🟢 **Nice-to-have** — опційно, залежно від ніші / бюджету часу.

**Що вже покрито деінде** (не дублюємо):
- Контраст, focus rings, tap targets 44×44 → `.claude/references/design-defaults.md`.
- Дизайн-токени, shadcn-формат → там само.

---

## 0. Tech-gate 🔴 (блокує все інше)

Запускається ПЕРШИМ. Якщо червоне — СТОП, виправ, повтори.

- [ ] `npx tsc --noEmit` → 0 помилок
- [ ] `npm run lint` → 0 помилок
- [ ] `npm run build` → успіх
- [ ] `npm audit --audit-level=high` → 0 high/critical
- [ ] Git working tree clean, відсутні нерозв'язані merge-конфлікти
- [ ] `.env.local` та інші secrets у `.gitignore`, перевірено через `git ls-files | grep -E '\.env|secret'`

## 1. Метадані і SEO

### Blocker 🔴
- [ ] Унікальні `<title>` ≤60 символів на кожній сторінці через Next.js `metadata`
- [ ] Унікальний `meta description` ≤160 символів на кожній сторінці
- [ ] `metadata.title.template` у `layout.tsx` (`"%s | {{ BRAND }}"`)
- [ ] `metadata.alternates.canonical` на кожній сторінці
- [ ] `lang` на `<html>` відповідає мові контенту
- [ ] `robots.txt` у `public/`: prod → `Allow: /`, preview → `Disallow: /`. Закрити `/api/*`, `/login`, `/register`, `/admin/*` і будь-які приватні routes. Розрізнення env через `VERCEL_ENV` / `NODE_ENV`.
- [ ] `sitemap.xml` згенеровано через `next-sitemap`
- [ ] Sitemap відправлений у Google Search Console (після першого prod-деплою)

### Required 🟡
- [ ] Schema.org JSON-LD: Organization на всіх сторінках; Product/Service/FAQ/BreadcrumbList/Review/LocalBusiness де релевантно ніші
- [ ] Google Mobile-Friendly Test пройдений (https://search.google.com/test/mobile-friendly)
- [ ] Усі важливі сторінки проіндексовані (перевірити у Search Console → Coverage після 7-14 днів)

### Nice-to-have 🟢
- [ ] `hreflang` (якщо сайт мультимовний)

> **НЕ використовувати** `metadata.keywords` — Google ігнорує з 2009.

## 2. Open Graph

### Blocker 🔴
- [ ] `openGraph` у `metadata`: `title`, `description`, `url`, `type`, `locale` на кожній сторінці
- [ ] `og-image` 1200×630 через `src/app/opengraph-image.tsx`
- [ ] `twitter` у `metadata`: `card: "summary_large_image"`, title/description/image

## 3. Іконки і theme

### Blocker 🔴
- [ ] `src/app/icon.tsx` або `public/favicon.ico` (брендований)
- [ ] `src/app/apple-icon.tsx` (180×180)
- [ ] `export const viewport = { themeColor: ... }` у `layout.tsx` (НЕ через `<meta>`)

### Nice-to-have 🟢
- [ ] `public/manifest.json` (лише якщо реально PWA — для маркетингових сайтів зазвичай не треба)

## 4. Продуктивність

### Blocker 🔴
- [ ] Усі зображення через `next/image` з `alt`, `width`, `height`
- [ ] LCP-зображення має `priority`
- [ ] Шрифти через `next/font`
- [ ] Lighthouse на prod (mobile throttled): Performance ≥85, SEO ≥95
- [ ] Core Web Vitals: LCP <2.5s, CLS <0.1, INP <200ms

### Required 🟡
- [ ] `@next/bundle-analyzer` запущено, немає підозрілих чанків >300KB
- [ ] `images.remotePatterns` налаштовано — ЯКЩО реально вантажаться external зображення

### Automation
```bash
npx lighthouse-ci autorun --collect.url=http://localhost:3000
ANALYZE=true npm run build   # якщо налаштовано bundle-analyzer
```

## 5. Доступність (a11y)

### Blocker 🔴
- [ ] Semantic HTML: `<header>/<main>/<nav>/<footer>/<section>/<article>`
- [ ] Один `<h1>` на сторінку, ієрархія без стрибків
- [ ] Усі `<img>` мають осмислений `alt` (декоративні — `alt=""`)
- [ ] Alt ≠ ім'я файлу (`alt="hero.jpg"` — заборонено)
- [ ] Форми: кожен input має `<label>`, помилки через `aria-invalid` + `aria-describedby`
- [ ] `axe-core` через Playwright → 0 critical/serious
- [ ] Lighthouse Accessibility ≥95

### Required 🟡
- [ ] Skip-to-content посилання для клавіатурної навігації
- [ ] Множинні `<nav>` мають `aria-label`

### Automation
```bash
npx playwright test tests/a11y.spec.ts   # з axe-core
```

## 6. Мобільна і респонсив

### Blocker 🔴
- [ ] Перевірено на 375px (iPhone SE 3), 768px, 1280px
- [ ] Немає горизонтального скролу
- [ ] Header-навігація адаптована (hamburger або stack)

### Required 🟡
- [ ] Перевірено на 320px (найстаріші пристрої)

## 7. Legal / GDPR

**Важливо**: якщо сайт доступний у EU — privacy-policy практично завжди потрібен, бо IP-адреси є ПД за GDPR. Визначайся за брифом (цільова аудиторія).

> **Правило**: на сайті ніколи не публікується інформація про **власника** (ФО, паспорт, ІПН, особиста адреса, особистий телефон). Виключно інформація про **компанію/проєкт**.

### Required 🟡 (EU-трафік → Blocker 🔴)

**Legal-сторінки** (усі чотири обов'язкові, кожна — окрема route, усі лінкуються у footer):
- [ ] **Privacy Policy** (`/privacy`) — які ПД збираємо, мета, правові підстави, терміни зберігання, права суб'єкта
- [ ] **Terms of Service** (`/terms`) — умови використання, обмеження відповідальності, юрисдикція
- [ ] **Cookie Policy** (`/cookies`) — типи cookies, призначення кожного, як відмовитись
- [ ] **GDPR Policy** (`/gdpr`) — права суб'єкта (access/rectification/erasure/portability/objection), процедура звернень, контакт DPO або відповідальної особи

**Cookie-банер** (якщо використовуються cookies-based трекери; Plausible/Umami не потребують):
- [ ] Дві кнопки: **«Прийняти всі»** та **«Прийняти лише необхідні»**
- [ ] Третя опція-посилання «Налаштувати» для гранулярного вибору категорій (necessary / analytics / marketing) — Required 🟡
- [ ] Скрипти аналітики/маркетингу НЕ вантажаться до отримання згоди (Google Consent Mode v2 для GA4/Ads)
- [ ] Вибір користувача зберігається (localStorage/cookie `consent=...`) і переоцінюється не рідше ніж раз на рік
- [ ] Посилання на Cookie Policy з банера

**Інше:**
- [ ] **У footer** — контакти компанії/проєкту: робочий email, телефон, адреса + посилання на всі чотири legal-сторінки
- [ ] **На сторінці Contact** продубльовано: email, телефон, адреса, (опційно) карта або форма зворотного зв'язку
- [ ] Email реальний і робочий (не `hello@example.com`, тестовий лист доходить)
- [ ] Телефон реальний компанії (якщо тимчасово приховується — дозволено маска `+380 XX XXX **-**`)
- [ ] Консент-чекбокс у формах перед збором ПД з посиланням на `/privacy`

## 8. Аналітика

### Required 🟡
- [ ] Підключено один з: Plausible / Umami / Vercel Analytics / GA4
- [ ] Скрипти через `next/script` з `strategy="afterInteractive"`
- [ ] Трекінг submit-форм і кліків на основні CTA
- [ ] **Якщо GA4 або Meta Pixel** — consent перед ініціалізацією (Google Consent Mode v2)
- [ ] **Якщо Plausible/Umami** — consent НЕ потрібен

## 9. Форми і ліди

### Blocker 🔴 (коли на сайті є форма)
- [ ] Backend-handler підключений (Resend / Formspree / webhook на email клієнта)
- [ ] Валідація на клієнті І на сервері (zod або аналог)
- [ ] Honeypot-поле або CAPTCHA (Cloudflare Turnstile / hCaptcha) проти спаму
- [ ] Success/error стан після submit
- [ ] Помилка handler-а не показує stack trace користувачу

### Required 🟡
- [ ] Rate limiting (Upstash/Vercel KV або аналог)
- [ ] Процедура видалення лідів за запитом (GDPR Right to Erasure)

## 10. Контент

### Blocker 🔴
- [ ] 0 `Lorem ipsum`, `[PLACEHOLDER]` на сторінках
- [ ] Незакриті `{{ TODO: ... }}` — дозволені ТІЛЬКИ якщо продубльовані у `brief.md` → `## TODO` (див. CLAUDE.md)
- [ ] `linkinator` → 0 broken links
- [ ] Контактні дані реальні і робочі (email отримує тестовий лист)
- [ ] Посилання в footer працюють (соцмережі, legal)

### Required 🟡
- [ ] Орфографія перевірена (мова з брифу)

### Automation
```bash
npx linkinator http://localhost:3000 --recurse
```

## 11. 404 і error states

### Blocker 🔴
- [ ] `src/app/not-found.tsx` — брендований, з кнопкою на головну
- [ ] `src/app/error.tsx` — graceful fallback, не показує stack
- [ ] Перевірено: `/nonexistent` → 404 рендериться

### Required 🟡
- [ ] `src/app/loading.tsx` для плавних переходів між route-ами

## 12. Security headers

### Blocker 🔴
- [ ] `next.config.ts` → `headers()` задає:
  - `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`
  - `X-Frame-Options: DENY`
  - `X-Content-Type-Options: nosniff`
  - `Referrer-Policy: strict-origin-when-cross-origin`
  - `Permissions-Policy: camera=(), microphone=(), geolocation=()`
- [ ] HTTPS-only (автоматично на Vercel/Netlify/Cloudflare)

### Required 🟡
- [ ] Content-Security-Policy (з nonce-ами через middleware) — окремо кропітко, але важливо

## 13. Backup / Rollback

### Required 🟡
- [ ] Попередній успішний prod-деплой видно у dashboard хостингу (Vercel/Netlify redeploy previous button)
- [ ] Git має clean tag/commit, на який можна відкотити (`git tag prod-YYYY-MM-DD`)
- [ ] Форми-handler не залежить від нового коміту (або зворотньо сумісний)

---

## Що автоматизувати в CI (коли з'явиться)

Рекомендовано винести в GitHub Actions / Vercel Checks:
- `tsc --noEmit`, `next lint`, `next build` — на кожний push
- `lighthouse-ci autorun` — на кожний prod-деплой
- `playwright test` (smoke + a11y) — на кожний push
- `linkinator` — weekly на prod
- `npm audit` — daily

Чеклист залишається як reference, а реальна перевірка — в CI.

## Офіційні джерела (для верифікації)

- Next.js Deploy Checklist — https://nextjs.org/docs/app/building-your-application/deploying/production-checklist
- Web.dev Lighthouse — https://web.dev/lighthouse-performance/
- WCAG 2.2 — https://www.w3.org/WAI/WCAG22/quickref/
- OWASP Security Headers — https://owasp.org/www-project-secure-headers/
