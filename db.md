# База данных Redspider Solution

## Общая информация

- **Тип:** MySQL 5.7.44
- **Кодировка:** UTF‑8
- **Общее количество таблиц:** 180+
- **Приложение:** Образовательная платформа с e‑commerce и email‑маркетингом

## Основные группы таблиц

### 1. Пользователи и авторизация
- `so_customer` — Клиенты/организации (основная сущность)
- `so_user` — Администраторы системы
- `so_customer_group` — Группы клиентов с правами
- `so_customer_auto_login_token` — Токены автологина
- `so_customer_password_reset` — Восстановление паролей

### 2. Образовательная платформа
- `so_course` — Курсы (основная таблица)
- `so_course_category` — Категории курсов (иерархическая)
- `so_course_topic` — Темы уроков
- `so_course_topic_lesson` — Уроки
- `so_course_schedule` — Расписание занятий
- `so_course_consultation` — Консультации по курсам

### 3. E‑commerce и заказы
- `so_shop_order` — Заказы
- `so_shop_cart` — Корзины покупок
- `so_shop_order_items` — Позиции в заказах
- `so_price_plan` — Тарифные планы
- `so_price_plan_order` — Заказы тарифных планов
- `so_course_discount` — Скидки и промокоды

### 4. Email‑маркетинг
- `so_campaign` — Email‑кампании
- `so_list` — Списки рассылки
- `so_list_subscriber` — Подписчики
- `so_campaign_delivery_log` — Логи доставки
- `so_campaign_track_open` — Отслеживание открытий
- `so_campaign_track_url` — Отслеживание кликов

### 5. Контент и статьи
- `so_article` — Статьи/блог
- `so_article_category` — Категории статей
- `so_article_comments` — Комментарии
- `so_page` — Статические страницы
- `so_master` — Управляемый контент

### 6. Видео‑конференции
- `so_meetings` — Онлайн‑встречи
- `so_meeting_participants` — Участники встреч
- `so_meeting_videos` — Записи видео‑конференций
- `so_meeting_video_participants` — Участники записей

### 7. Модули госуслуг (опционально)
- `so_module_birth_certificate` — Свидетельства о рождении
- `so_module_marriage_certificate` — Свидетельства о браке
- `so_module_business_registration` — Регистрация бизнеса
- `so_module_property_registration` — Регистрация имущества

### 8. Системные таблицы
- `so_option` — Настройки системы
- `so_session` — Сессии пользователей
- `so_cache` — Кеш данных
- `so_translation_source_message` — Переводы интерфейса
- `so_console_command_history` — История выполнения команд

### 9. Отзывы и рейтинги
- `so_reviews` — Отзывы о курсах
- `so_course_reviews` — Привязка отзывов к курсам
- `mw_reviews_deleted` — Удалённые отзывы

### 10. География
- `so_country` — Страны
- `so_zone` — Регионы/области
- `so_city_list` — Города
- `so_city_district` — Районы городов

## Ключевые связи

### 1. Основная цепочка курсов
- `so_customer` (организация)
- ` so_course` (курсы)
- `so_course_category` (категории)
- `so_course_topic` (темы)
- `so_course_topic_lesson` (уроки)

### 2. Цепочка заказов
- `so_customer` (клиент)
- `so_shop_cart` (корзина)
- `so_shop_order` (заказ)
- `so_shop_order_items` (позиции заказа)

### 3. Email‑маркетинг

- `so_customer` (владелец)
- `so_list` (списки)
- `so_list_subscriber` (подписчики)
- `so_campaign` (кампании)
- `so_campaign_delivery_log` (логи)


## Особенности структуры

### 1. Мульти‑тенантность
- Каждая организация (`so_customer`) имеет изолированные данные.
- `customer_id` присутствует в большинстве бизнес‑таблиц.

### 2. Иерархические категории
- `parent_id` для создания деревьев категорий.
- Используется в `so_course_category`, `so_article_category`.

### 3. Гибкая система контента
- Поля с суффиксом `_so` для перевода на сомалийский язык.
- Поле `f_type` для определения типа контента (`C` = курс, `A` = статья и т. д.).

### 4. Отслеживание активности
- Все таблицы имеют `date_added`, `last_updated`.
- Логирование действий пользователей.
- Аналитика email‑кампаний.

## Технические особенности

### 1. Индексы
- UID‑поля (уникальные идентификаторы): `*_uid`.
- Внешние ключи на `customer_id`, `course_id`, `order_id`.
- Составные индексы для часто используемых запросов.

### 2. Типы данных
- `DECIMAL` для финансовых операций.
- `BLOB`/`TEXT` для контента и настроек.
- `ENUM` для статусов и ограниченных значений.
- `JSON`/`Serialized data` в полях типа `serial_data`, `params_json`.

### 3. Архитектурные паттерны
- Active Record (Yii 1.1).
- Soft delete через `is_trash`‑флаги.
- Audit trail через таблицы логов.


## Производительность
- Крупные таблицы: `so_campaign_delivery_log`, `so_campaign_track_open`.
- Рекомендуется партиционирование для логов.
- Кеширование частых запросов через `so_cache`.
