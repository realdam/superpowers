---
name: kotlin-collections
description: Use when storing and manipulating groups of data, working with sequences of items, or managing key-value pairs in Kotlin - covers lists, sets, maps with read-only and mutable variants
---

# Kotlin Collections

## Overview

Kotlin provides collections with distinct read-only and mutable variants for safer data handling.

**Core principle:** Prefer read-only collections by default - mutability should be explicit and localized.

## Collection Types

| Type | Read-Only | Mutable | Purpose |
|------|-----------|---------|---------|
| List | `List<T>` | `MutableList<T>` | Ordered, allows duplicates |
| Set | `Set<T>` | `MutableSet<T>` | Unordered, unique items |
| Map | `Map<K,V>` | `MutableMap<K,V>` | Key-value pairs |

## Lists

**Create read-only list:**
```kotlin
val colors = listOf("red", "green", "blue")
```

**Create mutable list:**
```kotlin
val colors: MutableList<String> = mutableListOf("red", "green")
colors.add("blue")
colors.remove("red")
```

**Access elements:**
```kotlin
val first = colors[0]           // Index access
val firstItem = colors.first()  // Extension function
val lastItem = colors.last()    // Extension function
```

**Check and count:**
```kotlin
if ("red" in colors) { }        // Check membership
val size = colors.count()       // Get count
```

## Sets

**Create read-only set:**
```kotlin
val ids = setOf(1, 2, 3, 3)  // [1, 2, 3] - duplicates removed
```

**Create mutable set:**
```kotlin
val ids = mutableSetOf(1, 2, 3)
ids.add(4)
ids.remove(1)
```

**Check and count:**
```kotlin
if (42 in ids) { }
val size = ids.count()
```

## Maps

**Create read-only map:**
```kotlin
val scores = mapOf("Alice" to 100, "Bob" to 95)
```

**Create mutable map:**
```kotlin
val scores = mutableMapOf("Alice" to 100)
scores["Bob"] = 95              // Add entry
scores.remove("Alice")          // Remove entry
```

**Access values:**
```kotlin
val score = scores["Alice"]     // Returns Int? (nullable)
val score = scores["Alice"] ?: 0  // With default
```

**Check keys and values:**
```kotlin
if ("Alice" in scores) { }              // Check key
if (scores.containsKey("Alice")) { }    // Explicit check
val keys = scores.keys                  // Get all keys
val values = scores.values              // Get all values
```

## Common Operations

**Iteration:**
```kotlin
for (item in list) {
    println(item)
}

for ((key, value) in map) {
    println("$key: $value")
}
```

**Transformation:**
```kotlin
val doubled = numbers.map { it * 2 }
val evens = numbers.filter { it % 2 == 0 }
val sum = numbers.sum()
val joined = items.joinToString(", ")
```

**Checking:**
```kotlin
val hasNegative = numbers.any { it < 0 }
val allPositive = numbers.all { it > 0 }
val empty = list.isEmpty()
```

## Read-Only Views

**Prevent unwanted mutations:**
```kotlin
val mutable = mutableListOf(1, 2, 3)
val readOnly: List<Int> = mutable
// readOnly.add(4)  // Compiler error
```

This creates a read-only view - the underlying list can still change through the mutable reference.

## Quick Reference

| Operation | List | Set | Map |
|-----------|------|-----|-----|
| Create read-only | `listOf()` | `setOf()` | `mapOf()` |
| Create mutable | `mutableListOf()` | `mutableSetOf()` | `mutableMapOf()` |
| Add item | `.add(item)` | `.add(item)` | `[key] = value` |
| Remove item | `.remove(item)` | `.remove(item)` | `.remove(key)` |
| Check contains | `item in collection` | `item in collection` | `key in map` |
| Get size | `.count()` or `.size` | `.count()` or `.size` | `.count()` or `.size` |
| Access element | `[index]` | N/A | `[key]` |

## Common Mistakes

**❌ Using mutable by default:**
```kotlin
val items = mutableListOf(1, 2, 3)  // Unnecessary mutability
```
**✅ Use read-only unless mutation needed:**
```kotlin
val items = listOf(1, 2, 3)
```

**❌ Not handling null from map access:**
```kotlin
val score: Int = scores["Alice"]  // Compiler error - might be null
```
**✅ Handle nullable result:**
```kotlin
val score = scores["Alice"] ?: 0
```

**❌ Modifying while iterating:**
```kotlin
for (item in list) {
    list.remove(item)  // ConcurrentModificationException
}
```
**✅ Use iterator or create new collection:**
```kotlin
list.removeIf { condition }
// or
val filtered = list.filter { !condition }
```

## When to Use Which

**Use List when:**
- Order matters
- Duplicates are allowed
- Need index-based access

**Use Set when:**
- Only unique items needed
- Order doesn't matter
- Fast membership testing required

**Use Map when:**
- Key-value associations needed
- Fast lookup by key required
- Each key maps to one value

## Key Insights

- **Immutable by default** - Read-only collections prevent accidental modification
- **Type-safe** - Generics enforce type consistency
- **Rich API** - Extension functions like `map`, `filter`, `reduce` on all collections
- **Null-safe** - Collection operations handle nullability explicitly
- **View casting** - Mutable collections can be cast to read-only views

