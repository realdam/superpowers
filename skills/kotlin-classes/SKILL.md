---
name: kotlin-classes
description: Use when organizing code with classes, creating data structures, implementing inheritance, or using polymorphism in Kotlin - covers classes, data classes, inheritance, interfaces, and object-oriented patterns
---

# Kotlin Classes and OOP

## Overview

Kotlin provides concise class syntax with powerful OOP features including data classes, sealed classes, and flexible inheritance.

**Core principle:** Classes are final by default - inheritance must be explicitly allowed.

## Basic Classes

**Simple class:**
```kotlin
class Customer
```

**Class with properties:**
```kotlin
class Contact(val id: Int, var email: String)
```

**Properties in class body:**
```kotlin
class Contact(val id: Int, var email: String) {
    val category: String = "work"
}
```

**Create instance:**
```kotlin
val contact = Contact(1, "alice@example.com")
```

**Access properties:**
```kotlin
println(contact.email)
contact.email = "new@example.com"  // Mutable property
```

**Member functions:**
```kotlin
class Contact(val id: Int, var email: String) {
    fun printId() {
        println(id)
    }
}

contact.printId()
```

## Data Classes

**Declare with `data` keyword:**
```kotlin
data class User(val name: String, val id: Int)
```

**Automatic functions:**
```kotlin
val user = User("Alice", 1)

// toString()
println(user)  // User(name=Alice, id=1)

// equals()
val user2 = User("Alice", 1)
println(user == user2)  // true

// copy()
val user3 = user.copy(name = "Bob")
println(user3)  // User(name=Bob, id=1)
```

## Inheritance

**Abstract classes (inheritable by default):**
```kotlin
abstract class Product(val name: String, var price: Double) {
    abstract val category: String
    
    fun productInfo() = "Product: $name, Category: $category"
}

class Electronic(name: String, price: Double) : Product(name, price) {
    override val category = "Electronic"
}
```

**Open classes (explicitly inheritable):**
```kotlin
open class Vehicle(val make: String, val model: String) {
    open fun displayInfo() {
        println("Vehicle: $make $model")
    }
}

class Car(make: String, model: String, val doors: Int) : Vehicle(make, model) {
    override fun displayInfo() {
        println("Car: $make $model with $doors doors")
    }
}
```

## Interfaces

**Define interface:**
```kotlin
interface PaymentMethod {
    fun initiatePayment(amount: Double): String
}
```

**Implement interface:**
```kotlin
class CreditCard(val cardNumber: String) : PaymentMethod {
    override fun initiatePayment(amount: Double): String {
        return "Paying $amount with card ${cardNumber.takeLast(4)}"
    }
}
```

**Multiple interfaces:**
```kotlin
class CreditCard : PaymentMethod, Refundable {
    override fun initiatePayment(amount: Double): String { }
    override fun refund(amount: Double): String { }
}
```

**Interface delegation:**
```kotlin
class PaymentProxy(method: PaymentMethod) : PaymentMethod by method {
    // Delegates all PaymentMethod functions to method
    // Override only what you need to change
}
```

## Object Declarations

**Singleton object:**
```kotlin
object Database {
    fun connect() {
        println("Connecting...")
    }
}

Database.connect()  // No instance needed
```

**Data object:**
```kotlin
data object AppConfig {
    var appName = "MyApp"
    var version = "1.0.0"
}

println(AppConfig)  // AppConfig
```

**Companion object:**
```kotlin
class User(val name: String) {
    companion object {
        fun create(name: String) = User(name)
        const val DEFAULT_NAME = "Guest"
    }
}

val user = User.create("Alice")
println(User.DEFAULT_NAME)
```

## Special Classes

**Sealed classes (restricted hierarchy):**
```kotlin
sealed class Result
data class Success(val data: String) : Result()
data class Error(val message: String) : Result()
object Loading : Result()

fun handle(result: Result) = when (result) {
    is Success -> println(result.data)
    is Error -> println(result.message)
    Loading -> println("Loading...")
    // No else needed - compiler knows all cases
}
```

**Enum classes:**
```kotlin
enum class State {
    IDLE, RUNNING, FINISHED
}

enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF);
    
    fun containsRed() = (rgb and 0xFF0000 != 0)
}

val state = State.RUNNING
val color = Color.RED
println(color.rgb)           // 16711680
println(color.containsRed()) // true
```

**Inline value classes (performance):**
```kotlin
@JvmInline
value class Email(val address: String)

fun sendEmail(email: Email) {
    println("Sending to ${email.address}")
}

val email = Email("user@example.com")
sendEmail(email)
```

## Quick Reference

| Feature | Syntax | Purpose |
|---------|--------|---------|
| Basic class | `class Name` | Define custom type |
| Data class | `data class User(val id: Int)` | Auto-generated `equals`, `toString`, `copy` |
| Abstract class | `abstract class Base` | Provide shared implementation |
| Open class | `open class Base` | Allow inheritance |
| Interface | `interface Clickable` | Define contract |
| Object | `object Singleton` | Single instance |
| Companion | `companion object { }` | Class-level members |
| Sealed class | `sealed class Result` | Restricted hierarchies |
| Enum | `enum class State` | Fixed set of values |
| Inline value | `@JvmInline value class Id(val value: Int)` | Zero-cost wrapper |

## Common Mistakes

**❌ Making classes open unnecessarily:**
```kotlin
open class User { }  // Why open if not inheriting?
```
**✅ Keep classes final (default):**
```kotlin
class User { }
```

**❌ Regular class instead of data class:**
```kotlin
class User(val name: String, val id: Int) {
    override fun toString() = "User(name=$name, id=$id)"
    override fun equals(other: Any?) = // ...
    // Manual boilerplate
}
```
**✅ Use data class:**
```kotlin
data class User(val name: String, val id: Int)
```

**❌ Object when class is needed:**
```kotlin
object User {
    var name: String = ""  // Shared across all uses!
}
```
**✅ Use class for multiple instances:**
```kotlin
class User(var name: String)
```

## When to Use Which

**Use regular class when:**
- Need multiple instances
- Standard object behavior
- No special requirements

**Use data class when:**
- Primarily storing data
- Need `equals`, `toString`, `copy`
- Immutable value objects

**Use object when:**
- Need singleton
- Utility functions
- Single configuration

**Use sealed class when:**
- Limited set of subclasses
- Exhaustive `when` expressions
- Type-safe state machines

**Use enum when:**
- Fixed set of constants
- Each constant is same type
- Need to iterate all values

**Use interface when:**
- Multiple inheritance needed
- Define contract only
- Want delegation

**Use abstract class when:**
- Share implementation
- Single inheritance
- Need constructor

## Key Insights

- **Final by default** - Classes can't be inherited unless `open` or `abstract`
- **Data classes** - Automatic `equals`, `hashCode`, `toString`, `copy`
- **Single inheritance** - One parent class, multiple interfaces
- **Delegation** - Reduce boilerplate with `by` keyword
- **Object declarations** - Built-in singleton pattern
- **Sealed hierarchies** - Exhaustive when expressions

