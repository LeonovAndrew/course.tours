### CustomerMessage.php
## 🎯 Назначение модели
CustomerMessage - это система внутренних уведомлений и сообщений между системой и клиентами (школами, преподавателями, студентами).

## 📊 Структура таблицы so_customer_message:

```sql
message_id                   INT           # Уникальный ID сообщения
message_uid                  VARCHAR       # Публичный UUID сообщения
customer_id                  INT           # ID получателя (клиента)
title                        VARCHAR(255)  # Заголовок сообщения (ключ перевода)
message                      TEXT          # Текст сообщения (ключ перевода)
title_translation_params     TEXT          # Параметры для перевода заголовка (сериализованные)
message_translation_params   TEXT          # Параметры для перевода текста (сериализованные)
status                       ENUM          # Статус: 'unseen'/'seen'
date_added                   DATETIME      # Дата создания
last_updated                 DATETIME      # Дата обновления
```

## 🔍 Ключевые особенности:
### 1. Система переводов (i18n):

```php
public function getTranslatedTitle(): string
{
    if (!empty($this->title_translation_params)) {
        return t('messages', $this->title, $this->title_translation_params);
    }
    return t('messages', (string)$this->title);
}
```
- title и message хранятся как ключи перевода
- Параметры хранятся сериализованными в *_translation_params
- Использует Yii функцию t() для локализации

### 2. Статусы сообщений:
```php
const STATUS_UNSEEN = 'unseen';
const STATUS_SEEN = 'seen';

public function getIsUnseen(): bool
{
    return $this->getStatusIs(self::STATUS_UNSEEN);
}
```
- Unseen - новое, непрочитанное сообщение
- Seen - прочитанное сообщение
### 3. Массовая рассылка (Broadcast):

```php
public function broadcast(): void
{
    CustomerCollection::findAll($criteria)->each(function (Customer $customer) {
        $message = clone $this;
        $message->customer_id = (int)$customer->customer_id;
        $message->save();
    });
}
```
- Отправка одного сообщения всем активным клиентам
- Используется для системных уведомлений
## 🎯 Типы сообщений (из кода):

### 1. Системные уведомления:

```php
// Примеры из кода:
'Please confirm your email'
'Confirmed your email' 
'New customer registration'
'New seat request'
```
### 2. Бизнес-уведомления:
```php
'Upcoming Conference Details'   // О предстоящей конференции
'Course Purchased'              // О покупке курса
'new registration'              // О новой регистрации
```
### 3. Связанные с действиями:
```php
// Интеграция с CustomerActionLog
'category_title'    // Категория действия
'relation_id'       // ID связанной сущности
'message_log'       // Сообщение из лога действий
```
## 🔗 Интеграция с другими моделями:
### С CustomerActionLog:

```php
// В search() методе:
$criteria->join .= ' LEFT JOIN {{customer_action_log}} lg on t.log_id = lg.log_id';

// Отображение связанных действий:
public function getMessageTitle()
{
    $LogModel = new CustomerActionLog();
    $LogModel->message = $this->message_log;
    $LogModel->category = $this->category_title;
    // ... формирование заголовка с ссылкой
}
```
### С курсами и встречами:
```php
public function getCourseTitle(): string
{
    $info = json_decode($this->info, true);
    if (!empty($info['course_id'])) {
        $course = Course::model()->findByPk($info['course_id']);
        return $course ? $course->getCourseTitle() : '';
    }
    return '';
}

public function getMessageDetail()
{
    switch ($this->title) {
        case 'Upcoming Conference Details':
            $meeting = Meetings::model()->findByAttributes(['room_id' => $info['room_uid']]);
            return '<div>Конференция: <a href="' . $meeting->MeetingUrl . '">' . $meeting->course_title . '</a></div>';
        case 'Course Purchased':
            $course = Course::model()->findByPk($info['course_id']);
            return '<div>Курс: <a href="' . $course->detailUrl . '">' . $course->title . '</a></div>';
    }
}
```
## 📱 Интерфейсные методы:
### 1. Короткие версии для списков:

```php
public function getShortMessage(int $length = 45): string
{
    return StringHelper::truncateLength($this->getMessageTitle1(), $length);
}

public function getShortTitle(int $length = 25): string
{
    return StringHelper::truncateLength($this->getTranslatedTitle(), $length);
}
```
### 2. Ссылки на сообщения:
```php
public function getMessageLink()
{
    if (Yii::App()->isAppName('backend')) {
        return createUrl('messages/view', ['message_uid' => $this->message_uid]);
    } else {
        return createUrl('notifications/view', ['message_uid' => $this->message_uid]);
    }
}
```
### 3. Форматированное отображение:

