# REST API v1

## 1. Общие договоренности

- Базовый префикс API: `/api/v1`
- Формат данных: `application/json`
- Идентификаторы сущностей: `UUID`
- Время: ISO 8601 UTC, например `2026-04-09T18:30:00Z`

### Авторизация

- Основной механизм для учебного MVP: `JWT Bearer`
- Заголовок доступа: `Authorization: Bearer <accessToken>`
- `accessToken` используется для доступа к защищенным endpoint-ам
- `refreshToken` используется для обновления access token
- Refresh token должен быть серверно управляемым и отзываться через logout
- Пароль из `LoginRequest` используется только для проверки учетных данных; в хранилище должен находиться только `password_hash`
- Для MVP допускается только одна активная refresh-сессия на пользователя
- Новый login инвалидирует предыдущую refresh-сессию пользователя
- При refresh сервер выдает новую пару токенов и отзывает предыдущий refresh token
- Рекомендуемый TTL для MVP: `accessToken = 15 минут`, `refreshToken = 7 дней`

### Пагинация

Для коллекционных endpoint-ов, где это уместно, используется общий формат:

```json
{
  "items": [],
  "page": 1,
  "pageSize": 10,
  "total": 42
}
```

### Сортировка

Общий формат query-параметров:

```text
?sortBy=createdAt&order=desc
```

Для списка предложений по умолчанию применяется бизнес-сортировка:

- сначала по `score` по убыванию;
- при равенстве по `createdAt` по возрастанию.

### Семантика score

- `score = количество голосов Up - количество голосов Down`
- Значение `score` является вычисляемым и не хранится как отдельное обязательное поле БД

## 2. Контрактные типы

### 2.1 Enum

#### `SuggestionStatus`

```json
["New", "InProgress", "Accepted", "Rejected"]
```

#### `VoteType`

```json
["Up", "Down"]
```

#### `DraftType`

```json
["Suggestion", "Comment"]
```

#### `ProjectRole`

```json
["Member", "Admin"]
```

### 2.2 DTO

#### `UserDto`

```json
{
  "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
  "displayName": "Иван Петров",
  "email": "ivan.petrov@example.local"
}
```

#### `LoginRequest`

```json
{
  "email": "ivan.petrov@example.local",
  "password": "password"
}
```

#### `RefreshTokenRequest`

```json
{
  "refreshToken": "refresh-token-value"
}
```

#### `LoginResponse`

```json
{
  "accessToken": "jwt-access-token",
  "refreshToken": "refresh-token-value",
  "expiresIn": 3600,
  "user": {
    "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
    "displayName": "Иван Петров",
    "email": "ivan.petrov@example.local"
  }
}
```

#### `CurrentUserResponse`

```json
{
  "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
  "displayName": "Иван Петров",
  "email": "ivan.petrov@example.local",
  "authMode": "DevLogin"
}
```

#### `ProjectSummary`

```json
{
  "id": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
  "name": "Core Platform",
  "description": "Проект команды Core Platform",
  "role": "Admin",
  "lastAccessedAt": "2026-04-09T18:30:00Z"
}
```

#### `ProjectMemberDto`

```json
{
  "userId": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
  "displayName": "Иван Петров",
  "email": "ivan.petrov@example.local",
  "role": "Member",
  "joinedAt": "2026-04-01T10:00:00Z"
}
```

#### `ProjectDetails`

```json
{
  "id": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
  "name": "Core Platform",
  "description": "Проект команды Core Platform",
  "createdByUserId": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
  "createdAt": "2026-04-01T10:00:00Z",
  "members": []
}
```

#### `SuggestionSummary`

```json
{
  "id": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
  "projectId": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
  "text": "Добавить обязательный шаблон ретро перед встречей",
  "status": "New",
  "author": {
    "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
    "displayName": "Иван Петров"
  },
  "score": 5,
  "createdAt": "2026-04-09T18:30:00Z",
  "updatedAt": "2026-04-09T18:30:00Z"
}
```

#### `VoteBreakdownItem`

```json
{
  "userId": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
  "displayName": "Иван Петров",
  "voteType": "Up",
  "createdAt": "2026-04-09T18:35:00Z"
}
```

