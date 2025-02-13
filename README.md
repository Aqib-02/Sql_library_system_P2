# Library Management System using SQL Project --P2

## Project Overview

**Project Title**: Library Management System    
**Database**: `library_db`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.


## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status.

```sql
CREATE DATABASE library_db;

-- Create table "Branch"
DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
 branch_id VARCHAR(10) PRIMARY KEY,
 manager_id VARCHAR(10),
 branch_address VARCHAR(30),
 contact_no VARCHAR(15)
 );
 
-- Create table "Employee"
DROP TABLE IF EXISTS employees;
CREATE TABLE employees
(
            emp_id VARCHAR(10) PRIMARY KEY,
            emp_name VARCHAR(30),
            position VARCHAR(30),
            salary DECIMAL(10,2),
            branch_id VARCHAR(10),
            FOREIGN KEY (branch_id) REFERENCES  branch(branch_id)
);

-- Create table "Members"
DROP TABLE IF EXISTS members;
CREATE TABLE members
(
            member_id VARCHAR(10) PRIMARY KEY,
            member_name VARCHAR(30),
            member_address VARCHAR(30),
            reg_date DATE
);

-- Create table "Books"
DROP TABLE IF EXISTS books;
CREATE TABLE books
(
            isbn VARCHAR(50) PRIMARY KEY,
            book_title VARCHAR(80),
            category VARCHAR(30),
            rental_price DECIMAL(10,2),
            status VARCHAR(10),
            author VARCHAR(30),
            publisher VARCHAR(30)
);

-- Create table "IssueStatus"
DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
(
            issued_id VARCHAR(10) PRIMARY KEY,
            issued_member_id VARCHAR(30),
            issued_book_name VARCHAR(80),
            issued_date DATE,
            issued_book_isbn VARCHAR(50),
            issued_emp_id VARCHAR(10),
            FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
            FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id),
            FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn) 
);

-- Create table "ReturnStatus"
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
(
            return_id VARCHAR(10) PRIMARY KEY,
            issued_id VARCHAR(30),
            return_book_name VARCHAR(80),
            return_date DATE,
            return_book_isbn VARCHAR(50),
            FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);
```

### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');

SELECT * FROM books;
```
**Task 2: Update an Existing Member's Address**

```sql
UPDATE members
SET member_address = '125 Oak St'
WHERE member_id = 'C103';
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
DELETE FROM issued_status
WHERE   issued_id =   'IS121';
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101'
```


**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
SELECT 
issued_member_id,
COUNT(*) as total_books_issued
FROM issued_status
GROUP BY 1
HAVING COUNT(*)>1;
```

### 3. CTAS (Create Table As Select)

- **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```sql
CREATE TABLE book_issued_cnt AS 
SELECT 
 b.isbn,
 b.book_title,
 COUNT(ist.issued_id) AS total_books_issued
FROM issued_status AS ist
JOIN books AS b
ON ist.issued_book_isbn=b.isbn
GROUP BY 1,2;

SELECT *
FROM book_issued_cnt;
```


### 4. Data Analysis & Findings

The following SQL queries were used to address specific questions:

Task 7. **Retrieve All Books in a Specific Category**:

```sql
SELECT * FROM books
WHERE category = 'Classic';
```

8. **Task 8: Find Total Rental Income by Category**:

```sql
SELECT 
  b.category, 
  SUM(b.rental_price) AS total_rental_income,
  COUNT(*) AS number_of_books
FROM books AS b
JOIN issued_status AS ist
 ON b.isbn=ist.issued_book_isbn
GROUP BY 1;
```

9. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
SELECT
 e1.emp_id,
 e1.emp_name,
 e1.position,
 e1.salary,
 b.*,
 e2.emp_name AS manager
 FROM employees AS e1
 JOIN branch AS b
 ON e1.branch_id=b.branch_id
 JOIN employees AS e2
 ON e2.emp_id=b.manager_id;
```

Task 10. **Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
CREATE TABLE expensive_books AS
SELECT * FROM books
WHERE rental_price > 7.00;
```

Task 11: **Retrieve the List of Books Not Yet Returned**
```sql
SELECT 
  *
 FROM issued_status AS ist
 LEFT JOIN return_status AS ret
 ON ist.issued_id=ret.issued_id
 WHERE ret.return_id IS NULL;
```

## Advanced SQL Operations

**Task 12: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
SELECT 
  t1.member_id,
  t1.member_name,
  t3.book_title, 
  t2.issued_date,
  t4.return_date
  FROM members AS t1
  JOIN issued_status AS t2
   ON t1.member_id=t2.issued_member_id
  JOIN books AS t3
   ON t2.issued_book_isbn=t3.isbn
   LEFT JOIN return_status AS t4
    ON t2.issued_id=t4.issued_id
  WHERE t4.return_date IS NULL 
  ORDER BY 1;
```

**Task 13: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
CREATE TABLE branch_report AS
SELECT 
 e.branch_id,
 COUNT(i.issued_id) AS books_issued,
 COUNT(r.return_id) AS books_returned,
 SUM(b.rental_price) AS revenue_generated
FROM 
 employees AS e
JOIN issued_status AS i
 ON e.emp_id=i.issued_emp_id
LEFT JOIN return_status AS r
 ON i.issued_id=r.issued_id
JOIN books AS b
 ON b.isbn=i.issued_book_isbn
 GROUP BY e.branch_id;
 
 SELECT * FROM branch_report;
 
```

**Task 14: Find Employees with the Most Book Issues Processed**  
```sql
SELECT 
 e.emp_id,
 e.emp_name,
 e.branch_id,
 COUNT(i.issued_id) AS total_books_processed
FROM employees AS e
JOIN issued_status AS i
 ON e.emp_id=i.issued_emp_id
GROUP BY 1,2,3
ORDER BY 4 DESC 
LIMIT 5;
```


## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.


## Author - Aqib uddin


