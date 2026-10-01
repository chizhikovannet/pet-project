Функции системы

3.1 Модуль аутентификации и авторизации

3.1.1 Описание

Обеспечивает регистрацию новых пользователей, безопасно сохраняет учетные данные, аутентифицирует пользователей в системе и выдает маркеры доступа (JWT) для защиты закрытых эндпоинтов API.

3.1.2 Функциональные требования

FR-1: Система должна предоставлять регистрацию по логину и паролю.

FR-2: Пароль пользователя должен хэшироваться с использованием алгоритма bcrypt перед сохранением в базу данных.

FR-3: Система должна выполнять аутентификацию пользователя при входе и возвращать JWT-токен (access_token).

FR-4: Система должна проверять наличие и валидность JWT-токена в заголовке Authorization: Bearer  для всех защищенных эндпоинтов.

FR-5: Система должна поддерживать валидацию логина на уникальность и проверять сложность пароля (не менее 8 символов).

```mermaid
sequenceDiagram
    autonumber
    actor User as Пользователь
    participant Client as Frontend / API Client
    participant API as FastAPI Backend
    participant DB as База Данных

    alt Регистрация (FR-1, FR-2, FR-5)
        User->>Client: Ввод логина и пароля
        Client->>API: POST /api/auth/register
        API->>DB: SELECT * FROM users WHERE login = X
        alt Логин уже занят
            DB-->>API: Запись найдена
            API-->>Client: 400 Bad Request ("Логин занят")
        else Логин свободен
            DB-->>API: Запись не найдена
            API->>API: Хэширование пароля (bcrypt)
            API->>DB: INSERT INTO users (login, hashed_password)
            DB-->>API: Подтверждение
            API-->>Client: 201 Created ("Пользователь зарегистрирован")
        end
    else Вход в систему (FR-3, FR-4)
        User->>Client: Ввод логина и пароля
        Client->>API: POST /api/auth/login
        API->>DB: SELECT * FROM users WHERE login = X
        DB-->>API: Данные пользователя
        API->>API: Проверка пароля (bcrypt verify)
        alt Неверные данные
            API-->>Client: 401 Unauthorized ("Неверный логин или пароль")
        else Успешная аутентификация
            API->>API: Генерация JWT (access_token)
            API-->>Client: 200 OK {access_token, token_type: "bearer"}
        end
    end
```

3.2 Модуль управления анкетами пользователей

3.2.1 Описание

Позволяет студентам создавать, просматривать, редактировать и архивировать свои персональные анкеты, содержащие бытовые предпочтения и информацию о себе.

3.2.2 Функциональные требования

FR-6: Авторизованный пользователь должен иметь возможность создать только одну активную анкету.

FR-7: Анкета должна содержать обязательные поля: ФИО, город, ВУЗ, возраст/дата рождения, планируемый бюджет аренды, пол, отношение к курению/питомцам.

FR-8: Пользователь должен иметь возможность редактировать данные своей анкеты в любое время.

FR-9: Пользователь должен иметь возможность мягкого удаления (архивации) своей анкеты, после чего она скрывается из общего поиска.

FR-10: Пользователь должен иметь возможность восстановить анкету из архива.

Жизненный цикл анкеты пользователя 

```mermaid
stateDiagram-v2
    [*] --> Draft: Начало создания анкеты
    Draft --> Active: Обязательные поля заполнены (FR-7)
    
    state Active {
        [*] --> Published
        Published --> Visible: Отображается в поиске (FR-11)
    }

    Active --> Archived: Мягкое удаление / Пользователь скрыл (FR-9)
    Archived --> Active: Восстановление из архива (FR-10)

    Active --> Banned: Нарушение правил / Заблокировано модератором
    Banned --> Active: Разблокировка администратором
    
    Archived --> [*]: Окончательное удаление аккаунта
```

```mermaid
sequenceDiagram
    autonumber
    actor Student as Студент
    participant Client as Frontend / API Client
    participant API as FastAPI Backend
    participant DB as База Данных

    Student->>Client: Заполнение / Изменение анкеты
    Client->>API: POST/PUT /api/profiles/me (с JWT токеном)
    API->>API: Валидация JWT и данных Pydantic
    API->>DB: SELECT * FROM profiles WHERE user_id = current_user.id
    
    alt Создание анкеты (POST)
        alt Анкета уже существует (FR-6)
            DB-->>API: Запись найдена
            API-->>Client: 400 Bad Request ("Анкета уже создана")
        else Анкеты нет
            DB-->>API: Запись не найдена
            API->>DB: INSERT INTO profiles (user_id, full_name, budget, ...)
            DB-->>API: Данные сохранены
            API-->>Client: 201 Created (Объект профиля)
        end
    else Архивация / Восстановление (DELETE/POST) (FR-9, FR-10)
        API->>DB: UPDATE profiles SET status = 'archived' / 'active'
        DB-->>API: Успешно обновлено
        API-->>Client: 200 OK (Обновленный статус)
    end
```

3.3 Модуль поиска и фильтрации анкет

3.3.1 Описание

Обеспечивает поиск и подбор потенциальных сожителей по настраиваемым бытовым и финансовым критериям.

3.3.2 Функциональные требования

FR-11: Система должна возвращать список активных анкет других пользователей.

FR-12: Поиск должен поддерживать фильтрацию по следующим параметрам: город, ВУЗ, диапазон бюджета (мин/макс), отношение к курению и наличие животных.

FR-13: Система должна исключать из результатов поиска собственную анкету текущего пользователя и заблокированные аккаунты.

FR-14: Поддержка сортировки результатов (по дате создания, величине бюджета, среднему рейтингу).

