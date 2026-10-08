## 1. SELECT

**1.1 кто сейчас онлайн**

```sql
SELECT user_id, username, email
FROM users
WHERE status = 'online';
```

просто берем юзеров со статусом online

**1.2 сообщения из первого чата, новые сверху**

```sql
SELECT message_id, user_id, text, sent_at
FROM messages
WHERE chat_id = 1
ORDER BY sent_at DESC;
```

---

## 2. SELECT + CASE

**2.1 статус пишем по-русски**

```sql
SELECT
    username,
    status,
    CASE
        WHEN status = 'online' THEN 'в сети'
        WHEN status = 'offline' THEN 'не в сети'
        ELSE 'хз'
    END AS status_text
FROM users;
```

**2.2 делим сообщения по длине**

```sql
SELECT
    message_id,
    text,
    LENGTH(text) AS len,
    CASE
        WHEN LENGTH(text) < 10 THEN 'короткое'
        WHEN LENGTH(text) BETWEEN 10 AND 20 THEN 'среднее'
        ELSE 'длинное'
    END AS size
FROM messages;
```

длину считаем через LENGTH

---

## 3. JOIN

### INNER JOIN
(только то что совпало в обеих таблицах)

**3.1.1 сообщения + кто написал**

```sql
SELECT m.message_id, u.username, m.text, m.sent_at
FROM messages m
INNER JOIN users u ON m.user_id = u.user_id;
```

**3.1.2 сообщения + название чата**

```sql
SELECT c.title, m.text, m.sent_at
FROM messages m
INNER JOIN chats c ON m.chat_id = c.chat_id
ORDER BY c.title;
```

чата "Новости" тут не будет, там сообщений нет

### LEFT JOIN
(все из левой таблицы, если справа нет пары - NULL)

**3.2.1 все чаты и сообщения в них**

```sql
SELECT c.title, m.text
FROM chats c
LEFT JOIN messages m ON c.chat_id = m.chat_id;
```

у "Новости" text будет NULL

**3.2.2 сколько сообщений у каждого юзера**

```sql
SELECT u.username, COUNT(m.message_id) AS messages_count
FROM users u
LEFT JOIN messages m ON u.user_id = m.user_id
GROUP BY u.username;
```

### RIGHT JOIN
(тоже самое только наоборот, все из правой)

**3.3.1 сообщения и все чаты**

```sql
SELECT m.text, c.title
FROM messages m
RIGHT JOIN chats c ON m.chat_id = c.chat_id;
```

по сути тоже самое что LEFT выше, просто таблицы местами поменяли

**3.3.2 участники чатов и все юзеры**

```sql
SELECT u.username, cm.chat_id, cm.role
FROM chat_members cm
RIGHT JOIN users u ON cm.user_id = u.user_id;
```

### CROSS JOIN
(каждая строка с каждой)

**3.4.1 все пары юзер - чат**

```sql
SELECT u.username, c.title
FROM users u
CROSS JOIN chats c;
```

4 * 4 = 16 строк должно получиться

**3.4.2 все пары юзер - роль**

```sql
SELECT u.username, r.role
FROM users u
CROSS JOIN (SELECT DISTINCT role FROM chat_members) r;
```

### FULL OUTER JOIN
(вообще все из обеих таблиц)

**3.5.1 чаты и участники**

```sql
SELECT c.title, cm.user_id, cm.role
FROM chats c
FULL OUTER JOIN chat_members cm ON c.chat_id = cm.chat_id;
```

"Новости" будет с NULL вместо участника

**3.5.2 юзеры и реакции**

```sql
SELECT u.username, r.reaction_id, r.emoji
FROM users u
FULL OUTER JOIN reactions r ON u.user_id = r.user_id;
```

реакций в таблице пока нет, поэтому у всех юзеров справа будет NULL