#### `SuggestionDetails`

```json
{
  "id": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
  "projectId": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
  "text": "Добавить обязательный шаблон ретро перед встречей",
  "status": "New",
  "author": {
    "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
    "displayName": "Иван Петров"
  },
  "score": 5,
  "currentUserVote": "Up",
  "createdAt": "2026-04-09T18:30:00Z",
  "updatedAt": "2026-04-09T18:40:00Z",
  "votes": []
}
```

#### `CommentDto`

```json
{
  "id": "4da7d53c-3389-4bf1-ac11-d4e2720fccd9",
  "suggestionId": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
  "parentCommentId": null,
  "text": "Поддерживаю, это сократит время встречи.",
  "author": {
    "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
    "displayName": "Иван Петров"
  },
  "createdAt": "2026-04-09T18:45:00Z",
  "updatedAt": "2026-04-09T18:45:00Z"
}
```

#### `DraftDto`

```json
{
  "id": "f14855cd-0bfd-49cf-b59e-b41f5e8ef2aa",
  "projectId": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
  "type": "Comment",
  "payload": {
    "suggestionId": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
    "parentCommentId": "4da7d53c-3389-4bf1-ac11-d4e2720fccd9",
    "text": "Согласен, но нужно еще шаблон action items."
  },
  "updatedAt": "2026-04-09T19:00:00Z"
}
```

#### `CreateProjectRequest`

```json
{
  "name": "Core Platform",
  "description": "Проект команды Core Platform"
}
```

#### `AddProjectMemberRequest`

```json
{
  "userId": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
  "role": "Member"
}
```

#### `UpdateProjectMemberRoleRequest`

```json
{
  "role": "Admin"
}
```

#### `CreateSuggestionRequest`

```json
{
  "text": "Добавить обязательный шаблон ретро перед встречей"
}
```

#### `UpdateSuggestionRequest`

```json
{
  "text": "Добавить обязательный шаблон ретро и owner для action items"
}
```

#### `UpdateSuggestionStatusRequest`

```json
{
  "status": "InProgress"
}
```

#### `VoteRequest`

```json
{
  "voteType": "Up"
}
```

#### `CreateCommentRequest`

```json
{
  "text": "Поддерживаю, это сократит время встречи.",
  "parentCommentId": null
}
```

#### `UpdateCommentRequest`

```json
{
  "text": "Поддерживаю, это сократит время встречи и повысит предсказуемость."
}
```

#### `SaveSuggestionDraftRequest`

```json
{
  "text": "Добавить шаблон ретро..."
}
```

#### `SaveCommentDraftRequest`

```json
{
  "suggestionId": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
  "parentCommentId": null,
  "text": "Нужно уточнить формат шаблона."
}
```

## 3. Ошибки

Общий формат ошибки:

```json
{
  "errorCode": "validation_error",
  "message": "Поле text обязательно",
  "details": {}
}
```

Основные коды:

- `400 Bad Request` — ошибка валидации или некорректное состояние запроса
- `401 Unauthorized` — пользователь не аутентифицирован
- `403 Forbidden` — недостаточно прав
- `404 Not Found` — проект или сущность не найдены
- `409 Conflict` — конфликт состояния

## 4. Endpoints

### 4.1 `POST /api/v1/auth/login`

Назначение: аутентифицировать пользователя и выдать токены доступа.

Тело запроса: `LoginRequest`.

Пример запроса:

```json
{
  "email": "ivan.petrov@example.local",
  "password": "password"
}
```

Пример ответа:

```json
{
  "accessToken": "jwt-access-token",
  "refreshToken": "refresh-token-value",
  "expiresIn": 3600,
  "user": {
    "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
    "displayName": "Иван Петров",
    "email": "ivan.petrov@example.local"
  }
}
```

Ошибки: `400`, `401`.

### 4.2 `POST /api/v1/auth/refresh`

Назначение: обновить access token по refresh token.

Тело запроса: `RefreshTokenRequest`.

Пример запроса:

```json
{
  "refreshToken": "refresh-token-value"
}
```

Пример ответа:

