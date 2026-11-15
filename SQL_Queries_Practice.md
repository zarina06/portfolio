# Практика SQL-запросов: Проверка базы данных интернет-магазина

**Инструменты и синтаксис:** PostgreSQL / MySQL (стандарт ANSI SQL)  
**Автор:** Зарина (Junior Manual QA)

---

## 📋 Структура тестовой базы данных (E-commerce)
Для проверки целостности данных и бизнес-логики используются три связанные таблицы:
* `products` (товары): `id`, `title` (название), `category` (категория), `price` (цена), `stock_fbs` (остаток на складе).
* `orders` (заказы): `id`, `user_id` (ID покупателя), `status` (статус заказа), `total_amount` (сумма), `created_at` (дата).
* `users` (пользователи): `id`, `name` (имя), `email`, `city` (город).

---

## 🔍 Примеры проверочных запросов

### 1. Выборка и фильтрация товаров (`SELECT`, `WHERE`, `ORDER BY`)
**Задача:** Проверить, что в категории «Домашний текстиль» отображаются товары дороже 1500 ₽ с ненулевым остатком на складе, отсортированные от дешёвых к дорогим.
```sql
SELECT id, title, price, stock_fbs
FROM products
WHERE category = 'Домашний текстиль'
  AND price > 1500
  AND stock_fbs > 0
ORDER BY price ASC;
```

### 2. Поиск заказов в определённом статусе за период (`BETWEEN`, `IN`)
**Задача:** Найти все оплаченные или доставленные заказы на сумму от 2000 до 5000 ₽ для сверки с отчётом в личном кабинете продавца.
```sql
SELECT id, user_id, status, total_amount, created_at
FROM orders
WHERE status IN ('paid', 'delivered')
  AND total_amount BETWEEN 2000 AND 5000;
```

### 3. Подсчёт статистики и группировка данных (`COUNT`, `AVG`, `GROUP BY`)
**Задача:** Подсчитать количество товаров и среднюю цену в каждой категории магазина, оставив только категории, где больше 5 позиций.
```sql
SELECT category,
       COUNT(id) AS items_count,
       ROUND(AVG(price), 2) AS avg_price
FROM products
GROUP BY category
HAVING COUNT(id) > 5;
```

### 4. Объединение таблиц для проверки данных покупателя (`INNER JOIN`)
**Задача:** Проверить, что после оформления заказа в базе корректно связались данные клиента и сумма его покупки.
```sql
SELECT o.id AS order_id,
       u.name AS customer_name,
       u.email,
       o.total_amount,
       o.status
FROM orders o
INNER JOIN users u ON o.user_id = u.id
WHERE o.status = 'new';
```

### 5. Поиск пользователей без заказов (`LEFT JOIN`, `IS NULL`)
**Задача:** Найти зарегистрированных клиентов, которые ещё не оформили ни одного заказа (для проверки триггерных рассылок).
```sql
SELECT u.id, u.name, u.email
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

