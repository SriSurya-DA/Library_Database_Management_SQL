# "SQL for Library Systems: Designing and Managing Book Issuing, Returning, and Member Information"
![library](https://github.com/SriSurya-DA/Library_Database_Management_SQL/blob/main/library%20management%20system.jpg)

## Overview

Database: library_db

This project demonstrates the development and implementation of a Library Management System using SQL. It covers the creation and management of tables, execution of CRUD operations, and the application of advanced SQL queries. The primary objective is to highlight proficiency in database design, data manipulation, and complex query execution for efficient library management.

## Objective

- Demonstrate proficiency in database design, management, and querying
- Showcase the use of advanced SQL techniques for data manipulation and reporting
- Ensure smooth and efficient management of library operations through well-structured database queries

## Key Features

### Database Setup

Create a relational database library_db with the necessary tables:

- Branches for managing branch details
- employees to store employee information (including branch affiliation)
- members for library members
- issued_status to track books issued to members
- return_status to track the returned books

### CRUD Operations

Implement CRUD operations to manage data effectively

- **Create**: Add new records to each table
- **Read**: Retrieve records using various queries to display library data
- **Update**: Modify existing records as needed (e.g., updating member details or book status)
- **Delete**: Remove records (e.g., deleting a book or a member's account)

### CTAS (Create Table As Select)

Use CTAS to generate new tables from existing data based on specific query results, allowing for more efficient data management and reporting

## Project Structure

## 1.Database Setup

![ERD](https://github.com/SriSurya-DA/Library_Database_Management_SQL/blob/main/library_erd.png)

## 2. Table Creation

- Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
-- Library Management System Project
/* Total number of Tables -6

1. Branch
2. Employees
3. Books
4. Members
5. Issued_Status (Book Issued info)
6. Return Status
*/

-- Creating the Branch Table

Drop Table if exists branch;

create table branch (

	branch_id varchar(10) PRIMARY KEY,
	manager_id varchar(10),
	branch_address varchar(60),
	contact_number varchar(15)
	
);

-- Creating the Employees Table

Drop table if exists employees;

create table employees(

	emp_id varchar(10) PRIMARY KEY,
	emp_name varchar(25),
	position varchar(25),
	salary int,
	branch_id varchar(10)
);

ALTER TABLE employees
ALTER COLUMN salary TYPE float; 

-- Creating the Books Table

Drop table if exists books;

create table books (

	isbn varchar(25) PRIMARY KEY,
	book_title varchar(75),
	category varchar(20),
	rental_price float,
	status varchar(15),
	author varchar(35),
	publisher varchar(55)
	
);

-- Creating the Members Table

Drop table if exists members;

create table members(

	member_id varchar(10) PRIMARY KEY,
	member_name varchar(25),
	member_address varchar (75),
	reg_date DATE
	
);

-- Creating the Issued_Status Table

Drop table if exists issued_status;

create table issued_status(

	issued_id varchar(20) PRIMARY KEY,
	issued_member_id varchar(20), --Foreign Key
	issued_book_name varchar(85),
	issued_date DATE,
	issued_book_isbn varchar(30), --Foreign Key
	issued_emp_id varchar(10) --Foreign Key
	
	);

-- Creating the Return_Status Table

Drop table if exists return_status;

create table return_status(

	return_id varchar(10) PRIMARY KEY,
	issued_id varchar(10),
	return_book_name varchar(75),
	return_date DATE,
	return_book_isbn varchar(25)

);


--Foreign Key

Alter table issued_status
Add constraint fk_members
Foreign key (issued_member_id) --Adding foreign key
References members(member_id); -- In this "Members" table the added foreign key is an Primary key.


Alter table issued_status
Add constraint fk_books
Foreign key (issued_book_isbn) --Adding foreign key
References books(isbn); -- In this "Books" table the added foreign key is an Primary key.


Alter table issued_status
Add constraint fk_employees
Foreign key (issued_emp_id) --Adding foreign key
References employees(emp_id); -- In this "Employees" table the added foreign key is an Primary key.


Alter table employees
Add constraint fk_branch
Foreign key (branch_id) --Adding foreign key
References branch(branch_id); -- In this "Branch" table the added foreign key is an Primary key.


Alter table return_status
Add constraint fk_issued_status
Foreign key (issued_id) --Adding foreign key
References issued_status(issued_id); -- In this "Issued_Status" table the added foreign key is an Primary key.


-- Removing the values in retun_status where it is not in "Issued_status"
-- This is occurring due to foreign key

SELECT * 
FROM return_status
WHERE issued_id NOT IN (SELECT issued_id FROM issued_status);

DELETE FROM return_status
WHERE issued_id NOT IN (SELECT issued_id FROM issued_status);

-- Now again re-enabling the foreign key (Before its disabled to upload the table)

ALTER TABLE return_status
ADD CONSTRAINT fk_issued_status FOREIGN KEY (issued_id) REFERENCES issued_status(issued_id);

```

## 2. CRUD Operations

### Task 1. Create a New Book Record "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
-- Here the order of given value is correct, hence directly using an values:

insert into books values(

	'978-1-60129-456-2',
	'To Kill a Mockingbird',
	'Classic',
	 6.00,
	'yes',
	'Harper Lee',
	'J.B. Lippincott & Co.'
	
);
```

### Task 2: Update an Existing Member's Address

```sql
update members
set member_address = '543 Apple Park'
where member_id = 'C104';
```

### Task 3: Delete a Record from the Issued Status Table

- Objective: Delete the record with issued_id = 'IS121' from the issued_status table

```sql
Delete from issued_status
where issued_id = 'IS121';
```

### Task 4: Retrieve All Books Issued by a Specific Employee

- Objective: Select all books issued by the employee with emp_id = 'E101'

```sql
select *
from issued_status
where issued_emp_id = 'E101';

```


### Task 5: List Members Who Have Issued More Than One Book

- Objective: Use GROUP BY to find members who have issued more than one book

```sql
select count(s.issued_id) as Total_count, s.issued_emp_id
from issued_status s
group by s.issued_emp_id
Having count(s.issued_id) > 1
order by s.issued_emp_id asc;
```

## CTAS (Create Table As Select)

### Task 6: Create Summary Tables: Used CTAS to generate new tables based on query results each book and total book_issued_cnt**

```sql
Drop table if exists issued_cnt;
create table issued_cnt as
select b.isbn as Books_name, b.book_title, count(s.*) as Total_count_of_Issued_Books
from issued_status s
join books b on b.isbn = s.issued_book_isbn
group by 1,2
order by 1,2 asc;

-- To view the Created table

select *
from issued_cnt;
```

## Data Analysis & Findings

### Task 7: Retrieve All Books in a Specific Category

```sql
select *
from books
where category = 'Classic';
```

### Task 8: Find Total Rental Income by Category:

```sql
select b.category, sum(b.rental_price), count(*)
from books b
join issued_status ist on ist.issued_book_isbn = b.isbn
group by 1;
```

### Task-9: List Members Who Registered in the Last 180 Days

```sql
select *
from members
where reg_date >= current_date - Interval '180 days';
```

### Task-10: List Employees with Their Branch Manager's Name and their branch details

```sql
select e.emp_id, e.emp_name,e.position, e.salary, b.*, e2.emp_name as manager
from employees e
join branch b on e.branch_id = b.branch_id
join employees e2 on e2.emp_id = b.manager_id;
```

### Task 11. Create a Table of Books with Rental Price Above a Certain Threshold

```sql
create table expensive_books as

select *
from books
where rental_price >7.0;

-- TO view the created table

select *
from expensive_books;
```

### Task 12: Retrieve the List of Books Not Yet Returned

```sql
select *
from issued_status ist
left join return_status rs on rs.issued_id = ist.issued_id
where return_id is null;
```

## Reports

- **Database Schema:** Detailed table structures and relationships.
- **Data Analysis:** Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports:** Aggregated data on high-demand books and employee performance.
