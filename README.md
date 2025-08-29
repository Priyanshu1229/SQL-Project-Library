# Library Management System using SQL Project
Project Title: Library Management System  

Level: Intermediate  

 This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

![LIBRARY IMAGE](https://github.com/Priyanshu1229/SQL-Project-Library/blob/main/library.jpg)

Objectives
Set up the Library Management System Database: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
CRUD Operations: Perform Create, Read, Update, and Delete operations on the data.
CTAS (Create Table As Select): Utilize CTAS to create new tables based on query results.
Advanced SQL Queries: Develop complex queries to analyze and retrieve specific data.
Project Structure
1. Database Setup
   
![ERD IMAGE](https://github.com/Priyanshu1229/SQL-Project-Library/blob/main/library1.png)
Database Creation: Created a database named library_db.
```sql
Table Creation: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.
CREATE DATABASE library_db;

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
2. CRUD Operations
Create: Inserted sample records into the books table.
Read: Retrieved and displayed data from various tables.
Update: Updated records in the employees table.
Delete: Removed records from the members table as needed.

SELECT * FROM employee;
SELECT * FROM books;
SELECT * FROM members;
SELECT * FROM return_status;
SELECT * FROM issued_status;
SELECT * FROM branch;

USE library;

-- Task 1: Create a New Book Record
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES
('978-1-60129-456-2','to kill a mocking bird','classic','6.00','yes','Harper Lee','J.B. Lippincott & Co.');

SELECT * FROM books;

-- Task 2: Update an Existing Member's Address
UPDATE members
SET member_address ='125 main st'
WHERE member_id='C101';
SELECT * FROM members;

-- Task 3: Delete a Record from the Issued Status Table
DELETE FROM issued_status
WHERE issued_id='IS107';

-- Task 2: Update an Existing Member's Address
update members
set member_address ='125 main st'
where member_id='C101';
SELECT *FROM MEMBERS;


-- Task 3: Delete a Record from the Issued Status Table
delete from  issued_status
where issued_id='IS107';

-- Task 4: Retrieve All Books Issued by a Specific Employee
SELECT *FROM ISSUED_STATUS
where issued_emp_id='E101';

-- Task 5: List Members Who Have Issued More Than One Book

select  issued_emp_id,
count(issued_id) as total_book_issued
from issued_status
Group  by Issued_emp_id
having count(issued_id)>1;

-------------------------------------------------
-- CTAS (Create Table As Select)
-------------------------------------------------

-- Task 6: Create Summary Table (Book Issue Count)
CREATE TABLE book_counts AS
SELECT 
    b.isbn,
    b.book_title,
    COUNT(ist.issued_id) AS no_issued
FROM books AS b
JOIN issued_status AS ist
    ON ist.issued_book_isbn = b.isbn
GROUP BY b.isbn, b.book_title;
SELECT*FROM BOOK_COUNTS;


-------------------------------------------------
-- DATA ANALYSIS & FINDINGS
-------------------------------------------------

-- Task 7: Retrieve All Books in a Specific Category

SELECT *FROM BOOKS
WHERE category ='Classic';
-- Task 8: Find Total Rental Income by Category

SELECT  
    b.category,
    SUM(b.rental_price) AS total_rental_income,
    COUNT(*) AS total_issues
FROM books AS b
JOIN issued_status AS ist
    ON ist.issued_book_isbn = b.isbn
GROUP BY b.category;

-- Task 9: List Members Who Registered in the Last 180 Days

SELECT * 
FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL 180 DAY;


INSERT INTO members (member_id, member_name, member_address, reg_date)
VALUES
('C120','SAM','145 MAIN ST','2025-06-01'),
('C121','PAM','136 MAIN ST','2025-06-05');

-- Task 10: List Employees with Their Branch Manager's Name and Branch Details

SELECT e1.*,
 e2.emp_name as manager,
 b1.branch_id
 from employees as e1
join branch as b1
on  b1.branch_id=e1.branch_id
join 
employees as e2
on b1.manager_id=e2.emp_id;



-- Task 11: Create Table of Books with Rental Price Above Threshold like 7 dollar
create table book_price_greater_than_7
as
select *from books
where rental_price>7;
 

-- Task 12: Retrieve Books Not Yet Returned

select distinct ist.issued_book_name
 from issued_status as ist
left join 
return_status AS rs
ON ist.issued_id =rs.issued_id
where rs.return_id IS null;


-------------------------------------------------
-- ADVANCED SQL OPERATIONS
-------------------------------------------------

-- Task 13: Identify Members with Overdue Books (30+ days)
-- display the  member id, members  name issue data and days overdue
 
 -- issued status ==members ==book==return status
-- filter book which is return
 -- overdue>30
 
SELECT 
    ist.issued_member_id,
    m.member_name,
    bk.book_title,
    ist.issued_date,
    rs.return_date,
    CURRENT_DATE - ist.issued_date AS over_due_days
FROM issued_status AS ist
JOIN members AS m
    ON m.member_id = ist.issued_member_id
JOIN books AS bk
    ON bk.isbn = ist.issued_book_isbn
LEFT JOIN return_status AS rs
    ON rs.issued_id = ist.issued_id
WHERE rs.return_date IS NULL
  AND (CURRENT_DATE - ist.issued_date) > 30
  order by 1;


-- Task 14: Update Book Status on Return
-- write  a query to update the status of books in the books table to yes ( when they are returened (based on entries in the 
-- return_status table

select *from issued_status
where issued_book_isbn ='978-0-451-52994-2';

select *from books
where isbn ='978-0-451-52994-2';

update  books
set status ='no'
where  isbn='978-0-451-52994-2';
select *from return_status
where issued_id= 'IS130';

INSERT INto return_status(return_id,issued_id,return_date,book_quality)
values
('RS125','IS130',CURRENT_DATE,'GOOD');

update  books
set status ='YES'
where  isbn='978-0-451-52994-2';

--  STORE PROCEDURES
DELIMITER $$

CREATE PROCEDURE add_return_record(
    IN p_return_id VARCHAR(10),
    IN p_issued_id VARCHAR(10),
    IN p_book_quality VARCHAR(15)
)
BEGIN
    DECLARE v_isbn VARCHAR(50);
    DECLARE v_book_name VARCHAR(80);

    -- Insert into return_status
    INSERT INTO return_status(return_id, issued_id, return_date, book_quality)
    VALUES (p_return_id, p_issued_id, CURDATE(), p_book_quality);

    -- Get ISBN and Book Title from issued_status + books
    SELECT ist.issued_book_isbn, bk.book_title
    INTO v_isbn, v_book_name
    FROM issued_status ist
    JOIN books bk ON bk.isbn = ist.issued_book_isbn
    WHERE ist.issued_id = p_issued_id
    LIMIT 1;

    -- Update book status
    UPDATE books
    SET status = 'YES'
    WHERE isbn = v_isbn;

    -- Instead of RAISE NOTICE (not in MySQL), return a message
    SELECT CONCAT('Thank you for returning the book: ', v_book_name) AS message;
END$$

DELIMITER ;



-- issued_id =IS135
-- ISBN =WHERE ISBN='978-0-307-58837-1'

-- TESTING FUNCTION & ADD_RETURN_RECORDS
USE LIBRARY;
SELECT *FROM
BOOKs
WHERE isbn= '978-0-307-58837-1';

SELECT *FROM
issued_status
WHERE issued_book_isbn= '978-0-307-58837-1';

SELECT *FROM
return_status
WHERE issued_id = 'IS135';
 CALL add_return_record('RS138','IS135','GOOD');

-- Task 15: Branch Performance Report
-- create a query that generates a performacne report for each  branch showing the number off books issued , \
-- the number of books returned and the total revnue generated from book rentals
use library;
select  *from branch;
select *from issued_status;
select * from employees;
select *from books;
select *from return_status;


create table branch_report
as 
select  
b.branch_id,
b.manager_id,
count(ist.issued_id)as number_book_issued,
count(rs.return_id)as number_of_book_return,
sum(bk.rental_price)as total_revenue
from issued_status as  ist 
join 
employees as e
on e.emp_id=ist.issued_emp_id
join 
branch as b
on e.branch_id=b.branch_id
left join
 return_status as rs
on rs.issued_id=ist.issued_id
join 
books as bk
on ist.issued_book_isbn = bk.isbn
group by 1,2;
select *from branch_report;
-- Task 16: CTAS - Active Members (last 6 months)
-- use the create table as  CTAS statment to create a new table active_members contain members who
-- have isuued at least one book in the last six month

create table active_members
as
SELECT *FROM MEMBERS
WHERE  MEMBER_ID in(select  
distinct
issued_member_id
from issued_status
where  issued_date >= current_date -interval 6 month);

select *from active_members;

-- Task 17: Employees with Most Book Issues

-- write a query to to find  top 3 employee who have processed the mosty book issue .
-- display the empolyee name,number of book processed and their branch

  select
  e.emp_name,
  b.*,
  count(ist.issued_id) as no_book_issued
  
  from  issued_status as ist 
  join 
  employees as e
  on e.emp_id=ist.issued_emp_id
  join branch as b
  on e.branch_id=b.branch_id
  group by 1,2;
  
-- Task 18: Members Issuing High-Risk (Damaged) Books
-- Write a query to identify members who have issued books
--  more than twice with the status 0"damaged" in the books table.
 --  Display the member name, book title, and the number oftimes they've issued damaged books.

SELECT m.member_name,
       b.book_title,
       COUNT(*) AS damaged_count
FROM issued_status AS ist
JOIN members AS m
  ON m.member_id = ist.issued_member_id
JOIN books AS b
  ON b.isbn = ist.issued_book_isbn
JOIN return_status AS rs
  ON rs.issued_id = ist.issued_id
WHERE rs.book_quality = 'Damaged'
GROUP BY m.member_name, b.book_title
HAVING COUNT(*) >= 2;

use library;


-- Task 19: Stored Procedure - Update Book Status
-- Objective: Create a stored procedure to manage the status of books in a library system. Description:
-- Write a stored procedure that updates the status of a book in the library based on its issuance. 
--   The procedure should function as follows: The stored procedure should take the book_id as an input parameter.
--  The procedure should first check if the book is available (status = 'yes'). If the book is available, 
--  it should be issued, and the status in the books table should be updated to 'no'. If the book is not available (status = 'no'), 
--  the procedure should return an error message indicating that the book is currently not available.

select *from books;
select *from issued_status; 

DELIMITER $$

CREATE PROCEDURE issue_book(
    IN p_issue_id VARCHAR(10),
    IN p_issue_member_id VARCHAR(30),
    IN p_issue_book_isbn VARCHAR(50),
    IN p_issued_emp_id VARCHAR(10)
)
BEGIN
    DECLARE v_status VARCHAR(10);
    DECLARE v_error_message VARCHAR(255);

    -- check if the book is available
    SELECT status
    INTO v_status
    FROM books
    WHERE isbn = p_issue_book_isbn;

    -- If available, issue it
    IF v_status = 'yes' THEN
        INSERT INTO issued_status (issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
        VALUES (p_issue_id, p_issue_member_id, CURRENT_DATE, p_issue_book_isbn, p_issued_emp_id);

        UPDATE books
        SET status = 'no'
        WHERE isbn = p_issue_book_isbn;

        SELECT CONCAT('Book record added successfully for book ISBN: ', p_issue_book_isbn) AS message;

    ELSE
        SET v_error_message = CONCAT('Error: Book is currently not available. ISBN: ', p_issue_book_isbn);
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = v_error_message;
    END IF;
END$$

DELIMITER ;



select *from books;
-- "978-0-553-29698-2"  --yes 
-- "978-0-375-41398-8"  -- no 
select *from issued_status;
call issue_book('IS155','C108','978-0-553-29698-2','E104');
call issue_book('IS156','C108','978-0-375-41398-8','E104');


SELECT *FROM BOOKS  
WHERE ISBN ='978-0-375-41398-8';
-- Task 20: CTAS - Overdue Books with Fines
-- Description: Write a CTAS query to create a new table that lists each member and the books
--  they have issued but not returned within 30 days. The table should include: 
-- The number of overdue books. The total fines, with each day's fine calculated at $0.50.
 -- The number of books issued by each member.
-- The resulting table should show: Member ID Number of overdue books Total fines 


CREATE TABLE overdue_fines AS
SELECT 
    ist.issued_member_id AS member_id,
    COUNT(CASE 
             WHEN rs.return_date IS NULL 
                  AND DATEDIFF(CURDATE(), ist.issued_date) > 30 
             THEN 1 
         END) AS overdue_books,
    SUM(CASE 
            WHEN rs.return_date IS NULL 
                 AND DATEDIFF(CURDATE(), ist.issued_date) > 30 
            THEN (DATEDIFF(CURDATE(), ist.issued_date) - 30) * 0.50
            ELSE 0
        END) AS total_fine,
    COUNT(*) AS total_books_issued
FROM issued_status AS ist
LEFT JOIN return_status AS rs 
       ON ist.issued_id = rs.issued_id
GROUP BY ist.issued_member_id;

SELECT *FROM OVERDUES_FINES;


Reports
Database Schema: Detailed table structures and relationships.
Data Analysis: Insights into book categories, employee salaries, member registration trends, and issued books.
Summary Reports: Aggregated data on high-demand books and employee performance.
Conclusion
This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.
