# Модуль "Статьи" - Frontend часть

## Обзор
Frontend модуль статей предоставляет интерфейс для публичного доступа к контенту с поддержкой разных типов материалов (блоги, FAQ, галереи, события).


## 1. Структура файлов

```
apps/customer/controllers/ArticlesController.php   # Контроллер
apps/customer/views/articles/                      # Шаблоны
├── index.php                                      # Главная + поиск
├── category.php                                   # Категория (общий)
├── view.php                                       # Детальная статья
├── content_page.php                               # Контент-страница
├── Специальные страницы/
│   ├── testimonail.php                            # Отзывы
│   ├── partners.php                               # Партнеры
│   ├── about_us.php                               # О нас
│   ├── add_a_school.php                           # Добавить школу
│   └── instructor.php                            # Стать инструктором
├── Стили категорий (page_style)/
│   ├── category/
│   │   ├── blog.php                               # Блог
│   │   └── faq.php                                # FAQ
│   ├── detail/
│   │   └── blog.php                               # Детальная блога
│   ├── event/
│   │   ├── event_listing.php                     # События
│   │   └── _event_listing_style.php              # Стиль событий
│   ├── gallery/
│   │   └── view.php                              # Галерея
│   └── publication/
│       ├── publiction_listing.php                # Публикации
│       └── _publication_listing_style.php
├── Компоненты/
│   ├── include/
│   │   ├── widget_share.php                      # Кнопки «Поделиться»
│   │   └── slider_latest_updates.php             # Слайдер обновлений
│   ├── plan_sidebar.php                          # Сайдбар плана
│   └── latest_blogs.php                          # Последние блоги
└── Партиалы/
    ├── _search_items.php                         # Элементы поиска
    └── _share_widget.php                         # Виджет шаринга
```

---

## 2. Контроллер (ArticlesController)

### Действия

```php
actionIndex()    // Главная страница + поиск
actionCategory() // Статьи категории
actionView()     // Детальная статья
```

### Ключевая логика

- Динамический выбор шаблонов по `page_style`
- Специальные обработки для конкретных `slug`
- Проверка прав доступа к неопубликованному контенту
- Локализация контента (английский / сомалийский)

---

## 3. Маршрутизация

```
/                     → Главная (SiteController)
/articles             → Все статьи
/articles/page/2      → Пагинация
/articles/{slug}      → Категория статей
/article/{slug}       → Детальная статья
```

### Примеры

```
/articles/blog
/article/how-to-create-course
```

---

## 4. Типы контента (page_style)

| Style          | Шаблон                                   | Лимит | Особенности              |
|----------------|-------------------------------------------|-------|--------------------------|
| blog           | category/blog.php                         | 10    | Стандартный блог         |
| faq            | category/faq.php                          | 100   | Вопросы–ответы           |
| gallery        | gallery/view.php                          | 100   | Изображения              |
| publication    | publication/publiction_listing.php        | 100   | Документы, PDF           |
| event-page     | event/event_listing.php                   | 30    | Сортировка по дате       |
| testimonials   | testimonail.php                           | –     | Отзывы клиентов          |
| our-partners   | partners.php                              | –     | Партнеры                 |

---

## 5. Бизнес-воронки

### Стать инструктором

```
instructor.php
└── instructor-instructions.php
```

### Создать курс

```
plan-your-course.php
├── plan_sidebar.php
├── create-your-course.php
└── promote-your-course.php
```

---

## 6. Шаблоны

### A. Главная (index.php)

- Форма поиска
- Результаты поиска (`_search_items.php`)
- Пагинация (`CLinkPager`)
- Сообщение «Нет результатов»

### B. Категория (category.php)

- Баннер категории
- Грид статей (3 колонки)
- Пагинация
- Локализация

### C. Элемент статьи

```php
<div class="col-sm-6 col-md-4 p-b-40">
    <div class="blog-item">
        <div class="hov-img0">
            <a href="<?php echo $v->detailUrl; ?>">
                <img src="<?php echo $v->ArticleImage; ?>" alt="IMG-BLOG">
            </a>
        </div>
        <div class="p-t-15">
            <span class="cl5"><?php echo Yii::t('app', $v->eventDate); ?></span>
            <h4 class="p-b-12">
                <a href="<?php echo $v->detailUrl; ?>">
                    <?php echo Yii::t('articles', $v->ArticleTitle); ?>
                </a>
            </h4>
            <p class="stext-108 cl6"><?php echo $v->excerpt; ?></p>
        </div>
    </div>
</div>
```

---

## 7. Модели и данные

### Статья

```php
$article->detailUrl
$article->ArticleImage
$article->ArticleTitle
$article->eventDate
$article->excerpt
$article->slug
$article->status
```

### Категория

```php
$category->banner
$category->name
$category->page_style
$category->parent_id
```

---

## 8. Локализация

Файлы переводов:

```
protected/messages/en/articles.php
protected/messages/so/articles.php
```

Использование:

```php
Yii::t('articles', $category->name);
Yii::t('articles', $article->title);
Yii::t('app', 'common phrases');
```

---

## 9. SEO и производительность

### SEO

- ЧПУ URL
- Meta-теги в контроллере
- Canonical links
- Alt для изображений

### Производительность

- Пагинация (10–100)
- Адаптивная верстка
- Ленивая загрузка

---

## 10. Стили и адаптивность

```css
@media (max-width: 577px) {
    .blog-item { margin-bottom: 35px !important; }
    .stext-108 { display: none; }
}
```

---

## 11. Особенности реализации

- Динамический выбор шаблонов
- Хардкод для отдельных slug
- Баннеры категорий через inline-style

---

## 12. Компоненты и виджеты

- Widget Share (Facebook, Twitter, LinkedIn)
- Slider Latest Updates

---

## 13. Безопасность

- Draft-статьи видны только администраторам
- XSS-фильтрация
- Проверка доступа

---

## 14. Пути к ресурсам

```php
apps()->getBaseUrl('assets/files/articles/' . $category->banner);
```

---

## 15. Навигация и UX

- ACTIVE_LINK
- Хлебные крошки
- Подсветка меню

---

## 16. Возможности для расширения

- Поиск с фильтрами
- Теги и рейтинги
- Связанные статьи
- API
- Кеширование
- CDN
- AMP
- Push-уведомления
