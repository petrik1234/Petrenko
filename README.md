# Курсовая работа: Разработка базы данных для овощного магазина.

## Примеры типовых SQL-запросов

---

![Kirill P](https://github.com/user-attachments/assets/0c02ff82-62d0-4dd0-89dc-8998a29eccae)

---

### 1. Получить список всех продуктов с их поставщиками и ценой за кг

```
SELECT 
    p.product_id,
    p.product_name,
    p.category,
    p.price_per_kg,
    s.supplier_name
FROM 
    Products p
LEFT JOIN 
    Suppliers s ON p.supplier_id = s.supplier_id
ORDER BY p.product_name;
```
**Описание:**
Запрос выводит все продукты с указанием их категории, цены за килограмм и имени поставщика. Помогает получить полный список ассортимента с информацией о поставщиках.

---

### 2. Получить продажи за последний месяц с деталями клиента и продукта

```
SELECT 
    s.sale_id,
    c.full_name AS customer_name,
    p.product_name,
    s.quantity_kg,
    s.total_price,
    s.sale_date
FROM 
    Sales s
JOIN 
    Customers c ON s.customer_id = c.customer_id
JOIN 
    Products p ON s.product_id = p.product_id
WHERE 
    s.sale_date >= DATE_SUB(CURDATE(), INTERVAL 1 MONTH)
ORDER BY s.sale_date DESC;
```
**Описание:**
Запрос выводит все продажи за последний месяц с информацией о покупателе, названии продукта, количестве и сумме.

---

### 3. Получить общее количество и сумму продаж по категориям продуктов

```
SELECT 
    p.category,
    SUM(s.quantity_kg) AS total_quantity,
    SUM(s.total_price) AS total_revenue
FROM 
    Sales s
JOIN 
    Products p ON s.product_id = p.product_id
GROUP BY 
    p.category
ORDER BY 
    total_revenue DESC;
```
**Описание:**
Этот запрос показывает статистику по категориям — сколько всего килограмм продано и сколько заработано.

---

### 4. Получить список сотрудников с их должностями и датой найма

```
SELECT 
    employee_id,
    full_name,
    position,
    hire_date,
    phone,
    email
FROM 
    Employees
ORDER BY hire_date;
```
**Описание:**
Выводит всех сотрудников с основной контактной информацией и датой их найма.

---

## Хранимые процедуры

### Процедура добавления нового клиента

```
CREATE PROCEDURE AddCustomer(
    IN in_full_name VARCHAR(100),
    IN in_phone VARCHAR(20),
    IN in_email VARCHAR(100)
)
BEGIN
    INSERT INTO Customers (full_name, phone, email, registration_date)
    VALUES (in_full_name, in_phone, in_email, CURDATE());
END;
```
**Описание:**
Процедура добавляет нового клиента в таблицу `Customers` с текущей датой регистрации.

---

### Процедура создания заказа продажи
```
CREATE PROCEDURE CreateSale(
    IN in_product_id INT,
    IN in_customer_id INT,
    IN in_quantity DECIMAL(10,2)
)
BEGIN
    DECLARE product_price DECIMAL(10,2);
    DECLARE stock_available DECIMAL(10,2);

    SELECT price_per_kg, stock_kg INTO product_price, stock_available
    FROM Products WHERE product_id = in_product_id;

    IF stock_available < in_quantity THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Недостаточно товара на складе';
    ELSE
        INSERT INTO Sales (product_id, customer_id, sale_date, quantity_kg, total_price)
        VALUES (in_product_id, in_customer_id, CURDATE(), in_quantity, in_quantity * product_price);

        UPDATE Products SET stock_kg = stock_kg - in_quantity WHERE product_id = in_product_id;
    END IF;
END;
```
**Описание:**
Процедура создает новую продажу, проверяя наличие товара на складе и обновляя остаток.

---

## Триггеры

### Обновление остатка товара после продажи

```
DELIMITER $$

CREATE TRIGGER AfterSaleInsert
AFTER INSERT ON Sales
FOR EACH ROW
BEGIN
    UPDATE Products
    SET stock_kg = stock_kg - NEW.quantity_kg
    WHERE product_id = NEW.product_id;
END $$

DELIMITER ;
```
**Описание:**
Этот триггер автоматически уменьшает количество товара на складе после добавления записи о продаже.

---

## Представления (Views)

### Информация о продажах с деталями клиента и продукта

```
CREATE OR REPLACE VIEW view_SalesDetails AS
SELECT
    s.sale_id,
    c.full_name AS customer_name,
    p.product_name,
    s.quantity_kg,
    s.total_price,
    s.sale_date
FROM Sales s
JOIN Customers c ON s.customer_id = c.customer_id
JOIN Products p ON s.product_id = p.product_id;
```
**Описание:**
Упрощает получение полной информации о продажах в одном объекте.
