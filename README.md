# AI-site-builder

Шаблон агента Claude Code для потокового створення Next.js сайтів.

**Модель**: один агент = один сайт. Цей репозиторій — майстер-шаблон, який копіюється в директорію кожного нового сайту.

## Структура

```
.claude/
├── CLAUDE.md                       # головна інструкція агента
├── commands/                       # slash-команди
│   ├── new-site.md                 # /new-site — створити сайт з нуля
│   ├── design.md                   # /design — дизайн-система
│   └── deploy.md                   # /deploy — деплой
├── templates/
│   └── design-system.md            # шаблон дизайн-документа
└── references/
    ├── design-defaults.md          # фіксовані дефолти (spacing, motion, CVA)
    └── globals.reference.css       # базовий CSS для копіювання
```

## Як використовувати

1. Створи директорію для нового сайту: `mkdir c:\my-site`
2. Скопіюй шаблон агента: `cp -r c:\projects\.claude c:\my-site\.claude`
3. Зайди в директорію і запусти Claude Code: `cd c:\my-site && claude`
4. Виконай у чаті: `/new-site`

Агент проведе повний цикл: бриф → дизайн-система → Next.js код → favicon/OG → перевірки.

## Стек

Next.js 15 (App Router), TypeScript, Tailwind CSS, shadcn/ui.
