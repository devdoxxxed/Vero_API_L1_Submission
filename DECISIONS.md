**Name:** Devaansh Pareek

**Date started:** 13 June 2026

**Date submitted:** 13 June 2026

I started by reading the README and understanding the expected API functionality before making any code changes. My first priority was obtaining a clean build and working development environment because compilation issues can hide application defects. Once the build was stable, I investigated failing tests and unfinished implementations before making targeted fixes.

---

## 1. Code & Design Decisions

### The codebase includes an `Auditable` abstract class that is not currently used by any entity. What did you do with it, if anything? Walk through your reasoning — what is the purpose of the Auditable pattern, what are the tradeoffs of using it versus not, and why did you make the choice you did?

I did not modify or remove the `Auditable` class. The purpose of the pattern is to centralize common audit fields such as creation timestamps, update timestamps, and user tracking information. This reduces duplication across entities and improves consistency.

The tradeoff is that introducing inheritance adds complexity and requires careful consideration of entity design. Since the current transaction functionality did not depend on auditing behavior and there was no active usage of the class, I chose not to introduce additional changes that were outside the scope of the assignment.

---

### `TransactionResponse` is used as the outbound DTO for the API. What changes did you make to it, if any? Why does the shape of a response DTO matter — and what is the risk of returning an entity directly from a controller?

I did not make significant structural changes to `TransactionResponse`.

Using a response DTO creates a stable API contract that is independent of the persistence model. Returning entities directly from controllers can expose internal implementation details, create unwanted coupling between the database model and API consumers, and may accidentally expose sensitive fields in the future. DTOs also make it easier to evolve the API without changing the underlying database entities.

---

### The `BudgetCalculator` requires grouping and sorting data. What data structure or approach did you choose to implement it? Walk through the alternatives you considered and why you landed where you did.

I implemented the calculator using Java Streams. Transactions are grouped by category, reduced into category totals using `BigDecimal::add`, sorted by total spend in descending order, limited to the requested number of categories, and collected into a `LinkedHashMap`.

I considered manually iterating through transactions and maintaining a map, but the Stream API provided a concise and readable implementation while preserving the required ordering through `LinkedHashMap`.

---

### Were there any decisions you made that are not covered by the questions above? Describe the most significant one and your reasoning.

The most significant decision was removing an unused service-layer method that referenced a repository method which did not exist. The method was not declared in the service interface, was not used by any controller, and prevented compilation. Removing dead code was preferable to introducing unsupported functionality that had no active consumer.

---

## 2. Bug Fixes & Issues Found

### Describe each problem you found in the codebase. For each one: where was it, how did you identify it, what did it cause, and how did you fix it?

1. Build Failure Due to Missing Repository Method

    * Location: TransactionServiceImpl
    * Issue: An implementation method referenced `findByCategoryAndMonth(...)`, which did not exist in the repository.
    * Impact: Project would not compile.
    * Fix: Removed the unused method because it was not part of the service interface and had no callers.

2. Monthly Spend Aggregation Excluded First Day of Month

    * Location: `calculateMonthlySpend(...)`
    * Issue: The implementation used `isAfter(startOfMonth)` which excluded transactions occurring on the first day.
    * Impact: Monthly totals were incorrect.
    * Fix: Updated the filtering logic to include both start and end dates.

3. Incomplete BudgetCalculator Implementation

    * Location: BudgetCalculator
    * Issue: Method always returned an empty map.
    * Impact: Top spending category calculations failed.
    * Fix: Implemented grouping, aggregation, sorting, and top-N selection logic.

4. Date Range Filtering Not Implemented

    * Location: TransactionServiceImpl
    * Issue: Method returned `Collections.emptyList()`.
    * Impact: Date filtering functionality did not work.
    * Fix: Implemented filtering based on inclusive start and end dates.

5. Environment and Build Configuration Issues

    * Issue: Maven was running with Java 24 while the project targets Java 17.
    * Impact: Build investigation became more difficult and initially masked the actual code issues.
    * Fix: Aligned the runtime environment with project requirements.

### Were there any problems you noticed but chose not to fix? If so, explain why.

I noticed that monthly spend aggregation and date range filtering currently perform in-memory filtering over all transactions. For the current scope and application size this is acceptable, but for larger datasets I would move these operations into repository-level database queries. I chose not to change this because it would significantly expand the scope beyond the assignment requirements.

---

## 4. AI Tool Usage

### Which AI tools did you use?

I used ChatGPT during development.

### Give two or three specific examples of how you used AI on this project.

1. Environment Troubleshooting

    * I used AI to help diagnose compilation failures and identify the Java runtime mismatch between the project requirements and Maven execution environment.
    * I verified all suggested changes before applying them.

2. Test Failure Analysis

    * I used AI to help interpret failing test output and locate the underlying defects in monthly spend calculation and category aggregation.
    * I reviewed the code paths manually before making changes.

3. Documentation Support

    * I used AI to help organize and summarize decisions, findings, and reasoning into structured documentation.

### Describe a moment where AI gave you something wrong, incomplete, or subtly misleading.

At one point AI focused heavily on Lombok configuration as the root cause of compilation issues. Further investigation showed that the larger issue was the project environment and Java version mismatch. I validated assumptions through build output and Maven diagnostics before making changes.

### What is your general philosophy on using AI when writing backend code?

AI is useful for accelerating investigation, summarizing information, and generating implementation ideas. However, I do not treat AI output as authoritative. Build results, tests, runtime behavior, and code review remain the final source of truth.
