# 5. Интерфейсы взаимодействия системы (ICD-lite)

В рамках проектируемой системы аренды парковочных мест были определены внешние и внутренние интерфейсы взаимодействия между компонентами системы.

Внешние интерфейсы используются для взаимодействия клиентских приложений с серверной частью системы посредством REST API.

Внутренние интерфейсы используются для обмена событиями между сервисами системы через брокер сообщений Kafka. Для сериализации сообщений используется формат Protocol Buffers (protobuf).

---

# Внешние интерфейсы: Parking Rental Platform API

Версия API: 1.0.0  
Формат взаимодействия: REST API / JSON  
Протокол: HTTPS

## Servers

`https://parking-rent.example.com/api/v1`

---

# 1. Авторизация пользователя

## POST /auth/login

Авторизация пользователя в системе.

### Общая информация о эндпоинте

| Параметр | Значение |
|---|---|
| Тип взаимодействия | REST API |
| Метод | POST |
| Endpoint | /auth/login |
| Producer | Frontend (React.js) |
| Consumer | Authentication Service |
| Формат данных | JSON |
| Версия API | v1 |

### JSON-схема запроса

```json
{
  "email": "user@mail.com",
  "password": "string"
}
```

### Поля запроса

| Поле | Тип | Описание |
|---|---|---|
| email | string | Электронная почта пользователя |
| password | string | Пароль пользователя |

### JSON-схема ответа

```json
{
  "userId": 15,
  "role": "CLIENT",
  "token": "jwt-token",
  "expiresIn": 86400
}
```

### Поля ответа

| Поле | Тип | Описание |
|---|---|---|
| userId | long | Идентификатор пользователя |
| role | string | Роль пользователя |
| token | string | JWT токен |
| expiresIn | integer | Время действия токена |

### Возможные ошибки

| Код ошибки | Описание |
|---|---|
| INVALID_CREDENTIALS | Неверный логин или пароль |
| USER_NOT_FOUND | Пользователь не найден |
| ACCOUNT_BLOCKED | Аккаунт заблокирован |
| INTERNAL_SERVER_ERROR | Внутренняя ошибка сервера |

---

# 2. Получение списка парковочных зон

## GET /parking-zones

Получение списка парковочных зон.

### Общая информация о эндпоинте

| Параметр | Значение |
|---|---|
| Тип взаимодействия | REST API |
| Метод | GET |
| Endpoint | /parking-zones |
| Producer | Frontend (React.js) |
| Consumer | Parking Zone Service |
| Формат данных | JSON |
| Версия API | v1 |

### Query Parameters

| Поле | Тип | Описание |
|---|---|---|
| latitude | double | Географическая широта |
| longitude | double | Географическая долгота |
| radius | integer | Радиус поиска |

### JSON-схема ответа

```json
[
  {
    "id": 1,
    "title": "ЖК Северный",
    "address": "ул. Ленина, 10",
    "latitude": 51.533,
    "longitude": 46.034,
    "availableSpaces": 12
  }
]
```

### Возможные ошибки

| Код ошибки | Описание |
|---|---|
| INVALID_COORDINATES | Некорректные координаты |
| ZONE_NOT_FOUND | Парковочная зона не найдена |
| INTERNAL_SERVER_ERROR | Внутренняя ошибка сервера |

---

# 3. Создание бронирования

## POST /bookings

Создание бронирования парковочного места.

### Общая информация о эндпоинте

| Параметр | Значение |
|---|---|
| Тип взаимодействия | REST API |
| Метод | POST |
| Endpoint | /bookings |
| Producer | Frontend (React.js) |
| Consumer | Booking Service |
| Формат данных | JSON |
| Версия API | v1 |

### JSON-схема запроса

```json
{
  "parkingSpaceId": 15,
  "startTime": "2026-05-12T10:00:00",
  "endTime": "2026-05-12T18:00:00",
  "paymentMethodId": 5
}
```

### Поля запроса

| Поле | Тип | Описание |
|---|---|---|
| parkingSpaceId | long | Идентификатор парковочного места |
| startTime | timestamp | Время начала бронирования |
| endTime | timestamp | Время окончания бронирования |
| paymentMethodId | long | Способ оплаты |

### JSON-схема ответа

```json
{
  "bookingId": 88,
  "status": "PENDING_CONFIRMATION",
  "totalPrice": 1500
}
```

### Возможные ошибки

| Код ошибки | Описание |
|---|---|
| PARKING_SPACE_NOT_FOUND | Парковочное место не найдено |
| BOOKING_TIME_CONFLICT | Временной интервал занят |
| INVALID_TIME_RANGE | Некорректное время |
| INTERNAL_SERVER_ERROR | Внутренняя ошибка сервера |

---

# 4. Отправка сообщения

## POST /messages

Отправка сообщения пользователю.

### Общая информация о эндпоинте

| Параметр | Значение |
|---|---|
| Тип взаимодействия | REST API |
| Метод | POST |
| Endpoint | /messages |
| Producer | Frontend (React.js) |
| Consumer | Messaging Service |
| Формат данных | JSON |
| Версия API | v1 |

### JSON-схема запроса

```json
{
  "chatId": 5,
  "recipientId": 12,
  "content": "Здравствуйте, место ещё свободно?"
}
```

### JSON-схема ответа

```json
{
  "messageId": 100,
  "status": "SENT",
  "timestamp": "2026-05-12T18:15:00"
}
```

