# ER-диаграмма

```mermaid
erDiagram
    USERS {
        uuid id PK
        string display_name
        string email UK
        string password_hash
        string domain_login
        datetime created_at
        datetime updated_at
    }

    PROJECTS {
        uuid id PK
        string name
        string description
        uuid created_by_user_id FK
        datetime created_at
        datetime updated_at
    }

    PROJECT_MEMBERS {
        uuid id PK
        uuid project_id FK
        uuid user_id FK
        string role "Member|Admin"
        datetime joined_at
    }

    SUGGESTIONS {
        uuid id PK
        uuid project_id FK
        uuid author_id FK
        string status "New|InProgress|Accepted|Rejected"
        text text
        datetime created_at
        datetime updated_at
    }

    VOTES {
        uuid id PK
        uuid suggestion_id FK
        uuid user_id FK
        string vote_type "Up|Down"
        datetime created_at
        datetime updated_at
    }

    COMMENTS {
        uuid id PK
        uuid project_id FK
        uuid suggestion_id FK
        uuid author_id FK
        uuid parent_comment_id FK
        text text
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    DRAFTS {
        uuid id PK
        uuid project_id FK
        uuid user_id FK
        string type "Suggestion|Comment"
        uuid suggestion_id FK
        uuid parent_comment_id FK
        json payload_json
        datetime created_at
        datetime updated_at
    }

    AUTH_SESSIONS {
        uuid id PK
        uuid user_id FK
        string refresh_token_hash
        datetime expires_at
        datetime revoked_at
        datetime created_at
        datetime updated_at
    }

    USERS ||--o{ PROJECTS : creates
    USERS ||--o{ PROJECT_MEMBERS : joins
    PROJECTS ||--o{ PROJECT_MEMBERS : contains

    PROJECTS ||--o{ SUGGESTIONS : contains
    USERS ||--o{ SUGGESTIONS : authors

    SUGGESTIONS ||--o{ VOTES : collects
    USERS ||--o{ VOTES : casts

    PROJECTS ||--o{ COMMENTS : contains
    SUGGESTIONS ||--o{ COMMENTS : has
    USERS ||--o{ COMMENTS : writes
    COMMENTS ||--o{ COMMENTS : replies_to

    PROJECTS ||--o{ DRAFTS : contains
    USERS ||--o{ DRAFTS : owns
    SUGGESTIONS ||--o{ DRAFTS : references
    COMMENTS ||--o{ DRAFTS : parent_for

    USERS ||--o{ AUTH_SESSIONS : owns
```

## Ограничения модели

- Роль администратора хранится в `PROJECT_MEMBERS`, отдельная таблица `project_admins` не требуется.
- На пару `project_id + user_id` в `PROJECT_MEMBERS` действует ограничение уникальности.
- На пару `suggestion_id + user_id` в `VOTES` действует ограничение уникальности.
- `score` предложения не хранится отдельно и вычисляется как производная величина из активных голосов.
- `score = количество голосов Up - количество голосов Down`.
- Для учебного MVP локальная авторизация использует `password_hash` в `USERS`, а не пароль в открытом виде.
- Для корневого комментария `parent_comment_id = null`.
- Для черновика предложения `suggestion_id` и `parent_comment_id` могут быть `null`.
- Для черновика комментария обязательно хранить `suggestion_id`, а `parent_comment_id` заполняется только для ответа.
- Refresh token не хранится в открытом виде и должен быть привязан к `AUTH_SESSIONS`.
- Для MVP допускается только одна активная refresh-сессия на пользователя.
