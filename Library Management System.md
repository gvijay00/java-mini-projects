# **Library Management System**  

- This project builds a **Library Management System** using Java and a database, allowing users to manage books, members, librarians, and borrow/return operations while ensuring data integrity and security.  

## **Project Structure**  
```
com.example.librarymanagement/
├── dao/
│   ├── BookDAO.java
│   ├── MemberDAO.java
│   ├── LibrarianDAO.java
│   ├── BorrowDAO.java
├── model/
│   ├── Book.java
│   ├── Member.java
│   ├── Librarian.java
│   ├── Borrow.java
├── util/
│   └── DatabaseConnection.java
├── service/
│   ├── BookService.java
│   ├── MemberService.java
│   ├── LibrarianService.java
│   ├── BorrowService.java
├── exception/
│   └── DatabaseException.java
└── main/
    └── LibraryManagementApp.java
```

## **Database Schema with Relationships**  

```sql
-- Books table
CREATE TABLE books (
    book_id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(200) NOT NULL,
    author VARCHAR(100) NOT NULL,
    genre VARCHAR(100),
    isbn VARCHAR(20) UNIQUE NOT NULL,
    available_copies INT NOT NULL
);

-- Members table
CREATE TABLE members (
    member_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(15),
    join_date DATE
);

-- Librarians table
CREATE TABLE librarians (
    librarian_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    hire_date DATE
);

-- Borrow table (junction table for many-to-many relationship)
CREATE TABLE borrow (
    borrow_id INT PRIMARY KEY AUTO_INCREMENT,
    member_id INT NOT NULL,
    book_id INT NOT NULL,
    borrow_date DATE NOT NULL,
    return_date DATE,
    FOREIGN KEY (member_id) REFERENCES members(member_id),
    FOREIGN KEY (book_id) REFERENCES books(book_id),
    UNIQUE KEY (member_id, book_id) -- Prevents duplicate borrow records
);
```

## **Relationship Types Implemented**  

1. **One-to-Many**: Librarian to Books  
   - One librarian can manage multiple books  
   - Each book is managed by one librarian  

2. **Many-to-Many**: Members to Books (through Borrow Table)  
   - Members can borrow multiple books  
   - Books can be borrowed by multiple members  
   - The Borrow table serves as a junction table  

## **Updated Package Details**  

### **1. Model Classes**  
- **Book.java**: Stores book details including author, genre, and available copies  
- **Member.java**: Represents library members with contact information  
- **Librarian.java**: Manages library operations  
- **Borrow.java**: Tracks borrowing transactions including borrow and return dates  

### **2. DAO Layer**  
- **BookDAO.java**: CRUD operations for book data  
- **MemberDAO.java**: CRUD operations for member data  
- **LibrarianDAO.java**: CRUD operations for librarian data  
- **BorrowDAO.java**: Manages borrow/return operations, including:  
  - `borrowBook(int memberId, int bookId)`  
  - `returnBook(int memberId, int bookId)`  
  - `getBorrowedBooksForMember(int memberId)`  
  - `getMembersWithBorrowedBooks(int bookId)`  

### **3. Service Layer**  
- **BookService.java**: Handles book availability and catalog management  
- **MemberService.java**: Manages member registration and borrowing rules  
- **LibrarianService.java**: Manages librarian-related operations  
- **BorrowService.java**: Enforces borrowing policies, including:  
  - Validates member and book existence before borrowing  
  - Prevents duplicate borrow entries  
  - Updates book availability upon borrow/return  

## **Implementation Details for Relationships**  

### **Join Operations**  
- Implements SQL JOIN operations to fetch related data:  
  ```java
  // Example methods in DAO classes
  List<Book> getBooksWithBorrowers();
  List<Member> getMembersWithBorrowedBooks();
  ```

### **Cascading Operations**  
- Implements cascading delete for certain relationships:  
  - When deleting a book, ensure existing borrow records are handled  
  - When deleting a member, manage their borrow history  

### **Transaction Management**  
- Ensures data integrity across related tables:  
  ```java
  // Example borrowing transaction
  connection.setAutoCommit(false);
  try {
      // Insert borrow record
      // Update book availability
      connection.commit();
  } catch (SQLException e) {
      connection.rollback();
      throw new DatabaseException("Borrow transaction failed", e);
  } finally {
      connection.setAutoCommit(true);
  }
  ```

### **Reporting Queries**  

- Implementation of complex queries spanning multiple tables: 
  - List books with the highest borrow count  
  - Find overdue books and their borrowers  
  - Track the most active members