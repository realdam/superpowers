---
name: kotlin-advanced
description: Use when building DSLs, using property delegation, creating type-safe builders, or working with advanced Kotlin patterns - covers lambdas with receiver, properties, delegation, and DSL patterns
---

# Kotlin Advanced Features

## Overview

Advanced Kotlin features enable DSL creation, property management, and elegant code patterns through receivers, delegation, and specialized properties.

**Core principle:** Use these features to create readable, type-safe APIs and reduce boilerplate.

## Lambdas with Receiver

**Function type with receiver:**
```kotlin
String.() -> Unit  // Receiver is String
```

**Define and use:**
```kotlin
fun buildString(init: StringBuilder.() -> Unit): String {
    val builder = StringBuilder()
    builder.init()  // Call lambda on receiver
    return builder.toString()
}

val result = buildString {
    append("Hello")  // this: StringBuilder
    append(" World")
}
```

**DSL example:**
```kotlin
class Menu(val name: String) {
    val items = mutableListOf<String>()
    fun item(name: String) {
        items.add(name)
    }
}

fun menu(name: String, init: Menu.() -> Unit): Menu {
    val menu = Menu(name)
    menu.init()
    return menu
}

val mainMenu = menu("Main") {
    item("Home")      // Direct access to Menu functions
    item("Settings")
    item("Exit")
}
```

## Property Accessors

**Custom getters and setters:**
```kotlin
class Person {
    var name: String = ""
        set(value) {
            field = value.replaceFirstChar { it.uppercase() }
        }
        get() = field.trim()
}

val person = Person()
person.name = "  alice  "
println(person.name)  // Alice (trimmed and capitalized)
```

**Backing field with `field` keyword:**
```kotlin
var counter: Int = 0
    set(value) {
        if (value >= 0) {
            field = value  // Use field to avoid infinite recursion
        }
    }
```

**Computed property (no backing field):**
```kotlin
class Rectangle(val width: Int, val height: Int) {
    val area: Int
        get() = width * height  // Computed each access
}
```

## Extension Properties

**Add property to existing class:**
```kotlin
val String.lastChar: Char
    get() = this[length - 1]

println("hello".lastChar)  // o
```

**Must provide getter (no backing field):**
```kotlin
val List<Int>.secondOrNull: Int?
    get() = if (size >= 2) this[1] else null
```

## Property Delegation

**Delegate property management:**
```kotlin
val displayName: String by CachedStringDelegate()
```

**Lazy properties:**
```kotlin
val database: Database by lazy {
    println("Initializing database...")
    Database().apply { connect() }
}

database.query()  // Initialized on first access
database.query()  // Uses cached instance
```

**Observable properties:**
```kotlin
import kotlin.properties.Delegates.observable

var temperature: Double by observable(20.0) { _, old, new ->
    println("Temperature changed: $old°C -> $new°C")
}

temperature = 25.0  // Temperature changed: 20.0°C -> 25.0°C
```

**Map-backed properties:**
```kotlin
class User(map: Map<String, Any>) {
    val name: String by map
    val age: Int by map
}

val user = User(mapOf("name" to "Alice", "age" to 25))
println(user.name)  // Alice
```

## Delegation Patterns

**Lazy initialization:**
```kotlin
val expensiveResource by lazy {
    loadExpensiveResource()
}
// Only loaded when first accessed
```

**Observable changes:**
```kotlin
var score: Int by observable(0) { _, old, new ->
    if (new > old) {
        println("Score increased!")
    }
}
```

**Custom delegate:**
```kotlin
class CachedValue<T> {
    private var cached: T? = null
    
    operator fun getValue(thisRef: Any?, property: Any?): T {
        return cached ?: computeValue().also { cached = it }
    }
    
    operator fun setValue(thisRef: Any?, property: Any?, value: T) {
        cached = value
    }
}

var data: String by CachedValue()
```

## Quick Patterns

