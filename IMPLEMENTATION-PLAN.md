# План реализации backend

На основе артефактов deadline-01 (TZ.md, API.md, er-diagram.md).

## Этап 1. Каркас проекта

- Создать solution `CollabSystem` с проектами:
  - `CollabSystem.Api` — веб-приложение, контроллеры
  - `CollabSystem.Domain` — сущности, интерфейсы репозиториев, бизнес-правила
  - `CollabSystem.Infrastructure` — EF Core, миграции, реализация репозиториев
- Настроить ASP.NET Core Minimal API / Controllers
- Подключить EF Core (PostgreSQL)
- Настроить Swagger/OpenAPI
- Настроить Serilog (структурированное логирование)

## Этап 2. Доменная модель и БД

Реализовать сущности из ER-диаграммы:

- `User` — id, displayName, email, domainLogin, createdAt, updatedAt
- `Project` — id, name, description, createdByUserId, createdAt, updatedAt
- `ProjectMember` — id, projectId, userId, role, joinedAt
- `Suggestion` — id, projectId, authorId, status, text, createdAt, updatedAt
- `Vote` — id, suggestionId, userId, voteType, createdAt, updatedAt
- `Comment` — id, projectId, suggestionId, authorId, parentCommentId, text, createdAt, updatedAt, deletedAt
- `Draft` — id, projectId, userId, type, suggestionId, parentCommentId, payloadJson, createdAt, updatedAt

Ограничения:
- UK на `ProjectMember (project_id, user_id)`
- UK на `Vote (suggestion_id, user_id)`
- `score` — вычисляемый, не хранится

Настроить DbContext, конфигурации через Fluent API, создать начальную миграцию.

## Этап 3. Репозитории

Интерфейсы в Domain, реализация в Infrastructure:

- `IUserRepository` — GetById, GetCurrentUser (абстракция)
- `IProjectRepository` — GetAllForUser, GetById, Create
- `IProjectMemberRepository` — Add, Remove, UpdateRole, GetByProject, GetMembership
- `ISuggestionRepository` — GetByProjectAndStatus, GetById, Create, Update
- `IVoteRepository` — GetByUserAndSuggestion, Create, Update, Delete, GetBySuggestion
- `ICommentRepository` — GetTreeBySuggestion, Create, Update, SoftDelete
- `IDraftRepository` — GetByUserAndProject, Save, Delete

## Этап 4. Сервисный слой — бизнес-логика

Реализовать правила из TZ:

- `ProjectService` — создание проекта, автороль админа
- `ProjectMemberService` — добавление/удаление участников, проверка прав админа
- `SuggestionService` — CRUD предложений, фильтрация по статусу, сортировка (score desc, createdAt asc)
- `VoteService` — поставить/заменить/отменить голос, вычисление score
- `CommentService` — CRUD комментариев, древовидная структура, soft delete
- `DraftService` — сохранить/получить/удалить черновик
- `CurrentUserService` — абстракция получения текущего пользователя (для dev-режима)

Проверки доступа:
- Пользователь состоит в проекте — для всех операций
- Администратор проекта — для управления участниками
- Автор предложения — для редактирования текста
- Автор комментария — для редактирования/удаления

## Этап 5. API-контроллеры

Реализовать 21 endpoint из API.md:

| # | Метод | Маршрут | Контроллер |
|---|-------|---------|------------|
| 4.1 | GET | /api/projects | ProjectsController |
| 4.2 | POST | /api/projects | ProjectsController |
| 4.3 | GET | /api/projects/{projectId} | ProjectsController |
| 4.4 | POST | /api/projects/{projectId}/members | ProjectMembersController |
| 4.5 | PATCH | /api/projects/{projectId}/members/{userId} | ProjectMembersController |
| 4.6 | DELETE | /api/projects/{projectId}/members/{userId} | ProjectMembersController |
| 4.7 | GET | /api/projects/{projectId}/suggestions | SuggestionsController |
| 4.8 | POST | /api/projects/{projectId}/suggestions | SuggestionsController |
| 4.9 | GET | /api/projects/{projectId}/suggestions/{suggestionId} | SuggestionsController |
| 4.10 | PATCH | /api/projects/{projectId}/suggestions/{suggestionId} | SuggestionsController |
| 4.11 | PUT | /api/projects/{projectId}/suggestions/{suggestionId}/vote | VotesController |
| 4.12 | DELETE | /api/projects/{projectId}/suggestions/{suggestionId}/vote | VotesController |
| 4.13 | GET | /api/projects/{projectId}/suggestions/{suggestionId}/comments | CommentsController |
| 4.14 | POST | /api/projects/{projectId}/suggestions/{suggestionId}/comments | CommentsController |
| 4.15 | PATCH | /api/projects/{projectId}/comments/{commentId} | CommentsController |
| 4.16 | DELETE | /api/projects/{projectId}/comments/{commentId} | CommentsController |
| 4.17 | GET | /api/projects/{projectId}/drafts | DraftsController |
| 4.18 | PUT | /api/projects/{projectId}/drafts/suggestion/{draftId} | DraftsController |
| 4.19 | PUT | /api/projects/{projectId}/drafts/comment/{draftId} | DraftsController |
| 4.20 | DELETE | /api/projects/{projectId}/drafts/{draftId} | DraftsController |
| 4.21 | GET | /api/users/me | UsersController |

DTO и контрактные типы (enum, request/response) — из API.md.

Обработка ошибок: единый middleware → формат `errorCode`, `message`, `details`.

## Этап 6. Авторизация (dev-режим)

- Реализовать `ICurrentUserService` с методом `GetCurrentUserAsync()`
- Dev-реализация: dev-login endpoint, хранение в cookie/сессии
- Middleware подстановки тестового пользователя
- Возможность подключения реальной доменной авторизации без изменения бизнес-логики

## Этап 7. Тестирование

- Unit-тесты сервисного слоя (xUnit):
  - правила голосования (один голос на предложение, замена, отмена)
  - проверки доступа (админ/автор/участник)
  - сортировка предложений
  - soft delete комментариев
- Integration-тесты API (WebApplicationFactory):
  - полный цикл: создать проект → добавить участников → создать предложение → проголосовать → добавить комментарий → сохранить черновик
  - негативные сценарии (403, 404, 409)

## Этап 8. Инфраструктура и запуск

- Dockerfile для API
- docker-compose (API + PostgreSQL)
- Скрипт применения миграций при старте
- Настройка переменных окружения (connection string, auth mode)
- Seed-данные для dev-режима

## Порядок работы

1. Этап 1 — каркас, без бизнес-логики
2. Этапы 2 + 3 параллельно — модель + репозитории
3. Этап 6 — авторизация (нужна сервисам)
4. Этап 4 — бизнес-логика
5. Этап 5 — контроллеры
6. Этап 7 — тесты
7. Этап 8 — инфраструктура
