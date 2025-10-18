---
name: kotlin-control-flow
description: Use when making decisions, iterating over collections or ranges, or controlling execution flow in Kotlin - covers conditionals like if and when, loops like for and while, and ranges
---

# Kotlin Control Flow

## Overview

Kotlin provides `when` expressions, `if` as an expression, and powerful range-based loops for clean control flow.

**Core principle:** Prefer `when` over `if-else` chains and use expressions over statements when possible.

## if as Expression

**Returns a value:**
```kotlin
val max = if (a > b) a else b
```

**Multi-line:**
```kotlin
val result = if (x > 0) {
    println("Positive")
    x
} else {
    println("Non-positive")
    0
}
```

**No ternary operator** - `if` serves this purpose.

## when Expression

**Prefer `when` over `if-else` chains:**

**With subject:**
```kotlin
val result = when (x) {
    1 -> "one"
    2 -> "two"
    3, 4 -> "three or four"
    in 5..10 -> "five to ten"
    else -> "other"
}
```

**Without subject (boolean conditions):**
```kotlin
when {
    x < 0 -> println("Negative")
    x == 0 -> println("Zero")
    x > 0 -> println("Positive")
}
```

**Type checking:**
```kotlin
when (obj) {
    is String -> println("String: $obj")
    is Int -> println("Int: $obj")
    else -> println("Unknown")
}
```

**Smart casts in when:**
```kotlin
when (obj) {
    is String -> println(obj.length)  // obj auto-cast to String
    is List<*> -> println(obj.size)   // obj auto-cast to List
}
```

## Ranges

**Inclusive range (`..`):**
```kotlin
1..5        // 1, 2, 3, 4, 5
'a'..'z'    // a through z
```

**Exclusive range (`..<`):**
```kotlin
1..<5       // 1, 2, 3, 4 (excludes 5)
```

**Reverse range:**
```kotlin
5 downTo 1  // 5, 4, 3, 2, 1
```

**Step:**
```kotlin
1..10 step 2  // 1, 3, 5, 7, 9
10 downTo 1 step 3  // 10, 7, 4, 1
```

## for Loops

**Iterate range:**
```kotlin
for (i in 1..5) {
    println(i)
}
```

**Iterate collection:**
```kotlin
val items = listOf("a", "b", "c")
for (item in items) {
    println(item)
}
```

**With index:**
```kotlin
for (i in items.indices) {
    println("$i: ${items[i]}")
}

// Or use withIndex()
for ((index, value) in items.withIndex()) {
    println("$index: $value")
}
```

**Iterate map:**
```kotlin
val map = mapOf("a" to 1, "b" to 2)
for ((key, value) in map) {
    println("$key: $value")
}
```

## while Loops

**Standard while:**
```kotlin
var i = 0
while (i < 5) {
    println(i)
    i++
}
```

**do-while (executes at least once):**
```kotlin
var i = 0
do {
    println(i)
    i++
} while (i < 5)
```

## Quick Patterns

| Task | Pattern |
|------|---------|
| Max of two | `if (a > b) a else b` |
| Multiple conditions | `when { ... }` |
| Type check | `when (x) { is Type -> ... }` |
| Range iteration | `for (i in 1..10) { }` |
| Collection iteration | `for (item in list) { }` |
| Map iteration | `for ((k, v) in map) { }` |
| Reverse iteration | `for (i in 10 downTo 1) { }` |
| Step iteration | `for (i in 0..100 step 10) { }` |

## Common Mistakes

**❌ Using if-else chain:**
```kotlin
val color = if (x == 1) "red"
    else if (x == 2) "green"
    else if (x == 3) "blue"
    else "unknown"
```
**✅ Use when:**
```kotlin
val color = when (x) {
    1 -> "red"
    2 -> "green"
    3 -> "blue"
    else -> "unknown"
}
```

**❌ Forgetting else in when expression:**
```kotlin
val result = when (x) {
    1 -> "one"
    2 -> "two"
    // Compiler error: when must be exhaustive
}
```
**✅ Add else or cover all cases:**
```kotlin
val result = when (x) {
    1 -> "one"
    2 -> "two"
    else -> "other"
}
```

**❌ Using while when for is clearer:**
```kotlin
var i = 0
while (i < items.size) {
    println(items[i])
    i++
}
```
**✅ Use for loop:**
```kotlin
for (item in items) {
    println(item)
}
```

**❌ Inclusive range when exclusive needed:**
```kotlin
for (i in 0..array.size) {  // Off-by-one error
    array[i]
}
```
**✅ Use exclusive range or indices:**
```kotlin
for (i in 0..<array.size) { }
// or
for (i in array.indices) { }
```

## Advanced when

**Multiple values per branch:**
```kotlin
when (x) {
    0, 1 -> println("small")
    in 2..10 -> println("medium")
    else -> println("large")
}
```

**Ranges and contains:**
```kotlin
when (x) {
    in 1..10 -> "single digit or 10"
    in validValues -> "valid"
    !in 20..30 -> "not in 20-30"
    else -> "other"
}
```

**Capture subject in variable:**
```kotlin
when (val response = getResponse()) {
    is Success -> println(response.data)
    is Error -> println(response.message)
}
// response only in scope of when
```

## Loop Control

**break - exit loop:**
```kotlin
for (i in 1..10) {
    if (i == 5) break
    println(i)  // Prints 1, 2, 3, 4
}
```

**continue - skip iteration:**
```kotlin
for (i in 1..5) {
    if (i == 3) continue
    println(i)  // Prints 1, 2, 4, 5
}
```

**Labels for nested loops:**
```kotlin
outer@ for (i in 1..3) {
    for (j in 1..3) {
        if (i == 2 && j == 2) break@outer
        println("$i,$j")
    }
}
```

## Key Insights

- **`when` over `if-else`** - More readable, easier to maintain
- **Expressions** - `if` and `when` return values
- **Smart casts** - Type checks enable auto-casting
- **Ranges** - Concise iteration with `..`, `..<`, `downTo`, `step`
- **Exhaustive when** - Compiler ensures all cases handled
- **No fallthrough** - `when` branches don't fall through (unlike `switch`)