```json
{
  "accessToken": "new-jwt-access-token",
  "refreshToken": "new-refresh-token-value",
  "expiresIn": 3600
}
```

Ошибки: `400`, `401`.

### 4.3 `POST /api/v1/auth/logout`

Назначение: завершить текущую refresh-сессию пользователя.

Тело запроса: `RefreshTokenRequest`.

Пример запроса:

```json
{
  "refreshToken": "refresh-token-value"
}
```

Пример ответа: `204 No Content`

Ошибки: `400`, `401`.

### 4.4 `GET /api/v1/users/me`

Назначение: получить текущего пользователя.

Пример ответа:

```json
{
  "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
  "displayName": "Иван Петров",
  "email": "ivan.petrov@example.local",
  "authMode": "DevLogin"
}
```

Ошибки: `401`.

### 4.5 `GET /api/v1/projects`

Назначение: вернуть список проектов, доступных текущему пользователю.

Query params:

- `page` — номер страницы, опционально
- `pageSize` — размер страницы, опционально

Пример ответа:

```json
{
  "items": [
    {
      "id": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
      "name": "Core Platform",
      "description": "Проект команды Core Platform",
      "role": "Admin",
      "lastAccessedAt": "2026-04-09T18:30:00Z"
    }
  ],
  "page": 1,
  "pageSize": 10,
  "total": 1
}
```

Ошибки: `401`.

### 4.6 `POST /api/v1/projects`

Назначение: создать новый проект. Создатель автоматически становится администратором.

Тело запроса: `CreateProjectRequest`.

Пример запроса:

```json
{
  "name": "Core Platform",
  "description": "Проект команды Core Platform"
}
```

Пример ответа:

```json
{
  "id": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
  "name": "Core Platform",
  "description": "Проект команды Core Platform",
  "createdByUserId": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
  "createdAt": "2026-04-01T10:00:00Z",
  "members": []
}
```

Ошибки: `400`, `401`.

### 4.7 `GET /api/v1/projects/{projectId}`

Назначение: получить карточку проекта и его участников.

Path params:

- `projectId` — идентификатор проекта

Пример ответа:

```json
{
  "id": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
  "name": "Core Platform",
  "description": "Проект команды Core Platform",
  "createdByUserId": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
  "createdAt": "2026-04-01T10:00:00Z",
  "members": [
    {
      "userId": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
      "displayName": "Иван Петров",
      "email": "ivan.petrov@example.local",
      "role": "Admin",
      "joinedAt": "2026-04-01T10:00:00Z"
    }
  ]
}
```

Ошибки: `401`, `403`, `404`.

### 4.8 `POST /api/v1/projects/{projectId}/members`

Назначение: добавить участника в проект.

Доступ: только администратор проекта.

Path params:

- `projectId`

Тело запроса: `AddProjectMemberRequest`.

Пример запроса:

```json
{
  "userId": "37b64aa3-970a-4d27-9966-b0f17d89f10a",
  "role": "Member"
}
```

Пример ответа:

```json
{
  "userId": "37b64aa3-970a-4d27-9966-b0f17d89f10a",
  "displayName": "Анна Соколова",
  "email": "anna.sokolova@example.local",
  "role": "Member",
  "joinedAt": "2026-04-09T19:10:00Z"
}
```

Ошибки: `400`, `401`, `403`, `404`, `409`.

### 4.9 `PATCH /api/v1/projects/{projectId}/members/{userId}`

Назначение: изменить роль участника проекта.

Доступ: только администратор проекта.

Path params:

- `projectId`
- `userId`

Тело запроса: `UpdateProjectMemberRoleRequest`.

Пример запроса:

```json
{
  "role": "Admin"
}
```

Пример ответа:

```json
{
  "userId": "37b64aa3-970a-4d27-9966-b0f17d89f10a",
  "displayName": "Анна Соколова",
  "email": "anna.sokolova@example.local",
  "role": "Admin",
  "joinedAt": "2026-04-09T19:10:00Z"
}
```

Ошибки: `400`, `401`, `403`, `404`.

### 4.10 `DELETE /api/v1/projects/{projectId}/members/{userId}`

