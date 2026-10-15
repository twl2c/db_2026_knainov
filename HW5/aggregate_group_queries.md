# 1. COUNT(), SUM(), AVG(), MIN(), MAX(), STRING_AGG()

**Запрос 1.** Количество сообщений, первое и последнее время отправки.

```sql
SELECT COUNT(*) AS total_messages,
       MIN(sent_at) AS first_message,
       MAX(sent_at) AS last_message
FROM messages;
```

**Запрос 2.** Суммарная и средняя длина текста сообщений, а также все тексты через запятую.

```sql
SELECT SUM(LENGTH(text)) AS total_length,
       AVG(LENGTH(text)) AS avg_length,
       STRING_AGG(text, ', ') AS all_texts
FROM messages;
```

# 2. GROUP BY, HAVING

**Запрос 1.** Количество сообщений в каждом чате.

```sql
SELECT chat_id, COUNT(*) AS messages_count
FROM messages
GROUP BY chat_id;
```

**Запрос 2.** Чаты, в которых больше одного сообщения.

```sql
SELECT chat_id, COUNT(*) AS messages_count
FROM messages
GROUP BY chat_id
HAVING COUNT(*) > 1;
```

# 3. GROUPING SETS, ROLLUP и CUBE

## GROUPING SETS. 

**Запрос 1.** Количество участников отдельно по чатам и отдельно по ролям.

```sql
SELECT chat_id, role, COUNT(*) AS members_count
FROM chat_members
GROUP BY GROUPING SETS ((chat_id), (role));
```

**Запрос 2.** Количество пользователей по статусам и общее количество.

```sql
SELECT status, COUNT(*) AS users_count
FROM users
GROUP BY GROUPING SETS ((status), ());
```

## ROLLUP 

**Запрос 1.** Участники по чатам и ролям с итогами по чату и общим итогом.

```sql
SELECT chat_id, role, COUNT(*) AS members_count
FROM chat_members
GROUP BY ROLLUP (chat_id, role);
```

**Запрос 2.** Сообщения по чатам и пользователям с итогами.

```sql
SELECT chat_id, user_id, COUNT(*) AS messages_count
FROM messages
GROUP BY ROLLUP (chat_id, user_id);
```

## CUBE
**Запрос 1.** Все комбинации чата и роли.

```sql
SELECT chat_id, role, COUNT(*) AS members_count
FROM chat_members
GROUP BY CUBE (chat_id, role);
```

**Запрос 2.** Все комбинации чата и пользователя для сообщений.

```sql
SELECT chat_id, user_id, COUNT(*) AS messages_count
FROM messages
GROUP BY CUBE (chat_id, user_id);
```

## 4. SELECT, FROM, WHERE, GROUP BY, HAVING, ORDER BY

**Запрос 1.** Количество сообщений по чатам начиная с 20 сентября, только чаты с сообщениями, сортировка по убыванию.

```sql
SELECT chat_id, COUNT(*) AS messages_count
FROM messages
WHERE sent_at >= '2026-09-20 00:00:00'
GROUP BY chat_id
HAVING COUNT(*) >= 1
ORDER BY messages_count DESC;
```

**Запрос 2.** Количество пользователей по статусам (только с почтой на mail.com), сортировка по количеству.

```sql
SELECT status, COUNT(*) AS users_count
FROM users
WHERE email LIKE '%@mail.com'
GROUP BY status
HAVING COUNT(*) >= 1
ORDER BY users_count DESC;
```
