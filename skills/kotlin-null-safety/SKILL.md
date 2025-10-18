---
name: kotlin-null-safety
description: Use when working with potentially null values, preventing NullPointerException, or handling optional data in Kotlin - covers nullable types, safe calls, Elvis operator, and type checking
---

# Kotlin Null Safety

## Overview

Kotlin eliminates most null pointer errors at compile time through nullable types and safe operators.

**Core principle:** Null values must be explicitly declared and safely handled - the compiler enforces this.

## Nullable Types

**Non-nullable by default:**
```kotlin
var name: String = "Alice"
name = null  // Compiler error
```

**Nullable type with `?`:**
```kotlin
var name: String? = "Alice"
name = null  // OK
```

**Type inference respects nullability:**
```kotlin
val x = "text"      // Type: String (non-nullable)
val y: String? = x  // Type: String? (nullable)
```

## Safe Operations

| Operator | Purpose | Example |
|----------|---------|---------|
| `?.` | Safe call - returns `null` if receiver is `null` | `name?.length` |
| `?:` | Elvis - provides default if left is `null` | `name ?: "Guest"` |
| `!!` | Not-null assertion - throws if `null` | `name!!.length` |
| `as?` | Safe cast - returns `null` on failure | `obj as? String` |

## Quick Patterns

**Safe call:**
```kotlin
val length: Int? = name?.length
// Returns null if name is null
```

**Elvis operator for defaults:**
```kotlin
val displayName = name ?: "Guest"
// Uses "Guest" if name is null
```

**Chain safe calls:**
```kotlin
val country = user?.company?.address?.country
// Returns null if any part is null
```

**Safe call with let:**
```kotlin
name?.let {
    println("Name: $it")
}
// Only executes if name is not null
```

**Early return with Elvis:**
```kotlin
fun process(input: String?): Int {
    val value = input ?: return -1
    return value.length
}
```

**Safe cast:**
```kotlin
val str: String? = obj as? String
// Returns null if obj is not a String
```

## Common Mistakes

**❌ Using `!!` unnecessarily:**
```kotlin
val length = name!!.length  // Crashes if name is null
```
**✅ Use safe call or check first:**
```kotlin
val length = name?.length ?: 0
```

**❌ Verbose null checks:**
```kotlin
if (name != null) {
    println(name.length)
}
```
**✅ Use safe call with let:**
```kotlin
name?.let { println(it.length) }
```

**❌ Nested null checks:**
```kotlin
if (user != null) {
    if (user.address != null) {
        println(user.address.city)
    }
}
```
**✅ Chain safe calls:**
```kotlin
println(user?.address?.city)
```

**❌ Unnecessary null check:**
```kotlin
val name: String = "Alice"
if (name != null) {  // Always true, unnecessary
    println(name)
}
```
**✅ Trust non-nullable types:**
```kotlin
val name: String = "Alice"
println(name)  // No check needed
```

## Type Checking

**Check type with `is`:**
```kotlin
when (obj) {
    is String -> println("String: $obj")
    is Int -> println("Int: $obj")
    else -> println("Unknown")
}
```

**Check NOT a type with `!is`:**
```kotlin
if (obj !is String) {
    return
}
// obj is smart cast to String here
```

## Collections and Null

**Filter out nulls:**
```kotlin
val list: List<String?> = listOf("a", null, "b")
val valid: List<String> = list.filterNotNull()
// [a, b]
```

**Create list without nulls:**
```kotlin
val items = listOfNotNull(a, null, b, null, c)
// Only non-null values included
```

**Find max/min safely:**
```kotlin
val max = numbers.maxOrNull() ?: 0
val min = numbers.minOrNull() ?: 0
```

## When NOT to Use

**Don't use nullable types when:**
- Value is always present (use non-nullable)
- Better to throw exception than return `null`
- API contract guarantees non-null

**Don't use `!!` when:**
- You can use safe call `?.` instead
- Elvis operator `?:` provides better alternative
- You haven't checked for `null` first

## Key Insights

- **Nullable types are explicit** - `String` vs `String?` are different types
- **Compiler enforces safety** - Can't call methods on nullable without check
- **Smart casts** - After `null` check, compiler auto-casts to non-nullable
- **Safe by default** - Most types are non-nullable, opt-in to nullability
- **Prefer `?.` and `?:` over `!!`** - Avoid runtime crashes