### Возможные ошибки

| Код ошибки | Описание |
|---|---|
| CHAT_NOT_FOUND | Чат не найден |
| ACCESS_DENIED | Нет доступа |
| EMPTY_MESSAGE | Пустое сообщение |

---

# Внутренние интерфейсы: Kafka Events

Для обеспечения масштабируемости системы используется брокер сообщений Apache Kafka.

---

# 5. Booking Created Event

Событие создания бронирования.

### Общая информация о событии

| Параметр | Значение |
|---|---|
| Тип взаимодействия | Kafka |
| Название топика | parking.booking.created.v1 |
| Producer | Booking Service |
| Consumer | Notification Service, Payment Service |
| Гарантия доставки | at-least-once |
| Версия события | v1 |

### Protobuf schema

```proto
syntax = "proto3";

package parking.booking.created.v1;

message BookingCreatedEvent {

  string booking_id = 1;

  string client_id = 2;

  string parking_space_id = 3;

  string parking_zone_id = 4;

  int64 start_time_unix = 5;

  int64 end_time_unix = 6;

  int32 total_price = 7;

  string status = 8;

  int64 created_at_unix = 9;
}
```

### Возможные ошибки обработки

| Код ошибки | Описание |
|---|---|
| BOOKING_ALREADY_EXISTS | Бронь уже существует |
| INVALID_TIME_RANGE | Некорректное время |
| PARKING_SPACE_UNAVAILABLE | Место недоступно |

---

# 6. Parking Space Availability Changed Event

Событие изменения доступности парковочного места.

### Общая информация о событии

| Параметр | Значение |
|---|---|
| Тип взаимодействия | Kafka |
| Название топика | parking.space.availability.v1 |
| Producer | Parking Space Service |
| Consumer | Search Service, Analytics Service |
| Гарантия доставки | at-least-once |
| Версия события | v1 |

### Protobuf schema

```proto
syntax = "proto3";

package parking.space.availability.v1;

message ParkingSpaceAvailabilityChangedEvent {

  string parking_space_id = 1;

  string owner_id = 2;

  string previous_status = 3;

  string new_status = 4;

  int64 changed_at_unix = 5;
}
```

### Возможные ошибки обработки

| Код ошибки | Описание |
|---|---|
| SPACE_NOT_FOUND | Парковочное место не найдено |
| INVALID_STATUS_TRANSITION | Некорректный переход статуса |
| INTERNAL_PROCESSING_ERROR | Ошибка обработки |

---

# 7. Complaint Created Event

Событие создания жалобы.

### Общая информация о событии

| Параметр | Значение |
|---|---|
| Тип взаимодействия | Kafka |
| Название топика | parking.complaint.created.v1 |
| Producer | Complaint Service |
| Consumer | Moderation Service, Admin Service |
| Гарантия доставки | at-least-once |
| Версия события | v1 |

### Protobuf schema

```proto
syntax = "proto3";

package parking.complaint.created.v1;

message ComplaintCreatedEvent {

  string complaint_id = 1;

  string complainant_id = 2;

  string accused_user_id = 3;

  string complaint_text = 4;

  string status = 5;

  int64 created_at_unix = 6;
}
```

### Возможные ошибки обработки

| Код ошибки | Описание |
|---|---|
| USER_NOT_FOUND | Пользователь не найден |
| DUPLICATE_COMPLAINT | Повторная жалоба |
| INVALID_COMPLAINT_TYPE | Некорректный тип жалобы |

---

# Общий формат событий (Event Envelope)

Все события внутри системы публикуются через единый формат EventEnvelope.

### Protobuf schema

```proto
syntax = "proto3";

package parking.events.v1;

message EventEnvelope {

  // Уникальный идентификатор события
  string event_id = 1;

  // Тип события
  string event_type = 2;

  // Временная метка создания события
  int64 created_at_unix = 3;

  // Сервис-источник события
  string source_service = 4;

  // Полезная нагрузка события
  oneof payload {

    BookingCreatedEvent booking_created = 5;

    ParkingSpaceAvailabilityChangedEvent availability_changed = 6;

    ComplaintCreatedEvent complaint_created = 7;
  }
}
```

---

# Enum: BookingStatus

Статусы бронирования парковочного места.

| Name | Number | Description |
|---|---|---|
| BOOKING_STATUS_UNSPECIFIED | 0 | Не указан |
| PENDING_CONFIRMATION | 1 | Ожидает подтверждения |
| CONFIRMED | 2 | Подтверждено |
| CANCELLED | 3 | Отменено |
| COMPLETED | 4 | Завершено |

---

# Enum: ParkingSpaceStatus

Статусы парковочного места.

| Name | Number | Description |
|---|---|---|
| SPACE_STATUS_UNSPECIFIED | 0 | Не указан |
| AVAILABLE | 1 | Доступно |
| RESERVED | 2 | Забронировано |
| BLOCKED | 3 | Заблокировано |
| INACTIVE | 4 | Неактивно |

---

# Enum: ComplaintStatus

Статусы жалобы.

| Name | Number | Description |
|---|---|---|
| COMPLAINT_STATUS_UNSPECIFIED | 0 | Не указан |
| OPEN | 1 | Жалоба создана |
| IN_PROGRESS | 2 | На рассмотрении |
| RESOLVED | 3 | Решено |
| REJECTED | 4 | Отклонено |
