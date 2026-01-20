# Модуль «Статьи» — Backend часть

## Обзор
Backend-модуль предоставляет административный интерфейс для управления статьями, категориями и связанным контентом. Поддерживается многоязычность, сложные связи (many-to-many), приоритизация, AJAX-операции и расширяемая архитектура.

---

## Архитектура

### 1. Структура файлов

```
apps/backend/controllers/
├── ArticlesController.php              # Управление статьями
└── Article_categoriesController.php    # Управление категориями

apps/backend/views/
├── articles/
│   ├── list.php                        # Список статей
│   └── form.php                        # Форма статьи
└── article_categories/
    ├── list.php                        # Список категорий
    └── form.php                        # Форма категории

apps/backend/assets/js/
└── articles.js                         # JavaScript для статей
```

---

## 2. Контроллеры

### A. ArticlesController

```php
actionIndex($id = null)          // Список статей (фильтр по категории)
actionCreate($cid = null)        // Создание статьи
actionUpdate($id, $cid = null)   // Редактирование статьи
actionDelete($id)                // Удаление статьи
actionSlug()                     // AJAX генерация slug
actionView_ajax($id, $cid)       // AJAX быстрый просмотр
loadCategoryModel($id)           // Загрузка категории
_registerJuiBs($event)           // Регистрация UI-компонентов
```

### B. Article_categoriesController

```php
actionIndex()        // Список категорий + массовый перевод
actionCreate()       // Создание категории
actionUpdate($id)    // Редактирование категории
actionDelete($id)    // Удаление категории
actionSlug()         // AJAX генерация slug
```

---

## 3. Модели и связи

### Таблицы базы данных

```sql
so_article
├── article_id
├── title
├── title_so
├── content
├── content_so
├── slug
├── status
├── priority
└── category_id

so_article_category
├── category_id
├── parent_id
├── name
├── slug
├── page_style
├── banner
└── description

so_article_to_category
├── article_id
└── category_id

so_article_images
├── id
├── article_id
├── file_name
└── caption
```

---

## 4. Функциональность

### A. Управление статьями

- WYSIWYG-редактор
- Два языка (EN / SO)
- Загрузка изображений (Dropzone.js)
- Multiple select категорий
- SEO-friendly slug
- Приоритет сортировки

---

## 5. Безопасность

- CSRF защита
- XSS фильтрация
- Валидация данных

---

## 6. AJAX и интеграции

- Генерация slug
- Быстрый просмотр
- TranslationService
- Hooks API

---

## 7. Производительность и расширения

- Кеширование
- Пагинация
- Версионирование контента
