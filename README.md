# 🎬 BookMyShow Clone - Full-Stack Movie Ticket Booking System

A full-stack movie ticket booking application inspired by BookMyShow, built with **Spring Boot 3, Java 21, PostgreSQL, React 19, and Vite**.

The project focuses on building a reliable and scalable booking system with **concurrency-safe seat booking, JWT authentication, database migrations, pagination, idempotent booking requests, and RESTful APIs**.

---

## 🚀 Features

### 🎟️ Movie & Show Management

* Browse movies with pagination
* Search movies by title
* Filter movies by genre and language
* View available shows
* View shows by movie and date
* Browse theaters and cities

### 💺 Concurrency-Safe Seat Booking

* Prevents double booking when multiple users select the same seat
* Per-show seat inventory using `show_seats`
* Pessimistic database locking
* Unique database constraints
* Transactional booking flow
* Idempotent booking requests using `Idempotency-Key`
* Returns `409 Conflict` when a seat is already booked

### 🔐 Authentication & Security

* User registration and login
* BCrypt password hashing
* JWT-based authentication
* Stateless Spring Security
* Protected booking APIs
* Passwords are never returned in API responses
* Configurable CORS allowlist

### 🗄️ Database

* PostgreSQL
* Flyway database migrations
* Database indexes for faster queries
* `pg_trgm` support for efficient text search
* HikariCP connection pooling
* Schema validation using Hibernate

### 🧪 Testing

* Booking concurrency tests
* Idempotency tests
* Regression testing for critical booking scenarios

### 🐳 Docker

* Dockerized Spring Boot application
* Non-root application user
* Application health check
* Environment-based configuration

---

## 🛠️ Tech Stack

### Backend

* Java 21
* Spring Boot 3
* Spring Security
* Spring Data JPA
* Hibernate
* REST APIs
* JWT
* BCrypt
* Flyway
* Maven

### Frontend

* React 19
* TypeScript
* Vite

### Database

* PostgreSQL

### Testing & Tools

* JUnit
* H2
* Docker
* Swagger / OpenAPI
* Git & GitHub

---

## 🏗️ Architecture

```text
                    ┌─────────────────────┐
                    │      React 19       │
                    │      Frontend       │
                    └──────────┬──────────┘
                               │
                               │ REST API
                               ▼
                    ┌─────────────────────┐
                    │   Spring Boot 3     │
                    │      Backend        │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
        Controllers         Services       Spring Security
              │                │                │
              └────────────────┼────────────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Spring Data JPA   │
                    │      Hibernate       │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │     PostgreSQL       │
                    └─────────────────────┘
```

---

## 🔥 Booking Concurrency Design

One of the main challenges in a movie booking system is preventing two users from booking the same seat at the same time.

### Problem

A simple check-then-book approach can cause race conditions:

```text
User A → Check Seat → AVAILABLE
User B → Check Seat → AVAILABLE

User A → BOOK
User B → BOOK ❌
```

This can result in double booking.

### Solution

This project uses a database-backed seat inventory:

```text
show_seats
-------------------------
show_id
seat_id
status
-------------------------
1       101      AVAILABLE
1       102      BOOKED
1       103      AVAILABLE
```

The booking transaction:

```text
POST /bookings
       │
       ▼
Check Idempotency-Key
       │
       ▼
Lock show_seats rows
       │
       ▼
Check seat availability
       │
       ▼
Mark seats as BOOKED
       │
       ▼
Create booking
       │
       ▼
Commit Transaction
```

The system uses:

* `PESSIMISTIC_WRITE` locking
* Database `UNIQUE` constraints
* Transactions
* Idempotency keys

### Concurrency Test

The booking system is tested using two concurrent requests for the same seat:

```text
2 concurrent requests
        │
        ▼
   Same seat
        │
   ┌────┴────┐
   ▼         ▼
Request A  Request B
   │         │
   ▼         ▼
  SUCCESS   409 Conflict
```

Result:

**Exactly one request succeeds and the other receives `409 Conflict`.**

---

## 🔐 Authentication Flow

```text
User
 │
 ├── Register
 │      ↓
 │   BCrypt Password
 │      ↓
 │   PostgreSQL
 │
 └── Login
        ↓
    Validate Credentials
        ↓
    Generate JWT
        ↓
    Access Token
        ↓
    Authorization Header
        ↓
    Bearer Token
        ↓
    JwtAuthFilter
        ↓
    Protected API
```

Example:

```http
Authorization: Bearer <access-token>
```

---

## 📡 API Endpoints

### Authentication

| Method | Endpoint              | Description           |
| ------ | --------------------- | --------------------- |
| POST   | `/api/users/register` | Register user         |
| POST   | `/api/users/login`    | Login and receive JWT |

### Movies

| Method | Endpoint                          | Description          |
| ------ | --------------------------------- | -------------------- |
| GET    | `/api/movies`                     | Get paginated movies |
| GET    | `/api/movies/{id}`                | Get movie by ID      |
| GET    | `/api/movies/search?title=`       | Search movies        |
| GET    | `/api/movies/genre/{genre}`       | Filter by genre      |
| GET    | `/api/movies/language/{language}` | Filter by language   |

### Shows

| Method | Endpoint                           | Description           |
| ------ | ---------------------------------- | --------------------- |
| GET    | `/api/shows`                       | Get shows             |
| GET    | `/api/shows/{id}`                  | Get show              |
| GET    | `/api/shows/movie/{id}`            | Get shows for a movie |
| GET    | `/api/shows/movie/{id}/date?date=` | Get shows by date     |

