## ⚙️ Настройки групп клиентов (Customer Group Options)

### 1. Структура таблицы so_customer_group_option:
```sql
   option_id       INT           # ID настройки
   group_id        INT           # ID группы (связь с customer_group)
   code            VARCHAR       # Код настройки (например: 'system.customer_sending.quota')
   is_serialized   TINYINT(1)    # Сериализовано ли значение? (0/1)
   value           TEXT          # Значение настройки (сериализованное или строка)
   date_added      DATETIME      # Дата создания
   last_updated    DATETIME      # Дата обновления
```
### 2. Сериализация значений:
```php
   protected function beforeSave()
   {
   $this->is_serialized = 0;
   if ($this->value !== null && !is_string($this->value)) {
   $this->value = @serialize($this->value);
   $this->is_serialized = 1;
   }
   return parent::beforeSave();
   }
   ```
- Нестроковые значения сериализуются в базу
- Строковые значения хранятся как есть
- Автоматическая десериализация при чтении

### 3. Типы настроек (предполагаемые из кода):
   Из анализа CustomerGroup.php и Customer.php:

#### Основные категории настроек:

- Системные настройки (system.customer_*)
- Настройки отправки (system.customer_sending.*)
- Настройки квот (quota, hourly_quota)
- Настройки регистрации (registration.*)
- Настройки прав доступа (access.*)

### 4. Примеры настроек из кода:
```php
   // Из Customer.php:
   'system.customer_servers.max_bounce_servers'
   'system.customer_servers.max_delivery_servers'
   'system.customer_campaigns.max_campaigns'
   'system.customer_lists.max_subscribers'
   'system.customer_sending.quota'
   'system.customer_sending.hourly_quota'
   'system.customer_sending.quota_time_value'
```
### Таблица: `so_customer_group_option`

| Поле | Тип | Описание |
|------|-----|----------|
| `option_id` | INT | Уникальный ID настройки |
| `group_id` | INT | Связь с группой |
| `code` | VARCHAR | Код настройки (уникальный в рамках группы) |
| `is_serialized` | TINYINT(1) | 1 = значение сериализовано, 0 = строка |
| `value` | TEXT | Значение настройки |
| `date_added` | DATETIME | Дата создания |
| `last_updated` | DATETIME | Дата обновления |

### Система хранения значений:

#### 1. **Сериализация:**
```php
// Автоматическая сериализация при сохранении
if (!is_string($this->value)) {
    $this->value = serialize($this->value);
    $this->is_serialized = 1;
}

// Автоматическая десериализация при чтении
if ($this->is_serialized) {
    $this->value = unserialize($this->value);
}
```