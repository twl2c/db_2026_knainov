## Подзапросы в SELECT

Название каждого чата и общее количество сообщений:

```sql
SELECT title,
       (SELECT COUNT(*) FROM messages) AS total_messages
FROM chats;
```

Имя каждого пользователя и время последнего сообщения в системе:

```sql
SELECT username,
       (SELECT MAX(sent_at) FROM messages) AS last_message_time
FROM users;
```

## Подзапросы в FROM

Чаты, в которых больше одного сообщения:

```sql
SELECT t.chat_id, t.cnt
FROM (SELECT chat_id, COUNT(*) AS cnt
      FROM messages
      GROUP BY chat_id) t
WHERE t.cnt > 1;
```

Имена пользователей и количество написанных ими сообщений:

```sql
SELECT u.username, s.cnt
FROM users u
JOIN (SELECT user_id, COUNT(*) AS cnt
      FROM messages
      GROUP BY user_id) s ON u.user_id = s.user_id;
```

## Подзапросы в WHERE

Самый первый созданный чат:

```sql
SELECT *
FROM chats
WHERE created_at = (SELECT MIN(created_at) FROM chats);
```

Автор сообщения с id = 1:

```sql
SELECT username
FROM users
WHERE user_id = (SELECT user_id FROM messages WHERE message_id = 1);
```

## Подзапросы в HAVING

Чаты, в которых сообщений больше, чем четверть от всех сообщений:

```sql
SELECT chat_id, COUNT(*) AS cnt
FROM messages
GROUP BY chat_id
HAVING COUNT(*) > (SELECT COUNT(*) FROM messages) / 4.0;
```

Чаты, в которых участников больше, чем в чате с id = 4:

```sql
SELECT chat_id, COUNT(*) AS members_count
FROM chat_members
GROUP BY chat_id
HAVING COUNT(*) > (SELECT COUNT(*) FROM chat_members WHERE chat_id = 4);
```

## ALL

Чаты, созданные позже, чем все чаты с id 1 и 2:

```sql
SELECT title
FROM chats
WHERE created_at > ALL (SELECT created_at
                        FROM chats
                        WHERE chat_id IN (1, 2));
```

Пользователи, у которых id больше, чем у всех авторов сообщений в первом чате:

```sql
SELECT username
FROM users
WHERE user_id > ALL (SELECT user_id
                     FROM messages
                     WHERE chat_id = 1);
```

## IN

Пользователи, которые написали хотя бы одно сообщение:

```sql
SELECT username
FROM users
WHERE user_id IN (SELECT user_id FROM messages);
```

Чаты, в которых есть администратор:

```sql
SELECT title
FROM chats
WHERE chat_id IN (SELECT chat_id
                  FROM chat_members
                  WHERE role = 'admin');
```

## ANY

Пользователи, которые состоят в первом чате:

```sql
SELECT username
FROM users
WHERE user_id = ANY (SELECT user_id
                     FROM chat_members
                     WHERE chat_id = 1);
```

Сообщения, отправленные позже, чем вступил хотя бы один администратор:

```sql
SELECT message_id, text
FROM messages
WHERE sent_at > ANY (SELECT joined_at
                     FROM chat_members
                     WHERE role = 'admin');
```

## EXISTS

Чаты, в которых есть хотя бы одно сообщение:

```sql
SELECT title
FROM chats c
WHERE EXISTS (SELECT 1
              FROM messages m
              WHERE m.chat_id = c.chat_id);
```

Пользователи, которые ни разу не ставили реакцию:

```sql
SELECT username
FROM users u
WHERE NOT EXISTS (SELECT 1
                  FROM reactions r
                  WHERE r.user_id = u.user_id);
```

## Сравнение по нескольким столбцам

Участники чатов, которые писали сообщения в этих же чатах:

```sql
SELECT *
FROM chat_members
WHERE (chat_id, user_id) IN (SELECT chat_id, user_id FROM messages);
```

Сообщения, написанные администраторами в своих чатах:

```sql
SELECT *
FROM messages
WHERE (chat_id, user_id) IN (SELECT chat_id, user_id
                             FROM chat_members
                             WHERE role = 'admin');
```

## Коррелированные подзапросы

1. Количество сообщений каждого пользователя:

```sql
SELECT u.username,
       (SELECT COUNT(*)
        FROM messages m
        WHERE m.user_id = u.user_id) AS messages_count
FROM users u;
```

2. Последнее сообщение в каждом чате:

```sql
SELECT *
FROM messages m
WHERE m.sent_at = (SELECT MAX(sent_at)
                   FROM messages
                   WHERE chat_id = m.chat_id);
```

3. Участники, которые вступили в чат раньше, чем написали там первое сообщение:

```sql
SELECT cm.chat_id, cm.user_id
FROM chat_members cm
WHERE cm.joined_at < (SELECT MIN(m.sent_at)
                      FROM messages m
                      WHERE m.chat_id = cm.chat_id
                        AND m.user_id = cm.user_id);
```

4. Чаты, в которых не меньше двух участников:

```sql
SELECT c.title
FROM chats c
WHERE (SELECT COUNT(*)
       FROM chat_members cm
       WHERE cm.chat_id = c.chat_id) >= 2;
```

5. Сообщения без вложений:

```sql
SELECT m.message_id, m.text
FROM messages m
WHERE NOT EXISTS (SELECT 1
                  FROM attachments a
                  WHERE a.message_id = m.message_id);
```