| Feature | Pattern | Use Case |
|---------|---------|----------|
| Lambda with receiver | `Type.() -> Unit` | DSL building |
| Custom getter | `val x get() = computed` | Derived values |
| Custom setter | `set(value) { field = value }` | Validation |
| Lazy property | `by lazy { init }` | Expensive initialization |
| Observable | `by observable(initial) { }` | Change tracking |
| Extension property | `val Type.prop get()` | Add computed property |
| Delegation | `by Delegate()` | Reuse property logic |

## Standard Library Delegates

**Available delegates:**
```kotlin
// Lazy
val heavy by lazy { HeavyObject() }

// Observable
var value by Delegates.observable(0) { _, old, new -> }

// Vetoable (can reject changes)
var checked by Delegates.vetoable(false) { _, old, new ->
    new != old  // Only allow if value actually changes
}

// NotNull (for lateinit var alternative)
var notNull: String by Delegates.notNull()
```

## Common Mistakes

**❌ Using mutable property when computed is better:**
```kotlin
class Rectangle(val width: Int, val height: Int) {
    var area: Int = width * height  // Stale if dimensions change
}
```
**✅ Use computed property:**
```kotlin
class Rectangle(val width: Int, val height: Int) {
    val area: Int
        get() = width * height
}
```

**❌ Infinite recursion in setter:**
```kotlin
var name: String = ""
    set(value) {
        name = value.uppercase()  // Calls setter recursively!
    }
```
**✅ Use backing field:**
```kotlin
var name: String = ""
    set(value) {
        field = value.uppercase()
    }
```

**❌ Not returning from lambda with receiver:**
```kotlin
fun build(init: Builder.() -> Unit): Result {
    val builder = Builder()
    builder.init()
    // Forgot to return builder.build()
}
```
**✅ Return the result:**
```kotlin
fun build(init: Builder.() -> Unit): Result {
    val builder = Builder()
    builder.init()
    return builder.build()
}
```

## DSL Building

**Type-safe builder pattern:**
```kotlin
class HTML {
    fun body(init: Body.() -> Unit) {
        Body().init()
    }
}

class Body {
    fun h1(text: String) {
        println("<h1>$text</h1>")
    }
}

fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()
    html.init()
    return html
}

// Usage
html {
    body {
        h1("Welcome")
    }
}
```

**Standard library examples:**
```kotlin
// buildList
val list = buildList {
    add(1)
    add(2)
    addAll(listOf(3, 4))
}

// buildString
val text = buildString {
    append("Hello")
    append(" ")
    append("World")
}

// apply (built-in DSL pattern)
val user = User().apply {
    name = "Alice"
    email = "alice@example.com"
}
```

## Property Patterns

**Validate on set:**
```kotlin
var age: Int = 0
    set(value) {
        require(value >= 0) { "Age cannot be negative" }
        field = value
    }
```

**Transform on set:**
```kotlin
var email: String = ""
    set(value) {
        field = value.lowercase().trim()
    }
```

**Computed from other properties:**
```kotlin
class Person(val firstName: String, val lastName: String) {
    val fullName: String
        get() = "$firstName $lastName"
}
```

## Advanced Delegation

**Delegate to another property:**
```kotlin
class User {
    var email: String = ""
    var username: String by ::email  // Delegates to email
}
```

**Provide delegate:**
```kotlin
class LoggingDelegate<T>(private var value: T) {
    operator fun getValue(thisRef: Any?, property: Any?): T {
        println("Getting ${property?.toString()}: $value")
        return value
    }
    
    operator fun setValue(thisRef: Any?, property: Any?, newValue: T) {
        println("Setting ${property?.toString()}: $value -> $newValue")
        value = newValue
    }
}

var name: String by LoggingDelegate("Alice")
```

## Key Insights

- **Receivers** - Lambdas can have implicit `this` context
- **DSL patterns** - Lambdas with receivers enable type-safe builders
- **Property delegation** - Reuse property behavior across classes
- **Lazy by default is thread-safe** - Safe for concurrent access
- **Custom delegates** - `getValue` and `setValue` operators
- **Extension properties** - Must be computed (no backing field)
- **Backing fields** - Use `field` keyword to avoid recursion