```php
public function getMessageTitle1()
{
    if (!empty($this->category_title)) {
        // Форматирование с HTML
        return '<a href="' . createUrl('messages/view', ['message_uid' => $this->message_uid]) . '">' 
               . strip_tags(nl2br($this->message), '<b><br><span>') . '</a>';
    }
}
```
## 🔄 Жизненный цикл сообщения:
### Создание:
```php
$message = new CustomerMessage();
$message->customer_id = $customerId;
$message->title = 'New Course Enrollment';
$message->message = 'Student {student} enrolled in {course}';
$message->message_translation_params = [
    '{student}' => $studentName,
    '{course}' => $courseTitle
];
$message->status = CustomerMessage::STATUS_UNSEEN;
$message->save();
```
### Отметка как прочитанное:
```php
// Для одного сообщения
$message->saveStatus(CustomerMessage::STATUS_SEEN);

// Для всех сообщений пользователя
CustomerMessage::markAllAsSeenForCustomer($customerId);
```

## 🎯 Примеры использования:
### 1. Уведомление о покупке курса:

```php
// При успешной покупке
$message = new CustomerMessage();
$message->customer_id = $instructorId;
$message->title = 'Course Purchased';
$message->message = 'Student {studentName} purchased your course "{courseTitle}"';
$message->message_translation_params = [
    '{studentName}' => $student->getFullName(),
    '{courseTitle}' => $course->title
];
$message->save();
```
### 2. Напоминание о конференции:
```php
// За 1 час до конференции
$message = new CustomerMessage();
$message->customer_id = $participantId;
$message->title = 'Upcoming Conference Details';
$message->message = 'Conference "{meetingTitle}" starts in 1 hour';
$message->message_translation_params = [
    '{meetingTitle}' => $meeting->title
];
// Дополнительная информация в info поле
$message->info = json_encode([
    'room_uid' => $meeting->room_id,
    'utc' => $meeting->start_time,
    'course_id' => $meeting->course_id
]);
$message->save();
```

### 3. Системное уведомление:
```php
// При изменении правил системы
$message = new CustomerMessage();
$message->title = 'System Update Notification';
$message->message = 'Please be advised of upcoming system maintenance on {date}';
$message->message_translation_params = [
    '{date}' => date('Y-m-d', strtotime('+3 days'))
];
$message->broadcast(); // Отправка всем активным пользователям
```
## 📊 Поиск и фильтрация:
### Поля для поиска:
```php
public function search()
{
    $criteria->compare('t.title', $this->title, true);
    $criteria->compare('t.message', $this->message, true);
    $criteria->compare('t.status', $this->status);
    
    // Поиск по имени клиента
    if (!is_numeric($this->customer_id)) {
        $criteria->with['customer'] = [
            'condition' => 'customer.email LIKE :name OR customer.first_name LIKE :name',
            'params' => [':name' => '%' . $this->customer_id . '%'],
        ];
    }
}
```

## 🔐 Безопасность и сериализация:
### Сериализация параметров:
```php
protected function beforeSave()
{
    if (!empty($this->title_translation_params)) {
        $this->title_translation_params = serialize($this->title_translation_params);
    }
    // ... аналогично для message_translation_params
    return true;
}

protected function afterFind()
{
    if (!empty($this->title_translation_params)) {
        $this->title_translation_params = unserialize((string)$this->title_translation_params);
    }
    // ...
}
```

## 🎯 Ключевые преимущества системы

- **Локализация:** полная поддержка перевода сообщений;
- **Гибкость:** параметризованные сообщения с динамическими данными;
- **Интеграция:** связь с бизнес‑процессами (курсы, встречи, покупки);
- **Масштабируемость:** массовая рассылка и эффективное хранение;
- **UX:** разные представления для backend/frontend, короткие версии для списков.

## 📈 Использование в бизнес‑процессах

### Для школ:
- Уведомления о новых регистрациях;
- Запросы на места на курсы;
- Напоминания о платежах.

### Для преподавателей:
- Покупки их курсов;
- Вопросы от студентов;
- Напоминания о конференциях.

### Для студентов:
- Подтверждение регистрации;
- Уведомления о новых уроках;
- Напоминания о дедлайнах заданий;
- Реферальные начисления.

## Описание компонента

`CustomerMessage` — это центральный хаб коммуникации в системе, соединяющий все бизнес‑процессы с конечными пользователями через локализованные, параметризованные уведомления.
