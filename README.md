# Aiti_Guru_test
Тестовое задание в компанию Aiti Guru

Иванов Никита Алексеевич
+7 985 474-67-72
TG: @ri4puss

## 1. Проектирование схемы БД
### ER диаграмма схемы БД:
![ER диаграмма схемы БД](https://i.imgur.com/c7ttzeZ.png)

### SQL запросы для создания схемы БД:
В таблице categories для реализации дерева категорий выбран метод хранения древовидных структур данных adjacency list - parent_id указывает на родителя, а для корня parent_id = NULL.
```SQL
-- Клиенты
CREATE TABLE clients (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  address TEXT
);

-- Категории
CREATE TABLE categories (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  parent_id INTEGER REFERENCES categories(id) ON DELETE CASCADE
);

-- Товары
CREATE TABLE items (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  quantity INTEGER NOT NULL DEFAULT 0,
  price NUMERIC(12,2) NOT NULL,
  category_id INTEGER REFERENCES categories(id) ON DELETE SET NULL
);

-- Заказы
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  client_id INTEGER REFERENCES clients(id) ON DELETE SET NULL,
  order_date TIMESTAMP WITH TIME ZONE DEFAULT now(),
  status TEXT
);

-- Позиции заказа
CREATE TABLE order_items (
  order_id INTEGER REFERENCES orders(id) ON DELETE CASCADE,
  item_id INTEGER REFERENCES items(id) ON DELETE SET NULL,
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  PRIMARY KEY (order_id, item_id)
);
```

## 2. SQL запросы
### 2.1. Получение информации о сумме товаров заказанных под каждого клиента (Наименование клиента, сумма)
```SQL
SELECT
  c.name AS client_name,
  COALESCE(SUM(oi.quantity * i.price), 0) AS total_sum
FROM clients c
LEFT JOIN orders o ON o.client_id = c.id
LEFT JOIN order_items oi ON oi.order_id = o.id
LEFT JOIN items i ON oi.item_id = i.id
GROUP BY c.id, c.name
ORDER BY c.name;
```

### 2.2. Найти количество дочерних элементов первого уровня вложенности для категорий номенклатуры

```SQL
SELECT
  parent.id,
  parent.name,
  COUNT(child.id) AS direct_children_count
FROM categories parent
LEFT JOIN categories child ON child.parent_id = parent.id
GROUP BY parent.id, parent.name
ORDER BY parent.id;
```