Назначение: удалить участника из проекта.

Доступ: только администратор проекта.

Path params:

- `projectId`
- `userId`

Пример ответа: `204 No Content`

Ошибки: `401`, `403`, `404`.

### 4.11 `GET /api/v1/projects/{projectId}/suggestions`

Назначение: получить список предложений проекта по статусу.

Path params:

- `projectId`

Query params:

- `status` — один из `New`, `InProgress`, `Accepted`, `Rejected`
- `page` — номер страницы, опционально
- `pageSize` — размер страницы, опционально

Пример ответа:

```json
{
  "items": [
    {
      "id": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
      "projectId": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
      "text": "Добавить обязательный шаблон ретро перед встречей",
      "status": "New",
      "author": {
        "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
        "displayName": "Иван Петров"
      },
      "score": 5,
      "createdAt": "2026-04-09T18:30:00Z",
      "updatedAt": "2026-04-09T18:30:00Z"
    }
  ],
  "page": 1,
  "pageSize": 10,
  "total": 1
}
```

Ошибки: `400`, `401`, `403`, `404`.

### 4.12 `POST /api/v1/projects/{projectId}/suggestions`

Назначение: создать новое предложение в проекте.

Path params:

- `projectId`

Тело запроса: `CreateSuggestionRequest`.

Пример запроса:

```json
{
  "text": "Добавить обязательный шаблон ретро перед встречей"
}
```

Пример ответа:

```json
{
  "id": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
  "projectId": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
  "text": "Добавить обязательный шаблон ретро перед встречей",
  "status": "New",
  "author": {
    "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
    "displayName": "Иван Петров"
  },
  "score": 0,
  "createdAt": "2026-04-09T18:30:00Z",
  "updatedAt": "2026-04-09T18:30:00Z"
}
```

Ошибки: `400`, `401`, `403`, `404`.

### 4.13 `GET /api/v1/projects/{projectId}/suggestions/{suggestionId}`

Назначение: получить карточку предложения.

Path params:

- `projectId`
- `suggestionId`

Пример ответа:

```json
{
  "id": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
  "projectId": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
  "text": "Добавить обязательный шаблон ретро перед встречей",
  "status": "New",
  "author": {
    "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
    "displayName": "Иван Петров"
  },
  "score": 5,
  "currentUserVote": "Up",
  "createdAt": "2026-04-09T18:30:00Z",
  "updatedAt": "2026-04-09T18:40:00Z",
  "votes": [
    {
      "userId": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
      "displayName": "Иван Петров",
      "voteType": "Up",
      "createdAt": "2026-04-09T18:35:00Z"
    }
  ]
}
```

Ошибки: `401`, `403`, `404`.

### 4.14 `PATCH /api/v1/projects/{projectId}/suggestions/{suggestionId}`

Назначение: изменить текст предложения.

Доступ:

- только автор предложения;
- для MVP допускается редактирование текста только отдельно от смены статуса.

Path params:

- `projectId`
- `suggestionId`

Тело запроса: `UpdateSuggestionRequest`.

Пример запроса:

```json
{
  "text": "Добавить обязательный шаблон ретро и owner для action items"
}
```

Пример ответа:

```json
{
  "id": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
  "projectId": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
  "text": "Добавить обязательный шаблон ретро и owner для action items",
  "status": "New",
  "author": {
    "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
    "displayName": "Иван Петров"
  },
  "score": 5,
  "createdAt": "2026-04-09T18:30:00Z",
  "updatedAt": "2026-04-09T19:20:00Z"
}
```

Ошибки: `400`, `401`, `403`, `404`, `409`.

### 4.15 `PATCH /api/v1/projects/{projectId}/suggestions/{suggestionId}/status`

Назначение: изменить статус предложения.

Доступ:

- только администратор проекта.

Path params:

- `projectId`
- `suggestionId`

Тело запроса: `UpdateSuggestionStatusRequest`.

Пример запроса:

```json
{
  "status": "InProgress"
}
```

Пример ответа:

