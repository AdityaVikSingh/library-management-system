# Library Management System

## Introduction

This project is a Library Management System designed to manage and track books in a college library. It allows students to register, issue, and return books seamlessly. The backend follows a **Monolithic Architecture**, with its components and design explained below.

## Technologies and Dependencies

* [Maven](https://maven.apache.org/) – Dependency management tool.
* [Spring Boot](https://spring.io/projects/spring-boot) – Framework for building web applications and REST APIs with ease.
* [Spring Security](https://spring.io/projects/spring-security) – Provides authentication and authorization.
* [Spring Data JPA (Hibernate)](https://hibernate.org/) – Simplifies database interactions by avoiding manual SQL queries.
* [MySQL](https://www.mysql.com/) – Database used as the persistence layer.
* [Project Lombok](https://projectlombok.org/) – Minimizes boilerplate code in Java.

## Running the Application

**CLI**

```
git clone 
cd Library-Management-System
mvn package
java -jar target/Student-library-0.0.1-SNAPSHOT.jar
```

**IntelliJ/Eclipse**

1. Let Maven resolve dependencies.
2. Run the SpringBootApplication class.

## Backend Design

### Entities

The system models real-world entities:

1. **Student**: student\_id (PK), country, emailId, name, age, card\_id (FK)
2. **Card**: card\_id (PK), createdOn, updatedOn, status (ACTIVATED/DEACTIVATED)
3. **Book**: book\_id (PK), isAvailable, genre, author\_id (FK)
4. **Author**: author\_id (PK), country, name, emailId
5. **User**: user\_id (PK), role (STUDENT/ADMIN), username (emailId), password

### Relationships and ER Diagram

A **Transaction table** manages the N\:M relationship between **Card** and **Book**, with fields:

* transaction\_id (PK, internal use only)
* Randomly generated UUID (returned to clients)
* card\_id (FK), book\_id (FK)
* isIssue (true = issue, false = return)
* transactionStatus (SUCCESSFUL/PENDING/FAILED)
* date, fineAmount (applicable on returns, calculated in Transaction Service)

## Exposed Functionalities

### Student Controller

* CRUD APIs for managing students.

  * Example: `http://localhost:8080/student/createStudent` creates a Student, Card, and User entity with role **STUDENT**.
* Password management API (default credentials: `username=emailId`, `password=pass123` (bcrypt encoded)).

### Book and Author Controllers

* Standard CRUD APIs for Book and Author entities.

### Transaction Controller

#### Issue Book

**Endpoint:** `https://localhost:8080/issueBook?bookId=_&cardId=_`

Checks:

1. Card is activated.
2. Book is available.
3. Card has not exceeded issue limit.

Actions:

1. Mark book unavailable.
2. Map book to card.
3. Record transaction, return UUID to client.

#### Return Book

**Endpoint:** `https://localhost:8080/returnBook?bookId=_&cardId=_`

Checks:

1. Valid book\_id and card\_id.
2. Card is activated.

Actions:

1. Mark book available.
2. Unlink book from card.
3. Locate latest issue transaction for the given card and book, calculate fine.
4. Record return transaction and return UUID and fine to client.

## Security (Checkout Branch: Security)

Spring Security ensures authentication and authorization. Every API call validates cookies and ensures the username (emailId) matches the entity being modified.

Examples (all prefixed with `http://localhost:8080`):

* `/student/all` – List all students (**ADMIN**)
* `/student/findById` – Fetch student by ID (**ADMIN**)
* `/student/updateStudent` – Update student details (**STUDENT only**)
* `/student/changePassword` – Change password (**STUDENT only**)
* `/transaction/all` – List all transactions (**ADMIN**)
* `/transaction/issueBook` – Issue book (**STUDENT**)
---