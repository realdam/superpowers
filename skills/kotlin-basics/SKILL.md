---
name: kotlin-basics
description: Use when starting with Kotlin, declaring variables, working with basic types, or using string interpolation - covers variables, types, string templates, and core syntax fundamentals
---

# Kotlin Basics

## Overview

Master Kotlin fundamentals: immutable-by-default variables, type inference, and expressive string templates.

**Core principle:** Prefer `val` (read-only) over `var` (mutable) - mutability should be explicit and intentional.

## Variables

**Declare read-only variables with `val`:**
```kotlin
val name = "Alice"
val count = 42
```

**Declare mutable variables with `var`:**
```kotlin
var score = 0
score = 10  // Allowed
```

**Explicit types (optional with type inference):**
```kotlin
val year: Int = 2025
val message: String = "Hello"
```

**Delayed initialization:**
```kotlin
val result: Int
result = computeValue()  // Must initialize before first read
```

## Basic Types

| Category | Types | Example |
|----------|-------|---------|
| Integers | `Byte`, `Short`, `Int`, `Long` | `val count: Int = 42` |
| Unsigned | `UByte`, `UShort`, `UInt`, `ULong` | `val score: UInt = 100u` |
| Floats | `Float`, `Double` | `val pi: Double = 3.14` |
| Boolean | `Boolean` | `val isActive = true` |
| Characters | `Char` | `val initial = 'A'` |
| Strings | `String` | `val text = "Hello"` |

**Type inference works:**
```kotlin
val x = 42        // Int inferred
val y = 3.14      // Double inferred
val z = "text"    // String inferred
```

## String Templates

**Variable interpolation with `$`:**
```kotlin
val name = "Alice"
println("Hello, $name")  // Hello, Alice
```

**Expression interpolation with `${}`:**
```kotlin
val count = 5
println("Total: ${count + 1}")  // Total: 6
```

**Access properties:**
```kotlin
val user = User("Bob", 25)
println("${user.name} is ${user.age} years old")
```

## Quick Patterns

| Task | Pattern |
|------|---------|
| Immutable variable | `val name = "Alice"` |
| Mutable variable | `var count = 0` |
| Explicit type | `val age: Int = 25` |
| Delayed init | `val x: String; x = "value"` |
| String template | `"Hello, $name"` |
| Expression in string | `"Total: ${x + y}"` |
| Arithmetic | `+=`, `-=`, `*=`, `/=`, `%=` |

## Common Mistakes

**❌ Using `var` by default:**
```kotlin
var name = "Alice"  // Unnecessarily mutable
```
**✅ Use `val` unless mutation is required:**
```kotlin
val name = "Alice"  // Immutable, safer
```

**❌ Reading uninitialized variable:**
```kotlin
val x: Int
println(x)  // Error: Variable 'x' must be initialized
```
**✅ Initialize before first read:**
```kotlin
val x: Int
x = 42
println(x)  // OK
```

**❌ Forgetting curly braces for expressions:**
```kotlin
println("Result: $x + $y")  // Result: 5 + 3
```
**✅ Use `${}` for expressions:**
```kotlin
println("Result: ${x + y}")  // Result: 8
```

## Entry Point

**Every Kotlin program starts with `main()`:**
```kotlin
fun main() {
    println("Hello, Kotlin!")
}
```

**Main with arguments:**
```kotlin
fun main(args: Array<String>) {
    println("Args: ${args.joinToString()}")
}
```

## Key Insights

- **Immutability by default** - Use `val` unless you need `var`
- **Type inference** - Compiler infers types, explicit types optional
- **String templates** - Use `$variable` or `${expression}` for interpolation
- **No semicolons** - Semicolons are optional in Kotlin
- **Type safety** - Types must match, enforced at compile time

