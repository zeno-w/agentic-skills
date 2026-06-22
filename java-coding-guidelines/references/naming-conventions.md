# Naming Conventions

1. Names must not start/end with `_` or `$`. ❌ `_name` / `name$` / `Object$`
2. No Chinese/Pinyin/mixed naming. Use English. ✅ `alibaba` / `taobao` (internationally recognized names OK)
3. No discriminatory or offensive words.

## Class Names
4. UpperCamelCase. ✅ `ForceCode` / `UserDO` / `HtmlDTO`
5. DO/BO/DTO/VO/AO/PO/UID suffixes allowed. ✅ `UserDO` / `HtmlDTO`
6. Abstract classes: `Abstract`/`Base` prefix. ✅ `BaseUserService`
7. Exception classes: `Exception` suffix. ✅ `BusinessException`
8. Test classes: `{Class}Test`. ✅ `UserServiceTest`

## Method/Parameter/Member/Local Variable Names
9. lowerCamelCase. ✅ `localValue` / `getHttpMessage()`

## Constants
10. UPPER_SNAKE_CASE, semantically complete. ✅ `MAX_STOCK_COUNT` ❌ `MAX_COUNT`
11. Group related constants with shared prefix. ✅ `STOCK_TYPE_IN` / `STOCK_TYPE_OUT`

## Packages
12. All lowercase, dot-separated, single word per segment. ✅ `com.alibaba.openapi` ❌ `com.alibaba.open.api`

## Boolean Fields
13. No `is` prefix (serialization ambiguity). ❌ `boolean isDeleted` ✅ `boolean deleted`
14. POJO boolean getter: `isXxx()`. RPC local boolean: `is` prefix OK.

## Arrays
15. Brackets with type. ✅ `String[] args` ❌ `String args[]`

## Enums
16. UpperCamelCase class, UPPER_SNAKE_CASE values. ✅ `enum ProcessStatus { SUCCESS, FAILED }`

## Service/DAO Naming
17. Interface: `UserService`. Impl: `UserServiceImpl`. DAO: `UserDAO`.

## Method Naming
18. Get single: `getXxx`. List: `listXxx`. Count: `countXxx`. Insert: `saveXxx`/`insertXxx`. Delete: `removeXxx`/`deleteXxx`. Update: `updateXxx`.

## Design Patterns
19. Include pattern name. ✅ `OrderFactory` / `LoginProxy`
20. No `I` prefix on interfaces. ✅ `UserService` ❌ `IUserService`
21. `Impl` suffix on implementations. ✅ `UserServiceImpl`

## Other
22. No names differing only in case. ❌ `name` vs `Name`
23. No generic Map/Set keys: `key`, `value`, `item`.
24. Consistent naming convention across codebase.

## Anti-Patterns
```java
// ❌ is prefix for boolean POJO field
private boolean isSuccess;
```

## Corrected
```java
// ✅ No is prefix
private boolean success;
```