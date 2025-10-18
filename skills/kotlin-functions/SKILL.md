---
name: kotlin-functions
description: Use when defining reusable logic, working with higher-order functions, or using functional programming patterns in Kotlin - covers function declaration, parameters, lambdas, and single-expression functions
---

# Kotlin Functions

## Overview

Functions in Kotlin are first-class citizens with concise syntax, lambda expressions, and powerful functional programming support.

**Core principle:** Use single-expression functions and lambdas to write concise, readable code.

## Function Declaration

**Basic function:**
```kotlin
fun greet(name: String): String {
    return "Hello, $name"
}
```

**Function without return value:**
```kotlin
fun log(message: String) {
    println(message)
    // Return type Unit is implicit
}
```

**Single-expression function:**
```kotlin
fun sum(a: Int, b: Int) = a + b
// Return type inferred, curly braces optional
```

**Explicit return type (recommended for clarity):**
```kotlin
fun sum(a: Int, b: Int): Int = a + b
```

## Parameters

**Named arguments:**
```kotlin
fun greet(name: String, title: String) {
    println("$title $name")
}

greet(title = "Dr.", name = "Smith")  // Any order
```

**Default parameters:**
```kotlin
fun connect(host: String, port: Int = 8080) {
    println("Connecting to $host:$port")
}

connect("localhost")           // Uses default port
connect("localhost", 9000)     // Overrides port
```

**Skip parameters with defaults:**
```kotlin
fun config(a: Int = 1, b: Int = 2, c: Int = 3) { }

config(c = 5)  // Must use named argument after first skip
```

## Lambda Expressions

**Lambda syntax:**
```kotlin
val square = { x: Int -> x * x }
println(square(5))  // 25
```

**Lambda without parameters:**
```kotlin
val greet = { println("Hello") }
greet()  // Hello
```

**Function type:**
```kotlin
val transform: (Int) -> Int = { x -> x * 2 }
val action: () -> Unit = { println("Action") }
```

**Pass lambda to function:**
```kotlin
val numbers = listOf(1, -2, 3, -4)
val positive = numbers.filter { x -> x > 0 }
// [1, 3]
```

**Trailing lambda:**
```kotlin
// When lambda is last parameter, move outside parentheses
numbers.filter { it > 0 }

// When lambda is only parameter, drop parentheses entirely
repeat(3) { println("Hi") }
```

**`it` for single parameter:**
```kotlin
// Explicitly named
numbers.map { x -> x * 2 }

// Implicit 'it' parameter
numbers.map { it * 2 }
```

## Higher-Order Functions

**Accept function as parameter:**
```kotlin
fun operate(a: Int, b: Int, op: (Int, Int) -> Int): Int {
    return op(a, b)
}

val result = operate(5, 3) { x, y -> x + y }  // 8
```

**Return function:**
```kotlin
fun multiplier(factor: Int): (Int) -> Int {
    return { value -> value * factor }
}

val double = multiplier(2)
println(double(5))  // 10
```

## Quick Patterns

| Task | Pattern |
|------|---------|
| Single expression | `fun add(a: Int, b: Int) = a + b` |
| No return | `fun log(msg: String) { println(msg) }` |
| Default param | `fun connect(port: Int = 8080)` |
| Lambda | `val f = { x: Int -> x * 2 }` |
| Pass lambda | `list.filter { it > 0 }` |
| Function type | `(Int, String) -> Boolean` |
| Unit return | `() -> Unit` |

## Common Mistakes

**❌ Unnecessary curly braces:**
```kotlin
fun double(x: Int): Int {
    return x * 2
}
```
**✅ Use single expression:**
```kotlin
fun double(x: Int) = x * 2
```

**❌ Verbose lambda parameter:**
```kotlin
numbers.map { number -> number * 2 }
```
**✅ Use `it` for single parameter:**
```kotlin
numbers.map { it * 2 }
```

**❌ Redundant parentheses with trailing lambda:**
```kotlin
numbers.filter({ it > 0 })
```
**✅ Move lambda outside or drop parentheses:**
```kotlin
numbers.filter { it > 0 }
```

**❌ Declaring function type when unnecessary:**
```kotlin
val action: () -> Unit = { println("Hi") }
```
**✅ Let type inference work:**
```kotlin
val action = { println("Hi") }  // When context is clear
```

**❌ Early return in lambda (doesn't compile):**
```kotlin
fun process(items: List<Int>) {
    items.forEach {
        if (it == 0) return  // Non-local return
    }
}
```
**✅ Use label or change approach:**
```kotlin
items.forEach {
    if (it == 0) return@forEach  // Local return
}
// or
for (item in items) {
    if (item == 0) return  // OK in for loop
}
```

## Useful Higher-Order Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `filter` | Keep matching items | `list.filter { it > 0 }` |
| `map` | Transform items | `list.map { it * 2 }` |
| `forEach` | Perform action on each | `list.forEach { println(it) }` |
| `fold` | Accumulate value | `list.fold(0) { sum, x -> sum + x }` |
| `reduce` | Accumulate without initial | `list.reduce { acc, x -> acc + x }` |
| `any` | Check if any match | `list.any { it > 10 }` |
| `all` | Check if all match | `list.all { it > 0 }` |
| `take` | Take first N items | `list.take(3)` |
| `drop` | Skip first N items | `list.drop(2)` |

## Key Insights

- **Concise syntax** - Single-expression functions reduce boilerplate
- **Trailing lambdas** - Move lambda outside parentheses for readability
- **Type inference** - Compiler infers types for lambdas and functions
- **Functions as values** - Store, pass, and return functions
- **Named arguments** - Improve readability when calling functions

