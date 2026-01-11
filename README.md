# Задание 2.2 — Кастомизация статического сайта (MkDocs + Jinja2 + GitHub Actions + GitHub Pages)

## Ссылки
- GitHub Pages: https://ran3boy.github.io/site-python_2-2/
- Репозиторий: https://github.com/ran3boy/site-python_2-2

## Цель работы
1) Создать собственную тему (шаблон) сайта на HTML/CSS/JS.  
2) Реализовать шаблонизацию (Jinja2) и интеграцию Markdown-контента в HTML-шаблон с помощью MkDocs.  
3) Настроить pipeline GitHub Actions: проверка/валидация HTML, минификация HTML, сборка CSS через PostCSS, сборка MkDocs и деплой на GitHub Pages.  

---

## Структура проекта
- `src/` — исходные HTML/CSS (frontend)
- `dist/` — результат сборки frontend (минифицированный HTML и CSS после PostCSS)
- `docs/` — контент сайта в Markdown (MkDocs)
- `theme/` — кастомная тема MkDocs (Jinja2 шаблон + стили)
- `mkdocs.yml` — конфигурация MkDocs
- `.github/workflows/deploy.yml` — workflow деплоя на GitHub Pages

---

## Кастомная тема (Jinja2)
Файл: `theme/main.html`

Реализовано:
- кастомный `header` и `footer`
- навигация по страницам
- подключение CSS темы
- метаданные сайта через `<meta>`:
  - `description` берётся из `site_description`
  - `author` берётся из `site_author`

---

## Стилизация секции (главная страница)
Файл: `theme/assets/styles/index.css`

Реализовано:
- стилизованный header (тёмная панель навигации)
- карточный блок контента на главной (фон, рамка, скругления, тень)
- стилизованный footer

---

## Сборка статики: HTML/CSS
Сборка frontend выполняется через npm-скрипты:
- HTML минифицируется (html-minifier)
- CSS собирается через PostCSS (postcss-import + csso)

Дополнительно подключена HTML-валидация:
- `html-validate "src/**/*.html"`

---

## GitHub Actions и деплой на GitHub Pages

### Workflow файл
`.github/workflows/deploy.yml`

### Этапы сборки в pipeline
1. Checkout репозитория
2. Установка Node.js
3. Установка зависимостей npm
4. Сборка frontend:
   - HTML validate
   - HTML minify
   - PostCSS build
5. Копирование собранного CSS в тему MkDocs: `theme/assets/styles/index.css`
6. Установка Python и MkDocs
7. Сборка сайта MkDocs в папку `site/`
8. Публикация артефакта и деплой на GitHub Pages

### Содержимое deploy.yml
```yml
name: Deploy to GitHub Pages

on:
  push:
    branches: ["main"]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install Node dependencies
        run: npm install

      - name: Build frontend (HTML minify + PostCSS + validate)
        run: npm run build

      - name: Copy built CSS into MkDocs theme
        run: |
          mkdir -p theme/assets/styles
          cp dist/styles/index.css theme/assets/styles/index.css

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install MkDocs
        run: pip install mkdocs

      - name: Build MkDocs site
        run: mkdocs build --site-dir site

      - name: Upload GitHub Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: site

  deploy:
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
