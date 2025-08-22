# PROJECT-TASK
Personal Finance Tracker

Mini Guide

Create Schema

Tables:

Users (user_id, name, email)

Income (income_id, user_id, amount, date, source)

Expenses (expense_id, user_id, amount, date, category_id)

Categories (category_id, category_name)

Insert Sample Data

Add a few users.

Insert income transactions (salary, freelancing, etc.).

Insert expense transactions (food, travel, shopping).

Insert categories like Food, Rent, Travel, Shopping.

Summarize Monthly Expenses

SELECT user_id, 
       strftime('%Y-%m', date) AS month, 
       SUM(amount) AS total_expenses
FROM Expenses
GROUP BY user_id, month;


Category-Wise Spending

SELECT u.name, c.category_name, SUM(e.amount) AS total_spent
FROM Expenses e
JOIN Users u ON e.user_id = u.user_id
JOIN Categories c ON e.category_id = c.category_id
GROUP BY u.name, c.category_name;


Create Views for Balance Tracking

CREATE VIEW user_balance AS
SELECT u.user_id, u.name,
       (SELECT SUM(amount) FROM Income WHERE user_id = u.user_id) -
       (SELECT SUM(amount) FROM Expenses WHERE user_id = u.user_id) AS balance
FROM Users u;


Export Monthly Reports

Use SELECT ... INTO OUTFILE in MySQL (or .mode csv in SQLite) to export.

Example:

SELECT * FROM user_balance 
INTO OUTFILE '/tmp/monthly_report.csv' 
FIELDS TERMINATED BY ',' 
LINES TERMINATED BY '\n';







-- ================================
-- 1. Create Database (optional)
-- ================================
CREATE DATABASE personal_finance;
USE personal_finance;

-- ================================
-- 2. Create Tables
-- ================================
CREATE TABLE Users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);

CREATE TABLE Categories (
    category_id INT AUTO_INCREMENT PRIMARY KEY,
    category_name VARCHAR(50)
);

CREATE TABLE Income (
    income_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    amount DECIMAL(10,2),
    date DATE,
    source VARCHAR(50),
    FOREIGN KEY (user_id) REFERENCES Users(user_id)
);

CREATE TABLE Expenses (
    expense_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    amount DECIMAL(10,2),
    date DATE,
    category_id INT,
    FOREIGN KEY (user_id) REFERENCES Users(user_id),
    FOREIGN KEY (category_id) REFERENCES Categories(category_id)
);

-- ================================
-- 3. Insert Dummy Data
-- ================================

-- Users
INSERT INTO Users (name, email) VALUES
('Alice', 'alice@example.com'),
('Bob', 'bob@example.com');

-- Categories
INSERT INTO Categories (category_name) VALUES
('Food'), ('Rent'), ('Travel'), ('Shopping');

-- Income
INSERT INTO Income (user_id, amount, date, source) VALUES
(1, 50000, '2025-08-01', 'Salary'),
(1, 5000,  '2025-08-10', 'Freelance'),
(2, 40000, '2025-08-01', 'Salary');

-- Expenses
INSERT INTO Expenses (user_id, amount, date, category_id) VALUES
(1, 8000,  '2025-08-02', 2),  -- Rent
(1, 2000,  '2025-08-05', 1),  -- Food
(1, 1500,  '2025-08-07', 4),  -- Shopping
(1, 3000,  '2025-08-12', 3),  -- Travel
(2, 10000, '2025-08-03', 2),  -- Rent
(2, 2500,  '2025-08-06', 1);  -- Food

-- ================================
-- 4. Monthly Expense Summary
-- ================================
SELECT user_id,
       DATE_FORMAT(date, '%Y-%m') AS month,
       SUM(amount) AS total_expenses
FROM Expenses
GROUP BY user_id, month;

-- ================================
-- 5. Category-wise Spending
-- ================================
SELECT u.name, c.category_name, SUM(e.amount) AS total_spent
FROM Expenses e
JOIN Users u ON e.user_id = u.user_id
JOIN Categories c ON e.category_id = c.category_id
GROUP BY u.name, c.category_name;

-- ================================
-- 6. Balance Tracking View
-- ================================
CREATE VIEW user_balance AS
SELECT u.user_id, u.name,
       (SELECT IFNULL(SUM(amount),0) FROM Income WHERE user_id = u.user_id) -
       (SELECT IFNULL(SUM(amount),0) FROM Expenses WHERE user_id = u.user_id) AS balance
FROM Users u;

-- View Data
SELECT * FROM user_balance;

-- ================================
-- 7. Export Report (MySQL only)
-- ================================
-- Note: Change file path as needed
-- This won’t work in SQLite
/*
SELECT * FROM user_balance 
INTO OUTFILE '/tmp/monthly_report.csv' 
FIELDS TERMINATED BY ',' 
LINES TERMINATED BY '\n';
*/

