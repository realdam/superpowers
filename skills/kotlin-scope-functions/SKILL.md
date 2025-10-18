---
name: kotlin-scope-functions
description: Use when initializing objects, performing null-safe operations, chaining transformations, or adding side effects in Kotlin - covers let, apply, run, also, and with for concise object manipulation
---

# Kotlin Scope Functions

## Overview

Scope functions create temporary scopes for object operations, making code more concise and readable.

**Core principle:** Choose the right scope function based on what you return and how you access the object.

## Function Overview

| Function | Access via | Returns | Use Case |
|----------|------------|---------|----------|
| `let` | `it` | Lambda result | Null checks, transformations |
| `apply` | `this` | Object itself | Object initialization |
| `run` | `this` | Lambda result | Initialize and compute |
| `also` | `it` | Object itself | Side effects, logging |
| `with` | `this` | Lambda result | Multiple calls on object |

## let

**Null-safe operations:**
```kotlin
val name: String? = getName()
name?.let {
    println("Name: $it")
    sendEmail(it)
}
// Only executes if name is not null
```

**Transform and use result:**
```kotlin
val length = name?.let {
    it.trim().length
}
```

**Early return with null check:**
```kotlin
val email = user?.email ?: return
val domain = email.substringAfter("@")
```

## apply

**Object initialization:**
```kotlin
val client = HttpClient().apply {
    timeout = 30
    retries = 3
    enableLogging()
}
// Returns configured client
```

**Configure and return:**
```kotlin
fun createUser(name: String) = User(name).apply {
    email = "$name@example.com"
    role = "user"
}
```

## run

**Initialize and compute result:**
```kotlin
val result = client.run {
    connect()
    authenticate()
    getData()
}
// Returns getData() result
```

**Scoped computations:**
```kotlin
val hexColor = run {
    val r = 255
    val g = 128
    val b = 0
    "#%02X%02X%02X".format(r, g, b)
}
```

## also

**Side effects while chaining:**
```kotlin
val numbers = listOf(1, -2, 3, -4)
    .map { it * 2 }
    .also { println("After map: $it") }
    .filter { it > 0 }
    .also { println("After filter: $it") }
// After map: [2, -4, 6, -8]
// After filter: [2, 6]
```

**Logging:**
```kotlin
fun process(user: User) = user
    .also { println("Processing: ${it.name}") }
    .validate()
    .also { println("Validated: ${it.name}") }
    .save()
```

## with

**Multiple calls on object:**
```kotlin
val result = with(canvas) {
    drawCircle(10, 10, 50)
    drawRect(20, 20, 100, 100)
    drawText(30, 30, "Hello")
    getSnapshot()
}
```

**Not an extension - different syntax:**
```kotlin
// with takes object as argument
with(object) { }

// Other scope functions are extensions
object.apply { }
object.let { }
```

## Quick Decision Guide

**Use `let` when:**
- Performing null checks
- Transforming object to different type
- Limiting variable scope
- Chaining operations on nullable

**Use `apply` when:**
- Initializing object
- Configuring object properties
- Builder-style setup
- Want to return configured object

**Use `run` when:**
- Initializing and computing result
- Scoped computations
- Combining initialization with calculation
- Don't need original object back

**Use `also` when:**
- Adding logging
- Debugging pipelines
- Side effects (validation, notifications)
- Want object back after side effect

**Use `with` when:**
- Calling multiple functions on object
- Object passed as parameter
- Not chaining operations
- "With this object, do these things"

## Practical Examples

**Null-safe email sending:**
```kotlin
// Before
val address: String? = getAddress()
if (address != null) {
    sendEmail(address)
}

// After
getAddress()?.let { sendEmail(it) }
```

**Object configuration:**
```kotlin
// Before
val user = User()
user.name = "Alice"
user.email = "alice@example.com"
user.role = "admin"

// After
val user = User().apply {
    name = "Alice"
    email = "alice@example.com"
    role = "admin"
}
```

**Pipeline with logging:**
```kotlin
val result = data
    .filter { it.isValid }
    .also { println("Valid items: ${it.size}") }
    .map { it.transform() }
    .also { println("Transformed: ${it.size}") }
    .sortedBy { it.priority }
```

## Common Mistakes

**❌ Using wrong function for use case:**
```kotlin
// Want object back but using let
val user = User().let {
    it.name = "Alice"
    it  // Must explicitly return
}
```
**✅ Use apply when returning object:**
```kotlin
val user = User().apply {
    name = "Alice"
}
```

**❌ Confusing `this` vs `it`:**
```kotlin
user.apply {
    println(it.name)  // Error: unresolved reference 'it'
}
```
**✅ Know which keyword each function uses:**
```kotlin
user.apply { println(this.name) }  // or just: println(name)
user.also { println(it.name) }
```

**❌ Overusing scope functions:**
```kotlin
val x = value.let { it }.also { it }.run { this }  // Pointless
```
**✅ Use only when beneficial:**
```kotlin
val x = value  // Simple is better
```

**❌ Nested scope functions (hard to read):**
```kotlin
user.apply {
    address.apply {
        city.apply {
            // Too deep
        }
    }
}
```
**✅ Keep nesting minimal:**
```kotlin
user.apply {
    address.city = "NYC"
}
```

## Standard Library Usage

**Scope functions in stdlib:**
```kotlin
// let for null safety
val length = text?.let { it.trim().length }

// apply for initialization
val list = mutableListOf<Int>().apply {
    add(1)
    add(2)
}

// run for scoped computation
val result = run {
    val x = 10
    val y = 20
    x + y
}

// also for debugging
val processed = data
    .map { it * 2 }
    .also { println("Mapped: $it") }
    .filter { it > 10 }

// with for repeated operations
with(builder) {
    append("Hello")
    append(" ")
    append("World")
}
```

## Quick Reference

**Access pattern:**
- `this` - Direct access like inside the class
- `it` - Object passed as lambda parameter

**Return value:**
- Lambda result - `let`, `run`, `with`
- Object itself - `apply`, `also`

**When unsure, use:**
- `let` for nullable handling
- `apply` for configuration
- `also` for logging

## Key Insights

- **Context object** - Temporary scope around an object
- **Two access modes** - `this` (implicit) vs `it` (explicit parameter)
- **Two return types** - Lambda result vs object itself
- **Readability** - Eliminates repeated object names
- **Idiomatic Kotlin** - Widely used in Kotlin codebases
- **Not magic** - Just extension functions with lambda receivers

