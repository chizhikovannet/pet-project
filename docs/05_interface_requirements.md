Требования к внешним интерфейсам

5.1 Пользовательские интерфейсы 

Для тестирования, отладки и взаимодействия с API используется автоматическая интерактивная документация:
- Swagger UI 
- ReDoc 

5.2 Интерфейсы ПО 

Все запросы к API передаются по протоколу HTTP/HTTPS. Данные в теле запроса (Request Body) и ответа (Response Body) передаются в формате application/json.   

Аутентификация защищенных эндпоинтов осуществляется с помощью JWT-токенов в заголовке: Authorization: Bearer    

Сводная таблица эндпоинтов API:

| Модуль | Метод | Эндпоинт | Доступ | Описание |
| :--- | :--- | :--- | :--- | :--- |
| **Auth** | `POST` | `/api/auth/register` | Публичный | Регистрация нового студента |
| **Auth** | `POST` | `/api/auth/login` | Публичный | Вход в систему и получение JWT-токена |
| **Profiles** | `POST` | `/api/profiles` | Bearer Token | Создание анкеты текущего пользователя |
| **Profiles** | `GET` | `/api/profiles/me` | Bearer Token | Получение собственной анкеты |
| **Profiles** | `PUT` | `/api/profiles/me` | Bearer Token | Редактирование своей анкеты |
| **Profiles** | `DELETE` | `/api/profiles/me` | Bearer Token | Архивирование анкеты (`soft delete`) |
| **Profiles** | `POST` | `/api/profiles/restore` | Bearer Token | Восстановление анкеты из архива |
| **Search** | `GET` | `/api/search` | Bearer Token | Поиск и фильтрация анкет студентов |
| **Search** | `GET` | `/api/search/{profile_id}` | Bearer Token | Просмотр подробной карточки анкеты |
| **Ratings** | `POST` | `/api/ratings/{user_id}` | Bearer Token | Оставить отзыв и оценку пользователю |
| **Ratings** | `GET` | `/api/ratings/{user_id}` | Bearer Token | Получить все отзывы и средний рейтинг пользователя |
| **Favorites** | `GET` | `/api/favorites` | Bearer Token | Просмотр сохраненных анкет |
| **Favorites** | `POST` | `/api/favorites/{profile_id}` | Bearer Token | Добавление анкеты в «Избранное» |
| **Favorites** | `DELETE` | `/api/favorites/{profile_id}` | Bearer Token | Удаление анкеты из «Избранного» |
| **Chats** | `GET` | `/api/messages/chats` | Bearer Token | Получение списка личных диалогов |
| **Chats** | `POST` | `/api/messages/chats` | Bearer Token | Создание или переход в чат с пользователем |
| **Messages** | `GET` | `/api/messages/chats/{id}/messages` | Bearer Token | История сообщений в чате |
| **Messages** | `POST` | `/api/messages/chats/{id}/messages` | Bearer Token | Отправка личного сообщения |

5.3 Коммуникационные интерфейсы

Протокол передачи данных: HTTP/1.1 или HTTP/2 поверх TLS/SSL (HTTPS) для обеспечения шифрования трафика.

Формат ошибок: В случае возникновения ошибки сервер возвращает стандартный HTTP-код состояния (400, 401, 403, 404, 422, 500) и тело ответа с сообщением об ошибке:

JSON
{
  "detail": "Текст с описанием ошибки"
}
