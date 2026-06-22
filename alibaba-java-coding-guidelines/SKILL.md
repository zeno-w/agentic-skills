---
name: "alibaba-java-coding-guidelines"
description: "Enforces Alibaba Java Coding Guidelines for code review, generation, and refactoring. Invoke when writing Java code, reviewing Java PRs, fixing Java style issues, or when user mentions Alibaba coding standards, Java conventions, or Chinese Java development norms."
---

# Alibaba Java Coding Guidelines

This skill encodes the core rules from the Alibaba Java Coding Guidelines (latest edition - v1.7.1+), the de facto standard for Java development in China. Apply these rules when writing, reviewing, or refactoring Java code.

The guidelines are organized into the following sections. For detailed rules, read the corresponding reference file:

| Section | Reference File | When to Read |
|---------|---------------|-------------|
| Naming Conventions | `references/naming-conventions.md` | When naming classes, methods, variables, packages, constants |
| OOP Conventions | `references/oop-conventions.md` | When writing POJOs, using equals/hashCode, inheritance, casting |
| Collection & Concurrency | `references/collection-concurrency.md` | When using collections, thread pools, locks, concurrent utilities |
| Exception, Logging & Control | `references/exception-logging-control.md` | When handling exceptions, writing log statements, control flow |
| Format, Comments & Structure | `references/format-comments-structure.md` | When formatting code, writing comments, organizing project structure |
| Unit Test & Security | `references/unit-test-security.md` | When writing tests, handling security concerns |

## Quick-Reference: Top 27 Mandatory Rules

These are the most frequently violated mandatory rules. Always check these first:

### Naming (5 rules)
1. Names must not start/end with `_` or `$`
2. Class names use UpperCamelCase; method/variable names use lowerCamelCase
3. Constants use UPPER_SNAKE_CASE with semantic clarity (e.g., `MAX_STOCK_COUNT`)
4. Boolean fields must NOT use `is` prefix (causes serialization issues)
5. Abstract classes start with `Abstract`/`Base`; exceptions end with `Exception`

### OOP (6 rules)
6. Access static members via class name, not object reference
7. All override methods must have `@Override` annotation
8. Use `Objects.equals()` instead of `object.equals()` to avoid NPE
9. POJO classes must have no-arg constructor and `toString()`
10. No business logic in constructors; use `init()` method
11. Do not modify `serialVersionUID` when adding fields to serializable classes

### Collections & Concurrency (6 rules)
12. Do not use `Executors` for thread pools; use `ThreadPoolExecutor` directly
13. `SimpleDateFormat` is NOT thread-safe; use `DateTimeFormatter` or `ThreadLocal`
14. Always call `ThreadLocal.remove()` after use to prevent memory leaks
15. Do not add/remove elements in `foreach` loop; use `Iterator` or `removeIf`
16. `Arrays.asList()` returns a fixed-size list; do not modify it
17. Use `ConcurrentHashMap` instead of `Hashtable` for concurrent maps

### Exception & Logging (5 rules)
18. Catch specific exceptions, not `Exception` or `Throwable`
19. Never leave a catch block empty; at minimum log the exception
20. Do not use `return` in finally block
21. Use SLF4J + parameterized logging: `log.info("User {}", userId)`
22. Never log sensitive data (passwords, tokens, ID numbers)

### Control & Format (2 rules)
23. Every `switch` must have a `default` case
24. Always use braces `{}` even for single-line `if/else/for/while`

### Security (3 rules)
25. Use BCrypt/Argon2 for passwords; never MD5/SHA1
26. Validate all user input server-side; use Bean Validation annotations
27. Do not expose stack traces in error responses to clients

## How to Apply These Guidelines

### When Writing New Java Code

1. Read the relevant reference file(s) based on what you are writing
2. Follow naming conventions for all identifiers
3. Use the correct project structure and layering pattern
4. Apply OOP rules for POJOs, service classes, and data models
5. Use proper exception handling and logging from the start
6. Write unit tests following the test conventions

### When Reviewing Java Code

1. Check the Quick-Reference Top 27 rules first - these are the most common violations
2. For each code area, read the corresponding reference file and verify compliance
3. Categorize violations as:
   - **Mandatory**: Must be fixed. These prevent bugs, security issues, or serious maintainability problems.
   - **Recommended**: Should be fixed. These improve code quality but are not critical.
   - **Reference**: Nice to have. These are suggestions for better code style.

### When Refactoring Java Code

1. Prioritize fixing Mandatory violations first
2. Apply naming convention fixes across the entire codebase consistently
3. Refactor thread-unsafe patterns (SimpleDateFormat, improper thread pools, missing ThreadLocal cleanup)
4. Add proper exception handling and logging

## Common Anti-Patterns to Flag

These patterns almost always indicate a violation:

```java
// WRONG: Executors for thread pools
ExecutorService pool = Executors.newFixedThreadPool(10);

// WRONG: SimpleDateFormat as static field
private static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");

// WRONG: is prefix for boolean POJO field
private boolean isSuccess;

// WRONG: Catching Exception broadly
catch (Exception e) { }

// WRONG: String concatenation in log
log.info("User " + userId + " logged in");

// WRONG: foreach with remove
for (String item : list) {
    list.remove(item);
}

// WRONG: NPE-prone equals
name.equals("test");

// WRONG: Magic number
if (status == 3) { ... }
```

## Corrected Patterns

```java
// CORRECT: ThreadPoolExecutor directly
ExecutorService pool = new ThreadPoolExecutor(
    10, 10, 60L, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(1000),
    new ThreadFactoryBuilder().setNameFormat("pool-%d").build(),
    new ThreadPoolExecutor.CallerRunsPolicy()
);

// CORRECT: DateTimeFormatter (thread-safe)
private static final DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd");

// CORRECT: No is prefix for boolean
private boolean success;

// CORRECT: Catch specific exception
catch (IOException e) {
    log.error("IO error reading file: {}", filePath, e);
}

// CORRECT: Parameterized logging
log.info("User {} logged in", userId);

// CORRECT: Iterator for removal
list.removeIf(item -> condition);

// CORRECT: Null-safe equals
Objects.equals(name, "test");
// or
"test".equals(name);

// CORRECT: Named constant
if (status == OrderStatus.COMPLETED) { ... }
```

## Version Note

This skill is based on the Alibaba Java Coding Guidelines v1.7.1 (Huangshan Edition) and subsequent updates. The guidelines are continuously evolving - when in doubt, refer to the latest official publication from Alibaba.