```mermaid
sequenceDiagram
    autonumber
    actor Student as Студент
    participant Client as Frontend / API Client
    participant API as FastAPI Backend
    participant DB as База Данных

    Student->>Client: Задает фильтры (город, бюджет, ВУЗ)
    Client->>API: GET /api/search?city=Msk&budget_max=30000 (с JWT)
    API->>API: Извлечение user_id из JWT
    API->>DB: SELECT * FROM profiles WHERE status = 'active' AND user_id != current_user.id AND city = 'Msk' AND budget <= 30000 ORDER BY created_at DESC
    DB-->>API: Список отфильтрованных записей
    API-->>Client: 200 OK [Массив анкет со средним рейтингом]
    Client-->>Student: Отображение карточек сожителей
```

3.4 Модуль рейтинга и отзывов

3.4.1 Описание

Ключевой модуль системы, отвечающий за формирование прозрачного уровня доверия к пользователю на основе оценок и текстовых отзывов от других студентов.

3.4.2 Функциональные требования

FR-15: Пользователь должен иметь возможность выставить оценку (от 1 до 5 звезд) и оставить текстовый отзыв другому пользователю.

FR-16: Система должна автоматически рассчитывать средний рейтинг пользователя на основе всех полученных оценок.

FR-17: Поддержка детализированной оценки по критериям (чистоплотность, соблюдение тишины, своевременность оплаты, коммуникабельность).

FR-18: Пользователь не может ставить оценку самому себе.

FR-19: Защита от спама оценок (ограничение на повторное оставление отзыва одному и тому же пользователю).

```mermaid
sequenceDiagram
    autonumber
    actor StudentA as Студент A
    participant Client as Frontend / API Client
    participant API as FastAPI Backend
    participant DB as База Данных

    StudentA->>Client: Нажимает "Оставить отзыв"
    Client->>API: POST /api/ratings/{target_user_id}
    API->>API: Валидация JWT и проверка (StudentA != target_user_id)
    
    alt Попытка оценить самого себя
        API-->>Client: 400 Bad Request ("Нельзя оценить себя")
    else Валидация успешна
        API->>DB: INSERT INTO ratings
        DB-->>API: Подтверждение
        API->>DB: SELECT AVG(score) FROM ratings
        DB-->>API: Среднее значение
        API-->>Client: 201 Created (Обновленный профиль)
    end
```

3.5 Модуль «Избранное»

3.5.1 Описание

Позволяет сохранять интересные анкеты в персональный список быстрых закладок.

3.5.2 Функциональные требования

FR-20: Пользователь должен иметь возможность добавить анкету другого студента в список «Избранное».

FR-21: Пользователь должен иметь возможность просматривать свой список избранных анкет.

FR-22: Пользователь должен иметь возможность удалять анкеты из «Избранного».

```mermaid
sequenceDiagram
    autonumber
    actor Student as Студент
    participant Client as Frontend / API Client
    participant API as FastAPI Backend
    participant DB as База Данных

    alt Добавление в избранное (FR-20)
        Student->>Client: Нажимает "Добавить в избранное"
        Client->>API: POST /api/favorites/{profile_id} (с JWT)
        API->>DB: INSERT INTO favorites (user_id, target_profile_id)
        DB-->>API: Запись создана
        API-->>Client: 201 Created ("Добавлено")
    else Просмотр избранного (FR-21)
        Student->>Client: Переходит в раздел "Избранное"
        Client->>API: GET /api/favorites (с JWT)
        API->>DB: SELECT profiles.* FROM favorites JOIN profiles ON ... WHERE favorites.user_id = X
        DB-->>API: Список сохраненных анкет
        API-->>Client: 200 OK [Массив анкет]
    else Удаление из избранного (FR-22)
        Student->>Client: Нажимает "Удалить"
        Client->>API: DELETE /api/favorites/{profile_id} (с JWT)
        API->>DB: DELETE FROM favorites WHERE user_id = X AND target_profile_id = Y
        DB-->>API: Запись удалена
        API-->>Client: 200 OK ("Удалено")
    end
```

3.6 Модуль сообщений и диалогов (Чаты)

3.6.1 Описание

Предоставляет инструмент прямого текстового взаимодействия между потенциальными сожителями для обсуждения деталей аренды.

3.6.2 Функциональные требования

FR-23: Пользователь должен иметь возможность начать личный диалог с владельцем интересующей анкеты.

FR-24: Пользователь должен иметь возможность получать список своих активных чатов с указанием имени собеседника и последнего сообщения.

FR-25: Пользователь должен иметь возможность просматривать историю сообщений в выбранном диалоге.

FR-26: Доступ к переписке должны иметь только ее участники (защита на уровне авторизации API).

```mermaid
sequenceDiagram
    autonumber
    actor StudentA as Студент A
    participant Client as Frontend / API Client
    participant API as FastAPI Backend
    participant DB as База Данных

    StudentA->>Client: Открывает чат с Студентом B
    Client->>API: GET /api/messages/chats/{chat_id}/messages (с JWT)
    API->>DB: SELECT * FROM chat_participants WHERE chat_id = X AND user_id = StudentA.id
    
    alt Не участник чата (FR-26)
        DB-->>API: Запись не найдена
        API-->>Client: 403 Forbidden ("Нет доступа к переписке")
    else Доступ разрешен
        DB-->>API: Запись найдена
        API->>DB: SELECT * FROM messages WHERE chat_id = X ORDER BY created_at ASC
        DB-->>API: История сообщений
        API-->>Client: 200 OK [Список сообщений]
        
        StudentA->>Client: Вводит текст и отправляет сообщение
        Client->>API: POST /api/messages/chats/{chat_id}/messages {text: "Привет!"}
        API->>DB: INSERT INTO messages (chat_id, sender_id, text)
        DB-->>API: Сообщение сохранено
        API-->>Client: 201 Created (Объект сообщения)
    end
```
