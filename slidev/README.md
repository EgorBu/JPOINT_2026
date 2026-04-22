# JPoint 2026 — Slidev-версия презентации

Этот каталог содержит презентацию на [Slidev](https://sli.dev/). Источник контента — `PRESENTATION_SLIDEV.md`. Оригинал Marp (`../PRESENTATION.md`) оставлен нетронутым.

## Быстрый старт

```bash
cd slidev
npm install
npm run dev           # http://localhost:3030
```

## Сборка и экспорт

```bash
npm run build         # статический сайт в ./dist
npm run export        # PDF: PRESENTATION_SLIDEV-export.pdf
npm run export:pptx   # PPTX: PRESENTATION_SLIDEV-export.pptx
npm run export:png    # Папка PNG-картинок по слайду
```

Для экспорта PDF/PPTX нужен `playwright-chromium` (ставится через `npm install`). Первый запуск скачает Chromium (~170 МБ).

## GitHub Pages

В `.github/workflows/slidev.yml` настроен workflow: при пуше в `main`/`master` с изменениями в `slidev/**` или `materials/**` Slidev собирается и публикуется через GitHub Pages.

Нужно один раз включить: **Settings → Pages → Source: GitHub Actions**.

URL будет вида `https://<user>.github.io/<repo>/`.

## Материалы (картинки)

`slidev/public/materials` — это симлинк на `../../materials`, поэтому в слайдах пути выглядят так: `/materials/main_image.png`. В CI материалы копируются в `dist/materials/` явно (симлинки в GitHub Pages не работают).

## Чем отличается от Marp-оригинала

| Аспект | Marp (`PRESENTATION.md`) | Slidev (`PRESENTATION_SLIDEV.md`) |
|---|---|---|
| Размер картинок | `![w:450](...)` | `<img class="w-[450px] mx-auto" />` |
| Картинка справа | `![bg right:40% w:500](...)` | `layout: image-right` + `image: ...` во frontmatter слайда |
| Центрированная большая картинка | `![bg center h:720](...)` | `layout: image` + `image: ...` |
| Стили per-slide | `<style scoped>` | Удалены — общий `style.css` для таблиц, остальное берут layouts seriph |
| Заметки докладчика | `<!-- notes -->` | `<!-- notes -->` (та же нотация) |

## Темы

Сейчас используется [`@slidev/theme-seriph`](https://github.com/slidevjs/themes/tree/main/packages/theme-seriph). Альтернативы: `default`, `apple-basic`, `bricks`, `geist`. Чтобы поменять — правим ключ `theme:` в frontmatter файла `PRESENTATION_SLIDEV.md` и перезапускаем `npm install`.
