# Final Project: DeOroAtelier Store Management System
## Modernization with Spring Boot, JPA, and Database Persistence

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Learning Objectives](#learning-objectives)
3. [Domain Requirements](#domain-requirements)
4. [Technical Architecture Requirements](#technical-architecture-requirements)
5. [JPA and Database Design](#jpa-and-database-design)
6. [REST API Endpoints](#rest-api-endpoints)
7. [Validation Requirements](#validation-requirements)
8. [Exception Handling](#exception-handling)
9. [Docker Compose Requirements](#docker-compose-requirements)
10. [Business Rules Implementation](#business-rules-implementation)
11. [Submission Requirements](#submission-requirements)

---

## Project Overview

### Context

In **Project 1**, you developed the DeOroAtelier Store (DOA Store) management system using pure Java with CSV file persistence. The system managed employees, jewelry inventory, customers, orders, and payments using object-oriented principles and file-based data storage.

### Project Goal

This final project requires you to **modernize the DOA Store system** by migrating from CSV-based storage to a professional, production-ready architecture using:

- **Spring Boot Framework** for rapid application development
- **Spring Data JPA** for database persistence and object-relational mapping
- **MySQL or PostgreSQL** as the relational database management system
- **Docker Compose** for containerized database deployment
- **RESTful API** for all operations, replacing console-based interaction
- **Layered Architecture** with clear separation of concerns

### Key Differences from Project 1

| Aspect | Project 1 | Final Project |
|--------|-----------|---------------|
| **Persistence** | CSV files | MySQL/PostgreSQL via JPA |
| **Architecture** | Monolithic with manager classes | Layered (Controller-Service-Repository) |
| **Interface** | Console-based | REST API with JSON |
| **Data Access** | Manual file I/O | Spring Data JPA repositories |
| **Deployment** | Local execution | Docker Compose for database |
| **Validation** | Manual in code | Jakarta Bean Validation annotations |
| **Error Handling** | Return codes/messages | Global exception handler with HTTP status codes |
| **Testing** | Manual console testing | API testing with Postman/cURL |

### Scope

You must implement the **exact same domain model and business rules** from Project 1, but using modern Spring Boot technologies and database persistence. All functionality must be accessible through REST API endpoints.

---

## Learning Objectives

Upon completion of this final project, students will demonstrate ability to:

1. **Apply Spring Boot Framework:**
   - Configure Spring Boot applications with proper dependencies
   - Understand Spring's dependency injection and IoC container
   - Implement RESTful services using Spring MVC

2. **Implement JPA and Database Persistence:**
   - Design JPA entity mappings with proper annotations
   - Implement inheritance strategies in JPA
   - Define entity relationships (one-to-many, many-to-one, many-to-many)
   - Use Spring Data JPA repositories for data access

3. **Design Layered Architecture:**
   - Implement Controller-Service-Repository pattern
   - Apply separation of concerns across layers
   - Use DTOs for API communication

4. **Develop RESTful APIs:**
   - Design REST endpoints following best practices
   - Implement CRUD operations via HTTP methods
   - Return appropriate HTTP status codes
   - Handle request/response with JSON

5. **Apply Validation and Error Handling:**
   - Use Jakarta Bean Validation annotations
   - Implement global exception handling with @ControllerAdvice
   - Return structured error responses

6. **Work with Docker:**
   - Write docker-compose.yml for database provisioning
   - Configure database connections and environment variables
   - Understand containerization benefits

---

## Domain Requirements

### Important Note

**All entities, attributes, relationships, and business rules from Project 1 must be preserved.** This section provides a complete reference, but you should refer to your Project 1 documentation for detailed business rules.

---

### 1. Employee Entity Hierarchy

**Base Entity: Employee**

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | @Id, @GeneratedValue | Unique identifier (database-generated) |
| name | String | @NotBlank, max 100 chars | Full name |
| nif | String | @NotBlank, @Pattern (9 digits), unique | Tax identification number |
| hireDate | LocalDate | @NotNull, @PastOrPresent | Date of employment |
| salary | BigDecimal | @NotNull, @Positive | Monthly salary |

**Inheritance Strategy:** Use `@Inheritance` with `JOINED` or `SINGLE_TABLE` strategy

**Derived Entities:**

**Salesperson (extends Employee)**
- `commissionRate` (BigDecimal): @NotNull, @DecimalMin("0.0"), @DecimalMax("100.0") - percentage
- `totalSales` (BigDecimal): @NotNull, @PositiveOrZero - cumulative sales amount

**Manager (extends Employee)**
- `department` (String): @NotBlank - department managed (e.g., "Sales", "Operations")
- `bonus` (BigDecimal): @NotNull, @PositiveOrZero - annual performance bonus

**JPA Annotations Required:**
- `@Entity`
- `@Inheritance(strategy = InheritanceType.JOINED)` or `SINGLE_TABLE`
- `@DiscriminatorColumn` (if using SINGLE_TABLE)
- `@DiscriminatorValue` in subclasses

---

### 2. Jewelry Entity Hierarchy

**Base Entity: Jewelry**

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | @Id, @GeneratedValue | Unique identifier |
| name | String | @NotBlank, max 200 chars | Product name |
| type | JewelryType | @NotNull, @Enumerated | Enum: NECKLACE, EARRING, RING |
| material | String | @NotBlank | Material (gold, silver, platinum) |
| weight | BigDecimal | @NotNull, @Positive | Weight in grams |
| price | BigDecimal | @NotNull, @Positive | Unit price in euros |
| stock | Integer | @NotNull, @PositiveOrZero | Available quantity |
| category | Category | @NotNull, @Enumerated | Enum: LUXURY, CASUAL, BRIDAL |

**Inheritance Strategy:** Use `@Inheritance` (same strategy as Employee)

**Derived Entities:**

**Necklace (extends Jewelry)**
- `length` (BigDecimal): @NotNull, @Positive - length in centimeters

**Earring (extends Jewelry)**
- `claspType` (String): @NotBlank - type of clasp (e.g., "Stud", "Hook", "Leverback")

**Ring (extends Jewelry)**
- `size` (Integer): @NotNull, @Min(10), @Max(30) - ring size (European standard)

---

### 3. Customer Entity

**Entity: Customer**

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | @Id, @GeneratedValue | Unique identifier |
| name | String | @NotBlank, max 100 chars | Full name |
| nif | String | @NotBlank, @Pattern (9 digits), unique | Tax identification number |
| email | String | @NotBlank, @Email | Email address |
| phone | String | @NotBlank, @Pattern (9 digits) | Phone number |
| address | String | @NotBlank | Full address |

**Relationships:**
- `@OneToMany(mappedBy = "customer")` with Order

---

### 4. Order Entity

**Entity: Order**

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | @Id, @GeneratedValue | Unique identifier |
| orderDate | LocalDate | @NotNull, auto-set | Date order was placed |
| totalAmount | BigDecimal | Calculated field | Total order value |
| status | OrderStatus | @NotNull, @Enumerated | Enum: PENDING, ACCEPTED, DELIVERED, CANCELED |

**Relationships:**
- `@ManyToOne` with Customer (FK: customer_id)
- `@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)` with OrderItem
- `@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)` with Payment

**OrderStatus Enum:** PENDING, ACCEPTED, DELIVERED, CANCELED

---

### 5. OrderItem Entity

**Entity: OrderItem (Association Entity)**

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | @Id, @GeneratedValue | Unique identifier |
| quantity | Integer | @NotNull, @Positive | Number of units |
| bookPriceAtPurchase | BigDecimal | @NotNull, @Positive | Price at time of order |
| subtotal | BigDecimal | Calculated: quantity × price | Line item total |

**Relationships:**
- `@ManyToOne` with Order (FK: order_id)
- `@ManyToOne` with Jewelry (FK: jewelry_id)

**Important:** This is the associative entity that resolves the many-to-many relationship between Order and Jewelry.

---

### 6. Payment Entity

**Entity: Payment**

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | @Id, @GeneratedValue | Unique identifier |
| amount | BigDecimal | @NotNull, @Positive | Payment amount |
| paymentDate | LocalDate | @NotNull, auto-set | Date of payment |
| paymentMethod | PaymentMethod | @NotNull, @Enumerated | Enum: CREDIT_CARD, BANK_TRANSFER, CASH |

**Relationships:**
- `@ManyToOne` with Order (FK: order_id)

**PaymentMethod Enum:** CREDIT_CARD, BANK_TRANSFER, CASH

---

### Entity Relationship Summary

```
Customer 1 ────────── * Order
Order 1 ────────── * OrderItem
OrderItem * ────────── 1 Jewelry
Order 1 ────────── * Payment

Employee (abstract or concrete)
    ├── Salesperson
    └── Manager

Jewelry (abstract or concrete)
    ├── Necklace
    ├── Earring
    └── Ring
```

---

## Technical Architecture Requirements

### Mandatory Technology Stack

| Component | Technology | Version/Notes |
|-----------|-----------|---------------|
| **Framework** | Spring Boot | Latest stable version (3.x recommended) |
| **Language** | Java | 17 or higher |
| **Build Tool** | Maven | pom.xml configuration |
| **Database** | MySQL or PostgreSQL | Choose one (must use Docker Compose) |
| **ORM** | Spring Data JPA | With Hibernate implementation |
| **Validation** | Jakarta Bean Validation | Hibernate Validator |
| **JSON Processing** | Jackson | Included with Spring Boot |
| **Containerization** | Docker Compose | For database deployment |

---

### Required Package Structure

Organize your code according to the following layered architecture:

```
pt.ipp.estg.doa.store
│
├── controller
│   ├── EmployeeController.java
│   ├── JewelryController.java
│   ├── CustomerController.java
│   ├── OrderController.java
│   └── PaymentController.java
│
├── service
│   ├── EmployeeService.java
│   ├── JewelryService.java
│   ├── CustomerService.java
│   ├── OrderService.java
│   └── PaymentService.java
│
├── repository
│   ├── EmployeeRepository.java
│   ├── SalespersonRepository.java
│   ├── ManagerRepository.java
│   ├── JewelryRepository.java
│   ├── NecklaceRepository.java
│   ├── EarringRepository.java
│   ├── RingRepository.java
│   ├── CustomerRepository.java
│   ├── OrderRepository.java
│   ├── OrderItemRepository.java
│   └── PaymentRepository.java
│
├── model
│   ├── entity
│   │   ├── Employee.java
│   │   ├── Salesperson.java
│   │   ├── Manager.java
│   │   ├── Jewelry.java
│   │   ├── Necklace.java
│   │   ├── Earring.java
│   │   ├── Ring.java
│   │   ├── Customer.java
│   │   ├── Order.java
│   │   ├── OrderItem.java
│   │   ├── Payment.java
│   │   ├── JewelryType.java (enum)
│   │   ├── Category.java (enum)
│   │   ├── OrderStatus.java (enum)
│   │   └── PaymentMethod.java (enum)
│   │
│   └── dto
│       ├── request
│       │   ├── CreateEmployeeRequest.java
│       │   ├── CreateSalespersonRequest.java
│       │   ├── CreateManagerRequest.java
│       │   ├── CreateJewelryRequest.java
│       │   ├── CreateNecklaceRequest.java
│       │   ├── CreateEarringRequest.java
│       │   ├── CreateRingRequest.java
│       │   ├── CreateCustomerRequest.java
│       │   ├── CreateOrderRequest.java
│       │   ├── AddOrderItemRequest.java
│       │   ├── AddPaymentRequest.java
│       │   └── UpdateOrderStatusRequest.java
│       │
│       └── response
│           ├── EmployeeResponse.java
│           ├── SalespersonResponse.java
│           ├── ManagerResponse.java
│           ├── JewelryResponse.java
│           ├── NecklaceResponse.java
│           ├── EarringResponse.java
│           ├── RingResponse.java
│           ├── CustomerResponse.java
│           ├── OrderResponse.java
│           ├── OrderItemResponse.java
│           ├── PaymentResponse.java
│           └── ErrorResponse.java
│
├── exception
│   ├── ResourceNotFoundException.java
│   ├── InvalidOperationException.java
│   ├── OutOfStockException.java
│   ├── InvalidStatusTransitionException.java
│   ├── DuplicateNifException.java
│   └── GlobalExceptionHandler.java
│
├── config
│   └── (optional configuration classes)
│
└── DoaStoreApplication.java (main class)
```

---

### Layer Responsibilities

**Controller Layer:**
- Handles HTTP requests and responses
- Maps endpoints to service methods
- Validates incoming DTOs
- Transforms entities to response DTOs
- Returns appropriate HTTP status codes
- **Must not contain business logic**

**Service Layer:**
- Contains all business logic and rules
- Validates business constraints
- Orchestrates operations across multiple repositories
- Manages transactions
- Throws custom exceptions for error cases
- Transforms between entities and DTOs

**Repository Layer:**
- Extends Spring Data JPA interfaces (`JpaRepository` or `CrudRepository`)
- Defines custom query methods if needed
- **Contains no business logic**

**Entity Layer:**
- JPA entities with proper annotations
- Relationships defined with JPA annotations
- No business logic (only getters/setters and basic methods)

**DTO Layer:**
- Request DTOs for incoming data
- Response DTOs for outgoing data
- Includes validation annotations
- Decouples API from internal entity structure

---

## JPA and Database Design

### JPA Entity Mapping Requirements

#### 1. Inheritance Mapping

**You must implement JPA inheritance for both Employee and Jewelry hierarchies.**

Choose one of the following strategies and document your choice:

**Option A: JOINED Strategy**
```
@Inheritance(strategy = InheritanceType.JOINED)
```
- Creates separate tables for base class and each subclass
- Base table contains common attributes
- Subclass tables contain specific attributes
- Foreign key joins tables together

**Option B: SINGLE_TABLE Strategy**
```
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "employee_type", discriminatorType = DiscriminatorType.STRING)
```
- All classes in hierarchy stored in one table
- Uses discriminator column to identify type
- Subclass-specific columns nullable

**Recommendation:** Use JOINED strategy for better normalization, unless performance is critical.

#### 2. Relationship Mappings

**Customer to Order (One-to-Many):**
```
In Customer entity:
@OneToMany(mappedBy = "customer", cascade = CascadeType.ALL, orphanRemoval = true)

In Order entity:
@ManyToOne
@JoinColumn(name = "customer_id", nullable = false)
```

**Order to OrderItem (One-to-Many, Composition):**
```
In Order entity:
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)

In OrderItem entity:
@ManyToOne
@JoinColumn(name = "order_id", nullable = false)
```

**OrderItem to Jewelry (Many-to-One):**
```
In OrderItem entity:
@ManyToOne
@JoinColumn(name = "jewelry_id", nullable = false)
```

**Order to Payment (One-to-Many):**
```
In Order entity:
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)

In Payment entity:
@ManyToOne
@JoinColumn(name = "order_id", nullable = false)
```

#### 3. ID Generation

Use database-generated IDs:
```
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

Alternatively, you may use UUID:
```
@Id
@GeneratedValue(strategy = GenerationType.UUID)
private UUID id;
```

#### 4. Enum Mapping

```
@Enumerated(EnumType.STRING)
private OrderStatus status;
```

Always use `EnumType.STRING` for better readability and maintainability.

#### 5. Calculated Fields

Fields like `totalAmount` in Order and `subtotal` in OrderItem should be calculated:

**Option A:** Calculate on-demand (no persistence)
```
@Transient
public BigDecimal getTotalAmount() {
    return orderItems.stream()
        .map(OrderItem::getSubtotal)
        .reduce(BigDecimal.ZERO, BigDecimal::add);
}
```

**Option B:** Store in database and update when needed
```
private BigDecimal totalAmount;

// Update in service layer when order changes
```

---

### Database Schema Requirements

Your JPA entities will generate database tables automatically. Ensure:

**Primary Keys:**
- All tables have primary key columns (id)
- Use auto-increment or UUID generation

**Foreign Keys:**
- Properly defined for all relationships
- Include ON DELETE rules where appropriate:
  - Order → Customer: RESTRICT (cannot delete customer with orders)
  - Order → OrderItem: CASCADE (delete items when order deleted)
  - OrderItem → Jewelry: RESTRICT (cannot delete jewelry in past orders)

**Unique Constraints:**
- Employee.nif
- Customer.nif
- Customer.email

**Check Constraints (if supported by database):**
- salary > 0
- price > 0
- stock >= 0
- quantity > 0
- ring size between 10 and 30

**Indexes (optional but recommended):**
- Index on foreign key columns
- Index on frequently queried fields (nif, email)

---

### application.properties Configuration

Your `src/main/resources/application.yaml` must include:

**Required Properties:**
- `spring.datasource.url` - JDBC URL to your Docker database
- `spring.datasource.username` - database username
- `spring.datasource.password` - database password
- `spring.jpa.hibernate.ddl-auto` - use `update` for development
- `spring.jpa.show-sql` - set to `true` for debugging
- `spring.jpa.properties.hibernate.format_sql` - set to `true` for readability

**Example Structure (do not hardcode - use environment variables or profiles):**
```
Database connection URL: jdbc:mysql://localhost:3306/doa_store
or: jdbc:postgresql://localhost:5432/doa_store

Username: doa_user
Password: secure_password

DDL Auto: update (creates/updates schema automatically)
Show SQL: true (for debugging)
```

---

## REST API Endpoints

### General API Design Principles

1. **Use RESTful URL patterns:**
   - Collections: `/api/employees`, `/api/books`
   - Specific resource: `/api/employees/1`, `/api/books/5`
   - Nested resources: `/api/orders/1/items`, `/api/orders/1/payments`

2. **Use appropriate HTTP methods:**
   - GET: Retrieve resources
   - POST: Create new resources
   - PUT: Update entire resource
   - PATCH: Partial update (optional)
   - DELETE: Remove resource

3. **Return appropriate HTTP status codes:**
   - 200 OK: Successful GET/PUT
   - 201 Created: Successful POST
   - 204 No Content: Successful DELETE
   - 400 Bad Request: Validation errors
   - 404 Not Found: Resource doesn't exist
   - 409 Conflict: Business rule violation
   - 500 Internal Server Error: Unexpected errors

4. **Use DTOs for all requests and responses**
   - Never expose JPA entities directly in API

5. **Support JSON format**
   - Content-Type: application/json
   - Accept: application/json

---

### Employee Endpoints

**Base Path:** `/api/employees`

| Method | Endpoint | Description | Request Body | Response | Status Codes |
|--------|----------|-------------|--------------|----------|--------------|
| GET | `/api/employees` | List all employees | None | List<EmployeeResponse> | 200 |
| GET | `/api/employees/{id}` | Get employee by ID | None | EmployeeResponse | 200, 404 |
| GET | `/api/employees/nif/{nif}` | Find by NIF | None | EmployeeResponse | 200, 404 |
| POST | `/api/employees/salesperson` | Create salesperson | CreateSalespersonRequest | SalespersonResponse | 201, 400 |
| POST | `/api/employees/manager` | Create manager | CreateManagerRequest | ManagerResponse | 201, 400 |
| PUT | `/api/employees/{id}/salary` | Update salary | { "salary": 3500.00 } | EmployeeResponse | 200, 404, 400 |
| DELETE | `/api/employees/{id}` | Delete employee | None | None | 204, 404, 409 |

**Example Response - GET /api/employees/1:**
```json
{
  "id": 1,
  "type": "SALESPERSON",
  "name": "Maria Silva",
  "nif": "123456789",
  "hireDate": "2020-01-15",
  "salary": 2500.00,
  "commissionRate": 5.5,
  "totalSales": 15000.00
}
```

**Example Request - POST /api/payments:**
```json
{
  "orderId": 1,
  "amount": 570.00,
  "paymentMethod": "CREDIT_CARD"
}
```

**Example Response:**
```json
{
  "id": 1,
  "orderId": 1,
  "amount": 570.00,
  "paymentDate": "2024-11-24",
  "paymentMethod": "CREDIT_CARD"
}
```

### Jewelry Endpoints

**Base Path:** `/api/jewelry`

| Method | Endpoint | Description | Request Body | Response | Status Codes |
|--------|----------|-------------|--------------|----------|--------------|
| GET | `/api/jewelry` | List all jewelry | None | List<JewelryResponse> | 200 |
| GET | `/api/jewelry/{id}` | Get jewelry by ID | None | JewelryResponse | 200, 404 |
| GET | `/api/jewelry/type/{type}` | Filter by type | None | List<JewelryResponse> | 200 |
| GET | `/api/jewelry/category/{category}` | Filter by category | None | List<JewelryResponse> | 200 |
| GET | `/api/jewelry/in-stock` | Only items with stock > 0 | None | List<JewelryResponse> | 200 |
| POST | `/api/jewelry/necklace` | Create necklace | CreateNecklaceRequest | NecklaceResponse | 201, 400 |
| POST | `/api/jewelry/earring` | Create earring | CreateEarringRequest | EarringResponse | 201, 400 |
| POST | `/api/jewelry/ring` | Create ring | CreateRingRequest | RingResponse | 201, 400 |
| PUT | `/api/jewelry/{id}/price` | Update price | { "price": 28.50 } | JewelryResponse | 200, 404, 400 |
| PUT | `/api/jewelry/{id}/stock` | Update stock | { "stock": 15 } | JewelryResponse | 200, 404, 400 |
| DELETE | `/api/jewelry/{id}` | Delete jewelry | None | None | 204, 404, 409 |

**Example Response - GET /api/jewelry/1:**
```json
{
  "id": 1,
  "type": "NECKLACE",
  "name": "Golden Chain",
  "material": "Gold",
  "weight": 15.5,
  "price": 450.00,
  "stock": 10,
  "category": "LUXURY",
  "length": 50.0
}
```

**Example Request - POST /api/jewelry/ring:**
```json
{
  "name": "Diamond Ring",
  "material": "Platinum",
  "weight": 8.0,
  "price": 2500.00,
  "stock": 5,
  "category": "BRIDAL",
  "size": 18
}
```

---

### Customer Endpoints

**Base Path:** `/api/customers`

| Method | Endpoint | Description | Request Body | Response | Status Codes |
|--------|----------|-------------|--------------|----------|--------------|
| GET | `/api/customers` | List all customers | None | List<CustomerResponse> | 200 |
| GET | `/api/customers/{id}` | Get customer by ID | None | CustomerResponse | 200, 404 |
| GET | `/api/customers/nif/{nif}` | Find by NIF | None | CustomerResponse | 200, 404 |
| GET | `/api/customers/email/{email}` | Find by email | None | CustomerResponse | 200, 404 |
| POST | `/api/customers` | Create customer | CreateCustomerRequest | CustomerResponse | 201, 400 |
| PUT | `/api/customers/{id}` | Update customer | CreateCustomerRequest | CustomerResponse | 200, 404, 400 |
| DELETE | `/api/customers/{id}` | Delete customer | None | None | 204, 404, 409 |

**Example Response - GET /api/customers/1:**
```json
{
  "id": 1,
  "name": "Ana Costa",
  "nif": "111222333",
  "email": "ana.costa@email.com",
  "phone": "912345678",
  "address": "Rua das Flores 123, Lisboa"
}
```

---

### Order Endpoints

**Base Path:** `/api/orders`

| Method | Endpoint | Description | Request Body | Response | Status Codes |
|--------|----------|-------------|--------------|----------|--------------|
| GET | `/api/orders` | List all orders | None | List<OrderResponse> | 200 |
| GET | `/api/orders/{id}` | Get order by ID | None | OrderResponse | 200, 404 |
| GET | `/api/orders/customer/{customerId}` | Orders by customer | None | List<OrderResponse> | 200 |
| GET | `/api/orders/status/{status}` | Filter by status | None | List<OrderResponse> | 200 |
| POST | `/api/orders` | Create order | CreateOrderRequest | OrderResponse | 201, 400, 404 |
| POST | `/api/orders/{id}/items` | Add item to order | AddOrderItemRequest | OrderResponse | 200, 400, 404, 409 |
| PUT | `/api/orders/{id}/status` | Update status | UpdateOrderStatusRequest | OrderResponse | 200, 400, 404, 409 |
| DELETE | `/api/orders/{id}` | Cancel order | None | None | 204, 404, 409 |

**Example Response - GET /api/orders/1:**
```json
{
  "id": 1,
  "customer": {
    "id": 1,
    "name": "Ana Costa"
  },
  "orderDate": "2024-11-01",
  "status": "DELIVERED",
  "items": [
    {
      "id": 1,
      "jewelry": {
        "id": 1,
        "name": "Golden Chain",
        "type": "NECKLACE"
      },
      "quantity": 1,
      "priceAtPurchase": 450.00,
      "subtotal": 450.00
    },
    {
      "id": 2,
      "jewelry": {
        "id": 2,
        "name": "Silver Studs",
        "type": "EARRING"
      },
      "quantity": 1,
      "priceAtPurchase": 120.00,
      "subtotal": 120.00
    }
  ],
  "totalAmount": 570.00,
  "payments": [
    {
      "id": 1,
      "amount": 570.00,
      "paymentDate": "2024-11-01",
      "paymentMethod": "CREDIT_CARD"
    }
  ]
}
```

**Example Request - POST /api/orders:**
```json
{
  "customerId": 1,
  "items": [
    {
      "jewelryId": 1,
      "quantity": 2
    },
    {
      "jewelryId": 3,
      "quantity": 1
    }
  ]
}
```

**Example Request - POST /api/orders/1/items:**
```json
{
  "jewelryId": 5,
  "quantity": 1
}
```

**Example Request - PUT /api/orders/1/status:**
```json
{
  "newStatus": "ACCEPTED"
}
```

### Payment Endpoints

**Base Path:** `/api/payments`

| Method | Endpoint | Description | Request Body | Response | Status Codes |
|--------|----------|-------------|--------------|----------|--------------|
| GET | `/api/payments` | List all payments | None | List<PaymentResponse> | 200 |
| GET | `/api/payments/{id}` | Get payment by ID | None | PaymentResponse | 200, 404 |
| GET | `/api/payments/order/{orderId}` | Payments for order | None | List<PaymentResponse> | 200 |
| POST | `/api/payments` | Add payment | AddPaymentRequest | PaymentResponse | 201, 400, 404, 409 |

**Example Response - GET /api/payments/1:**
```json
{
  "id": 1,
  "orderId": 1,
  "amount": 570.00,
  "paymentDate": "2024-11-01",
  "paymentMethod": "CREDIT_CARD"
}
```

**Example Request - POST /api/payments:**
```json
{
  "orderId": 1,
  "amount": 570.00,
  "paymentMethod": "CREDIT_CARD"
}
```

**Example Response - POST /api/payments:**
```json
{
  "id": 1,
  "orderId": 1,
  "amount": 570.00,
  "paymentDate": "2024-11-24",
  "paymentMethod": "CREDIT_CARD"
}
```

**Example Response - GET /api/payments/order/1:**
```json
[
  {
    "id": 1,
    "orderId": 1,
    "amount": 570.00,
    "paymentDate": "2024-11-01",
    "paymentMethod": "CREDIT_CARD"
  },
  {
    "id": 2,
    "orderId": 1,
    "amount": 200.00,
    "paymentDate": "2024-11-05",
    "paymentMethod": "BANK_TRANSFER"
  }
]
```


## Validation Requirements

### Jakarta Bean Validation

All request DTOs must include appropriate validation annotations. Spring Boot will automatically validate these when the request is received.

### Required Validation Annotations

**Employee/Customer Validation:**
- `@NotBlank` on name fields
- `@Pattern(regexp = "\\d{9}", message = "NIF must be exactly 9 digits")` on NIF
- `@Email` on email addresses
- `@Pattern(regexp = "\\d{9}", message = "Phone must be exactly 9 digits")` on phone
- `@PastOrPresent` on hireDate
- `@Positive` on salary
- `@DecimalMin("0.0")` and `@DecimalMax("100.0")` on commissionRate

**Jewelry Validation:**
- `@NotBlank` on name, material
- `@Positive` on weight, price
- `@PositiveOrZero` on stock
- `@NotNull` on type, category
- `@Min(10)` and `@Max(30)` on ring size
- `@Positive` on necklace length

**Order/OrderItem Validation:**
- `@NotNull` on customerId, jewelryId, orderId
- `@Positive` on quantity, amount
- `@NotNull` on status, paymentMethod

**Payment Validation:**
- `@NotNull` on orderId
- `@Positive` on amount
- `@NotNull` on paymentMethod

### Validation Error Response Format

When validation fails, return HTTP 400 with structured error response:

**Example Validation Error Response:**
```json
{
  "timestamp": "2024-11-24T10:30:00",
  "status": 400,
  "error": "Bad Request",
  "message": "Validation failed",
  "errors": [
    {
      "field": "nif",
      "rejectedValue": "12345",
      "message": "NIF must be exactly 9 digits"
    },
    {
      "field": "email",
      "rejectedValue": "invalid-email",
      "message": "must be a well-formed email address"
    }
  ]
}
```

---

## Exception Handling

### Required Custom Exceptions

Create the following custom exception classes:

**1. ResourceNotFoundException**
- Thrown when: Entity not found by ID
- HTTP Status: 404 NOT FOUND
- Example: "Employee with id 5 not found"

**2. InvalidOperationException**
- Thrown when: Business rule violation
- HTTP Status: 409 CONFLICT
- Example: "Cannot delete customer with existing orders"

**3. OutOfStockException**
- Thrown when: Insufficient stock for order
- HTTP Status: 409 CONFLICT
- Example: "Insufficient stock for jewelry 'Golden Chain'. Available: 2, Requested: 5"

**4. InvalidStatusTransitionException**
- Thrown when: Invalid order status change
- HTTP Status: 409 CONFLICT
- Example: "Cannot transition from DELIVERED to PENDING"

**5. DuplicateNifException**
- Thrown when: NIF already exists
- HTTP Status: 409 CONFLICT
- Example: "Employee with NIF 123456789 already exists"

### Global Exception Handler

Implement `@ControllerAdvice` class to handle all exceptions globally:

**GlobalExceptionHandler Requirements:**
- Catch all custom exceptions and return appropriate ErrorResponse
- Catch `MethodArgumentNotValidException` for validation errors
- Catch generic `Exception` as fallback
- Return consistent ErrorResponse structure
- Include timestamp, status code, error message, and path

**ErrorResponse Structure:**
```json
{
  "timestamp": "2024-11-24T10:30:00",
  "status": 404,
  "error": "Not Found",
  "message": "Employee with id 5 not found",
  "path": "/api/employees/5"
}
```

**For Validation Errors (400):**
```json
{
  "timestamp": "2024-11-24T10:30:00",
  "status": 400,
  "error": "Bad Request",
  "message": "Validation failed",
  "errors": [
    {
      "field": "salary",
      "rejectedValue": -1000,
      "message": "must be greater than 0"
    }
  ]
}
```

---

## Docker Compose Requirements

### Mandatory Docker Compose Setup

You **must** provide a `docker-compose.yml` file that provisions the full application database. 

### Docker Compose File Requirements

**File Location:** Project root directory (`docker-compose.yml`)

**Required Services:**
- Database service (MySQL or PostgreSQL)
- Named volumes for data persistence
- Environment variables for configuration
- Port mapping to host
- Health checks (optional but recommended)

### MySQL Example Structure

**Service Definition Requirements:**
- Service name: `mysql-db` or similar
- Image: `mysql:8.0` or `mysql:latest`
- Environment variables:
  - `MYSQL_ROOT_PASSWORD`
  - `MYSQL_DATABASE` (e.g., `doa_store`)
  - `MYSQL_USER` (e.g., `doa_user`)
  - `MYSQL_PASSWORD`
- Port mapping: `3306:3306`
- Volume mount for data persistence
- Restart policy: `unless-stopped` or `always`

**Expected Connection Details:**
- Host: `localhost` (from Spring Boot running on host)
- Port: `3306`
- Database: `doa_store`
- Username: `doa_user`
- Password: (your chosen password)

### PostgreSQL Example Structure

**Service Definition Requirements:**
- Service name: `postgres-db` or similar
- Image: `postgres:15` or `postgres:latest`
- Environment variables:
  - `POSTGRES_DB` (e.g., `doa_store`)
  - `POSTGRES_USER` (e.g., `doa_user`)
  - `POSTGRES_PASSWORD`
- Port mapping: `5432:5432`
- Volume mount for data persistence
- Restart policy: `unless-stopped` or `always`

**Expected Connection Details:**
- Host: `localhost`
- Port: `5432`
- Database: `doa_store`
- Username: `doa_user`
- Password: (your chosen password)

### Volume Requirements

Define named volume for database data persistence:
- Volume name: `doa-db-data` or similar
- Mounted to appropriate database data directory
- Ensures data survives container restarts

### Health Check (Optional but Recommended)

Include health check configuration:
- Test command to verify database is ready
- Interval: 10-30 seconds
- Timeout: 5 seconds
- Retries: 5

### Usage Instructions

Your README must include:
1. Command to start database: `docker-compose up -d`
2. Command to stop database: `docker-compose down`
3. Command to view logs: `docker-compose logs -f`
4. How to verify database is running
5. How to connect to database for debugging (optional)

---

## Business Rules Implementation

All business rules from Project 1 must be implemented in the Service layer. Key rules include:

### Employee Rules

1. **NIF Uniqueness:** No two employees can have the same NIF
   - Check before creating employee
   - Throw `DuplicateNifException` if exists
   - This can also be done with database restriction

2. **NIF Format:** Must be exactly 9 digits
   - Validate with `@Pattern` annotation
   - Return 400 if invalid

3. **Salary Constraints:** Salary must be greater than 0
   - Validate with `@Positive` annotation

4. **Commission Rate:** Must be between 0 and 100 for salespeople
   - Validate with `@DecimalMin` and `@DecimalMax`

### Jewelry Rules

5. **Stock Management:**
   - Stock cannot be negative
   - When order status changes to DELIVERED, reduce stock
   - When order is CANCELED, restore stock
   - Check stock availability before creating order

6. **Out of Stock Prevention:**
   - Before adding item to order, verify sufficient stock
   - Throw `OutOfStockException` if insufficient
   - Include details: available vs requested quantity

7. **Pricing:** Price must be greater than 0
   - Validate with `@Positive`

8. **Ring Size:** Must be between 10 and 30
   - Validate with `@Min(10)` and `@Max(30)`

### Customer Rules

9. **NIF and Email Uniqueness:**
   - No duplicate NIFs across customers
   - No duplicate emails across customers
   - Throw `DuplicateNifException` or similar

10. **Deletion Prevention:**
    - Cannot delete customer with existing orders
    - Throw `InvalidOperationException` with message

### Order Rules

11. **Order Creation:**
    - Must have at least one order item
    - Validate stock availability for all items
    - Calculate totalAmount from order items
    - Set status to PENDING by default
    - Capture current jewelry price in OrderItem

12. **Status Transitions:**
    - Valid transitions only:
      - PENDING → ACCEPTED
      - PENDING → CANCELED
      - ACCEPTED → DELIVERED
      - ACCEPTED → CANCELED
    - Invalid transitions throw `InvalidStatusTransitionException`
    - DELIVERED and CANCELED are terminal states

13. **Stock Updates on Status Change:**
    - PENDING → ACCEPTED: Validate stock (reserve)
    - ACCEPTED → DELIVERED: Reduce stock permanently
    - Any status → CANCELED: Restore stock if previously reduced

14. **Cannot Modify Delivered/Canceled Orders:**
    - Cannot add items to delivered or canceled orders
    - Cannot change status from delivered or canceled
    - Throw `InvalidOperationException`

### Payment Rules

15. **Payment Amount:** Must be greater than 0
    - Validate with `@Positive`

16. **Multiple Payments Allowed:**
    - An order can have multiple partial payments
    - Track total paid amount (optional: validate doesn't exceed order total)

17. **Payment Method:** Must be valid enum value
    - CREDIT_CARD, BANK_TRANSFER, or CASH

### Data Integrity Rules

18. **Referential Integrity:**
    - Cannot delete jewelry that appears in any order
    - Cannot delete customer with existing orders
    - Throw `InvalidOperationException` with explanation

19. **Cascade Operations:**
    - When order is deleted, delete associated order items and payments
    - Implemented via JPA cascade settings

---

## Submission Requirements

Submit the repository link in a simple .txt file in moodle. Must be given permission to the teacher in order tov iew the repository.

## Tips for Success

### Common Pitfalls to Avoid

1. **Circular Dependencies:** Be careful with bidirectional JPA relationships
   - Use `@JsonIgnore` or DTOs to avoid serialization issues

2. **Lazy Loading Issues:** Understand fetch types
   - Use `FetchType.LAZY` wisely
   - Load necessary data in service layer before returning

3. **Transaction Management:** Service methods should be transactional
   - Use `@Transactional` on service methods that modify data

4. **Entity Exposure:** Never expose JPA entities directly in API
   - Always use DTOs for requests and responses

5. **Hardcoded Values:** Use environment variables or profiles
   - Don't commit passwords or sensitive data

6. **Validation Placement:** Validate at API boundary (controllers)
   - Business rule validation in service layer
   - Use `@Valid` annotation in controller methods

7. **Error Messages:** Provide clear, helpful error messages
   - Include enough context for debugging
   - Don't expose internal implementation details

### Testing Your Application

1. **Database Connection:**
   - Start Docker Compose: `docker-compose up -d`
   - Check database is running: `docker ps`
   - View logs: `docker-compose logs -f`

2. **Application Startup:**
   - Build: `mvn clean package`
   - Run: `mvn spring-boot:run` or `java -jar target/doa-store-0.0.1-SNAPSHOT.jar`
   - Check console for errors
   - Verify application starts on port 8080

3. **API Testing:**
   - Test with browser for GET requests
   - Use Postman for POST/PUT/DELETE
   - Test happy paths first, then error cases
   - Verify database changes directly (optional)

4. **Validation Testing:**
   - Send invalid data and verify 400 responses
   - Check error messages are helpful

5. **Business Rule Testing:**
   - Test order status transitions
   - Test stock management
   - Try to delete entities with dependencies
   - Verify all constraints work

---

## Additional Resources

### Spring Boot Documentation
- Official Spring Boot Documentation: https://spring.io/projects/spring-boot
- Spring Data JPA: https://spring.io/projects/spring-data-jpa
- Spring REST Guide: https://spring.io/guides/tutorials/rest/

### JPA and Hibernate
- JPA Inheritance Strategies: https://www.baeldung.com/hibernate-inheritance
- JPA Relationships: https://www.baeldung.com/jpa-one-to-one
- Hibernate Documentation: https://hibernate.org/orm/documentation/

### Docker
- Docker Compose Documentation: https://docs.docker.com/compose/
- MySQL Docker Image: https://hub.docker.com/_/mysql
- PostgreSQL Docker Image: https://hub.docker.com/_/postgres

### Tools
- Spring Initializr: https://start.spring.io/
- Postman: https://www.postman.com/
- DBeaver (database client): https://dbeaver.io/
- IntelliJ IDEA / Eclipse / VS Code

**Good luck, and happy coding!**

