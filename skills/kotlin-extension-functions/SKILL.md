---
name: kotlin-extension-functions
description: Use when adding utility functions to classes you don't control, creating domain-specific APIs, or enhancing third-party libraries in Kotlin - extends existing classes without inheritance or modification
---

# Kotlin Extension Functions

## Overview

Extension functions let you add new functions to existing classes without modifying source code or using inheritance.

**Core principle:** Extend classes with focused, single-purpose functions - keep core functionality separate from convenience features.

## Basic Syntax

**Define extension function:**
```kotlin
fun String.bold(): String = "<b>$this</b>"

println("hello".bold())  // <b>hello</b>
```

In this example:
- `String` is the receiver type (class being extended)
- `bold` is the extension function name
- `this` refers to the receiver object ("hello")

## Extension-Oriented Design

**Separate core from convenience:**
```kotlin
// Core functionality
class HttpClient {
    fun request(method: String, url: String): Response {
        // Network code
    }
}

// Extension functions for common use cases
fun HttpClient.get(url: String) = request("GET", url)
fun HttpClient.post(url: String) = request("POST", url)

val client = HttpClient()
client.get("https://api.example.com")  // Cleaner API
```

This pattern:
- Keeps core class simple
- Adds convenient shortcuts
- Makes API more readable
- No inheritance needed

## Extension Properties

**Add computed property:**
```kotlin
val String.lastChar: Char
    get() = this[length - 1]

println("hello".lastChar)  // o
```

**Must define getter:**
```kotlin
val List<Int>.secondOrNull: Int?
    get() = if (size >= 2) this[1] else null
```

**Note:** Extension properties can't have backing fields.

## Quick Patterns

| Task | Pattern |
|------|---------|
| Extend class | `fun ClassName.funcName() { }` |
| Access receiver | Use `this` keyword |
| Extension property | Must provide `get()` |
| Generic extension | `fun <T> List<T>.second() = this[1]` |
| Nullable receiver | `fun String?.orEmpty() = this ?: ""` |

## Common Extensions

**String extensions:**
```kotlin
fun String.isEmail() = contains("@") && contains(".")
fun String.truncate(maxLength: Int) = 
    if (length > maxLength) take(maxLength) + "..." else this

"user@example.com".isEmail()  // true
"Long text".truncate(5)        // Long...
```

**Collection extensions:**
```kotlin
fun <T> List<T>.secondOrNull(): T? = 
    if (size >= 2) this[1] else null

fun List<Int>.average() = sum() / size.toDouble()

listOf(1, 2, 3).secondOrNull()  // 2
listOf(10, 20, 30).average()    // 20.0
```

**Generic extensions:**
```kotlin
fun <T> T.also(block: (T) -> Unit): T {
    block(this)
    return this
}

val result = computeValue()
    .also { println("Computed: $it") }
    .also { validate(it) }
```

## Receiver Types

**Nullable receiver:**
```kotlin
fun String?.orDefault(default: String = "") = this ?: default

val name: String? = null
println(name.orDefault("Guest"))  // Guest
```

**Generic receiver:**
```kotlin
fun <T> T.print(): T {
    println(this)
    return this
}

42.print().print()  // Prints 42 twice, returns 42
```

## Scope and Resolution

**Extensions don't override:**
```kotlin
class C {
    fun foo() { println("Member") }
}

fun C.foo() { println("Extension") }

C().foo()  // Member - member wins
```

**Member functions always take precedence over extensions.**

**Extensions are resolved statically:**
```kotlin
open class Shape
class Circle : Shape()

fun Shape.name() = "Shape"
fun Circle.name() = "Circle"

val shape: Shape = Circle()
println(shape.name())  // Shape - uses declared type, not runtime type
```

## Common Mistakes

**❌ Trying to access private members:**
```kotlin
class User {
    private val secret = "hidden"
}

fun User.revealSecret() = secret  // Compiler error - can't access private
```
**✅ Extensions only access public API:**
```kotlin
fun User.greet() = "Hello, ${this.toString()}"
```

**❌ Extension with backing field:**
```kotlin
val String.cached: String
    field = this  // Error: extensions can't have backing fields
```
**✅ Use computed property:**
```kotlin
val String.cached: String
    get() = this
```

**❌ Expecting polymorphic behavior:**
```kotlin
open class Base
class Derived : Base()

fun Base.foo() = "Base"
fun Derived.foo() = "Derived"

val x: Base = Derived()
println(x.foo())  // Base - not polymorphic!
```
**✅ Use member functions for polymorphism:**
```kotlin
open class Base {
    open fun foo() = "Base"
}
class Derived : Base() {
    override fun foo() = "Derived"
}
```

**❌ Overusing extensions for everything:**
```kotlin
fun User.setName(name: String) { /* can't modify private field */ }
```
**✅ Extensions for utilities, not core behavior:**
```kotlin
fun User.toJson(): String = """{"name":"$name","id":$id}"""
```

## When to Use Extensions

**Use extensions for:**
- Utility functions on third-party classes
- Domain-specific operations
- Convenience shortcuts
- Adapter patterns
- DSL builders

**Don't use extensions for:**
- Core class behavior (use member functions)
- State modification (prefer member functions)
- Functionality needing private access
- When polymorphism is required

## Standard Library Examples

Kotlin's standard library heavily uses extensions:

```kotlin
// String extensions
"hello".capitalize()
"  text  ".trim()
"kotlin".reversed()

// Collection extensions
listOf(1, 2, 3).sum()
listOf(1, 2, 3).average()
listOf("a", "b").joinToString()

// Scope function extensions
user.apply { email = "new@example.com" }
value.let { it * 2 }
```

## Quick Reference

| Feature | Syntax | Notes |
|---------|--------|-------|
| Function | `fun Type.func()` | Called with `.` on receiver |
| Property | `val Type.prop get()` | Must define getter |
| Nullable receiver | `fun Type?.func()` | Can be called on `null` |
| Generic | `fun <T> List<T>.func()` | Works with any type |
| Access receiver | `this` | Implicit in extension body |

## Key Insights

- **No modification** - Extend classes without changing source code
- **Static dispatch** - Extensions resolved at compile time
- **Member priority** - Member functions always override extensions
- **No private access** - Extensions use public API only
- **Design pattern** - Separate core functionality from utilities
- **DSL building** - Key for creating readable domain-specific languages

