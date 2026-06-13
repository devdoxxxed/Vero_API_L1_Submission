## Java & Object-Oriented Design

### 1.

`@MappedSuperclass` tells JPA that the fields in the abstract class should be inherited by entity classes and mapped into their database tables. The superclass itself does not get its own table. If the annotation were removed, JPA would no longer automatically map those inherited fields, and entities extending the class would lose those columns unless they were redeclared.

Using a shared `Auditable` class avoids duplication and ensures that all entities use a consistent auditing model. It also makes future changes easier because audit-related modifications only need to be made in one place.

---

### 2.

Even with only one implementation today, the interface defines a contract that separates consumers from implementation details. This improves maintainability and testability.

A concrete example is introducing a new implementation that reads data from an external service or a cache layer. Controllers and other consumers would not need to change because they depend on the interface rather than a specific implementation.

---

### 3.

Using `@Enumerated(EnumType.STRING)` stores the enum name as text in the database, such as `FOOD`, `TRANSPORT`, or `SHOPPING`.

This is preferable to ordinal storage because the values remain readable and are not affected by enum ordering changes. If a developer adds a new category without considering existing data or migration requirements, application logic may not handle the new value correctly, potentially causing reporting or validation issues.

---

### 4.

This is a utility-class pattern. The class is marked `final` and has a private constructor so it cannot be instantiated or extended.

For the implementation, I grouped transactions by category and aggregated totals before sorting them. The final result was collected into a `LinkedHashMap` because insertion order is preserved after sorting, allowing the top spending categories to remain in descending order.

---

## Spring Boot & REST API Design

### 5.

Returning an entity directly exposes the persistence model to API consumers. This creates tight coupling between database structure and API contracts.

A DTO provides a controlled representation of the data that is safe to expose externally. It prevents accidental exposure of internal fields and allows the API to evolve independently of database entities.

---

### 6.

The request first reaches the controller through Spring MVC routing. The controller receives the JSON payload and uses validation annotations to validate the request object. The controller then delegates to the service layer, which contains business logic. The service creates a transaction entity and passes it to the repository. The repository interacts with JPA and Hibernate, which generate SQL and persist the record in the database.

If `@Valid` were removed, invalid requests could reach the service layer, potentially allowing incomplete or incorrect data to be persisted.

---

### 7.

All three annotations register Spring beans, but they communicate intent. `@RestController` indicates web-layer functionality, `@Service` indicates business logic, and `@Repository` indicates data access.

This separation improves readability and helps developers understand responsibilities quickly when navigating the codebase.

---

### 8.

The endpoint should return HTTP 400 Bad Request because the request contains invalid input.

Validation can be implemented using Spring validation annotations or explicit checks in the controller or service layer. I would validate that the month is between 1 and 12 and return a clear validation error message when the value is invalid.

---

## Data Access & SQL

### 9.

At startup, Spring Data JPA parses repository method names and generates query implementations automatically. The framework recognizes patterns such as `findByAccountId` and translates them into SQL based on entity metadata.

I would use `@Query` when the query becomes too complex for derived method names or when custom joins, aggregations, or database-specific functionality are required.

---

### 10.

The bug excluded transactions occurring on the first day of the month. The implementation used `isAfter(startOfMonth)`, which only includes dates strictly after the first day.

The issue became visible when a transaction occurred exactly on the first day of the month. Boundary-value test cases are effective at exposing this type of defect. Date and time logic frequently suffers from off-by-one errors because developers must carefully distinguish between inclusive and exclusive comparisons.

---

### 11.

For PostgreSQL, I would replace the H2 dependency in `pom.xml` with the PostgreSQL JDBC driver and update `application.properties` with PostgreSQL connection details, username, password, and dialect configuration.

Using `spring.jpa.hibernate.ddl-auto=create-drop` in production is dangerous because it can recreate or delete schema objects and lead to data loss. In production, schema migrations should be managed through tools such as Flyway or Liquibase.

---

## Testing

### 12.

Mockito replaces the repository with a mock object rather than using a real database. The service under test receives this mock through dependency injection.

The tests verify business logic in isolation by controlling repository responses and validating service behavior. These tests can catch logic errors but cannot detect database configuration problems, SQL issues, or JPA mapping errors.

---

### 13.

I do not agree. Controller tests validate request routing, validation behavior, HTTP status codes, request serialization, and response serialization.

For example, a controller test using MockMvc could detect that invalid request payloads are incorrectly accepted because validation annotations are missing. Service tests alone would not catch that issue.

---

### 14.

The first tests I focused on were the failing tests already present in the project because they provided the fastest feedback on broken functionality.

This approach reflects a debugging-first workflow. I prioritized restoring correctness in existing behavior before adding or extending functionality.

---

## AI & Modern Engineering

### 15.

I used ChatGPT during development for debugging, investigation, and documentation support.

One example was diagnosing build failures. I shared compilation errors and environment information, and the AI helped identify a Java version mismatch between the project requirements and the Maven runtime. I verified the diagnosis through Maven output before applying changes.

Another example was investigating failing tests. The AI helped interpret test failures and locate defects in monthly spend aggregation and category calculation logic. I reviewed the relevant code manually and validated the fixes through the test suite.

The most trustworthy AI assistance was interpreting build logs and identifying likely root causes. The area requiring the most scrutiny was implementation guidance, where suggestions had to be verified against the actual codebase and assignment requirements before being accepted.