```json
{
  "id": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
  "projectId": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
  "text": "Добавить обязательный шаблон ретро и owner для action items",
  "status": "InProgress",
  "author": {
    "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
    "displayName": "Иван Петров"
  },
  "score": 5,
  "createdAt": "2026-04-09T18:30:00Z",
  "updatedAt": "2026-04-09T19:20:00Z"
}
```

Ошибки: `400`, `401`, `403`, `404`, `409`.

### 4.16 `PUT /api/v1/projects/{projectId}/suggestions/{suggestionId}/vote`

Назначение: создать новый голос или заменить существующий.

Path params:

- `projectId`
- `suggestionId`

Тело запроса: `VoteRequest`.

Пример запроса:

```json
{
  "voteType": "Down"
}
```

Пример ответа:

```json
{
  "suggestionId": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
  "currentUserVote": "Down",
  "score": 3
}
```

Ошибки: `400`, `401`, `403`, `404`.

### 4.17 `DELETE /api/v1/projects/{projectId}/suggestions/{suggestionId}/vote`

Назначение: отменить текущий голос пользователя.

Path params:

- `projectId`
- `suggestionId`

Пример ответа:

```json
{
  "suggestionId": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
  "currentUserVote": null,
  "score": 4
}
```

Ошибки: `401`, `403`, `404`.

### 4.18 `GET /api/v1/projects/{projectId}/suggestions/{suggestionId}/comments`

Назначение: получить все комментарии предложения плоским списком.

Комментарий на клиенте собирается в дерево по `parentCommentId`.

Path params:

- `projectId`
- `suggestionId`

Пример ответа:

```json
[
  {
    "id": "4da7d53c-3389-4bf1-ac11-d4e2720fccd9",
    "suggestionId": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
    "parentCommentId": null,
    "text": "Поддерживаю, это сократит время встречи.",
    "author": {
      "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
      "displayName": "Иван Петров"
    },
    "createdAt": "2026-04-09T18:45:00Z",
    "updatedAt": "2026-04-09T18:45:00Z"
  },
  {
    "id": "a644f68a-9754-42c7-9cd4-549617e0bca1",
    "suggestionId": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
    "parentCommentId": "4da7d53c-3389-4bf1-ac11-d4e2720fccd9",
    "text": "Согласен, нужен еще шаблон action items.",
    "author": {
      "id": "37b64aa3-970a-4d27-9966-b0f17d89f10a",
      "displayName": "Анна Соколова"
    },
    "createdAt": "2026-04-09T18:50:00Z",
    "updatedAt": "2026-04-09T18:50:00Z"
  }
]
```

Ошибки: `401`, `403`, `404`.

### 4.19 `POST /api/v1/projects/{projectId}/suggestions/{suggestionId}/comments`

Назначение: создать комментарий или ответ на комментарий.

Path params:

- `projectId`
- `suggestionId`

Тело запроса: `CreateCommentRequest`.

Пример запроса:

```json
{
  "text": "Поддерживаю, это сократит время встречи.",
  "parentCommentId": null
}
```

Пример ответа:

```json
{
  "id": "4da7d53c-3389-4bf1-ac11-d4e2720fccd9",
  "suggestionId": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
  "parentCommentId": null,
  "text": "Поддерживаю, это сократит время встречи.",
  "author": {
    "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
    "displayName": "Иван Петров"
  },
  "createdAt": "2026-04-09T18:45:00Z",
  "updatedAt": "2026-04-09T18:45:00Z"
}
```

Ошибки: `400`, `401`, `403`, `404`.

### 4.20 `PATCH /api/v1/projects/{projectId}/comments/{commentId}`

Назначение: отредактировать собственный комментарий.

Path params:

- `projectId`
- `commentId`

Тело запроса: `UpdateCommentRequest`.

Пример запроса:

```json
{
  "text": "Поддерживаю, это сократит время встречи и повысит предсказуемость."
}
```

Пример ответа:

```json
{
  "id": "4da7d53c-3389-4bf1-ac11-d4e2720fccd9",
  "suggestionId": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
  "parentCommentId": null,
  "text": "Поддерживаю, это сократит время встречи и повысит предсказуемость.",
  "author": {
    "id": "3f11a6dc-79a6-43f7-ac88-bb78dd70d712",
    "displayName": "Иван Петров"
  },
  "createdAt": "2026-04-09T18:45:00Z",
  "updatedAt": "2026-04-09T19:10:00Z"
}
```

