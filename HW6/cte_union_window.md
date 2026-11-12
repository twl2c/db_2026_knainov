# HW6. CTE, UNION, оконные функции

## Добавление данных

```sql
INSERT INTO users (username, email, status, phone) VALUES
('olga', 'olga@mail.com', 'online', '+79000000005'),
('petr', 'petr@mail.com', 'offline', '+79000000006');

INSERT INTO chats (title, created_at, type, description) VALUES
('Друзья', '2026-09-23 18:00:00', 'group', 'Чат друзей'),
('Архив', '2026-09-24 12:00:00', 'private', 'Старый чат');

INSERT INTO messages (chat_id, user_id, text, sent_at) VALUES
(1, 3, 'Всем добрый день', '2026-09-20 10:10:00'),
(1, 1, 'Сегодня созвон в 15:00', '2026-09-20 10:20:00'),
(2, 1, 'Работа идёт по плану', '2026-09-20 11:15:00'),
(2, 3, 'Отлично, жду отчёт', '2026-09-20 11:20:00'),
(3, 2, 'Новости на сегодня', '2026-09-21 12:05:00'),
(3, 4, 'Спасибо за новости', '2026-09-21 12:10:00'),
(4, 1, 'Сдаём в пятницу', '2026-09-22 09:20:00'),
(4, 4, 'Хорошо, успеем', '2026-09-22 09:25:00'),
(5, 5, 'Кто идёт в кино?', '2026-09-23 18:05:00'),
(5, 2, 'Я иду', '2026-09-23 18:10:00'),
(5, 5, 'Тогда встречаемся в 19:00', '2026-09-23 18:15:00');

INSERT INTO chat_members (chat_id, user_id, joined_at, role) VALUES
(1, 3, '2026-09-20 10:02:00', 'member'),
(2, 1, '2026-09-20 11:01:00', 'member'),
(2, 4, '2026-09-20 11:02:00', 'member'),
(3, 2, '2026-09-21 12:00:00', 'admin'),
(3, 4, '2026-09-21 12:01:00', 'member'),
(4, 1, '2026-09-22 09:01:00', 'member'),
(5, 5, '2026-09-23 18:00:00', 'admin'),
(5, 2, '2026-09-23 18:01:00', 'member');

INSERT INTO attachments (message_id, file_name, file_type) VALUES
(1, 'hello.png', 'image'),
(6, 'agenda.pdf', 'pdf'),
(7, 'report.docx', 'docx'),
(9, 'news.png', 'image'),
(11, 'project.zip', 'zip'),
(13, 'poster.jpg', 'image');

INSERT INTO reactions (message_id, user_id, emoji) VALUES
(1, 2, 'like'),
(1, 3, 'smile'),
(2, 1, 'like'),
(6, 2, 'like'),
(6, 4, 'fire'),
(9, 1, 'smile'),
(11, 4, 'like'),
(13, 2, 'like'),
(13, 3, 'smile');
```

## 1. CTE

**1.1. Количество сообщений в каждом чате**

```sql
WITH chat_msg AS (
    SELECT chat_id, COUNT(*) AS msg_count
    FROM messages
    GROUP BY chat_id
)
SELECT c.title, cm.msg_count
FROM chats c
JOIN chat_msg cm ON cm.chat_id = c.chat_id;
```


**1.2. Сколько сообщений написали пользователи со статусом online**

```sql
WITH online_users AS (
    SELECT user_id, username
    FROM users
    WHERE status = 'online'
)
SELECT o.username, COUNT(m.message_id) AS msg_count
FROM online_users o
JOIN messages m ON m.user_id = o.user_id
GROUP BY o.username;
```


**1.3. Сообщения первого чата и количество реакций на них (два CTE)**

```sql
WITH chat1_msgs AS (
    SELECT message_id, text
    FROM messages
    WHERE chat_id = 1
),
reactions_count AS (
    SELECT message_id, COUNT(*) AS cnt
    FROM reactions
    GROUP BY message_id
)
SELECT c.text, rc.cnt
FROM chat1_msgs c
JOIN reactions_count rc ON rc.message_id = c.message_id;
```


## 2. UNION, INTERSECT, EXCEPT

**2.1. UNION: имена пользователей и названия чатов**

```sql
SELECT username AS name FROM users
UNION
SELECT title FROM chats;
```


