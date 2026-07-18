# Dashboard Cat CRM

Статичний UI-макет адмін-панелі (dashboard) у стилі CRM-системи. Проєкт демонструє верстку інтерфейсу для управління командою та задачами: бічне меню навігації, картки/таблицю співробітників, список задач і блок звітів із прогрес-барами.

## Основний функціонал

- **Бічне меню (Sidebar)** — навігація з іконками, адаптивне відкриття/закриття на мобільних пристроях (з блокуванням скролу сторінки).
- **Header** — пошук, кнопка сповіщень, лічильник непрочитаних сповіщень.
- **Welcome** — привітальний блок на головному екрані.
- **Co-workers** — список/таблиця співробітників з аватарами, посадами, статусами (бейджі danger/warning/success) та деталями по кожному.
- **Tasks (Board)** — дошка задач із тегами та деталями.
- **Reports** — блок звітності з прогрес-барами по статусах.

Уся взаємодія — клієнтська (vanilla JS), без звернень до API чи серверної логіки.

## Технологічний стек

| Категорія | Інструменти |
|---|---|
| Збірка | [Vite](https://vitejs.dev/) 4 |
| Стилі | Sass/SCSS, PostCSS (`postcss-sort-media-queries`, `@fullhuman/postcss-purgecss`, `autoprefixer`) |
| JavaScript | Vanilla JS (ES-модулі) |
| Анімації/UX | `animejs`, `body-scroll-lock` |
| Оптимізація зображень | `vite-plugin-image-optimizer`, `imagemin`, `imagemin-webp`, `sharp`, `svgo` |
| CI/CD | GitHub Actions → деплой на GitHub Pages (`.github/workflows/deploy.yml`) |
| Форматування | Prettier (`.prettierrc.json`) |

## Структура проєкту

```
dashboard-cat-crm/
├── public/              # Статичні файли публічного доступу
├── src/
│   ├── fonts/            # Шрифти (Montserrat, woff/woff2)
│   ├── img/               # Зображення, іконки (icons.svg), webp-версії
│   ├── js/
│   │   └── main.js        # Точка входу: імпорт стилів, логіка sidebar-меню
│   └── scss/               # Стилі проєкту (детально нижче)
├── dist/                  # Згенерована продакшн-збірка (npm run build)
├── index.html              # Розмітка сторінки (UI kit / дашборд)
├── vite.config.js          # Конфігурація Vite (оптимізація зображень, PurgeCSS, entry points)
├── postcss.config.cjs       # Конфігурація PostCSS
└── .github/workflows/       # CI: автодеплой на GitHub Pages
```

## Стилізація через Sass/SCSS

Стилі організовані за принципом **7-1 (спрощений варіант)** — розділення на шари `utils → base → layout → components`, які підключаються через єдину точку входу `src/scss/main.scss`:

```scss
@use "base";
@use "layout";
@use "components";
```

### Структура шарів

```
scss/
├── utils/
│   ├── _variables.scss     # SCSS-мапи: кольори, брейкпоінти, відступи, шрифти, easing
│   ├── _functions.scss     # Функції (наприклад get-color() — вибірка кольору з мапи)
│   ├── _mixins.scss        # Міксини (media-query, frame, scroll-bar, ease)
│   └── _placeholders.scss  # %-плейсхолдери для @extend (типографіка, секції)
├── base/                   # Reset, базові стилі, шрифти, службові класи
├── layout/                 # Header, Sidebar, Main, Page — глобальна структура сторінки
└── components/              # Ізольовані компоненти (кнопки, бейджі, меню, задачі, звіти тощо)
```

### Ключові підходи

- **Змінні через SCSS-мапи.** Кольори, брейкпоінти, шрифти, відступи й easing-криві зберігаються не як окремі `$змінні`, а як мапи (`$colors`, `$breakpoints`, `$font-sizes` і т.д.) у `utils/_variables.scss`, що дозволяє звертатись до них через `map.get()`.
- **Функції.** `utils/_functions.scss` містить `get-color($key)` — обгортку над `map.get($colors, $key)` для зручного виклику кольору за назвою (`fn.get-color("accent-primary")`).
- **Міксини.** `utils/_mixins.scss`:
  - `media-query($bp, $type, $is-retina)` — адаптивні брейкпоінти (`min`/`max`-width, retina-медіазапити) на основі мапи `$breakpoints`;
  - `frame($width, $height, $is-circle)` — розміри та обрізка зображень/аватарів (у т.ч. кругла форма);
  - `scroll-bar()` — кастомізація скролбару (`::-webkit-scrollbar`);
  - `ease($ease, $properties...)` — уніфіковані transition на основі мапи `$easings`.
- **Плейсхолдери (`%...`) і `@extend`.** `utils/_placeholders.scss` визначає типографічні шаблони (`%main-title`, `%section-title`, `%main-text`, `%details` тощо) та `%section-frame` — вони перевикористовуються в компонентах через `@extend` замість дублювання CSS-властивостей.
- **Партіали та вкладеність папок.** Кожен компонент — окремий партіал (`_badges.scss`, `_menu.scss`, `_reports.scss`, `_tasks.scss`, `_welcome.scss` тощо), а складніші компоненти виділені у вкладені підпапки з власним `index.scss`, що імпортує внутрішні частини:
  - `components/btn/` — базові стилі кнопки (`_btn-base.scss`, `_btn.scss`) + `types/` з варіантами (`_primary`, `_secondary`, `_danger`, `_warning`, `_success`, `_info`, `_default`, `_frame`);
  - `components/coworkers/` — `_board.scss`, `_co-workers.scss`, `_details.scss`, `_table.scss`, зібрані через `index.scss`.
- **Модульний Sass (`@use`/`@forward`).** Проєкт використовує сучасний модульний синтаксис Sass (`@use ... as`) замість застарілого `@import`, що ізолює простір імен між файлами (`fn.get-color(...)`, `mx.media-query(...)`, `var.$colors`).
- **Вкладеність селекторів.** У компонентах активно використовується вкладена нотація SCSS (наприклад `&::-webkit-scrollbar-thumb`, `&:hover`) для читабельного опису станів і псевдоелементів у межах одного блоку.

## Як запустити проєкт локально

1. Встановити Node.js (LTS).
2. Встановити залежності:
   ```bash
   npm install
   ```
3. Запустити режим розробки (Vite dev-сервер із hot reload):
   ```bash
   npm run dev
   ```
   Проєкт буде доступний на [http://localhost:5173](http://localhost:5173).
4. Зібрати продакшн-версію:
   ```bash
   npm run build
   ```
   Результат — у папці `dist/`.
5. Локально переглянути продакшн-збірку:
   ```bash
   npm run preview
   ```
6. Збірка в режимі спостереження за змінами (наприклад, для перевірки згенерованого CSS у `dist/assets`):
   ```bash
   npm run watch
   ```

## Деплой

При пуші в гілку `main` GitHub Actions (`.github/workflows/deploy.yml`) автоматично збирає проєкт (`npm run build`) і публікує вміст `dist/` у гілку `gh-pages` для GitHub Pages.

## Ліцензія

[MIT](LICENSE)