### Theaters & Cities

| Method | Endpoint                      | Description          |
| ------ | ----------------------------- | -------------------- |
| GET    | `/api/theaters`               | Get theaters         |
| GET    | `/api/theaters/{id}`          | Get theater          |
| GET    | `/api/theaters/city/{cityId}` | Get theaters by city |
| GET    | `/api/cities`                 | Get cities           |
| GET    | `/api/cities/{id}`            | Get city             |

### Bookings

| Method | Endpoint                                      | Description         |
| ------ | --------------------------------------------- | ------------------- |
| POST   | `/api/bookings`                               | Create booking      |
| GET    | `/api/bookings/{id}`                          | Get booking         |
| GET    | `/api/bookings/user/{uid}`                    | Get user's bookings |
| PUT    | `/api/bookings/{id}/cancel`                   | Cancel booking      |
| GET    | `/api/bookings/show/{showId}/available-seats` | Get available seats |

---

## ❌ Error Handling

The application uses **RFC 7807 ProblemDetail** for consistent API errors.

Example:

```json
{
  "type": "about:blank",
  "title": "Seat Already Booked",
  "status": 409,
  "detail": "Seat with id 101 is already booked",
  "timestamp": "2026-10-07T12:30:00Z"
}
```

Common responses:

```text
200 OK
201 Created
400 Bad Request
401 Unauthorized
404 Not Found
409 Conflict
500 Internal Server Error
```

---

## 📂 Project Structure

```text
BMS/
│
├── src/
│   ├── main/
│   │   ├── java/com/cfs/BMS/
│   │   │
│   │   ├── config/
│   │   ├── security/
│   │   ├── controller/
│   │   ├── service/
│   │   ├── repository/
│   │   ├── entity/
│   │   ├── enums/
│   │   ├── exception/
│   │   └── dto/
│   │
│   └── resources/
│       └── db/migration/
│           └── V1__baseline.sql
│
├── src/test/
│   └── BookingConcurrencyTest.java
│
├── frontend/
│   ├── src/
│   ├── package.json
│   └── vite.config.ts
│
├── Dockerfile
├── pom.xml
└── README.md
```

---

## ⚙️ Getting Started

### Prerequisites

Make sure you have:

* JDK 21
* Maven 3.9+
* PostgreSQL 15+
* Node.js 18+
* npm
* Docker (optional)

---

### 1. Clone Repository

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git

cd YOUR_REPOSITORY
```

---

### 2. Create PostgreSQL Database

```sql
CREATE DATABASE BMS;
```

---

### 3. Configure Environment Variables

```bash
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/BMS
SPRING_DATASOURCE_USERNAME=postgres
SPRING_DATASOURCE_PASSWORD=your_password

APP_JWT_SECRET=your-secure-secret-key

CORS_ALLOWED_ORIGINS=http://localhost:5173
```

> Never commit production passwords or JWT secrets to GitHub.

---

### 4. Run Backend

```bash
mvn spring-boot:run
```

Backend:

```text
http://localhost:8080
```

Swagger:

```text
http://localhost:8080/swagger-ui/index.html
```

Health:

```text
http://localhost:8080/actuator/health
```

Flyway automatically runs the database migrations.

---

### 5. Run Frontend

```bash
cd frontend

npm install

npm run dev
```

Frontend:

```text
http://localhost:5173
```

---

## 🐳 Run with Docker

Build the image:

```bash
docker build -t bookmyshow-clone .
```

Run:

```bash
docker run -p 8080:8080 \
  -e SPRING_DATASOURCE_URL=jdbc:postgresql://host.docker.internal:5432/BMS \
  -e SPRING_DATASOURCE_USERNAME=postgres \
  -e SPRING_DATASOURCE_PASSWORD=your_password \
  -e APP_JWT_SECRET=your-secure-secret-key \
  bookmyshow-clone
```

---

## 🧪 Running Tests

Run the concurrency test:

```bash
mvn test -Dtest=BookingConcurrencyTest
```

Build the application:

```bash
mvn package -DskipTests
```

The generated JAR will be available in:

```text
target/
```

---

## 📊 Database Design

Main entities:

```text
Users
  │
  └── Bookings
         │
         └── Booking Seats
                │
                └── Seats

Movies
  │
  └── Shows
         │
         ├── Screens
         │      └── Seats
         │
         └── Show Seats

Theaters
  │
  └── Screens
```

Important tables:

```text
users
movies
cities
theaters
screens
seats
shows
show_seats
bookings
booking_seats
```

The critical booking table is:

```text
show_seats
-------------------------
show_id
seat_id
status
```

with a unique constraint on:

```text
(show_id, seat_id)
```

---

## 🔮 Future Improvements

* Redis-based temporary seat holds
* 10-minute seat reservation timeout
* Payment gateway integration
* Kafka-based event processing
* Email/SMS booking notifications
* Refresh token rotation
* Role-based admin dashboard
* Rate limiting
* Distributed tracing
* Read replicas
* Advanced movie search using PostgreSQL full-text search
* Production deployment using Docker and cloud infrastructure

---

## 👨‍💻 Author

**Pradip Yadav**

B.E. Computer Engineering | 2026 Graduate

Interested in **Java, Spring Boot, Backend Development, REST APIs, Microservices, and Full-Stack Development**.

---

## ⭐ If You Like This Project

If you find this project useful, consider giving it a ⭐ on GitHub.
