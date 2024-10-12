
### Структура базы данных

#### 1. Таблица `customers`

**Описание**: Эта таблица хранит информацию о клиентах интернет-магазина. Она содержит уникальные идентификаторы клиентов, их имена и адреса электронной почты. Поля:
- `customer_id` – это первичный ключ (PK), уникальный для каждого клиента.
- `customer_name` – имя клиента, обязательное к заполнению.
- `email` – уникальный адрес электронной почты, используемый для связи с клиентом.

**SQL для создания таблицы**:
```sql
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE
);
```

---

#### 2. Таблица `categories`

**Описание**: Эта таблица хранит список категорий товаров. Каждая категория, например, "Электроника", "Одежда" или "Книги", имеет уникальный идентификатор и название.
- `category_id` – первичный ключ, уникальный идентификатор для каждой категории.
- `category_name` – название категории, обязательное к заполнению.

**SQL для создания таблицы**:
```sql
CREATE TABLE categories (
    category_id SERIAL PRIMARY KEY,
    category_name VARCHAR(255) NOT NULL
);
```

---

#### 3. Таблица `products`

**Описание**: Таблица `products` содержит информацию о товарах, продаваемых в интернет-магазине. Каждый товар относится к определённой категории и имеет свою цену.
- `product_id` – первичный ключ для каждого товара.
- `product_name` – название товара.
- `price` – цена товара в валюте.
- `category_id` – внешний ключ, указывающий на категорию, к которой относится товар.

**SQL для создания таблицы**:
```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(255) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    category_id INT REFERENCES categories(category_id)
);
```

---

#### 4. Таблица `orders`

**Описание**: Эта таблица хранит информацию о заказах, оформленных клиентами. Она содержит ссылки на клиентов и дату оформления заказа.
- `order_id` – уникальный идентификатор для каждого заказа.
- `customer_id` – внешний ключ, указывающий на клиента, который сделал заказ.
- `order_date` – дата заказа.

**SQL для создания таблицы**:
```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    order_date DATE NOT NULL
);
```

---

#### 5. Таблица `order_items`

**Описание**: Эта таблица хранит позиции каждого заказа, связывая товары и заказы. Одному заказу могут соответствовать несколько позиций.
- `order_item_id` – уникальный идентификатор для каждой позиции заказа.
- `order_id` – внешний ключ, ссылающийся на заказ.
- `product_id` – внешний ключ, ссылающийся на товар.
- `quantity` – количество единиц товара в заказе.

**SQL для создания таблицы**:
```sql
CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(order_id),
    product_id INT REFERENCES products(product_id),
    quantity INT NOT NULL
);
```

---

### Примеры SQL-запросов

#### 1. **Запрос для получения списка заказов с именами клиентов и товарами**

Этот запрос объединяет несколько таблиц, чтобы вывести список заказов с указанием клиента, даты заказа и товаров, которые были включены в этот заказ.

**SQL-запрос**:
```sql
SELECT c.customer_name, o.order_date, p.product_name, oi.quantity
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id;
```

**Описание**:
- `JOIN` связывает таблицы `orders`, `customers`, `order_items` и `products`, чтобы получить информацию о заказах, клиентах и товарах.
- Выводится имя клиента, дата заказа, название товара и количество единиц товара в заказе.

---

#### 2. **Подсчёт общей суммы заказа**

Этот запрос позволяет рассчитать общую сумму каждого заказа, умножив количество единиц товара на его цену.

**SQL-запрос**:
```sql
SELECT o.order_id, c.customer_name, SUM(oi.quantity * p.price) AS total_amount
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
GROUP BY o.order_id, c.customer_name;
```

**Описание**:
- `SUM(oi.quantity * p.price)` вычисляет общую стоимость товаров в заказе.
- Группировка по `order_id` и `customer_name` выводит сумму для каждого заказа.

---

#### 3. **Вывод клиентов, потративших более 1000 единиц**

Этот запрос отображает всех клиентов, которые сделали заказы на сумму более 1000 единиц валюты.

**SQL-запрос**:
```sql
SELECT c.customer_name, SUM(oi.quantity * p.price) AS total_spent
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
GROUP BY c.customer_name
HAVING SUM(oi.quantity * p.price) > 1000;
```

**Описание**:
- `HAVING SUM(oi.quantity * p.price) > 1000` фильтрует клиентов, которые потратили больше 1000.
- Группировка по клиентам отображает итоговую сумму для каждого клиента.

---

#### 4. **Топ-5 самых продаваемых товаров**

Этот запрос выводит пять самых популярных товаров по количеству проданных единиц.

**SQL-запрос**:
```sql
SELECT p.product_name, SUM(oi.quantity) AS total_quantity
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
GROUP BY p.product_name
ORDER BY total_quantity DESC
LIMIT 5;
```

**Описание**:
- `SUM(oi.quantity)` подсчитывает общее количество проданных единиц каждого товара.
- Ограничение до 5 строк выводит только самые популярные товары.

---

### Работа с транзакциями

Транзакции позволяют выполнить несколько операций в одной сессии так, что если хотя бы одна операция завершается с ошибкой, все изменения откатываются. Это гарантирует целостность данных.

#### Пример транзакции: добавление нового заказа и его деталей

```sql
BEGIN;

-- Добавление нового заказа клиента
INSERT INTO orders (customer_id, order_date) 
VALUES (1, CURRENT_DATE)
RETURNING order_id;

-- Добавление товаров в заказ
INSERT INTO order_items (order_id, product_id, quantity)
VALUES (3, 1, 1),  -- 1 смартфон
       (3, 2, 2);  -- 2 рубашки

COMMIT;
```

**Описание**:
- Транзакция начинается с `BEGIN`.
- Заказ добавляется в таблицу `orders`, а его идентификатор возвращается с помощью `RETURNING`.
- Позиции заказа добавляются в таблицу `order_items`.
- Если все операции проходят успешно, изменения фиксируются командой `COMMIT`.

