Требования к данным

4.1 Логическая модель данных 

База данных веб-сервиса состоит из 6 основных реляционных таблиц, обеспечивающих хранение аккаунтов, профилей, оценок, чатов и сообщений.

```mermaid
erDiagram
    USERS ||--o| PROFILES : "имеет"
    USERS ||--o{ RATINGS : "оставляет / получает"
    USERS ||--o{ FAVORITES : "добавляет"
    USERS ||--o{ MESSAGES : "отправляет"
    CHATS ||--o{ MESSAGES : "содержит"

    USERS {
        int id PK
        string login
        string hashed_password
        datetime created_at
    }
    PROFILES {
        int id PK
        int user_id FK
        string full_name
        string city
        string university
        int budget
        string status
    }
    RATINGS {
        int id PK
        int author_id FK
        int target_user_id FK
        int score
        string comment
    }
    FAVORITES {
        int id PK
        int user_id FK
        int target_profile_id FK
    }
    CHATS {
        int id PK
        datetime created_at
    }
    MESSAGES {
        int id PK
        int chat_id FK
        int sender_id FK
        text text
        boolean is_read
    }
```
    
Связи между сущностями:

users (1) ── (1) profiles (Один пользователь имеет одну анкету)

users (1) ── (N) favorites (Один пользователь может добавить много анкет в избранное)

users (1) ── (N) ratings (Один пользователь может оставить и получить множество отзывов)

users (N) ── (N) chats (Пользователи объединены в личные диалоги)

chats (1) ── (N) messages (В одном чате находится множество сообщений)