**2.2. INTERSECT: пользователи, которые и пишут сообщения, и состоят в чатах**

```sql
SELECT user_id FROM messages
INTERSECT
SELECT user_id FROM chat_members;
```


**2.3. EXCEPT: чаты, в которых нет сообщений**

```sql
SELECT chat_id FROM chats
EXCEPT
SELECT chat_id FROM messages;
```


## 3. Оконные функции

### 3.1. PARTITION BY

**Количество сообщений в чате рядом с каждым сообщением**

```sql
SELECT message_id, chat_id, user_id,
       COUNT(*) OVER (PARTITION BY chat_id) AS chat_messages
FROM messages;
```


**Сколько пользователей с таким же статусом**

```sql
SELECT username, status,
       COUNT(*) OVER (PARTITION BY status) AS users_in_status
FROM users;
```

### 3.2. PARTITION BY + ORDER BY

**Номер сообщения в чате по времени**

```sql
SELECT chat_id, message_id, sent_at,
       COUNT(*) OVER (PARTITION BY chat_id ORDER BY sent_at) AS msg_number
FROM messages;
```


**Сколько символов написал пользователь с нарастающим итогом**

```sql
SELECT user_id, sent_at, LENGTH(text) AS len,
       SUM(LENGTH(text)) OVER (PARTITION BY user_id ORDER BY sent_at) AS total_chars
FROM messages;
```


### 3.3. ROWS

**Средняя длина текущего и предыдущего сообщения**

```sql
SELECT message_id, LENGTH(text) AS len,
       AVG(LENGTH(text)) OVER (
           ORDER BY sent_at
           ROWS BETWEEN 1 PRECEDING AND CURRENT ROW
       ) AS avg_len
FROM messages;
```

**Сколько сообщений в окне из текущего и двух следующих**

```sql
SELECT message_id, sent_at,
       COUNT(*) OVER (
           ORDER BY sent_at
           ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING
       ) AS next_msgs
FROM messages;
```


### 3.4. RANGE

**Сколько сообщений было за последние 10 минут**

```sql
SELECT message_id, sent_at,
       COUNT(*) OVER (
           ORDER BY sent_at
           RANGE BETWEEN INTERVAL '10 minutes' PRECEDING AND CURRENT ROW
       ) AS msgs_last_10min
FROM messages;
```


**Сколько сообщений отправлено к этому моменту**

```sql
SELECT message_id, sent_at,
       COUNT(*) OVER (
           ORDER BY sent_at
           RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS total_so_far
FROM messages;
```


### 3.5. Ранжирующие функции

**ROW_NUMBER: номер сообщения в чате**

```sql
SELECT chat_id, message_id, sent_at,
       ROW_NUMBER() OVER (PARTITION BY chat_id ORDER BY sent_at) AS row_num
FROM messages;
```

**RANK: место пользователя по количеству сообщений (с пропусками)**

```sql
SELECT user_id, COUNT(*) AS msg_count,
       RANK() OVER (ORDER BY COUNT(*) DESC) AS rank
FROM messages
GROUP BY user_id;
```


**DENSE_RANK: место пользователя по количеству сообщений (без пропусков)**

```sql
SELECT user_id, COUNT(*) AS msg_count,
       DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS dense_rank
FROM messages
GROUP BY user_id;
```


### 3.6. Функции смещения

**LAG: время предыдущего сообщения в чате**

```sql
SELECT chat_id, message_id, sent_at,
       LAG(sent_at) OVER (PARTITION BY chat_id ORDER BY sent_at) AS prev_sent
FROM messages;
```


**LEAD: текст следующего сообщения в чате**

```sql
SELECT chat_id, message_id, text,
       LEAD(text) OVER (PARTITION BY chat_id ORDER BY sent_at) AS next_text
FROM messages;
```


**FIRST_VALUE: первое сообщение в чате**

```sql
SELECT chat_id, message_id, text,
       FIRST_VALUE(text) OVER (PARTITION BY chat_id ORDER BY sent_at) AS first_text
FROM messages;
```


**LAST_VALUE: последнее сообщение в чате**

```sql
SELECT chat_id, message_id, text,
       LAST_VALUE(text) OVER (
           PARTITION BY chat_id
           ORDER BY sent_at
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS last_text
FROM messages;
```