Ошибки: `400`, `401`, `403`, `404`.

### 4.21 `DELETE /api/v1/projects/{projectId}/comments/{commentId}`

Назначение: удалить собственный комментарий.

Path params:

- `projectId`
- `commentId`

Пример ответа: `204 No Content`

Ошибки: `401`, `403`, `404`.

### 4.22 `GET /api/v1/projects/{projectId}/drafts`

Назначение: получить черновики текущего пользователя в проекте.

Path params:

- `projectId`

Query params:

- `type` — опционально, `Suggestion` или `Comment`
- `page` — номер страницы, опционально
- `pageSize` — размер страницы, опционально

Пример ответа:

```json
{
  "items": [
    {
      "id": "f14855cd-0bfd-49cf-b59e-b41f5e8ef2aa",
      "projectId": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
      "type": "Comment",
      "payload": {
        "suggestionId": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
        "parentCommentId": "4da7d53c-3389-4bf1-ac11-d4e2720fccd9",
        "text": "Согласен, но нужно еще шаблон action items."
      },
      "updatedAt": "2026-04-09T19:00:00Z"
    }
  ],
  "page": 1,
  "pageSize": 20,
  "total": 1
}
```

Ошибки: `401`, `403`, `404`.

### 4.23 `PUT /api/v1/projects/{projectId}/drafts/suggestion/{draftId}`

Назначение: создать или обновить черновик предложения.

Path params:

- `projectId`
- `draftId`

Тело запроса: `SaveSuggestionDraftRequest`.

Пример запроса:

```json
{
  "text": "Добавить шаблон ретро..."
}
```

Пример ответа:

```json
{
  "id": "0d4ef999-b2b0-43c6-bc97-b005a6852808",
  "projectId": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
  "type": "Suggestion",
  "payload": {
    "text": "Добавить шаблон ретро..."
  },
  "updatedAt": "2026-04-09T19:12:00Z"
}
```

Ошибки: `400`, `401`, `403`, `404`.

### 4.24 `PUT /api/v1/projects/{projectId}/drafts/comment/{draftId}`

Назначение: создать или обновить черновик комментария.

Path params:

- `projectId`
- `draftId`

Тело запроса: `SaveCommentDraftRequest`.

Пример запроса:

```json
{
  "suggestionId": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
  "parentCommentId": null,
  "text": "Нужно уточнить формат шаблона."
}
```

Пример ответа:

```json
{
  "id": "f14855cd-0bfd-49cf-b59e-b41f5e8ef2aa",
  "projectId": "7ca7d640-d843-45b2-9701-0b0efb8c4af1",
  "type": "Comment",
  "payload": {
    "suggestionId": "d68650b5-dfc5-45be-b525-8b0c64c4e54a",
    "parentCommentId": null,
    "text": "Нужно уточнить формат шаблона."
  },
  "updatedAt": "2026-04-09T19:15:00Z"
}
```

Ошибки: `400`, `401`, `403`, `404`.

### 4.25 `DELETE /api/v1/projects/{projectId}/drafts/{draftId}`

Назначение: удалить черновик текущего пользователя.

Path params:

- `projectId`
- `draftId`

Пример ответа: `204 No Content`

Ошибки: `401`, `403`, `404`.

## 5. Согласованные ограничения модели

- Роль администратора проекта хранится в membership-модели, отдельная таблица `project_admins` не требуется.
- Агрегированный `score` предложения вычисляется из активных голосов.
- Комментарии в API возвращаются плоским списком с `parentCommentId`; дерево строится на клиенте.
- Для MVP комментарии возвращаются без пагинации, целиком по предложению.
- Черновики моделируются как отдельная сущность с `type` и `payload`.
- В MVP `drafts.payload` хранится как JSON для ускорения реализации.
- Refresh token должен быть связан с серверной auth-session.
- Настройка количества голосов, расписание встреч и внешние интеграции вынесены в future scope.
