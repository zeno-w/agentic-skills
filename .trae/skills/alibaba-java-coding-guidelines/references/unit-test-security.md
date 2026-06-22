# Unit Test and Security Conventions

## Unit Test Rules

### 1. Test Coverage

- Core business logic must have unit tests. Target coverage >= 70% for new code.

### 2. Test Naming

- Test class: `XxxTest`
- Test method: `test_methodName_scenario_expectedResult`
  - Positive example: `test_getUserById_whenUserExists_returnsUser`

### 3. Test Independence

- Tests must be independent. No dependencies between test methods.
- Each test should set up its own data and clean up after itself.

### 4. Mock vs Real

- Use Mock (Mockito) for external dependencies (database, RPC, HTTP).
- Do not mock the class under test.

### 5. Assertions

- Use meaningful assertions. Do not use `assertTrue(true)` or empty assertions.
- Use `assertEquals(expected, actual)` with a descriptive message.

### 6. Boundary Testing

- Test boundary conditions: null, empty, max, min, negative, zero.
- Test exception paths, not just happy paths.

### 7. Test Data

- Do not rely on pre-existing data in the database.
- Use `@BeforeEach` / `@BeforeAll` to prepare test data.
- Clean up test data after tests.

### 8. No Sleep in Tests

- Do not use `Thread.sleep()` in tests. Use `Awaitility` or similar for async testing.

### 9. Integration Tests

- Mark integration tests with `@SpringBootTest` and a separate profile.
- Integration tests should not run in the default `mvn test` phase.

## Security Conventions

### 1. SQL Injection Prevention

- Always use parameterized queries (`#{}` in MyBatis).
- Never concatenate user input into SQL strings.
- Validate and whitelist `ORDER BY` column names.

### 2. XSS Prevention

- Escape all user input before rendering in HTML.
- Use framework-provided escaping (e.g., Thymeleaf auto-escaping).

### 3. CSRF Prevention

- Use CSRF tokens for state-changing requests (POST, PUT, DELETE).

### 4. Authentication and Authorization

- All API endpoints must have proper authentication checks.
- Use role-based access control (RBAC).
- Principle of least privilege: grant minimum necessary permissions.

### 5. Sensitive Data

- Do not log passwords, tokens, ID numbers, bank card numbers.
- Mask sensitive data in responses: `phone: 138****1234`.
- Encrypt passwords using BCrypt or Argon2. Do not use MD5/SHA1.
- Encrypt configuration file passwords.

### 6. File Upload

- Validate file size, type, and content.
- Do not trust the client-provided filename or content-type.
- Store uploaded files outside the web root.
- Generate new filenames (UUID) instead of using user-provided filenames.

### 7. HTTPS

- Use HTTPS for all production APIs.
- Disable HTTP access in production.

### 8. Dependency Security

- Regularly scan dependencies for known vulnerabilities.
- Keep dependencies up to date.
- Use OWASP Dependency-Check or similar tools.

### 9. Error Messages

- Do not expose stack traces or internal details in error responses.
- Return generic error messages to clients. Log detailed errors server-side.

### 10. API Rate Limiting

- Implement rate limiting for public APIs.
- Use exponential backoff for retries.

### 11. Session Management

- Use secure, HTTP-only, SameSite cookies.
- Set appropriate session timeout.
- Invalidate sessions on logout.

### 12. Input Validation

- Validate all user input on the server side, even if client-side validation exists.
- Use Bean Validation (JSR-380 / `javax.validation`) annotations.
- Define maximum length for all string inputs.