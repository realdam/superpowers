---
name: kotlin-reference
description: Use when looking up Kotlin syntax, common patterns, or quick reference - comprehensive Kotlin syntax guide with quick lookup tables, learning paths, and code examples
---

# Kotlin Reference

## About Kotlin Skills

Pragmatic, minimal skills for writing efficient Kotlin code focusing on practical patterns over academic completeness.

### Skill Organization

**Fundamentals:**
- **kotlin-basics** - Variables, types, string templates
- **kotlin-collections** - Lists, sets, maps
- **kotlin-control-flow** - if, when, loops, ranges
- **kotlin-functions** - Functions, lambdas, higher-order functions

**Core Features:**
- **kotlin-null-safety** - Nullable types, safe calls, Elvis operator
- **kotlin-extension-functions** - Extending classes without inheritance
- **kotlin-scope-functions** - let, apply, run, also, with

**Advanced:**
- **kotlin-classes** - Classes, data classes, objects, inheritance
- **kotlin-advanced** - Lambdas with receiver, properties, delegation

### Learning Path

**Beginner:**
1. kotlin-basics
2. kotlin-collections
3. kotlin-control-flow
4. kotlin-functions
5. kotlin-null-safety

**Intermediate:**
6. kotlin-classes
7. kotlin-extension-functions
8. kotlin-scope-functions

**Advanced:**
9. kotlin-advanced

## Quick Reference

### Variables
```kotlin
val x = 42              // Immutable (preferred)
var y = 10              // Mutable
val z: Int = 5          // Explicit type
```

### Basic Types
```kotlin
val int: Int = 42
val long: Long = 42L
val double: Double = 3.14
val bool: Boolean = true
val str: String = "text"
```

### Strings
```kotlin
val name = "Alice"
"Hello, $name"                    // Variable
"Sum: ${x + y}"                   // Expression
```

### Collections
```kotlin
listOf(1, 2, 3)                   // Read-only list
mutableListOf(1, 2, 3)            // Mutable list
setOf(1, 2, 3)                    // Read-only set
mapOf("a" to 1, "b" to 2)         // Read-only map
```

### Control Flow
```kotlin
// if expression
val max = if (a > b) a else b

// when expression
when (x) {
    1 -> "one"
    in 3..10 -> "3 to 10"
    else -> "other"
}

// for loop
for (i in 1..5) { }
for (item in list) { }
```

### Functions
```kotlin
fun add(a: Int, b: Int) = a + b   // Single expression
val square = { x: Int -> x * x }  // Lambda
list.filter { it > 0 }            // Trailing lambda
```

### Null Safety
```kotlin
var nullable: String? = null      // Nullable type
name?.length                      // Safe call
name?.length ?: 0                 // Elvis operator
```

### Classes
```kotlin
class User(val name: String)      // Basic class
data class User(val name: String) // Data class
object Config { }                 // Singleton
```

### Extension Functions
```kotlin
fun String.isPalindrome() = this == reversed()
"racecar".isPalindrome()          // true
```

### Scope Functions
```kotlin
name?.let { sendEmail(it) }       // Null check
User().apply { email = "a@b.com" } // Config
list.also { println("Size: ${it.size}") } // Side effect
```

## Index by Symptom

| Problem | Skill |
|---------|-------|
| NullPointerException | kotlin-null-safety |
| Can't modify list after creation | kotlin-collections (use mutable) |
| Need to add function to String class | kotlin-extension-functions |
| Verbose null checks | kotlin-null-safety (safe calls) |
| Long if-else chains | kotlin-control-flow (use when) |
| Can't decide which scope function | kotlin-scope-functions |
| Boilerplate equals/toString | kotlin-classes (data classes) |
| Need singleton | kotlin-classes (object) |
| Property initialization expensive | kotlin-advanced (lazy) |
| Want to track property changes | kotlin-advanced (observable) |

## Index by Task

| Task | Skill |
|------|-------|
| Store multiple items | kotlin-collections |
| Make decisions | kotlin-control-flow |
| Iterate over range | kotlin-control-flow |
| Define reusable logic | kotlin-functions |
| Handle optional values | kotlin-null-safety |
| Configure object | kotlin-scope-functions (apply) |
| Transform with null check | kotlin-scope-functions (let) |
| Add logging to pipeline | kotlin-scope-functions (also) |
| Extend third-party class | kotlin-extension-functions |
| Create data structure | kotlin-classes |
| Build DSL | kotlin-advanced (lambdas with receiver) |

## Index by Keyword

**Variables & Types:**
`val`, `var`, `Int`, `String`, `Boolean` → kotlin-basics

**Collections:**
`List`, `Set`, `Map`, `listOf`, `mutableListOf` → kotlin-collections
`map`, `filter`, `forEach`, `fold` → kotlin-functions, kotlin-collections

**Control Flow:**
`if`, `when`, `for`, `while`, `in`, `..` → kotlin-control-flow

**Functions:**
`fun`, `lambda`, `->`, `it`, `return` → kotlin-functions

**Null Safety:**
`?`, `?.`, `?:`, `!!`, `as?`, `is` → kotlin-null-safety

**Classes:**
`class`, `data class`, `object`, `companion`, `interface` → kotlin-classes
`abstract`, `open`, `override`, `sealed`, `enum` → kotlin-classes

**Extensions:**
`fun Type.ext()`, `val Type.prop` → kotlin-extension-functions

**Scope Functions:**
`let`, `apply`, `run`, `also`, `with` → kotlin-scope-functions

**Advanced:**
`by lazy`, `by observable`, `Type.() -> Unit` → kotlin-advanced
`get()`, `set()`, `field` → kotlin-advanced

## Common Patterns

### Prefer
- `val` over `var`
- `when` over `if-else` chains
- Data classes for value objects
- Extension functions for utilities
- Scope functions for object manipulation
- Lambdas over anonymous classes
- `?.` and `?:` over explicit null checks
- Read-only collections by default

### Avoid
- Unnecessary mutability
- `!!` not-null assertions
- Open classes without reason
- Verbose null checks
- Manual `equals`/`toString`/`copy` for data classes

## Gotchas

- Collections are read-only by default (not immutable)
- `when` must be exhaustive for expressions
- Extensions don't override member functions
- Extension properties can't have backing fields
- Ranges are inclusive (`..`) unless using `..<`
- `lateinit` only for `var`, not `val`
- Companion objects aren't static (just singletons)

## Use Cases

**Starting new Kotlin project:**
1. kotlin-basics
2. kotlin-collections
3. kotlin-functions

**Working with existing Java code:**
- kotlin-null-safety (handle Java nulls)
- kotlin-extension-functions (enhance Java APIs)

**Building library/API:**
- kotlin-extension-functions (design patterns)
- kotlin-advanced (DSLs)
- kotlin-classes (public API)

**Data processing:**
- kotlin-collections
- kotlin-functions (lambdas, map/filter)
- kotlin-null-safety (handle missing data)

**Configuration/setup:**
- kotlin-scope-functions (apply, also)
- kotlin-advanced (DSL builders)

**Type-safe domain modeling:**
- kotlin-classes (sealed classes)
- kotlin-null-safety
- kotlin-advanced (properties)

