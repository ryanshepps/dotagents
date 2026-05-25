---
slug: writing-kotlin
categories: [languages]
priority: 1
description: Kotlin style — null safety, data classes, sealed hierarchies, coroutines, error handling, testing.
applies_when:
  - writing Kotlin code
  - reviewing Kotlin code
  - designing Kotlin APIs
  - using coroutines
related: [coding-style, writing-tests]
source: https://kotlinlang.org/docs/coding-conventions.html
---

# Kotlin

## Null Safety

Treat nullable types as a precise part of the domain model, not as a default.
Use non-null types unless absence is a valid state the caller must handle.

```kotlin
// WRONG: every caller has to guess which nulls are meaningful
data class User(val id: String?, val email: String?)

// CORRECT: absence is modeled only where it is real
data class User(val id: UserId, val email: Email?)
```

Rules:
- Never use `!!` in production code unless proving an invariant at a boundary; prefer `requireNotNull`, explicit branching, or a narrower type
- Use safe calls and Elvis (`?.`, `?:`) for small local nullable flows; use sealed states when null changes behavior
- Do not return `null` for expected failure -- return a sealed result type or throw a domain exception
- When consuming Java platform types, declare the Kotlin type explicitly at public boundaries
- Prefer empty collections over nullable collections: `List<Item>` with `emptyList()` beats `List<Item>?`

## Type Design

Use `data class` for immutable value carriers. Constructor properties should be
`val` by default; reach for `var` only when mutation is part of the abstraction.

```kotlin
data class Money(val cents: Long, val currency: Currency)
```

Use value classes to wrap primitives with domain meaning:

```kotlin
@JvmInline
value class UserId(val value: String)
```

### Make Invalid States Unrepresentable

Use sealed interfaces or sealed classes for closed variants, especially state
machines, API results, and UI states.

```kotlin
// WRONG: fields are meaningful only for some statuses
data class Order(
    val status: String,
    val shippedAt: Instant?,
    val tracking: String?,
)

// CORRECT: each state carries exactly its own data
sealed interface Order {
    data class Pending(val items: List<Item>) : Order
    data class Shipped(
        val items: List<Item>,
        val shippedAt: Instant,
        val tracking: TrackingNumber,
    ) : Order
}
```

When switching on sealed types, make `when` exhaustive and avoid an `else`
branch. Let the compiler tell you when a new variant needs handling.

### Enums and Objects Over Booleans

```kotlin
// WRONG: what does true mean?
fun send(message: Message, urgent: Boolean)

// CORRECT: the call site explains itself
enum class Priority { Normal, Urgent }
fun send(message: Message, priority: Priority)
```

Use `data object` for singleton variants in sealed hierarchies when equality and
readable output matter.

## Error Handling

Kotlin exceptions are unchecked. Throw exceptions for unexpected failures and
programmer errors; model expected business outcomes as values.

```kotlin
sealed interface CreateUserResult {
    data class Created(val user: User) : CreateUserResult
    data object EmailTaken : CreateUserResult
    data class InvalidInput(val reason: String) : CreateUserResult
}
```

Rules:
- Define module-specific exception types with useful fields, not just messages
- Use `require` for caller preconditions and `check` for internal state invariants
- Catch specific exceptions only; never catch `Throwable` in business logic
- Preserve causes: `throw ConfigError("invalid config at $path", cause)`
- Keep `runCatching` at narrow synchronous boundaries; do not let it swallow coroutine cancellation

## Coroutines

Prefer suspending APIs over callback, future, or "start and return a Job" APIs.
Concurrency should be explicit and structured.

```kotlin
suspend fun loadProfile(id: UserId): Profile = coroutineScope {
    val user = async { userRepository.get(id) }
    val settings = async { settingsRepository.get(id) }
    Profile(user.await(), settings.await())
}
```

Rules:
- Use `coroutineScope` when child work must succeed or fail together
- Use `supervisorScope` only when sibling failures are intentionally independent
- Do not launch unowned background work from business logic; the caller should own the scope
- Use `async` only for true concurrency; sequential suspending calls are the default
- Move blocking I/O behind `withContext(Dispatchers.IO)` or an injected dispatcher
- Re-throw `CancellationException`; swallowing it breaks cancellation propagation
- In CPU loops, cooperate with cancellation using `ensureActive()`, `isActive`, or `yield()`
- Use `NonCancellable` only for short cleanup that must finish during cancellation

## Scope Functions

Scope functions are for local clarity, not style points. Choose by intent:

- `let` for nullable flow or introducing a short local expression
- `apply` for configuring a newly created object
- `also` for side effects such as logging or metrics
- `run` for computing a result from a receiver
- `with` for grouping calls on an existing object

Avoid nesting scope functions. If `this` and `it` become ambiguous, name the
intermediate value and write ordinary code.

## Naming Conventions

| Item | Convention | Example |
|---|---|---|
| Packages | lowercase, no underscores | `com.example.billing` |
| Files with one public type | Type name | `InvoiceService.kt` |
| Files with top-level declarations | UpperCamelCase description | `InvoiceParsing.kt` |
| Classes, interfaces, objects | UpperCamelCase | `HttpClient` |
| Functions, properties, variables | lowerCamelCase | `parseConfig` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_RETRIES` |
| Type parameters | Single uppercase or short PascalCase | `T`, `RequestT` |
| Test methods | backtick sentence or lowerCamelCase | `` `rejects blank email` `` |

Naming patterns:
- Predicate properties and functions start with `is`, `has`, `can`, or `should`
- Conversions use `toX()` for new values and `asX()` for cheap views
- Factory functions may share the abstract return type name when that improves call sites
- Avoid `Util` files and classes; name files after the domain operation they contain

## Testing

Use JUnit 5 for JVM projects unless the repository has already standardized on
Kotest. Kotlin test names may use backticks for readable behavior statements.

```kotlin
class EmailValidatorTest {
    @Test
    fun `rejects blank email`() {
        val result = EmailValidator.validate("")

        assertEquals(ValidationResult.Blank, result)
    }
}
```

Rules:
- Test behavior through public APIs, not implementation details
- Use parameterized tests for repeated input/output cases
- Prefer fakes over mocks for dependencies with meaningful behavior
- Never mock what you do not own; wrap third-party APIs behind an interface you control
- Test coroutine code with `kotlinx-coroutines-test`; do not use sleeps as synchronization
- Assert cancellation and failure paths as deliberately as success paths

## Package Organization

Follow the package directory structure. In pure Kotlin projects, omit the common
root package from the source path; in mixed Java/Kotlin JVM projects, keep both
languages in the same source root and package layout.

Organize by domain or feature, not by technical layer:

```text
src/main/kotlin/com/example/billing/
  Invoice.kt
  InvoiceRepository.kt
  InvoiceService.kt
  PricingPolicy.kt
src/test/kotlin/com/example/billing/
  InvoiceServiceTest.kt
```

Keep extension functions close to the type or domain they serve. Public
extension functions are API surface; avoid clever global extensions on common
types like `String`, `List`, or `Any`.

## Public APIs and KDoc

Document public APIs with KDoc when they cross module, package, or user-facing
boundaries. The first paragraph should summarize behavior. Use `@param`,
`@property`, `@return`, and `@throws` only when they add information not already
obvious from the signature.

Rules:
- Prefer explicit return types on public functions and properties
- Keep visibility narrow; use `internal` for module-only APIs
- Avoid exposing mutable collections; return read-only interfaces and copy at boundaries
- Do not leak implementation-specific coroutine scopes, dispatchers, or platform types through public APIs
